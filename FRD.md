# FRD: Chuyển Tiền Nội Bộ Ví MoMo

---

## Thông tin tài liệu

| Trường | Nội dung |
|---|---|
| **Document ID** | FRD-MOMO-TRANSFER-001 |
| **Version** | 1.0 |
| **Status** | Draft |
| **Ngày tạo** | 2026-05-25 |
| **Ngày cập nhật** | 2026-05-25 |
| **Tác giả** | BA Team |
| **Reviewer** | Tech Lead, QA Lead, Security Team |
| **Approver** | Product Manager |
| **Tài liệu liên quan** | BRD-MOMO-TRANSFER-001 |
| **Bảo mật** | Internal — Không phát hành ra ngoài |

## Người đọc dự kiến

| Vai trò | Mục đích |
|---|---|
| Tech Lead / Developer | Thiết kế và implement tính năng |
| QA Lead / Tester | Xây dựng test cases, acceptance testing |
| Security Team | Review authentication và security flows |
| Product Manager | Validate yêu cầu chức năng |

## Lịch sử thay đổi

| Phiên bản | Ngày | Tác giả | Nội dung thay đổi |
|---|---|---|---|
| 1.0 | 2026-05-25 | BA Team | Khởi tạo tài liệu |

---

## 1. Overview

FRD này mô tả chi tiết chức năng của tính năng Chuyển Tiền Nội Bộ MoMo, bao gồm ba module chính:
- **2.1** — Chuyển tiền P2P (Cá nhân → Cá nhân)
- **2.2** — Authentication Flow (áp dụng cho mọi GD)
- **2.3** — Business Disbursement (Doanh nghiệp → Cá nhân)

---

## 2. User Roles & Permissions

| Role | Quyền trong tính năng này |
|---|---|
| Cá nhân — ví đã eKYC | Gửi tối đa 20M/GD, 100M/ngày, 200M/tháng; nhận không giới hạn |
| Cá nhân — ví chưa eKYC | Gửi/nhận tổng ≤ 20M/tháng; tối đa 20M/GD |
| Business Account (Maker) | Tạo GD đơn lẻ và batch; không thể tự approve nếu bật Maker-Checker |
| Business Account (Checker) | Approve/reject GD do Maker tạo; không phải là Maker của cùng GD |
| Admin / Operations | Tra cứu, xử lý tranh chấp; không tạo được GD |

---

## 3. Module 2.1 — Chuyển tiền P2P

### 3.1 Happy Path

| Bước | Hành động |
|---|---|
| 1 | Sender mở app → chọn "Chuyển tiền" |
| 2 | Nhập thông tin người nhận: SĐT / quét QR / tìm theo tên |
| 3 | Hệ thống hiển thị ảnh đại diện + tên recipient để xác nhận |
| 4 | Sender confirm đúng người |
| 5 | Nhập số tiền (1,000 – 20,000,000 VND) và nội dung (tùy chọn) |
| 6 | Hệ thống kiểm tra realtime: balance, hạn mức ngày/tháng |
| 7 | Xác thực theo ngưỡng (xem Module 2.2) |
| 8 | Hệ thống thực hiện atomic: debit sender → credit recipient |
| 9 | Notification cả 2 bên (push + SMS nếu cấu hình) |
| 10 | Hiển thị màn hình thành công + mã GD |

### 3.2 Alternate Flows

| ID | Điều kiện | Xử lý |
|---|---|---|
| AF-01 | SĐT không tìm thấy trong hệ thống | Hiển thị "Số điện thoại chưa đăng ký MoMo" — gợi ý mời bạn |
| AF-02 | Sender muốn sửa thông tin trước khi confirm | Cho phép quay lại bước nhập recipient bất kỳ lúc nào trước khi xác thực |
| AF-03 | Recipient chưa eKYC, amount > 5,000,000 VND | Hiển thị cảnh báo cho sender; sender có quyền tiếp tục hoặc huỷ |
| AF-04 | Sender gần đạt hạn mức tháng (còn < 10%) | Banner cảnh báo khi mở màn hình chuyển tiền |

