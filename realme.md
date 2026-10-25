# Fashion Shop — tài liệu dự án

Tài liệu này tập hợp hướng dẫn cài đặt, chạy, kiểm thử và các ghi chú vận hành của Fashion Shop. Các file attribution ảnh và tài liệu đi kèm thư viện bên thứ ba vẫn được giữ riêng vì cần thiết cho nguồn gốc và quyền sử dụng.

## Thông tin đề tài

- Tên đề tài: Hệ thống website thương mại điện tử thời trang Fashion.
- Nhóm: Nhóm 5.
- Thành viên:
  - Nguyễn Mạnh Quyền — 23010198 — Leader
  - Nguyễn Minh Nguyệt — 23010408
  - Nguyễn Hữu Huy — 23010240
  - Nguyễn Trọng Dương — 23010227
- URL triển khai thử: chưa được cung cấp.

## 1. Công nghệ và yêu cầu

| Thành phần | Phiên bản / cổng mặc định | Mục đích |
| --- | --- | --- |
| Java JDK | 21 | Backend Spring Boot |
| Maven | 3.9+ hoặc Maven Wrapper | Build và test backend |
| Node.js / npm | Node.js 20+ | Frontend React/Vite |
| Docker Desktop / Compose | Bản ổn định hiện hành | PostgreSQL, Redis, Elasticsearch, RabbitMQ |
| Git | Bản ổn định hiện hành | Quản lý mã nguồn |
| Backend | `8081` | API Spring Boot |
| Frontend | `5173` | Vite dev server |
| PostgreSQL | `5432` | Cơ sở dữ liệu |
| Redis | `6379` | Cache |
| Elasticsearch | `9200` | Tìm kiếm |
| RabbitMQ | `5672`; giao diện quản trị `15672` | Messaging |

Đảm bảo Docker Desktop đang chạy và các cổng trên không bị ứng dụng khác chiếm dụng. Lần đầu chạy Maven hoặc npm có thể cần internet để tải dependency.

## 2. Cài đặt và chạy toàn bộ ứng dụng

Mở PowerShell tại thư mục gốc dự án.

### 2.1 Khởi động các dịch vụ phụ trợ

```powershell
docker compose up -d --wait
docker compose ps
```

Compose khởi động PostgreSQL, Redis, Elasticsearch và RabbitMQ, đồng thời đợi health check. Để dừng container nhưng giữ dữ liệu trong volume:

```powershell
docker compose down
```

### 2.2 Cấu hình tùy chọn

Nếu cần cấu hình SMTP cho đăng ký bằng email OTP, tạo `.env` từ mẫu rồi thay toàn bộ giá trị mẫu bằng cấu hình riêng của bạn:

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
notepad .env
```

Không commit `.env`, mật khẩu SMTP, JWT secret, khóa VNPay hoặc API key. `.env` được Spring Boot và launcher backend đọc khi chạy từ thư mục gốc. Không cần SMTP thì các tính năng gửi OTP sẽ không hoạt động.

### 2.3 Chạy backend

Mở terminal thứ nhất tại thư mục gốc:

```powershell
.\scripts\run-backend-current.ps1
```

Hoặc:

```powershell
.\mvnw.cmd spring-boot:run
```

Backend mặc định ở `http://localhost:8081`. Có thể đổi cổng bằng biến môi trường `PORT`.

Tài khoản demo local (không dùng trên production):

| Vai trò | Email | Mật khẩu |
| --- | --- | --- |
| Admin | `admin@fashion.local` | `Admin@123` |
| Customer | `customer@fashion.local` | `Customer@123` |

### 2.4 Chạy frontend

Mở terminal thứ hai:

```powershell
Set-Location .\frontend
npm install
npm run dev
```

Mở `http://localhost:5173`. Không đóng hai terminal backend/frontend khi đang sử dụng ứng dụng.

## 3. Cấu hình môi trường thường dùng

Các giá trị mặc định phù hợp cho demo local; triển khai thật cần dùng secret và dịch vụ do bạn quản lý.

