# Capstone Defense Scheduler Backend 📅🤖

> Hệ thống Backend thông minh chuyên trách việc **xếp lịch bảo vệ đồ án tốt nghiệp** tự động và tối ưu, sử dụng AI Constraint Solver để giải quyết bài toán phân bổ hội đồng.

---

## 📌 Tổng Quan Dự Án
Bài toán lập lịch bảo vệ đồ án (Capstone Defense Scheduling) là một bài toán tối ưu hóa phức tạp (NP-Hard). Hệ thống này tự động hóa việc phân bổ Giảng viên vào các Hội đồng đánh giá sao cho thỏa mãn đồng thời các ràng buộc nghiêm ngặt về thời gian rảnh, vai trò, định mức chấm thi, chuyên môn, và tính khách quan.

Hệ thống được phát triển dựa trên nền tảng **Java 21** kết hợp với bộ công cụ tối ưu hóa **Timefold Solver**.

---

## 🛠️ Stack Công Nghệ (Tech Stack)

### Core Backend
*   **Java 21** & **Spring Boot 3.5.10**
*   **Maven** làm công cụ quản lý package và build.

### Lưu Trữ & Bảo Mật
*   **Database:** PostgreSQL (Cấu hình đám mây qua Supabase).
*   **ORM:** Spring Data JPA + Hibernate (Cấu hình tự động ánh xạ `ddl-auto=update`).
*   **Security:** Spring Security & JWT (JSON Web Token) cho xác thực và phân quyền API.

### AI & Thuật Toán Tối Ưu
*   **Timefold Solver 1.18.0:** Bộ công cụ lập lịch AI cực mạnh giúp giải quyết các bài toán tối ưu hóa ràng buộc.

### Các Dịch Vụ & Tiện Ích Khác
*   **Spring Dotenv:** Đọc các biến môi trường trực tiếp từ file `.env` cục bộ.
*   **Apache POI:** Hỗ trợ import/export dữ liệu Excel (dành cho danh sách giảng viên, sinh viên, hội đồng).
*   **Firebase Admin SDK:** Gửi thông báo đẩy (Push Notification) đến thiết bị di động.
*   **Resend API:** Dịch vụ gửi email thông báo tự động (thông báo lịch, tài khoản).
*   **OpenAPI / Swagger UI:** Tự động tạo và hiển thị tài liệu API.
*   **Lombok:** Giảm thiểu boilerplate code (Getter, Setter, Constructor,...).

---

## 📐 Kiến Trúc Thư Mục (Layered Architecture)
Dự án được cấu trúc theo mô hình **N-Tier Layered Architecture** chuẩn mực:

```text
src/main/java/com/capstone/scheduler/
├── config/             # Cấu hình hệ thống (CORS, Swagger, Firebase, Mail,...)
├── controller/         # Khai báo các RESTful APIs tiếp nhận request từ Client
├── dto/                # Data Transfer Objects trao đổi dữ liệu giữa các layer
├── entity/             # Các lớp ánh xạ trực tiếp sang các bảng Database (JPA Entities)
├── exception/          # Xử lý ngoại lệ tập trung (Global Exception Handling)
├── repository/         # Tương tác với cơ sở dữ liệu (Spring Data JPA Repositories)
├── security/           # Cấu hình Spring Security, filter xác thực JWT
├── service/            # Xử lý Business Logic của ứng dụng
└── solver/             # TRÁI TIM TỐI ƯU HÓA (AI Constraint Solving)
    ├── domain/         # Định nghĩa các model bài toán cho Timefold (PlanningEntity, PlanningSolution)
    └── constraint/     # Định nghĩa các luật / ràng buộc xếp lịch (DefenseScheduleConstraintProvider)
```

---

## 🧠 Các Quy Tắc Tối Ưu Lịch (Constraints Definition)

Hệ thống hoạt động dựa trên các bộ quy tắc được chia làm hai nhóm chính:

