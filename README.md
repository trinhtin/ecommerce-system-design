# ecommerce-system-design

# Phần 1: Tắt SSL trong môi trường phát triển

Theo mặc định, **WooCommerce API** yêu cầu **HTTPS** để đảm bảo tính bảo mật khi truyền dữ liệu (đặc biệt là các thông tin nhạy cảm như thông tin khách hàng, đơn hàng, và token xác thực). Tuy nhiên, bạn **có thể chạy WooCommerce API mà không cần HTTPS** trong môi trường phát triển (development environment) bằng cách **tắt kiểm tra SSL**.

## **Cách 1: Tắt kiểm tra SSL trong WooCommerce**
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
## Cách 2: Để bật HTTPS trên WAMP (Windows Apache MySQL PHP), bạn cần thực hiện các bước sau:

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

-----------------
# Phần 2: Hướng dẫn các thao tác cơ bản để vận hành store thông qua API
Quản trị WooCommerce shop thông qua API giúp tự động hóa nhiều tác vụ, tiết kiệm thời gian và giảm thiểu lỗi khi quản lý cửa hàng trực tuyến. Dưới đây là hướng dẫn các tác vụ cơ bản mà bạn có thể thực hiện qua **WooCommerce REST API**.

---

### **1. Kết nối với WooCommerce API**
Đầu tiên, bạn cần cài đặt và kích hoạt **REST API**:
- Đăng nhập WordPress Admin > WooCommerce > Settings > **Advanced** > REST API.
- Tạo một **API Key**:
  - Chọn quyền truy cập (Read, Write, hoặc Read/Write).
  - Lưu `Consumer Key` và `Consumer Secret`.

---

### **2. Cấu hình HTTP Client**
Bạn có thể sử dụng các công cụ để kết nối với API:
- **Postman:** Công cụ phổ biến để kiểm tra API.
- **Ngôn ngữ lập trình:** Dùng thư viện như `requests` trong Python hoặc `axios` trong JavaScript.

**Base URL** của WooCommerce API sẽ là:  
```
https://yourstore.com/wp-json/wc/v3/
```

Xác thực bằng:
- **Query string:** Thêm `consumer_key` và `consumer_secret` vào URL.
- **HTTP Basic Auth:** Gửi key/secret qua header.

---

### **3. Các task quản trị cơ bản qua WooCommerce API**

#### **3.1. Quản lý sản phẩm**
**a. Lấy danh sách sản phẩm**  
Endpoint: `GET /products`

Ví dụ:
```bash
GET https://yourstore.com/wp-json/wc/v3/products?consumer_key=ck_xxx&consumer_secret=cs_xxx
```

**b. Tạo sản phẩm mới**  
Endpoint: `POST /products`  
Dữ liệu mẫu:
```json
{
  "name": "Áo Thun",
  "type": "simple",
  "regular_price": "250000",
  "description": "Áo thun chất liệu cotton cao cấp.",
  "images": [
    {
      "src": "https://example.com/image.jpg"
    }
  ]
}
```

**c. Cập nhật sản phẩm**  
Endpoint: `PUT /products/{id}`  
Dữ liệu mẫu:
```json
{
  "regular_price": "200000",
  "stock_quantity": 10
}
```

**d. Xóa sản phẩm**  
Endpoint: `DELETE /products/{id}`  
Ví dụ:
```bash
DELETE https://yourstore.com/wp-json/wc/v3/products/123?consumer_key=ck_xxx&consumer_secret=cs_xxx
```

---

#### **3.2. Quản lý đơn hàng**
**a. Lấy danh sách đơn hàng**  
Endpoint: `GET /orders`  
Lọc theo trạng thái:
```bash
GET https://yourstore.com/wp-json/wc/v3/orders?status=completed&consumer_key=ck_xxx&consumer_secret=cs_xxx
```

**b. Tạo đơn hàng mới**  
Endpoint: `POST /orders`  
Dữ liệu mẫu:
```json
{
  "payment_method": "cod",
  "payment_method_title": "Cash on Delivery",
  "billing": {
    "first_name": "Nguyen",
    "last_name": "Van A",
    "address_1": "123 Đường ABC",
    "city": "Hà Nội",
    "email": "email@example.com",
    "phone": "0123456789"
  },
  "line_items": [
    {
      "product_id": 123,
      "quantity": 2
    }
  ]
}
```

**c. Cập nhật trạng thái đơn hàng**  
Endpoint: `PUT /orders/{id}`  
Dữ liệu mẫu:
```json
{
  "status": "completed"
}
```

**d. Xóa đơn hàng**  
Endpoint: `DELETE /orders/{id}`  
Ví dụ:
```bash
DELETE https://yourstore.com/wp-json/wc/v3/orders/456?force=true&consumer_key=ck_xxx&consumer_secret=cs_xxx
```

---

#### **3.3. Quản lý khách hàng**
**a. Lấy danh sách khách hàng**  
Endpoint: `GET /customers`

**b. Tạo khách hàng mới**  
Endpoint: `POST /customers`  
Dữ liệu mẫu:
```json
{
  "email": "email@example.com",
  "first_name": "Nguyen",
  "last_name": "Van A",
  "username": "nguyenvana",
  "billing": {
    "address_1": "123 Đường ABC",
    "city": "Hà Nội",
    "postcode": "100000",
    "country": "VN"
  }
}
```

**c. Cập nhật thông tin khách hàng**  
Endpoint: `PUT /customers/{id}`  
Dữ liệu mẫu:
```json
{
  "billing": {
    "address_1": "456 Đường XYZ"
  }
}
```

**d. Xóa khách hàng**  
Endpoint: `DELETE /customers/{id}`

---

#### **3.4. Quản lý danh mục**
**a. Lấy danh sách danh mục**  
Endpoint: `GET /products/categories`

**b. Tạo danh mục mới**  
Endpoint: `POST /products/categories`  
Dữ liệu mẫu:
```json
{
  "name": "Áo Quần",
  "slug": "ao-quan"
}
```

**c. Cập nhật danh mục**  
Endpoint: `PUT /products/categories/{id}`  
Ví dụ:
```json
{
  "name": "Thời Trang Nam"
}
```

**d. Xóa danh mục**  
Endpoint: `DELETE /products/categories/{id}`

---

### **4. Thực hành tốt**
1. **Xác thực API an toàn:**  
   - Sử dụng HTTPS để bảo mật dữ liệu.
   - Giới hạn quyền truy cập cho API Key theo nhu cầu (Read, Write).

2. **Thử nghiệm trước khi áp dụng:**  
   Dùng **Postman** để kiểm tra endpoint trước khi tích hợp vào hệ thống.

3. **Xử lý lỗi API:**  
   Khi API trả về lỗi, thường có mã trạng thái HTTP và thông báo chi tiết (ví dụ: `400 Bad Request`, `401 Unauthorized`).

---

Nếu bạn cần trợ giúp cụ thể trong việc triển khai API WooCommerce, hãy cho mình biết nhé!
