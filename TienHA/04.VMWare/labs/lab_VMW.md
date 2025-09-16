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

1. Ping kiểm tra kết nối internet

```plaintext
ping google.com
```

Nếu nhận được phản hồi, máy ảo đã kết nối internet thành công.

![VMW](../.png)
