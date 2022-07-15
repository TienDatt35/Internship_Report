# **Table of Contents:**

## I. Một số khái niệm
- ### 1. Direct Connect
- ### 2. VPC
- ### 3. Subnet
- ### 4. Route tables
- ### 5. Internet gateway và NAT gateway
- ### 6. Security group và Network ACLs
- ### 7. Virtual Interface
- ### 8. VSwitch

## II. Mô hình và use case

- ### 1. Mô hình
- ### 2. Use case

## III. Định tuyến

- ### 1. Route table basic
- ### 2. Routing between subnets inside of a local VPC
- ### 3. Routing between multiple VPCs (VPC Peering)
- ### 4. Routing to the public internet (0.0.0.0/0)
- ### 5. Routing via VPN connection to an on-premise network

---

# **I. Một số khái niệm**:
### 1. Direct Connect
Direct connect cung cấp một đường truyền chuyên dụng, ổn định được thiết lập để kết nối riêng giữa AWS và on-premise 
Đặc điểm: 
- Không VPN
- Không gateway
- Không public ip

### 2. VPC
Virtual Private Cloud (VPC) cho phép bạn khởi chạy các tài nguyên trong môi trường mạng ảo của mình.  
Đặc điểm:
- IP trong VPC được định nghĩa bằng CIDR.  

### 3. Subnet
Subnet hay sub network (mạng con ảo). Bạn có thể chia một hoặc nhiều subnet trong mỗi AZs. Khi tạo một subnet cần định nghĩa khối CIDR cho subnet đó sao cho IP các subnet không được chồng lên nhau. Có 2 loại là public subnet và private subnet.  
- Public subnet: Được định tuyến đến một internet gateway để có thể giao tiếp với Internet thông qua public IP hoặc Elastic IP.
- Private subnet: Được định tuyến đến một NAT gateway. Không thể truy cập vào các instance này bằng internet.

### 4. Route tables
Là bảng định tuyến được sử dụng để xác định đường đi của các gói tin từ subnet hay gateway

### 5. Internet gateway và NAT gateway
- Internet gateway: Cho phép giao tiếp giữa public IP và internet.  
- NAT gateway: Cho phép giao tiếp giữa private IP và internet nhưng ngăn không cho internet kết nối đến instance đó.  


### 6. Security group và Network ACLs
- Sercurity group kiểm soát lưu lượng vào ra cho các instance. 
- Network ACLs kiểm soát lưu lượng vào ra cho các subnet.

### 7. Virtual Interface
Virtual interface (VIF) là giao diện cần thiết để truy cập các dịch vụ của AWS. Nó có thể là public hoặc private. 
- Public VIF dùng để truy cập dịch vụ như S3. 
- Private VIF dùng để truy cập dịch vụ VPC của mình.  

### 8. VSwitch
Một virtual switch (vswitch) là một chương trình phầm mềm cho phép một máy ảo (VM) giao tiếp với một máy ảo khác. Hoặc sử dụng để thiết lập kết nối giữa mạng ảo và mạng vật lý khác.

