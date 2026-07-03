# HW02 – BÁO CÁO KIỂM THỬ MIỀN TRỊ & GIÁ TRỊ BIÊN (EShop SUT)

---

## 1. Tính năng A: FR-03 – Quên mật khẩu và Đặt lại mật khẩu (Forgot Password and Password Reset)

### 1.1. Tổng quan tính năng & Yêu cầu
- **Mô tả:** Người dùng yêu cầu nhận mã đặt lại mật khẩu qua email. Sau đó dùng mã đó để nhập mật khẩu mới.
- **Các trường dữ liệu đầu vào:**
  - `email` (Chuỗi ký tự, định dạng email).
  - `resetToken` (Chuỗi chữ số, mã OTP gồm 4 ký tự).
  - `newPassword` (Chuỗi ký tự, độ dài và các ký tự bắt buộc).

---

### 1.2. Kiểm thử miền trị (Domain Testing)
#### 1.2.1. Phân tích miền trị từng bước
- **Bước 1: Phân hoạch tương đương (Equivalence Partitioning - EP):**
  - **Email:**
    - EP-EMAIL-VAL-01: Định dạng email hợp lệ, có tồn tại trong CSDL.
    - EP-EMAIL-INV-01: Định dạng email hợp lệ, không tồn tại trong CSDL.
    - EP-EMAIL-INV-02: Định dạng email không hợp lệ (ví dụ: thiếu `@`, thiếu tên miền).
    - EP-EMAIL-INV-03: Để trống email (chuỗi rỗng).
  - **Reset Token (OTP):**
    - EP-TOKEN-VAL-01: OTP trùng khớp hoàn toàn với OTP được tạo trong CSDL.
    - EP-TOKEN-INV-01: OTP không trùng khớp (nhập sai số).
    - EP-TOKEN-INV-02: Để trống mã OTP.
  - **Mật khẩu mới (New Password):**
    - EP-PASS-VAL-01: Độ dài từ 8-20 ký tự, chứa chữ hoa, chữ thường, số, ký tự đặc biệt.
    - EP-PASS-INV-01: Mật khẩu quá ngắn (< 8 ký tự).
    - EP-PASS-INV-02: Mật khẩu quá dài (> 20 ký tự).
    - EP-PASS-INV-03: Mật khẩu thiếu một trong các thành phần bắt buộc (chữ hoa, số, v.v.).
    - EP-PASS-INV-04: Để trống mật khẩu mới.

- **Bước 2: Xây dựng các ca kiểm thử bằng phương pháp 1-to-1 (kết hợp tối thiểu):**
  - Che phủ tất cả các phân hoạch hợp lệ bằng các kịch bản thành công.
  - Che phủ các phân hoạch không hợp lệ bằng cách test đơn lẻ từng lỗi một.

#### 1.2.2. Danh sách các ca kiểm thử miền trị (Domain Test Cases)
| Mã Test Case | Phân hoạch kiểm thử | Dữ liệu đầu vào (Input) | Kết quả mong đợi (Expected) | Kết quả thực tế (Actual) | Trạng thái |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **FR03-DT-01** | EP-EMAIL-VAL-01 | email: `test@eshop.com` | Trả về mã OTP 4 số thành công (HTTP 200) | Trả về mã OTP thành công, có kèm mã OTP trong JSON response | **PASS** (Có lỗi bảo mật) |
| **FR03-DT-02** | EP-EMAIL-INV-01 | email: `unknown@eshop.com` | Báo lỗi email không tồn tại (HTTP 404) | Phản hồi lỗi "User not found" (HTTP 404) | **PASS** |
| **FR03-DT-03** | EP-EMAIL-INV-02 | email: `invalid-email` | Báo lỗi định dạng email không hợp lệ | Hệ thống không báo lỗi định dạng mà báo "User not found" vì không tìm thấy email này trong DB | **FAIL** (Thiếu validation định dạng) |
| **FR03-DT-04** | EP-EMAIL-INV-03 | email: `""` (để trống) | Yêu cầu nhập email | Phản hồi lỗi "User not found" | **FAIL** |
| **FR03-DT-05** | EP-TOKEN-VAL-01, EP-PASS-VAL-01 | email: `test@eshop.com`<br>resetToken: `[OTP đúng]`<br>newPassword: `NewPass123!` | Đặt lại mật khẩu thành công (HTTP 200) | Đặt lại mật khẩu thành công | **PASS** |
| **FR03-DT-06** | EP-TOKEN-INV-01 | email: `test@eshop.com`<br>resetToken: `9999` (sai)<br>newPassword: `NewPass123!` | Báo lỗi mã xác thực không đúng (HTTP 400) | Phản hồi lỗi "Invalid token or email" | **PASS** |
| **FR03-DT-07** | EP-TOKEN-INV-02 | email: `test@eshop.com`<br>resetToken: `""`<br>newPassword: `NewPass123!` | Báo lỗi thiếu mã xác thực | Phản hồi lỗi "Invalid token or email" | **PASS** |
| **FR03-DT-08** | EP-PASS-INV-01 | email: `test@eshop.com`<br>resetToken: `[OTP đúng]`<br>newPassword: `123` | Báo lỗi mật khẩu quá ngắn (phải >= 8 ký tự) | Hệ thống vẫn cho phép đổi mật khẩu thành công ở backend mà không báo lỗi | **FAIL** (Thiếu validate độ dài) |
| **FR03-DT-09** | EP-PASS-INV-04 | email: `test@eshop.com`<br>resetToken: `[OTP đúng]`<br>newPassword: `""` (để trống) | Báo lỗi mật khẩu không được để trống | Đổi mật khẩu thành công về chuỗi rỗng | **FAIL** (Thiếu validate chuỗi rỗng) |

