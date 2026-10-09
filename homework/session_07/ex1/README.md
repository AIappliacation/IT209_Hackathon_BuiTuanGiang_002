# Bài 1: Quản lý người dùng giới hạn và truyền tải dữ liệu qua SFTP trên Windows

## 1. Mục tiêu

* Tạo tài khoản người dùng giới hạn `sftp-user`.
* Không cấp quyền `sudo` cho tài khoản.
* Tạo thư mục chứa log giả lập trên Ubuntu Server.
* Cấp quyền để `sftp-user` chỉ có thể đọc file log.
* Sử dụng phần mềm SFTP Client trên Windows để kết nối đến server.
* Tải file `backup-check.log` từ server Linux về máy tính Windows.

---

## 2. Tạo tài khoản `sftp-user`

Đăng nhập vào Azure VM bằng tài khoản có quyền `sudo`.

Kiểm tra user hiện tại:

```bash
whoami
```

Kết quả:

```text
azureuser
```

Tạo tài khoản `sftp-user`:

```bash
sudo adduser sftp-user
```

Nhập mật khẩu mạnh cho tài khoản.

Kết quả mẫu:

```text
Adding user `sftp-user' ...
Adding new group `sftp-user' (1002) ...
Adding new user `sftp-user' (1002) with group `sftp-user' ...
Creating home directory `/home/sftp-user' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for sftp-user
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
```

---

## 3. Kiểm tra tài khoản

Kiểm tra thông tin user:

```bash
id sftp-user
```

Kết quả mẫu:

```text
uid=1002(sftp-user) gid=1002(sftp-user) groups=1002(sftp-user)
```

Kết quả cho thấy `sftp-user` chỉ thuộc nhóm riêng của tài khoản và **không thuộc nhóm sudo**.

Có thể kiểm tra thêm:

```bash
groups sftp-user
```

Kết quả:

```text
sftp-user : sftp-user
```

---

## 4. Tạo thư mục chứa log

Tạo thư mục:

```bash
sudo mkdir -p /var/log/app-backup/
```

Tạo file log:

```bash
sudo touch /var/log/app-backup/backup-check.log
```

Ghi nội dung log giả lập:

```bash
sudo bash -c 'echo "Backup status: SUCCESS at $(date)" > /var/log/app-backup/backup-check.log'
```

Kiểm tra nội dung:

```bash
sudo cat /var/log/app-backup/backup-check.log
```

Kết quả mẫu:

```text
Backup status: SUCCESS at Wed Oct  7 12:45:31 UTC 2026
```

---

## 5. Cấp quyền cho `sftp-user` đọc file

Gán group của thư mục và file là `sftp-user`:

```bash
sudo chown -R root:sftp-user /var/log/app-backup
```

Cấp quyền truy cập thư mục:

```bash
sudo chmod 750 /var/log/app-backup
```

Cấp quyền đọc file:

```bash
sudo chmod 640 /var/log/app-backup/backup-check.log
```

Kiểm tra:

```bash
ls -ld /var/log/app-backup
ls -l /var/log/app-backup/backup-check.log
```

Kết quả mẫu:

```text
drwxr-x--- 2 root sftp-user 4096 Oct  7 12:45 /var/log/app-backup

-rw-r----- 1 root sftp-user 62 Oct  7 12:45 /var/log/app-backup/backup-check.log
```

Ý nghĩa:

* `root` là owner của file.
* `sftp-user` là group.
* Owner có quyền `rw-`.
* Group có quyền `r--`.
* User ngoài group không có quyền truy cập file.
* Thư mục có quyền `750`, cho phép `sftp-user` truy cập vào thư mục.

---

## 6. Kiểm tra `sftp-user` có đọc được file

Chuyển sang tài khoản `sftp-user`:

```bash
su - sftp-user
```

Thử đọc file:

```bash
cat /var/log/app-backup/backup-check.log
```

Kết quả:

```text
Backup status: SUCCESS at Wed Oct  7 12:45:31 UTC 2026
```

Như vậy `sftp-user` có thể đọc được file log.

Thử kiểm tra quyền sudo:

```bash
sudo -l
```

Kết quả mong đợi:

```text
Sorry, user sftp-user may not run sudo on mayao2.
```

Điều này chứng minh `sftp-user` không có quyền sudo.

Thoát khỏi tài khoản:

```bash
exit
```

---

## 7. Kiểm tra file trên server

Chạy:

```bash
id sftp-user
ls -l /var/log/app-backup/backup-check.log
```

Kết quả mẫu:

```text
uid=1002(sftp-user) gid=1002(sftp-user) groups=1002(sftp-user)

-rw-r----- 1 root sftp-user 62 Oct  7 12:45 /var/log/app-backup/backup-check.log
```

Kiểm tra nội dung file:

