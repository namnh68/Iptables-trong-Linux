Iptables-trong-Linux
====================
Xin chào các bạn.Tôi viết bài viết này để chia sẻ những thứ hay ho trong quá trình tìm hiểu về iptabls, và một số lưu ý trong quá trình tìm hiểu về nó:
Nội dung bài viết:

1. Các kiến thức cần phải có khi học về iptables
2. Phân mục rõ ràng các bảng, chain, rule trong iptables
3. Quá trình xử lý gói tin trong iptables
4. Một số tùy chọn và target thường sử dụng
5. Lời kết

=====

## 1. Các kiến thức cân phải có:

iptables là gì : iptables là một gói phần mềm để tạo tường lửa cho máy linux của bạn nó có các chức năng lọc gói tin, nat gói tin qua đó để giúp làm nhiệm vụ bảo mật thông tin cá nhân tránh mất mát thông tin và áp dụng nhưng chính sách đổi với người sử dụng. Vậy nat, lọc gói tin như thế nào:

1. NAT (network address translation): nat là quá trình chuyển đối các port nguồn, đích, địa chỉ nguồn và địa chỉ đích của một gói tin
VD: 1 gói tin có địa chỉ đích là 192.168.1.2:22 và nguồn là 10.0.30.100:1922. Lúc này nat có thể đổi được địa chỉ nguồn và đích của gói tin.
Tác dụng của NAT: trong mang internet thực tế nat dùng để chuyển đổi địa chỉ prive thành địa chỉ publish để có thể đi ra được mạng internet vì trong mạng internet dung địa chỉ publish. Nó cũng có thể để giấu đi địa chỉ thật của một hệ thông server qua đó giảm thiểu được các cuộc tấn công mạng nhằm vào các hệ thống server.Một số hình ảnh của quá trình nat
<img class="image__pic js-image-pic" src="http://i.imgur.com/icxlmIY.png" alt="" id="screenshot-image">
*Đến đây bạn đã hiểu nôm na nat là gì chưa???:D*

2. Filter gói tin: là quá trình bắt gói tin theo một số yêu cầu

Quá trình bắt gói tin được diễn ra như sau: Giả sử khi một gói tin đi đến, nó có các bộ tham số như: địa chỉ nguồn, địa chỉ đích, port nguồn đích, TTL,TOS giao thức. Lúc này bộ lọc sẽ dự vào các tham số trên của gói tin để bắt gói tin đó rồi áp dụng phương pháp xử với nó.

VD: bộ lọc lọc các gói tin có kiểu tcp thì tất cả các gói tin có giao thức tcp sẽ được iptables giữ lại và áp dụng các phương pháp xử lý, ngoài ra còn có thể kết hợp các điều kiện lại với nhau như gói tin dang tcp có địa chỉ port đích, nguồn,...
```
root@centos:#iptables -A INPUT -p tcp -dport 80 -j ACCEPT
```
Ví dụ trên cho biết diễn sự kết hợp giữa các điều kiện lại với nhau: "là gói tin tcp và địa chỉ port đích là 80"

3. Gói tin: 
Các bạn phải hiểu rõ ràng về gói tin. Ví dụ như gối với gói tin tcp thì phần tiêu đề của nó có các trường nào, ý nghĩa của các trường đó, và hiểu về port.


## 2. Phân mục rõ ràng các khái niệm bảng, chain, rule:

Mới đầu tiên khi tìm hiểu về iptables tôi rất loạn các khái niệm này với nhau. chúng ta cần phân biệt rõ ràng các khái niệm này trước khi tìm hiểu sâu hơn về iptables

Đầu tiên ta phải định nghĩa rõ ràng. Trong iptable được chia thành 3 **bảng (table)**
* Bảng mangle: bảng này để thay đổi QoS của gói tin như thay đôi các tham số TOS,TTL,..
* Bảng NAT: dùng để thay đổi địa chỉ đích,nguồn, port đích, nguồn của gói tin
* Bảng filter: áp dụng các chính sách lọc gói tin qua đó cho phép gói tin được qua hay không qua

