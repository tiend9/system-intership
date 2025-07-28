# 2. Giao thức VTP , STP
## 2.1 Giao thức VLAN Trunking Protocal
###  Khái niệm
VTP (Virtual Trunking Protocol) là một giao thức được sử dụng trong các mạng Ethernet để quản lý thông tin về VLAN (Virtual Local Area Network). Với VTP, các thông tin về VLAN như tên VLAN, số hiệu VLAN và các thông tin khác được đồng bộ hóa và chia sẻ tự động giữa các switch trong mạng.

Với VTP, việc thay đổi cấu hình VLAN chỉ cần thực hiện trên một switch duy nhất, và sau đó thông tin mới sẽ được gửi tới các switch khác trong cùng mạng. Điều này giúp giảm công sức và thời gian cấu hình VLAN trên các switch trong mạng lớn.

Tuy nhiên, cần lưu ý rằng VTP chỉ hoạt động trong các mạng Ethernet sử dụng giao thức IEEE 802.1Q để đánh dấu VLAN. Ngoài ra, cần cấu hình chính xác các thông số như domain name và mode trên các switch để đảm bảo tính chính xác và an toàn trong việc sử dụng VTP.  
![Alt text](../imgs/vtp-domain.png)
### Lợi ích khi sử dụng giao thức VTP 
- Cấu hình đúng các VLAN khi qua mạng , theo dõi chính xác và kiểm tra VLAN
- Hệ thống ánh xạ cho phép 1 VLAN được trunk qua các môi trường chuyền hỗn hợp 
- Báo cáo về việc thêm vào các VLAN và dễ dàng cấu hình khi thêm mới VLAN 
- Biết được VLAN và thông số cụ thể 
### Hạn chế của giao thức VTP 
- VTP không cung cấp tính tin cậy cao. Nếu một thiết bị VTP vô tình được cấu hình sai và được kết nối vào mạng, nó có thể ghi đè lên cấu hình VLAN hiện có trên các thiết bị khác trong cùng một domain VTP. Điều này có thể gây ra sự cố mạng nghiêm trọng hoặc mất dữ liệu.
- VTP không cung cấp các cơ chế bảo mật mạnh. Nó sử dụng một số phương pháp đơn giản như mật khẩu VTP để xác thực và gửi dữ liệu. Mật khẩu VTP được lưu dưới dạng plain text trong cấu hình, điều này có nghĩa là nó có thể dễ dàng bị đánh cắp và gây nguy hiểm cho mạng
- VTP không mở rộng tốt trên các mạng lớn hoặc phân tán. Khi một mạng lớn được chia thành các domain VTP nhỏ hơn, việc đồng bộ hóa thông tin VLAN giữa các domain có thể trở thành một vấn đề phức tạp và tốn nhiều thời gian.

### Các mode của VTP 
Tất cả switch muốn nhận lưu lượng với nhau thì phải cùng tên miền . Các chế độ VTP muốn tham gia vào miền quản lý , mỗi switch phải được cấu hình để hoạt động ở chế độ nhất định nào đó . Ta có 3 chế độ sau 

*Mode server*
Các switch ở mode này sẽ điều khiển việc tạo VLAN và thay đổi domain . Tất cả thông tin VTP đều được quảng bá đến các Switch trong domain . Mặc định một switch hoạt động ở mode server 
    - Tạo VLAN 
    - Chỉnh sửa VLAN 
    - Xóa VLAN
    - Gửi hoặc chuyển thông tin quảng bá
    - Đồng bộ hóa thông tin VLAN
    - Lưu cấu hình vào NVRAM

*Mode client*
Các Switch ở mode này không có quyền tạo sửa xóa bất kì VLAN nào mà chỉ nhận các quảng bá VTP từ các Switch khác và thay đổi cấu hình VLAN của nó . Đây là chế độ lắng nghe thụ động . Các thông tin được VTP chuyển tiếp ra liên kết trunk đến các Switch lân cận trong domain .
    - Chuyển thông tin quảng bá 
    - Đồng bộ hóa thông tin VLAN 
    - Không lưu cấu hình vào NVRAM

