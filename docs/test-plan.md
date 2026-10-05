# Test Plan — Fruitables E-Commerce Website

| Trường | Thông tin |
|--------|-----------|
| **Tên dự án** | Fruitables — Hệ thống Thương mại Điện tử |
| **Phiên bản** | v1.0 |
| **Ngày viết** | 09/10/2026 |
| **Người viết** | Nguyễn Quang Huy |
| **Tài liệu tham chiếu** | SRS v2.0 — 20/03/2026 |

---

## 1. Thông tin chung

Fruitables là website bán trái cây và rau củ trực tuyến, xây dựng trên nền tảng ASP.NET Core MVC (.NET 8), Entity Framework Core và SQL Server. Hệ thống phục vụ 4 nhóm người dùng: Guest, Customer, Admin và SuperAdmin.

Test plan này xác định phạm vi, mục tiêu, nguồn lực, lịch trình và tiêu chí hoàn thành cho đợt kiểm thử **v1.0**, tập trung vào các module nghiệp vụ cốt lõi liên quan đến luồng mua hàng và quản trị đơn hàng.

---

## 2. Phạm vi kiểm thử (Scope)

### 2.1 In Scope — Các module sẽ được kiểm thử

| STT | Module | Mã SRS | Lý do ưu tiên |
|-----|--------|--------|---------------|
| 1 | Xác thực người dùng (Đăng ký / Đăng nhập) | F1 | Nền tảng của mọi module khác; Google OAuth có nhiều edge case |
| 2 | Quản lý sản phẩm (Admin) | F2 | Cần có dữ liệu sản phẩm để test các module phía dưới |
| 3 | Giỏ hàng | F4 | Nghiệp vụ phức tạp: guest vs logged-in, phí ship, áp dụng coupon |
| 4 | Thanh toán & Đặt hàng | F5 | Luồng tiền — critical nhất trong hệ thống |
| 5 | Quản lý đơn hàng (Admin) | F10 | State machine Pending→Processing→Shipped→Delivered, concurrency |
| 6 | Mã giảm giá | F11 | Nhiều điều kiện biên: loại giảm, thời hạn, giới hạn lượt dùng |

### 2.2 Out of Scope — Các module sẽ không được kiểm thử trong đợt này

| Module | Mã SRS | Lý do loại trừ |
|--------|--------|----------------|
| Quản lý danh mục | F3 | Ít rủi ro nghiệp vụ, không liên quan trực tiếp đến luồng tiền |
| Quản lý Profile | F6 | Chức năng phụ trợ, không ảnh hưởng đến luồng mua hàng |
| Lịch sử mua hàng | F7 | Phụ thuộc kết quả F5, có thể kiểm tra thủ công ở bước sau |
| Phân quyền (RBAC) | F8 | Cần môi trường test phức tạp hơn, dự kiến mở rộng sang đợt 2 |
| Quản lý người dùng | F9 | Ưu tiên thấp trong timeline hiện tại |
| Kiểm duyệt nội dung | F12 | Không liên quan đến luồng tiền, rủi ro thấp |
| Cấu hình hệ thống | F13 | Thường chỉ cần test một lần khi setup môi trường |
| Thống kê & Báo cáo | F14 | Phụ thuộc dữ liệu đơn hàng, kiểm tra sau khi F5/F10 đã pass |
| Live Chat | F15 | Tính năng bổ trợ, không ảnh hưởng đến core business |
| Testimonials | F16 | Tính năng nội dung, rủi ro thấp |
| Điểm tích lũy | F17 | Tính năng mở rộng, chưa critical ở giai đoạn đầu |
| Thanh toán VNPay / SePay | F5 (một phần) | Yêu cầu tài khoản merchant thật và môi trường sandbox riêng; chưa có trong giai đoạn này |
| Google OAuth | F1 (một phần) | Yêu cầu cấu hình Google Cloud Console; test thủ công luồng email/password trước |
| Kiểm thử trên mobile | — | Ngoài phạm vi SRS giai đoạn 1 |
| Kiểm thử hiệu năng / load test | — | Không thuộc phạm vi kiểm thử chức năng đợt này |

