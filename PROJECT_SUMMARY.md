# Tổng quan Dự án: Capstone Defense Scheduler Backend

Chào bạn, với góc nhìn của một Senior Developer, mình đã phân tích và tóm tắt lại toàn bộ dự án này để bạn có thể nắm bắt nhanh chóng và chính xác nhất. Dưới đây là các thông tin cốt lõi về nghiệp vụ, công nghệ và cấu trúc triển khai của hệ thống.

---

## 1. Nghiệp vụ cốt lõi (Business Logic)

**Bài toán chính:** Hệ thống chuyên trách việc **xếp lịch bảo vệ đồ án tốt nghiệp (Capstone Defense Scheduling)** một cách tự động và tối ưu. Bài toán phân bổ Giảng viên vào các Hội đồng đánh giá sao cho thỏa mãn hàng loạt các điều kiện ràng buộc khắt khe.

### Các thực thể chính (Domain Entities):
- **Lecturer (Giảng viên):** Người đánh giá đồ án, có lịch rảnh (Availability), hạn mức chấm thi (Quota) và năng lực cho từng vai trò (Competency).
- **Project (Đồ án):** Đồ án của sinh viên, có giáo viên hướng dẫn (Supervisor).
- **Council Block / Round Block:** Các khung giờ / phiên bảo vệ của hội đồng.
- **Council Role (Vai trò):** 5 vai trò trong một hội đồng (ví dụ: Chủ tịch, Thư ký, Ủy viên...).
- **Defense Day / Semester:** Ngày bảo vệ và Học kỳ.

### Thuật toán xếp lịch (Scheduling Optimization):
Sử dụng AI Constraint Solver (**Timefold**) để giải quyết bài toán NP-Hard này với các bộ quy tắc (Constraints):

* **Hard Constraints (Bắt buộc thỏa mãn - Kỷ luật thép):**
  1. Không bị trùng lịch (No double booking).
  2. Giảng viên **không được** ngồi hội đồng chấm đồ án do chính mình hướng dẫn.
  3. Giảng viên phải có lịch rảnh (`Available`) tại khung giờ đó.
  4. Không phân công vượt quá số lượng hội đồng tối đa (`Max Quota`).
  5. Mỗi hội đồng phải có đúng 5 vai trò khác nhau và mỗi giảng viên chỉ đóng 1 vai trò trong 1 hội đồng.

* **Soft Constraints (Ưu tiên tối ưu hóa):**
  1. Cố gắng đạt số lượng hội đồng tối thiểu (`Min Quota`) của mỗi giảng viên.
  2. Cân bằng khối lượng công việc giữa các giảng viên (Balanced workload).
  3. Ưu tiên gán vai trò phù hợp với thế mạnh của giảng viên (Maximize role competency).

---

## 2. Stack Công nghệ (Tech Stack)

Dự án áp dụng các công nghệ Backend hiện đại và ổn định của Java ecosystem:

* **Ngôn ngữ & Framework:** Java 21, Spring Boot 3.5.10.
* **Database:** PostgreSQL (hiện tại đang cấu hình dùng qua Supabase).
* **AI & Optimization:** **Timefold Solver 1.18.0** (Công cụ cực mạnh cho bài toán lên lịch).
* **Security:** Spring Security + JWT (JSON Web Token) cho Authentication & Authorization.
* **API Documentation:** OpenAPI (Springdoc / Swagger UI).
* **Tiện ích:**
  * `Lombok`: Giảm boilerplate code.
  * `Spring Dotenv`: Quản lý biến môi trường qua file `.env`.
  * `Apache POI`: Import/Export dữ liệu Excel (dự kiến dùng cho danh sách giảng viên, sinh viên).
  * `Firebase Admin SDK`: Gửi Push Notification tới thiết bị di động.
  * `Resend API`: Dịch vụ gửi Email thông báo.

---

## 3. Cấu trúc triển khai & Architecture

Hệ thống được thiết kế theo kiến trúc **Layered Architecture (N-Tier)** chuẩn mực của Spring Boot, giúp dễ bảo trì và mở rộng:

### Kiến trúc thư mục (Packages):
* `controller`: Expose các RESTful APIs cho Client (Mobile, Web Admin).
* `service`: Chứa Business Logic, xử lý nghiệp vụ thông thường.
* `repository`: Data Access Layer sử dụng Spring Data JPA để giao tiếp với PostgreSQL.
* `entity`: Các Java class ánh xạ với bảng trong Database (JPA Entities).
* `dto`: Data Transfer Objects để nhận/trả dữ liệu thay vì expose trực tiếp Entity.
* `security`: Cấu hình Spring Security, các filter xử lý JWT.
* `solver`: **Trái tim của bài toán tối ưu**, chứa các `domain` model dành riêng cho Timefold và `constraint` định nghĩa các quy tắc xếp lịch.
* `exception`: Global Exception Handling (`@ControllerAdvice`) chuẩn hóa lỗi trả về.

### Triển khai & Cấu hình:
* **Database Schema:** Đang dùng Hibernate `ddl-auto=update` để tự động map schema từ code lên Database.
* **Environment Variables:** Toàn bộ credential quan trọng (Database URL, User/Pass, JWT Secret, Firebase Key, Resend API Key) được đưa ra ngoài file `.env` (không push lên Git).
* **Build Tool:** Maven (`pom.xml`).
* **Timefold Config:** Giải bài toán tối đa 10s (`timefold.solver.termination.spent-limit=10s`) để đưa ra kết quả tối ưu nhất trong thời gian cho phép.

---

**Tóm lại:** Đây là một hệ thống Backend chất lượng tốt, chia layer rõ ràng. Điểm "đắt giá" và phức tạp nhất chính là package `solver` (nơi cấu hình Timefold để tính toán lịch bảo vệ). Nếu bạn tiếp quản dự án, hãy tập trung nắm vững luồng hoạt động của Spring Security (để test API) và logic ràng buộc trong `DefenseScheduleConstraintProvider.java`.
