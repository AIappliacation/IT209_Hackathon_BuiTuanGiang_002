# Bài 4: Cấu hình Reverse Proxy Nginx cho ứng dụng Spring Boot

## 1. Mục tiêu

* Cấu hình Nginx Server Block.
* Phục vụ trang web tĩnh thông qua Nginx.
* Sử dụng Reverse Proxy để chuyển tiếp request API đến Spring Boot.
* Phân chia request:

  * `/` → nội dung tĩnh tại `/var/www/html/`
  * `/api/` → Spring Boot tại port `8082`
* Kiểm tra cú pháp Nginx trước khi reload.
* Kiểm tra kết quả bằng `curl`.

---

# 2. Kiểm tra Nginx

Kiểm tra Nginx đã được cài đặt:

```bash
nginx -v
```

Kết quả mẫu:

```text
nginx version: nginx/1.24.0 (Ubuntu)
```

Kiểm tra trạng thái:

```bash
sudo systemctl status nginx
```

Kết quả mong đợi:

```text
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded
     Active: active (running)
```

---

# 3. Tạo thư mục chứa trang web

Tạo thư mục:

```bash
sudo mkdir -p /var/www/html
```

Kiểm tra:

```bash
ls -ld /var/www/html
```

Kết quả mẫu:

```text
drwxr-xr-x 2 root root 4096 Oct  7 14:00 /var/www/html
```

---

# 4. Tạo file `index.html`

Tạo file:

```bash
sudo nano /var/www/html/index.html
```

Nhập nội dung:

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>PTIT DevOps Course</title>
</head>
<body>
    <h1>PTIT DevOps Course - Session 07</h1>

    <h2>Thông tin học viên</h2>

    <p><strong>Họ tên:</strong> Bùi Giang</p>
    <p><strong>Mã lớp:</strong> PTIT K24</p>

    <p>Nginx Reverse Proxy đang hoạt động.</p>
</body>
</html>
```

Lưu file:

```text
Ctrl + O
Enter
Ctrl + X
```

---

# 5. Kiểm tra nội dung trang HTML

Chạy:

```bash
sudo cat /var/www/html/index.html
```

Kết quả:

```text
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>PTIT DevOps Course</title>
</head>
<body>
    <h1>PTIT DevOps Course - Session 07</h1>

    <h2>Thông tin học viên</h2>

    <p><strong>Họ tên:</strong> Bùi Giang</p>
    <p><strong>Mã lớp:</strong> PTIT K24</p>

    <p>Nginx Reverse Proxy đang hoạt động.</p>
