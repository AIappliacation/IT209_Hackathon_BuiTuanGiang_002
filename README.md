# DevOps Hackathon - Đề 002

## 1. Thông tin sinh viên

| Họ và tên | Mã sinh viên | Lớp | Tài khoản Linux | GitHub | Cổng Nginx |
|-----------|--------------|-----|-----------------|--------|------------|
| Bùi Tuấn Giang | PTIT-HCM-067 | KS24-CNTT1 | buituangiang-ks24cntt1 | [AIappliacation](https://github.com/AIappliacation) | 8080 |

## 2. Môi trường triển khai

- **Hệ điều hành**: Ubuntu 22.04/24.04 (VPS Azure)
- **Nginx version**: [Cài đặt trên server]
- **Git version**: [Cài đặt trên server]
- **Địa chỉ triển khai**: VPS Azure (20.18.24.239)
- **Kết nối SSH**: `ssh azureuser@20.18.24.239`

## 3. Cấu trúc dự án

```
devops-hackathon-de002-buituangiang/
├── src/
│   └── index.html          # Web root
├── nginx/
│   └── buituangiang-ks24cntt1.conf  # Cấu hình Nginx
├── screenshots/
│   └── 04-website.png      # Ảnh chụp website
├── .gitignore              # File ignore
└── README.md               # Tài liệu dự án
```

## 4. Cấu hình Nginx

| Tham số trong template | Giá trị điền | Giải thích |
|------------------------|--------------|------------|
| `<PORT>` | 8080 | Cổng Nginx dành riêng |
| `<SERVER_NAME>` | 20.18.24.239 | Địa chỉ IP server |
| `<WEB_ROOT>` | /var/www/devops-hackathon-de002-buituangiang/src | Đường dẫn đến thư mục src |
| `<INDEX_FILE>` | index.html | File index mặc định |
| `<TEN_TAI_KHOAN>` | buituangiang-ks24cntt1 | Tên tài khoản Linux cho log file |
| `<ALLOW_DIRECTIVE>` | allow all; | Cho phép tất cả các request từ bên ngoài |

## 5. Tường lửa UFW

**Quy tắc đã thêm:**
```bash
sudo ufw allow 8080/tcp
sudo ufw allow ssh
sudo ufw enable
```

**Output của `sudo ufw status verbose`:**
```
Status: active

To                         Action      From
--                         ------      ----
8080/tcp                   ALLOW       Anywhere
22/tcp                     ALLOW       Anywhere
8080/tcp (v6)              ALLOW       Anywhere (v6)
22/tcp (v6)                ALLOW       Anywhere (v6)
```

## 6. Các bước triển khai

### 6.1 Tạo tài khoản người dùng
```bash
sudo adduser buituangiang-ks24cntt1
sudo usermod -aG sudo buituangiang-ks24cntt1
su - buituangiang-ks24cntt1
```

### 6.2 Cài đặt phần mềm
```bash
sudo apt update
sudo apt install nginx git ufw curl -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 6.3 Cấu hình Git
```bash
git config --global user.name "Bùi Tuấn Giang"
git config --global user.email "[email]"
```

### 6.4 Clone repository từ GitHub
```bash
sudo mkdir -p /var/www/devops-hackathon-de002
cd /var/www/devops-hackathon-de002
sudo git clone https://github.com/AIappliacation/IT209_Hackathon_BuiTuanGiang_002.git buituangiang
sudo chown -R buituangiang-ks24cntt1:buituangiang-ks24cntt1 /var/www/devops-hackathon-de002/buituangiang
```

### 6.5 Thiết lập quyền file
```bash
sudo find /var/www/devops-hackathon-de002/buituangiang -type d -exec chmod 755 {} \;
sudo find /var/www/devops-hackathon-de002/buituangiang -type f -exec chmod 644 {} \;
```

### 6.6 Cấu hình Nginx
```bash
sudo cp /var/www/devops-hackathon-de002/buituangiang/nginx/buituangiang-ks24cntt1.conf /etc/nginx/sites-available/
sudo ln -s /etc/nginx/sites-available/buituangiang-ks24cntt1.conf /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx
```

### 6.7 Cấu hình tường lửa
```bash
sudo ufw allow 8080/tcp
sudo ufw allow ssh
sudo ufw enable
```

## 7. Xác minh & Bằng chứng

![Website](screenshots/04-website.png)

Website có thể truy cập tại: `http://20.18.24.239:8080`

## 8. Quy trình cập nhật website

1. **Chỉnh sửa code** trên máy local
2. **Commit** thay đổi:
   ```bash
   git add .
   git commit -m "Mô tả thay đổi"
   ```
3. **Push** lên GitHub:
   ```bash
   git push origin main
   ```
4. **Pull** trên server:
   ```bash
   cd /var/www/devops-hackathon-de002/buituangiang
   git pull origin main
   ```
5. **Kiểm tra** website tại `http://20.18.24.239:8080`

## 9. Vấn đề gặp phải & Giải pháp (nếu có)

*(Chưa có vấn đề nào được ghi nhận)*