# Task 1.1 – Bối Cảnh Nghiệp Vụ, Phân Tích Bên Liên Quan (STAKEHOLDER) và Phân Định Phạm Vi Dự Án Brewlite

- **Dự Án:** BrewLite - Hệ thống Đặt và Thanh toán Đồ uống Trực tuyến
- **Giai đoạn:** Khai thác và Đặc tả yêu cầu (Task 1.1)
- **Tài liệu:** `docs/requirements/context-and-scope.md`
- **Người thực hiện:** Nam
    - Vấn đề nghiệp vụ (1)
    - Phân tích 6 stakeholder (2)
    - Phạm vi MVP, mở rộng, out-of-scope (3)
    - Giả định, ràng buộc, rủi ro (4)

---
## 1. Bối cảnh và Vấn đề Nghiệp vụ (Problem Statement)

### 1.1 Bối cảnh thực tế ngành F&B trong lĩnh vực takeaway
Các cửa hàng cà phê vận hành theo mô hình phục vụ nhanh (Quick-Service), bố trí một quầy pha chế tích hợp quầy thu ngân với quy mô 1–2 nhân sự trong các khung giờ cao điểm (07:30–09:00 và 13:00–14:30). Qua khảo sát thực tế, mô hình vận hành truyền thống tại quầy bộc lộ các bài toán nan giải:

1. **Ùn tắc hàng đợi vật lý (Physical Queue Congestion):** Vào các khung giờ cao điểm, khách hàng phải xếp hàng chờ gọi món và thanh toán với thời gian chờ trung bình từ 5 đến 8 phút mỗi lượt vào giờ cao điểm, gây sụt giảm mức độ hài lòng và tỷ lệ từ bỏ mua hàng.
2. **Sai lệch thông tin do ghi nhận thủ công (Transcription Errors):** Nhân viên quầy ghi nhận yêu cầu (món, kích cỡ, mức đường/đá, topping) bằng sổ tay hoặc giấy nhớ, dẫn đến tỷ lệ pha chế nhầm lẫn trung bình ước tính 4–6% tổng đơn hàng vào giờ cao điểm.
3. **Rủi ro và ma sát trong quản lý tiền mặt (Cash Handling Inefficiency):** Thanh toán tiền mặt làm tăng thời gian xử lý giao dịch tại quầy (trung bình 30–45 giây/giao dịch cho việc đếm và trả tiền thừa), đồng thời phát sinh rủi ro thất thoát và phức tạp hóa công tác đối soát kế toán cuối ngày.
4. **Mù mờ tiến độ xử lý đơn hàng (Order State Opacity):** Khách hàng hoàn toàn không nắm được tiến trình pha chế sau khi đặt món, dẫn đến hành vi đứng chờ chen lấn tại khu vực lấy đồ và liên tục hỏi lại nhân viên quầy.
5. **Thiếu hụt dữ liệu phục vụ điều hành (Absence of Operational Data):** Ban quản lý thiếu các chỉ số đo lường tức thời (sản phẩm bán chạy, tần suất tiêu thụ theo khung giờ, tỷ lệ hủy đơn) để tối ưu hóa nguồn nguyên liệu và nhân sự.
6. **Thiếu liên kết thành viên và tích lũy điểm thưởng (Customer Retention):** Các quán nhỏ thường dùng thẻ tích điểm giấy hoặc ghi chép rời rạc, làm giảm tỷ lệ quay lại của khách quen và không xây dựng được tệp khách hàng trung thành bền vững.

### 1.2 Giải pháp hệ thống Brewlite
> "Đối với khách hàng sử dụng dịch vụ ăn uống hiện đại mong muốn tiết kiệm thời gian, **BrewLite** là nền tảng ứng dụng web định hướng Mobile-first / PWA cho phép người dùng chủ động xem thực đơn, tùy biến đồ uống, đặt hàng và thanh toán trực tuyến không dùng tiền mặt trước khi đến quầy. Khác với phương thức gọi món tại quầy truyền thống, hệ thống rút ngắn thời gian tiếp nhận đơn xuống dưới 60 giây, loại bỏ hoàn toàn sai sót ghi chép và minh bạch hóa trạng thái đơn hàng theo thời gian thực."

Hệ thống số hóa toàn diện quy trình phục vụ theo một chuỗi khép kín:
$$\text{Xem thực đơn} \longrightarrow \text{Tùy biến món} \longrightarrow \text{Giỏ hàng} \longrightarrow \text{Đặt đơn} \longrightarrow \text{Thanh toán điện tử} \longrightarrow \text{Theo dõi tiến trình \& Tích điểm}$$

