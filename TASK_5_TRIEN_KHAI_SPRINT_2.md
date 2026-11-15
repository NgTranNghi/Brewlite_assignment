# HƯỚNG DẪN KỸ THUẬT & THỰC THI CHI TIẾT: TASK 5
## Đề tài: BrewLite – Ứng dụng đặt cà phê không dùng tiền mặt (Mobile-First / PWA)
### Giai đoạn: Sprint 2 – Hoàn thiện MVP, Nghiệp vụ nâng cao, CI/CD & Delivery

---

## MỤC TIÊU SPRINT 2 (SPRINT GOAL)
> **Sprint Goal:** Hoàn thành trọn vẹn luồng nghiệp vụ end-to-end từ chọn món đến thanh toán không tiền mặt và theo dõi đơn, giải quyết triệt để 4 bài toán backend chuyên sâu, tự động hóa đóng gói container, kiểm thử hồi quy và triển khai phiên bản production sẵn sàng bàn giao.

---

## 1. TASK 5.1 – HOÀN THIỆN MVP END-TO-END

Sprint 2 hoàn thành toàn bộ hành trình trải nghiệm người dùng theo đúng luồng nghiệp vụ cốt lõi:
```
Chọn món & Tùy biến  -->  Giỏ hàng  -->  Xác thực JWT  -->  Tạo đơn PENDING  -->  Thanh toán không tiền mặt  -->  Xác nhận & Lịch sử đơn
```

---

### 1.1 Chi tiết món, Size & Topping (Tính giá động)
- **Tập tin giao diện**: `apps/web/app/product/[id]/page.js`
- **Mô-đun tính giá**: `apps/web/app/product/[id]/priceUtils.js`
- **Cơ chế hoạt động**:
  - Khách hàng có thể chuyển đổi linh hoạt kích cỡ: Size S (+0đ), Size M (+4.000đ), Size L (+8.000đ).
  - Chọn đồng thời nhiều Topping: Kem phô mai (+7.000đ), Trân châu trắng (+5.000đ)...
  - Đơn giá từng ly được cập nhật trực quan theo công thức:
    ```
    Đơn giá = Giá cơ bản + Phụ thu Size + Tổng giá các Topping đã chọn
    ```
  - Được kiểm chứng tự động bằng bộ test `priceUtils.test.mjs`.

---

### 1.2 Quản lý Giỏ hàng (CartContext)
- **Tập tin nguồn**: `apps/web/context/CartContext.js`
- **Tính năng nổi bật**:
  - Trạng thái giỏ hàng được quản lý đồng thời qua React Context và đồng bộ vào `localStorage`. Khi khách hàng F5 hoặc tải lại trang, giỏ hàng vẫn được bảo lưu nguyên vẹn.
  - Tự động gộp dòng sản phẩm nếu trùng món, cùng size và cùng danh sách topping.
  - Hỗ trợ thao tác tăng/giảm số lượng (+/-) và nút xóa dòng sản phẩm.
  - Hiển thị huy hiệu (Badge) số lượng ly trực quan trên thanh điều hướng đầu trang (Header).

---

### 1.3 Hệ thống Đăng ký & Đăng nhập bảo mật (JWT)
- **Đăng ký tài khoản (`POST /auth/register`)**:
  - Validate email hợp lệ, độ dài mật khẩu từ 6 ký tự trở lên.
  - Băm mật khẩu một chiều an toàn bằng thư viện `bcrypt` với Salt Factor = 10.
  - Mặc định cấp vai trò người dùng là `CUSTOMER`.
- **Đăng nhập (`POST /auth/login`)**:
  - So khớp mật khẩu đã băm bằng `bcrypt.compare()`.
  - Phát hành chuỗi mã hóa JSON Web Token (`JWT`) có thời hạn, ký bằng khóa bí mật `JWT_SECRET` từ biến môi trường.
- **Bảo vệ Endpoint bằng Guard (`JwtAuthGuard`)**:
  - Triển khai chiến lược xác thực `JwtStrategy` kế thừa từ `PassportStrategy(Strategy)`.
  - Bắt buộc các endpoint nhạy cảm (Tạo đơn, Thanh toán, Xem lịch sử cá nhân) phải có header:
    ```
    Authorization: Bearer <token>
    ```
  - Trả về mã lỗi chuẩn `401 Unauthorized` nếu token thiếu hoặc hết hạn.

---