| Biến | Công dụng |
| --- | --- |
| `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD` | Kết nối PostgreSQL |
| `PORT` | Cổng backend; mặc định `8081` |
| `FASHION_ADMIN_PASSWORD`, `FASHION_CUSTOMER_PASSWORD` | Mật khẩu tài khoản demo |
| `FASHION_JWT_SECRET` | Khóa ký JWT |
| `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD` | SMTP gửi OTP; mặc định host Gmail và port `587` |
| `GROQ_API_KEY`, `GROQ_MODEL` | Tùy chọn cho chatbot stylist |
| `VNPAY_TMN_CODE`, `VNPAY_HASH_SECRET`, `VNPAY_RETURN_URL`, `VNPAY_FRONTEND_RETURN_URL` | Cấu hình thanh toán VNPay |
| `GEOCODING_PROVIDER`, `GEOCODING_URL`, `ROUTING_URL`, `MAPS_USER_AGENT` | Tìm địa chỉ và tính tuyến đường |
| `SHIPPING_SIMULATOR_ENABLED` | Bật/tắt mô phỏng vận chuyển |
| `VITE_CARTO_API_KEY` | Khóa bản đồ frontend nếu cấu hình sử dụng |

### Đăng ký bằng email OTP

Người đăng ký nhận mã OTP sáu chữ số; tài khoản ở trạng thái chờ đến khi xác thực mã. Mã hết hạn sau 10 phút, có thể gửi lại sau 60 giây và bị vô hiệu sau năm lần nhập sai. Cấu hình SMTP authenticated STARTTLS trên port 587 bằng `MAIL_*`. Không có thông tin SMTP thì đăng ký không tạo tài khoản có thể sử dụng.

## 4. Kiểm thử

### 4.1 Backend — Maven/JUnit

```powershell
.\mvnw.cmd test
```

Chạy một lớp test:

```powershell
.\mvnw.cmd "-Dtest=VoucherServiceUnitTest" test
```

Test dùng profile `test` và H2 trong bộ nhớ theo `src/test/resources/application-test.properties`; không cần PostgreSQL. Báo cáo ở `target/surefire-reports`.

Kết quả được ghi nhận trong tài liệu cũ: 88 tests, 0 failures, 0 errors, 0 skipped. Đây là kết quả lịch sử; chạy lại lệnh trên để xác nhận.

### 4.2 Frontend — lint và build

```powershell
Set-Location .\frontend
npm run lint
npm run build
```

### 4.3 Playwright — functional E2E

Cần PostgreSQL và backend đang chạy. Playwright có thể tự khởi động Vite nếu chưa có server tại `localhost:5173`. Từ `frontend`:

```powershell
npx.cmd playwright --version
npx.cmd playwright test --reporter=list
npx.cmd playwright show-report
```

Trên Windows dùng `npx.cmd` để tránh lỗi execution policy của `npx.ps1`. Bộ test bao phủ auth, sản phẩm, tìm kiếm, giỏ hàng, voucher, checkout, thanh toán, đơn hàng, giao hàng, đánh giá, wishlist, smart size và admin. Test admin cần thông tin đăng nhập:

```powershell
$env:E2E_ADMIN_EMAIL="admin@fashion.local"
$env:E2E_ADMIN_PASSWORD="<admin-password>"
npx.cmd playwright test tests/admin.spec.js
```

Lần chạy được ghi nhận trước đây: 41 case pass, 4 case admin bị skip vì chưa cấu hình `E2E_ADMIN_PASSWORD`; runner sau đó bị kẹt ở bước cleanup/report. Đây là kết quả lịch sử, không đảm bảo trạng thái hiện tại.

### 4.4 Kiểm thử bảo mật

Chạy regression test:

```powershell
.\mvnw.cmd "-Dtest=SecurityIntegrationTest" test
```

Kết quả được ghi nhận trước đây: 7 passed, 0 failed, 0 skipped; hãy chạy lại để xác nhận trên checkout hiện tại.

Để quét OWASP ZAP, khởi động backend và frontend, mở ứng dụng ZAP desktop, chọn **Automated Scan**, rồi quét `http://localhost:5173`. Đánh giá từng alert dựa trên request/response và mã nguồn; phân loại true positive, false positive hoặc accepted risk thay vì mặc định mọi alert đều là lỗi.