---

## 2. Phân Tích Bên Liên Quan (Stakeholder Analysis)

Nhằm đảm bảo sản phẩm đáp ứng đầy đủ góc nhìn từ vận hành kinh doanh, hệ thống **BrewLite** xác định và phân tích toàn diện **đúng 6 nhóm Stakeholder chính** theo yêu cầu như sau:

| Mã ST | Bên liên quan (Stakeholder) | Phân loại Tác nhân | Vai trò trong hệ thống | Mục tiêu và Kỳ vọng cốt lõi |
| :---: | :--- | :--- | :--- | :--- |
| **ST-01** | **Khách hàng (Customer/end-user)** | Human Actor (Tác nhân chính) | Người trực tiếp duyệt menu, tạo đơn, thanh toán và theo dõi tiến độ | • Thao tác hoàn tất đặt đơn trong vòng < 60 giây.<br>• Minh bạch tình trạng đơn hàng.<br>• Giỏ hàng được bảo toàn khi xảy ra sự cố mạng.<br>• Thanh toán không tiền mặt bảo mật, tức thời. |
| **ST-02** | **Nhân viên quầy (Barista)** | Human Actor (Tác nhân phụ) | Tiếp nhận danh sách đơn hàng đã thanh toán và cập nhật trạng thái chế biến | • Hiển thị thông số pha chế (size, topping) chính xác 100%.<br>• Cập nhật trạng thái đơn (Pha chế → Sẵn sàng) chỉ với 1 thao tác. |
| **ST-03** | **Quản lý cửa hàng / Product Owner** | Nghiệp vụ nội bộ | Quản lý danh mục sản phẩm, chính sách giá, khuyến mãi và nghiệm thu | • Kiểm soát doanh thu và tồn kho chính xác.<br>• Linh hoạt cấu hình khuyến mãi và điểm thưởng.<br>• Nghiệm thu chuẩn xác sản phẩm sau mỗi Sprint. |
| **ST-04** | **Giảng viên / Khách hàng nghiệm thu** | Đơn vị đánh giá độc lập | Giám sát chất lượng quy trình phát triển và kiểm định kỹ thuật | • Đánh giá năng lực của nhóm qua tính hoàn chỉnh của 5 Task.<br>• Pipeline CI/CD vận hành thực tế.<br>• Kiểm chứng việc áp dụng đúng đắn kiến trúc 3 tầng, tính toán tiền tại Business Layer. |
| **ST-05** | **Nhóm phát triển (Dev Team)** | Nhóm thực thi kỹ thuật | Phân tích, thiết kế, hiện thực hóa mã nguồn và duy trì CI/CD | • Ngăn xếp công nghệ đồng nhất (Next.js, NestJS, Postgres).<br>• Quy trình tự động hóa kiểm thử và triển khai trơn tru.<br>• Kiến trúc rõ ràng, dễ bảo trì. |
| **ST-06** | **Đơn vị thanh toán đối tác (Payment Partner / Sandbox Gateway)** | External System Actor | Xử lý các giao dịch tài chính điện tử (Ví/Thẻ qua API mô phỏng) | • Tiếp nhận yêu cầu thanh toán có gắn khóa Idempotency-Key.<br>• Phản hồi trạng thái giao dịch xác định (SUCCESS / FAILED). |

---

## 3. Phạm Vi Sản Phẩm (System Scope)

### 3.1 Phạm vi cốt lõi của MVP (In-Scope)
1. **Quản lý Danh mục & Menu:** Cung cấp API và giao diện hiển thị danh mục đồ uống với đầy đủ hình ảnh, mô tả, giá chuẩn và trạng thái tồn kho (`GET /products`).
2. **Chi tiết & Tùy biến món:** Cho phép chọn size chuẩn (S, M, L) và các loại topping kèm theo, cập nhật giá động theo cấu hình (`GET /products/:id`).
3. **Quản lý Giỏ hàng:** Lưu trữ và đồng bộ giỏ hàng phía Client, hỗ trợ tính toán tổng tiền tạm tính và lưu trạng thái qua phiên duyệt web.
4. **Xác thực & Ủy quyền:** Đăng ký, đăng nhập người dùng, mã hóa mật khẩu một chiều (`bcrypt`), cấp phát và xác thực phân quyền qua JWT.
5. **Khởi tạo Đơn hàng (Order Placement):** Xử lý giao dịch tạo đơn hàng (`POST /orders`), kiểm tra tính sẵn sàng của tồn kho bằng Transaction mức CSDL.
6. **Xử lý Thanh toán (Payment Processing):** Tích hợp cổng thanh toán mô phỏng Ví điện tử/Thẻ ngân hàng, xử lý kiểm soát trùng lặp giao dịch bằng `Idempotency-Key`.
7. **Máy trạng thái Đơn hàng (Order State Machine):** Quản lý chu trình trạng thái đơn theo luồng chuẩn: `PENDING → PAID → PREPARING → READY → COMPLETED` (hoặc `PAYMENT_FAILED`, `CANCELLED`).
8. **Biên nhận & Lịch sử Đơn hàng:** Cung cấp biên nhận xác nhận sau thanh toán và giao diện tra cứu lịch sử đơn hàng cá nhân (`GET /orders/me`).
9. **Đóng gói & Hạ tầng Vận hành:** Toàn bộ hệ thống chạy khép kín trong Docker Compose (Frontend, Backend, PostgreSQL) kèm kiểm tra trạng thái sức khỏe (`GET /health`).