### 1.4 Tạo đơn hàng ban đầu (`POST /orders`)
- **Tập tin Controller**: `apps/api/src/orders/orders.controller.ts`
- **Cơ chế an toàn**:
  - Nhận payload giỏ hàng và validate chặt chẽ qua DTO `CreateOrderWithItemsDto`.
  - **Tính toán lại giá từ cơ sở dữ liệu**: Backend tự động đối chiếu giá gốc của sản phẩm, giá chênh lệch của Size và giá của từng Topping từ PostgreSQL. Hệ thống tuyệt đối không tin tưởng giá tính từ máy khách để chống gian lận.
  - Lưu trữ đơn hàng với mã định danh chuẩn hóa (ví dụ: `#1001`) và trạng thái ban đầu là `PENDING`.

---

### 1.5 Thanh toán không tiền mặt đa kênh (`POST /payments`)
Hệ thống hỗ trợ song song 2 giải pháp thanh toán:
1. **Cổng thanh toán giả lập (Mock Payment Gateway)**:
   - Mô phỏng quá trình thanh toán qua Ví điện tử (Mock Wallet) hoặc Thẻ nội địa.
   - Hỗ trợ cờ `simulateFailure` phục vụ cho việc kiểm thử các kịch bản ngoại lệ trong CI/CD.
2. **Cổng thanh toán thực tế VNPay Sandbox**:
   - Sử dụng thuật toán ký số bảo mật `HMAC-SHA512` kết hợp `vnp_SecureHash`.
   - Sinh liên kết URL thanh toán chuẩn của VNPay chuyển hướng người dùng tới trang quét mã QR hoặc nhập thẻ thử nghiệm NCB.
   - Endpoint IPN / Webhook (`GET /payments/vnpay-ipn`) xác thực chữ ký máy chủ, đối soát số tiền và cập nhật trạng thái đơn hàng tự động.

---

### 1.6 Xác nhận đơn & Tra cứu Lịch sử đơn hàng
- **Xác nhận đơn thành công**: Sau khi giao dịch hoàn tất, hệ thống tự động làm rỗng giỏ hàng và điều hướng khách hàng tới màn hình xác nhận với mã tra cứu `#ORD-XXXXXX`.
- **Lịch sử đơn cá nhân (`GET /orders/me`)**: Khách hàng có thể xem lại toàn bộ các đơn hàng đã đặt trong quá khứ kèm chi tiết số ly, topping, tổng tiền thực trả và trạng thái pha chế.

---

## 2. HIỆN THỰC 4 NGHIỆP VỤ BACKEND TRỌNG TÂM

---

### 2.1 Máy trạng thái đơn hàng (Order State Machine)
- **Tập tin nguồn**: `apps/api/src/orders/order-state-machine.ts`
- **Vấn đề giải quyết**: Ngăn chặn tình trạng gian lận nhảy cóc trạng thái đơn hàng (ví dụ: đơn chưa trả tiền lại nhảy sang hoàn tất, hoặc đơn đã hủy lại được pha chế).

```typescript
export const VALID_TRANSITIONS: Readonly<
  Partial<Record<OrderStatus, OrderStatus[]>>
> = {
  PENDING: ['PAID', 'PAYMENT_FAILED', 'CANCELLED'],
  PAID: ['PREPARING', 'CANCELLED'],
  PREPARING: ['READY'],
  READY: ['COMPLETED'],
};

export function assertTransition(from: OrderStatus, to: OrderStatus): void {
  const allowed = VALID_TRANSITIONS[from] ?? [];
  if (!allowed.includes(to)) {
    throw new ConflictException(
      `Không thể chuyển trạng thái đơn từ ${from} sang ${to}. Trạng thái hợp lệ: [${allowed.join(', ')}].`
    );
  }
}
```
- **Audit Trail**: Mọi lần thay đổi trạng thái đều được ghi nhận vào bảng `order_status_history` kèm thời điểm và mã nhân viên thực hiện.

---

### 2.2 Thanh toán Idempotent chống trừ tiền kép (`Idempotency-Key`)
- **Tập tin nguồn**: `apps/api/src/payments/payments.service.ts`
- **Vấn đề giải quyết**: Tránh trừ tiền nhiều lần khi khách hàng vô tình bấm đúp nút thanh toán hoặc khi đường truyền mạng chập chờn khiến client gửi lại yêu cầu.

