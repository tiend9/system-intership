# VPN

## 1.VPN là gì ?

VPN (Virtual Private Network) hay còn gọi là mạng riêng ảo, là một công nghệ tạo ra một kết nối mạng riêng tư và an toàn giữa các thiết bị thông qua một mạng công cộng như Internet. VPN hoạt động bằng cách ẩn địa chỉ IP thực của người dùng và mã hóa toàn bộ dữ liệu được truyền tải.

## 2. Cách thức hoạt động VPN

1. Khởi tạo kết nối: Khi bạn bật VPN, thiết bị của bạn (Client) sẽ thiết lập một kết nối được mã hóa đến một máy chủ VPN do nhà cung cấp VPN vận hành.

2. Mã hóa dữ liệu: Mọi dữ liệu đi ra từ thiết bị của bạn, ví dụ như yêu cầu truy cập website, sẽ được mã hóa (biến đổi thành dạng không thể đọc được nếu không có khóa giải mã) trước khi rời khỏi máy.

3. Thiết lập kênh truyền an toàn: Dữ liệu mã hóa sau đó được đóng gói và truyền đi thông qua một kênh bảo mật, sử dụng giao thức đường hầm, qua mạng Internet đến máy chủ VPN. Kênh truyền này bảo vệ dữ liệu khỏi các mối đe dọa từ bên ngoài.

4. Giải mã tại máy chủ VPN: Máy chủ VPN nhận dữ liệu mã hóa từ bạn, sau đó giải mã.

5. Truy cập Internet ẩn danh: Máy chủ VPN sẽ thay mặt bạn gửi yêu cầu (đã được giải mã) đến website hoặc dịch vụ trực tuyến bạn muốn. Yêu cầu này sẽ mang địa chỉ IP của máy chủ VPN, không phải địa chỉ IP thực của bạn, giúp ẩn địa chỉ IP của bạn.

6. Mã hóa và gửi lại phản hồi: Khi nhận được phản hồi từ internet, máy chủ VPN sẽ mã hóa lại dữ liệu này và gửi về thiết bị của bạn thông qua kênh truyền an toàn đã thiết lập.

7. Giải mã tại thiết bị: Thiết bị của bạn nhận dữ liệu mã hóa, giải mã và hiển thị cho bạn.

## 3. Các giao thức VPN phổ biến

### 3.1. Giao thức PPTP (Point-to-Point Tunneling Protocol)

Giao thức PPTP là giao thức được phát triển từ Microsoft. PPTP cũng là giao thức phổ biến và đơn giản nhất cho phép cài đặt dễ dàng. Tuy nhiên, PPTP không có tính bảo mật cao, do đó rất dễ bị tấn công.

### 3.2. Giao thức L2TP (Layer 2 Tunneling Protocol)

Giao thức L2TP đem đến môi trường kết nối an toàn và có nhiệm vụ hỗ trợ mã hóa dữ liệu. L2TP thường kết hợp với giao thức mã hóa IPSec (Internet Protocol Security) để tăng cường tính bảo mật.

### 3.3 Giao thức IPSec

Giao thức IPSec (Internet Protocol Security) là một bộ giao thức bảo mật được thiết kế để bảo vệ lưu lượng dữ liệu trên mạng Internet Protocol (IP). IPSec cung cấp hai dịch vụ bảo mật chính

- Mã hóa: IPSec mã hóa dữ liệu để chỉ những đối tượng được ủy quyền mới có thể đọc được
- Xác thực: IPSec xác thực nguồn và đích của dữ liệu để ngăn chặn việc giả mạo.

### 3.4 Giao thức OpenVPN

Giao thức OpenVPN là mã nguồn mở phổ biến và có thể hỗ trợ trên nhiều nền tảng. Giao thức này thường sử dụng SSL/TLS để thiết lập, kết nối và mã hóa dữ liệu. OpenVPN có tính linh hoạt và khả năng tùy chỉnh, vì vậy nó cũng được xem là giao thức phổ biến để triển khai mạng riêng ảo.

### 3.5. Giao thức SSTP (Secure Socket Tunneling Protocol)

Giao thức SSTP là giao thức được tạo ra bởi Microsoft, do đó nó chỉ hoạt động tốt trên hệ điều hành Windows. SSTP thường sử dụng SSL/TLS để mã hóa dữ liệu và cung cấp kết nối an toàn. Giao thức SSTP hoạt động trên cổng TCP 443 nên có thể vượt qua các rào cản và hạn chế trên không gian mạng.

## 4. Các giải pháp sử dụng VPN dành cho doanh nghiệp

### 4.1 VPN client to site

VPN client là gì? Đây là loại mạng riêng ảo giúp các bạn kết nối đến mạng riêng và ở xa, thông qua một VPN server. Thông thường, để sử dụng VPN client to site, thiết bị của bạn phải cài một phần mềm VPN client. Nếu không, máy của bạn sẽ không kết nối được tới VPN server. 

Khi khởi động VPN client trên máy tính cá nhân, bạn cần sử dụng tên người dùng và mật khẩu để xác thực. Sau đó, bạn sẽ tiếp tục khởi tạo đường truyền VPN đã được mã hóa từ máy cá nhân của bạn đến mạng riêng ở xa. 

Tiếp theo, dữ liệu từ máy của bạn đến mạng ở xa sẽ được truyền trên đường truyền riêng biệt, an toàn và bảo mật. Một trong các VPN client to site thịnh hành nhất hiện nay là OpenVPN.

VPN client to site thường được sử dụng để: 

- Truy cập vào các mạng riêng ở xa. Ví dụ, các bạn có thể truy cập vào máy chủ chỉ có mạng LAN trong trung tâm dữ liệu.
- Truy cập mạng an toàn và bảo mật thông qua việc kết nối qua mạng riêng ảo.
- Ẩn địa chỉ IP thật của thiết bị cá nhân trên không gian mạng

### 4.2 VPN site to site

Nếu bạn có một mạng ở văn phòng Hà Nội, một mạng ở văn phòng Tp.HCM và hai mạng riêng này cần kết nối cùng nhau để chia sẻ hoặc trao đổi dữ liệu thì nên sử dụng VPN site to site.

VPN site to site là một dạng mạng riêng ảo kết nối 2 hay nhiều mạng riêng với nhau thông qua đường truyền an toàn và bảo mật. VPN site to site giúp mở rộng mạng của doanh nghiệp, giúp cho những chi nhánh ở các nơi khác nhau có thể truy cập đến các ứng dụng hay tài nguyên dùng chung. Một trong các VPN site to site thịnh hành nhất hiện nay là IPsec.

VPN site to site giúp giải quyết nhiều vấn đề cho doanh nghiệp như:

- Kết nối mạng giữa các trung tâm dữ liệu với nhau
- Kết nối giữa trung tâm dữ liệu với VPS trên cloud
- Kết nối mạng giữa trụ sở với các chi nhánh văn phòng
