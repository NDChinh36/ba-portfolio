# Use Case Specification: Chuyển Tiền Nội Bộ Ví MoMo

---

## Thông tin tài liệu

| Trường | Nội dung |
|---|---|
| **Document ID** | UC-MOMO-TRANSFER-001 |
| **Version** | 1.0 |
| **Status** | Draft |
| **Ngày tạo** | 2026-05-25 |
| **Ngày cập nhật** | 2026-05-25 |
| **Tác giả** | BA Team |
| **Reviewer** | Tech Lead, QA Lead |
| **Approver** | Head of Product |
| **Tài liệu liên quan** | BRD-MOMO-TRANSFER-001, FRD-MOMO-TRANSFER-001, US-MOMO-TRANSFER-001, ERD-MOMO-TRANSFER-001 |
| **Bảo mật** | Internal — Không phát hành ra ngoài |

## Người đọc dự kiến

| Vai trò | Mục đích |
|---|---|
| Tech Lead / Developer | Hiểu luồng tương tác để thiết kế hệ thống |
| QA Lead | Xây dựng test scenarios và test cases |
| Product Manager | Review scope hành vi hệ thống |
| UX Designer | Hiểu step-by-step interaction để thiết kế UI |

## Lịch sử thay đổi

| Phiên bản | Ngày | Tác giả | Nội dung thay đổi |
|---|---|---|---|
| 1.0 | 2026-05-25 | BA Team | Khởi tạo tài liệu |

---

## 1. Danh sách Use Case

| ID | Tên Use Case | Actor chính | Độ ưu tiên |
|---|---|---|---|
| UC-01 | Chuyển tiền P2P — Luồng cơ bản | Người dùng cá nhân | Must Have |
| UC-02 | Chuyển tiền P2P — Xác thực 2FA/3FA | Người dùng cá nhân, Auth Service | Must Have |
| UC-03 | Giải ngân đơn lẻ (Business → Cá nhân) | Maker, Checker, Business System | Must Have |
| UC-04 | Giải ngân hàng loạt — Batch Disbursement | Maker, Checker, Batch Processor | Must Have |
| UC-05 | Tra cứu lịch sử giao dịch | Người dùng cá nhân / Doanh nghiệp | Should Have |

---

## 2. UC-01: Chuyển Tiền P2P — Luồng Cơ Bản

### 2.1 Tổng quan

| Trường | Nội dung |
|---|---|
| **Use Case ID** | UC-01 |
| **Tên** | Chuyển tiền P2P — Luồng cơ bản |
| **Actor chính** | Người dùng cá nhân (Sender) |
| **Actor phụ** | Wallet Core, Notification Service, Limit Engine |
| **Preconditions** | Sender đã đăng nhập; ví Sender active; balance đủ |
| **Postconditions** | Balance Sender giảm; balance Recipient tăng; transaction log created |
| **Trigger** | Sender nhấn "Chuyển tiền" trên màn hình chính |

### 2.2 Main Flow

| Bước | Actor | Hành động |
|---|---|---|
| 1 | Sender | Nhấn "Chuyển tiền" |
| 2 | System | Hiển thị màn hình nhập thông tin người nhận |
| 3 | Sender | Nhập SĐT / quét QR / tìm tên người nhận |
| 4 | System | Tra cứu Recipient theo định danh; kiểm tra trạng thái active |
| 5 | System | Hiển thị tên đầy đủ + avatar Recipient để Sender xác nhận |
| 6 | Sender | Xác nhận đúng người nhận → nhấn "Tiếp tục" |
| 7 | Sender | Nhập số tiền |
| 8 | System | Gọi Limit Engine: kiểm tra hạn mức GD, ngày, tháng |
| 9 | System | Xác định mức xác thực cần thiết (1FA / 2FA / 3FA) |
| 10 | Sender | Hoàn thành xác thực theo yêu cầu |
| 11 | System | Hiển thị màn hình xác nhận giao dịch (số tiền, người nhận, phí) |
| 12 | Sender | Nhấn "Xác nhận chuyển" |
| 13 | System | Gọi Wallet Core: atomic debit Sender + credit Recipient |
| 14 | System | Tạo transaction record; ghi audit log |
| 15 | System | Gửi push notification cho cả Sender và Recipient |
| 16 | System | Hiển thị màn hình thành công với transaction ID |

### 2.3 Alternative Flows

| ID | Kích hoạt tại bước | Mô tả |
|---|---|---|
| AF-01 | Bước 3 | Sender dùng QR code: camera mở → scan → auto-populate SĐT |
| AF-02 | Bước 3 | Sender tìm theo tên: hiển thị danh sách matches → Sender chọn |
| AF-03 | Bước 6 | Sender nhấn "Sai người — thay đổi" → quay về bước 3 |
| AF-04 | Bước 7 | Sender nhập ghi chú (optional, max 100 ký tự) |

