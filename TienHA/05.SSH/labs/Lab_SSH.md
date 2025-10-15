# Lab SSH

## Từ máy chủ (windows) đến webserver

### 1. CentOS Stream 9

`Bước 1`: Kiểm tra IP trên máy ảo

- Nhập lệnh:

```ruby
ip a
```

- Kết quả:

![SSH](../labs/lab_images/SSH_37.png)

- IP của máy ảo là `192.168.157.129`.

`Bước 2`: Kiểm tra SSH Server trên máy ảo

- Trên máy ảo CentOS stream 9, kiểm tra dịch vụ OpenSSH có đang chạy không:

```ruby
sudo systemctl status sshd
```

- Nếu thấy **active (running)** → SSH Server đang chạy:

![SSH](../labs/lab_images/SSH_38.png)

- Nếu không, khởi động với lệnh:

```ruby
sudo systemctl start sshd
```

- Nếu muốn SSH Server tự động chạy mỗi khi khởi động máy ảo:

```ruby
sudo systemctl enable sshd
```

`Bước 3:` Cấu hình tường lửa trên máy ảo

- CentOS dùng firewall, kiểm tra cổng SSH đã mở chưa (mặc định là 22):

```ruby
sudo firewall-cmd --list-all
```

- Nếu có dòng `ssh` trong danh sách **services** là OK.

![SSH](../labs/lab_images/SSH_39.png)

- Trường hợp cổng chưa mở, thực hiện các lệnh sau:

```ruby
# Mở cổng SSH
sudo firewall-cmd --permanent --add-service=ssh

# reload lại firewall
sudo firewall-cmd --reload

# Thực hiện kiểm tra lại danh sách
sudo firewall-cmd --list-all
```

`Bước 4`: SSH từ máy host (windows) vào máy ảo CentOS Stream 9

- Mở terminal trên máy host và chạy lệnh:

    ```ruby
    ssh root@192.168.157.129
    ```

- `root`: Tài khoản trên máy ảo.
- `192.168.157.129`: IP của máy ảo đã kiểm tra ở `bước 1`.

- Kết quả sau kết nối:

![SSH](../labs/lab_images/SSH_40.png)

### 2. Ubuntu Server

`Bước 1`: Kiểm tra IP máy ảo ubuntu

- Sử dụng lệnh:

```ruby
ip a
```

![SSH](../labs/lab_images/SSH_41.png)

`Bước 2`: Kiểm tra và cài đặt SSH Server

```ruby
# Kiểm tra SSH đã cài trên máy chưa
sudo systemctl status ssh

# Nếu chưa, cài đặt SSH Server
sudo apt install openssh-server -y

# Khởi động SSH Server
sudo systemctl start ssh

# Để SSH tự động chạy sau mỗi lần bật máy
sudo systemctl enable ssh
```

- Sau đó kiểm tra lại trạng thái SSH, nếu thấy active (running) → SSH Server đang chạy.

![SSH](../labs/lab_images/SSH_42.png)

`Bước 3`: Mở cổng tường lửa (nếu có) - ufw (ubuntu firewall)

- Nhập lệnh sau:

```ruby
# Cho phép ssh trên tường lửa
sudo ufw allow ssh

# Cho phép mỗi khi khởi động
sudo ufw enable

# Kiểm tra lại trạng thái
sudo ufw status
```

- Nếu thấy cổng 22 -> OK.

![SSH](../labs/lab_images/SSH_43.png)

`Bước 4`: SSH từ windows vào ubuntu

- Mở terminal trên máy host và chạy lệnh:

    ```ruby
    ssh tien9a@192.168.157.128
    ```

- `tien9a`: Tài khoản trên máy ảo.
- `192.168.57.128`: IP của máy ảo đã kiểm tra ở `bước 1`.

- Kết quả sau kết nối:

![SSH](../labs/lab_images/SSH_44.png)

## Từ CentOS Stream 9 đến Ubuntu Server

1. Tương tự như các bước ở trên, kiểm tra IP, OpenSSH, tường lửa trên ubuntu.
2. Thực hiện lệnh ssh trên CentOS Stream 9 với lệnh:

```ruby
ssh tien9a@192.168.157.128
```

- Kết quả thành công:

![SSH](../labs/lab_images/SSH_45.png)