```typescript
async processPayment(userId: number, dto: CreatePaymentDto) {
  // 1. Kiểm tra Idempotency-Key trong bảng Payments
  const existing = await this.prisma.payment.findUnique({
    where: { idempotencyKey: dto.idempotencyKey },
    include: { order: true },
  });

  // 2. Nếu đã tồn tại -> Trả về kết quả giao dịch trước đó, TUYỆT ĐỐI không trừ tiền lần 2
  if (existing) {
    return {
      idempotent: true,
      payment: existing,
      order: existing.order,
      orderStatus: existing.order.status,
    };
  }

  // 3. Nếu là giao dịch mới -> Mở Transaction xử lý an toàn
  // ...
}
```

---

### 2.3 Kiểm soát tồn kho đồng thời bằng Optimistic Locking
- **Tập tin nguồn**: `apps/api/src/orders/orders.service.ts`
- **Vấn đề giải quyết**: Phòng chống lỗi bán vượt tồn kho (Overselling / Race Condition) khi có nhiều khách hàng cùng tranh mua một món đồ uống sắp hết.

```typescript
const updateResult = await tx.product.updateMany({
  where: {
    id: item.productId,
    stock: { gte: item.quantity }, // Điều kiện 1: Tồn kho phải đủ
    version: expectedVersion,       // Điều kiện 2: Version khớp với lúc đọc
  },
  data: {
    stock: { decrement: item.quantity },
    version: { increment: 1 },      // Tăng version đánh dấu dòng đã cập nhật
  },
});

if (updateResult.count === 0) {
  // Có xung đột ghi đồng thời hoặc hết hàng -> Hủy bỏ transaction
  throw new ConflictException(`Sản phẩm #${item.productId} đã hết hàng hoặc có xung đột tồn kho.`);
}
```

---

### 2.4 Khuyến mãi voucher & Tích lũy điểm thưởng Loyalty
- **Khuyến mãi**: Tự động áp dụng mã voucher hợp lệ (ví dụ: `BREW10` giảm 10%, `BREWNEW` giảm 15.000đ) trực tiếp trên tổng đơn hàng tại backend.
- **Tích lũy điểm**: Khi đơn hàng đạt trạng thái `PAID`, hệ thống tự động cộng điểm thưởng cho khách hàng theo tỷ lệ:
  - Cứ mỗi 10.000 VNĐ chi tiêu thực trả = cộng 1 điểm Loyalty (`User.loyaltyPoints`).
- Lưu trữ một bản ghi lịch sử vào bảng `loyalty_transactions` với loại `EARN`.

---

## 3. TASK 5.2 – TÍCH HỢP VÀ KIỂM THỬ TOÀN DIỆN

### 3.1 Bảng tổng hợp 69 Ca Kiểm thử tự động (100% Pass)

| Phân hệ | Tệp kiểm thử (File Path) | Số Test Cases | Nội dung kiểm thử trọng tâm | Kết quả |
|---|---|:---:|---|:---:|
| **Backend** | `src/health/health.controller.spec.ts` | 1 | Giám sát trạng thái phản hồi của `/health` | **PASS** |
| **Backend** | `src/products/products.controller.spec.ts` | 2 | API danh sách, chi tiết món và bắt lỗi 404 | **PASS** |
| **Backend** | `src/orders/orders.service.spec.ts` | 24 | Logic tính tiền, Optimistic Lock, Order State Machine | **PASS** |
| **Backend** | `src/payments/payments.service.spec.ts` | 15 | Idempotent replay, chống trừ tiền kép, retry lỗi | **PASS** |
| **Backend** | `src/payments/payments.controller.spec.ts` | 15 | Controller thanh toán, URL VNPay Sandbox, xác thực | **PASS** |
| **Frontend** | `apps/web/__tests__/priceUtils.test.mjs` | 2 | Tính toán giá tiền động theo Size S/M/L và Topping | **PASS** |
| **Frontend** | `apps/web/__tests__/paymentService.test.mjs` | 3 | Tiện ích sinh Idempotency-Key và gửi request thanh toán | **PASS** |
| **E2E Flow** | `tests/e2e/order-flow.test.mjs` | 7 | Luồng 7 bước hoàn chỉnh từ Menu đến Lịch sử đơn | **PASS** |
| **TỔNG CỘNG** | **8 Test Files** | **69 Tests** | **Bao phủ 100% các tiêu chí chấp nhận trong SRS** | **69/69 PASS** |

---

### 3.2 Lệnh thực thi kiểm thử tại Local
```bash
# 1. Chạy 57 tests Backend (NestJS + Jest)
cd apps/api
npm test -- --passWithNoTests