### 2.4 Exception Flows

| ID | Điều kiện | Hệ thống phản hồi |
|---|---|---|
| EX-01 | Bước 4: Recipient không tồn tại hoặc không active | Hiển thị "Số điện thoại chưa đăng ký MoMo" → dừng |
| EX-02 | Bước 8: Vượt hạn mức GD (> 20M VND) | Hiển thị lỗi hạn mức cụ thể → reject |
| EX-03 | Bước 8: Vượt hạn mức ngày (> 100M VND/ngày) | Hiển thị số tiền còn lại trong ngày → reject |
| EX-04 | Bước 8: Ví chưa eKYC vượt 20M VND/tháng | Hiển thị thông báo + hướng dẫn eKYC |
| EX-05 | Bước 10: Xác thực thất bại 3 lần liên tiếp | Khóa tính năng chuyển tiền 30 phút; ghi log |
| EX-06 | Bước 13: Wallet Core timeout > 30s | Rollback; hiển thị "Giao dịch thất bại — thử lại" |
| EX-07 | Bước 13: Sender balance không đủ tại thời điểm debit | Hiển thị lỗi số dư không đủ → dừng |

---

## 3. UC-02: Chuyển Tiền P2P — Xác Thực 2FA/3FA

### 3.1 Tổng quan

| Trường | Nội dung |
|---|---|
| **Use Case ID** | UC-02 |
| **Tên** | Xác thực đa yếu tố cho chuyển tiền |
| **Actor chính** | Người dùng cá nhân |
| **Actor phụ** | Auth Service, OTP Gateway |
| **Preconditions** | UC-01 bước 8 hoàn tất; mức xác thực đã được xác định |
| **Postconditions** | Auth token hợp lệ; giao dịch được phép tiếp tục |

### 3.2 Logic Xác Thực

| Điều kiện | Mức xác thực | Yếu tố bắt buộc |
|---|---|---|
| amount < 10M VND **VÀ** cumulative_1h < 20M VND | 1FA | PIN **hoặc** Face ID |
| amount ≥ 10M VND **VÀ** cumulative_1h < 20M VND | 2FA | PIN **VÀ** Face ID |
| (cumulative_1h + amount) ≥ 20M VND | 3FA | PIN **VÀ** Face ID **VÀ** OTP |

### 3.3 Main Flow — 3FA

| Bước | Actor | Hành động |
|---|---|---|
| 1 | System | Xác định level 3FA dựa trên amount + cumulative |
| 2 | System | Hiển thị màn hình nhập PIN (bước 1/3) |
| 3 | Sender | Nhập PIN 6 số |
| 4 | System | Verify PIN với Auth Service |
| 5 | System | Hiển thị màn hình Face ID (bước 2/3) |
| 6 | Sender | Thực hiện Face ID scan |
| 7 | System | Verify Face ID với Auth Service |
| 8 | System | Gọi OTP Gateway: gửi OTP 6 số qua SMS |
| 9 | System | Hiển thị màn hình nhập OTP (bước 3/3, countdown 5 phút) |
| 10 | Sender | Nhập OTP |
| 11 | System | Verify OTP: đúng → issue auth token; sai → xem EX |
| 12 | System | Trả auth token về UC-01 để tiếp tục bước 11 |

### 3.4 Exception Flows

| ID | Điều kiện | Hệ thống phản hồi |
|---|---|---|
| EX-AUTH-01 | PIN sai ≤ 2 lần | Thông báo sai + số lần còn lại |
| EX-AUTH-02 | PIN sai lần 3 | Khóa auth session; thoát luồng; ghi log security |
| EX-AUTH-03 | Face ID thất bại (lighting/liveness) | Cho phép retry tối đa 2 lần → fallback PIN nếu thiết lập |
| EX-AUTH-04 | OTP hết hạn (> 5 phút) | Nút "Gửi lại OTP"; tối đa 3 lần/phiên |
| EX-AUTH-05 | OTP sai lần 3 | Hủy giao dịch; reset auth session |
| EX-AUTH-06 | OTP Gateway timeout | Retry 1 lần tự động; nếu vẫn lỗi → thông báo thử lại sau |

---

## 4. UC-03: Giải Ngân Đơn Lẻ — Business Disbursement

### 4.1 Tổng quan

