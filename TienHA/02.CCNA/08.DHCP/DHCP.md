# TÌM HIỂU DHCP

## I. DHCP LÀ GÌ ?

### 1. Khái niệm DHCP ?

![DHCP](./images/DHCP_1.png)

DHCP là viết tắt của Dynamic Host Configuration Protocol, là một giao thức mạng được sử dụng để tự động cấp phát các thông số mạng như địa chỉ IP, địa chỉ Gateway, địa chỉ DNS cho các thiết bị trên mạng.

### 2. DHCP để làm gì ?

- **Tự động gán IP** cho thiết bị khi thiết bị kết nối vào mạng.
- **Tránh xung đột IP**, vì DHCP server quản lý các địa chỉ IP đã cấp.
- **Quản lý địa chỉ IP tập trung:** Cho phép quản trị viên mạng dễ dàng theo dõi và quản lý việc sử dụng địa chỉ IP trong toàn bộ mạng.
- **Linh hoạt và dễ dàng thay đổi cấu hình:** Khi có sự thay đổi về cấu trúc mạng hoặc các thông số cấu hình, chỉ cần thực hiện thay đổi trên máy chủ DHCP, các thiết bị sẽ tự động nhận cấu hình mới khi chúng kết nối lại mạng.

### 3. DHCP gồm những thành phần nào ?

![DHCP](./images/DHCP_2.png)

**DHCP Server**:

DHCP Server là máy chủ hoặc thiết bị chịu trách nhiệm cấp phát địa chỉ IP và thông số mạng cho các DHCP Client. Nó lưu trữ một dải địa chỉ IP để phân phối, đồng thời quản lý thông tin thuê địa chỉ IP (DHCP Lease). Ngoài ra, DHCP Server cung cấp các thông tin mạng quan trọng như Subnet Mask, Gateway, và DNS Server, đảm bảo các thiết bị trong mạng có thể kết nối và giao tiếp một cách hiệu quả.

**DHCP Client**:

DHCP Client là thiết bị yêu cầu và nhận thông tin cấu hình mạng từ DHCP Server. Các thiết bị như máy tính, điện thoại, hoặc máy in đóng vai trò làm DHCP Client khi cần kết nối mạng. Chúng gửi yêu cầu để nhận địa chỉ IP và các thông số mạng như Subnet Mask, Gateway, và DNS Server. Sau khi nhận được thông tin từ DHCP Server, DHCP Client sử dụng các thông số này để kết nối và giao tiếp trong mạng.

**DHCP relay agents**:

DHCP Relay Agents là các thiết bị trung gian, đảm nhiệm vai trò chuyển tiếp gói tin DHCP giữa Client và Server khi chúng không nằm trong cùng một mạng (khác Subnet). Relay Agent nhận các gói tin từ DHCP Client trong mạng con và chuyển tiếp chúng đến DHCP Server. Sau khi Server phản hồi, Relay Agent gửi lại thông tin cho Client. Thành phần này rất hữu ích trong các hệ thống mạng lớn, nơi DHCP Server không được triển khai trong từng mạng con.

Hoạt động:

- Lắng nghe các gói tin DHCP broadcast từ client trong subnet của nó.
- Thêm thông tin về subnet của client vào gói tin.
- Chuyển tiếp các gói tin DHCP (Discover, Request) dưới dạng unicast đến DHCP server.
- Chuyển tiếp các gói tin phản hồi từ DHCP server (Offer, ACK) dưới dạng broadcast đến subnet của client.

**DHCP Lease**:

DHCP Lease là khoảng thời gian mà DHCP Server cấp phát địa chỉ IP cho một thiết bị cụ thể. Thời gian thuê này được quản lý để tối ưu hóa việc sử dụng tài nguyên địa chỉ IP trong mạng. Khi thời gian thuê sắp hết, DHCP Client có thể yêu cầu gia hạn để tiếp tục sử dụng địa chỉ IP đó. Nếu không có yêu cầu gia hạn, địa chỉ IP sẽ được giải phóng và tái sử dụng cho thiết bị khác, giúp đảm bảo việc quản lý IP hiệu quả.

**DHCP Binding**:

DHCP Binding là bản ghi lưu trữ trong DHCP Server, chứa thông tin ánh xạ giữa địa chỉ IP được cấp và địa chỉ MAC của thiết bị nhận. Đây là cơ chế giúp DHCP Server ghi lại các thông tin cấp phát, đảm bảo tính nhất quán và minh bạch trong quản lý mạng. Ngoài ra, DHCP Binding cũng hỗ trợ cấp phát địa chỉ IP cố định (Reservation) dựa trên địa chỉ MAC của thiết bị, rất hữu ích cho các thiết bị cần sử dụng một IP cụ thể, chẳng hạn như máy in hoặc máy chủ trong mạng.

