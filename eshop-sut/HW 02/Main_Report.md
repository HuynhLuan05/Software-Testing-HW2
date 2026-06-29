# HW02 – BÁO CÁO KIỂM THỬ MIỀN TRỊ & GIÁ TRỊ BIÊN (EShop SUT)

---

## 1. Tính năng A: FR-03 – Quên mật khẩu và Đặt lại mật khẩu (Forgot Password and Password Reset)

### 1.1. Tổng quan tính năng & Yêu cầu
- **Mô tả:** Người dùng yêu cầu nhận mã đặt lại mật khẩu qua email. Sau đó dùng mã đó để nhập mật khẩu mới.
- **Các trường dữ liệu đầu vào:**
  - `email` (Chuỗi ký tự, định dạng email, độ dài).
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
