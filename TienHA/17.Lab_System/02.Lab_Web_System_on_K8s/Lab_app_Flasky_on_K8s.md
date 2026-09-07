# LAB: Deploy Flasky từ Docker lên Kubernetes trong VMware

## 1. Mục tiêu và phạm vi

Lab này chuyển hệ thống Flasky [này](https://github.com/tiend9/system-intership/blob/master/TienHA/17.Lab_System/01.Lab_Default_Web_System/03.Practice_Deploy_App_Flasky.md) đang chạy Docker Compose gồm Flasky, MariaDB, Redis và MinIO sang Kubernetes tự dựng trên các VM VMware. Hai mục tiêu chính là:

- Flasky chạy ít nhất 2 replica, được cân bằng tải và tự tạo lại Pod khi lỗi.
- Database, cache và object storage chỉ được truy cập nội bộ trong cluster.

Đây là **môi trường lab có độ sẵn sàng cho tầng ứng dụng**, không phải HA hoàn chỉnh cho database.

> Giả định: ứng dụng đã được chỉnh như lab Docker trước đó, tức có DATABASE_URL, REDIS_URL, MINIO_ENDPOINT, MINIO_ACCESS_KEY, MINIO_SECRET_KEY và MINIO_BUCKET. Image Flasky lắng nghe cổng 5000.

## 2. Thiết kế node đề xuất

### 2.1. Phương án dùng cho lab: 3 VM

**Sơ đồ lab**:

![Lab_WebSystem_K8s](./images/Lab_WebSystem_K8s_1.png)

Tại sao chọn 3 node:

- Control plane được tách khỏi workload; không chạy Flasky trên k8s-cp1.
- Hai worker giúp hai replica Flasky nằm trên hai VM khác nhau. Khi một worker tắt, ứng dụng vẫn còn một replica.
- MariaDB, Redis, MinIO của lab chỉ một replica và dùng local disk nên **không chịu được việc máy chủ worker (or VM) chứa volume bị mất**. (Máy hạ tầng mà lỗi không chữa được là dữ liệu Database sẽ mất - đây là rủi ro **Single Point of Failure** )

  - Trong production sẽ sửa điểm yếu này bằng cách backup dữ liệu vào 1 máy NFS/S3 khác.

Quy hoạch trên 3 Node:

| VM   | Hostname | IP node    | vCPU / RAM / disk  | Vai trò                                   |
| ---- | -------- | ---------- | ------------------ | ----------------------------------------- |
| VM 1 | k8s-cp1  | 10.0.40.50 | 2 / 4 GiB / 40 GiB | Kubernetes control plane                  |
| VM 2 | k8s-w1   | 10.0.40.51 | 2 / 6 GiB / 60 GiB | Worker, chạy Flasky hoặc dịch vụ stateful |
| VM 3 | k8s-w2   | 10.0.40.52 | 2 / 6 GiB / 60 GiB | Worker, chạy Flasky và ingress            |

Tạo thêm một IP trống, ngoài DHCP và ngoài các IP VM, cho MetalLB:(VMWare bật chế độ `Bridge` cho dải `10.0.40.0/24` và `unable DHCP` của máy host)

| Thành phần                 | IP                            |
| -------------------------- | ----------------------------- |
| MetalLB pool               | 10.0.40.60 - 10.0.40.69       |
| VIP ingress Flasky dự kiến | 10.0.40.60                    |
| DNS/hosts                  | flasky.lab.local → 10.0.40.60 |

## 3. Mapping các thành phần from Docker Compose sang Kubernetes

| Docker Compose trước đây   | Kubernetes                                        |
| -------------------------- | ------------------------------------------------- |
| Container Flasky web1/web2 | Deployment flasky, replicas: 2                    |
| HAProxy                    | ingress-nginx Service kiểu LoadBalancer + Ingress |
| MariaDB container          | StatefulSet mariadb + PVC                         |
| Redis container            | StatefulSet redis + PVC                           |
| MinIO container            | StatefulSet minio + PVC                           |
| docker-compose và .env     | Secret và ConfigMap                               |
| restart: always            | Deployment/StatefulSet controller                 |
| Docker bridge network      | Service DNS nội bộ + Cilium NetworkPolicy         |
| bind mount dữ liệu         | PersistentVolumeClaim                             |

## 4. Chuẩn bị trên cả 3 VM

Ví dụ dưới đây dùng Ubuntu Server 22.04/24.04 và Kubernetes 1.37. Chạy các lệnh phần này trên **cả k8s-cp1, k8s-w1 và k8s-w2**; thay hostname tương ứng trước.

### 4.1. Chuẩn bị OS, kernel và containerd

```bash
sudo hostnamectl set-hostname k8s-cp1   # đổi thành k8s-w1 hoặc k8s-w2 trên worker

sudo tee -a /etc/hosts > /dev/null <<'EOF'
10.0.40.50 k8s-cp1
10.0.40.51 k8s-w1
10.0.40.52 k8s-w2
EOF

sudo swapoff -a
sudo sed -ri '/\sswap\s/s/^#?/#/' /etc/fstab

sudo tee /etc/modules-load.d/k8s.conf > /dev/null <<'EOF'
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/sysctl.d/99-kubernetes-cri.conf > /dev/null <<'EOF'
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

sudo apt-get update
sudo apt-get install -y ca-certificates curl gpg containerd

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl enable --now containerd
```

Kiểm tra trước khi đi tiếp:

```bash
swapon --show                 # không được có output
sysctl net.ipv4.ip_forward    # phải là 1
systemctl is-active containerd
ip route show                 # default route phải đi qua NIC 10.0.40.x
```

### 4.2. Cài kubelet, kubeadm, kubectl

Kubernetes tách repository theo từng minor version. Đoạn dưới dùng 1.37; nếu chọn version khác phải thay đồng thời repository và đọc đúng tài liệu của version đó.

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.37/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.37/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

> kubelet có thể restart liên tục trước khi node được kubeadm bootstrap; đó là trạng thái bình thường.

## 5. Tạo cluster

### 5.1. Khởi tạo control plane trên k8s-cp1

```bash
sudo kubeadm init \
  --apiserver-advertise-address=10.0.40.50 \
  --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown "$(id -u):$(id -g)" $HOME/.kube/config

kubectl get nodes
sudo kubeadm token create --print-join-command
```

Lệnh cuối in ra một lệnh `kubeadm` join có token. Token là thông tin nhạy cảm và hết hạn; không đưa nó vào tài liệu hoặc Git.

### 5.2. Join worker

Chạy đúng lệnh join vừa sinh ra trên k8s-w1 và k8s-w2. Ví dụ minh họa:

```bash
sudo kubeadm join 10.0.40.50:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

Trên k8s-cp1 xác nhận:

```bash
kubectl get nodes -o wide
```

Node sẽ là NotReady cho tới khi CNI được cài ở bước kế tiếp.

## 6. Cài các thành phần nền của cluster

### 6.1. Cilium CNI

Cilium cung cấp Pod networking và thực thi NetworkPolicy. Cài Helm trên k8s-cp1 (nếu chưa có). Sau đó:

```bash
curl -fsSL -o /tmp/get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 /tmp/get_helm.sh
/tmp/get_helm.sh

helm version

helm repo add cilium https://helm.cilium.io/
helm repo update
helm upgrade --install cilium cilium/cilium \
  --version 1.20.1 \
  --namespace kube-system \
  --set ipam.mode=kubernetes

kubectl -n kube-system rollout status daemonset/cilium --timeout=5m
kubectl -n kube-system rollout status deployment/cilium-operator --timeout=5m
kubectl get nodes -o wide
```

![Lab_WebSystem_K8s](./images/Lab_WebSystem_K8s_2.png)

Cả ba node phải `Ready` trước khi triển khai ứng dụng.

### 6.2. Storage cho lab: Local Path Provisioner

VMware Workstation không có vSphere CSI như môi trường vCenter. Lab dùng local-path-provisioner để tạo PVC trên disk local của worker. Cần hiểu rằng PVC chỉ nằm ở một worker(ta đặt trên worker 1); hãy backup dữ liệu ra bên ngoài.

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.36/deploy/local-path-storage.yaml
kubectl -n local-path-storage rollout status deployment/local-path-provisioner --timeout=5m
kubectl get storageclass

# Gắn nhãn worker sẽ chứa MariaDB, Redis và MinIO của lab
kubectl label node k8s-w1 node-role.flasky.io/stateful=true
```

Local Path Provisioner sử dụng thư mục `/opt/local-path-provisioner` ở mỗi node. Đảm bảo VMDK của `k8s-w1` đủ dung lượng trước khi dùng. Snapshot VMware không thay thế backup nhất quán của MariaDB.

### 6.3. MetalLB: cấp VIP cho LoadBalancer

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.16.1/config/manifests/metallb-native.yaml
kubectl -n metallb-system wait --for=condition=available deployment/controller --timeout=5m
```

Tạo file k8s/00-metallb-pool.yaml:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: lan-pool
  namespace: metallb-system
spec:
  addresses:
    - 10.0.40.60-10.0.40.69
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: lan-l2
  namespace: metallb-system
spec:
  ipAddressPools:
    - lan-pool
  nodeSelectors:
    - matchLabels:
        kubernetes.io/hostname: k8s-w1
    - matchLabels:
        kubernetes.io/hostname: k8s-w2
```

Apply & Check:

```bash
kubectl apply -f k8s/00-metallb-pool.yaml
kubectl -n metallb-system get ipaddresspool,l2advertisement
```

Chỉ chọn hai worker làm nơi quảng bá VIP. MetalLB L2 chỉ có một node quảng bá một VIP tại một thời điểm; nếu node đó lỗi, speaker ở worker còn lại tiếp quản.

### 6.4. ingress-nginx

```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer \
  --set controller.replicaCount=2 \
  --set controller.ingressClassResource.default=true

kubectl -n ingress-nginx rollout status deployment/ingress-nginx-controller --timeout=5m
kubectl -n ingress-nginx get svc ingress-nginx-controller
```

![Lab_WebSystem_K8s](./images/Lab_WebSystem_K8s_3.png)

Đợi đến khi EXTERNAL-IP của Service là `10.0.40.60`. Tạo bản ghi DNS nội bộ hoặc thêm vào file hosts của máy truy cập:

```text
10.0.40.60 flasky.lab.local
```

## 7. Chuẩn bị image Flasky

Build image từ source code máy chủ backend bài lab WebSystem Docker.

### 7.1. Điều chỉnh code để hợp Kubernetes

Không chạy database migration trong entrypoint của từng Flasky Pod. Khi có 2 replica, hai Pod cùng chạy flask db upgrade có thể race condition. Sửa boot.sh trong source Docker hiện tại thành chỉ khởi động Gunicorn:

```bash
#!/bin/sh
set -eu
exec gunicorn -b 0.0.0.0:5000 --workers 2 --access-logfile - --error-logfile - manage:app
```

Thêm endpoint không yêu cầu đăng nhập vào app/main/views.py để Kubernetes kiểm tra sống/chờ sẵn sàng:

```python
@main.route('/healthz')
def healthz():
    return '', 200
```

Sửa dòng SSLify trong `./app/__init__.py` (dòng 39-40) thành:

```python
sslify = SSLify(app, skips=['main.healthz'])
```

Database migration sẽ chạy bằng Kubernetes Job ở bước 9. Không sửa trực tiếp revision alembic trong database nếu chưa kiểm tra lịch sử migration; cách này chỉ dành cho xử lý sự cố đã được hiểu rõ.

### 7.2. Build và push image

Mỗi worker phải pull được image. Cách đơn giản nhất cho lab là dùng Docker Hub hoặc GHCR private/public. Thay DOCKERHUB_USER và tag theo release thực tế:

```bash
docker login -u DOCKERHUB_USER
docker build -t flasky:v4 .
docker tag flasky:v4 DOCKERHUB_USER/flasky:v4
docker push DOCKERHUB_USER/flasky:v4
```

Nếu registry là private, tạo imagePullSecret trước khi deploy:

```bash
kubectl create namespace flasky
kubectl -n flasky create secret docker-registry regcred \
  --docker-server=<registry> \
  --docker-username=<user> \
  --docker-password=<password>
```

Sửa file Dockerfile add thêm venv path :

```bash
RUN venv/bin/pip install -r requirements/common.txt
RUN venv/bin/pip install gunicorn pymysql Flask-SSLify

ENV PATH="/home/flasky/venv/bin:$PATH" # thêm vào đây

COPY app app
COPY migrations migrations
```

Thêm dòng đọc env trực tiếp vào ProductionConfig:

```python
class ProductionConfig(Config):
    SSL_DISABLE = bool(os.environ.get('SSL_DISABLE'))
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data.sqlite')
```

Phần manifest dưới đây mặc định image public. Với registry private, bỏ comment imagePullSecrets trong Deployment.

## 8. Manifest ứng dụng

Tạo thư mục k8s, rồi tạo lần lượt ba file ở các phần dưới. Không commit Secret chứa mật khẩu thật lên Git. Ví dụ dùng stringData cho lab để tránh phải base64 thủ công; production dùng External Secrets hoặc Sealed Secrets.

```bash
#Sinh chuỗi ngẫu nhiên dựa hash base64
openssl rand -base64 24
```

### 8.1. k8s/01-infra.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: flasky
---
apiVersion: v1
kind: Secret
metadata:
  name: flasky-secrets
  namespace: flasky
type: Opaque
stringData:
  MARIADB_ROOT_PASSWORD: "THAY_MAT_KHAU_ROOT_DAI_NGAU_NHIEN"
  MARIADB_PASSWORD: "THAY_MAT_KHAU_APP_DAI_NGAU_NHIEN"
  REDIS_PASSWORD: "THAY_MAT_KHAU_REDIS_DAI_NGAU_NHIEN"
  MINIO_ROOT_USER: "minio-admin"
  MINIO_ROOT_PASSWORD: "THAY_MAT_KHAU_MINIO_ROOT_DAI_NGAU_NHIEN"
  MINIO_APP_ACCESS_KEY: "flasky-app"
  MINIO_APP_SECRET_KEY: "THAY_MINIO_APP_SECRET_DAI_NGAU_NHIEN"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: flasky-config
  namespace: flasky
data:
  FLASK_CONFIG: "production"
  MINIO_BUCKET: "flasky-avatars"
  MINIO_ENDPOINT: "minio.flasky.svc.cluster.local:9000"
---
apiVersion: v1
kind: Service
metadata:
  name: mariadb
  namespace: flasky
spec:
  clusterIP: None
  selector:
    app.kubernetes.io/name: mariadb
  ports:
    - name: mysql
      port: 3306
      targetPort: mysql
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mariadb
  namespace: flasky
spec:
  serviceName: mariadb
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: mariadb
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mariadb
        app.kubernetes.io/part-of: flasky
    spec:
      nodeSelector:
        node-role.flasky.io/stateful: "true"
      containers:
        - name: mariadb
          image: mariadb:11.4
          imagePullPolicy: IfNotPresent
          ports:
            - name: mysql
              containerPort: 3306
          env:
            - name: MARIADB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MARIADB_ROOT_PASSWORD
            - name: MARIADB_DATABASE
              value: flasky_db
            - name: MARIADB_USER
              value: flasky
            - name: MARIADB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MARIADB_PASSWORD
          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
          startupProbe:
            exec:
              command:
                - sh
                - -c
                - mariadb-admin ping -h 127.0.0.1 -uroot -p"$MARIADB_ROOT_PASSWORD"
            failureThreshold: 30
            periodSeconds: 5
          readinessProbe:
            exec:
              command:
                - sh
                - -c
                - mariadb-admin ping -h 127.0.0.1 -uroot -p"$MARIADB_ROOT_PASSWORD"
            periodSeconds: 10
          resources:
            requests:
              cpu: 250m
              memory: 512Mi
            limits:
              cpu: "1"
              memory: 1Gi
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: local-path
        resources:
          requests:
            storage: 8Gi
---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: flasky
spec:
  selector:
    app.kubernetes.io/name: redis
  ports:
    - name: redis
      port: 6379
      targetPort: redis
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: flasky
spec:
  serviceName: redis
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: redis
  template:
    metadata:
      labels:
        app.kubernetes.io/name: redis
        app.kubernetes.io/part-of: flasky
    spec:
      nodeSelector:
        node-role.flasky.io/stateful: "true"
      containers:
        - name: redis
          image: redis:7-alpine
          command: ["sh", "-c"]
          args:
            - exec redis-server --appendonly yes --requirepass "$REDIS_PASSWORD"
          ports:
            - name: redis
              containerPort: 6379
          env:
            - name: REDIS_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: REDIS_PASSWORD
          volumeMounts:
            - name: data
              mountPath: /data
          startupProbe:
            tcpSocket:
              port: redis
            failureThreshold: 30
            periodSeconds: 5
          readinessProbe:
            exec:
              command:
                ["sh", "-c", 'redis-cli -a "$REDIS_PASSWORD" ping | grep PONG']
            periodSeconds: 10
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: local-path
        resources:
          requests:
            storage: 2Gi
---
apiVersion: v1
kind: Service
metadata:
  name: minio
  namespace: flasky
spec:
  selector:
    app.kubernetes.io/name: minio
  ports:
    - name: api
      port: 9000
      targetPort: api
    - name: console
      port: 9001
      targetPort: console
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: minio
  namespace: flasky
spec:
  serviceName: minio
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: minio
  template:
    metadata:
      labels:
        app.kubernetes.io/name: minio
        app.kubernetes.io/part-of: flasky
    spec:
      nodeSelector:
        node-role.flasky.io/stateful: "true"
      containers:
        - name: minio
          image: minio/minio:latest
          args: ["server", "/data", "--console-address", ":9001"]
          ports:
            - name: api
              containerPort: 9000
            - name: console
              containerPort: 9001
          env:
            - name: MINIO_ROOT_USER
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MINIO_ROOT_USER
            - name: MINIO_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MINIO_ROOT_PASSWORD
          volumeMounts:
            - name: data
              mountPath: /data
          startupProbe:
            httpGet:
              path: /minio/health/live
              port: api
            failureThreshold: 30
            periodSeconds: 5
          readinessProbe:
            httpGet:
              path: /minio/health/ready
              port: api
            periodSeconds: 10
          resources:
            requests:
              cpu: 250m
              memory: 512Mi
            limits:
              cpu: "1"
              memory: 1Gi
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: local-path
        resources:
          requests:
            storage: 20Gi
---
apiVersion: batch/v1
kind: Job
metadata:
  name: minio-bootstrap
  namespace: flasky
spec:
  backoffLimit: 6
  template:
    metadata:
      labels:
        app.kubernetes.io/name: minio-bootstrap
        app.kubernetes.io/part-of: flasky
    spec:
      restartPolicy: OnFailure
      containers:
        - name: mc
          image: minio/mc:latest
          command: ["sh", "-c"]
          args:
            - |
              until mc alias set local http://minio:9000 "$MINIO_ROOT_USER" "$MINIO_ROOT_PASSWORD"; do sleep 5; done
              mc mb --ignore-existing local/flasky-avatars
              cat > /tmp/flasky-policy.json <<'EOF'
              {
                "Version":"2012-10-17",
                "Statement":[{
                  "Effect":"Allow",
                  "Action":["s3:GetObject","s3:PutObject","s3:ListBucket"],
                  "Resource":["arn:aws:s3:::flasky-avatars","arn:aws:s3:::flasky-avatars/*"]
                }]
              }
              EOF
              mc admin policy create local flasky-avatars-rw /tmp/flasky-policy.json || true
              mc admin user add local "$MINIO_APP_ACCESS_KEY" "$MINIO_APP_SECRET_KEY" || true
              mc admin policy attach local flasky-avatars-rw --user "$MINIO_APP_ACCESS_KEY"
          env:
            - name: MINIO_ROOT_USER
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MINIO_ROOT_USER
            - name: MINIO_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MINIO_ROOT_PASSWORD
            - name: MINIO_APP_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MINIO_APP_ACCESS_KEY
            - name: MINIO_APP_SECRET_KEY
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MINIO_APP_SECRET_KEY
```

Áp dụng và chờ hạ tầng:

```bash
kubectl apply -f k8s/01-infra.yaml
kubectl -n flasky get pods,pvc
kubectl -n flasky rollout status statefulset/mariadb --timeout=10m
kubectl -n flasky rollout status statefulset/redis --timeout=10m
kubectl -n flasky rollout status statefulset/minio --timeout=10m
kubectl -n flasky wait --for=condition=complete job/minio-bootstrap --timeout=10m
```

### 8.2. k8s/02-migrate.yaml

**Lưu ý quan trọng**: Job này chạy migration **một lần cho mỗi release**. Trước lần deploy mới, xóa Job cũ rồi apply lại file.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: flasky-db-migrate
  namespace: flasky
spec:
  backoffLimit: 3
  template:
    metadata:
      labels:
        app.kubernetes.io/name: flasky-db-migrate
        app.kubernetes.io/part-of: flasky
    spec:
      restartPolicy: OnFailure
      initContainers:
        - name: wait-mariadb
          image: busybox:1.36
          command: ["sh", "-c", "until nc -z mariadb 3306; do sleep 3; done"]
      containers:
        - name: migrate
          image: DOCKERHUB_USER/flasky:v4
          imagePullPolicy: IfNotPresent
          command: ["sh", "-c"]
          args: ["venv/bin/flask db upgrade"]
          env:
            - name: FLASK_APP
              value: manage.py
            - name: FLASK_CONFIG
              valueFrom:
                configMapKeyRef:
                  name: flasky-config
                  key: FLASK_CONFIG
            - name: MARIADB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MARIADB_PASSWORD
            - name: DATABASE_URL
              value: mysql+pymysql://flasky:$(MARIADB_PASSWORD)@mariadb:3306/flasky_db
```

```bash
kubectl apply -f k8s/02-migrate.yaml
kubectl -n flasky wait --for=condition=complete job/flasky-db-migrate --timeout=10m
kubectl -n flasky logs job/flasky-db-migrate
```

Nếu image private, thêm vào spec.template.spec của Job:

```yaml
imagePullSecrets:
  - name: regcred
```

### 8.3. k8s/03-app.yaml

Thay DOCKERHUB_USER/flasky:v4 bằng image vừa push. Ingress này chuyển cả đường dẫn /flasky-avatars tới MinIO để URL avatar hiện tại của ứng dụng vẫn hoạt động.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flasky
  namespace: flasky
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: flasky
  template:
    metadata:
      labels:
        app.kubernetes.io/name: flasky
        app.kubernetes.io/part-of: flasky
    spec:
      # Nếu dùng registry private, bỏ comment:
      # imagePullSecrets:
      #   - name: regcred
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  app.kubernetes.io/name: flasky
              topologyKey: kubernetes.io/hostname
      containers:
        - name: flasky
          image: DOCKERHUB_USER/flasky:v4
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 5000
          env:
            - name: FLASK_CONFIG
              valueFrom:
                configMapKeyRef:
                  name: flasky-config
                  key: FLASK_CONFIG
            - name: MINIO_BUCKET
              valueFrom:
                configMapKeyRef:
                  name: flasky-config
                  key: MINIO_BUCKET
            - name: MINIO_ENDPOINT
              valueFrom:
                configMapKeyRef:
                  name: flasky-config
                  key: MINIO_ENDPOINT
            - name: MARIADB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MARIADB_PASSWORD
            - name: REDIS_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: REDIS_PASSWORD
            - name: MINIO_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MINIO_APP_ACCESS_KEY
            - name: MINIO_SECRET_KEY
              valueFrom:
                secretKeyRef:
                  name: flasky-secrets
                  key: MINIO_APP_SECRET_KEY
            - name: DATABASE_URL
              value: mysql+pymysql://flasky:$(MARIADB_PASSWORD)@mariadb:3306/flasky_db
            - name: REDIS_URL
              value: redis://:$(REDIS_PASSWORD)@redis:6379/0
            - name: MINIO_SECURE
              value: "false"
            - name: SSL_DISABLE
              value: "true"
          startupProbe:
            httpGet:
              path: /healthz
              port: http
            failureThreshold: 30
            periodSeconds: 5
          readinessProbe:
            httpGet:
              path: /healthz
              port: http
            initialDelaySeconds: 3
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /healthz
              port: http
            initialDelaySeconds: 15
            periodSeconds: 20
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: "1"
              memory: 768Mi
---
apiVersion: v1
kind: Service
metadata:
  name: flasky
  namespace: flasky
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: flasky
  ports:
    - name: http
      port: 80
      targetPort: http
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: flasky
  namespace: flasky
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - flasky.lab.local
      secretName: flasky-tls
  rules:
    - host: flasky.lab.local
      http:
        paths:
          - path: /flasky-avatars
            pathType: Prefix
            backend:
              service:
                name: minio
                port:
                  name: api
          - path: /
            pathType: Prefix
            backend:
              service:
                name: flasky
                port:
                  name: http
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: flasky-only-required-traffic
  namespace: flasky
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: flasky
  policyTypes: ["Ingress", "Egress"]
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: ingress-nginx
          podSelector:
            matchLabels:
              app.kubernetes.io/name: ingress-nginx
      ports:
        - protocol: TCP
          port: 5000
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: flasky
          podSelector:
            matchLabels:
              app.kubernetes.io/name: mariadb
      ports:
        - protocol: TCP
          port: 3306
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: flasky
          podSelector:
            matchLabels:
              app.kubernetes.io/name: redis
      ports:
        - protocol: TCP
          port: 6379
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: flasky
          podSelector:
            matchLabels:
              app.kubernetes.io/name: minio
      ports:
        - protocol: TCP
          port: 9000
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

### 8.4. TLS cho lab

Tạo self-signed certificate trên máy quản trị, không đưa private key vào Git:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout flasky.lab.local.key \
  -out flasky.lab.local.crt \
  -subj "/CN=flasky.lab.local" \
  -addext "subjectAltName=DNS:flasky.lab.local"

kubectl -n flasky create secret tls flasky-tls \
  --cert=flasky.lab.local.crt \
  --key=flasky.lab.local.key
```

Áp dụng application:

```bash
kubectl apply -f k8s/03-app.yaml
kubectl -n flasky rollout status deployment/flasky --timeout=10m
kubectl -n flasky get all,ingress,pvc
kubectl -n flasky get pods -o wide
```

## 9. Kiểm tra end-to-end

```bash
# Từ control plane: kiểm tra Pod hai replica nằm trên hai worker khác nhau
kubectl -n flasky get pod -l app.kubernetes.io/name=flasky -o wide

# Check service nội bộ
kubectl -n flasky run curl --rm -it --restart=Never \
  --image=curlimages/curl:8.12.1 -- curl -fsS http://flasky/healthz

# Từ máy đã có DNS/hosts flasky.lab.local
curl -kI https://flasky.lab.local/healthz

# Theo dõi khi có lỗi
kubectl -n flasky get events --sort-by=.lastTimestamp
kubectl -n flasky logs deploy/flasky --tail=100
kubectl -n ingress-nginx get svc ingress-nginx-controller
```

Kết quả đúng:

- ingress-nginx-controller có EXTERNAL-IP trong pool MetalLB, thông thường là 10.0.40.60.
- Có hai Flasky Pod Running/Ready và cột NODE là k8s-w1, k8s-w2.
- MariaDB, Redis, MinIO cùng PVC Bound.
- GET <https://flasky.lab.local/healthz> trả 200. Với self-signed certificate, browser cần tin cậy CA/certificate nội bộ hoặc sẽ cảnh báo.

## 10. Kết quả

![Lab_WebSystem_K8s](./images/Lab_WebSystem_K8s_4.png)

