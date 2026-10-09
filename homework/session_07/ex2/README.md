# Bài 2: Quản trị Tường lửa UFW cho Cụm Dịch vụ Multi-port

## 1. Mục tiêu

* Thiết lập tường lửa UFW trên Ubuntu Server.
* Chặn mặc định tất cả kết nối đi vào server.
* Cho phép SSH thông qua port `22/tcp`.
* Cho phép Nginx/HTTP thông qua port `80/tcp`.
* Cho phép ứng dụng Spring Boot thông qua port `8082/tcp`.
* Không mở port MySQL `3306/tcp` ra Internet.
* Kiểm tra trạng thái và các rule của UFW.

---

# 2. Kiểm tra trạng thái UFW ban đầu

Kiểm tra UFW:

```bash
sudo ufw status verbose
```

Nếu UFW chưa được kích hoạt, kết quả có thể là:

```text
Status: inactive
```

Trong trường hợp UFW đã được cấu hình từ các bài trước, có thể xuất hiện:

```text
Status: active
```

---

# 3. Cấu hình chính sách mặc định

Đặt chính sách mặc định:

```bash
sudo ufw default deny incoming
```

Kết quả:

```text
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)
```

Cho phép tất cả kết nối đi ra:

```bash
sudo ufw default allow outgoing
```

Kết quả:

```text
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)
```

Kiểm tra lại:

```bash
sudo ufw status verbose
```

Kết quả mẫu:

```text
Status: inactive
Default: deny (incoming), allow (outgoing), disabled (routed)
```

---

# 4. Cho phép SSH port 22

Vì đang quản trị server thông qua SSH nên cần cho phép port `22/tcp` trước.

Chạy:

```bash
sudo ufw allow 22/tcp
```

Kết quả:

```text
Rule added
Rule added (v6)
```

Kiểm tra:

```bash
sudo ufw status
```

Kết quả:

```text
Status: inactive

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
```

---

# 5. Cho phép HTTP port 80

Mở port HTTP:

```bash
sudo ufw allow 80/tcp
```

Kết quả:

```text
Rule added
Rule added (v6)
```

Port `80/tcp` được sử dụng cho dịch vụ web HTTP, ví dụ Nginx.

---

# 6. Cho phép Spring Boot port 8082

Mở port ứng dụng:

```bash
sudo ufw allow 8082/tcp
```

Kết quả:

```text
Rule added
Rule added (v6)
```

Port `8082/tcp` được sử dụng cho ứng dụng Spring Boot.

---

# 7. Không mở port MySQL 3306

Theo yêu cầu bài thực hành, **không được thêm rule allow cho port 3306**.

Không chạy:

```bash
sudo ufw allow 3306/tcp
```

Do chính sách mặc định là:

```text
deny incoming
```

nên các kết nối từ bên ngoài đến port `3306` sẽ bị UFW chặn nếu không có rule cho phép riêng.

---

# 8. Kích hoạt UFW

Sau khi đã cho phép SSH port 22, kích hoạt UFW:

```bash
sudo ufw enable
```

Kết quả:

```text
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

---

# 9. Kiểm tra trạng thái UFW

Chạy:

```bash
sudo ufw status verbose
```

Kết quả mong đợi:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
8082/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
80/tcp (v6)                ALLOW IN    Anywhere (v6)
8082/tcp (v6)              ALLOW IN    Anywhere (v6)
```

---

# 10. Kiểm tra riêng từng rule

Có thể sử dụng:

```bash
sudo ufw status numbered
```

Kết quả mẫu:

```text
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 80/tcp                     ALLOW IN    Anywhere
[ 3] 8082/tcp                   ALLOW IN    Anywhere
[ 4] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 5] 80/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 6] 8082/tcp (v6)              ALLOW IN    Anywhere (v6)
```

Port `3306` không xuất hiện trong danh sách.

---

# 11. Kiểm tra port 3306

Kiểm tra xem UFW có rule cho MySQL hay không:

```bash
sudo ufw status | grep 3306
```

Nếu không có output:

```text
```

điều đó có nghĩa là không có rule `ALLOW` cho port `3306`.