### 4. Các thông điệp của giao thức DHCP

![DHCP](./images/DHCP_3.png)

**DHCP Discover**:

- **Mục đích:** Đây là thông điệp đầu tiên được gửi bởi một DHCP client khi nó khởi động hoặc kết nối vào mạng và chưa có địa chỉ IP. Mục đích là để tìm kiếm các máy chủ DHCP có sẵn trên mạng.
- **Nguồn:** DHCP client (địa chỉ IP nguồn là 0.0.0.0).
- **Đích:** Địa chỉ broadcast (255.255.255.255) hoặc địa chỉ broadcast của subnet.

**DHCP Offer**:

- **Mục đích:** Thông điệp này được gửi bởi DHCP server để phản hồi lại gói tin DHCP Discover từ client. Nó chứa một địa chỉ IP mà server sẵn sàng cấp phát, cùng với các thông số cấu hình mạng khác (subnet mask, default gateway, DNS server) và thời gian thuê (lease time).
- **Nguồn:** DHCP server (địa chỉ IP của server).
- **Đích:** Địa chỉ broadcast (255.255.255.255) hoặc địa chỉ broadcast của subnet.

**DHCP Request**:

- **Mục đích:** Thông điệp này được gửi bởi DHCP client trong hai trường hợp chính:
  - **Chấp nhận Offer:** Sau khi nhận được một hoặc nhiều gói tin DHCP Offer, client sẽ chọn một (thường là gói đầu tiên) và gửi DHCP Request để thông báo cho server đã chọn biết rằng nó chấp nhận địa chỉ IP và các thông số cấu hình đã được đề nghị.
  - **Gia hạn Lease:** Khi còn một nửa thời gian thuê, client sẽ gửi DHCP Request trực tiếp đến server đã cấp phát địa chỉ IP để yêu cầu gia hạn.
- **Nguồn:** DHCP client (địa chỉ IP nguồn có thể là 0.0.0.0 nếu là yêu cầu sau Discover, hoặc địa chỉ IP đã được cấp phát nếu là yêu cầu gia hạn).
- **Đích:** Địa chỉ broadcast (255.255.255.255) hoặc địa chỉ unicast đến DHCP server (nếu là yêu cầu gia hạn).

**DHCP Acknowledge (DHCP ACK)**:

- **Mục đích:** Thông điệp này được gửi bởi DHCP server để xác nhận rằng địa chỉ IP và các thông số cấu hình đã được cấp phát (hoặc gia hạn) thành công cho client.
- **Nguồn:** DHCP server (địa chỉ IP của server).
- **Đích:** Địa chỉ unicast đến DHCP client.

**DHCP Negative Acknowledge (DHCP NAK)**:

- **Mục đích:** Thông điệp này được gửi bởi DHCP server để thông báo cho client rằng yêu cầu của nó không được chấp nhận. Điều này có thể xảy ra nếu địa chỉ IP mà client yêu cầu không còn hợp lệ (ví dụ: đã được cấp phát cho máy khác) hoặc nếu có lỗi trong quá trình gia hạn. Client sẽ yêu cầu lại IP mới.
- **Nguồn:** DHCP server (địa chỉ IP của server).
- **Đích:** Địa chỉ unicast đến DHCP client.

**DHCP Release – Giải phóng IP**:

- **Mục đích:** Thông điệp này được gửi bởi DHCP client để thông báo cho server biết rằng nó không còn sử dụng địa chỉ IP đã được cấp phát nữa. Điều này thường xảy ra khi client tắt máy hoặc ngắt kết nối mạng.
- **Nguồn:** DHCP client (địa chỉ IP đã được cấp phát).
- **Đích:** Địa chỉ unicast đến DHCP server.

**DHCP Inform**:

- **Mục đích:** Thông điệp này được gửi bởi DHCP client để yêu cầu thêm các thông số cấu hình mạng từ server mà không cần cấp phát địa chỉ IP mới. Client có thể đã được cấu hình địa chỉ IP tĩnh hoặc nhận địa chỉ IP theo cách khác.
- **Nguồn:** DHCP client (địa chỉ IP đã được cấu hình).
- **Đích:** Địa chỉ unicast đến DHCP server.

### 5. Sơ đồ hoạt động DHCP

Sơ đồ nguyên lí hoạt động của DHCP: Quy trình **DORA**

![DHCP](./images/DHCP_4.png)

