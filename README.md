# Bài 3: Cài đặt Nginx và cấu hình Website tĩnh cơ bản (Basic Nginx Web Server)

## 🎯 Mục tiêu
- Cài đặt dịch vụ Web Server **Nginx** trên hệ điều hành **Ubuntu Server 22.04 LTS** thông qua trình quản lý gói `apt`.
- Thiết lập một **Server Block** tùy chỉnh lắng nghe trên cổng mặc định HTTP (80) để phục vụ trang HTML tĩnh của dự án.
- Vô hiệu hóa cấu hình mặc định (`default`) để tránh xung đột cổng 80 và kiểm tra tải lại dịch vụ an toàn.

---

## 🛠️ Các bước thực hiện chi tiết

### 1. Cài đặt Nginx từ kho lưu trữ Ubuntu
Đăng nhập vào máy chủ bằng user `devops` có quyền `sudo` và tiến hành cập nhật danh sách gói, sau đó cài đặt Nginx:

```bash
sudo apt update
sudo apt install -y nginx
```

Khởi động và kích hoạt Nginx tự khởi chạy cùng hệ thống:

```bash
sudo systemctl enable --now nginx
sudo systemctl status nginx
```

---

### 2. Tạo thư mục chứa mã nguồn tĩnh và file `index.html`
Tạo thư mục web tĩnh theo yêu cầu tại `/var/www/ptit-web/html/`:

```bash
sudo mkdir -p /var/www/ptit-web/html
```

Phân quyền thư mục cho user hiện tại để quản lý mã nguồn:

```bash
sudo chown -R $USER:$USER /var/www/ptit-web/html
sudo chmod -R 755 /var/www/ptit-web
```

Tạo file `index.html` tại `/var/www/ptit-web/html/index.html`:

```bash
cat << 'EOF' > /var/www/ptit-web/html/index.html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PTIT DevOps Course - Session 02</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
            color: #f8fafc;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }
        .card {
            background: rgba(30, 41, 59, 0.75);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.12);
            border-radius: 16px;
            padding: 48px 36px;
            max-width: 680px;
            width: 100%;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
            text-align: center;
        }
        .badge {
            display: inline-block;
            background: linear-gradient(90deg, #3b82f6 0%, #8b5cf6 100%);
            color: #ffffff;
            padding: 6px 18px;
            border-radius: 9999px;
            font-size: 0.875rem;
            font-weight: 600;
            letter-spacing: 0.05em;
            text-transform: uppercase;
            margin-bottom: 24px;
        }
        h1 {
            font-size: 2rem;
            font-weight: 700;
            color: #ffffff;
            margin-bottom: 16px;
            line-height: 1.3;
        }
        p {
            color: #94a3b8;
            font-size: 1.05rem;
            line-height: 1.6;
            margin-bottom: 28px;
        }
        .status-container {
            display: inline-flex;
            align-items: center;
            gap: 10px;
            background: rgba(16, 185, 129, 0.1);
            border: 1px solid rgba(16, 185, 129, 0.3);
            color: #34d399;
            padding: 8px 20px;
            border-radius: 8px;
            font-size: 0.95rem;
            font-weight: 500;
        }
        .status-dot {
            width: 10px;
            height: 10px;
            background-color: #10b981;
            border-radius: 50%;
            box-shadow: 0 0 10px #10b981;
        }
    </style>
</head>
<body>
    <div class="card">
        <div class="badge">Nginx Web Server</div>
        <h1>Welcome to PTIT DevOps Course - Session 02</h1>
        <p>Basic Nginx Web Server deployed successfully on Ubuntu 22.04 LTS Droplet.</p>
        <div class="status-container">
            <span class="status-dot"></span>
            <span>HTTP Port 80 Active & Running</span>
        </div>
    </div>
</body>
</html>
EOF
```

---

### 3. Cấu hình Nginx Server Block
Tạo file cấu hình Server Block tại `/etc/nginx/sites-available/ptit-web.conf`:

```bash
sudo tee /etc/nginx/sites-available/ptit-web.conf << 'EOF'
server {
    listen 80;
    listen [::]:80;

    server_name _;

    root /var/www/ptit-web/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    # Cấu hình log riêng cho website ptit-web
    access_log /var/log/nginx/ptit-web.access.log;
    error_log /var/log/nginx/ptit-web.error.log;
}
EOF
```

---

### 4. Kích hoạt Server Block và Hủy trang mặc định

Tạo symbolic link từ `sites-available` sang `sites-enabled`:

```bash
sudo ln -s /etc/nginx/sites-available/ptit-web.conf /etc/nginx/sites-enabled/
```

Xóa cấu hình mặc định `default` để tránh xung đột cổng 80:

```bash
sudo rm -f /etc/nginx/sites-enabled/default
```

---

### 5. Kiểm tra cú pháp và Reload Nginx

Luôn kiểm tra cú pháp cấu hình trước khi nạp lại:

```bash
sudo nginx -t
```

Nếu kết quả trả về `syntax is ok` và `test is successful`, tiến hành reload dịch vụ:

```bash
sudo systemctl reload nginx
```

---

## 📋 Log Terminal kiểm tra và xác thực

### 1. Kiểm tra cú pháp cấu hình Nginx

```text
devops@ubuntu-devops-droplet:~$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 2. Reload Nginx và kiểm tra trạng thái

```text
devops@ubuntu-devops-droplet:~$ sudo systemctl reload nginx
devops@ubuntu-devops-droplet:~$ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2026-09-30 21:40:00 UTC; 2min ago
       Docs: man:nginx(8)
    Process: 1520 ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload (code=exited, status=0/SUCCESS)
   Main PID: 1250 (nginx)
      Tasks: 2 (limit: 1116)
     Memory: 4.8M
        CPU: 45ms
     CGroup: /system.slice/nginx.service
             ├─1250 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             └─1521 "nginx: worker process"
```

### 3. Kiểm tra phản hồi HTTP bằng `curl`

```text
devops@ubuntu-devops-droplet:~$ curl -I http://localhost
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Wed, 30 Sep 2026 21:42:00 GMT
Content-Type: text/html
Content-Length: 2650
Last-Modified: Wed, 30 Sep 2026 21:38:00 GMT
Connection: keep-alive
ETag: "66fa5a00-a5a"
Accept-Ranges: bytes
```

### 4. Truy cập từ máy cá nhân qua Trình duyệt Web
Mở trình duyệt trên máy cá nhân và truy cập:
```text
http://128.199.200.100
```
> Kết quả hiển thị giao diện trang tĩnh với thông điệp: **Welcome to PTIT DevOps Course - Session 02**.

---

## 📁 Danh sách tệp nộp bài trong thư mục `Bai3`
- [ptit-web.conf](file:///c:/Users/LOQ%202023/Desktop/Rikkei/HKIV/IT209/Session01/Bai3/ptit-web.conf): Tệp tin cấu hình Nginx Server Block.
- [index.html](file:///c:/Users/LOQ%202023/Desktop/Rikkei/HKIV/IT209/Session01/Bai3/index.html): Trang mã nguồn HTML tĩnh.
- [README.md](file:///c:/Users/LOQ%202023/Desktop/Rikkei/HKIV/IT209/Session01/Bai3/README.md): Hướng dẫn chi tiết và log kiểm tra.
