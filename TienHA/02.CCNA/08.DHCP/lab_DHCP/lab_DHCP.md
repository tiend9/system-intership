# LAB DHCP

Trước tiên, để có thể thực hành lab DHCP trên mô hình Client - Sever thì thay vì phải dùng nhiều máy tính hoặc thiết bị mạng để cấu hình thì ta có thể tạo ra 2 Host ảo trên cùng 1 máy tính xách tay của mình và thức hiện cấu hình DHCP trên chùng.

Và để có thể tạo máy ảo trước hết ta cần phải cài đặt hệ diều hành Linux (đây là hệ điều hành rất hữu ích vì nó tích hợp sẵn các công nghệ nhúng cho pháp can thiệp vào hệ thống sever hay dữ liệu )

Để có thể cài đặt Linux trên Máy tính Win ta có 3 cách phổ biến:

1. Dual Boot (cài song song Windows + Linux)
2. Dùng máy ảo (VMware, VirtualBox, Hyper-V)
3. WSL (Windows Subsystem for Linux - Dùng code Linux cơ bản-Không có GUI Linux)

Và trong bài Lab này thì ta chỉ sử dụng cách 2 do chỉ cần CLI để cấu hình thui.(Cách cài tra ChatGPT)

## Vai trò

- **Ubuntu:** DHCP server
- **CentOS:** DHCP client