*Mode transparent*
Switch ở mode transparent không tham gia trong VTP . Ở mode này , Switch không quảng bá cấu hình VLAN của chính nó và không đồng bộ cở sở dữ liệu VLAN của nó với thông tin quảng bá nhận được mà chỉ quảng bá thông tin VTP nhận được qua cổng trunk của nó 
    - Tạo VLAN trên local 
    - Chỉnh sửa VLAN trên local 
    - Xóa VLAN trên local 
    - Chuyển thông tin quảng bá 
    - Không đồng bộ hóa thông tin trên VLAN
    - Lưu cấu hình vào NVRAM


![Alt text](../imgs/lab8.1.PNG)

## 2.2 Giao thức Spanning Tree Protocal
### Khái niệm 
Spanning tree protocal là một giao thức được sử dụng để ngăn chặn một vòng lặp . Giao thức này cho phép các Bridge truyền thông với nhau từ đó phát hiện vòng lặp vật lý trong mạng 
![Alt text](../imgs/STP.png)

### Lợi ích khi sử dụng giao thức STP
- Cung cấp liên kết dự phòng đồng thời ngăn chặn các vòng lặp không mong muốn 
- Ngăn chặn Broadcast storm
- Kết nối với các thiết bị không hỗ trợ STP , chẳng hạn như PC , máy chủ , bộ định tuyến hoặc trung tâm không được kết nối với các bộ chuyển mạch khác bằng cách sử dụng cổng cạnh 

### Các lỗi loop trong giao thức STP
***Broadcast storm***

Là hiện tượng hệ thống mạng quá tải lưu lượng phát khi cấu hình IP bị trùng hoặc không có địa chỉ IP đích .Điều này làm cho dữ liệu truyền lặp đi lặp lại , khiến hệ thống mạng ngưng hoạt động.
Giả sử PC A tiến hành gửi một broadcast frame vào hệ thống. Khi SwX nhận được frame này nó sẻ đẩy frame ra tất cả các port đến SwY. SwY nhận được broadcast frame này lại tiếp tục gửi ra tất cả các port trừ port nhận vào và quá trình frame này cứ chạy mãi một vòng giữa SwX và SwY. Các Sw cứ nhân bản và flood broadcast frame này ra. Số lượng frame này sẻ ngày càng lớn. Và khi Sw không còn khả năng xử lý nữa thì sẻ khiến Sw bị treo.  

![Alt text](../imgs/broadcaststorm.png)
 ***Trùng lặp Frame***
 PCA gửi một unicast frame đến PCB và địa chỉ MAC của B chưa được cập nhật vào bảng MAC của Sw thì Sw sẻ xử lý các frame này như một broadcast frame và flood ra tất cả các port trừ port nhận vào. Và SwX và SwY đều thực hiện chuyển flood frame này ra nhiều port khiến PCB phải xử lí frame này 2 lần.
![Alt text](../imgs/trunglapframe.png)