### 3.3 Exception Flows

| ID | Lỗi / Điều kiện | Thông báo hiển thị | Xử lý hệ thống |
|---|---|---|---|
| EX-01 | Balance không đủ | "Số dư ví không đủ. Số dư hiện tại: [X] VND." | Không thực hiện GD |
| EX-02 | Vượt hạn mức ngày | "Bạn đã đạt hạn mức chuyển tiền hôm nay (100 triệu/ngày)." | Block GD |
| EX-03 | Vượt hạn mức tháng (chưa eKYC) | "Đã đạt hạn mức 20 triệu/tháng. Xác minh danh tính để tăng hạn mức." | Block GD, link đến eKYC |
| EX-04 | Vượt hạn mức tháng (đã eKYC) | "Đã đạt hạn mức 200 triệu/tháng theo quy định NHNN." | Block GD |
| EX-05 | Timeout > 30 giây | "Giao dịch chưa được xử lý. Kiểm tra lịch sử trước khi thử lại." | Không retry tự động; trạng thái GD = PENDING_REVIEW |
| EX-06 | Recipient account bị suspended | "Không thể chuyển tiền đến tài khoản này lúc này." | Block GD; không tiết lộ lý do cụ thể |
| EX-07 | Xác thực thất bại 3 lần liên tiếp | "Giao dịch bị tạm khóa 15 phút do nhập sai nhiều lần." | Lock GD 15 phút; log security event |
| EX-08 | Sender = Recipient | "Không thể chuyển tiền cho chính mình." | Block ngay ở bước validate |

### 3.4 Business Rules

| ID | Rule |
|---|---|
| BR-P2P-01 | 1,000 VND ≤ amount ≤ 20,000,000 VND / GD |
| BR-P2P-02 | Tổng GD trong ngày ≤ 100,000,000 VND; reset lúc 00:00 giờ địa phương |
| BR-P2P-03 | Sender và recipient không được là cùng 1 tài khoản |
| BR-P2P-04 | GD là tức thì và không thể hoàn sau khi hệ thống confirm thành công |
| BR-P2P-05 | Bắt buộc hiển thị và xác nhận tên recipient trước khi tiến hành xác thực |
| BR-P2P-06 | Idempotency key bắt buộc — unique per sender per 24 giờ; tránh duplicate debit |

### 3.5 Edge Cases

| Trường hợp | Xử lý |
|---|---|
| Sender và recipient cùng submit GD cho nhau đồng thời | Mỗi GD xử lý độc lập; không block lẫn nhau |
| GD thành công nhưng notification service lỗi | GD vẫn hợp lệ; retry notification async; không rollback |
| Sender mất mạng sau khi confirm, trước khi nhận kết quả | Kiểm tra idempotency key khi reconnect; trả kết quả GD đã tồn tại; không tạo GD mới |
| QR code hết hạn | Thông báo "Mã QR không hợp lệ hoặc đã hết hạn"; cho quét lại |

### 3.6 Data Fields

| Field | Type | Required | Validation |
|---|---|---|---|
| `recipient_id` | String (UUID) | Yes | MoMo account ID; active; ≠ sender_id |
| `amount` | Long (VND) | Yes | 1,000 ≤ amount ≤ 20,000,000 |
| `description` | String | No | Max 200 ký tự; sanitize HTML/SQL |
| `idempotency_key` | UUID v4 | Yes | Generated client-side; unique per sender per 24h |
| `auth_token` | String (JWT) | Yes | Token từ Auth Service; expiry ≤ 5 phút |

---

## 4. Module 2.2 — Authentication Flow

### 4.1 Logic Ngưỡng Xác Thực

| Điều kiện | Auth bắt buộc |
|---|---|
| amount < 10,000,000 VND **VÀ** cumulative_1h < 20,000,000 VND | 1FA: PIN **hoặc** Face ID |
| amount ≥ 10,000,000 VND **VÀ** cumulative_1h < 20,000,000 VND | 2FA: PIN **và** Face ID (theo thứ tự) |
| cumulative_1h + amount ≥ 20,000,000 VND (bất kể amount) | 3FA: PIN **+** Face ID **+** OTP |