---

## 3. Mục tiêu kiểm thử (Test Objectives)

Xác nhận rằng 6 module được chọn (F1, F2, F4, F5, F10, F11) hoạt động đúng theo các yêu cầu chức năng mô tả trong SRS v2.0, bao gồm các luồng chính (happy path), luồng ngoại lệ (alternative flow) và các trường hợp biên (boundary/edge case), với tỷ lệ pass đạt tối thiểu **90%** tổng số test case thực thi.

---

## 4. Nguồn lực (Resources)

### 4.1 Con người

| Vai trò | Họ tên | Trách nhiệm |
|---------|--------|-------------|
| Tester | Nguyễn Quang Huy | Viết test case, thực thi, log bug, viết báo cáo |

### 4.2 Môi trường kiểm thử

| Thành phần | Chi tiết |
|------------|---------|
| **Hệ điều hành** | Windows 11 |
| **Trình duyệt** | Brave (Chromium-based, tương đương Chrome) |
| **Ứng dụng** | Fruitables chạy local (localhost) trên IIS Express / .NET 8 |
| **Cơ sở dữ liệu** | SQL Server (local instance) |
| **Môi trường** | Development / Local — không phải production |

### 4.3 Công cụ

| Công cụ | Mục đích |
|---------|---------|
| **Microsoft Excel** | Viết và quản lý test case |
| **Jira Cloud** (miễn phí) | Log bug, theo dõi trạng thái bug |
| **GitHub** | Lưu trữ tài liệu test, test case, báo cáo |
| **Snipping Tool / ShareX** | Chụp màn hình làm evidence |

---

## 5. Lịch trình (Schedule)

| Giai đoạn | Công việc | Thời gian |
|-----------|-----------|-----------|
| **Chuẩn bị** | Viết test plan, setup môi trường, tạo dữ liệu test | 09/10 – 12/10/2026 |
| **Tuần 1** | Viết + thực thi test case F1 (Auth) và F2 (Product Admin) | 13/10 – 19/10/2026 |
| **Tuần 2** | Viết + thực thi test case F4 (Cart) và F11 (Coupon) | 20/10 – 26/10/2026 |
| **Tuần 3** | Viết + thực thi test case F5 (Checkout) | 27/10 – 02/11/2026 |
| **Tuần 4** | Viết + thực thi test case F10 (Order Admin) | 03/11 – 09/11/2026 |
| **Hoàn thiện** | Regression test các bug đã fix, viết báo cáo tổng kết | 10/11 – 15/11/2026 |
| **Nộp** | Deadline nộp báo cáo | **Giữa tháng 11/2026** |

---

## 6. Tiêu chí hoàn thành (Exit Criteria)

Đợt kiểm thử được coi là **hoàn tất** khi đáp ứng đủ các điều kiện sau:

| # | Tiêu chí |
|---|---------|
| 1 | 100% test case đã được thực thi (không còn test case ở trạng thái "Not Run") |
| 2 | Tỷ lệ pass đạt tối thiểu **90%** tổng số test case |
| 3 | Không còn bug nào ở mức **Critical** hoặc **High** đang ở trạng thái Open |
| 4 | Tất cả bug đã fix đều được **retest** và đóng (Closed/Verified) |
| 5 | Báo cáo kết quả kiểm thử (Test Report) đã được hoàn thiện và lưu vào repo |

> **Entry Criteria** (điều kiện để bắt đầu test): Môi trường local chạy ổn định, dữ liệu test đã được seed vào database, test case đã được review.

---

## 7. Lịch sử thay đổi

| Phiên bản | Ngày | Người sửa | Nội dung thay đổi |
|-----------|------|-----------|-------------------|
| v1.0 | 09/10/2026 | Nguyễn Quang Huy | Bản đầu tiên |
