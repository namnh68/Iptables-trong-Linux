Iptables-trong-Linux
====================
Xin chào các bạn.Tôi viết bài viết này để chia sẻ những thứ hay ho trong quá trình tìm hiểu về iptabls, và một số lưu ý trong quá trình tìm hiểu về nó:
Nội dung bài viết:

1. Các kiến thức cần phải có khi học về iptables
2. Phân mục rõ ràng các bảng, chain, rule trong iptables
3. Một số target thường áp dụng
4. Thêm chút kiến thức về tạo new chain

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

VD: bộ lọc lọc các gói tin có kiểu tcp thì tất cả các gói tin có giao thức tcp sẽ được iptables giữ lại và phân tích tiếp

