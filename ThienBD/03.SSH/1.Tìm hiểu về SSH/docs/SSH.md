# 1.SSH là gì 
## 1.1 Khái niệm
SSH là một giao thức đăng nhập vào hệ thống server từ xa, sử dụng cơ chế mã hóa đủ mạnh nhằm ngăn chặn các hiện tượng nghe trộm, đánh cắp thông tin trên đường truyền. SSH cho người dùng khả năng kiểm soát, quản trị dữ liệu của server thông qua internet. SSH chính là từ viết tắt của cụm từ Secure Socket Shell. SSH giúp cho việc kết nối của mạng lưới máy chủ với máy khách được đảm bảo an toàn và bảo mật tốt.
![Alt text](../imgs/1.jpg)
- SSH hoạt động ở lớp trên trong mô hình phân lớp TCP/IP
- Các công cụ SSH (như là OpenSSH, PuTTy,…) cung cấp cho người dùng cách thức để thiết lập kết nối mạng được mã hoá để tạo 1 kênh kết nối riêng tư.
- Hơn nữa tính năng tunneling (hoặc còn gọi là port forwarding) của các công cụ này cho phép chuyển tải các giao vận theo các giao thức khác.
- Mỗi khi dữ liệu được gửi bởi 1 máy tính vào mạng, SSH tự động mã hoá nó. Khi dữ liệu được nhận vào, SSH tự động giải mã nó

Có một vài cách để SSH tới SSH server như dùng password hoặc keypair. Trong đó phương thức dùng keypair được cho là có tính bảo mật cao hơn bởi nếu trong quá trình sử dụng mà các gói tin của bạn bị bắt lại, các phiên trao đổi khóa giữa SSH server và Client sẽ bị lộ và attacker có thể dùng nó để giải mã dữ liệu. Hơn nữa, việc này cũng tạo điều kiện cho các cuộc tấn công Brute Force mật khẩu.

SSH có hỗ trợ sử dụng cặp khóa Private Key và Public Key được chia sẻ với nhau từ trước. Nghĩa là bạn đã có sẵn Private Key để trao đổi với server mà không cần đến quá trình trao đổi khóa, điều này sẽ hạn chế khả năng bị bắt gói. Hơn nữa cặp khóa này còn có một mật khẩu riêng của nó, gọi là passphrase (hay keyphrase). Mật khẩu này được dùng để mở khóa Private Key (được hỏi khi bạn SSH vào server) và tạo lớp xác thực thứ 2 cho bộ khóa. Nghĩa là:

- Nếu attacker không có Private Key, việc truy cập vào server gần như không thể, chỉ cần bạn giữ kĩ Private Key.
- Tuy nhiên trong trường hợp Private Key bị lộ, bạn vẫn khá an toàn vì đối phương không có passphrase thì vẫn chưa thể làm được gì, tuy nhiên đó chỉ là tạm thời. Bạn cần truy cập server thông qua cách trực tiếp hoặc qua VNC của nhà cung cấp nếu đó là một VPS để thay đổi lại bộ khóa.

## 1.2 Cơ chế hoạt động 
Một phiên làm việc của SSH đều phải trải qua 4 bước:

- Thiết lập kết nối ban đầu (SSH-TRANS)
- Tiến hành xác thực (SSH-AUTH)
- Mở phiên kết nối để thực hiện các dịch vụ (SSH-CONN)
- Chạy các ứng dụng SSH (Có thể là SSH-SFTP, SCP)


SSH-TRANS: là khối xây dựng cơ bản cung cấp kết nối ban đầu, ghi chép giao thức, xác thực server, mã hóa cơ bản và bảo toàn dữ liệu. Sau khi thiết lập kết nối, client có một kết nối độc lập và bảo mật

Sau đó, client dùng SSH-AUTH để xác thực đến server. SSH-AUTH yêu cầu một phương thức: Public key với thuật toán DSA. Ngoài ra, sử dụng mật khẩu và hostbased

Sau khi xác thực, SSH client yêu cầu SSH-CONN để cung cấp một kênh riêng biệt qua SSH-TRANS

Ngoài ra, còn cung cấp các dịch vụ như Remote Login and Command Execution, agent fowarding, files transfer, TCP port fowarding, X fowarding,...

Cuối cùng, một ứng dụng có thể sử dụng SSH-SFTP hoặc SCP truyền file hoặc thao tác remote từ xa
## 1.3 Chức năng của SSH
SSH có thể được sử dụng cho nhiều mục đích khác nhau, bao gồm:

- Quản trị máy tính từ xa: SSH cho phép quản trị viên hệ thống quản lý máy tính từ xa một cách an toàn.
- Truyền tệp: SSH có thể được sử dụng để truyền tệp an toàn giữa hai máy tính.
- Truy cập vào máy chủ từ xa: SSH có thể được sử dụng để truy cập vào máy chủ từ xa, chẳng hạn như máy chủ web hoặc máy chủ ứng dụng.
- Xác thực: SSH sử dụng xác thực hai yếu tố để xác thực người dùng. Điều này giúp ngăn chặn truy cập trái phép vào máy tính từ xa.
- Mã hóa: SSH sử dụng thuật toán mã hóa để bảo vệ dữ liệu trao đổi giữa hai máy tính. Điều này giúp ngăn chặn kẻ tấn công nghe trộm hoặc thay đổi dữ liệu.
- Khả năng kiểm soát truy cập: SSH cung cấp các tính năng kiểm soát truy cập để ngăn chặn truy cập trái phép vào máy tính từ xa.