---

### 1.3. Phân tích giá trị biên (Boundary Value Analysis - BVA)
#### 1.3.1. Phân tích giá trị biên từng bước
- **Biến kiểm thử:** Độ dài của mật khẩu mới (`newPassword`).
- **Khoảng biên hợp lệ:** [8, 20] ký tự.
- **Các điểm biên xác định (3-point BVA):**
  - Cận dưới: 8. Các giá trị kiểm thử: 7 (Min-), 8 (Min), 9 (Min+).
  - Cận trên: 20. Các giá trị kiểm thử: 19 (Max-), 20 (Max), 21 (Max+).

#### 1.3.2. Danh sách các ca kiểm thử giá trị biên (BVA Test Cases)
| Mã Test Case | Điểm biên kiểm thử | Độ dài mật khẩu đầu vào | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
| :--- | :--- | :---: | :--- | :--- | :---: |
| **FR03-BVA-01** | Cận dưới ngoại biên (Min-) | 7 ký tự (`Abc123!`) | Báo lỗi mật khẩu quá ngắn | Đổi mật khẩu thành công | **FAIL** |
| **FR03-BVA-02** | Cận dưới nội biên (Min) | 8 ký tự (`Abcd123!`) | Đổi mật khẩu thành công | Đổi mật khẩu thành công | **PASS** |
| **FR03-BVA-03** | Cận dưới nội biên (Min+) | 9 ký tự (`Abcde123!`) | Đổi mật khẩu thành công | Đổi mật khẩu thành công | **PASS** |
| **FR03-BVA-04** | Cận trên nội biên (Max-) | 19 ký tự (`Abcdefghijkl123456!`) | Đổi mật khẩu thành công | Đổi mật khẩu thành công | **PASS** |
| **FR03-BVA-05** | Cận trên nội biên (Max) | 20 ký tự (`Abcdefghijklm123456!`) | Đổi mật khẩu thành công | Đổi mật khẩu thành công | **PASS** |
| **FR03-BVA-06** | Cận trên ngoại biên (Max+) | 21 ký tự (`Abcdefghijklmn123456!`) | Báo lỗi mật khẩu vượt quá độ dài quy định | Đổi mật khẩu thành công | **FAIL** |

---

### 1.4. Phân tích khoảng cách AI (AI Gap Analysis)
- **Những gì AI đã bỏ sót:** Ban đầu AI cho rằng hệ thống backend có các cơ chế validate chặt chẽ (như kiểm tra regex email và kiểm tra độ dài mật khẩu ở API reset password) dựa theo lẽ thường. AI đã bỏ qua việc kiểm tra thực tế mã nguồn backend để thấy rằng backend hoàn toàn không có dòng code validate mật khẩu nào.
- **Lý do AI bỏ sót:** AI có xu hướng "suy luận lý tưởng hóa", giả định các chuẩn bảo mật mặc định mà không kiểm tra cấu trúc mã nguồn thực tế.

---

