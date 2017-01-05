# Ghi chép về Zookeeper 

## 1. Tổng quan

Zookeeper là một dịch vụ tập trung để duy trì thông tin cấu hình, định danh, cung cấp dịch vụ phân tán và cung cấp nhóm dịch vụ. Tất cả các dịch vụ được sử dụng trong các ứng dụng phân tán, mỗi khi chúng được thực hiện nhiều công việc sửa chữa lỗi, vấn đề tương tranh là không tránh khỏi. Ngay cả khi thực hiện một cách chính xác, sự thực thi khác nhau của các dịch vụ này dẫn đến sự phức tạp quản lý khi ứng dụng được triển khai.

Zookeeper rút ra bản chất của các dịch vụ khác nhau vào một giao diện đơn giản thành một dịch vụ phối hợp tập trung. Các dịch vụ chính được phân phối với độ tin cậy cao. 

Quản lý nhóm và các giao thức sẽ được thực hiện bởi dịch vụ để các ứng dụng không phải thực hiện trên chính nó.

ZooKeeper Recipes cho thấy cách dịch vụ đơn giản này có thể được sử dụng để xây dựng các khái niệm trừu tượng mạnh hơn nhiều. 

ZooKeeper cho phép phân phối các tiến trình thông qua phân cấp của các thanh ghi dữ liệu (gọi là các thanh ghi znode), giống như một hệ thống tập tin. Không giống như các hệ thống tập tin bình thường, ZooKeeper cung cấp cho các client của nó thông lượng cao, độ trễ thấp, tính sẵn sàng cao, quyền truy cập vào các znode nghiêm ngặt. Hiệu suất của ZooKeeper cho phép nó được sử dụng trong hệ thống phân phối lớn. 

Namespace được cung cấp bởi ZooKeeper như một hệ thống file chuẩn. Mỗi tên là một chuỗi các phần tử được phân cách bởi dấu gạch chéo "/" . Mỗi znode trong namespace ZooKeeper được xác định bởi một đường dẫn. Mỗi znode sẽ có một znode cha, trừ root ("/"). Một znode có thể không bị xóa nếu nó có znode con.

Sự khác nhau chính giữa ZooKeeper và các file hệ thống chuẩn là mọi znode có thể liên kết dữ liệu với nó (mỗi tập tin có thể là thư mục và ngược lại) và các znode bị giới hạn số lượng dữ liệu có thể có. ZooKeeper được thiết kế để lưu trữ dữ liệu phối hợp : các thông tin trạng thái, cấu hình, thông tin vị trí,... Meta-information thường đo bằng kilobyte hoặc là byte. Nói chung, ZooKeeper sử dụng để lưu trữ nhiều dữ liệu có kích thước nhỏ.

<img src="https://github.com/locvx1234/Zookeeper-kafka/blob/master/image/scheme.png?raw=true">

Các server tạo nên dịch vụ ZooKeeper đều phải biết về nhau. Các client cũng phải biết danh sách các server. 

Mỗi client chỉ kết nối tới một ZooKeeper server. Client duy trì kết nối TCP để gửi nhận request, response. Nếu kết nối TCP đó bị ngắt, client sẽ kết nối tới một server khác. Khi một client đầu tiên kết nối tới ZooKeeper server, đầu tiên ZooKeeper server sẽ thiết lập một phiên làm việc cho các client. Nếu client cần phải kết nối với server khác, phiên này sẽ được tái lập với server mới.

Các yêu cầu đọc được gửi bởi một ZooKeeper client được xử lý bởi ZooKeeper server mà client được kết nối. Nếu các yêu cầu đọc trên một znode, nó cũng được theo dõi trên ZooKeeper server. Các yêu cầu ghi được chuyển tiếp tới các server khác và phải được sự đồng thuận. Yêu cầu đồng bộ hóa cũng được chuyển tiếp đến các máy chủ khác nhưng không qua sự đồng thuận. Như vậy, thông lượng yêu cầu đọc tỷ lệ với số lượng máy chủ và thông lượng của các yêu cầu ghi giảm theo số lượng máy chủ.








*Tham khảo*

https://cwiki.apache.org/confluence/display/ZOOKEEPER/ProjectDescription