# Các bước cấu hình Giám sát MySQL với Zabbix Agent 2

Bước 1: Tạo User giám sát trong MySQL
Zabbix Agent 2 cần một user có quyền hạn vừa đủ để thu thập dữ liệu (metrics) từ MySQL mà không làm ảnh hưởng đến bảo mật hệ thống.
```
CREATE USER 'zbx_monitor'@'localhost' IDENTIFIED BY 'MatKhauManhCuaBan@123';
GRANT REPLICATION CLIENT, PROCESS, SHOW DATABASES, SHOW VIEW ON *.* TO 'zbx_monitor'@'localhost';
FLUSH PRIVILEGES;
```

Giải thích quyền: 
- REPLICATION CLIENT: Để kiểm tra trạng thái replication (nếu có).
- PROCESS: Để xem các luồng đang chạy (SHOW PROCESSLIST).
- SHOW DATABASES và SHOW VIEW: Để lấy danh sách và cấu trúc cơ bản phục vụ metric định lượng.

Bước 2: Cấu hình Zabbix Agent 2

Chúng ta sẽ khai báo thông tin đăng nhập của MySQL cho Zabbix Agent 2 thông qua file cấu hình của plugin.
```
vi /etc/zabbix/zabbix_agent2.d/plugins.d/mysql.conf
Plugins.Mysql.Sessions.mysql_server.Uri=tcp://localhost:3306
Plugins.Mysql.Sessions.mysql_server.User=zbx_monitor
Plugins.Mysql.Sessions.mysql_server.Password=MatKhauManhCuaBan@123

sudo systemctl restart zabbix-agent2
```

Bước 3: Kiểm tra Zabbix Agent 2 đã lấy được dữ liệu chưa
```
zabbix_agent2 -t mysql.ping[mysql_server]
```

Bước 3: Cấu hình trên Zabbix Web UI

Bây giờ bạn chỉ cần gán Template chuẩn của Zabbix vào Host MySQL là xong:
-  Đăng nhập vào Zabbix Web.
- Đi tới Configuration -> Hosts -> Chọn Host chạy MySQL của bạn (hoặc tạo mới).
- Tại tab Templates, tìm và thêm template: MySQL by Zabbix agent 2.
- Chuyển sang tab Macros, chúng ta cần khai báo lại tên Session đã tạo ở Bước 2 để template hiểu:
- Thêm Macro: {$MYSQL.DSN}
- Giá trị (Value): mysql_server (đây chính là tên session Plugins.Mysql.Sessions.mysql_server bạn đặt trong file config).

=> Hoặc có thể change trực tiếp trong template 
<img width="786" height="49" alt="image" src="https://github.com/user-attachments/assets/5378103a-8f9b-406f-9eac-f6d630c63d8f" />

Nhấn Update để lưu lại.

Các thông số (Metrics) quan trọng bạn sẽ nhận được

Sau khoảng 5-10 phút, dữ liệu sẽ đổ về phần Latest data. Hệ thống của bạn sẽ được giám sát toàn diện với các thông số:
- Trạng thái: MySQL up/down (Ping), Uptime.
- Hiệu năng lệnh: Số lượng câu lệnh SELECT, INSERT, UPDATE, DELETE mỗi giây.
- Kết nối: Số lượng connection đang mở, max connection, connection bị từ chối.
- Bộ nhớ đệm: InnoDB Buffer Pool utilization (rất quan trọng cho hiệu năng).
- Traffic: Băng thông mạng In/Out của database.

Riêng việc giám sát replication giữa 2 node master - slave, sẽ có các trigger ở node slave

<img width="1720" height="176" alt="image" src="https://github.com/user-attachments/assets/92cb48e1-78a8-40f5-9876-ca9e7b8f6dde" />