Checklist cần xem xét:

| ID | Hạng mục | Hiện trạng / kiểm tra |
| --- | --- | --- |
| SEC-01 | HTTPS | Local dùng HTTP; production cần TLS tại hosting/reverse proxy. Backend có HSTS khi truy cập qua HTTPS. |
| SEC-02 | Password hash | Có JUnit xác nhận mật khẩu lưu bằng BCrypt, không lưu plain text. |
| SEC-03 | SQL injection | Kiểm tra bằng ZAP và input thử; luồng chính dùng Spring Data/repository parameter binding. |
| SEC-04 | XSS | Kiểm tra ZAP và UI; chú ý review, search, profile. React escape text mặc định. |
| SEC-05 | CSP | Có JUnit kiểm tra `Content-Security-Policy`, `default-src 'self'` và `frame-ancestors 'none'`. |
| SEC-06 | CSRF | API dùng JWT bearer stateless thay vì session-cookie authentication. |
| SEC-07 | RBAC | JUnit xác nhận CUSTOMER không gọi được `POST /api/admin/products` (403). |
| SEC-08 | IDOR | JUnit xác nhận khách không đọc được đơn của khách khác. |
| SEC-09/10 | JWT sai/hết hạn | JUnit xác nhận token không hợp lệ hoặc hết hạn không truy cập được API bảo vệ. |
| SEC-11 | Login rate limit | Chưa có rate limit trong code được ghi nhận; cân nhắc giới hạn theo email/IP ở app hoặc gateway. |
| SEC-12/13 | Upload type/size | Khi mở upload thật, whitelist MIME/type và xác thực kích thước file/request. |
| SEC-14 | Audit giao dịch | Có `OrderStatusHistory`; cần thêm actor/hành động thanh toán nếu cần audit đầy đủ. |
| SEC-15 | Dữ liệu thanh toán | Kiểm tra runtime log/response không lộ secret hoặc dữ liệu nhạy cảm. |
| SEC-16 | Backup/restore | Không tìm thấy `backup.sql` trong checkout hiện tại; cần tạo backup PostgreSQL và thử restore vào database riêng. |

Các case kiểm tra quyền tiêu biểu:

```text
CUSTOMER token -> POST /api/admin/products -> 403 Forbidden
Customer A token -> GET /api/orders/{orderId cua Customer B} -> khong tra du lieu cua B
```

Ghi chú test trước đây: truy cập đơn của khách khác trả `400 Bad Request` với thông báo đơn không tồn tại theo ngữ cảnh người dùng; điều này không làm lộ dữ liệu đơn hàng.

### 4.5 Kiểm thử hiệu năng bằng JMeter

Chỉ chạy sau khi functional/security regression ổn định; không tạo tải lớn nếu không có yêu cầu NFR. Test plan: `performance/fashion-performance.jmx`. Các request chính là login, danh sách sản phẩm và search. Backend mặc định hiện chạy ở port `8081`; truyền `-Jport=8081` nếu test plan đang dùng cổng khác. Tài khoản demo được ghi nhận trong cấu hình khởi tạo là `customer@fashion.local` / `Customer@123`; chỉ dùng ở local demo.

Chạy non-GUI:

```powershell
jmeter -n -t performance/fashion-performance.jmx -l performance/results-10u.jtl -e -o performance/report-10u -Jhost=localhost -Jport=8081 -Jthreads=10 -Jramp=10 -Jloops=5
jmeter -n -t performance/fashion-performance.jmx -l performance/results-50u.jtl -e -o performance/report-50u -Jhost=localhost -Jport=8081 -Jthreads=50 -Jramp=50 -Jloops=5
jmeter -n -t performance/fashion-performance.jmx -l performance/results-100u.jtl -e -o performance/report-100u -Jhost=localhost -Jport=8081 -Jthreads=100 -Jramp=100 -Jloops=5
```

