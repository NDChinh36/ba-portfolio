# BRD: Chuyển Tiền Nội Bộ Ví MoMo

---

## Thông tin tài liệu

| Trường | Nội dung |
|---|---|
| **Document ID** | BRD-MOMO-TRANSFER-001 |
| **Version** | 1.0 |
| **Status** | Draft |
| **Ngày tạo** | 2026-05-25 |
| **Ngày cập nhật** | 2026-05-25 |
| **Tác giả** | BA Team |
| **Reviewer** | Product Manager, Compliance Lead |
| **Approver** | Head of Product |
| **Bảo mật** | Internal — Không phát hành ra ngoài |

## Người đọc dự kiến

| Vai trò | Mục đích |
|---|---|
| Head of Product | Phê duyệt scope và mục tiêu kinh doanh |
| Product Manager | Align yêu cầu nghiệp vụ |
| Compliance / Legal | Xác minh tuân thủ TT23/2019/TT-NHNN |
| Tech Lead | Hiểu context trước khi thiết kế kỹ thuật |
| QA Lead | Nắm business constraints cho test plan |

## Lịch sử thay đổi

| Phiên bản | Ngày | Tác giả | Nội dung thay đổi |
|---|---|---|---|
| 1.0 | 2026-05-25 | BA Team | Khởi tạo tài liệu |

---

## 1. Executive Summary

Tính năng Chuyển Tiền Nội Bộ cho phép các ví MoMo giao dịch trực tiếp với nhau trong hệ sinh thái, không qua ngân hàng trung gian. Bao gồm hai luồng chính: **P2P** (cá nhân → cá nhân) và **Business Disbursement** (doanh nghiệp → cá nhân). Toàn bộ tuân thủ Thông tư 23/2019/TT-NHNN về giới hạn giao dịch ví điện tử và xác minh danh tính.

---

## 2. Business Objectives

| # | Mục tiêu | KPI | Target |
|---|---|---|---|
| OBJ-01 | Cung cấp kênh chuyển tiền nội bộ tức thì, không phí | Tỷ lệ giao dịch thành công | ≥ 99.5% |
| OBJ-02 | Tuân thủ quy định SBV — không vi phạm hạn mức | Số vi phạm hạn mức/tháng | 0 |
| OBJ-03 | Giảm tỷ lệ bỏ giữa chừng do xác thực phức tạp | Drop-off rate tại màn auth | ≤ 5% |
| OBJ-04 | Hỗ trợ doanh nghiệp giải ngân hàng loạt | Thời gian xử lý batch 1,000 GD | ≤ 30 phút |
| OBJ-05 | Tăng tần suất sử dụng ví | DAU sử dụng tính năng chuyển tiền | +20% sau 3 tháng |

---

## 3. Stakeholders

| Vai trò | Đại diện | Trách nhiệm |
|---|---|---|
| Business Owner | Head of Product MoMo | Approve scope và business priority |
| Business Analyst | BA Team | Định nghĩa yêu cầu, viết tài liệu |
| Compliance / Legal | Legal Team | Đảm bảo tuân thủ TT23/2019/TT-NHNN |
| Tech Lead | Engineering Lead | Thiết kế kỹ thuật, estimate |
| Security | Security Team | Review authentication, fraud prevention |
| QA Lead | QA Team | Test strategy, acceptance testing |
| Risk Team | Risk & Fraud Team | Định nghĩa ngưỡng nghi ngờ, auto-block rules |
| Người dùng cá nhân | End User (Consumer) | Gửi và nhận tiền P2P |
| Doanh nghiệp | Business Account Holder | Giải ngân lương, thưởng, hoàn tiền |
| Cơ quan quản lý | NHNN | Kiểm tra tuân thủ, nhận báo cáo định kỳ |

---

## 4. Scope

### 4.1 In Scope — v1.0

- Cá nhân → Cá nhân (P2P): chuyển tiền qua SĐT / QR code / tìm tên
- Doanh nghiệp → Cá nhân (Disbursement): giải ngân đơn lẻ và batch (tối đa 1,000 dòng/lần)
- Authentication 3 module (PIN, Face ID, OTP) với logic ngưỡng
- Kiểm tra và enforce hạn mức realtime theo TT23/2019
- Notification người gửi và người nhận (push, SMS)
- Lịch sử giao dịch, tra cứu, tải về

### 4.2 Out of Scope — v1.0