*Bình thường bảng nat và filter rất hay được sử dụng còn bảng mangle trong mạng soho thường ít sử dụng đến nó*

##### a. Bảng NAT
Trong bảng NAT có các **chain** và có 3 chain được xây dựng sẵn trong table NAT:
- **Chain** PREROUTING : đấy là chain dùng để thay đổi địa chỉ đích của gói tin và trong chain FREROUTING target(tác vụ) được sử dụng là DNAT (destination NAT):
VD:
```
# iptables -t NAT -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.0.30.100:80
```
Câu lệnh này có ý nghĩa. Đối với gói tin tcp có port đích là 80 thì sẽ được đổi địa chỉ đích thành 10.0.30.100
ví dụ trên đã định nghĩa rõ cho chúng ta:nói ra chọn bảng NAT áp dụng rule và trong chain PREROUTING sử dụng target DNAT.

- **Chain** POSROUTING : đây là chain dùng để thay đổi địa chỉ nguồn của gói tin và targer được sư dụng là SNAT (source NAT) 
VD:
```
#iptables -t NAT -A POSTROUTING -p tcp -j SNAT --to-source 10.0.30.200
```
Câu lệnh này có ý nghĩa đổi địa chỉ nguồn đối với gói tin tcp thành 10.0.30.200

- **Chain** OUTPUT :

**Note:** Các ví dụ trên đây tôi không đề cập đến các tùy chọn được sử dụng như -t , -p ,-j là gì mà tôi muốn chỉ cho các bạn thấy bao quát của NAT: Các bạn có thể hiểu nôm na là trong bảng NAT chứa 3 chain và trong mỗi trên lại có các rule và target. Các bạn có thể tham khảo thêm bảng sau đây để thấy rõ hơn sự phân cấp của nó:

<img class="image__pic js-image-pic" src="http://i.imgur.com/tLbwXZg.png" alt="" id="screenshot-image">

##### b. Bảng filter
Trong bảng filter có các **chain** được xây dựng sẵn:
- **chain** INPUT: đây là chain dùng để lọc các gói tin đầu vào. Chain này là chain các gói tin bắt buộc phải đi qua để có thể được xử lý
- **Chain** OUTPUT: đây là chain dùng để lọc các gói tin đâu ra. Chain này là chain sau khi gói tin được xử lý phải đi qua chain này để ra được bên ngoài.
- **Chain** FORWARD : đây là chain dùng để chuyển gói tin qua lại giữa các card mạng với nhau.

Các target được sử dụng trong các chain này có thể là ACCEPT (đồng ý) , DROP (xóa bỏ)

##### c. Bảng Mangle
Trong bảng mangle bao gồm tất cả các chain được xây dựng sẵn là: PREROUTING, POSTROUTING, INPUT, OUTPUT, FORWARD. Bảng mangle rất ít được sử dụng trong mạng SOHO nên tôi sẽ không đề cập đến (cũng một phần là tôi cũng chưa thành thạo sử dụng bảng này lắm vì nó cần chuyên sâu về gói tin TCP)

OK!!! Bây giờ các bạn đã phân biệt rõ ràng đươc các tables, chain, rule, target rồi chứ. Tiếp theo để làm tốt phần iptabes nhất thiết các bạn phải hiểu **quá trình xử lý gói tin trong iptables** cụ thể là xử lý gói tin đối với quá trình NAT,FILTER,MANGLE để có thể đưa ra các luật cho gói tin

## 3. Quá trình xử lý gói tin trong iptables:

<img style="-webkit-user-select: none" src="http://vnexperts.net/images/stories/iptable.jpg">

Đây là quá trình xử lý gói tin trong iptables. Các gói tin mặc định sẽ **phải** đi qua các bảng này để hoàn thành xong một chu trình xử lý.