Có thể kiểm tra trực tiếp toàn bộ rule:

```bash
sudo ufw status verbose
```

Không thấy:

```text
3306/tcp ALLOW
```

=> MySQL không được mở trực tiếp qua UFW.

---

# 12. Kiểm tra các rule hiện tại

Sử dụng:

```bash
sudo ufw show added
```

Kết quả mẫu:

```text
Added user rules (see 'ufw status' for running firewall):
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 8082/tcp
```

Không có:

```text
ufw allow 3306/tcp
```

---

# 13. Kiểm tra trạng thái các cổng đang lắng nghe

Có thể sử dụng:

```bash
sudo ss -tlnp
```

Ví dụ:

```text
State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port
LISTEN  0       511      0.0.0.0:80          0.0.0.0:*
LISTEN  0       128      0.0.0.0:22          0.0.0.0:*
LISTEN  0       4096     0.0.0.0:8082        0.0.0.0:*
```

Kết quả thực tế phụ thuộc vào những dịch vụ đang chạy trên server.

**Lưu ý:** UFW cho phép port `8082` không có nghĩa là chắc chắn `ss` sẽ hiển thị port `8082`. Port chỉ xuất hiện trong `ss` khi có chương trình thực tế đang `LISTEN` trên port đó.

---

# 14. Tổng hợp cấu hình

|     Port | Dịch vụ     | UFW           | Giải thích                       |
| -------: | ----------- | ------------- | -------------------------------- |
|   22/tcp | SSH         | ALLOW         | Cho phép quản trị server         |
|   80/tcp | HTTP/Nginx  | ALLOW         | Cho phép truy cập website        |
| 8082/tcp | Spring Boot | ALLOW         | Cho phép kiểm tra ứng dụng từ xa |
| 3306/tcp | MySQL       | DENY mặc định | Không mở trực tiếp ra Internet   |

---

# 15. Kiểm tra cuối cùng

Chạy:

```bash
sudo ufw status verbose
```

Kết quả cuối cùng:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
8082/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
80/tcp (v6)                ALLOW IN    Anywhere (v6)
8082/tcp (v6)              ALLOW IN    Anywhere (v6)
```

Port `3306` không xuất hiện trong danh sách cho phép.

---

# 16. Kết quả đạt được

Sau khi thực hiện bài lab:

* UFW đã được kích hoạt.
* Chính sách mặc định là `deny incoming`.
* Chính sách mặc định là `allow outgoing`.
* SSH port `22/tcp` được phép truy cập.
* HTTP port `80/tcp` được phép truy cập.
* Spring Boot port `8082/tcp` được phép truy cập.
* MySQL port `3306/tcp` không được mở ra Internet.
* Có thể kiểm tra toàn bộ cấu hình bằng `sudo ufw status verbose`.

---

# 17. Cấu trúc nộp bài

Đường dẫn GitHub:

```text
homework/session_07/ex2/
```

Cấu trúc:

```text
homework/
└── session_07/
    └── ex2/
        └── README.md
```

Trong `README.md` cần ghi lại quá trình cấu hình và kết quả của lệnh:

```bash
sudo ufw status verbose
```

Kết quả cần thể hiện rõ:

```text
Status: active
Default: deny (incoming), allow (outgoing)
22/tcp      ALLOW IN
80/tcp      ALLOW IN
8082/tcp    ALLOW IN
```

và **không có rule `ALLOW` cho `3306/tcp`**.

---

# 18. Kết luận

Bài thực hành đã cấu hình thành công UFW để bảo vệ máy chủ chạy nhiều dịch vụ. Tường lửa sử dụng chính sách mặc định chặn tất cả kết nối đi vào và chỉ mở các cổng cần thiết gồm SSH `22/tcp`, HTTP `80/tcp` và Spring Boot `8082/tcp`.

Cổng MySQL `3306/tcp` không được thêm rule cho phép nên vẫn bị chặn từ bên ngoài theo chính sách `deny incoming`. Cấu hình này giúp hạn chế việc cơ sở dữ liệu bị truy cập trực tiếp từ Internet.