### 1.5. Báo cáo lỗi (Bug Report)
*   **FR03-BUG-01: Lộ mã resetToken qua API Response.**
    *   *Mô tả:* Endpoint `POST /api/forgot-password` trả về mã OTP trực tiếp trong response body: `{"message": "Mã đặt lại mật khẩu đã được tạo", "resetToken": "123456"}`. Điều này giúp bất kỳ ai biết email người khác đều có thể lấy được OTP và đặt lại mật khẩu.
    *   *Mức độ:* Nghiêm trọng (Critical).
*   **FR03-BUG-02: Backend thiếu validation độ dài và độ phức tạp mật khẩu mới.**
    *   *Mô tả:* API `POST /api/reset-password` không validate mật khẩu, cho phép đặt mật khẩu cực ngắn (ví dụ: `1`) hoặc để trống (`""`).
    *   *Mức độ:* Cao (High).


---

## 2. Tính năng B: FR-09 – Mã giảm giá (Discount Coupons)

### 2.1. Tổng quan tính năng & Yêu cầu
- **Mô tả:** Áp dụng mã giảm giá để tính toán số tiền cần trả sau giảm.
- **Các trường dữ liệu đầu vào:**
  - `code` (Mã giảm giá).
  - `total_amount` (Tổng giá trị đơn hàng hiện tại).
  - `user_id` (ID của người dùng).

---

### 2.2. Kiểm thử miền trị (Domain Testing)
#### 2.2.1. Phân tích miền trị từng bước
- **Phân hoạch tương đương (EP):**
  - **Mã giảm giá (Coupon Code):**
    - EP-CODE-VAL: Mã giảm giá hoạt động, tồn tại (`SAVE10`, `BIGBUY`).
    - EP-CODE-INV-EXP: Mã giảm giá đã hết hạn (`EXPIRED`).
    - EP-CODE-INV-NOT: Mã giảm giá không tồn tại.
  - **Tổng giá trị đơn hàng (Total Amount) so với `min_order_amount`:**
    - EP-VAL-ABOVE: Lớn hơn giá trị tối thiểu của mã.
    - EP-INV-BELOW: Nhỏ hơn hoặc bằng giá trị tối thiểu của mã.

#### 2.2.2. Danh sách các ca kiểm thử miền trị
| Mã Test Case | Dữ liệu đầu vào (Input) | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
| :--- | :--- | :--- | :--- | :---: |
| **FR09-DT-01** | code: `SAVE10`<br>total_amount: `500,000`<br>(min_order_amount = 300,000) | Áp dụng thành công, giảm 10% (giảm 50,000 ₫), giá cuối: 450,000 ₫ | Áp dụng thành công nhưng giá cuối cùng thành **5,000,000 ₫** (tăng gấp 10 lần) | **FAIL** (Lỗi tính toán phần trăm) |
| **FR09-DT-02** | code: `BIGBUY`<br>total_amount: `600,000`<br>(min_order_amount = 500,000) | Áp dụng thành công, giảm cố định 50,000 ₫, giá cuối: 550,000 ₫ | Áp dụng thành công, giá cuối: 550,000 ₫ | **PASS** |
| **FR09-DT-03** | code: `EXPIRED`<br>total_amount: `200,000` | Báo lỗi mã giảm giá đã hết hạn | Phản hồi lỗi "Mã giảm giá đã hết hạn" | **PASS** |
| **FR09-DT-04** | code: `UNKNOWN`<br>total_amount: `100,000` | Báo lỗi mã không tồn tại | Phản hồi lỗi "Mã giảm giá không tồn tại..." | **PASS** |
| **FR09-DT-05** | code: `SAVE10`<br>total_amount: `200,000` | Báo lỗi đơn hàng chưa đạt giá trị tối thiểu 300,000 ₫ | Phản hồi lỗi đơn hàng chưa đủ giá trị tối thiểu | **PASS** |

---

### 2.3. Phân tích giá trị biên (Boundary Value Analysis - BVA)
#### 2.3.1. Phân tích giá trị biên từng bước
- **Biến kiểm thử:** Tổng tiền đơn hàng (`total_amount`) so với giá trị tối thiểu (`min_order_amount` = 300,000 ₫ của mã `SAVE10`).
- **Điểm biên:** 300,000 ₫.
- **Các giá trị kiểm thử:**
  - 299,999 ₫ (Min-): Không áp dụng được.
  - 300,000 ₫ (Min): Áp dụng được (theo nghiệp vụ thông thường: đơn hàng tối thiểu 300k thì đúng 300k phải áp dụng được).
  - 300,001 ₫ (Min+): Áp dụng được.