## Tiến trình bầu chọn và hoạt động của giao thức STP
***Chọn Root-Bridge của Giao thức Spanning Tree***
Một khi tiến trình STP được bật, các Sw sẻ gửi các gói tin BPDU (Bridge Protocol Data Unit) để trao đổi giữa các Switch với nhau, BPDU là một gói tin quan trọng trong tiến trình STP, BPDU chứa một thông tin quan trọng là Bridge-ID của các switch, giá trị này dùng để dịnh danh mỗi Switch khi nào tham gia tiến trình STP.
Bridge ID dài 8 byte,Số Priority(2byte) có giá trị từ 0 – 65535 mặc định là 32768, MAC address(6byte)
Tiến trình bầu chọn Root-Bridge sẻ tiến hành như sau:
- Đầu tiên sẻ so sánh Sw nào có số Priority thấp nhất sẻ là Root-Bridge
- Các Sw được thiết lập số Priority bằng nhau thì tiến trình thứ 2 là so sánh MAC sẻ thực hiện, Sw nào có MAC nhỏ nhất sẻ làm Root-Bridge. MAC là địa chỉ duy nhất trên thế giới nên sẻ không xảy ra trùng lập được. VD như hình trên SW1 có MAC nhỏ nhất nên sẻ được bầu chọn làm Root-Bridge.
- Sau khi đã bầu chọn được Root-Bridge thì chỉ có SW làm root mới gửi BPDU ra khỏi cổng để duy trì tiến trình STP ( gửi 2s/lần). Các SW con chỉ nhận, bổ xung thông tin BPDU và forward thông tin BPDU này.

***Bầu chọn Root Port của giao thức Spanning tree***
Sau khi bầu chọn các Root Bridge các Switch sẽ bầu chọn các Rootport là port có đường về Root-bridge có tổng cost tích lũy nhỏ nhất 
Để xác định được cost tích lũy của một port đến Switch làm Root-bridge ta thực hiện tính ngược từ Root về cổng đó theo qui tắc “vào cộng ra không cộng” dựa theo chiều lan truyền BPDU.
***Bầu chọn Disignated Port***
Tiếp theo trong tiến trình của Giao thức Spanning Tree ta thực hiện bầu chọn Designated port trên các phân đoạn mạng. Designated port là port cung cấp đường về root-bridge có tổng cost nhỏ nhất trên phân đoạn mạng đang xét. Một link kết nối chỉ có một Designated port
Các quy tắc trong bầu chọn Designated port 
- Tất cả các port của Root bridge đều là Designated port
- Trên một link p2p thì đối diện Root port là Designate port 
- Nếu trên một link có 2 cổng cung cấp đường về Root-bridge có cost tích lũy bằng nhau. Lúc đó sẻ dùng Sender ID để xác định, nếu Sender ID lại bằng nhau thì dùng đến port-ID để xét.

***Tiến trình Blocking các port còn lại của giao thức STP***
Các port không có vai trò là Root hay Designated sẻ bị Block và được gọi là Alternated port


## Quá trình tìm Block Port Spanning tree
### Root Switch
- Khi các Sw được đấu nối khởi động nó sẽ gửi gói tin BPDU(bridge protocol data unit) trên các port của Switch.
- Thông số quyết định Sw nào được làm Root Sw là Bridge-ID(8 byte) gồm có các thông số :  
  priority(của switch): dài 2 byte(9 -> 65535), default = 32768. Sw nào có chỉ số priority có chỉ số nhỏ nhất sẽ được chọn làm Root-switch 
  MAC Address Switch: dài 6 byte. Xét từ trái sang phải từng giá trị hexa thì switch nào có MAC nhỏ nhất làm Root-switch
- Khi bầu xong Root-switch thì chỉ có Root-switch được gửi BPDU(2s/1 lần). Việc gửi đó để duy trì cây spanning tree đó không bị Loop
- Theo nguyên tắc đánh số MAC của nhà sản xuất thì khi bầu chọn root-switch nó sẽ chọn switch đời đầu làm root-switch => sw cùi cắp làm lãnh đạo. Nên trong thực tế ta ko bao giờ cho bầu chọn bằng MAC mà ta chỉnh priority

### Root port
- Là port cung cấp đường về Root-switch mà có tổng path-cost là nhỏ nhất
- Khi bầu chọn Root-port thì Root-Switch ko tham gia quá trình bầu chọn này
- Mỗi Root-switch chỉ có 1 Root-port
- Path-cost là giá trị cost trên từng cổng của Switch.
![Alt text](../imgs/a1.jpg)
- Nguyên tắc tính tổng path-cost: tính từ switch đang muốn tính --> Root-switch
    Đi ra: ko cộng
    Đi vào: cộng cost