# **II. Mô hình và use case**:
### 1. Mô hình
[Mô hình](https://vticloud.io/wp-content/uploads/2021/05/AWS-Direct-Connect-la-gi.png)

### 2. Use case
[VPC with a single public subnet](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario1.html)  
[VPC with public and private subnets (NAT)](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)  
[VPC with public and private subnets and AWS Site-to-Site VPN access](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario3.html)  
[VPC with a private subnet only and AWS Site-to-Site VPN access](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario4.html)  

# **III. Định tuyến**:
### 1. Route Table Basics
Theo nghĩa chung một bảng định tuyến cho các gói mạng biết rằng chúng cần phải đi theo cách nào để đến đích. Các bảng tuyến đường được quản lý bởi các bộ định tuyến, hoạt động như "nút giao thông" trong mạng - chúng kết nối nhiều tuyến đường với nhau và chứa thông tin hữu ích để đưa lưu lượng truy cập đến điểm đến cuối cùng. Mỗi VPC có một bộ định tuyến VPC. Chức năng chính của bộ định tuyến VPC này là dùng tất cả các bảng định tuyến được xác định trong VPC đó rồi định hướng luồng lưu lượng bên trong VPC cũng như đến các mạng con bên ngoài VPC, dựa trên các quy tắc được xác định trong các bảng đó.  
Route tables bao gồm một danh sách các subnet cũng như "bước tiếp theo" để đến được đích.  
[Route tables có thể trông như sau](https://miro.medium.com/max/898/1*_AuYWR7QDEaRoxMYe85_6A.png)

### 2. Routing between subnets inside of a local VPC
Mặc định mỗi VPC đi kèm với 1 route table được định cấu hình trước với một tuyến đường “local”. Phạm vi của tuyến “local” chỉ nằm trong mạng con được xác định cho toàn bộ VPC. Ví dụ: nếu VPC của bạn được thiết lập để có không gian địa chỉ là 172.16.0.0/16, thì tuyến "local" của bạn sẽ được xác định là "172.16.0.0/16". Điều này cho phép tất cả các tài nguyên được tạo trong VPC nói chuyện với nhau mà không cần bất kỳ cấu hình bổ sung nào. Bạn không thể xóa tuyến đường "local" khỏi bảng định tuyến của mình và bất kỳ lúc nào một bảng định tuyến mới được tạo trong VPC, tuyến "local" mặc định được tạo.  
Trong VPC, các route table được gán cho các mạng con riêng lẻ. Bạn có thể tạo nhiều bảng định tuyến trong VPC nhưng bạn phải để nguyên 1 bảng định tuyến mặc định

### Configuration options:

Một bảng định tuyến duy nhất cho toàn bộ VPC  
Điều này có thể được sử dụng cho các môi trường đơn giản chỉ có các public subnet đều trỏ đến internet gateway và không yêu cầu các quy tắc định tuyến phức tạp hơn. Như được hiển thị bên dưới, chỉ có 1 bảng định tuyến nằm trong VPC và kết nối với mỗi mạng con.
[Một bảng định tuyến duy nhất](https://miro.medium.com/max/1400/1*Uy6F0TMzMKyGpItZBDWmZg.png)

Một bảng định tuyến cho mỗi subnet  
Trong trường hợp này, mỗi mạng con có 1 bảng định tuyến được gán và có mối quan hệ 1-1 giữa bảng định tuyến và mạng con trong VPC. Sử dụng phương pháp này gây ra nhiều chi phí quản lý hơn và tăng số lượng thay đổi cần thiết bất cứ khi nào có thay đổi định tuyến để triển khai trên toàn môi trường. Điều này thường được sử dụng trong các tình huống định tuyến phức tạp hơn. Trong sơ đồ bên dưới, bạn có thể thấy có 6 mạng con và 6 bảng định tuyến, mỗi bảng cho một mạng con.
[Mỗi bảng một subnet](https://miro.medium.com/max/1400/1*FF-kOl-UxRW6KRS_3-6oPg.png)

2-tier bảng định tuyến
Đối với các môi trường được chia thành các public subnet và private subnet, cách tốt nhất là nên có 1 bảng định tuyến cho các public subnet và 1 bảng định tuyến cho các private subnet. Trong trường hợp này, bạn cần phải tách các bảng tuyến đường của mình thành các availability zone. 
[2-tier bảng định tuyến](https://miro.medium.com/max/1400/1*4cRXG5XoWZbO7mhwjc0vrg.png)  

### 3. Định tuyến giữa các VPCs(VPC peering)
Trong các triển khai AWS lớn hơn, có thể có nhiều hơn 1 VPC. Điều này thường được sử dụng như một cách để tạo phân tách giữa các môi trường, thay vì chỉ sử dụng các mạng con khác nhau. Mặc dù bạn có thể thiết lập kết nối VPN giữa các VPC, nhưng điều đó sẽ làm phức tạp hơn mức cần thiết trong tình huống này. Để giải quyết vấn đề này, Amazon cung cấp VPC Peering. VPC Peering cho phép bạn yêu cầu kết nối ngang hàng với một VPC khác trong tài khoản của chính bạn hoặc trong một tài khoản Amazon khác, sau đó định tuyến giữa các VPC đó bằng cách chỉ cần thêm các tuyến vào bảng định tuyến. Lưu ý rằng, để điều này hoạt động, các mạng con trong mỗi VPC không được trùng nhau.  
Để cho phép kết nối giữa các VPC, bạn phải bắt đầu kết nối ngang hàng từ một VPC, sau đó chấp nhận yêu cầu trên VPC khác. Khi kết nối đã được chấp nhận và thiết lập, một route cần được thêm vào mỗi bảng định tuyến trong mỗi VPC.
[VPC peering](https://miro.medium.com/max/1400/1*jBndNOMq3sgpsd5R_1yG6g.png)

### 4. Định tuyến đến public internet
Để các tài nguyên bên trong VPC của bạn có thể truy cập public internet thì cần phải trỏ đến NAT Gateway hoặc Internet Gateway.  
Internet Gateway: Cổng internet (IGW) là thứ cho phép VPC tiếp cận public internet. Nó cho phép các thiết bị đã có public IP và được đặt trong public subnet kết nối trực tiếp với internet. Chỉ có 1 internet gateway cho mỗi VPC. Khi cổng internet được cấu hình và gắn vào VPC của bạn, bạn phải có một tuyến đường trong mỗi public subnet được xác định với CIDR đích là: 0.0.0.0  

NAT Gateway: Bởi vì private subnet chỉ chứa các tài nguyên có private IP, các thiết bị này không có cách nào để tự kết nối Internet, vì vậy chúng cần một chút trợ giúp. Đây là nơi xuất hiện NAT gateway. Một NAT gateway là thứ tạo điều kiện truy cập internet cho các tài nguyên nằm trong private subnet trong VPC. Cổng NAT (NGW) được đặt trong một public subnet trong VPC và được cấp một địa chỉ IP công cộng. Điều này cho phép NGW kết nối với internet thông qua internet gateway và chuyển private IP của các tài nguyên trong private subnet thành địa chỉ public có thể được sử dụng để kết nối với internet bên ngoài. Cổng NAT tạo điều kiện thuận lợi cho kết nối đó và ghi nhớ nơi định tuyến lưu lượng truy cập trở lại các địa chỉ IP riêng nguồn trong VPC. Để cho phép loại kết nối này, bảng định tuyến được liên kết với mỗi mạng con có một lộ trình được xác định với CIDR đích là: 0.0.0.0




