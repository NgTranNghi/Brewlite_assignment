# HƯỚNG DẪN KỸ THUẬT & THỰC THI CHI TIẾT: TASK 4
## Đề tài: BrewLite – Ứng dụng đặt cà phê không dùng tiền mặt (Mobile-First / PWA)
### Giai đoạn: Sprint 1 – Nền tảng kỹ thuật, Quản lý tài liệu, MVP Menu & CI Pipeline

---

## MỤC TIÊU SPRINT 1 (SPRINT GOAL)
> **Sprint Goal:** Tạo nền tảng kỹ thuật vững chắc, chuẩn hóa tài liệu dự án có kiểm soát phiên bản (Docs-as-Code) và hiện thực phiên bản MVP đầu tiên chạy được cho luồng xem thực đơn đồ uống.

---

## 1. TASK 4.1 – THIẾT LẬP REPOSITORY & KHUNG DỰ ÁN (PROJECT SETUP)

### 1.1 Cấu trúc Monorepo chuẩn mực
Kho mã nguồn được tổ chức theo mô hình Monorepo tại thư mục gốc, tách bạch giữa Frontend, Backend, Tài liệu và Kiểm thử (tuân thủ Mục 9 Đề bài):

```
Brewlite_assignment/
|-- apps/
|   |-- api/                    # Backend NestJS (TypeScript, Prisma ORM, Jest)
|   |   |-- prisma/             # Schema CSDL, migrations, seed script
|   |   |-- src/                # Controllers, Services, Modules, DTOs, Guards
|   |   |-- Dockerfile          # Multi-stage Dockerfile cho Backend API
|   |   `-- package.json
|   `-- web/                    # Frontend Next.js 15 (App Router, PWA, CSS Modules)
|       |-- app/                # Route pages (/product, /cart, /checkout, /history)
|       |-- components/         # Reusable UI components
|       |-- context/            # CartContext, AuthContext
|       |-- services/           # API services, client helpers
|       |-- Dockerfile          # Standalone Dockerfile cho Frontend
|       `-- package.json
|-- docs/                       # Quản lý tài liệu theo phiên bản (Task 4.2)
|   |-- requirements/           # Task 1: SRS, Scope, Use-cases
|   |-- diagrams/               # Task 2: Activity, Sequence, Class, Deployment (.puml & .svg)
|   |-- architecture/           # Task 3: C4 overview & 6 ADRs
|   |-- branching-strategy.md   # Chiến lược phân nhánh Git
|   |-- definition-of-done.md   # Tiêu chuẩn nghiệm thu DoD
|   |-- sprint-backlog.md       # Product Backlog & Kế hoạch 2 Sprint
|   `-- traceability-matrix.md  # Ma trận truy vết bắt buộc mục 8.1
|-- tests/                      # Kiểm thử tự động
|   `-- e2e/order-flow.test.mjs # Luồng tích hợp E2E
|-- .github/workflows/          # Tự động hóa CI/CD
|   |-- ci.yml                  # Quality Gate tự động cho PR (Task 4.4)
|   `-- deploy.yml              # Pipeline CD đóng gói và triển khai
|-- docker-compose.yml          # Khởi chạy đồng bộ 3 dịch vụ: web, api, db
|-- .env.example                # File biến môi trường mẫu
|-- .gitignore                  # Bỏ qua các file tạm, thư mục rác
`-- README.md                   # Hướng dẫn tổng quan dự án
```

---

### 1.2 Chiến lược phân nhánh Git & Quy ước Commit

#### 1. Mô hình phân nhánh (Branching Strategy)
- **`main`**: Nhánh production chính thức, được áp dụng quy tắc bảo vệ nghiêm ngặt (Branch Protection Ruleset):
  - Cấm push code trực tiếp lên `main`.
  - Bắt buộc merge thông qua Pull Request (PR) xuất phát từ `develop`.
  - Phải vượt qua 100% các bước kiểm tra của CI Pipeline.
  - Được gắn thẻ phát hành (Release Tag `v*.*.*`).
- **`develop`**: Nhánh tích hợp chính cho toàn bộ các chức năng và tạo phẩm của từng Sprint.
- **`feature/*`**: Các nhánh tính năng phát triển độc lập (ví dụ: `feature/menu-api`, `feature/menu-ui`).
- **`docs/*`**: Nhánh dành riêng cho việc cập nhật tài liệu kỹ thuật, sơ đồ UML và ma trận truy vết.

#### 2. Quy ước Commit (Conventional Commits 1.0.0)
Mọi commit phải tuân thủ cú pháp:
```
<type>(<scope>): <mô tả ngắn gọn>
```
- **Các loại `<type>` quy định**:
  - `feat`: Thêm chức năng mới cho người dùng.
  - `fix`: Sửa lỗi phát sinh.
  - `test`: Thêm hoặc chỉnh sửa các ca kiểm thử tự động.
  - `docs`: Cập nhật tài liệu, sơ đồ hoặc ma trận truy vết.
  - `ci`: Cấu hình hệ thống CI/CD GitHub Actions.
- **Các commit thực tế trong Sprint 1**:
  - `feat(api): implement GET /products and GET /products/:id with prisma`
  - `feat(web): build mobile-first menu view with loading skeleton and error state`
  - `test(api): add unit tests for products controller and service`
  - `ci(actions): configure quality gate workflow with postgresql test service`
  - `docs(requirements): complete context, scope and srs documentation`

---

### 1.3 Quản lý cấu hình biến môi trường (`.env.example`)
Tập tin `.env.example` được đưa vào Git để các thành viên và người đánh giá có thể cấu hình môi trường nhanh chóng, không để lộ secret:

```bash
# ==========================================================
# CẤU HÌNH CƠ SỞ DỮ LIỆU POSTGRESQL
# ==========================================================
POSTGRES_USER=postgres
POSTGRES_PASSWORD=change-me-local
POSTGRES_DB=brewlite_db
POSTGRES_PORT=5433
DATABASE_URL=postgresql://postgres:change-me-local@localhost:5433/brewlite_db?schema=public

# ==========================================================
# CẤU HÌNH BACKEND NESTJS API
# ==========================================================
PORT=4000
NODE_ENV=development
JWT_SECRET=ci-testing-jwt-secret-key-32chars-min

# ==========================================================
# CẤU HÌNH FRONTEND NEXT.JS WEB
# ==========================================================
NEXT_PUBLIC_API_URL=http://localhost:4000
```

---

### 1.4 Quản lý tệp bỏ qua (`.gitignore`)
Loại bỏ hoàn toàn các tệp tin tạm thời, mã biên dịch và thư mục nhạy cảm:
```gitignore
node_modules/
.next/
dist/
build/
.env
.env.local
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.DS_Store
*.tsbuildinfo
```

---

### 1.5 Tiêu chuẩn hoàn thành (Definition of Done - DoD)
Một User Story hoặc Nhiệm vụ kỹ thuật trong Sprint 1 chỉ được chuyển sang trạng thái **DONE** khi đáp ứng đủ 5 điều kiện:
1. Mã nguồn được viết bằng TypeScript/JavaScript rõ ràng, tuân thủ quy tắc ESLint.
2. Có đầy đủ Unit Test cho các Controller và Service liên quan; tỷ lệ pass đạt 100%.
3. Pipeline CI chạy tự động và báo xanh (Pass Install, Lint, Test và Build).
4. Tạo Pull Request, có ít nhất 1 thành viên khác review và phê duyệt trước khi merge vào `develop`.
5. Đã cập nhật tài liệu liên quan trong `docs/` và Ma trận truy vết (`traceability-matrix.md`).

---

## 2. TASK 4.2 – QUẢN LÝ TÀI LIỆU THEO PHIÊN BẢN (DOCS-AS-CODE)

Toàn bộ các tạo phẩm từ Task 1 đến Task 3 được quản lý trực tiếp trong thư mục `docs/` của kho mã nguồn, bảo đảm tài liệu luôn tiến hóa cùng mã nguồn:

### 2.1 Cấu trúc thư mục tài liệu `docs/`
- **`docs/requirements/`**:
  - `context-and-scope.md`: Phân tích bối cảnh bài toán, 6 nhóm Stakeholder, phạm vi In-scope & Out-of-scope.
  - `srs.md`: Đặc tả 16 yêu cầu chức năng (FR-01 đến FR-16) và 7 yêu cầu phi chức năng (NFR-01 đến NFR-07) theo mức độ ưu tiên MoSCoW.
  - `use-case-checkout.md`: Đặc tả chi tiết ca sử dụng trọng tâm UC-07 (Đặt và thanh toán đơn hàng) với Main Flow, Alternative Flows và Exception Flows.
- **`docs/diagrams/`**:
  - Lưu trữ song song cả file nguồn PlantUML (`.puml`) và ảnh vector kết xuất (`.svg`) cho Activity Diagram, Sequence Diagram, Class Diagram và Deployment Diagram.
- **`docs/architecture/`**:
  - `architecture.md`: Tài liệu tổng quan kiến trúc theo mô hình C4 và kiến trúc phân tầng.
  - Thư mục `adr/`: Chứa 6 quyết định kiến trúc cốt lõi (ADR-001 đến ADR-006).

### 2.2 Quy trình cập nhật tài liệu qua Pull Request
Mọi thay đổi về yêu cầu, điều chỉnh lược đồ CSDL hay sửa đổi thiết kế bắt buộc phải đi kèm Pull Request tương ứng:
- Ví dụ: Khi bổ sung bộ lọc danh mục cho menu, lập trình viên tạo PR cập nhật đồng thời file `apps/api/src/products/products.service.ts` và bổ sung mã yêu cầu `FR-03` vào `docs/requirements/srs.md` và `docs/traceability-matrix.md`.

---

## 3. TASK 4.3 – HIỆN THỰC MVP SPRINT 1 (MENU MOBILE-FIRST & API)

### 3.1 Backend RESTful API Danh mục sản phẩm (NestJS + Prisma)

#### 1. Controller (`apps/api/src/products/products.controller.ts`)
Tiếp nhận các yêu cầu HTTP từ máy khách:
```typescript
import { Controller, Get, Param, ParseIntPipe } from '@nestjs/common';
import { ProductsService } from './products.service';

@Controller('products')
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  /**
   * GET /products
   * Endpoint công khai: Trả về danh sách đồ uống đang mở bán kèm giá và ảnh.
   */
  @Get()
  async findAll() {
    return this.productsService.findAll();
  }

  /**
   * GET /products/:id
   * Endpoint chi tiết: Trả về chi tiết đồ uống kèm bảng Size và Topping.
   */
  @Get(':id')
  async findOne(@Param('id', ParseIntPipe) id: number) {
    return this.productsService.findOne(id);
  }
}
```

#### 2. Service (`apps/api/src/products/products.service.ts`)
Xử lý logic truy vấn cơ sở dữ liệu qua Prisma Client:
```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class ProductsService {
  constructor(private readonly prisma: PrismaService) {}

  async findAll() {
    const products = await this.prisma.product.findMany({
      where: { isAvailable: true },
      orderBy: { id: 'asc' },
    });
    return products.map((p) => this.toProductDto(p));
  }

  async findOne(id: number) {
    const product = await this.prisma.product.findUnique({
      where: { id },
      include: {
        sizes: { orderBy: { priceDelta: 'asc' } },
        toppings: { where: { isAvailable: true }, orderBy: { name: 'asc' } },
      },
    });

    if (!product) {
      throw new NotFoundException(`Product with id "${id}" not found`);
    }

    return this.toProductDto(product);
  }

  private toProductDto(p: any) {
    return {
      id: p.id,
      name: p.name,
      description: p.description,
      price: p.price ? Number(p.price) : 0,
      imageUrl: p.imageUrl,
      category: p.category,
      stock: p.stock,
      isAvailable: p.isAvailable,
      sizes: p.sizes?.map((s) => ({
        code: s.code,
        name: s.name,
        priceDelta: Number(s.priceDelta),
        isDefault: s.isDefault,
      })),
      toppings: p.toppings?.map((t) => ({
        id: t.id,
        name: t.name,
        price: Number(t.price),
      })),
    };
  }
}
```

---

### 3.2 Dữ liệu mẫu khởi tạo (Database Seeding)
Tệp `apps/api/prisma/seed.ts` nạp sẵn 8 món đồ uống đa dạng thuộc 3 danh mục để phục vụ việc kiểm thử và trải nghiệm người dùng:

| ID | Tên sản phẩm | Danh mục | Giá cơ sở | Kích thước hỗ trợ |
|:---:|---|---|:---:|---|
| 1 | Cà phê Sữa Đá Sài Gòn | Cà phê truyền thống | 35.000 VNĐ | S (+0đ), M (+4.000đ), L (+8.000đ) |
| 2 | Americano Đá Tỉnh Táo | Cà phê pha máy | 40.000 VNĐ | S (+0đ), M (+4.000đ), L (+8.000đ) |
| 3 | Cappuccino Kem Sữa Nghệ Thuật | Cà phê pha máy | 45.000 VNĐ | S (+0đ), M (+6.000đ), L (+12.000đ) |
| 4 | Caramel Macchiato Hảo Hạng | Cà phê pha máy | 48.000 VNĐ | S (+0đ), M (+6.000đ), L (+12.000đ) |
| 5 | Matcha Đá Xay Kem Tuyết | Trà & Đá xay | 52.000 VNĐ | S (+0đ), M (+6.000đ), L (+12.000đ) |
| 6 | Chocolate Bỉ Đá Xay Hạnh Nhân | Trà & Đá xay | 55.000 VNĐ | S (+0đ), M (+6.000đ), L (+12.000đ) |
| 7 | Trà Đào Cam Sả Tươi Mát | Trà trái cây | 39.000 VNĐ | S (+0đ), M (+5.000đ), L (+10.000đ) |
| 8 | Trà Ô Long Sen Vàng Macchiato | Trà trái cây | 45.000 VNĐ | S (+0đ), M (+5.000đ), L (+10.000đ) |

Lệnh thực hiện nạp dữ liệu mẫu vào CSDL:
```bash
cd apps/api
npx prisma db seed
```

---

### 3.3 Giao diện thực đơn Mobile-First (Next.js)

Giao diện trang chủ (`apps/web/app/page.js`) được thiết kế tối ưu hóa theo hướng Mobile-first / PWA, đáp ứng đầy đủ 3 trạng thái giao diện cốt lõi:
1. **Loading State**: Hiển thị khung chờ Skeleton hiệu ứng sóng mượt mà, giúp người dùng không cảm thấy ứng dụng bị đơ giật khi mạng chậm.
2. **Empty State**: Khi danh mục lọc không có món nào hoặc kết quả tìm kiếm rỗng, hiển thị thông báo: *"Không tìm thấy đồ uống nào phù hợp"* kèm biểu tượng thân thiện.
3. **Error State**: Khi mất kết nối tới Backend API, hiển thị thông báo lỗi mạng rõ ràng kèm nút bấm **"Thử lại"** cho phép gọi lại API trực tiếp mà không cần tải lại toàn bộ trang web.

---

### 3.4 Kiểm thử tối thiểu cho Sprint 1 (Unit Test)
Bộ kiểm thử `apps/api/src/products/products.controller.spec.ts` kiểm tra các trường hợp:
- Lấy danh sách toàn bộ đồ uống thành công (trả về HTTP 200 kèm danh sách mảng).
- Lấy chi tiết đồ uống thành công theo ID hợp lệ.
- Lệnh chạy kiểm thử:
```bash
cd apps/api
npm test -- src/products/products.controller.spec.ts
```
**Kết quả:** 2/2 test cases đạt trạng thái **PASS**.

---

## 4. TASK 4.4 – CONTINUOUS INTEGRATION (CI PIPELINE)

Quy trình tích hợp liên tục được xây dựng bằng **GitHub Actions** thông qua tệp `.github/workflows/ci.yml`.

### 4.1 Mô hình chuỗi công việc CI Quality Gate
```
Install Dependencies  -->  Lint Code  -->  Run Unit & Integration Tests  -->  Build Production Artifacts
```

- **Ảo hóa dịch vụ PostgreSQL**: Tự động kích hoạt container `postgres:15-alpine` trên runner của GitHub Actions, thực hiện kiểm tra `pg_isready` trước khi chạy test database.
- **Kiểm thử tự động trên cả hai phía**:
  - Backend: Chạy Jest với cờ `--coverage` và kết nối trực tiếp vào DB ảo hóa.
  - Frontend: Chạy kiểm tra lint Next.js (`npm run lint`) và kiểm thử giao diện.
- **Build Artifacts**: Biên dịch mã nguồn Backend ra thư mục `dist/` và Frontend thành standalone Next.js build.

---

### 4.2 Cơ chế khóa Merge khi CI có lỗi (Branch Protection)
- Nhóm đã thiết lập quy tắc bảo vệ nhánh cho `main` và `develop`.
- **Minh chứng chất lượng**:
  - Khi một lập trình viên mở Pull Request, GitHub Actions sẽ kích hoạt quy trình CI.
  - Nếu xảy ra lỗi Lint hoặc có test case bị Failed -> Pipeline chuyển sang màu Đỏ -> Nút **"Merge pull request" bị khóa hoàn toàn**.
  - Chỉ khi 100% các bước báo Xanh (Passed) và có sự phê duyệt (Approve) từ thành viên khác trong nhóm thì PR mới được phép sáp nhập.

---

## 5. MA TRẬN TRUY VẾT YÊU CẦU CHO SPRINT 1 (TASK 4)

| Mã yêu cầu | Use-case / User Story | Thiết kế / Sơ đồ | Mã nguồn API / Frontend | Test kiểm chứng | Trạng thái |
|:---:|---|---|---|---|:---:|
| **FR-01** | US-01 Xem thực đơn | Activity-01, Class-01 | `GET /products` | `products.controller.spec.ts` | **DONE** |
| **FR-02** | US-02 Xem chi tiết sản phẩm | Activity-01, Class-01 | `GET /products/:id` | `products.controller.spec.ts` | **DONE** |
| **FR-03** | US-03 Lọc theo danh mục | Activity-01 | `GET /products?category=` | `products.controller.spec.ts` | **DONE** |
| **NFR-01** | Hiệu năng phản hồi < 500ms | Architecture C4 | Prisma Indexing | Jest benchmarks | **DONE** |
| **NFR-05** | Tự động hóa CI Pipeline | Deployment-01 | `.github/workflows/ci.yml` | GitHub Actions Quality Gate | **DONE** |
