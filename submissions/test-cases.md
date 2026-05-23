# Test Cases — Bảng trường hợp kiểm thử

> **Hướng dẫn**: Viết tối thiểu **20 TC** phủ đủ các chức năng chính (REQ-01 → REQ-08).
> Xem [examples/sample-test-case.md](../examples/sample-test-case.md) để hiểu cách viết TC tốt.
> Tự tổ chức và phân nhóm test case theo cách hợp lý nhất.

| Thông tin | |
|---|---|
| **Nhóm** | `<!-- Tên nhóm -->` |
| **Ngày tạo** | `<!-- DD/MM/YYYY -->` |
| **Hệ thống** | https://stqa.rbc.vn |
| **Tham chiếu** | SRS v1.0 |

---

## Bước 1: Mô hình hóa miền đầu vào — Input Domain Modeling (IDM)

> 📖 **Textbook:** Chương 6 — *Input Domain Modeling*, Paul Ammann & Jeff Offutt.
>
> **Trước khi viết Test Case**, nhóm **phải** phân tích miền đầu vào bằng bảng IDM bên dưới.
> Mỗi chức năng cần xác định: **Đặc tính (Characteristic)**, **Phân vùng (Block/Partition)**, và **Giá trị đại diện (Value)**.

### IDM — Đăng nhập (REQ-01)

| Đặc tính (Characteristic) | Phân vùng (Block) | Giá trị đại diện (Value) | Kết quả mong đợi |
|---|---|---|---|
| Email có tồn tại trong DB? | Có | `librarian@library.com` | Đăng nhập thành công |
| | Không | `noone@email.com` | Thông báo lỗi |
| Mật khẩu có đúng? | Đúng | `admin123` | Đăng nhập thành công |
| | Sai | `wrongpass` | Thông báo lỗi |
| Ô nhập có rỗng? | Không rỗng | (giá trị bất kỳ) | Xử lý bình thường |
| | Rỗng | `""` | Thông báo "Vui lòng nhập..." |

### IDM — Tìm kiếm sách (REQ-03)

| Đặc tính (Characteristic) | Phân vùng (Block) | Giá trị đại diện (Value) | Kết quả mong đợi |
|---|---|---|---|
| Từ khóa có tồn tại trong DB? | Có (tên sách) | `"Flutter"` | Hiển thị sách chứa "Flutter" |
| | Có (tên tác giả) | `"Nguyễn"` | Hiển thị sách của tác giả Nguyễn |
| | Không | `"XYZ123"` | Danh sách rỗng |
| Phân biệt HOA/thường? | Chữ thường | `"flutter"` | Kết quả giống "Flutter" |
| | Chữ HOA | `"FLUTTER"` | Kết quả giống "Flutter" |

### IDM — Mượn sách (REQ-04, REQ-05)

| Đặc tính (Characteristic) | Phân vùng (Block) | Giá trị đại diện (Value) | Kết quả mong đợi |
|---|---|---|---|
| Trạng thái sách? | Có sẵn | BOOK001 | Cho phép mượn |
| | Đang mượn | BOOK003 | Không cho phép |
| | Thất lạc | BOOK007 | Không cho phép |
| Trạng thái thành viên? | Hoạt động | thanh.nguyen@email.com | Cho phép mượn |
| | Tạm ngưng | cu.le@email.com | Từ chối, thông báo lỗi |
| | Hết hạn | binh.pham@email.com | Từ chối, thông báo lỗi |
| Số sách đang mượn? | < 3 (BVA: 0, 1, 2) | Tài khoản mượn 0-2 cuốn | Cho phép mượn |
| | = 3 (BVA: giới hạn) | dam.tran@email.com (đã mượn đủ 3 cuốn) | Từ chối, thông báo vượt giới hạn |

#### 1. Bảng quyết định (Decision Table) cho REQ-04
| Điều kiện (Conditions) | Luật 1 (TC-13) | Luật 2 (TC-14) | Luật 3 (TC-15) | Luật 4 (TC-16) | Luật 5 (TC-17) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| 1. Tài khoản Member Active? | Đúng | Sai (Blocked) | Sai (Expired) | Đúng | Đúng |
| 2. Số sách đang mượn < 3? | Đúng | - | - | Sai ($\ge 3$) | Đúng |
| 3. Sách muốn mượn Available?| Đúng | - | - | - | Sai (Borrowed)|
| **Hành động (Actions)** | | | | | |
| - Cho phép mượn thành công | **X** | | | | |
| - Chặn lại & Báo lỗi tương ứng| | **X** | **X** | **X** | **X** |