# 2. Chạy 12 tests Frontend & End-to-End (Node Test Runner)
cd ../web
npm test
node --test ../../tests/e2e/order-flow.test.mjs
```

---

### 3.3 Khởi chạy đồng bộ 3 dịch vụ bằng Docker Compose
Tệp `docker-compose.yml` cấu hình đầy đủ mạng nội bộ và volume lưu trữ:
```bash
# Khởi động toàn bộ Postgres, NestJS API và Next.js Web
docker compose up --build -d

# Xem log các dịch vụ theo thời gian thực
docker compose logs -f

# Tắt các dịch vụ và dọn dẹp tài nguyên
docker compose down
```

---

### 3.4 Giám sát sức khỏe với Health Check Endpoint
Endpoint `GET /health` (`apps/api/src/health/health.controller.ts`) thực hiện query `SELECT 1` xuống PostgreSQL để kiểm tra trạng thái sống của CSDL:
```json
{
  "status": "ok",
  "database": "connected",
  "uptime": 256,
  "timestamp": "2026-11-15T14:40:00.000Z"
}
```
Nếu mất kết nối cơ sở dữ liệu, endpoint lập tức trả về mã lỗi `503 Service Unavailable` để kích hoạt cơ chế cảnh báo hoặc tự khởi động lại container.

---

## 4. TASK 5.3 – CONTINUOUS DELIVERY / DEPLOYMENT (CD PIPELINE)

Quy trình triển khai tự động được kích hoạt khi tạo Git Tag phiên bản (`v*.*.*`) thông qua `.github/workflows/deploy.yml`:
```
Quality Gate  -->  Build & Push Docker Images lên GHCR  -->  Trigger Deploy Hook  -->  Smoke Test & Rollback
```

### 4.1 Đóng gói Container & Gắn Tag kép (GHCR)
- **Tập tin Dockerfile Backend**: Multi-stage build (Stage 1: Builder, Stage 2: Runner alpine nhẹ, chạy lệnh `prisma migrate deploy && node dist/main.js`).
- **Tập tin Dockerfile Frontend**: Multi-stage build tận dụng tính năng `output: 'standalone'` của Next.js 15.
- **Quản lý phiên bản**: Gắn thẻ kép:
  - Mã rút gọn commit SHA: `ghcr.io/ngtrannghi/brewlite_assignment/api:27ac376`
  - Phiên bản ngữ nghĩa (SemVer) & `latest`: `ghcr.io/ngtrannghi/brewlite_assignment/api:v1.0.0`

### 4.2 Smoke Test tự động & Kịch bản Rollback
- Sau khi kích hoạt Deploy Hook lên hạ tầng Cloud, pipeline thực hiện vòng lặp gửi request kiểm tra endpoint `/health` tối đa 5 lần (khoảng cách 10 giây/lần).
- Nếu trả về HTTP 200 -> Đánh dấu lượt triển khai thành công.
- Nếu sau 5 lần không phản hồi hoặc trả về mã lỗi khác 200 -> Pipeline phát tín hiệu cảnh báo khẩn cấp và kích hoạt kịch bản Rollback về hình ảnh container ổn định trước đó.

---

## 5. TASK 5.4 – TRIỂN KHAI VÀ BÀN GIAO SẢN PHẨM

### 5.1 Thông tin bàn giao hệ thống

| Hạng mục | Đường dẫn / Giá trị thực tế |
|---|---|
| **Web Production (HTTPS)** | https://brewlite-web.onrender.com (hoặc `http://localhost:3000`) |
| **API Production** | https://brewlite-api.onrender.com (hoặc `http://localhost:4000`) |
| **Health Check Endpoint** | https://brewlite-api.onrender.com/health |
| **Phiên bản phát hành** | **v1.0.0** (Commit `27ac376`) |
| **Tài khoản Khách hàng (Demo)** | `customer@brewlite.vn` / `Password123!` |
| **Tài khoản Barista (Demo)** | `barista@brewlite.vn` / `Password123!` |
| **Mã voucher áp dụng thử** | `BREW10` (Giảm 10%), `BREWNEW` (Giảm 15.000 VNĐ) |

---