### 3.2 Phạm vi mở rộng (Out-of-Scope)
- Màn hình chuyên biệt dạng POS tại quầy cho Barista (hiện tại cập nhật trạng thái qua API).
- Tích hợp cổng thanh toán thương mại có tính pháp lý tiền thật (VNPay/Momo Merchant thật).
- Hệ thống điều phối giao hàng tận nơi (Delivery dispatch) và tính cước vận chuyển theo vị trí GPS.
- Giao diện Admin quản trị nghiệp vụ nâng cao (hiện tại dữ liệu được nạp qua Seed/Migration).

### 3.3 Nội dung loại trừ hoàn toàn (Out-of-System)
- Nghiệp vụ hạch toán kế toán doanh nghiệp, xuất hóa đơn tài chính VAT theo chuẩn cơ quan thuế.
- Quản trị chuỗi cung ứng nguyên vật liệu thô và chấm công nhân sự.
- Hệ thống định vị bản đồ GPS để điều phốt tài xế giao hàng
- Ứng dụng di động độc lập trên các nền tảng app store / google play

---

## 4. Giả Định, Ràng Buộc Kỹ Thuật và Đánh Giá Rủi Ro

### 4.1 Giả định

| ID   | Giả định                                                                                     | Nếu sai thì ảnh hưởng                                                |
| ---- | -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| A-01 | Khách hàng sở hữu smartphone và có kết nối Internet khi đặt món                              | Nếu không, cần kênh offline/kiosk → tăng phạm vi                     |
| A-02 | Hệ thống thanh toán chỉ cần **mô phỏng** trong phạm vi bài tập                               | Nếu cần thật → cần tài khoản merchant, hợp đồng, PCI (ngoài phạm vi) |
| A-03 | Nhân viên quầy sẽ dùng một thiết bị riêng hoặc tài khoản quản trị để cập nhật trạng thái đơn | Nếu không có thiết bị → cần màn hình quầy riêng (OS-01)              |
| A-04 | Dữ liệu món, giá và topping do nhóm seed sẵn và cập nhật qua migration                       | Nếu cần admin UI → thêm phạm vi                                      |
| A-05 | Không yêu cầu xác thực 2 lớp hay xác minh OTP thực cho bài tập                               | Nếu cần → thêm provider OTP, tăng NFR bảo mật                        |
| A-06 | Hạ tầng triển khai dùng dịch vụ PaaS có sẵn Postgres miễn phí (Render)                       | Nếu đổi nền tảng → cập nhật ADR-006 và deployment diagram            |
| A-07 | Nhóm phát triển gồm 3–5 thành viên, phát triển song song nhưng merge qua PR                  | Nếu nhóm nhỏ hơn → giảm số nhánh, tăng rủi ro merge conflict         |
| A-08 | Giá lưu dưới dạng số nguyên VND (hoặc Decimal) để tránh lỗi làm tròn số thực                 | Nếu dùng Float → lỗi làm tròn tổng tiền                              |

### 4.2 Ràng buộc (Constraints)

