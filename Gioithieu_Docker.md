## 1. Khái niệm Container
- Container
  - Đóng gói ứng dụng trong môi trường nó chạy
  - Cô lập với môi trường bên ngoài
  - Tạo 1 môi trường chứa mọi điều kiện cần để ứng dụng chạy
  - Các tiến trình container cô lập với container khác nhưng vẫn dùng chung hệ thống
- Ưu điểm:
  - Linh động: Triển khai ở bất kỳ nơi đâu do sự phụ thuộc của ứng dụng vào tầng OS cũng như cơ sở hạ tầng được loại bỏ.
  - Nhanh: Do chia sẻ host OS nên container có thể được tạo gần như một cách tức thì. Điều này khác với vagrant - tạo môi trường ảo ở level phần cứng, nên khi khởi động mất nhiều thời gian hơn.
  - Nhẹ: Container cũng sử dụng chung các images nên cũng không tốn nhiều disks.
  - Đồng nhất :Khi nhiều người cùng phát triển trong cùng một dự án sẽ không bị sự sai khác về mặt môi trường.
  - Đóng gói: Có thể ẩn môi trường bao gồm cả app vào trong một gói được gọi là container. Có thể test được các container. Việc bỏ hay tạo lại container rất dễ dàng.
  - Do có ưu điểm lớn như vậy, nên container (Docker) được sử dụng rất nhiều và rộng rãi bởi các công ty lớn như Google, Facebook, Netflix,...
- Nhược điểm
  - Nhưng nó cũng có một số nhược điểm về tính an toàn, do dùng chung OS nên nếu có bất kỳ lỗ hổng nào đó ở kernel thì nó sẽ ảnh hưởng tới tất cả các container có trong host đó. Và còn một vẫn đề nữa, nếu một ứng dụng nào đó trong container có được quyền superuser thì sẽ khác nguy hiểm.
## 2. Container && VM
![image](https://github.com/DinhHa1011/Docker/assets/119484840/6d795960-6fa5-4fb2-8275-6a4b6556eccf)
| --- | VMWare | Container |
| Cấu trúc ảo hóa | Sử dụng ảo hóa cấp độ máy ảo (VM-level virtualization), trong đó mỗi máy ảo chạy một hệ điều hành độc lập trên một hypervisor | Sử dụng ảo hóa cấp độ hệ điều hành (OS-level virtualization), chia sẻ hạt nhân (kernel) của hệ điề hành máy chủ và cung cấp một môi trường cô lập để chạy ứng dụng |
| Tài nguyên | VMware cung cấp một máy ảo đầy đủ với tài nguyên phần cứng được cấu hình trước như bộ nhớ, CPU, đĩa cứng và giao diện mạng | container chia sẻ tài nguyên vật lý của máy chủ và chỉ sử dụng những tài nguyên cần thiết để chạy ứng dụng, làm cho chúng nhẹ nhàng hơn và có khả năng chạy nhiều hơn trên một máy chủ |
| Khởi động và thời gian chạy | thường mất thời gian lâu hơn để khởi động và chạy do cần khởi động một hệ điều hành đầy đủ | khởi động nhanh hơn vì chúng sử dụng hạt nhân của hệ điều hành máy chủ và chia sẻ các thư viện và tài nguyên hệ thống |
| Quản lý và triển khai | cung cấp một hệ thống quản lý tập trung (vCenter) để quản lý máy ảo và tài nguyên trong môi trường ảo hóa | có thể được quản lý và triển khai bằng Docker hoặc các công cụ quản lý container khác như Kubernetes, và chúng thích hợp cho triển khai ứng dụng phân tán và có khả năng mở rộng |
| Độ cô lập và bảo mật | cung cấp cấp độ cô lập tốt giữa các máy ảo | cũng cung cấp cô lập, nhưng chúng chia sẻ hạt nhân và môi trường hệ điều hành. Điều này có thể tạo ra một số rủi ro bảo mật nếu không được cấu hình đúng |