Ghi lại samples, average, median, P90, P95, min/max, throughput và error rate. Điều tra nếu lỗi tăng, P95 tăng đột biến hoặc throughput không tăng theo số người dùng; không kết luận lỗi từ một spike đơn lẻ. Xác nhận NFR trước khi đánh giá đạt/không đạt.

## 5. Đơn hàng, giao hàng và bản đồ

### Luồng demo

1. Customer đặt hàng; đơn có trạng thái `PENDING`.
2. Admin xác nhận đơn (`CONFIRMED`), chuẩn bị hàng (`PROCESSING`), rồi tạo vận đơn với đơn vị, cân nặng, phí và ngày giao dự kiến tùy chọn.
3. Admin bàn giao hàng (`PICKED_UP`). Simulator chạy mỗi 5 giây và mô phỏng `IN_TRANSIT` → `OUT_FOR_DELIVERY` → `DELIVERED`.
4. Customer theo dõi tuyến và timeline, sau khi nhận hàng xác nhận để chuyển đơn sang `COMPLETED` và đủ điều kiện đánh giá.
5. Có thể mô phỏng giao thất bại (bắt buộc lý do), giao lại hoặc hoàn hàng (`RETURNED`).

Phí vận đơn không thay đổi số tiền checkout đã chốt. `RETURNED` chỉ ghi nhận kết quả hoàn hàng; hoàn tiền, kiểm đếm và nhập kho cần nghiệp vụ riêng. GHN, GHTK, Viettel Post là đơn vị vận chuyển, không phải role người dùng. Mỗi đơn có tối đa một vận đơn.

### API giao hàng

| Phương thức | Đường dẫn | Quyền |
| --- | --- | --- |
| `GET` | `/api/admin/shipping-providers` | Admin |
| `GET` / `POST` | `/api/admin/orders/{id}/shipment` | Admin: xem / tạo |
| `PATCH` | `/api/admin/orders/{id}/shipment/simulate` | Admin: mô phỏng phản hồi |
| `GET` | `/api/orders/{id}/shipment` | Customer sở hữu đơn |
| `PATCH` | `/api/orders/{id}/receive` | Customer sở hữu đơn đã giao |

Ví dụ tạo vận đơn: `{"providerId":1,"weightKg":1.2,"shippingFee":30000,"estimatedDelivery":"2026-09-25"}`. Ví dụ cập nhật: `{"status":"DELIVERY_FAILED","description":"Không liên hệ được người nhận","location":"Hà Nội"}`.

`ShippingGateway` là điểm tích hợp adapter hãng vận chuyển thật. Hiện chưa có API hãng thật hoặc webhook công khai; webhook sau này phải xác thực nguồn trước khi cập nhật nghiệp vụ. Tạo/cập nhật vận đơn khóa đơn hàng để tránh ghi đồng thời; cập nhật lặp không thêm event.

### Tuyến đường và geocoding

- Checkout: tìm địa điểm, chọn kết quả, kéo bản đồ dưới ghim cố định và xác nhận địa chỉ. Khi sửa địa chỉ/di chuyển bản đồ, tọa độ xác nhận cũ bị xóa.
- `GET /api/location/search?q=...`: tìm địa chỉ tại Việt Nam; cần đăng nhập; độ dài truy vấn 3–200 ký tự.
- `GET /api/location/reverse?lat=...&lng=...`: reverse geocoding và giữ tọa độ người dùng đã chọn.
- Tạo vận đơn lấy hình học tuyến OSRM một lần và lưu vào `shipment_route_points`. Nếu không có tuyến hợp lệ thì transaction rollback, không thay bằng đường thẳng.
- Simulator tiến mỗi tick một điểm; `IN_TRANSIT` (10%), `OUT_FOR_DELIVERY` (80%), `DELIVERED` (100%) là mốc theo chỉ số điểm, không phải thời gian thực. Thất bại/hoàn hàng không tự chạy. Vận đơn cũ thiếu tuyến chỉ lấy tuyến khi Admin yêu cầu cập nhật trạng thái tiếp theo.
- Vị trí xe là mô phỏng theo tuyến, không phải GPS thật. Cần restart backend để chạy migration tạo bảng tuyến.

