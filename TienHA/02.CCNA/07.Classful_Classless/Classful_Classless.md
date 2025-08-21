
# Tổng quan về Classful và Classless

Một major network là một mạng thuộc lớp A, B, C chưa bị chia nhỏ ra, ví dụ 10.0.0.0/8, 172.16.0.0/16 hay 192.168.1.0/24. Các major network sẽ được chia nhỏ ra thành các mạng subnet, ví dụ 10.128.1.0/24 là một subnet được chia ra từ major network 10.0.0.0/8.

## Cách đánh địa chỉ IP theo kiểu Classful và Classless

- Đánh địa chỉ IP theo kiểu classful là cách đặt địa chỉ sử dụng luật phân lớp A,B,C. Một địa chỉ sẽ được chia thành hai phần là network và host, đi kèm theo đó là một subnet mask để xác định phần mạng trong một địa chỉ IP.

- Cách địa chỉ IP theo kiểu classless bỏ qua luật phân lớp A,B,C. Các địa chỉ IP sẽ không được xem xét theo lớp, không sử dụng subnet mask. Kiểu đánh địa chỉ Classless xem một địa chỉ IP gồm hai phần là phần prefix và phần host. Các địa chỉ có cùng phần prefix có thể được xem như cùng một nhóm.

Ví dụ:

- Địa chỉ mạng 192.168.1.0 được biểu diễn dưới dạng Classful sẽ là 192.168.1.0 255.255.255.0 và biểu diễn dưới dạng Classless sẽ là 192.168.1.0/24.

- Khác với địa chỉ IPv4, IPv6 chỉ sử dụng cách đánh địa chỉ Classless và không sử dụng cách đánh địa chỉ Classful.

## Tra cứu bảng định tuyến theo kiểu Classful và Classless

- Classless: Khi tồn tại default route trong bảng định tuyến, nếu không có route nào cụ thể match với đích đến của gói tin, default route sẽ được sử dụng.

- Classful: Khi tồn tại default route trong bảng định tuyến, nếu không có route nào cụ thể match với đích đến của gói tin và không có bất kỳ route cụ thể nào trong bảng định tuyến cùng major network với đích đến của gói tin thì default route sẽ được sử dụng.

   Phương thức Classful được sử dụng trong mạng doanh nghiệp có IP được quy hoạch theo khối. Ví dụ, cả mạng sẽ dùng mạng 10.0.0.0/8, một chi nhánh sử dụng subnet 10.128.0.0/16 và một chi nhánh khác sẽ sử dụng subnet 10.64.0.0/16 và mỗi router của mỗi chi nhánh đều có các subnet của tất cả các nhánh khác trong bảng định tuyến của mình. Khi đó, default route sẽ có công dụng duy nhất là chỉ đường cho các gói tin đi ra khỏi mạng doanh nghiệp đó, còn việc đi đến subnet thuộc mạng đó sẽ tuân theo các route cụ thể trong bảng định tuyến của mỗi router.

## Các giao thức định tuyến thuộc trường phái Classful và các giao thức định tuyến thuộc trường phái Classless

- Các giao thức classful: không gửi kèm theo subnet mask trong các bản tin định tuyến từ đó không hỗ trợ VLSM và không hỗ trợ mạng gián đoạn. Các giao thức điển hình là RIPv1 và IGRP.

- Các giao thức classless: có gửi kèm theo subnet mask trong các bản tin định tuyến nên có hỗ trợ VLSM và hỗ trợ mạng gián đoạn. Hầu hết các giao thức hiện nay đều thuộc trường phái classless như RIPv2, OSPF và EIGRP.

## Thế nào là mạng gián đoạn sử dụng VLSM

![Sơ đồ mạng sử dụng Classful và Classless](https://waren.vn/upload/images/Classful-classless-waren-2.jpg)

Trên hình là mô hình một mạng sử dụng ba router R1, R2, R3 đại diện cho ba nhánh tương ứng.

- Chi nhánh 1 được kết nối với một mạng LAN có subnet là 172.16.1.0/24, chi nhánh 2 được kết nối với mạng LAN có subnet là 172.16.2.0/24 và chi nhánh 3 được kết nối với mạng LAN có subnet là 172.16.3.128/25.

- R1 và R2 được đấu nối với nhau bằng một kết nối serial điểm - điểm, sử dụng subnet 192.168.12.0/30. R2 và R3 được đấu nối tương tự sử dụng subnet 172.16.23.0/30.

Sơ đồ sử dụng nhiều subnet cùng một major network 172.16.0.0/16 là 172.16.2.0/24, 172.16.3.128/25 và 172.16.23.0/30. Khi trên một sơ đồ mạng tồn tại nhiều subnet của cùng một major và các subnet sử dụng nhiều prefix length khác nhau thì có thể nói sơ đồ sử dụng VLSM (Variable Length Subnet Mask).

Trên sơ đồ, có thể thấy mạng major 172.16.0.0/16 đã bị chia cắt bởi một mạng major khác đó là 192.168.12.0/24, thì có thể nói đây là sơ đồ mạng gián đoạn (discontiguous network).
