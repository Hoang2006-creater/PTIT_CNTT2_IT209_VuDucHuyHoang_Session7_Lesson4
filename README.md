Báo cáo Thực hành Bài 4: Cấu hình Reverse Proxy Nginx cho Spring Boot

Họ và tên: [Điền Họ và Tên của bạn]

Mã lớp / Khóa học: [Điền Mã lớp của bạn]

Đường dẫn thư mục nộp bài: homework/session_07/ex4/

1. Kết quả kiểm tra cú pháp cấu hình Nginx (nginx -t)

Thực thi lệnh kiểm tra tính hợp lệ của tệp cấu hình sau khi tạo liên kết mềm (symlink):

$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful


Đánh giá: Tệp cấu hình spring-proxy.conf không có lỗi cú pháp và đã sẵn sàng nạp vào tiến trình Nginx qua lệnh sudo systemctl reload nginx.

2. Kết quả kiểm tra truy cập trang tĩnh (/) bằng lệnh curl

2.1. Kiểm tra HTTP Response Header (curl -I)

$ curl -I http://localhost/
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 07 Oct 2026 12:40:15 GMT
Content-Type: text/html
Content-Length: 642
Last-Modified: Wed, 07 Oct 2026 12:30:00 GMT
Connection: keep-alive
ETag: "66fb10b0-282"
Accept-Ranges: bytes


2.2. Kiểm tra nội dung trang chủ tĩnh (curl http://localhost/)

$ curl http://localhost/
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Thông tin học viên</title>
</head>
<body>
    <h1>Thông tin học viên</h1>
    <p><strong>Họ và tên:</strong> [Điền Họ và Tên của bạn]</p>
    <p><strong>Mã lớp:</strong> [Điền Mã lớp của bạn]</p>
</body>
</html>


Đánh giá: Nginx lắng nghe chính xác trên cổng 80 và phục vụ đúng tệp index.html từ thư mục gốc /var/www/html/ với mã phản hồi 200 OK.

3. Kết quả kiểm tra Reverse Proxy sang Spring Boot Backend (/api/health)

Thực hiện truy vấn qua Nginx Reverse Proxy để chuyển tiếp yêu cầu đến Spring Boot đang chạy ở cổng 8082:

$ curl -i http://localhost/api/health
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 07 Oct 2026 12:41:02 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive

{"status":"UP","components":{"diskSpace":{"status":"UP"},"ping":{"status":"UP"}}}


Đánh giá:

Nginx bắt đúng tiền tố /api/ theo cấu hình location /api/ và chuyển tiếp luồng mạng đến http://127.0.0.1:8082.

Kết quả trả về mã trạng thái HTTP 200 OK cùng dữ liệu định dạng JSON từ Spring Boot Actuator.

Không phát sinh lỗi 502 Bad Gateway hay 404 Not Found.

4. Tóm tắt các lệnh đã thực thi trên hệ thống

# 1. Tạo liên kết mềm kích hoạt cấu hình
sudo ln -sf /etc/nginx/sites-available/spring-proxy.conf /etc/nginx/sites-enabled/

# 2. Vô hiệu hóa cấu hình mặc định (tránh xung đột cổng 80)
sudo rm -f /etc/nginx/sites-enabled/default

# 3. Kiểm tra và tải lại cấu hình
sudo nginx -t
sudo systemctl reload nginx