| Biến | Mặc định | Ghi chú |
| --- | --- | --- |
| `GEOCODING_PROVIDER` | `photon` | `photon` hoặc `nominatim` |
| `GEOCODING_URL` | `https://photon.komoot.io` | URL của provider đã chọn |
| `ROUTING_URL` | `https://router.project-osrm.org` | OSRM hoặc máy chủ OSRM tự triển khai |
| `MAPS_USER_AGENT` | `FashionShopDemo/1.0` | Production nên cung cấp tên app và thông tin liên hệ |
| `SHIPPING_SIMULATOR_ENABLED` | `true` | Tắt khi tích hợp hãng thật |

Photon là geocoder mặc định. Đổi sang Nominatim cần đặt `GEOCODING_PROVIDER=nominatim` và `GEOCODING_URL=https://nominatim.openstreetmap.org`, sau đó restart backend. Dịch vụ bản đồ công cộng chỉ phù hợp demo lưu lượng thấp; không gửi tên/số điện thoại người nhận vào ô tìm địa điểm. App cache tối đa 1.000 kết quả trong 24 giờ, tuần tự hóa và giới hạn 1 request/1,1 giây trên một tiến trình. Reverse geocoding debounce 750 ms. Khi mở rộng nhiều instance, dùng provider hoặc rate limiter dùng chung phù hợp.