| Trường | Nội dung |
|---|---|
| **Use Case ID** | UC-03 |
| **Tên** | Giải ngân đơn lẻ (Business → Cá nhân) |
| **Actor chính** | Maker (nhân viên tạo lệnh), Checker (người phê duyệt) |
| **Actor phụ** | Wallet Core, Notification Service, Audit Logger |
| **Preconditions** | Business account active; số dư đủ; Maker khác Checker |
| **Postconditions** | Balance Business giảm; balance Recipient tăng; log tạo ra |
| **Trigger** | Maker nhấn "Tạo lệnh giải ngân" trên Business Portal |

### 4.2 Maker-Checker Constraint

> **BR-CRITICAL**: Maker và Checker **bắt buộc** là hai tài khoản khác nhau. Hệ thống tự động block nếu Maker cố approve lệnh của chính mình.

### 4.3 Main Flow — Maker-Checker

| Bước | Actor | Hành động |
|---|---|---|
| 1 | Maker | Đăng nhập Business Portal; chọn "Tạo lệnh giải ngân" |
| 2 | Maker | Nhập SĐT Recipient; nhập số tiền; nhập mô tả (bắt buộc) |
| 3 | System | Tra cứu Recipient; hiển thị tên để Maker xác nhận |
| 4 | System | Kiểm tra số dư Business Wallet |
| 5 | Maker | Xác nhận thông tin → nhấn "Gửi duyệt" |
| 6 | System | Tạo disbursement_request với status = PENDING_APPROVAL |
| 7 | System | Notify Checker qua email + in-app notification |
| 8 | Checker | Mở danh sách lệnh chờ duyệt; chọn lệnh của Maker |
| 9 | Checker | Review thông tin: Recipient, Amount, Description |
| 10 | Checker | Nhập PIN + OTP để xác thực (bắt buộc mọi GD) |
| 11 | System | Verify auth Checker |
| 12 | Checker | Nhấn "Phê duyệt" |
| 13 | System | Kiểm tra Maker ≠ Checker; nếu vi phạm → reject |
| 14 | System | Gọi Wallet Core: debit Business + credit Recipient (atomic) |
| 15 | System | Update status = COMPLETED; ghi audit log |
| 16 | System | Notify Maker (approved) + Recipient (đã nhận tiền) |

### 4.4 Alternative Flows

| ID | Kích hoạt | Mô tả |
|---|---|---|
| AF-DIS-01 | Bước 12 | Checker nhấn "Từ chối" → nhập lý do từ chối (bắt buộc) → status = REJECTED; notify Maker |
| AF-DIS-02 | Bước 8 | Checker có thể "Yêu cầu chỉnh sửa" → Maker nhận lại bản draft để sửa |

### 4.5 Exception Flows

| ID | Điều kiện | Hệ thống phản hồi |
|---|---|---|
| EX-DIS-01 | Bước 4: Số dư Business không đủ | Hiển thị lỗi; block tạo lệnh |
| EX-DIS-02 | Bước 3: Recipient không active | Hiển thị cảnh báo; Maker không thể tiếp tục |
| EX-DIS-03 | Bước 13: Maker = Checker (cùng account) | System reject tự động; ghi security log |
| EX-DIS-04 | Bước 14: Wallet Core timeout | Rollback debit; status = FAILED; notify cả Maker và Checker |
| EX-DIS-05 | Bước 10: Checker auth thất bại 3 lần | Khóa session Checker; lệnh vẫn ở PENDING → Checker khác có thể duyệt |

---

## 5. UC-04: Giải Ngân Hàng Loạt — Batch Disbursement

### 5.1 Tổng quan

| Trường | Nội dung |
|---|---|
| **Use Case ID** | UC-04 |
| **Tên** | Batch Disbursement |
| **Actor chính** | Maker, Checker |
| **Actor phụ** | Batch Processor, Wallet Core, Notification Service |
| **Preconditions** | Business account active; file CSV/XLSX hợp lệ; tổng số dư đủ |
| **Postconditions** | Các dòng SUCCESS được debit/credit; dòng FAILED được log |
| **Trigger** | Maker upload file batch lên Business Portal |

### 5.2 File Batch Format

| Cột | Kiểu | Bắt buộc | Validation |
|---|---|---|---|
| recipient_phone | String | ✅ | 10 số, bắt đầu 0 |
| amount | Integer | ✅ | 1,000 ≤ amount ≤ 20,000,000 |
| description | String | ✅ | Max 200 ký tự |
| reference_id | String | ✅ | Unique per file; idempotency key |

### 5.3 Main Flow — Batch