#### 2.3.2. Danh sách các ca kiểm thử giá trị biên (BVA Test Cases)
| Mã Test Case | Điểm biên kiểm thử | Giá trị đơn hàng đầu vào | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
| :--- | :--- | :---: | :--- | :--- | :---: |
| **FR09-BVA-01** | Biên dưới ngoại biên (Min-) | 299,999 ₫ | Không áp dụng được (Báo lỗi chưa đủ giá trị tối thiểu) | Báo lỗi chưa đủ giá trị tối thiểu | **PASS** |
| **FR09-BVA-02** | Đúng giá trị biên (Min) | 300,000 ₫ | Áp dụng thành công | Phản hồi lỗi: "Đơn hàng chưa đủ giá trị tối thiểu 300,000 ₫ để áp dụng" | **FAIL** (Lỗi biên lớn hơn thay vì lớn hơn hoặc bằng) |
| **FR09-BVA-03** | Biên trên nội biên (Min+) | 300,001 ₫ | Áp dụng thành công | Áp dụng thành công (nhưng tính sai tiền) | **PASS** (Về mặt logic biên) |

---

### 2.4. Phân tích khoảng cách AI (AI Gap Analysis)
- **Những gì AI đã bỏ sót:** AI không phát hiện ra rằng công thức tính tiền giảm giá theo phần trăm trong backend sử dụng toán tử sai: `Math.floor(total_amount * (1 - coupon.discount_value))` dẫn đến số tiền giảm bị âm và giá cuối cùng bị đội lên.
- **Lý do AI bỏ sót:** Do AI chỉ phân tích đặc tả API (api_specification.md) mà không thực thi test thực tế trên endpoint này hoặc không đọc kỹ từng dòng code tính toán cụ thể trong file `server.js`.

---

### 2.5. Báo cáo lỗi (Bug Report)
*   **FR09-BUG-01: Lỗi công thức tính toán giảm giá theo phần trăm (Percent discount).**
    *   *Mô tả:* Ở `server.js` dòng 398-401, thay vì nhân với `(discount_value / 100)` để tính số tiền được giảm, mã nguồn lại dùng `(1 - discount_value)`, làm cho số tiền giảm bị tính sai (âm) và người dùng bị trả giá cao gấp 10 lần.
    *   *Mức độ:* Nghiêm trọng (Critical).
*   **FR09-BUG-02: Lỗi so sánh biên tối thiểu đơn hàng (`min_order_amount`).**
    *   *Mô tả:* Logic kiểm tra ở dòng 379 dùng toán tử `total_amount > coupon.min_order_amount`. Điều này chặn không cho áp dụng mã giảm giá khi đơn hàng có giá trị đúng bằng mức tối thiểu quy định.
    *   *Mức độ:* Trung bình (Medium).
*   **FR09-BUG-03: Cho phép giá cuối cùng của đơn hàng bị âm.**
    *   *Mô tả:* Khi áp dụng mã giảm giá cố định (fixed) có giá trị lớn hơn tổng giá trị giỏ hàng (ví dụ: giảm 100k cho đơn 80k), hệ thống vẫn cho phép áp dụng thành công và trả về `final_amount` là số âm (`-20,000 ₫`).
    *   *Mức độ:* Cao (High).

---

## 3. Tính năng C: FR-14 – Quản lý danh mục (Category Management - CRUD)

### 3.1. Tổng quan tính năng & Yêu cầu
- **Mô tả:** Cho phép quản trị viên Thêm (Create), Đọc (Read), Cập nhật (Update), và Xóa (Delete) danh mục sản phẩm.
- **Các trường dữ liệu đầu vào:**
  - `name` (Tên danh mục).
  - `id` (ID của danh mục khi sửa/xóa).

---