#### 2. Danh sách các Test Cases chi tiết cho REQ-04
| ID | Chức năng / Mục đích (Description) | Các bước thực hiện (Steps) | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi (Expected Result) |
| :--- | :--- | :--- | :--- | :--- |
| **TC-13** | Kiểm tra mượn sách thành công (Luật 1) | 1. Đăng nhập bằng tài khoản Member đang Active.<br>2. Tìm một cuốn sách đang có trạng thái "Available" (Sẵn sàng) trên danh sách.<br>3. Nhấn nút "Mượn sách" (Borrow). | - Email: `thanh.nguyen@email.com`<br>- Password: `password123`<br>- Sách: `BOOK001` (Sách về Flutter) | - Hệ thống hiển thị thông báo (Toast/Popup) "Mượn sách thành công".<br>- Trạng thái của cuốn sách `BOOK001` lập tức đổi từ "Available" sang "Borrowed".<br>- Một bản ghi mới xuất hiện trong danh sách Phiếu mượn. |
| **TC-14** | Kiểm tra chặn mượn sách khi tài khoản bị Tạm ngưng (Luật 2) | 1. Đăng nhập bằng tài khoản bị Tạm ngưng.<br>2. Chọn một cuốn sách bất kỳ đang "Available".<br>3. Nhấn nút "Mượn sách". | - Email: `cu.le@email.com`<br>- Password: `password123`<br>- Sách: Bất kỳ | - Hệ thống chặn không cho mượn.<br>- Xuất hiện thông báo lỗi dạng Dialog/Toast: "Tài khoản đang bị tạm ngưng".<br>- Không có phiếu mượn nào được tạo ra. |
| **TC-15** | Kiểm tra chặn mượn sách khi tài khoản đã Hết hạn (Luật 3) | 1. Đăng nhập bằng tài khoản đã Hết hạn.<br>2. Chọn một cuốn sách bất kỳ đang "Available".<br>3. Nhấn nút "Mượn sách". | - Email: `binh.pham@email.com`<br>- Password: `password123`<br>- Sách: Bất kỳ | - Hệ thống chặn không cho mượn.<br>- Xuất hiện thông báo lỗi dạng Dialog/Toast: "Tài khoản đã hết hạn".<br>- Không có phiếu mượn nào được tạo ra. |
| **TC-16** | Kiểm tra chặn mượn sách khi vượt quá giới hạn 3 cuốn (Luật 4) | 1. Đăng nhập bằng tài khoản Member đang Active.<br>2. Tiến hành nhấn "Mượn sách" liên tiếp cho 3 cuốn sách khác nhau đang "Available".<br>3. Tìm chọn thêm cuốn sách thứ 4 đang "Available" và nhấn "Mượn sách". | - Email: `dam.tran@email.com`<br>- Password: `password123`<br>- 4 cuốn sách "Available" bất kỳ. | - Ở bước 2: Hệ thống cho phép mượn thành công 3 cuốn đầu.<br>- Ở bước 3: Hệ thống chặn không cho phép mượn cuốn thứ 4.<br>- Hiển thị thông báo lỗi rõ ràng trên màn hình: "Đã đạt giới hạn mượn tối đa (3 cuốn)". |
| **TC-17** | Kiểm tra chặn mượn cuốn sách đã có người mượn (Luật 5) | 1. Đăng nhập bằng tài khoản Member.<br>2. Tìm đến một cuốn sách bất kỳ đang có nhãn trạng thái là "Borrowed" (Đã mượn) hoặc "Lost" (Thất lạc).<br>3. Thực hiện hành động mượn sách. | - Email: `thanh.nguyen@email.com`<br>- Password: `password123`<br>- Sách: Cuốn sách đang ở trạng thái "Borrowed" hoặc "Lost" trên web. | - Nút "Mượn sách" của cuốn sách đó bị vô hiệu hóa (Bị xám màu/Disabled) không cho bấm HOẶC nếu bấm được thì hệ thống phải báo lỗi "Sách không sẵn sàng để mượn". |
### IDM — `<!-- Nhóm tự bổ sung cho REQ-05 đến REQ-08 -->`

| Đặc tính (Characteristic) | Phân vùng (Block) | Giá trị đại diện (Value) | Kết quả mong đợi |
|---|---|---|---|
| `<!-- Nhóm tự điền -->` | | | |

> 💡 **Gợi ý kỹ thuật**: Sử dụng **Phân lớp tương đương (EP)** cho các phân vùng rời rạc, **Phân tích giá trị biên (BVA)** cho các phân vùng số (ví dụ: giới hạn 3 sách). Xem textbook §6.1–6.3.

---

## Bước 2: Test Cases

<!-- Tự tổ chức bảng test case: có thể chia nhóm theo chức năng, theo REQ, hoặc theo luồng nghiệp vụ — tùy nhóm quyết định. -->
<!-- Mỗi TC phải ánh xạ ngược về ít nhất 1 dòng trong bảng IDM ở Bước 1. -->

| Mã TC | Mục tiêu kiểm thử | Tiền điều kiện | Bước thực hiện | Dữ liệu đầu vào | Kết quả mong đợi | REQ | Kỹ thuật |
|-------|-------------------|---------------|---------------|-----------------|------------------|-----|---------|
| | | | | | | | |

---

## Tổng hợp

| Nhóm chức năng | Số TC | REQ phủ | Kỹ thuật IDM áp dụng |
|----------------|-------|---------|----------------------|
| | | | |
| **Tổng** | **<!-- ≥ 20 -->** | | |