| Bước | Actor | Hành động |
|---|---|---|
| 1 | Maker | Upload file CSV/XLSX (tối đa 1,000 dòng) |
| 2 | System | Validate format file; đếm số dòng |
| 3 | System | Parse từng dòng: validate phone, amount, description |
| 4 | System | Hiển thị preview: tổng dòng, tổng tiền, số dòng lỗi |
| 5 | System | Kiểm tra tổng số tiền ≤ số dư Business Wallet |
| 6 | Maker | Xác nhận → nhấn "Gửi duyệt batch" |
| 7 | System | Tạo batch_job với status = PENDING_APPROVAL |
| 8 | Checker | Review summary batch: tổng tiền, số dòng, sample records |
| 9 | Checker | Xác thực PIN + OTP → nhấn "Phê duyệt" |
| 10 | System | Batch Processor xử lý từng dòng tuần tự (hoặc parallel theo cấu hình) |
| 11 | System | Mỗi dòng: atomic debit + credit + tạo transaction record |
| 12 | System | Cập nhật batch_job progress realtime |
| 13 | System | Sau khi hoàn tất: tạo summary report (success/failed/total) |
| 14 | System | Notify Maker: báo cáo kết quả batch |

### 5.4 Exception Flows — Batch

| ID | Điều kiện | Hệ thống phản hồi |
|---|---|---|
| EX-B-01 | Bước 2: File > 1,000 dòng | Reject upload; thông báo giới hạn |
| EX-B-02 | Bước 3: Dòng có lỗi format | Mark dòng đó INVALID; hiển thị preview với highlight lỗi; Maker sửa và re-upload |
| EX-B-03 | Bước 5: Tổng tiền > số dư | Reject toàn bộ batch; thông báo shortfall amount |
| EX-B-04 | Bước 11: Một dòng xử lý thất bại (recipient inactive, timeout) | Mark dòng đó FAILED; tiếp tục xử lý các dòng còn lại |
| EX-B-05 | Bước 10: Batch timeout sau 30 phút | Dừng xử lý; báo cáo trạng thái dòng đã xử lý và chưa xử lý |

---

## 6. UC-05: Tra Cứu Lịch Sử Giao Dịch

### 6.1 Tổng quan

| Trường | Nội dung |
|---|---|
| **Use Case ID** | UC-05 |
| **Tên** | Tra cứu lịch sử giao dịch |
| **Actor chính** | Người dùng cá nhân / Doanh nghiệp |
| **Preconditions** | Đã đăng nhập |
| **Postconditions** | Danh sách giao dịch hiển thị theo filter |

### 6.2 Main Flow

| Bước | Actor | Hành động |
|---|---|---|
| 1 | User | Mở tab "Lịch sử giao dịch" |
| 2 | System | Hiển thị 20 GD gần nhất (mặc định) |
| 3 | User | Áp dụng filter: khoảng thời gian, loại GD, trạng thái |
| 4 | System | Query theo filter + phân trang (20 GD/trang) |
| 5 | User | Chọn một GD → xem chi tiết |
| 6 | System | Hiển thị: transaction_id, thời gian, sender, recipient, amount, status, auth_level |
| 7 | User | (Tùy chọn) Tải xuống PDF/CSV |
| 8 | System | Generate file và download |

### 6.3 Exception Flows

| ID | Điều kiện | Hệ thống phản hồi |
|---|---|---|
| EX-HS-01 | Không có GD nào trong khoảng filter | Hiển thị trạng thái empty state + gợi ý mở rộng thời gian |
| EX-HS-02 | Export > 10,000 dòng | Cảnh báo; tự động giới hạn 10,000 dòng đầu tiên |

---

## 7. Business Rules Tổng Hợp

| ID | Quy tắc | Áp dụng tại UC |
|---|---|---|
| BR-UC-01 | Giao dịch tức thì, không hoàn sau confirm | UC-01, UC-03, UC-04 |
| BR-UC-02 | Idempotency key bắt buộc mọi debit request | UC-01, UC-03, UC-04 |
| BR-UC-03 | Maker ≠ Checker (cùng user_id) | UC-03, UC-04 |
| BR-UC-04 | Business account: PIN + OTP mọi GD (không ngưỡng) | UC-03, UC-04 |
| BR-UC-05 | Log tối thiểu: transaction_id, timestamp, sender, recipient, amount, auth_level, status | Tất cả UC |
| BR-UC-06 | Hiển thị tên Recipient trước khi Sender confirm | UC-01, UC-03 |
| BR-UC-07 | Timeout 30 giây cho Wallet Core; không retry tự động | UC-01, UC-03, UC-04 |

---

*Tài liệu này chỉ dành cho nội bộ. Không sao chép hoặc chia sẻ ra ngoài khi chưa có sự phê duyệt của Head of Product.*
