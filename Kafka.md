# Ghi chép về Kafka 

<img src="https://github.com/locvx1234/Zookeeper-kafka/blob/master/image/logo_kafka.png">

## 1. Giới thiệu 

Kafka là một nền tảng luồng phân tán. 

#### Nền tảng luồng có 3 khả năng : 
	
1. Publish và subcribe các dòng bản ghi. Trong khía cạnh này, nó tương tự một hàng đợi message 
2. Lưu trữ các luồng bản ghi có khả năng chịu lỗi 
3. Xử lý các dòng bản ghi  
	
Kafka có thể hiểu là một hệ thống logging, lưu lại các trạng thái hệ thống nhằm phòng tránh mất thông tin. 
	
Message của Kafka được lưu trên đĩa cứng và được replicate trong cluster phòng tránh mất dữ liệu. 

#### Một vài khái niệm: 

- Kafka chạy như một cluster trên một hoặc nhiều server, mỗi server được gọi là `broker`.
- Kafka cluster lưu trữ, phân loại các message trong categories gọi là `topics`.
- Mỗi message bao gồm một key, một value và một timestamp.
- Kafka sử dụng `producers` để publish message vào các `topic`.
- Kafka sử dụng `consumers` để subcribe vào `topic`
	
<img src="https://raw.githubusercontent.com/locvx1234/Zookeeper-kafka/master/image/kafka_cluster.png">
	
	
#### Topic	

Topic có thể coi là trung gian giữa producers và consumers. Topic luôn luôn là multi-subcriber, một topic có thể có 0, 1, hoặc nhiều consumer mà subcribe các dữ liệu ghi vào nó.

Mỗi topic, Kafka Cluster duy trì một  partitioned log  như này: 

<img src="https://github.com/locvx1234/Zookeeper-kafka/blob/master/image/log_anatomy.png">
	
Mỗi partition là một chuỗi log, có thứ tự và không thể thay đổi.

Mỗi message trong partition sẽ có id tăng dần , gọi là offset

Kafka Cluster chứa tất cả các message đã publish (đã hoặc chưa consumer) - sử dụng một thời gian lưu trữ cấu hình. Ví dụ, nếu cấu hình 2 ngày, trong 2 ngày sau đó, một message publish, nó có thể consume. Sau đó nó sẽ được bỏ đi để giải phóng không gian. Hiệu suất của Kafka là có hiệu quả liên tục đối với dữ liệu trong một thời gian dài không phải là vấn đề.

<img src="https://github.com/locvx1234/Zookeeper-kafka/blob/master/image/log_consumer.png">

#### Producer

Producer sẽ publish các message vào topic. Producer có trách nhiệm lựa chọn message để gán cho partition nào trong topic. Điều này được thực hiện theo cách xoay vòng để cân bằng tải.

<img src="https://github.com/locvx1234/Zookeeper-kafka/blob/master/image/producer.png">

#### Consumer 

Consumer gắn nhãn cho chính nó với một tên nhóm người dùng, và mỗi message được publish đến một topic được gửi đến một consumer instance trong mỗi consumer group được subcribe. Các consumer instance có thể trên các tiến trình riêng hoặc trên các máy riêng biệt.

Nếu tất cả các consumer instance có cùng consumer group, các message sẽ có hiệu quả cân bằng trong các consumer instance. (Queuing)

Nếu tất cả  các consumer instance có consumer group khác nhau, khi đó mỗi message sẽ được broadcast đến tất cả các tiến trình consumer. (Pub/sub)

<img src="https://github.com/locvx1234/Zookeeper-kafka/blob/master/image/consumer.png">

## 2. Install Kafka 

### 2.1 Cài đặt

[Download](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.1.0/kafka_2.11-0.10.1.1.tgz) và giải nén

	$ tar -xzf kafka_2.11-0.10.1.1.tgz
	$ cd kafka_2.11-0.10.1.1

Bộ cài kafka đi kèm Zookeeper server nên nếu chưa có Zookeeper ở local thì có thể start một server Zookeeper:

	$ bin/zookeeper-server-start.sh config/zookeeper.properties
	
Sau đó khởi động Kafka server:

	$ bin/kafka-server-start.sh config/server.properties
	
### 2.2 Tạo một topic 
Topic với tên là "test":

	$  bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
	
Chúng ta sẽ kiểm tra topic với lệnh :

	$ bin/kafka-topics.sh --list --zookeeper localhost:2181
	test
	
Ngoài ra, thay vì tự tạo topic, bạn cũng có thể cấu hình các broker tự động tạo các topic khi một topic không tồn tại được publish.


### 2.3 Gửi một vài message:

Kafka sẽ lấy input từ file hoặc từ đầu vào tiêu chuẩn và gửi nó cho Kafka Cluster

	$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
	hello world
	I am LocVU
	
Nếu muốn input từ file ta dùng lệnh : 

	$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test < path/to/file/filename

### 2.4 Nhận message
Sử dụng consumer đi kèm bộ cài để in message vừa tạo 

	$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
	hello world
	I am LocVU

	
	
### 2.5 Cài đặt một multi-broker cluster

Chúng ta sẽ mở rộng với 3 broker 

Đầu tiên tạo file cấu hình cho mỗi broker

	$ cp config/server.properties config/server-1.properties
	$ cp config/server.properties config/server-2.properties
	
Sau đó sửa các file cấu hình mới tạo

	config/server-1.properties:
		broker.id=1
		listeners=PLAINTEXT://:9093
		log.dir=/tmp/kafka-logs-1

		
	
	config/server-2.properties:
		broker.id=2
		listeners=PLAINTEXT://:9094
		log.dir=/tmp/kafka-logs-2

