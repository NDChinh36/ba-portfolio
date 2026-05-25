# User Stories: Chuyển Tiền P2P — Ví MoMo

---

## Thông tin tài liệu

| Trường | Nội dung |
|---|---|
| **Document ID** | US-MOMO-TRANSFER-001 |
| **Version** | 1.0 |
| **Status** | Draft |
| **Ngày tạo** | 2026-05-25 |
| **Ngày cập nhật** | 2026-05-25 |
| **Tác giả** | BA Team |
| **Reviewer** | Product Manager, QA Lead |
| **Approver** | Product Manager |
| **Sprint Target** | Sprint 1–2 |
| **Tài liệu liên quan** | BRD-MOMO-TRANSFER-001, FRD-MOMO-TRANSFER-001 |
| **Bảo mật** | Internal |

## Người đọc dự kiến

| Vai trò | Mục đích |
|---|---|
| Product Owner | Prioritization, sprint planning |
| Developer | Implement đúng AC |
| QA Tester | Xây dựng test cases từ AC |

## Lịch sử thay đổi

| Phiên bản | Ngày | Tác giả | Nội dung thay đổi |
|---|---|---|---|
| 1.0 | 2026-05-25 | BA Team | Khởi tạo tài liệu |

---

## Phạm vi

User Stories trong tài liệu này bao phủ luồng **P2P (Cá nhân → Cá nhân)**. Luồng Business Disbursement được mô tả trong UC-MOMO-TRANSFER-001.

---

## Story Map

| Epic | User Story | Priority | Points |
|---|---|---|---|
| Tìm người nhận | US-01: Tìm và xác nhận người nhận | Must Have | 3 |
| Nhập & kiểm tra | US-02: Nhập số tiền và kiểm tra hạn mức | Must Have | 2 |
| Xác thực | US-03: Xác thực giao dịch theo ngưỡng | Must Have | 5 |
| Hoàn tất & tra cứu | US-04: Xem kết quả và tra cứu lịch sử | Must Have | 2 |
| **Tổng** | **4 stories** | | **12 pts** |

---

## [US-01] Tìm và xác nhận người nhận

> **As a** MoMo user (đã đăng nhập),
> **I want to** tìm người nhận qua SĐT, QR code hoặc tên và xác nhận đúng người trước khi chuyển tiền,
> **So that** tôi không chuyển nhầm tiền cho người khác.

**Priority:** Must Have | **Story Points:** 3
**Dependencies:** User Service API (lookup by phone/QR)

---

**AC-01: Tìm qua SĐT — tìm thấy**
```
Given sender đang ở màn hình "Chuyển tiền"
When sender nhập SĐT 10 chữ số hợp lệ đã đăng ký MoMo
Then hệ thống hiển thị: ảnh đại diện + họ tên đầy đủ của recipient
And hiển thị nút "Xác nhận đúng người" và "Không phải người này"
And sender phải bấm "Xác nhận đúng người" trước khi tiếp tục
```

**AC-02: Tìm qua SĐT — không tìm thấy**
```
Given sender nhập SĐT 10 chữ số hợp lệ
When SĐT chưa đăng ký MoMo
Then hiển thị thông báo: "Số điện thoại này chưa đăng ký MoMo"
And hiển thị nút "Mời bạn dùng MoMo"
And không cho phép tiếp tục giao dịch
```

**AC-03: Quét QR — hợp lệ**
```
Given sender chọn "Quét QR"
When camera scan QR code hợp lệ của 1 MoMo user
Then tự động điền thông tin recipient
And hiển thị ảnh đại diện + tên để sender xác nhận
```

**AC-04: Quét QR — không hợp lệ**
```
Given sender đang quét QR
When QR code hết hạn, bị hỏng, hoặc không phải QR MoMo
Then hiển thị: "Mã QR không hợp lệ. Vui lòng thử lại."
And cho phép quét lại mà không thoát màn hình
```

**AC-05: Sender cố chuyển tiền cho chính mình**
```
Given sender nhập SĐT hoặc quét QR
When SĐT / tài khoản trùng với tài khoản đang đăng nhập
Then hiển thị: "Không thể chuyển tiền cho chính mình."
And chặn không cho tiếp tục
```

**Edge Cases:**

| Trường hợp | Hành vi mong đợi |
|---|---|
| Recipient account bị suspended | Hệ thống không hiển thị tên; thông báo "Không thể chuyển đến tài khoản này lúc này" |
| SĐT có ký tự đặc biệt hoặc khoảng trắng | Tự động trim và normalize trước khi lookup |
| Mạng chậm khi lookup | Hiển thị loading spinner; timeout sau 10 giây → thông báo lỗi mạng |

**Definition of Done:**
- [ ] Tất cả AC pass
- [ ] Unit test: lookup found / not found / suspended
- [ ] Không lộ thông tin nhạy cảm ngoài tên và ảnh đại diện

---

## [US-02] Nhập số tiền và kiểm tra hạn mức

