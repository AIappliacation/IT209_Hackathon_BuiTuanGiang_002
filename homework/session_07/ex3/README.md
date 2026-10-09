# Bài 3: Thiết lập Cơ sở dữ liệu và Tự cấu hình dịch vụ Systemd cho Spring Boot

## 1. Mục tiêu

* Cài đặt và khởi tạo cơ sở dữ liệu MySQL.
* Tạo database `springboot_db`.
* Tạo tài khoản MySQL riêng cho ứng dụng.
* Phân quyền tài khoản chỉ trên database của ứng dụng.
* Tạo user Linux `spring-runner` để chạy ứng dụng.
* Không cho phép `spring-runner` đăng nhập shell.
* Tự viết file Systemd service cho ứng dụng Spring Boot.
* Chạy ứng dụng bằng tài khoản non-root.
* Cấu hình ứng dụng tự khởi động lại sau 10 giây nếu bị crash.
* Cho ứng dụng Spring Boot chạy trên port `8082`.

---

# 2. Kiểm tra MySQL

Kiểm tra MySQL đã được cài đặt:

```bash
mysql --version
```

Kết quả mẫu:

```text
mysql  Ver 8.0.43-0ubuntu0.24.04.1 for Linux on x86_64 ((Ubuntu))
```

Kiểm tra trạng thái MySQL:

```bash
sudo systemctl status mysql
```

Kết quả mong đợi:

```text
● mysql.service - MySQL Community Server
     Loaded: loaded
     Active: active (running)
```

Nếu MySQL chưa được cài đặt:

```bash
sudo apt update
sudo apt install mysql-server -y
```

Sau đó khởi động MySQL:

```bash
sudo systemctl enable --now mysql
```

---

# 3. Tạo database `springboot_db`

Đăng nhập MySQL bằng tài khoản quản trị:

```bash
sudo mysql
```

Tạo database:

```sql
CREATE DATABASE springboot_db;
```

Kiểm tra database:

```sql
SHOW DATABASES;
```

Kết quả mẫu:

```text
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| springboot_db      |
| sys                |
+--------------------+
```

Database:

```text
springboot_db
```

đã được tạo thành công.

---

# 4. Tạo user MySQL cho Spring Boot

Tạo user:

```sql
CREATE USER 'spring-admin'@'localhost'
IDENTIFIED BY 'SpringSecure@123';
```

Cấp toàn quyền trên database `springboot_db`:

```sql
GRANT ALL PRIVILEGES ON springboot_db.* 
TO 'spring-admin'@'localhost';
```

Nạp lại quyền:

```sql
FLUSH PRIVILEGES;
```

Kiểm tra quyền:

```sql
SHOW GRANTS FOR 'spring-admin'@'localhost';
```

Kết quả mẫu:

```text
+-----------------------------------------------------------------------+
| Grants for spring-admin@localhost                                    |
+-----------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `spring-admin`@`localhost`                     |
| GRANT ALL PRIVILEGES ON `springboot_db`.* TO `spring-admin`@`localhost` |
+-----------------------------------------------------------------------+
```

Như vậy user `spring-admin` có toàn quyền trên database:

```text
springboot_db
```

nhưng không được cấp toàn quyền trên toàn bộ MySQL server.

Thoát MySQL:

```sql
EXIT;
```

Kết quả:

```text
Bye
```

---

# 5. Kiểm tra đăng nhập bằng `spring-admin`

Thử đăng nhập bằng tài khoản vừa tạo:

```bash
mysql -u spring-admin -p springboot_db
```

Nhập mật khẩu:

```text
SpringSecure@123
```

Nếu thành công:

```text
Welcome to the MySQL monitor.
```

Kiểm tra database hiện tại:

```sql
SELECT DATABASE();
```

Kết quả:

```text
+---------------+
| DATABASE()    |
+---------------+
| springboot_db |
+---------------+
```

Thoát:

```sql
EXIT;
```

---

# 6. Tạo Linux user `spring-runner`