## 1.4.Các thành phần trong SSH
- Server : Một chương trình cho phép đi vào kết nối SSH với một bộ máy, trình bày xác thực, cấp phép, … Trong hầu hết SSH bổ sung của Unix thì server thường là sshd.
- Client : Một chương trình kết nối đến SSH server và đưa ra yêu cầu như là “log me in” hoặc “copy this file”. Trong SSH1, SSH2 và OpenSSH, client chủ yếu là ssh và scp.
- Session : Một phiên kết nối giữa một client và một server. Nó bắt đầu sau khi client xác thực thành công đến một server và kết thúc khi kết nối chấm dứt. Session có thể được tương tác với nhau hoặc có thể là một chuyến riêng.

![Alt text](../imgs/33.png)
# 2. SSH Key
## 2.1 Khái niệm
Là chuỗi kí tự sử dụng để xác thực.

Có 2 loại khóa : khóa đối xứng(khóa bí mật) và khóa bất đối xứng(khóa công khai). Một khóa bất đối xứng (khóa công khai) :có 2 thành phần là phần công khai và phần bí mật.

## 2.2. Các thành phần của SSH Key 

Khi tạo ra một SSH Key, bạn cần biết sẽ có 3 thành phần quan trọng như sau:

- Public Key (dạng file và string) – Bạn sẽ copy ký tự key này sẽ bỏ vào file ~/.ssh/authorized_keys trên server của bạn.
- Private Key (dạng file và string) – Bạn sẽ lưu file này vào máy tính, sau đó sẽ thiết lập cho PuTTY, WinSCP, MobaXterm,..để có thể login.
- Keypharse (dạng string, cần ghi nhớ) – Mật khẩu để mở private key, khi đăng nhập vào server nó sẽ hỏi cái này.

## 2.3.3. Cách thức hoạt động của SSH Keys
SSH key pairs là một cặp khóa được mã hóa có thể được dùng để xác thực giữa client và server. Mỗi một cặp khóa sẽ có public key và private key. Private key được giữ ở phía client và phải được bảo mật tuyệt đối. Nếu có được private key, attackers hoàn toàn có thể truy cập vào server. Cũng vì thế nó được mã hóa với passphrase.

Public key có thể được chia sẻ và phân tán rộng rãi bởi nó được dùng để mã hóa các tin nhắn mà chỉ private key mới giải mã được. Public key sẽ được upload lên phía server và được lưu tại thư mục người dùng (~/.ssh/authorized_keys).

Khi có client muốn xác thực bằng SSH keys, server có thể test xem client đó có giữ private key hay không. Nếu client chứng minh được nó có private key thì kết nối có thể được thiết lập.


# 3.Một số thuật toán sử dụng trong SSH

## 3.1. Thuật toán Public key (khóa công khai)
- RSA (Digital Signature Algorithm): là thuật toán mã hóa bất đối xứng, dùng cho mã hóa và chữ ký
- DSA: dùng chữ ký số
- Thuật toán thỏa thuận Diffie-Hellman: cho phép 2 bên lấy được khóa được chia sẻ trên một kênh mở
# 3.2. Thuật toán Private key (khóa bí mật)
- AES(Advanced Encryption Standard): là một thuật toán mã hóa khối, chiều dài có thể là 128 đến 256bit.
- DES(Data Encryption Standard): là một thuật toán mã hóa bảo mật
- 3DES: cải tiến của DES, tăng độ dài của khóa để đạt độ bảo mật cao hơn
- RC4: Kiểu mã hóa nhanh, nhưng kém bảo mật
- Blowfish: là một thuật toán mã hóa miễn phí, có tốc độ mã hóa nhanh hơn DES, nhưng chậm hơn RC4. Độ dài của key từ 32 đến 448bit.
# 3.3. Hàm băm (HASH)
- CRS-32: Băm dữ liệu nhưng không mã hóa. Chỉ sử dụng để kiểm tra tính toàn vẹn của gói tin, tránh thay đổi thông tin trên đường truyền
- MD5: Hàm băm có độ an toàn cao vì được mã hóa dữ liệu, với chiều dài là 128bit.
- SHA-1: Một cải tiến của MD5, với chiều dài là 160bit






*Tài liệu tham khảo*

[1] [https://fptcloud.com/ssh-la-gi/](https://fptcloud.com/ssh-la-gi/)
[2] [https://vi.wikipedia.org/wiki/SSH](https://vi.wikipedia.org/wiki/SSH)
[3] [https://blog.cloud365.vn/other/su-dung-SSH-KEY/](https://blog.cloud365.vn/other/su-dung-SSH-KEY/)