</body>
</html>
```

> Khi nộp bài, thay `Bùi Giang` và `PTIT K24` bằng thông tin học viên thực tế nếu mã lớp của bạn khác.

---

# 6. Kiểm tra Spring Boot backend

Trước khi cấu hình Reverse Proxy, cần đảm bảo ứng dụng Spring Boot đang chạy trên port `8082`.

Kiểm tra:

```bash
ss -tlnp | grep 8082
```

Kết quả mẫu:

```text
LISTEN 0      100      0.0.0.0:8082      0.0.0.0:*    users:(("java",pid=2458,fd=123))
```

Kiểm tra service:

```bash
sudo systemctl status spring-app.service
```

Kết quả cần có:

```text
Active: active (running)
```

---

# 7. Kiểm tra trực tiếp Spring Boot

Thử truy cập backend trực tiếp:

```bash
curl -I http://127.0.0.1:8082/
```

Ví dụ kết quả:

```text
HTTP/1.1 200
Content-Type: text/html;charset=UTF-8
```

Nếu ứng dụng không có endpoint `/`, có thể nhận:

```text
HTTP/1.1 404
```

Điều này không nhất thiết có nghĩa là Nginx bị lỗi. Quan trọng là Spring Boot đang lắng nghe port `8082`.

---

# 8. Tạo cấu hình Nginx Reverse Proxy

Tạo file:

```bash
sudo nano /etc/nginx/sites-available/spring-proxy.conf
```

Nhập cấu hình:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name _;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8082/;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Lưu file:

```text
Ctrl + O
Enter
Ctrl + X
```

---

# 9. Giải thích cấu hình

## Server Block

```nginx
server {
```

Định nghĩa một Virtual Host/Server Block của Nginx.

---

## Port 80

```nginx
listen 80;
listen [::]:80;
```

Nginx lắng nghe HTTP trên port `80` cho IPv4 và IPv6.

---

## Server name

```nginx
server_name _;
```

Dấu `_` được sử dụng để nhận các request không khớp với một domain cụ thể.

---

## Thư mục trang web

```nginx
root /var/www/html;
index index.html;
```

Nginx phục vụ nội dung tĩnh từ:

```text
/var/www/html/
```

File mặc định:

```text
index.html
```

---

# 10. Cấu hình Static Serve

Khối:

```nginx
location / {
    try_files $uri $uri/ =404;
}
```

Xử lý các request thông thường.

Ví dụ:

```text
http://IP_SERVER/
```

sẽ được Nginx tìm trong:

```text
/var/www/html/
```

Do đó:

```text
/var/www/html/index.html
```

sẽ được trả về trình duyệt.

---

# 11. Cấu hình Reverse Proxy

Khối:

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8082/;
```

có nhiệm vụ chuyển request `/api/` đến ứng dụng Spring Boot chạy tại:

```text
127.0.0.1:8082
```

Ví dụ client gửi:

```text
GET /api/health
```

Nginx sẽ chuyển tiếp đến backend.

---

# 12. Cấu hình HTTP Header

Thêm:

```nginx
proxy_set_header Host $host;
```

Để backend nhận được Host của request ban đầu.

```nginx
proxy_set_header X-Real-IP $remote_addr;
```

Để backend biết IP client.

```nginx
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

Lưu lại chuỗi IP của client qua các proxy.

```nginx
proxy_set_header X-Forwarded-Proto $scheme;
```

Cho backend biết request ban đầu sử dụng HTTP hay HTTPS.

---

# 13. Kích hoạt Server Block

Tạo symbolic link:

```bash
sudo ln -s /etc/nginx/sites-available/spring-proxy.conf /etc/nginx/sites-enabled/spring-proxy.conf
```

Kiểm tra:

```bash
ls -l /etc/nginx/sites-enabled/
```

Kết quả mẫu:

```text
lrwxrwxrwx 1 root root 52 Oct  7 14:10 spring-proxy.conf -> /etc/nginx/sites-available/spring-proxy.conf
```

---

# 14. Xử lý cấu hình Nginx mặc định

Ubuntu thường có file:

```text
/etc/nginx/sites-enabled/default
```

Nếu file `default` vẫn tồn tại, có thể gây xung đột với cấu hình bài lab.

Kiểm tra:

```bash
ls -l /etc/nginx/sites-enabled/
```

Nếu có:

```text
default
spring-proxy.conf
```

có thể tắt cấu hình mặc định:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

> Không xóa file gốc trong `sites-available`, chỉ xóa symbolic link trong `sites-enabled`.

---

# 15. Kiểm tra cú pháp Nginx

Đây là bước quan trọng trước khi reload.

Chạy:

```bash
sudo nginx -t
```

Kết quả mong đợi:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Nếu xuất hiện:

```text
syntax is ok
test is successful
```

thì cấu hình hợp lệ.

---

# 16. Reload Nginx

Sau khi kiểm tra cú pháp thành công:

```bash
sudo systemctl reload nginx
```

Kiểm tra:

```bash
sudo systemctl status nginx
```

Kết quả:

```text
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded
     Active: active (running)
```

---

# 17. Kiểm tra trang web tĩnh

Sử dụng:

```bash
curl -I http://localhost/
```

Kết quả mong đợi:

```text
HTTP/1.1 200 OK
Server: nginx/1.24.0
Content-Type: text/html
Content-Length:  ...
Connection: keep-alive
```

Mã:

```text
200 OK
```

cho biết Nginx đã phục vụ trang tĩnh thành công.

---

# 18. Kiểm tra nội dung trang web

Chạy:

```bash
curl http://localhost/
```

Kết quả:

```text
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>PTIT DevOps Course</title>
</head>
<body>
    <h1>PTIT DevOps Course - Session 07</h1>

    <h2>Thông tin học viên</h2>

    <p><strong>Họ tên:</strong> Bùi Giang</p>
    <p><strong>Mã lớp:</strong> PTIT K24</p>

    <p>Nginx Reverse Proxy đang hoạt động.</p>
</body>
</html>
```

---

# 19. Kiểm tra Reverse Proxy API

Kiểm tra endpoint:

```bash
curl -i http://localhost/api/health
```

Nếu Spring Boot có endpoint `/health`, kết quả mong đợi có thể là:

```text
HTTP/1.1 200
Server: nginx/1.24.0
Content-Type: application/json

{"status":"UP"}
```

Điều này chứng minh:

```text
Client
   |
   | HTTP :80
   v
 Nginx
   |
   | /api/
   v
Spring Boot :8082
```

---

# 20. Kiểm tra bằng IP của Azure VM

Lấy IP public:

```bash
curl -4 ifconfig.me
```

Ví dụ:

```text
20.xxx.xxx.xxx
```

Trên Windows mở trình duyệt:

```text
http://20.xxx.xxx.xxx/
```

Trang web phải hiển thị:

```text
PTIT DevOps Course - Session 07

Thông tin học viên

Họ tên: Bùi Giang
Mã lớp: PTIT K24

Nginx Reverse Proxy đang hoạt động.
```

---

# 21. Kiểm tra API từ máy Windows

Trên trình duyệt hoặc PowerShell:

```text
http://IP_AZURE:8082/api/health
```

hoặc:

```text
http://IP_AZURE/api/health
```

Trong bài Reverse Proxy, URL cần sử dụng là:

```text
http://IP_AZURE/api/health
```

vì Nginx nhận request tại port `80` rồi chuyển tiếp đến Spring Boot port `8082`.

---

# 22. Kiểm tra cấu hình cuối cùng

Kiểm tra Nginx:

```bash
sudo nginx -t
```

Kết quả:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Kiểm tra Nginx đang chạy:

```bash
sudo systemctl status nginx
```

Kết quả:

```text
Active: active (running)
```

Kiểm tra port 80:

```bash
ss -tlnp | grep :80
```

Kết quả:

```text
LISTEN 0      511      0.0.0.0:80      0.0.0.0:*    users:(("nginx",pid=1234,fd=6))
LISTEN 0      511         [::]:80         [::]:*    users:(("nginx",pid=1234,fd=7))
```

Kiểm tra Spring Boot:

```bash
ss -tlnp | grep 8082
```

Kết quả:

```text
LISTEN 0      100      0.0.0.0:8082      0.0.0.0:*    users:(("java",pid=2458,fd=123))
```

---

# 23. Sơ đồ hoạt động

```text
                    Internet
                        |
                        | HTTP :80
                        v
              +-------------------+
              |       Nginx       |
              |   Reverse Proxy   |
              +-------------------+
                   /          \
                  /            \
                 v              v
          /var/www/html/    /api/*
             Static             |
                                |
                                | proxy_pass
                                v
                    +----------------------+
                    |    Spring Boot       |
                    |      :8082           |
                    +----------------------+
```

---

# 24. Tổng hợp cấu hình

| Thành phần      | Cấu hình                                       |
| --------------- | ---------------------------------------------- |
| Nginx           | Port `80`                                      |
| Static root     | `/var/www/html/`                               |
| Static page     | `/var/www/html/index.html`                     |
| Nginx config    | `/etc/nginx/sites-available/spring-proxy.conf` |
| Enabled config  | `/etc/nginx/sites-enabled/spring-proxy.conf`   |
| API prefix      | `/api/`                                        |
| Backend         | `127.0.0.1:8082`                               |
| Spring Boot     | Port `8082`                                    |
| Reverse proxy   | `proxy_pass`                                   |
| Kiểm tra config | `sudo nginx -t`                                |

---

# 25. Cấu trúc nộp bài

Đường dẫn GitHub:

```text
homework/session_07/ex4/
```

Cấu trúc:

```text
homework/
└── session_07/
    └── ex4/
        ├── README.md
        └── spring-proxy.conf
```

Nội dung file `spring-proxy.conf`:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name _;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8082/;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

# 26. Kết luận

Bài thực hành đã cấu hình thành công Nginx làm Reverse Proxy cho ứng dụng Spring Boot.

Nginx tiếp nhận các request HTTP trên port `80`. Các request đến `/` được phục vụ trực tiếp từ thư mục:

```text
/var/www/html/
```

Trong khi các request bắt đầu bằng:

```text
/api/
```

được Nginx chuyển tiếp đến ứng dụng Spring Boot đang chạy tại:

```text
http://127.0.0.1:8082/
```

Cấu hình Nginx đã được kiểm tra bằng:

```bash
sudo nginx -t
```

và kết quả:

```text
syntax is ok
test is successful
```

cho thấy cấu hình hợp lệ.

Trang web tĩnh trả về HTTP `200 OK` và Reverse Proxy có thể chuyển tiếp request API đến Spring Boot backend.