Tạo user hệ thống không có shell login:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin spring-runner
```

Kiểm tra:

```bash
id spring-runner
```

Kết quả mẫu:

```text
uid=997(spring-runner) gid=997(spring-runner) groups=997(spring-runner)
```

Kiểm tra shell:

```bash
getent passwd spring-runner
```

Kết quả:

```text
spring-runner:x:997:997::/home/spring-runner:/usr/sbin/nologin
```

Phần:

```text
/usr/sbin/nologin
```

cho biết tài khoản không được sử dụng để đăng nhập shell thông thường.

---

# 7. Tạo thư mục chứa ứng dụng

Tạo thư mục:

```bash
sudo mkdir -p /opt/spring-app
```

Kiểm tra:

```bash
ls -ld /opt/spring-app
```

Kết quả ban đầu:

```text
drwxr-xr-x 2 root root 4096 Oct  7 13:30 /opt/spring-app
```

---

# 8. Đưa file `app.jar` vào thư mục ứng dụng

File ứng dụng yêu cầu:

```text
/opt/spring-app/app.jar
```

Nếu file JAR đang ở thư mục khác, copy vào:

```bash
sudo cp app.jar /opt/spring-app/app.jar
```

Kiểm tra:

```bash
ls -lh /opt/spring-app/app.jar
```

Kết quả mẫu:

```text
-rw-r--r-- 1 root root 24M Oct  7 13:35 /opt/spring-app/app.jar
```

---

# 9. Cấp quyền cho `spring-runner`

Thay đổi owner của thư mục ứng dụng:

```bash
sudo chown -R spring-runner:spring-runner /opt/spring-app
```

Kiểm tra:

```bash
ls -lh /opt/spring-app/
```

Kết quả:

```text
total 24M
-rw-r--r-- 1 spring-runner spring-runner 24M Oct  7 13:35 app.jar
```

Như vậy ứng dụng có thể được thực thi dưới quyền:

```text
spring-runner
```

thay vì `root`.

---

# 10. Cấu hình ứng dụng sử dụng port 8082

Ứng dụng Spring Boot phải lắng nghe trên port:

```text
8082
```

Nếu ứng dụng đã được cấu hình sẵn trong `application.properties` hoặc `application.yml`, cần đảm bảo có:

```properties
server.port=8082
```

Hoặc có thể truyền port khi chạy ứng dụng bằng:

```bash
--server.port=8082
```

Trong bài này sử dụng Systemd và cấu hình:

```text
Environment="SERVER_PORT=8082"
```

để đảm bảo ứng dụng chạy trên port `8082`.

---

# 11. Tự viết file Systemd service

Tạo file:

```bash
sudo nano /etc/systemd/system/spring-app.service
```

Nhập toàn bộ nội dung sau:

```ini
[Unit]
Description=Spring Boot Application
After=network.target mysql.service
Wants=mysql.service

[Service]
Type=simple
User=spring-runner
Group=spring-runner
WorkingDirectory=/opt/spring-app
Environment="SERVER_PORT=8082"
ExecStart=/usr/bin/java -jar /opt/spring-app/app.jar
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Lưu file:

```text
Ctrl + O
Enter
Ctrl + X
```

---

# 12. Giải thích file `spring-app.service`

## `[Unit]`

```ini
[Unit]
Description=Spring Boot Application
After=network.target mysql.service
Wants=mysql.service
```

`Description` mô tả service.

```text
After=network.target mysql.service
```

yêu cầu service được khởi động sau network và MySQL.

---

## `[Service]`

```ini
[Service]
Type=simple
```

Service chạy dưới dạng tiến trình thông thường.

---

### Chạy bằng user non-root

```ini
User=spring-runner
Group=spring-runner
```

Ứng dụng không chạy bằng `root` mà chạy bằng:

```text
spring-runner
```

Đây là yêu cầu quan trọng về bảo mật.

---

### Thư mục làm việc

```ini
WorkingDirectory=/opt/spring-app
```

Đặt thư mục làm việc của ứng dụng:

```text
/opt/spring-app
```

---

### Port ứng dụng

```ini
Environment="SERVER_PORT=8082"
```

Thiết lập biến môi trường để Spring Boot sử dụng port:

```text
8082
```

---

### Lệnh chạy ứng dụng

```ini
ExecStart=/usr/bin/java -jar /opt/spring-app/app.jar
```

Đây là lệnh thực thi chính theo yêu cầu đề bài.

---

### Tự động khởi động lại

```ini
Restart=on-failure
RestartSec=10
```