### 🔴 Ràng Buộc Cứng (Hard Constraints) - Bắt Buộc 100%
1.  **Không Trùng Lịch (No Double Booking):** Giảng viên không thể ở hai hội đồng khác nhau trong cùng một khung giờ.
2.  **Khách Quan Đối Với Giáo Viên Hướng Dẫn:** Giảng viên **không được** ngồi hội đồng chấm đồ án do chính mình làm Giáo viên hướng dẫn (Supervisor).
3.  **Lịch Rảnh (Availability):** Chỉ xếp giảng viên vào khung giờ mà họ đã đăng ký rảnh (`Available`).
4.  **Hạn Mức Chấm Thi (Max Quota):** Số lượng hội đồng được phân công không vượt quá hạn mức tối đa của giảng viên.
5.  **Cơ Cấu Hội Đồng:** Mỗi hội đồng phải có đúng 5 vai trò khác nhau và mỗi giảng viên chỉ đóng duy nhất 1 vai trò trong hội đồng đó.

### 🟡 Ràng Buộc Mềm (Soft Constraints) - Ưu Tiên Tối Ưu
1.  **Định Mức Tối Thiểu (Min Quota):** Cố gắng đáp ứng số lượng hội đồng tối thiểu đã cam kết của mỗi giảng viên.
2.  **Cân Bằng Tải (Workload Balance):** Phân bổ số lượng hội đồng đồng đều nhất có thể giữa các giảng viên để tránh quá tải.
3.  **Chuyên Môn Hóa (Role Competency):** Ưu tiên gán các vai trò quan trọng (Chủ tịch, Thư ký) cho giảng viên có điểm năng lực cao nhất đối với vai trò đó.

---

## 🚀 Khởi Động Dự Án (Local Setup)

### Yêu Cầu Hệ Thống
*   Java Development Kit (JDK) **21** trở lên.
*   Maven 3.9+.
*   Cơ sở dữ liệu PostgreSQL (hoặc kết nối Supabase có sẵn).

### Các Bước Triển Khai

1.  **Clone dự án:**
    ```bash
    git clone <repository-url>
    cd capstone-defense-scheduler-backend
    ```

2.  **Cấu hình môi trường:**
    Tạo một file `.env` tại thư mục gốc của dự án (cùng cấp với `pom.xml`) dựa trên file mẫu dưới đây:
    ```env
    # Database Configuration
    DB_URL=jdbc:postgresql://<your-db-host>:<port>/<db-name>
    DB_USERNAME=your_db_username
    DB_PASSWORD=your_db_password

    # JWT Configuration
    JWT_SECRET=your_super_secret_jwt_key_at_least_256_bits
    JWT_EXPIRATION=86400000 # 24 hours in ms

    # Third-party APIs
    RESEND_API_KEY=your_resend_api_key
    FIREBASE_CREDENTIALS_PATH=classpath:firebase-service-account.json
    ```

3.  **Build và chạy dự án:**
    Sử dụng Maven Wrapper có sẵn trong dự án:
    *   **Trên Windows (Git Bash / Command Prompt):**
        ```bash
        ./mvnw spring-boot:run
        ```
    *   **Trên Linux / macOS:**
        ```bash
        chmod +x mvnw
        ./mvnw spring-boot:run
        ```

4.  **Truy cập API Documentation:**
    Sau khi ứng dụng khởi chạy thành công, truy cập Swagger UI để kiểm thử API tại:
    `http://localhost:8080/swagger-ui/index.html`

---

## 🌿 Quy Tắc Sử Dụng Git (Git Workflow & Guidelines)

Để đảm bảo mã nguồn luôn ổn định, sạch sẽ và dễ quản lý khi làm việc nhóm, tất cả thành viên bắt buộc phải tuân thủ nghiêm ngặt các quy tắc dưới đây:

### 1. Phân Nhánh (Branching Strategy)
Dự án áp dụng mô hình **Git Flow** rút gọn với các nhánh chính sau:

*   `main`: Nhánh chứa mã nguồn chạy ổn định nhất (Production-ready). Không bao giờ commit trực tiếp lên đây.
*   `develop`: Nhánh tích hợp các tính năng mới phục vụ cho kiểm thử và deploy môi trường Staging.
*   `feature/<tên-tính-năng>`: Nhánh tạo ra để phát triển tính năng mới. Tên nhánh viết thường, không dấu, ngăn cách bằng dấu gạch ngang `-`.
    *   *Ví dụ:* `feature/lecturer-quota`, `feature/jwt-authentication`.
*   `hotfix/<tên-lỗi>`: Nhánh dùng để sửa đổi khẩn cấp các lỗi nghiêm trọng xảy ra trên production.
    *   *Ví dụ:* `hotfix/expired-token-leak`.

### 2. Quy Trình Làm Việc Hằng Ngày (Daily Workflow)
Mỗi khi bắt đầu làm một công việc mới:
1.  **Chuyển về nhánh `develop` và cập nhật code mới nhất:**
    ```bash
    git checkout develop
    git pull origin develop
    ```
2.  **Tạo một nhánh feature mới từ `develop`:**
    ```bash
    git checkout -b feature/ten-tính-nang-moi
    ```
3.  **Làm việc, commit cục bộ thường xuyên:** (Tham khảo cách viết commit ở mục dưới).
4.  **Trước khi đẩy code lên Remote, gộp code mới từ `develop` để tránh conflict:**
    ```bash
    git checkout develop
    git pull origin develop
    git checkout feature/ten-tính-nang-moi
    git merge develop
    ```
    *(Nếu có xung đột - conflict, hãy giải quyết ngay trên máy cục bộ và kiểm tra kỹ xem ứng dụng còn chạy được không).*
5.  **Push nhánh lên github:**
    ```bash
    git push origin feature/ten-tính-nang-moi
    ```
6.  **Tạo Pull Request (PR) trên GitHub:** Yêu cầu merge từ `feature/...` sang `develop`. Cần có ít nhất 1 thành viên khác duyệt (Approve) code trước khi được phép merge.

### 3. Quy Tắc Viết Commit (Commit Message Convention)
Dự án áp dụng quy tắc **Conventional Commits**. Định dạng tiêu chuẩn của một commit message:

```text
<type>(<scope>): <mô tả ngắn bằng tiếng Việt hoặc tiếng Anh>
```

#### Các loại `<type>` được chấp nhận:
*   `feat`: Thêm tính năng mới (Feature).
*   `fix`: Vá lỗi (Bug fix).
*   `refactor`: Sửa cấu trúc code giúp tối ưu hơn mà không thay đổi tính năng hiện tại.
*   `docs`: Cập nhật tài liệu (README, API docs,...).
*   `style`: Thay đổi về cách format code (không thay đổi logic code).
*   `test`: Viết thêm testcase hoặc sửa đổi code test.
*   `chore`: Các thay đổi lặt vặt khác (nâng cấp thư viện, đổi cấu hình build,...).

#### Ví dụ commit hợp lệ:
*   `feat(solver): tích hợp thêm ràng buộc tối ưu hóa min quota giảng viên`
*   `fix(auth): sửa lỗi null pointer exception khi giải mã token hết hạn`
*   `docs(readme): bổ sung hướng dẫn chạy local và quy tắc git`
*   `chore(deps): nâng cấp thư viện timefold-solver lên phiên bản 1.18.0`

### 4. Quy Tắc "Vàng" Khi Làm Việc Với Git
*   🚨 **Không bao giờ push trực tiếp lên `main` hoặc `develop`.**
*   🚨 **Tuyệt đối không commit các file nhạy cảm như `.env`, cấu hình database cá nhân, key Firebase lên Git.** Đảm bảo các file này đã được liệt kê trong `.gitignore`.
*   🚨 **Không commit code khi dự án đang bị lỗi compile (không build được).**
*   🚨 **Viết commit chi tiết, có ý nghĩa.** Tránh viết commit chung chung vô nghĩa như `fix`, `update`, `chay thu`.

---

*Chúc các bạn có trải nghiệm lập trình tuyệt vời cùng hệ thống Capstone Defense Scheduler!* 🚀