Tuân thủ [Nominatim usage policy](https://operations.osmfoundation.org/policies/nominatim/), [OSM tile usage policy](https://operations.osmfoundation.org/policies/tiles/) và [OSRM API](https://project-osrm.org/docs/v5.24.0/api/). Tọa độ gửi nhà cung cấp routing theo thứ tự longitude, latitude.

Kiểm thử liên quan:

```powershell
.\mvnw.cmd "-Dtest=ShipmentFlowTest,AdminOrderServiceTest,OrderServiceTest,CheckoutServiceTest,OrderReceiptTest,ReviewEligibilityTest,MapServicesTest" test
Set-Location .\frontend
npm run build
npm run lint
```

`scripts/system-smoke-test.mjs` có thể kiểm tra luồng vận đơn; cần hệ thống demo đang chạy. `MapServicesTest` dùng provider giả lập, không gọi dịch vụ bản đồ công cộng.

## 6. Dữ liệu demo và database

### Sinh dữ liệu demo

Các generator thêm dữ liệu xác định, không xóa row hiện có. Cài dependency:

```powershell
python -m pip install -r database/requirements.txt
```

Chạy toàn bộ hoặc từng bước:

```powershell
python database/generate_all.py
python database/generate_collections.py
python database/generate_products.py
python database/generate_users.py
python database/generate_reviews.py
```

Mục tiêu tối thiểu: 50 collections, 3.000 products, 15.000 variants, 10 users và 30.000 reviews. Có thể cấu hình `FASHION_DB_HOST`, `FASHION_DB_PORT`, `FASHION_DB_NAME`, `FASHION_DB_USER`, `FASHION_DB_PASSWORD`. `generate_reviews.py` cần users và products có sẵn; để đăng nhập bằng user sinh tự động, cung cấp bcrypt hash hợp lệ qua `FASHION_DEMO_PASSWORD_HASH`.

### Dataset mẫu 1.000 sản phẩm

Có thể dựng lại subset demo từ archive:

```powershell
python database/build_myntra_demo_seed.py E:\archive.zip
```

Script tạo `database/seed_myntra_demo.sql`, cập nhật khối được đánh dấu trong `database/seed.sql` và sao chép ảnh subset vào frontend. Các file output này hiện không có trong checkout cho đến khi chạy script. Seed có thể chạy lặp; không xóa row hiện có. Sau khi schema và migrations sẵn sàng, có thể áp dụng riêng file output:

```powershell
psql -U fashion -d fashion -v ON_ERROR_STOP=1 -f database/seed_myntra_demo.sql
```

Seed tạo danh mục riêng và brand kỹ thuật `Dataset Sample`. Giá, tồn kho và size là dữ liệu demo tổng hợp, không phải chào bán thật. Ảnh nguồn chỉ 60×80 px, không phù hợp ảnh sản phẩm production. Xem attribution bắt buộc tại [Myntra sample attribution](frontend/public/assets/products/myntra-sample/ATTRIBUTION.md).

### Tình trạng ảnh catalog Việt

Ba đường dẫn trong `database/seed_vietnamese_catalog.sql` đang thiếu file local:

| ID | Slug | Đường dẫn dự kiến |
| --- | --- | --- |
| 984 | `viet-phuc-ao-tac-nu-demo` | `/assets/products/vietnamese-traditional/ao-tac/viet-phuc-ao-tac-nu-demo/main.webp` |
| 997 | `ao-ba-ba-nu-demo` | `/assets/products/vietnamese-traditional/ao-ba-ba/ao-ba-ba-nu-demo/main.webp` |
| 990 | `ao-dai-nu-co-dien-demo` | `/assets/products/vietnamese-traditional/ao-dai/ao-dai-nu-co-dien-demo/main.webp` |

Downloader tham chiếu các ảnh Wikimedia theo giấy phép CC BY-SA 4.0/CC BY 2.0; outbound HTTP trước đây bị chặn nên chưa xác minh lại trang nguồn hoặc nội dung ảnh. Chưa có ảnh thay thế được chọn. Không bỏ qua kiểm tra integrity; trước khi dùng, chạy `python database/download_vietnamese_catalog_images.py`, kiểm tra ảnh/manifest và xác minh quyền sử dụng.

### Ghi chú archive ảnh Myntra

Archive được kiểm tra có 44.446 dòng metadata, 44.446 ID riêng, hai bản thư mục ảnh gần như trùng nhau và ảnh thumbnail 60×80. ZIP không kèm README/license/attribution; nguồn dữ liệu mô tả sản phẩm thời trang Ấn Độ, không có giá/tồn kho/variant đáng tin cậy cho catalog Fashion Shop. Kaggle hiển thị MIT cho dataset bản nhỏ, nhưng Myntra có điều khoản bảo vệ nội dung và license của dataset do người đăng khai báo; chưa có căn cứ xác nhận quyền thương mại của từng ảnh.

Chỉ dùng subset demo đã đánh dấu kèm attribution theo yêu cầu dự án; không xem đây là cleared cho storefront thương mại. Trước khi triển khai thương mại, xác nhận quyền ảnh/nội dung hoặc thay bằng ảnh tự chụp/ảnh được cấp phép. Giữ các thông tin attribution cụ thể tại [Myntra sample attribution](frontend/public/assets/products/myntra-sample/ATTRIBUTION.md).

### Sửa ảnh sản phẩm Arrow

Tài liệu trước đây mô tả một migration chỉ cập nhật `product_images.image_url` và `alt_text`, sau khi xác thực sản phẩm, variants/SKU và ảnh chính. Tuy nhiên, file `database/migrations/20261022_repair_arrow_product_image.sql` không có trong checkout hiện tại nên không thể chạy hướng dẫn đó. Các file `database/backups/fashion-*-before-repair.json` là dữ liệu backup dạng JSON, không thay thế PostgreSQL dump hay migration. Không ghi dữ liệu production cho đến khi migration đã xác minh được cung cấp.

Trước mọi thay đổi production, tạo backup PostgreSQL vào thư mục bảo vệ và xác minh file backup không rỗng. Khôi phục thử vào database riêng trước; không restore đè database đang chạy nếu chưa có kế hoạch recovery được duyệt.

## 7. Quyền sử dụng và tài liệu được giữ riêng

- Giữ [Myntra sample attribution](frontend/public/assets/products/myntra-sample/ATTRIBUTION.md) và [seed catalog attribution](frontend/public/assets/products/seed-catalog/ATTRIBUTION.md) cùng dữ liệu/ảnh tương ứng.
- README, license và tài liệu trong `performance/report/sbadmin2-1.0.7/bower_components/` thuộc các dependency bên thứ ba, không gộp hoặc xóa.
- Tài liệu này thay thế các hướng dẫn dự án trước đây: cài đặt/môi trường, test, OTP, shipping, database audit, frontend template README và deployment README dạng TXT.