Thuộc tính `broker.id` là duy nhất của mỗi node trong cluster. Chúng ta phải ghi đè các port và log directory.

Chúng ta đã có Zookeeper và 1 node đã start, bây giờ cần start 2 node mới : 

	$ bin/kafka-server-start.sh config/server-1.properties &
	
	$ bin/kafka-server-start.sh config/server-2.properties &
	

Bây giờ tạo một topic mới với replication factor : 3

	$ bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
	
Xem thông tin về cluster:

	$ bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
	
<img src="https://raw.githubusercontent.com/locvx1234/Zookeeper-kafka/master/image/multil-broker.png">

Dòng đầu tiên là tổng quan về tất cả các partition. 

"leader" là node chịu trách nhiệm đọc và ghi cho từng phân vùng. Mỗi node sẽ leader một phần ngẫu nhiên từ các phân vùng.

"replicas" là danh sách các node để tái tạo log cho partition này bất kể là lead hoặc ngay cả khi chúng còn sống.

"isr" là một tập  "in-sync replicas". Đây là tập con của danh sách các replicas hiện tại còn sống và bị đẩy lên làm leader.
	
Trong ví dụ này, node 1 đang là leader cho partition duy nhất của topic.

Chúng ta có thể chạy lệnh tương tự về topic lúc đầu (test):
 
	$ bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test

Các topic ban đầu không có bản sao (replicas) và đang trên server 0.

Publish một vài message cho topic mới : 

	$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
	This is my first line.
	Hello 
	World
	^C
	
Consume message: 

	$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
	This is my first line.
	Hello 
	World
	^C
	
Test lỗi tolerance. Broker 1 hoạt động như một leader nên ta sẽ kill nó:

	$ ps aux | grep server-1.properties
	7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
	kill -9 7564
	
Node 1 không còn trong tập in-sync replica (isr)

	$ bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
	
Nhưng các message vẫn khả dụng cho từng consume mặc dù leader thay đổi.

	$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
	

## 2.6 Sử dụng Kafka Connect để import/export data

Ghi dữ liệu và nhân dữ liệu là một cách đơn giản để bắt đầu. Nhưng chúng ta sẽ muốn sử dụng dữ liệu từ một nguồn khác và xuất ra từ Kafka tới các hệ thống khác.

Sử dụng Kafka Connect để làm được điều này.

Đầu tiên, tạo một file data để test:

	$  echo -e "foo\nbar" > test.txt
	
Chúng ta sẽ khởi độnng 2 connector chạy chế độ `standalone`. Có 3 file cấu hình : một file cho tiến trình  Kafka Connect , chứa cấu hình chung để kết nối, định dạng dữ liệu. Hai file còn lại, mỗi file xác định một kết nối được tạo.

	$ bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties

File `connect-file-source.properties` là source connector, nó đọc các dòng từ một file input và thực hiện produce tới topic

File `connect-file-sink.properties` là  sink connector, nó đọc các message từ Kafka topic.

Các file cấu hình mẫu này sử dụng cấu hình mặc đinh. Khi Kafka Connect, source connector đọc các dingf từ `test.txt` và produce chúng tới topic `connect-test`. Sau đó sink connector  sẽ đọc các message từ topic `connect-test` và ghi vào file `test.sink.txt` 

Xem file `test.sink.txt`  để thấy dữ liệu đã được ghi : 

	$ cat test.sink.txt

Chúng ta có thể xem data trong topic bằng console consume :

	$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
	
Các connector vẫn tiếp tục xử lý dữ liệu, nên chúng ta có thể thêm dữ liệu để xem sự thay đổi :

	$ echo "Another line" >> test.txt
	
Bạn sẽ nhìn thấy sự thay đổi trong output của console consume và file sink.

## 2.7 Sử dụng Kafka Stream để xử lý dữ liệu 

Kafka Stream là một library để xử lý dòng theo thời gian thực và phân tích dữ liệu được lưu trữ trong Kafka broker. 

Ví dụ `WordCountDemo` 


	KTable wordCounts = textLines
		// Split each text line, by whitespace, into words.
		.flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))

		// Ensure the words are available as record keys for the next aggregate operation.
		.map((key, value) -> new KeyValue<>(value, value))

		// Count the occurrences of each word (record key) and store the results into a table named "Counts".
		.countByKey("Counts")
		
Chuẩn bị một số input để sử dụng với Kafka Stream:

	$  echo -e "all streams lead to kafka\nhello kafka streams\njoin kafka summit" > file-input.txt
	
Bây giờ sẽ gửi đến làm input cho topic ` streams-file-input` sử dụng  console producer:


	$ bin/kafka-topics.sh --create \
            --zookeeper localhost:2181 \
            --replication-factor 1 \
            --partitions 1 \
            --topic streams-file-input
			
	$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic streams-file-input < file-input.txt
	
Chạy demo WordCount  để xử lý input: 


	$ bin/kafka-run-class.sh org.apache.kafka.streams.examples.wordcount.WordCountDemo
	
Output sẽ liên tục ghi vào trong một topic khác tên là `streams-wordcount-output` trong Kafka. 

Xem output nhân được trong topic `stream-wordcount-output`:

	$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
            --topic streams-wordcount-output \
            --from-beginning \
            --formatter kafka.tools.DefaultMessageFormatter \
            --property print.key=true \
            --property print.value=true \
            --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
            --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer

*Tham khảo*

https://kafka.apache.org/
	
https://kipalog.com/posts/Tim-hieu-ve-apache-kafka