### 3.2. Kiểm thử miền trị (Domain Testing)
#### 3.2.1. Phân tích miền trị từng bước
- **Phân hoạch tương đương (EP):**
  - **Tên danh mục (name):**
    - EP-NAME-VAL: Chuỗi ký tự chữ và số hợp lệ (ví dụ: "Thiết bị số").
    - EP-NAME-INV-EMPTY: Chuỗi rỗng hoặc khoảng trắng `""`.
    - EP-NAME-INV-DUP: Tên danh mục đã tồn tại trong hệ thống.
  - **ID danh mục (id):**
    - EP-ID-VAL: ID tồn tại trong DB.
    - EP-ID-INV: ID không tồn tại trong DB (ví dụ: `9999`).

#### 3.2.2. Danh sách các ca kiểm thử miền trị
| Mã Test Case | Hành động | Dữ liệu đầu vào (Input) | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **FR14-DT-01** | Create | name: `Gia dụng` | Tạo danh mục thành công | Tạo danh mục thành công | **PASS** |
| **FR14-DT-02** | Create | name: `""` (Rỗng) | Báo lỗi tên danh mục không được để trống | Hệ thống tạo thành công danh mục không có tên | **FAIL** (Thiếu validation) |
| **FR14-DT-03** | Create | name: `Laptop` (Đã có) | Báo lỗi tên danh mục đã tồn tại | Hệ thống tạo thành công danh mục trùng tên thứ hai | **FAIL** (Không kiểm tra trùng lặp) |
| **FR14-DT-04** | Update | id: `1`, name: `Điện thoại xịn` | Cập nhật thành công | Cập nhật thành công | **PASS** |
| **FR14-DT-05** | Update | id: `9999` (Không có), name: `Test` | Báo lỗi danh mục không tồn tại | Báo cập nhật thành công (`Category updated`) | **FAIL** (Thiếu kiểm tra sự tồn tại) |
| **FR14-DT-06** | Delete | id: `1` (Đang chứa sp) | Báo lỗi hoặc chặn xóa vì đang chứa sản phẩm | Xóa thành công danh mục, để lại các sản phẩm không có danh mục | **FAIL** (Lỗi toàn vẹn dữ liệu) |

---

### 3.3. Phân tích giá trị biên (Boundary Value Analysis - BVA)
- **Biến kiểm thử:** Độ dài ký tự của tên danh mục (`name`).
- Do backend sử dụng kiểu `TEXT` của SQLite và hoàn toàn không giới hạn độ dài ở backend, bất kỳ độ dài nào từ 1 ký tự trở lên (và thậm chí cả 0 ký tự) đều có thể được chèn vào DB. Tuy nhiên, theo nghiệp vụ:
  - Biên tối thiểu: 1 ký tự.
  - Biên tối đa (ví dụ): 50 ký tự.
- **BVA Test Cases:**
  - Biên dưới (0 ký tự): Hệ thống cho phép chèn (Bug).
  - Biên trên (51 ký tự): Hệ thống cho phép chèn mà không cắt bớt hay báo lỗi.

---

### 3.4. Phân tích khoảng cách AI (AI Gap Analysis)
- **Những gì AI đã bỏ sót:** AI không dự đoán được lỗi ràng buộc khóa ngoại (Foreign Key constraint) khi xóa danh mục. AI giả định rằng SQLite đã kích hoạt Foreign Key cascade hoặc chặn xóa mặc định.
- **Lý do AI bỏ sót:** SQLite mặc định không kích hoạt chế độ kiểm tra khóa ngoại (`PRAGMA foreign_keys = ON`) trừ khi được cấu hình cụ thể khi kết nối database.

---

### 3.5. Báo cáo lỗi (Bug Report)
*   **FR14-BUG-01: Cho phép xóa danh mục đang có sản phẩm liên kết.**
    *   *Mô tả:* Khi xóa một danh mục sản phẩm (ví dụ ID = 1), hệ thống xóa thành công mà không kiểm tra xem có sản phẩm nào đang thuộc danh mục đó không, khiến dữ liệu sản phẩm bị lỗi khóa ngoại mồ côi.
    *   *Mức độ:* Cao (High).
*   **FR14-BUG-02: Cho phép tạo danh mục có tên trùng lặp.**
    *   *Mô tả:* CSDL và backend không thiết lập thuộc tính `UNIQUE` cho cột `name`, cho phép tạo nhiều danh mục có cùng một tên gọi.
    *   *Mức độ:* Trung bình (Medium).
*   **FR14-BUG-03: Sửa/Xóa ID không tồn tại vẫn phản hồi 200 OK thành công.**
    *   *Mô tả:* APIs `PUT /api/categories/:id` và `DELETE /api/categories/:id` không kiểm tra `this.changes`, dẫn tới việc gửi ID không tồn tại vẫn báo thành công.
    *   *Mức độ:* Thấp (Low).

