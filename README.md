# Bài 1: Thay đổi cổng kết nối SSH (SSH Port Hardening)

## 1. Mục tiêu
Thay đổi cổng dịch vụ SSH mặc định từ 22 sang 2222 trên máy chủ ảo nhằm hạn chế tối đa các cuộc quét cổng tự động của botnet trên mạng internet. Thực hiện cấu hình tường lửa UFW trước khi khởi động lại dịch vụ để tránh nguy cơ bị khóa quyền truy cập (Lockout).

## 2. Các bước thực hiện bằng tài khoản `devops`

### Bước 1: Thay đổi chỉ thị cổng kết nối trong cấu hình SSH
Sử dụng trình soạn thảo `nano` với quyền `sudo` để chỉnh sửa file cấu hình của SSH Daemon:

```bash
sudo nano /etc/ssh/sshd_config
```

Tìm đến dòng cấu hình cổng (thường bị comment là `#Port 22` hoặc đang để là `Port 22`), sửa đổi thành cổng mới:

```text
Port 2222
```

Lưu tệp tin và thoát (Ctrl+O, Enter, Ctrl+X).

### Bước 2: Cấu hình tường lửa UFW mở cổng mới
Để tránh việc tự nhốt mình ở ngoài (Lockout) sau khi thay đổi port, cần mở cổng 2222 TCP trên tường lửa UFW trước khi khởi động lại dịch vụ SSH.

```bash
sudo ufw allow 2222/tcp
```

**Log Terminal:**
```text
Rule added
Rule added (v6)
```

*(Tuỳ chọn) Có thể kiểm tra lại trạng thái UFW để đảm bảo quy tắc đã được thêm vào:*
```bash
sudo ufw status
```

### Bước 3: Khởi động lại dịch vụ SSH
Khởi động lại dịch vụ SSH để áp dụng các thay đổi trong file cấu hình:

```bash
sudo systemctl restart ssh
```
*(Lưu ý: Không tắt cửa sổ Terminal hiện tại cho đến khi đã xác nhận kết nối thành công ở phiên bản mới).*

---

## 3. Kiểm tra kết quả từ máy tính cá nhân

Mở một cửa sổ Terminal mới trên máy tính cá nhân và thực hiện các lệnh kiểm tra sau:

### Kiểm tra kết nối qua cổng mới (2222)
```bash
ssh -p 2222 devops@<IP_ADDRESS_DROPLET>
```

**Kết quả mong đợi:** Đăng nhập thành công vào server.
```text
Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-x-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
...
devops@server:~$ 
```

### Kiểm tra kết nối qua cổng mặc định cũ (22)
```bash
ssh -p 22 devops@<IP_ADDRESS_DROPLET>
```

**Kết quả mong đợi:** Kết nối bị chặn hoặc từ chối hoàn toàn.
```text
ssh: connect to host <IP_ADDRESS_DROPLET> port 22: Connection refused
# hoặc
ssh: connect to host <IP_ADDRESS_DROPLET> port 22: Connection timed out
```