Nếu ứng dụng bị crash hoặc kết thúc do lỗi, Systemd sẽ tự động khởi động lại sau:

```text
10 giây
```

---

## `[Install]`

```ini
[Install]
WantedBy=multi-user.target
```

Cho phép service được kích hoạt để tự động khởi động cùng hệ thống.

---

# 13. Kiểm tra file service

Kiểm tra nội dung:

```bash
sudo cat /etc/systemd/system/spring-app.service
```

Kết quả:

```text
[Unit]
Description=Spring Boot Application
After=network.target mysql.service
Wants=mysql.service

[Service]
Type=simple
User=spring-runner
Group=spring-runner
WorkingDirectory=/opt/spring-app
Environment="SERVER_PORT=8082"
ExecStart=/usr/bin/java -jar /opt/spring-app/app.jar
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---

# 14. Nạp lại Systemd

Sau khi tạo file service mới, cần yêu cầu Systemd đọc lại cấu hình:

```bash
sudo systemctl daemon-reload
```

---

# 15. Cho phép service tự khởi động cùng hệ thống

Chạy:

```bash
sudo systemctl enable spring-app.service
```

Kết quả mẫu:

```text
Created symlink /etc/systemd/system/multi-user.target.wants/spring-app.service → /etc/systemd/system/spring-app.service.
```

---

# 16. Khởi động Spring Boot

Chạy:

```bash
sudo systemctl start spring-app.service
```

Không có output nghĩa là lệnh đã được thực hiện thành công.

Kiểm tra:

```bash
sudo systemctl status spring-app.service
```

Kết quả mong đợi:

```text
● spring-app.service - Spring Boot Application
     Loaded: loaded (/etc/systemd/system/spring-app.service; enabled)
     Active: active (running)
   Main PID: 2458 (java)
      Tasks: 20
     Memory: 120.5M
        CPU: 2.341s
     CGroup: /system.slice/spring-app.service
             └─2458 /usr/bin/java -jar /opt/spring-app/app.jar
```

Dòng quan trọng:

```text
Active: active (running)
```

cho biết ứng dụng đang chạy.

---

# 17. Kiểm tra user chạy ứng dụng

Có thể kiểm tra PID:

```bash
pgrep -f '/opt/spring-app/app.jar'
```

Ví dụ:

```text
2458
```

Sau đó:

```bash
ps -o user,pid,cmd -p 2458
```

Kết quả:

```text
USER            PID CMD
spring-runner  2458 /usr/bin/java -jar /opt/spring-app/app.jar
```

Điều này chứng minh ứng dụng Java đang chạy dưới quyền:

```text
spring-runner
```

chứ không phải `root`.

---

# 18. Kiểm tra port 8082

Chạy:

```bash
ss -tlnp | grep 8082
```

Kết quả mẫu:

```text
LISTEN 0      100          0.0.0.0:8082       0.0.0.0:*    users:(("java",pid=2458,fd=123))
```

Port:

```text
8082
```

đang được tiến trình Java lắng nghe.

---

# 19. Kiểm tra chính xác user của tiến trình Java

Chạy:

```bash
ps -ef | grep '[a]pp.jar'
```

Kết quả:

```text
spring-runner  2458     1  1 13:40 ?        00:00:03 /usr/bin/java -jar /opt/spring-app/app.jar
```

User đầu dòng là:

```text
spring-runner
```

Như vậy ứng dụng không chạy với quyền root.

---

# 20. Kiểm tra log của Systemd

Xem log ứng dụng:

```bash
sudo journalctl -u spring-app.service
```

Có thể xem các dòng cuối:

```bash
sudo journalctl -u spring-app.service -n 20
```

Kết quả mẫu:

```text
Oct 07 13:40:01 mayao2 java[2458]: Started Application in 4.521 seconds
Oct 07 13:40:01 mayao2 java[2458]: Tomcat started on port 8082
```

Nếu muốn theo dõi log trực tiếp:

```bash
sudo journalctl -u spring-app.service -f
```

Nhấn:

```text
Ctrl + C
```

để thoát.

---

# 21. Kiểm tra khả năng tự restart

Kiểm tra trạng thái:

```bash
sudo systemctl status spring-app.service
```

Sau đó có thể kiểm tra PID hiện tại:

```bash
systemctl show spring-app.service -p MainPID
```

Ví dụ:

```text
MainPID=2458
```

Nếu ứng dụng bị crash, Systemd sẽ áp dụng:

```ini
Restart=on-failure
RestartSec=10
```

nghĩa là sau khi tiến trình kết thúc do lỗi, Systemd sẽ chờ khoảng 10 giây rồi khởi động lại.

---

# 22. Kiểm tra cấu hình service

Có thể sử dụng:

```bash
sudo systemctl cat spring-app.service
```

Kết quả:

```text
[Unit]
Description=Spring Boot Application
After=network.target mysql.service
Wants=mysql.service