```bash
sudo cat /var/log/app-backup/backup-check.log
```

Kết quả:

```text
Backup status: SUCCESS at Wed Oct  7 12:45:31 UTC 2026
```

---

# 8. Kết nối SFTP bằng Windows

Trong bài này sử dụng **Bitvise SSH Client**.

Mở Bitvise SSH Client trên Windows.

Tại phần cấu hình kết nối, nhập:

```text
Host:      IP_PUBLIC_CUA_AZURE_VM
Port:      22
Username:  sftp-user
Method:    Password
```

Nhập mật khẩu đã tạo cho `sftp-user`.

Sau đó nhấn:

```text
Log in
```

Nếu kết nối thành công, mở:

```text
New SFTP Window
```

---

# 9. Truy cập thư mục log trên server

Trong cửa sổ SFTP, tìm đến thư mục:

```text
/var/log/app-backup/
```

Trong phần Remote Files sẽ thấy:

```text
backup-check.log
```

File cần tải về là:

```text
backup-check.log
```

Thực hiện kéo thả file từ:

```text
Remote Files
```

sang:

```text
Local Files
```

để tải file về máy tính Windows.

---

# 10. Kiểm tra file đã tải về Windows

Sau khi tải thành công, mở file:

```text
backup-check.log
```

Nội dung trên Windows phải giống với nội dung trên server:

```text
Backup status: SUCCESS at Wed Oct  7 12:45:31 UTC 2026
```

Như vậy quá trình truyền file qua SFTP đã thành công.

---

# 11. Kiểm tra tổng thể

### Kiểm tra user

```bash
id sftp-user
```

Kết quả:

```text
uid=1002(sftp-user) gid=1002(sftp-user) groups=1002(sftp-user)
```

### Kiểm tra quyền file

```bash
ls -l /var/log/app-backup/backup-check.log
```

Kết quả:

```text
-rw-r----- 1 root sftp-user 62 Oct  7 12:45 /var/log/app-backup/backup-check.log
```

### Kiểm tra thư mục

```bash
ls -ld /var/log/app-backup
```

Kết quả:

```text
drwxr-x--- 2 root sftp-user 4096 Oct  7 12:45 /var/log/app-backup
```

### Kiểm tra nội dung

```bash
sudo cat /var/log/app-backup/backup-check.log
```

Kết quả:

```text
Backup status: SUCCESS at Wed Oct  7 12:45:31 UTC 2026
```

---

# 12. Phân tích quyền

| Thành phần             | Quyền         | Ý nghĩa                                   |
| ---------------------- | ------------- | ----------------------------------------- |
| `/var/log/app-backup/` | `750`         | Owner toàn quyền, group được đọc/truy cập |
| `backup-check.log`     | `640`         | Owner đọc/ghi, group chỉ đọc              |
| Owner                  | `root`        | File được quản lý bởi root                |
| Group                  | `sftp-user`   | Cho phép sftp-user đọc file               |
| `sftp-user`            | Không có sudo | Không được thực hiện lệnh đặc quyền       |

---

# 13. Kết quả đạt được

Sau khi thực hiện bài lab:

* Đã tạo tài khoản `sftp-user`.
* `sftp-user` không thuộc nhóm sudo.
* Đã tạo thư mục `/var/log/app-backup/`.
* Đã tạo file `/var/log/app-backup/backup-check.log`.
* Đã cấp quyền để `sftp-user` đọc file log.
* Đã kiểm tra thành công quyền truy cập.
* Đã kết nối server bằng SFTP Client trên Windows.
* Đã tải file `backup-check.log` từ Ubuntu Server về máy Windows.
* Nội dung file tải về khớp với file trên server.

---

# 14. Cấu trúc nộp bài

Đường dẫn GitHub:

```text
homework/session_07/ex1/
```

Cấu trúc:

```text
homework/
└── session_07/
    └── ex1/
        └── README.md
```

README cần ghi lại quá trình thực hiện và chèn ảnh chụp màn hình giao diện Bitvise SSH Client/WinSCP thể hiện:

1. Đã đăng nhập bằng `sftp-user`.
2. Đã mở SFTP Window.
3. Đã truy cập `/var/log/app-backup/`.
4. Đã nhìn thấy `backup-check.log`.
5. Đã tải file về Windows thành công.

---

# 15. Kết luận

Bài thực hành đã triển khai thành công một tài khoản SFTP giới hạn dành cho việc truy xuất file log. Tài khoản `sftp-user` không có quyền sudo nhưng vẫn có thể đọc file `backup-check.log` thông qua SFTP.

Việc sử dụng SFTP giúp truyền tải dữ liệu giữa máy chủ Linux và máy tính Windows thông qua kết nối SSH được mã hóa, phù hợp cho việc sao lưu và phân tích log từ xa.
