# Ghi chép về Kafka 

## 1. Giới thiệu 

Kafka là một nền tảng luồng phân tán. 

####Nền tảng luồng có 3 khả năng : 
	
	1. Publish và subcribe các dòng bản ghi. Trong khía cạnh này, nó tương tự một hàng đợi message 
	2. Lưu trữ các luồng bản ghi có khả năng chịu lỗi 
	3. Xử lý các dòng bản ghi  
	
Kafka có thể hiểu là một hệ thống logging, lưu lại các trạng thái hệ thống nhằm phòng tránh mất thông tin. 
	
Message của Kafka được lưu trên đĩa cứng và được replicate trong cluster phòng tránh mất dữ liệu. 

####Một vài khái niệm: 

	- Kafka chạy như một cluster trên một hoặc nhiều server, mỗi server được gọi là `broker`.
	- Kafka cluster lưu trữ, phân loại các message trong categories gọi là `topics`.
	- Mỗi message bao gồm một key, một value và một timestamp.
	- Kafka sử dụng `producers` để publish message vào các `topic`.
	- Kafka sử dụng `consumers` để subcribe vào `topic`
	
<img src="">
	
	
####Topic	

Topic có thể coi là trung gian giữa producers và consumers. Topic luôn luôn là multi-subcriber, một topic có thể có 0, 1, hoặc nhiều consumer mà subcribe các dữ liệu ghi vào nó.

Mỗi topic, Kafka Cluster duy trì một  partitioned log  như này: 

<img src="">
	
Mỗi partition là một chuỗi log, có thứ tự và không thể thay đổi.

Mỗi message trong partition sẽ có id tăng dần , gọi là offset

Kafka Cluster chứa tất cả các message đã publish (đã hoặc chưa consumer) - sử dụng một thời gian lưu trữ cấu hình. Ví dụ, nếu cấu hình 2 ngày, trong 2 ngày sau đó, một message publish, nó có thể consume. Sau đó nó sẽ được bỏ đi để giải phóng không gian. Hiệu suất của Kafka là có hiệu quả liên tục đối với dữ liệu trong một thời gian dài không phải là vấn đề.

<img src="">

####Producer

Producer sẽ publish các message vào topic. Producer có trách nhiệm lựa chọn message để gán cho partition nào trong topic. Điều này được thực hiện theo cách xoay vòng để cân bằng tải.

<img src="">

####Consumer 

Consumer gắn nhãn cho chính nó với một têm nhóm người dùng, và mỗi message được publish đến một topic được gửi đến một consumer instance trong mỗi consumer group được subcribe. Các consumer instance có thể trên các tiến trình riêng hoặc trên các máy riêng biệt.

Nếu tất cả các consumer instance có cùng consumer group, các message sẽ có hiệu quả cân bằng trong các consumer instance. (Queuing)

Nếu tất cả  các consumer instance có consumer group khác nhau, khi đó mỗi message sẽ được broadcast đến tất cả các tiến trình consumer. (Pub/sub)

<img src="">

## 2. Install Kafka 

[Download](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.1.0/kafka_2.11-0.10.1.1.tgz) và giải nén

	$ tar -xzf kafka_2.11-0.10.1.1.tgz
	$ cd kafka_2.11-0.10.1.1

Bộ cài kafka đi kèm Zookeeper server nên nếu chưa có Zookeeper ở local thì có thể start một server Zookeeper:

	$ bin/zookeeper-server-start.sh config/zookeeper.properties
	
Sau đó khởi động Kafka server:

	$ bin/kafka-server-start.sh config/server.properties
	
Tạo một topic với tên là "test":

	$  bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
	
Chúng ta sẽ kiểm tra topic với lệnh :

	$ bin/kafka-topics.sh --list --zookeeper localhost:2181
	test
	
Ngoài ra, thay vì tự tạo topic, bạn cũng có thể cấu hình các broker tự động tạo các topic khi một topic không tồn tại được publish.


Gửi một vài message:

Kafka sẽ lấy input từ file hoặc từ đầu vào tiêu chuẩn và gửi nó cho Kafka Cluster

	$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
	hello world
	I am LocVU
	
Sử dụng consumer đi kèm bộ cài để in message vừa tạo 

	$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
	hello world
	I am LocVU





	

*Tham khảo*

https://kafka.apache.org/intro	
	
https://kipalog.com/posts/Tim-hieu-ve-apache-kafka