> **cumulative_1h** = tổng amount các GD thành công của sender trong 60 phút gần nhất tính đến thời điểm hiện tại.

### 4.2 Auth Module Specifications

| Module | Spec kỹ thuật | Failure Handling |
|---|---|---|
| **PIN** | 6 chữ số; hash bcrypt; không lưu plaintext | Sai 3 lần liên tiếp → lock 15 phút; log security event |
| **Face ID** | Liveness detection bắt buộc (chống ảnh tĩnh/video replay) | Thất bại 3 lần → fallback bắt buộc dùng PIN thay thế |
| **OTP** | 6 chữ số; sinh ngẫu nhiên; hết hạn sau 3 phút; gửi qua SMS đã đăng ký | Hết hạn → resend tối đa 3 lần / GD; sau 3 lần → huỷ GD |

### 4.3 Exception Flows — Auth

| ID | Lỗi | Thông báo | Xử lý |
|---|---|---|---|
| EX-A01 | Face ID không nhận diện được (ánh sáng kém, che mặt) | "Không nhận diện được khuôn mặt. Vui lòng thử lại ở nơi đủ ánh sáng." | Cho thử lại; sau 3 lần → fallback PIN |
| EX-A02 | SĐT nhận OTP không còn là SĐT đăng ký | "SĐT nhận OTP không khớp với tài khoản. Vui lòng cập nhật SĐT trước." | Block GD; hướng dẫn cập nhật SĐT |
| EX-A03 | Thiết bị không hỗ trợ Face ID | *(Không hiển thị lỗi)* | Bỏ qua module Face ID; PIN trở thành bắt buộc cho mọi ngưỡng |
| EX-A04 | OTP gửi thất bại (SMS service lỗi) | "Không gửi được mã OTP. Vui lòng thử lại sau." | Cho retry; sau 3 lần gửi thất bại → huỷ GD; alert ops |

---

## 5. Module 2.3 — Business Disbursement

### 5.1 Happy Path — Đơn lẻ

| Bước | Actor | Hành động |
|---|---|---|
| 1 | Maker | Đăng nhập Business Portal → chọn "Giải ngân đơn lẻ" |
| 2 | Maker | Nhập SĐT/MoMo ID recipient, amount, description, reference_id |
| 3 | System | Validate: recipient active, amount trong hạn mức, balance đủ |
| 4 | System | Hiển thị preview: tên recipient, số tiền, phí (nếu có) |
| 5 | Maker | Xác nhận thông tin |
| 6 | System | Yêu cầu PIN Business + OTP (bắt buộc mọi GD) |
| 7 | Maker | Nhập PIN + OTP thành công |
| 8 | System | Gửi request đến Checker (nếu bật Maker-Checker) |
| 9 | Checker | Nhận notification → review → bấm "Duyệt" |
| 10 | System | Atomic: debit Business wallet → credit ví cá nhân |
| 11 | System | Notification: push/SMS recipient; email xác nhận Maker |
| 12 | System | Hiển thị kết quả thành công + mã GD |

### 5.2 Happy Path — Batch

| Bước | Actor | Hành động |
|---|---|---|
| 1 | Maker | Chọn "Giải ngân hàng loạt" → upload file CSV/Excel theo template MoMo |
| 2 | System | Validate toàn bộ file: format, duplicate, hạn mức tổng |
| 3 | System | Hiển thị preview: tổng X dòng / tổng Y VND / Z dòng lỗi |
| 4 | Maker | Review preview; download file lỗi nếu cần sửa; confirm batch |
| 5 | System | Yêu cầu PIN + OTP (Maker) |
| 6 | Checker | Approve batch (nếu bật Maker-Checker) |
| 7 | System | Xử lý tuần tự từng dòng; cập nhật trạng thái realtime |
| 8 | System | Báo cáo kết quả: thành công X / thất bại Y + lý do từng dòng |