| ID   | Ràng buộc                                                                                                                             | Nguồn          |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------- | -------------- |
| C-01 | Bắt buộc dùng **Next.js** (frontend), **NestJS** (backend), **PostgreSQL** (CSDL), **Docker** (đóng gói)                              | Đề bài A2 §1.1 |
| C-02 | Phải có CI chạy `Install → Lint → Unit Test → Build` khi mở Pull Request; PR không được merge nếu pipeline fail                       | Đề bài A2 §6.4 |
| C-03 | Phải có CD: build image tag theo commit SHA/semver, push registry, deploy production, chạy migration an toàn, smoke test, có rollback | Đề bài A2 §7.3 |
| C-04 | Phải có **URL production truy cập được qua HTTPS** + URL API + health check; DB production không public trực tiếp                     | Đề bài A2 §7.4 |
| C-05 | Secret **không** được xuất hiện trong repository hoặc log CI                                                                          | Đề bài A2 §7.4 |
| C-06 | Thời gian phản hồi API **< 500 ms** với dữ liệu mẫu                                                                                   | Đề bài A1 §4.2 |
| C-07 | Phải có unit test, integration test cho tạo đơn/thanh toán, ít nhất 1 E2E test menu → xác nhận đơn                                    | Đề bài A2 §7.2 |
| C-08 | Thực hiện 5 task theo **đúng thứ tự**, tạo phẩm của task trước phải có trên kho mã nguồn kèm lịch sử thay đổi rõ ràng                 | Đề bài A2 §1   |
| C-09 | Stack bắt buộc: Next.js + NestJS + PostgreSQL + Docker; không thay thế các thành phần này                                             | Đề bài A2 §1.1 |
| C-10 | Mô hình lớp tối thiểu: User, Product, Order, OrderItem, Payment, Promotion, LoyaltyTransaction                                        | Đề bài A2 §4.3 |

### 4.3 Rủi ro ban đầu (Initial Risks)

| ID   | Rủi ro                                                                          | Xác suất   | Tác động   | Mức        | Biện pháp giảm thiểu                                                                                     |
| ---- | ------------------------------------------------------------------------------- | ---------- | ---------- | ---------- | -------------------------------------------------------------------------------------------------------- |
| R-01 | Đặt đồng thời vượt tồn kho (overselling)                                        | Cao        | Cao        | **Cao**    | Transaction + optimistic locking (`version`) + kiểm thử đồng thời (A1 Task 10)                           |
| R-02 | Thanh toán bị trừ tiền/đặt đơn 2 lần do người dùng double-click hoặc retry mạng | Cao        | Cao        | **Cao**    | `Idempotency-Key` + unique constraint ở DB + trả lại kết quả cũ                                          |
| R-03 | Rò rỉ secret (DB password, JWT secret) vào repo hoặc log CI                     | Trung bình | Cao        | **Cao**    | `.gitignore` + `.env.example`, secret trong CI/CD Settings, `prisma migrate deploy` dùng biến môi trường |
| R-04 | Pipeline CI chạy lâu (>10 phút) làm giảm tần suất merge                         | Trung bình | Thấp       | Trung bình | Cache npm, chạy lint/test song song, matrix test                                                         |
| R-05 | Frontend không build được trong CI vì khác biệt môi trường/Node version         | Trung bình | Trung bình | Trung bình | Ghim `node:20-alpine`, `engines` trong package.json, cache `~/.npm`                                      |
| R-06 | Deploy production thất bại giữa chừng, DB ở trạng thái nửa cũ nửa mới           | Trung bình | Cao        | Cao        | Migration chỉ **additive**, chạy `migrate deploy` trước khi cắt traffic, rollback bằng image tag trước   |
| R-07 | Phạm vi tăng trưởng (thêm màn hình quản trị, thanh toán thật) làm trễ deadline  | Cao        | Trung bình | **Cao**    | Danh sách out-of-scope đã chốt §3.2; mọi đề xuất mới vào backlog Sprint sau                              |
| R-08 | Tài liệu lệch với code khi refactor                                             | Trung bình | Trung bình | Trung bình | Ma trận truy vết cập nhật trong mỗi PR; CI check tồn tại endpoint                                        |
| R-09 | Hết hạn quota DB PaaS (free tier) làm sập production                            | Trung bình | Cao        | Trung bình | Chọn nền tảng có free tier ổn định, ghi rõ giới hạn, có kịch bản migrate dữ liệu ra ngoài                |
| R-10 | Nhóm thiếu thiết bị/điều kiện deploy                                            | Thấp       | Cao        | Thấp       | Dùng PaaS miễn phí; không phụ thuộc thẻ tín dụng                                                         |

---
## 5. Liên kết

- Yêu cầu chi tiết (FR/NFR, MoSCoW, tiêu chí chấp nhận): [`srs.md`](./srs.md)
- Đặc tả use-case trọng tâm: [`use-case-checkout.md`](./use-case-checkout.md)
- Ma trận truy vết: [`../traceability-matrix.md`](../traceability-matrix.md)
- Kiến trúc & ADR: [`../architecture/architecture.md`](../architecture/architecture.md)