| Tính năng | Lý do loại trừ |
|---|---|
| Cá nhân → Merchant | Thuộc luồng "Thanh toán" — tài liệu riêng |
| System → Cá nhân (cashback, promotion) | Automated flow riêng, trigger khác |
| Chuyển tiền xuyên biên giới / ngoại tệ | Yêu cầu license riêng, phạm vi mở rộng sau |
| API cho bên thứ ba ngoài hệ sinh thái | Cần security review riêng, v2 |
| Hoàn tiền (reversal) tự động | GD tức thì không hoàn — tranh chấp xử lý qua CS |

---

## 5. Business Requirements

| ID | Mô tả yêu cầu | Ưu tiên |
|---|---|---|
| BR-01 | Hạn mức tối thiểu 1 GD: 1,000 VND | Must Have |
| BR-02 | Hạn mức tối đa 1 GD: 20,000,000 VND (TT23/2019 Điều 9) | Must Have |
| BR-03 | Hạn mức ngày: 100,000,000 VND/ngày/ví | Must Have |
| BR-04 | Hạn mức tháng (ví đã eKYC): 200,000,000 VND/tháng | Must Have |
| BR-05 | Hạn mức tháng (ví chưa eKYC): 20,000,000 VND/tháng (tổng GD) | Must Have |
| BR-06 | Xác thực 2FA bắt buộc khi GD ≥ 10,000,000 VND | Must Have |
| BR-07 | Xác thực 3FA bắt buộc khi cộng dồn ≥ 20,000,000 VND trong 60 phút | Must Have |
| BR-08 | GD tức thì, không thể hoàn sau khi hệ thống confirm | Must Have |
| BR-09 | Sender và recipient là tài khoản MoMo active | Must Have |
| BR-10 | Hiển thị tên recipient trước khi confirm — bắt buộc | Must Have |
| BR-11 | Log đầy đủ mọi GD cho mục đích audit và báo cáo NHNN | Must Have |
| BR-12 | Business account bắt buộc PIN + OTP mọi GD (không có ngưỡng tối thiểu) | Must Have |

---

## 6. Assumptions

- eKYC đã được xây dựng sẵn; tính năng này tái sử dụng kết quả xác minh
- Notification service (push, SMS) đã có sẵn và ổn định
- Wallet Core xử lý atomic debit/credit đảm bảo tính nhất quán dữ liệu
- Business account đã có quy trình onboarding và xác minh riêng

---

## 7. Constraints

- Giao dịch phải hoàn tất trong 30 giây (timeout) — không retry tự động sau confirm
- Không duplicate debit: bắt buộc idempotency key cho mọi request
- Log tối thiểu: transaction_id, timestamp, sender, recipient, amount, auth_level, status
- Hệ thống phải sẵn sàng báo cáo giao dịch cho NHNN theo định kỳ

---

## 8. Dependencies

| Hệ thống | Loại | Mô tả |
|---|---|---|
| eKYC Service | Internal | Kiểm tra trạng thái xác minh danh tính |
| Wallet Core | Internal | Atomic debit/credit, kiểm tra balance |
| Auth Service | Internal | PIN, Face ID, OTP verification |
| Limit Engine | Internal | Tính toán và enforce hạn mức realtime |
| Notification Service | Internal | Push notification, SMS |
| Audit Logger | Internal | Log giao dịch phục vụ compliance |
| NHNN Reporting Module | Internal/External | Báo cáo định kỳ theo quy định |

---

## 9. Open Questions

| # | Câu hỏi | Owner | Deadline |
|---|---|---|---|
| OQ-01 | Hạn mức tháng cho Business account là bao nhiêu? TT23 có áp dụng khác không? | Compliance | Sprint 1 |
| OQ-02 | Tài khoản mới (< 30 ngày) có áp dụng cooling period / hạn mức thấp hơn không? | Risk Team | Sprint 1 |
| OQ-03 | Nếu recipient bị suspend tại thời điểm nhận — tiền về sender ngay hay giữ pending? | Tech Lead | Sprint 1 |
| OQ-04 | Business batch disbursement: Maker-Checker mặc định bật hay tắt? Ai cấu hình? | Product | Sprint 1 |
| OQ-05 | GD nghi ngờ gian lận — tự động block hay alert manual review? | Risk Team | Sprint 2 |

---

*Tài liệu này chỉ dành cho nội bộ. Không sao chép hoặc chia sẻ ra ngoài khi chưa có sự phê duyệt của Head of Product.*