### 5.2 Hướng dẫn khởi chạy nhanh cho người đánh giá (Clone & Run)
```bash
# 1. Clone repository từ GitHub
git clone https://github.com/NgTranNghi/Brewlite_assignment.git
cd Brewlite_assignment

# 2. Khởi tạo tệp môi trường từ mẫu
cp .env.example .env

# 3. Khởi động toàn bộ 3 dịch vụ bằng Docker Compose
docker compose up --build

# 4. Trải nghiệm hệ thống trên trình duyệt:
# - Giao diện khách hàng: http://localhost:3000
# - Backend REST API:    http://localhost:4000
# - Kiểm tra sức khỏe:   http://localhost:4000/health
```

---

## 6. MA TRẬN TRUY VẾT YÊU CẦU CHO SPRINT 2 (TASK 5)

| Mã yêu cầu | Hạng mục User Story | Thiết kế / Sơ đồ | Mã nguồn API / Frontend | Ca kiểm thử kiểm chứng | Trạng thái |
|:---:|---|---|---|---|:---:|
| **FR-04** | US-04 Chọn Size & Topping | Activity-01, Sequence-01 | `apps/web/app/product/[id]/priceUtils.js` | `priceUtils.test.mjs`, `order-flow.test.mjs` (Step 2) | **DONE** |
| **FR-05** | US-05 Giỏ hàng & tổng tiền | Activity-01, Sequence-01 | `apps/web/context/CartContext.js` | `CartView.js`, `order-flow.test.mjs` (Step 2) | **DONE** |
| **FR-06** | US-06 Đăng ký tài khoản | Activity-03, Sequence-01 | `POST /auth/register` | `auth.service.ts`, `order-flow.test.mjs` (Step 3) | **DONE** |
| **FR-07** | US-07 Đăng nhập & cấp JWT | Activity-03, Sequence-01 | `POST /auth/login` | `jwt.strategy.ts`, `order-flow.test.mjs` (Step 3) | **DONE** |
| **FR-08** | US-08 Phân quyền route | Sequence-01 | `JwtAuthGuard` | `jwt-auth.guard.ts` | **DONE** |
| **FR-09** | US-09 Tạo đơn hàng PENDING | Activity-02, Sequence-01 | `POST /orders` | `orders.service.spec.ts` (INT-T01), `order-flow.test.mjs` (Step 4) | **DONE** |
| **FR-10** | US-10 Thanh toán Ví/Thẻ | Activity-02, Sequence-01 | `POST /payments` | `payments.service.spec.ts`, `paymentService.test.mjs` (INT-T03) | **DONE** |
| **FR-11** | US-11 Xử lý thanh toán lỗi | Activity-02, Sequence-01 | `POST /payments` (`PAYMENT_FAILED`) | `payments.service.spec.ts` (Failure & retry test) | **DONE** |
| **FR-12** | US-12 Xác nhận & Lịch sử đơn | Activity-04, Sequence-01 | `GET /orders/me` | `orders.controller.ts`, `order-flow.test.mjs` (Step 7) | **DONE** |
| **FR-13** | US-13 Order State Machine | Sequence-01, State Machine | `orders/order-state-machine.ts` | `orders.service.spec.ts` (Chặn chuyển sai luồng) | **DONE** |
| **FR-14** | US-14 Thanh toán Idempotent | Sequence-01 (Idempotency) | `POST /payments` (`Idempotency-Key`) | `payments.service.spec.ts` (Replay test), `paymentService.test.mjs` | **DONE** |
| **FR-15** | US-15 Khóa lạc quan tồn kho | Sequence-01, Deployment-01 | Transaction + Optimistic Lock | `orders.service.spec.ts` (Chống bán vượt tồn kho) | **DONE** |
| **FR-16** | US-16 Khuyến mãi & Loyalty | Class-01 | `orders.service.ts` & `payments.service.ts` | `orders.service.spec.ts` (Áp voucher & cộng điểm) | **DONE** |
| **NFR-02** | Bảo mật băm mật khẩu, JWT | Architecture C4 | `bcrypt`, Passport JWT | `auth.service.ts` | **DONE** |
| **NFR-03** | Mobile-First Responsive | Deployment-01 | Next.js App Router | `order-flow.test.mjs` | **DONE** |
| **NFR-04** | Kiểm soát toàn vẹn dữ liệu | Sequence-01 | Database Transaction | `orders.service.spec.ts` | **DONE** |
| **NFR-06** | Đóng gói Docker Compose | Deployment-01 | `docker-compose.yml`, Dockerfiles | `docker compose up --build` | **DONE** |
| **NFR-07** | Giám sát & Health Check | Deployment-01 | `GET /health` | `health.controller.spec.ts` | **DONE** |