> **As a** MoMo user,
> **I want to** nhập số tiền muốn chuyển và biết ngay tôi có thể thực hiện giao dịch không,
> **So that** tôi không phải đi qua bước xác thực rồi mới bị từ chối vì vượt hạn mức.

**Priority:** Must Have | **Story Points:** 2
**Dependencies:** Limit Engine API; Wallet Balance API

---

**AC-01: Kiểm tra realtime khi nhập**
```
Given sender đang nhập số tiền
When sender gõ từng ký tự
Then hiển thị realtime bên dưới ô nhập:
  - "Số dư hiện tại: [X] VND"
  - "Hạn mức ngày còn lại: [Y] VND"
```

**AC-02: Số tiền hợp lệ**
```
Given sender nhập amount trong khoảng 1,000 – 20,000,000 VND
And amount ≤ balance hiện tại
And amount ≤ hạn mức ngày còn lại
When sender bấm "Tiếp tục"
Then chuyển sang màn hình xác thực
```

**AC-03: Dưới hạn mức tối thiểu**
```
Given sender nhập amount < 1,000 VND
When sender bấm "Tiếp tục" hoặc bỏ focus
Then inline error: "Số tiền tối thiểu là 1,000 VND"
And disable nút "Tiếp tục"
```

**AC-04: Vượt hạn mức ngày**
```
Given sender đã gửi 90,000,000 VND hôm nay
When sender nhập amount = 15,000,000 VND
Then inline warning: "Vượt hạn mức ngày. Bạn chỉ còn có thể chuyển 10,000,000 VND hôm nay."
And disable nút "Tiếp tục"
```

**AC-05: Vượt hạn mức 1 GD**
```
Given sender nhập amount > 20,000,000 VND
When sender bấm "Tiếp tục" hoặc bỏ focus
Then inline error: "Số tiền tối đa 1 lần chuyển là 20,000,000 VND"
And disable nút "Tiếp tục"
```

**AC-06: Balance không đủ**
```
Given số dư ví của sender < amount nhập vào
When sender bấm "Tiếp tục"
Then inline error: "Số dư không đủ. Số dư hiện tại: [X] VND."
And disable nút "Tiếp tục"
```

**AC-07: Cảnh báo gần đạt hạn mức tháng (chưa eKYC)**
```
Given sender chưa eKYC, đã dùng > 90% hạn mức tháng (> 18,000,000 / 20,000,000 VND)
When sender mở màn hình chuyển tiền
Then hiển thị banner: "Bạn còn [X] VND trong hạn mức tháng. Xác minh danh tính để tăng lên 200 triệu/tháng."
And có link nhanh "Xác minh ngay"
```

**Edge Cases:**

| Trường hợp | Hành vi mong đợi |
|---|---|
| Sender nhập chữ thay vì số | Chỉ cho phép input số; tự format dấu phân cách nghìn |
| Sender nhập số âm | Không cho phép; hiển thị error |
| Hạn mức engine timeout | Hiển thị loading; nếu timeout sau 5s → hiển thị lỗi; không cho phép tiếp tục |

**Definition of Done:**
- [ ] Tất cả AC pass
- [ ] Kiểm tra edge case: balance = 0, hạn mức = 0, eKYC flag thay đổi
- [ ] Limit Engine integration test

---

## [US-03] Xác thực giao dịch theo ngưỡng

> **As a** MoMo user,
> **I want to** xác thực giao dịch với mức độ bảo mật phù hợp với số tiền,
> **So that** giao dịch nhỏ nhanh chóng trong khi giao dịch lớn được bảo vệ chặt chẽ hơn.

**Priority:** Must Have | **Story Points:** 5
**Dependencies:** Auth Service (PIN, Face ID, OTP modules)

---

**AC-01: GD nhỏ — 1 factor**
```
Given amount < 10,000,000 VND
And tổng GD trong 60 phút gần nhất < 20,000,000 VND
When hệ thống hiển thị màn hình xác thực
Then chỉ yêu cầu PIN hoặc Face ID (sender chọn)
And sau khi xác thực thành công → thực hiện GD
```

**AC-02: GD lớn — 2 factor**
```
Given amount ≥ 10,000,000 VND
And tổng GD trong 60 phút gần nhất + amount < 20,000,000 VND
When hệ thống hiển thị màn hình xác thực
Then yêu cầu PIN trước, sau đó Face ID (cả 2, theo thứ tự)
And nếu 1 trong 2 thất bại → toàn bộ xác thực fail
```

**AC-03: Cộng dồn vượt ngưỡng — 3 factor**
```
Given tổng GD trong 60 phút gần nhất + amount ≥ 20,000,000 VND
When hệ thống tính toán ngưỡng tại lúc sender confirm
Then yêu cầu PIN + Face ID + OTP (cả 3, theo thứ tự)
And OTP gửi về SĐT đăng ký của tài khoản; hết hạn sau 3 phút
```