---

## 4. Tính năng D: Mobile App – Tìm kiếm sản phẩm (Product Search - FR-05 mobile)

### 4.1. Tổng quan tính năng & Yêu cầu
- **Mô tả:** Người dùng nhập từ khóa vào ô tìm kiếm trên ứng dụng di động để lọc và hiển thị các sản phẩm có tên khớp với từ khóa tìm kiếm.
- **Các trường dữ liệu đầu vào:**
  - `search` (Chuỗi ký tự nhập vào ô tìm kiếm).

---

### 4.2. Kiểm thử miền trị (Domain Testing)
#### 4.2.1. Phân tích miền trị từng bước
- **Phân hoạch tương đương (EP):**
  - **Từ khóa tìm kiếm (search):**
    - EP-SEARCH-VAL-EXACT: Chuỗi ký tự khớp chính xác tên sản phẩm trong CSDL (ví dụ: `"iPhone 13"`).
    - EP-SEARCH-VAL-PARTIAL: Chuỗi ký tự khớp một phần tên sản phẩm (ví dụ: `"Phone"`).
    - EP-SEARCH-VAL-NONE: Chuỗi ký tự không khớp với bất kỳ sản phẩm nào (ví dụ: `"Nokia 3310"`).
    - EP-SEARCH-VAL-EMPTY: Chuỗi rỗng `""` (hiển thị toàn bộ sản phẩm).
    - EP-SEARCH-INV-SQLINJ: Chuỗi chứa payload SQL Injection (ví dụ: `"' OR 1=1 --"`).
    - EP-SEARCH-INV-SPECIALCHAR: Chuỗi chứa ký tự đại diện wildcard của SQL LIKE (ví dụ: `"%"` hoặc `"_"`).

#### 4.2.2. Danh sách các ca kiểm thử miền trị (Domain Test Cases)
| Mã Test Case | Hành động | Dữ liệu đầu vào (Input) | Kết quả mong đợi (Expected) | Kết quả thực tế (Actual) | Trạng thái |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **FR05M-DT-01** | Tìm kiếm chính xác | search: `"iPhone 13"` | Chỉ hiển thị sản phẩm "iPhone 13" | Hiển thị sản phẩm "iPhone 13" | **PASS** |
| **FR05M-DT-02** | Tìm kiếm một phần | search: `"Phone"` | Hiển thị tất cả sản phẩm chứa chữ "Phone" | Hiển thị các sản phẩm chứa chữ "Phone" | **PASS** |
| **FR05M-DT-03** | Tìm kiếm không có kết quả | search: `"Nokia 3310"` | Hiển thị 0 sản phẩm | Hiển thị 0 sản phẩm | **PASS** |
| **FR05M-DT-04** | Tìm kiếm bằng chuỗi rỗng | search: `""` | Hiển thị toàn bộ sản phẩm trong hệ thống | Hiển thị toàn bộ sản phẩm | **PASS** |
| **FR05M-DT-05** | Tìm kiếm bằng SQL Injection | search: `"' OR 1=1 --"` | Không bị SQL Injection, trả về 0 sản phẩm hoặc báo lỗi tìm kiếm | Trả về toàn bộ sản phẩm (SQL Injection thành công) | **FAIL** (Lỗi bảo mật nghiêm trọng) |
| **FR05M-DT-06** | Tìm kiếm chứa wildcard | search: `"%"` | Chỉ hiển thị sản phẩm thực sự chứa ký tự `%` hoặc báo 0 kết quả | Trả về toàn bộ sản phẩm | **FAIL** (Không escape ký tự wildcard) |
| **FR05M-DT-07** | Tìm kiếm chứa dấu nháy đơn lỗi cú pháp | search: `"'"` | Hệ thống hoạt động bình thường, trả về 0 kết quả | Bị lỗi cú pháp SQL và phản hồi trang lỗi HTML `500` thô làm lỗi hiển thị trên ứng dụng di động | **FAIL** (Lỗi hiển thị HTML + Lỗi SQLi) |

---

