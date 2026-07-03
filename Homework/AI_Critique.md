# Đánh giá và Phản biện AI (AI Critique)

Trong quá trình thực hiện bài tập kiểm thử miền trị (Domain Testing) và phân tích giá trị biên (BVA) trên hệ thống EShop, việc sử dụng các mô hình AI lớn (như ChatGPT, Claude, Gemini) đã mang lại cả những hỗ trợ đắc lực lẫn những bài học sâu sắc về giới hạn của công nghệ này.

### 1. Những sai sót và điểm chưa hoàn thiện của AI
AI thường đưa ra kết quả mang tính "lý tưởng hóa" và mặc định rằng hệ thống đã tuân thủ đầy đủ các tiêu chuẩn lập trình an toàn. Cụ thể:
- **Tự động giả định tính bảo mật và kiểm thử logic biên:** AI mặc định hệ thống backend đã có sẵn các bộ kiểm tra (validation) định dạng email, độ dài mật khẩu mới hay kiểm tra khóa ngoại (Foreign Key) khi xóa danh mục, trong khi thực tế code backend hoàn toàn trống rỗng các phần này.
- **Bỏ lọt lỗi nghiêm trọng do thiếu phân tích mã nguồn chi tiết:** AI không tự phát hiện lỗi logic tính toán nghiêm trọng tại API mã giảm giá (`1 - coupon.discount_value` thay vì `coupon.discount_value / 100`). Đối với tính năng tìm kiếm trên ứng dụng di động (Feature D), AI ban đầu hoàn toàn bỏ sót lỗ hổng bảo mật SQL Injection nghiêm trọng tại API tìm kiếm (`LIKE '%${searchQuery}%'`) và lỗi hiển thị giao diện HTML thô khi database bị lỗi cú pháp, vì AI chỉ đọc tài liệu đặc tả API cấp cao thay vì phân tích tỉ mỉ mã nguồn backend/frontend.

### 2. Nguyên nhân AI bỏ sót các vấn đề
Nguyên nhân chủ yếu do AI thiếu môi trường thực thi động (dynamic execution context) để chạy thực tế ứng dụng và có xu hướng suy luận dựa trên các mẫu thiết kế phổ biến có sẵn trong dữ liệu huấn luyện. Nếu prompt đầu vào của người dùng chỉ ở dạng mô tả tính năng chung chung mà không đính kèm mã nguồn chi tiết, AI sẽ tạo ra các bộ test cases lý thuyết và bỏ lọt hầu hết các lỗi logic nội bộ cũng như lỗi bảo mật thực tế.

### 3. Bài học rút ra khi cộng tác với AI
Nguyên tắc cốt lõi là luôn duy trì vai trò "Human-in-the-loop" (con người kiểm duyệt). Không bao giờ tin cậy trực tiếp vào kết quả thô của AI mà không đối chiếu trực tiếp với mã nguồn thực tế và kết quả chạy thử nghiệm. Đồng thời, thay vì sử dụng prompt một bước duy nhất, việc chia nhỏ quy trình kiểm thử thành các phiên làm việc có cấu trúc (như phân tích phân hoạch tương đương trước, thiết kế test case sau, sau đó đối chiếu mã nguồn và thực thi kiểm thử thực tế) sẽ giúp định hướng AI làm việc chính xác và hiệu quả hơn.