# ecommerce-system-design

Theo mặc định, **WooCommerce API** yêu cầu **HTTPS** để đảm bảo tính bảo mật khi truyền dữ liệu (đặc biệt là các thông tin nhạy cảm như thông tin khách hàng, đơn hàng, và token xác thực). Tuy nhiên, bạn **có thể chạy WooCommerce API mà không cần HTTPS** trong môi trường phát triển (development environment) bằng cách **tắt kiểm tra SSL**.

---

### **1. Tắt kiểm tra SSL trong WooCommerce**
WooCommerce API có một tùy chọn để cho phép sử dụng HTTP thay vì HTTPS trong môi trường phát triển:

- Thêm đoạn code sau vào file `functions.php` của theme đang sử dụng:

```php
add_filter('woocommerce_api_check_ssl', '__return_false');
```

- Đoạn code này sẽ tắt kiểm tra SSL, cho phép truy cập WooCommerce API qua HTTP.

---

### **2. Dùng HTTP Basic Authentication**
Nếu bạn đang sử dụng **HTTP Basic Authentication** để xác thực API, hãy đảm bảo plugin `Basic Authentication` đã được cài đặt và kích hoạt. Bạn có thể tải plugin này tại [GitHub: Basic Authentication for WooCommerce REST API](https://github.com/WP-API/Basic-Auth).

---

### **3. Truy cập API bằng HTTP**
- Sau khi đã tắt kiểm tra SSL, bạn có thể truy cập API qua HTTP bằng các endpoint. Ví dụ:

```
http://yourwebsite.com/wp-json/wc/v3/products
```

Bạn cần cung cấp **key** và **secret** của API thông qua các thông số `consumer_key` và `consumer_secret` trong URL hoặc Basic Authentication.

---

### **4. Lưu ý quan trọng**
- **Không nên sử dụng HTTP trong môi trường sản xuất (production):** Việc này tiềm ẩn nguy cơ bảo mật vì dữ liệu có thể bị đánh cắp khi truyền qua mạng.
- **Giải pháp thay thế an toàn:**
  - Nếu bạn chưa có HTTPS, có thể sử dụng dịch vụ như [Let's Encrypt](https://letsencrypt.org/) để cài đặt chứng chỉ SSL miễn phí.
  - Trong môi trường phát triển, bạn cũng có thể sử dụng **ngrok** để tạo một địa chỉ HTTPS tạm thời.

---

Nếu bạn đang làm việc trong môi trường phát triển, cách này có thể chấp nhận được, nhưng hãy cân nhắc sử dụng HTTPS ngay khi có thể!




-------------------------------------------
## I. Để bật HTTPS trên WAMP (Windows Apache MySQL PHP), bạn cần thực hiện các bước sau:

---

### **1. Tạo hoặc lấy chứng chỉ SSL**
- **Sử dụng OpenSSL để tạo chứng chỉ tự ký (self-signed):**
  1. Đi tới thư mục cài đặt WAMP, thông thường là `C:\wamp64\bin\apache\apacheX.X.X\bin` (với `X.X.X` là phiên bản Apache).
  2. Chạy `cmd` và chuyển đến thư mục trên, sau đó nhập lệnh để tạo chứng chỉ:

     ```bash
     openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt
     ```
     - Nhập các thông tin như `Country`, `State`, `Common Name` (phải giống tên miền bạn muốn dùng, ví dụ: `localhost`).

  3. Lệnh này sẽ tạo hai tệp: `server.key` (khóa riêng) và `server.crt` (chứng chỉ).

---

### **2. Cấu hình Apache để sử dụng HTTPS**
- Mở **file cấu hình Apache** tại:  
  `C:\wamp64\bin\apache\apacheX.X.X\conf\extra\httpd-ssl.conf`.

- Nếu file chưa có, bạn có thể sao chép tệp mẫu:  
  `httpd-ssl.conf.default` và đổi tên thành `httpd-ssl.conf`.

- Chỉnh sửa các thông số sau trong file:
  ```apache
  <VirtualHost _default_:443>
      DocumentRoot "C:/wamp64/www"
      ServerName localhost:443
      SSLEngine on
      SSLCertificateFile "C:/wamp64/bin/apache/apacheX.X.X/conf/server.crt"
      SSLCertificateKeyFile "C:/wamp64/bin/apache/apacheX.X.X/conf/server.key"
  </VirtualHost>
  ```

---

### **3. Kích hoạt module SSL trong Apache**
- Mở file `httpd.conf` tại:  
  `C:\wamp64\bin\apache\apacheX.X.X\conf\httpd.conf`.

- Tìm và bỏ dấu `#` để kích hoạt các module sau:
  ```apache
  LoadModule ssl_module modules/mod_ssl.so
  LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
  Include conf/extra/httpd-ssl.conf
  ```

---

### **4. Khởi động lại WAMP**
- Khởi động lại WAMP để áp dụng cấu hình mới.

---

### **5. Truy cập bằng HTTPS**
- Mở trình duyệt và truy cập:  
  `https://localhost`

- **Lưu ý:** Vì đây là chứng chỉ tự ký, trình duyệt có thể cảnh báo không tin cậy. Bạn cần chọn "Advanced" -> "Proceed to localhost (unsafe)" để tiếp tục.

---

### **Tùy chọn nâng cao (nếu cần):**
- Nếu sử dụng domain tùy chỉnh, hãy cập nhật file `hosts` tại:  
  `C:\Windows\System32\drivers\etc\hosts` để trỏ domain đó tới `127.0.0.1`.

Ví dụ:
```plaintext
127.0.0.1 mysite.local
```

Bạn có thể kiểm tra lại và thử nghiệm! Nếu cần hỗ trợ thêm, mình sẵn sàng giúp!
