---
layout: post
title:  "Một số vấn đề khi làm việc với Mariadb"
date:   2016-08-11 03:43:45 +0700
categories: [mariadb]
---



**1. Sau cài đặt thì cần setup 1 số tool để có thể test hiệu năng Mariadb** 

* Lưu ý sau cài đặt (yum/apt) chạy lệnh `mysql_secure_installation` để tiếp tục cài đặt user/pass
* Dùng tool `mysqlslap` để test hiệu năng cấu hình 
[hướng dẫn cài đặt và sử dụng tại đây](https://www.digitalocean.com/community/tutorials/how-to-measure-mysql-query-performance-with-mysqlslap).
Lưu ý thời gian truy vấn cho mỗi query tính như sau: _Average number of seconds to run all queries / Average number of queries per client_
* Dùng tool [MySQLTuner](https://github.com/major/MySQLTuner-perl) để check cấu hình

**2. Một số câu truy vấn cần dùng**

Sau khi đăng nhập qua mysql client bằng tài khoản `root`

* Bật log truy vấn: 

`SET GLOBAL general_log=1, general_log_file='capture_queries.log';`

Lệnh này cho phép __Mariadb Server__ ghi lại toàn bộ query nhận được từ client. Lệnh này rất hữu ích đối với quá trình turning hệ thống.
Nó cho bạn biết mỗi request của bạn tới server thì dùng đến tài nguyên DB như thế nào. File `capture_queries.log` có thể dùng đường 
dẫn tương đối hoặc tuyệt đối, nếu là tương đối thì nó sẽ ghi vào thư mục mặc định cài mysql của bạn _(VD: /var/lib/mysql)_ 

* Kiểm tra giá trị các tham số cấu hình:

`show variables like '%tên tham số%'`

Lệnh này show ra giá trị các tham số đang __được cấu hình__ cho DB.

* Kiểm tra trạng thái của DB:

`show status like '%tên tham số%'';`

Lệnh này sẽ show ra __trạng thái sử dụng tài nguyên__ của DB đối với từng tham số cụ thể.

* Thay đổi giá trị của tham số cầu hình trong khi DB đang chạy:

`set global 'tên tham số' = 'giá trị mong muốn';`

**3. Một số tham số cấu hình ảnh hưởng đến hiệu năng của DB**

Sau đây là một số tham số mình chỉnh để nâng cao hiệu năng DB, việc chỉnh tham số nào và giá trị bao nhiêu thì không có giải pháp chung nhất
cho tất cả vì nó phụ thuộc vào tùy từng bài toán.

* `max_connections`:

Tham số này cho server biết nó sẽ chấp nhận (`max_connections + 1`) connections đồng thời kết nối tới.
Đây là tham số cần được quan tâm đầu tiên, nhất là khi bạn gặp lỗi `1040 Too many connections` thì có thể tham số này được cấu hình chưa phù hợp.
Bạn có thể set động giá trị này cho phù hợp với quy mô của App nếu lớn quá sẽ gây tốn RAM.

* `thread_cache_size`:

Mỗi connection tới DB, DB sẽ tạo mới 1 _thread_ để xử lý _query_ hoặc dùng lại _thread_ đã được tạo trước đó nếu có trên vùng nhớ _thread cache_.
Tham số này quy định giá trị của vùng nhớ đó. Muốn biết giá trị đó đã tối ưu chưa bạn có thể kiểm tra như sau:

Chạy lệnh:
`mysql> show status like 'Threads_created'; show status like 'Connections';`

Tính giá trị: `100 - ((Threads_created / Connections) * 100)`. 

Giá trị này cho biết tỷ lệ HIT cache của DB.
Theo mình thấy thì không phải cứ tỉ lệ HIT cache càng lớn càng tốt. Tốt nhất bạn nên thay đổi `thread_cache_size` và test liên tục bằng `mysqlslap`
sao cho đạt hiệu năng phù hợp.

* `innodb_buffer_pool_size`:

InnoDB engine sử dụng 1 vùng nhớ trên RAM gọi là __buffer pool__ để cache data và indexes để tăng tốc xử lý query.
Việc thay đổi tham số này bạn cần cân nhắc giữa lượng RAM sẽ bị tiêu tốn và tốc độ mong muốn. Nhất là khi DB chạy chung với App, nếu tăng 
nhiều quá có thể dẫn đến App bị thiếu RAM dẫn tới giảm hiệu năng của App.


* `skip-name-resolve`:

Mặc định Mariadb sẽ hỏi DNS server để biết kết nối của user đến từ IP address/Hostname nào. Bạn đặt giá trị của tham số này bằng 1 sẽ
bỏ qua việc này, cải thiện được chút ít thời gian.  

Mình đã từng gặp vấn đề với tham số này: DB mình đặt trong hạ tầng mạng của đối tác với tường lửa và mạng không ổn định lắm, bình thường việc tạo connection mất khoảng 10s @@!, nhưng khi thêm option này vào thì thời gian tạo connection đã giảm xuống < 1s. Điều này cho thấy tham số này đôi khi rất có tác dụng.

* `slow_query_log`:

Bật tham số này (set bằng ON) DB sẽ ghi lại truy vấn có thời gian vượt ngưỡng giá trị `long_query_time` (s). Giúp bạn dễ phát hiện và tối ưu lại câu truy vấn.
Log này sẽ được ghi theo cấu hình tham số `slow-query-log-file` 

Ngoài ra các bạn có thể tham khao các tips khác [tại đây](http://www.tecmint.com/mysql-mariadb-performance-tuning-and-optimization/)

_PS:_
_Sau khi lặp lại quá trình điều chỉnh tham số + đo hiệu năng nhiều lần các tham số trên hiệu năng của DB tăng lên khoảng `15%`._
_Kết hợp với xử lý 1 số nút thắt khác thì hiệu năng của service được tăng khoảng `6 lần` từ 200rq/s lên 1200 rq/s_