### Luật Tie-Break
- Sender Bridge ID:
    Cổng nào kết nối switch mà switch đó có bridge ID nhỏ nhất -> port đó sẽ được chọn làm Root-port.
    Bridge ID của B nhỏ hơn C à port số 2 làm Root-port
- Sender Port ID:
    - Port ID của Switch bên kia thì port nào của switch bên kia có giá trị port-ID nhỏ hơn thì chọn port bên switch mình kết nối với port ID nhỏ hơn đó.
        - Priority của port: có giá trị từ 0 -> 255, default=128. Port nào có priority nhỏ hơn thì port đó có Port ID nhỏ hơn.
        - Vị trí của port: Xét theo hạng của số thứ tự của port. Port số 1 < port 2 -> port số 2 làm root-port
![Alt text](../imgs/a2.jpg)
- Khi các luật trên không giải quyết được thì nó sẽ xét đến Port ID trên chính nó
  - Priority và vị trí của port
  - VD: Vì hub nó thực hiện flood ra tất cả các port nên frame từ port 1 của swD sẽ đi đến hub và đi cả 2 đường từ hub -> swC. Nên lúc này chúng ta không thể xác định bằng cách trên. Lúc này ta phải xet port-ID trên chính SwC
![Alt text](../imgs/a3.jpg)

### Designated port
- Tất cả các port của Root-sw đều là Designated port
- Trên 1 phân đoạn nếu port đối diện là Root-port thì mình là Designated port(ko có ý nghĩa ngược lại).
- Là port cung cấp đường về Root-sw trên phân đoạn mạng đang xét mà có tổng path-cost là nhỏ nhất.

Vd: tính path-cost trên phân đoạn ta tính từ
Root-sw(cat 2) --> cat 1 --> cat 4 = 38

![Alt text](../imgs/a4.jpg)
### Alternate port
- Khi 1 trong các phân đoạn khác bị đứt thì phân đoạn port lock sẽ được mở ra để chạy
- Khi phân đoạn trên có lại thì phân đoạn lock sẽ tiếp tục bị lock lại
- Tuy port lock không nhận được dữ liệu nhưng nó vẫn nhận gói tin BPDU từ Root-switch để duy trì cây spanning-tree. Nếu nó không nhận được gói BPDU thì nó sẽ mở port lock này ra à lúc này bị loop ráng chịu

### Peer PVST(peer Vlan Spanning tree)
PriorityVlan n = PriorityVlan n + n
- Lưu ý: Khi chỉnh sửa Priority thì số priority phải chia hết cho 4096
- Ví dụ: Vlan 1 à priority = 32768 + 1

![Alt text](../imgs/a5.jpg)

### STP timer
- Helo timer: 2(s) Thời gian gửi BPDU
- Forward timer: 15(s)
- Max-agetimes: 20(s) Nếu Root-Sw chết hay port lock không nhận được BPDU thì mất 20s nó mới hoạt động( tự mở lên hoặc bầu chọn lại Root-sw)

### STP state
- Các trạng thái khi Switch khởi động
Disable: down
Blocking: nhận BDPU, ko gửi BPDU, ko học MAC, ko forward frame
Listening: _________, gửi BPDU, ___________________
Leaning: __________________, học MAC, _______________
Forwarding: _____________________________, forward frame
- Việc chuyển từ trạng thái: Blocking à listening mất 20(s)
- Việc chuyển từ trạng thái: Listening à Leaning mất 15(s)
- Việc chuyển từ trạng thái: Leaning à Forwarding mất 15(s)
=> Vậy khi Sw khởi động xong or khi cắm dây vào port thì phải mất 30(s) đèn chuyển sang màu xanh
=> Mất 30+20+2 = 52(s) để STP port lock mới hoạt động