**AC-04: Face ID thất bại — fallback PIN**
```
Given sender đang ở bước Face ID
When Face ID thất bại 3 lần liên tiếp
Then hệ thống chuyển sang yêu cầu nhập PIN thay thế
And hiển thị: "Không nhận diện được khuôn mặt. Vui lòng dùng mã PIN thay thế."
And log security event
```

**AC-05: OTP hết hạn**
```
Given sender đã nhận OTP
When sender nhập OTP sau 3 phút kể từ lúc gửi
Then thông báo: "Mã OTP đã hết hạn."
And hiển thị nút "Gửi lại OTP"
And cho phép resend tối đa 3 lần; sau 3 lần → hủy GD
```

**AC-06: Thiết bị không hỗ trợ Face ID**
```
Given thiết bị của sender không có tính năng Face ID
When hệ thống xác định thiết bị capabilities tại lúc khởi tạo session
Then bỏ qua Face ID module hoàn toàn
And PIN trở thành bắt buộc cho mọi ngưỡng (kể cả GD ≥ 10M)
```

**Edge Cases:**

| Trường hợp | Hành vi mong đợi |
|---|---|
| cumulative_1h vượt ngưỡng đúng tại thời điểm xác thực (do GD khác vừa thành công song song) | Nâng bậc auth realtime; yêu cầu OTP thêm |
| SMS OTP không gửi được (SMS service down) | Thông báo lỗi; cho retry; sau 3 lần fail → hủy GD; alert ops |
| Session xác thực hết hạn (> 5 phút từ lúc tạo) | Yêu cầu bắt đầu lại; không tự động retry GD |

**Definition of Done:**
- [ ] Tất cả AC pass với cả 3 ngưỡng auth
- [ ] Security test: brute force PIN, replay OTP, liveness check bypass attempt
- [ ] cumulative_1h tính đúng trong môi trường concurrent

---

## [US-04] Xem kết quả và tra cứu lịch sử giao dịch

> **As a** MoMo user,
> **I want to** nhận xác nhận tức thì sau khi chuyển tiền và tra cứu lịch sử bất kỳ lúc nào,
> **So that** tôi có bằng chứng GD và có thể kiểm tra khi cần.

**Priority:** Must Have | **Story Points:** 2
**Dependencies:** Transaction Service; Notification Service

---

**AC-01: GD thành công — màn hình xác nhận**
```
Given GD được Wallet Core xử lý thành công
When kết quả trả về
Then hiển thị màn hình thành công với:
  - Mã giao dịch (transaction_id)
  - Tên và ảnh recipient
  - Số tiền đã chuyển
  - Thời gian GD (giờ:phút:giây ngày/tháng/năm)
  - Nội dung chuyển tiền (nếu có)
And push notification đến sender và recipient trong vòng 5 giây
And GD xuất hiện trong lịch sử ngay lập tức
```

**AC-02: GD timeout — trạng thái không rõ**
```
Given GD timeout sau 30 giây không phản hồi
When sender xem màn hình kết quả
Then hiển thị: "Giao dịch đang được xử lý. Vui lòng kiểm tra lịch sử trước khi thực hiện lại."
And KHÔNG hiển thị "Thất bại" (vì GD có thể đã thực hiện)
And link trực tiếp đến lịch sử giao dịch
```

**AC-03: Tra cứu lịch sử**
```
Given user mở "Lịch sử giao dịch"
When xem danh sách
Then có thể filter theo: khoảng ngày, loại GD (gửi/nhận), trạng thái (thành công/thất bại/đang xử lý)
And mỗi dòng hiển thị: tên đối tác, số tiền, thời gian, trạng thái (badge màu)
And click vào dòng → xem chi tiết đầy đủ
```

**AC-04: Xem chi tiết GD**
```
Given user click vào 1 GD trong lịch sử
Then hiển thị: transaction_id, tên sender, tên recipient, số tiền, thời gian, nội dung, trạng thái, auth_level đã dùng
And có nút "Tải biên lai" (PDF hoặc ảnh) và "Chia sẻ"
```

**Edge Cases:**

| Trường hợp | Hành vi mong đợi |
|---|---|
| Notification lỗi nhưng GD thành công | GD vẫn hiện trong lịch sử; retry notification async; không rollback |
| Sender mất mạng ngay sau khi confirm | Khi reconnect, check idempotency_key → trả kết quả GD đã tồn tại |
| Recipient không nhận được notification | GD vẫn hợp lệ; recipient xem trong lịch sử; retry notification lên đến 3 lần |

**Definition of Done:**
- [ ] Tất cả AC pass
- [ ] Test idempotency: gửi cùng idempotency_key 2 lần → chỉ 1 GD được tạo
- [ ] Load test: lịch sử với > 10,000 GD vẫn load < 2 giây

---

*Tài liệu này chỉ dành cho nội bộ. Không sao chép hoặc chia sẻ ra ngoài khi chưa có sự phê duyệt của Product Manager.*
