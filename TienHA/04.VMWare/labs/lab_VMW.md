# LAB

## I. Sử dụng chế độ mạng NAT để truy cập Internet

### `Bước 1:` Cấu hình NAT trên VMware Workstation

1. Tắt máy ảo CentOS 9 nếu đang chạy.
2. Chọn máy ảo CentOS 9 trong VMware Workstation.
3. Nhấp vào Edit virtual machine settings.
![VMW](../labs/lab_images/VMWARElab_1.png)
4. Chọn Network Adapter → Chọn NAT (Share the host’s IP address).
![VMW](../labs/lab_images/VMWARElab_2.png)
5. Nhấn OK để lưu lại cấu hình.

### `Bước 2:` Kiểm tra và kết nối internet

1. Ping kiểm tra kết nối internet(ping 4 gói gửi đi)

```ruby
sudo ping -c 4 google.com
```

Nếu nhận được phản hồi, máy ảo đã kết nối internet thành công.

![VMW](../labs/lab_images/VMWARElab_3.png)

## II. Sử dụng chế độ card Host-only để 2 máy ảo kết nối với nhau

### `Bước 1:` Cấu hình card mạng Host-Only trên VMware

1. Chọn CentOS 9 → Edit virtual machine settings → Chọn Network Adapter → Tích chọn Host-Only → OK để lưu cấu hình.
2. Lặp lại các bước trên Unbuntu Server.

![VMW](../labs/lab_images/VMWARElab_4.png)

Kiểm tra cấu hình card mạng Host-Only:

- Vào Edit → Virtual Network Editor.
- Đảm bảo có một VMnet1 đã được thiết lập cho Host-Only.

![VMW](../labs/lab_images/VMWARElab_5.png)

Nếu chưa có:

- Chọn Add Network → VMnet1 → Host-Only.
- Đặt dải IP (ví dụ: 192.168.186.0/24).

### `Bước 2:` Cấu hình/ Check IP tĩnh cho mỗi máy ảo

Ta dùng lệnh `ip a` ra ip tĩnh cho cả 2 máy ảo Ubuntu(192.168.60.76) và Centos(192.168.157.168).Lúc này ta chỉ cần chỉnh sửa sao cho 2 máy trùng dải mạng là `192.168.60.X` và ta sẽ chỉnh ở con CentOS.

**Trên máy ảo CentOS 9:**

- Chỉnh sửa file cấu hình card mạng:

```plaintext
sudo nano /etc/sysconfig/network-scripts/ifcfg-ens160
```

Cập nhật thêm thông tin nội dung như sau:

![confiure ens-33](../images/conf_ifcfg-ens33.png)

Lưu và khởi động lại card mạng:

```plaintext
sudo systemctl restart network
```

kiểm tra lại địa chỉ IP:

```plaintext
ip addr
```

![check address](../images/screenshot_4.png)

**Kiểm tra kết nối giữa 2 máy ảo:**

Từ CentOS 9 ping tới Ubuntu Server:

```plaintext
ping 192.168.186.100
```

kết quả ping thành công:

![ping to ubuntu](../images/ping_to_ubuntu.png)

Từ Ubuntu Server ping tới CentOS 9:

```plaintext
ping 192.168.186.10
```

Kết quả ping thành công:

![ping to centos](../images/ping_to_centos.png)