### 4.3. Phân tích giá trị biên (Boundary Value Analysis - BVA)
#### 4.3.1. Phân tích giá trị biên từng bước
- **Biến kiểm thử:** Độ dài ký tự của từ khóa tìm kiếm (`search`).
- **Biên tối thiểu:** 1 ký tự. Biên tối đa (dự kiến): 255 ký tự.
- **Các điểm biên xác định (3-point BVA):**
  - Cận dưới: 1 ký tự. Giá trị thử nghiệm: 0 ký tự (Min-), 1 ký tự (Min), 2 ký tự (Min+).
  - Cận trên: 255 ký tự. Giá trị thử nghiệm: 254 ký tự (Max-), 255 ký tự (Max), 256 ký tự (Max+).

#### 4.3.2. Danh sách các ca kiểm thử giá trị biên (BVA Test Cases)
| Mã Test Case | Điểm biên kiểm thử | Độ dài từ khóa | Dữ liệu đầu vào (Input) | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
| :--- | :--- | :---: | :--- | :--- | :--- | :---: |
| **FR05M-BVA-01** | Cận dưới ngoại biên (Min-) | 0 | `""` | Trả về toàn bộ sản phẩm | Trả về toàn bộ sản phẩm | **PASS** |
| **FR05M-BVA-02** | Cận dưới nội biên (Min) | 1 | `"i"` | Trả về sản phẩm chứa ký tự "i" | Trả về sản phẩm chứa ký tự "i" | **PASS** |
| **FR05M-BVA-03** | Cận dưới nội biên (Min+) | 2 | `"iP"` | Trả về sản phẩm chứa ký tự "iP" | Trả về sản phẩm chứa ký tự "iP" | **PASS** |
| **FR05M-BVA-04** | Cận trên nội biên (Max-) | 254 | Chuỗi 254 ký tự | Trả về 0 kết quả (Tìm kiếm an toàn) | Trả về 0 kết quả | **PASS** |
| **FR05M-BVA-05** | Cận trên nội biên (Max) | 255 | Chuỗi 255 ký tự | Trả về 0 kết quả (Tìm kiếm an toàn) | Trả về 0 kết quả | **PASS** |
| **FR05M-BVA-06** | Cận trên ngoại biên (Max+) | 256 | Chuỗi 256 ký tự | Báo lỗi giới hạn ký tự nhập vào hoặc tự động cắt ngắn | Tìm kiếm bình thường, trả về 0 kết quả | **FAIL** (Không giới hạn độ dài ký tự) |

---

### 4.4. Phân tích khoảng cách AI (AI Gap Analysis)
- **Những gì AI đã bỏ sót:** AI không phát hiện ra lỗ hổng SQL Injection nghiêm trọng tại endpoint tìm kiếm sản phẩm và việc trả về mã nguồn lỗi dạng trang web HTML thay vì định dạng JSON làm hỏng giao diện di động React Native.
- **Lý do AI bỏ sót:** AI thường giả định rằng các framework hoặc thư viện hiện đại tự động ngăn chặn lỗi SQL Injection bằng Parameterized Queries và tự động parse định dạng lỗi API JSON thống nhất.

---

### 4.5. Báo cáo lỗi (Bug Report)
*   **FR05M-BUG-01: Lỗ hổng SQL Injection nghiêm trọng trong API tìm kiếm sản phẩm.**
    *   *Mô tả:* API `/api/products?search=` trực tiếp nối chuỗi truy vấn đầu vào vào câu lệnh SQL thay vì dùng tham số hóa, cho phép kẻ tấn công chèn mã lệnh SQL nguy hiểm.
    *   *Mức độ:* Nghiêm trọng (Critical).
*   **FR05M-BUG-02: Phản hồi lỗi định dạng HTML thô làm lỗi giao diện ứng dụng di động.**
    *   *Mô tả:* Khi truy vấn SQL lỗi (ví dụ chứa `'`), backend trả về mã lỗi HTML thô khiến ứng dụng di động không parse được JSON và hiển thị chuỗi HTML thô lên màn hình.
    *   *Mức độ:* Cao (High).
*   **FR05M-BUG-03: Không lọc ký tự đại diện `%` và `_` khi tìm kiếm.**
    *   *Mô tả:* Nhập ký tự `%` hoặc `_` trong ô tìm kiếm sẽ trả về tất cả sản phẩm thay vì khớp ký tự thực tế do thiếu xử lý làm sạch (escaping) wildcard của mệnh đề `LIKE`.
    *   *Mức độ:* Trung bình (Medium).
