# Module 02 - Dịch vụ mạng trên AWS
---
### **Module 02-01: AWS Virtual Private Cloud (VPC)**

Đây là dịch vụ nền tảng, cho phép bạn tạo một mạng ảo riêng biệt và cô lập trên AWS Cloud.

* **VPC và Subnet**:
    * **VPC** nằm trong một Region (ví dụ: Singapore) và có thể bao quát nhiều Availability Zone (AZ) để đảm bảo tính sẵn sàng cao.
    * Khi tạo VPC, bạn cần xác định một dải địa chỉ IP (CIDR). Mỗi tài khoản có thể tạo tối đa 5 VPC trong một Region.
    * **Subnet** là các mạng con được chia từ VPC và nằm trong một AZ cụ thể.
    * AWS giữ lại 5 địa chỉ IP đầu tiên trong mỗi subnet cho các mục đích riêng (router, DNS, network address, broadcast).
    * **Public Subnet** là mạng con có thể kết nối ra ngoài Internet, trong khi **Private Subnet** thì không.

* **Định tuyến và Kết nối**:
    * **Route Table** là bảng định tuyến giúp xác định đường đi cho traffic trong mạng.
    * **Internet Gateway (IGW)** là cổng kết nối để các máy chủ trong Public Subnet có thể truy cập Internet.
    * **NAT Gateway** cho phép máy chủ trong Private Subnet truy cập Internet, nhưng chỉ theo chiều đi ra.
    * **VPC Endpoint** giúp kết nối các tài nguyên trong VPC với các dịch vụ khác của AWS (như S3, DynamoDB) mà không cần đi qua Internet, giúp tăng tốc độ và giảm chi phí.
    * **Elastic Network Interface (ENI)** là card mạng ảo được gán cho các máy chủ ảo (EC2).
    * **Elastic IP (EIP)** là một địa chỉ IPv4 công cộng tĩnh có thể gán cho ENI. Lưu ý: EIP sẽ bị tính phí nếu không được sử dụng.

* **Bảo mật**:
    * **Security Group (SG)** là tường lửa ảo có trạng thái (stateful). Nó kiểm soát lưu lượng đến và đi của một tài nguyên (ví dụ: máy chủ EC2) và chỉ cho phép các rule "allow" (cho phép).
    * **Network Access Control List (NACL)** là tường lửa ảo không có trạng thái (stateless), kiểm soát lưu lượng ở mức Subnet. Nó hỗ trợ cả rule "allow" và "deny".
    * **VPC Flow Logs** ghi lại metadata của các gói tin IP (như IP nguồn, IP đích, cổng kết nối) đi qua các ENI, giúp bạn giám sát và phân tích lưu lượng mạng.

---

### **Module 02-02: Kết nối nhiều VPC và Môi trường On-premises**

Khi bạn có nhiều VPC hoặc muốn kết nối mạng riêng của mình với AWS, bạn sẽ cần các dịch vụ sau:

* **VPC Peering**:
    * Cho phép kết nối hai hoặc nhiều VPC trực tiếp với nhau, giúp các tài nguyên trong đó có thể giao tiếp mà không cần đi qua Internet.
    * Đây là kết nối 1-1, không hỗ trợ định tuyến bắc cầu (transitive routing).
    * Hạn chế chính là cần cấu hình thủ công rất nhiều khi số lượng VPC tăng lên (ví dụ: 30 VPC cần tới 435 kết nối peering).

* **Transit Gateway (TGW)**:
    * Giải pháp cho việc kết nối số lượng lớn VPC bằng cách đóng vai trò như một "hub" trung tâm.
    * Giúp đơn giản hóa mô hình mạng và giảm bớt sự phức tạp của việc cấu hình route table.
    * Hỗ trợ kết nối các VPC và cả mạng on-premises.
    * Bạn chỉ cần tạo một TGW và các **Transit Gateway Attachment** để kết nối các VPC vào hub này.

* **Kết nối môi trường Hybrid (Cloud + On-premises)**:
    * **VPN Site-to-Site**: Dùng để thiết lập kết nối liên tục giữa trung tâm dữ liệu truyền thống của bạn với VPC trên AWS.
    * **AWS Direct Connect**: Dịch vụ giúp tạo một kết nối riêng tư, có độ trễ thấp và ổn định từ trung tâm dữ liệu của bạn đến AWS.
    * Lưu ý: Direct Connect không mã hóa dữ liệu, vì vậy bạn nên sử dụng thêm VPN Site-to-Site để bảo mật.

---

### **Module 02-03: Cân bằng tải và các tính năng nâng cao khác**

* **Elastic Load Balancing (ELB)**:
    * Đây là dịch vụ cân bằng tải được quản lý hoàn toàn bởi AWS, giúp phân phối lưu lượng truy cập cho nhiều máy chủ EC2 hoặc container.
    * **Health Check**: ELB chỉ gửi lưu lượng truy cập đến các máy chủ đang hoạt động ổn định.
    * **Access Logs**: Dữ liệu log truy cập có thể được lưu trữ trên S3 để phân tích.
    * Có bốn loại ELB:
        1.  **Application Load Balancer (ALB)**: Hoạt động ở Layer 7 (Application). Hỗ trợ các giao thức HTTP/HTTPS. Có tính năng định tuyến dựa trên đường dẫn (path-based routing) giúp phân phối traffic linh hoạt.
        2.  **Network Load Balancer (NLB)**: Hoạt động ở Layer 4 (Transport). Hỗ trợ giao thức TCP/TLS. Có hiệu năng cao nhất và hỗ trợ gán địa chỉ IP tĩnh.
        3.  **Classic Load Balancer (CLB)**: Hỗ trợ cả Layer 4 và 7.
        4.  **Gateway Load Balancer (GWLB)**: Hoạt động ở Layer 3 (Network). Được dùng để định tuyến traffic đến các thiết bị mạng ảo (virtual appliances) như firewall.

* **VPN Client-to-Site**:
    * Cho phép một thiết bị (host) duy nhất truy cập vào các tài nguyên trong VPC. Chi phí dịch vụ khá cao nên thường dùng các giải pháp bên thứ ba.