**Bước 1 - Discovery (Tìm kiếm máy chủ DHCP)**:

Khi một thiết bị (Client) kết nối vào mạng lần đầu, nó không có địa chỉ IP. Client sẽ gửi một gói tin DHCPDISCOVER dưới dạng broadcast (phát tán toàn mạng) để tìm kiếm DHCP Server. Gói tin này chứa:

- **Địa chỉ nguồn (Source Address)** là `0.0.0.0`.
- **Địa chỉ đích (Destination Address)** là `255.255.255.255`.

Mục đích là để thông báo cho tất cả các thiết bị trong mạng rằng nó cần một địa chỉ IP.

**Bước 2 - Offer (Đề nghị cấp phát địa chỉ IP)**:

Khi DHCP Server nhận được gói DHCPDISCOVER, nó sẽ phản hồi bằng một gói tin DHCPOFFER. Gói tin DHCPOFFER được gửi đến địa chỉ MAC của Client. Gói tin này bao gồm:

- Địa chỉ IP tạm thời được đề nghị cho Client.
- Thông tin mạng như Subnet Mask, Default Gateway, DNS Server, và thời gian thuê địa chỉ IP (Lease Time).

**Bước 3 - Request (Yêu cầu sử dụng địa chỉ IP)**:

Sau khi nhận được gói tin DHCPOFFER, Client sẽ gửi một gói tin DHCPREQUEST để chấp nhận địa chỉ IP mà DHCP Server đề nghị. Gói tin này xác nhận rằng Client muốn sử dụng địa chỉ IP được cấp. Đồng thời, Client cũng gửi yêu cầu xác nhận các thông tin khác như Subnet Mask, Gateway, DNS.

**Bước 4 - Acknowledgement (Xác nhận)**:

DHCP Server gửi gói tin DHCPACK để xác nhận rằng địa chỉ IP đã được cấp phát thành công cho Client. Gói tin này cũng bao gồm thời gian thuê địa chỉ IP (Lease Time) và các thông tin cấu hình mạng. Sau khi nhận được DHCPACK, Client cấu hình địa chỉ IP trên giao diện mạng của nó và bắt đầu sử dụng.

## II. ĐẶC ĐIỂM DHCP

### 1. Ưu Điểm

![DHCP](./images/DHCP_5.png)

- **Cấu hình đáng tin cậy:** Cấu hình địa chỉ IP theo cách thủ công có thể dẫn đến sai sót.

- **Ít công việc hơn cho quản trị viên mạng:** Quản trị viên mạng sẽ mất rất nhiều thời gian và tài nguyên để định cấu hình địa chỉ IP theo cách thủ công trong những mạng lớn. DHCP sẽ giúp mọi thứ hoạt động hiệu quả hơn.

- **Sửa đổi trong thời gian thực:** Quản trị viên có thể thực hiện những thay đổi đối với tùy chọn DHCP trong mạng ngay cả khi máy chủ DHCP đang chạy và cấp phát địa chỉ IP.

- **Miễn phí:** Đối với hầu hết hệ thống mạng, việc triển khai DHCP là hoàn toàn miễn phí.

- **Hỗ trợ nhiều thiết bị trên một mạng:** DHCP cho phép bạn kết nối và lướt web trên bất kỳ thiết bị nào bạn chọn mà vẫn có trải nghiệm liền mạch.

### 2. Nhược điểm

![DHCP](./images/DHCP_6.png)

- **Bảo mật:** Máy chủ DHCP không có cách nào để xác thực những máy khách yêu cầu địa chỉ IP. Vì vậy, khách hàng có thể truy cập vào những địa chỉ IP trái phép bằng cách giả vờ là một khách hàng khác.

- **Ảnh hưởng đến máy khách khi gặp lỗi:** Nếu một mạng chỉ có một máy chủ DHCP và nó bị lỗi, máy khách sẽ không thể truy cập vào web.

- **Cần tác nhân chuyển tiếp cần thiết:** Máy chủ DHCP phải có tác nhân chuyển tiếp để có thể giao tiếp với mạng vì dữ liệu DHCP không thể truyền qua bộ định tuyến.

- **Không có IP tĩnh:** Không thể sử dụng những máy tính được kết nối với mạng có triển khai DHCP làm máy chủ vì địa chỉ IP của chúng luôn thay đổi.

- **Theo dõi hoạt động trên Internet:** Việc theo dõi hoạt động trên Internet sẽ trở nên khó khăn hơn với DHCP vì cùng một thiết bị có thể có hai địa chỉ IP trở lên trong một khoảng thời gian nhất định.