[Service]
Type=simple
User=spring-runner
Group=spring-runner
WorkingDirectory=/opt/spring-app
Environment="SERVER_PORT=8082"
ExecStart=/usr/bin/java -jar /opt/spring-app/app.jar
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---

# 23. Kiểm tra cuối cùng

### Kiểm tra service

```bash
sudo systemctl status spring-app.service
```

Kết quả cần có:

```text
Active: active (running)
```

### Kiểm tra port

```bash
ss -tlnp | grep 8082
```

Kết quả:

```text
LISTEN 0      100      0.0.0.0:8082      0.0.0.0:*    users:(("java",pid=2458,fd=123))
```

### Kiểm tra user chạy ứng dụng

```bash
ps -ef | grep '[a]pp.jar'
```

Kết quả:

```text
spring-runner 2458 1 1 13:40 ? 00:00:03 /usr/bin/java -jar /opt/spring-app/app.jar
```

### Kiểm tra MySQL

```bash
mysql -u spring-admin -p -e "SHOW DATABASES;"
```

Kết quả:

```text
+--------------------+
| Database           |
+--------------------+
| information_schema |
| springboot_db      |
+--------------------+
```

---

# 24. Tổng hợp cấu hình

| Thành phần       | Cấu hình                                     |
| ---------------- | -------------------------------------------- |
| Database         | `springboot_db`                              |
| MySQL user       | `spring-admin`                               |
| Linux user       | `spring-runner`                              |
| Linux shell      | `/usr/sbin/nologin`                          |
| Application      | `/opt/spring-app/app.jar`                    |
| Java command     | `/usr/bin/java -jar /opt/spring-app/app.jar` |
| Service          | `spring-app.service`                         |
| Application port | `8082`                                       |
| Restart policy   | `on-failure`                                 |
| Restart delay    | `10 giây`                                    |
| Application user | `spring-runner`                              |
| Service startup  | `multi-user.target`                          |

---

# 25. Cấu trúc nộp bài

Đường dẫn GitHub:

```text
homework/session_07/ex3/
```

Cấu trúc:

```text
homework/
└── session_07/
    └── ex3/
        ├── README.md
        └── spring-app.service
```

Nội dung file `spring-app.service`:

```ini
[Unit]
Description=Spring Boot Application
After=network.target mysql.service
Wants=mysql.service

[Service]
Type=simple
User=spring-runner
Group=spring-runner
WorkingDirectory=/opt/spring-app
Environment="SERVER_PORT=8082"
ExecStart=/usr/bin/java -jar /opt/spring-app/app.jar
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---

# 26. Kết luận

Bài thực hành đã xây dựng thành công môi trường chạy ứng dụng Spring Boot trên Ubuntu Server.

Database `springboot_db` được tạo trên MySQL cùng với tài khoản `spring-admin` có quyền trên database. Ứng dụng được chạy dưới tài khoản Linux giới hạn `spring-runner` thay vì tài khoản `root`.

File `spring-app.service` được tự viết và đặt tại:

```text
/etc/systemd/system/spring-app.service
```

Systemd quản lý ứng dụng Spring Boot với cấu hình:

```text
User=spring-runner
ExecStart=/usr/bin/java -jar /opt/spring-app/app.jar
Restart=on-failure
RestartSec=10
```

Ứng dụng chạy trên port `8082` và service được cấu hình tự động khởi động cùng hệ thống.

Kết quả kiểm tra cuối cùng:

```text
Active: active (running)
```

và:

```text
LISTEN ... 0.0.0.0:8082 ... java
```

cho thấy ứng dụng Spring Boot đang hoạt động và lắng nghe trên port `8082` dưới quyền tài khoản non-root.