Đầu tiên gói tin từ mạng A đi vào hệ thống firewall sẽ phải đi qua bảng Mangle với chain là PREROUTING (với mục đích để thay đôi một số thông tin của gói tin trước khi đưa qua quyết định dẫn đường) sau đó gói tin đến bảng NAT với với chain PREROUTING tại đây địa chỉ đích của gói tin có thể bị thay đổi hoặc không, qua bộ routing và sẽ quyết định xem gói tin đó thuộc firewall hay không:
- TH1: gói tin đó là của firewall: gói tin sẽ đi qua bảng mangle và đến bản filter với chai là INPUT. Tại đây gói tin sẽ được áp dụng chính sách (rule) và ứng với mỗi rule cụ thể sẽ được áp dụng với target, sau quá trình xử lý gói tin sẽ đi đến bảng mangle tiếp đến là bảng NAT với chain OUTPUT được áp dụng một số chính sách và sau đó đi lần lượt qua các bảng magle với chain POSTROUTING cuối cùng đi đến bảng NAT với chain POSTROUTING để thay đổi địa chỉ nguồn nếu cần thiết.
- TH2: gói tin không phải của firewall sẽ được đưa đến bảng mangle với chain FORWARD đến bảng filter với chain FORWARD.
Đây là chain được sử dụng rất nhiều để bảo vệ người sử dụng mang trong lan với người sử dụng internet các gói tin thoải mãn các rule đặt ra mới có thể được chuyển qua giữa các card mạng với nhau, qua đó có nhiệm vụ thực hiện chính sách với người sử dụng nội bộ nhưng không cho vào internet, giới hạn thời gian,...và bảo vệ hệ thống máy chủ đối với người dung internet bên ngoài chống các kiểu tấn công. sau khi đi qua card mạng với nhau gói tin phải đi lần lượt qua bảng mangle và NAT với chain POSTROUTING để thực hiên việc chuyển đổi địa chỉ nguồn với target SNAT & MASQUERADE.

## 4. Một số tùy chọn và target thường sử dụng

##### a. Tùy chọn 1:
|Tùy chọn | Ý nghĩa|
|---------|--------|
|-A chain rule| Thêm Rule vào chain|
|-D [chain] [index] | Xóa rule có chỉ số trong chain đã chọn |
|-E [chain][new chain] | đổi tên cho chain |
|-F [chain] | Xóa tất cả các rule trong chain đã chọn, nếu ko chọn chain mặc định sẽ xóa hết rule trong tất cả các chain |
|-L [chain] | Hiển thị danh sách tất cả các rule trong chain, nếu ko chọn chain thì mặc định nó sẽ hiện hết chain trong một table |
|-P [chain][target] | Áp dụng chính sách đối với chain.|
|-Z [chain] | Xóa bộ đếm của chain đi |
|-N [name new chain] | Tạo một chain mới |

##### b. Tùy chọn 2:
| Tùy chọn | Ý nghĩa |
|----------|---------|
|-j [target] | dùng để chỉ rõ gói tin sau khi thoải mãn rule sẽ được nhảy đến taret để xử lý |
|-m [match] | dùng để mở rộng rule đối với với một gói tin (*) |
|-t [table] | dùng để chọn bảng. nếu bạn không chọn thì mặc định iptable sẽ chọn bảng filter |
|-p [protocol] | chỉ ra gói tin thuộc loại nào: tcp, udp, icmp,... |

Trên đây là các tùy chọn thường được sử dụng trong quá trình sử dụng iptables đói với firewall. Không phải ngẫu nhiên tôi lại chia thành 2  tùy chọn mà trong quá trình học và lab với iptables vô hình nó hình thành 2 phần tùy chọn có những điểm khác biệt mà tôi khó đặt tên được cho nó ý là gì.

## 5. Lời kết
Trên đây là những đúc rút của tôi sau quá trình tìm hiểu về iptables. Iptables là một mảng kiến thức khá rộng và khó. bài viết này tôi cũng chưa khai thác được bảng mangle trong iptable và các tác vụ nâng cao đối với nó, rất mong các bạn đọc và cùng chia sẻ những kiến thức về iptables.

Các bạn có thể liên hệ với tôi qua skype để cùng trao đổi về iptables nói riêng và kiến thức về Linux nói chung..

skype: namptit307

**Note:** [Link download sách về iptales](http://it-ebooks.info/book/338/)