### 5.3 Exception Flows — Batch

| ID | Lỗi | Xử lý |
|---|---|---|
| EX-B01 | File có dòng trùng (recipient + amount giống nhau) | Flag warning; Maker quyết định giữ hoặc xóa dòng trùng trước khi confirm |
| EX-B02 | Một số dòng fail giữa chừng (recipient suspended, SĐT sai) | Tiếp tục các dòng còn lại; báo cáo kết quả từng dòng |
| EX-B03 | Tổng batch vượt hạn mức ngày | Reject toàn batch ngay lúc validate; không xử lý một phần |
| EX-B04 | Balance Business không đủ cho toàn batch | Reject toàn batch; hiển thị "Số dư thiếu [X] VND để thực hiện toàn bộ batch" |
| EX-B05 | Checker không duyệt sau 24 giờ | Tự động hủy batch; chuyển trạng thái EXPIRED; notify Maker |

### 5.4 Business Rules — Disbursement

| ID | Rule |
|---|---|
| BR-DIS-01 | Business account bắt buộc PIN + OTP mọi GD — không có ngưỡng tối thiểu |
| BR-DIS-02 | Batch file tối đa 1,000 dòng / lần upload |
| BR-DIS-03 | Khi bật Maker-Checker: GD chưa được Checker duyệt thì chưa thực hiện |
| BR-DIS-04 | Checker không được là Maker của cùng 1 GD |
| BR-DIS-05 | Partial commit không được phép trong batch — all-or-nothing per batch |
| BR-DIS-06 | reference_id là unique trong phạm vi business account |

### 5.5 Data Fields — Batch File

| Column | Type | Required | Validation |
|---|---|---|---|
| `recipient_phone` | String | Yes | 10 chữ số; định dạng SĐT VN hợp lệ |
| `amount` | Long (VND) | Yes | 1,000 ≤ amount ≤ 20,000,000 |
| `description` | String | No | Max 200 ký tự |
| `reference_id` | String | Yes | Unique trong batch; do business tự định nghĩa; max 50 ký tự |

---

## 6. Non-Functional Requirements

| Loại | Yêu cầu |
|---|---|
| Performance | Xử lý P2P hoàn tất ≤ 3 giây (p95); timeout tối đa 30 giây |
| Throughput | Hỗ trợ tối thiểu 500 GD P2P / giây trong giờ cao điểm |
| Availability | Uptime ≥ 99.9% (≤ 8.7 giờ downtime / năm) |
| Consistency | Atomic debit/credit — không mất tiền, không tạo tiền |
| Security | TLS 1.2+ cho mọi API call; encrypt at-rest cho PIN hash, OTP |
| Idempotency | Duplicate request với cùng idempotency_key trả về kết quả GD đã tồn tại, không tạo GD mới |
| Audit | Log đầy đủ: transaction_id, timestamp, sender_id, recipient_id, amount, auth_level, ip_address, device_id, status |

---

## 7. Open Questions

| # | Câu hỏi | Owner | Deadline |
|---|---|---|---|
| OQ-01 | Hạn mức tháng Business account? TT23 có áp dụng khác với business không? | Compliance | Sprint 1 |
| OQ-02 | Tài khoản mới (< 30 ngày) có cooling period / hạn mức thấp hơn không? | Risk Team | Sprint 1 |
| OQ-03 | Recipient bị suspend tại lúc nhận — tiền về sender ngay hay pending? | Tech Lead | Sprint 1 |
| OQ-04 | Maker-Checker: mặc định bật hay tắt? Ai config (admin MoMo hay business tự config)? | Product | Sprint 1 |
| OQ-05 | GD nghi ngờ gian lận — tự block hay alert manual review? Threshold nào? | Risk Team | Sprint 2 |

---

*Tài liệu này chỉ dành cho nội bộ. Không sao chép hoặc chia sẻ ra ngoài khi chưa có sự phê duyệt của Product Manager.*
