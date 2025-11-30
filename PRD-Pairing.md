# 📄 PRODUCT REQUIREMENTS DOCUMENT (PRD)
**Project Name:** WOB Golf Pairings System  
**Version:** 3.3 (Final Comprehensive)  
**Date:** 2025-11-29  
**Status:** Ready for Development  
**Author:** WOB Tech Team  

---

## 1. GIỚI THIỆU (INTRODUCTION)
### 1.1. Mục đích dự án
Xây dựng hệ thống web-app (Single Page Application) để tự động hóa quy trình chia nhóm (pairing) cho giải đấu WOB Year-End Outing 2025. Hệ thống cần giải quyết bài toán chia bảng đấu phức tạp với nhiều điều kiện ưu tiên, đồng thời cung cấp giao diện trình diễn thị giác cao ("Wow moment") cho người tham dự.

### 1.2. Phạm vi (Scope)
* **In-scope:**
    * Có textbox nhập link CSV từ Google Sheet (CSV Public). Có giá trị mặc định là link CSV của Google Sheet chứa danh sách golfer.
    * Có button upload excel hoặc csv theo cấu trúc đã định. Có link download template excel hoặc csv để người dùng có thể tải về nhập và upload lên.
    * Có button "Chia Lại" để reset lại kết quả và bắt đầu lại quy trình.
    * Thuật toán chia nhóm theo thứ tự ưu tiên Bảng đấu > Cố định/Ngẫu nhiên > Handicap > Priority.
    * Giao diện hiển thị danh sách Flight (Card view).
    * Xuất kết quả ra Clipboard để dán vào Excel.
* **Out-scope:**
    * Hệ thống đăng nhập/quản lý user (Backend).
    * Lưu trữ database dài hạn (chỉ dùng LocalStorage).

---

## 2. USER STORIES (CÂU CHUYỆN NGƯỜI DÙNG)
| ID | Là một... (Role) | Tôi muốn... (Feature) | Để... (Benefit) |
| :--- | :--- | :--- | :--- |
| **US01** | Ban tổ chức | Load dữ liệu từ một file Google Sheet duy nhất | Không phải nhập tay danh sách 32 người chơi, tránh sai sót. |
| **US02** | Ban tổ chức | Cố định một số nhóm người chơi (VIP/Sponsor) đi cùng nhau | Đảm bảo ngoại giao và yêu cầu riêng của khách mời. |
| **US03** | Ban tổ chức | Ưu tiên giờ xuất phát (Tee-time) cho các nhóm VIP | Đáp ứng yêu cầu giờ giấc của khách mời quan trọng. |
| **US04** | Ban tổ chức | Copy kết quả kèm link ảnh Avatar ra Excel | Dễ dàng làm file in ấn, check-in hoặc gửi cho sân golf. |
| **US05** | Golfer | Xem kết quả chia nhóm trên màn hình lớn với hiệu ứng đẹp | Tạo cảm giác hồi hộp, chuyên nghiệp và minh bạch. |
| **US06** | Golfer | Thấy rõ mình thuộc Bảng nào, HDC bao nhiêu | Chuẩn bị tinh thần thi đấu. |

---

## 3. YÊU CẦU CHỨC NĂNG & THUẬT TOÁN (FUNCTIONAL REQUIREMENTS)

### 3.1. Logic Xử lý Dữ liệu (Data Processing)
**Đầu vào (Input):** File CSV với các cột (Case-insensitive):
* `PlayerName` (String): Tên golfer.
* `VGA` (String/Number): Mã VGA.
* `Handicap` (Number): Dùng để tính toán.
* `Group` (String): **Key phân loại chính** (Ví dụ: A, B).
* `IsRandom` (String): Nếu giá trị là "N" (hoặc "n") -> Chế độ Cố định. Khác "N" -> Chế độ Random.
* `PriorityFlyTime` (Number): Giá trị 1, 2, 3... (1 là ưu tiên cao nhất).

**Luồng Thuật toán (Algorithm Flow):**
1.  **Phân tách Bảng (Strict Grouping):** Sắp xếp danh sách người chơi theo Group (Alpha order: A -> B -> C). Hệ thống xử lý xong toàn bộ Group A mới sang Group B.
2.  **Chia nhóm trong từng Bảng:**
    * **Nhóm Cố định (Fixed):** Lọc người chơi `IsRandom=N`. Gom lại và **Sort ASC theo Handicap**. Cắt thành từng chunk 4 người.
    * **Nhóm Ngẫu nhiên (Random):** Lọc người chơi còn lại. **Shuffle (Xáo trộn)** ngẫu nhiên. Cắt thành từng chunk 4 người.
3.  **Sắp xếp Flight (Priority Sort):**
    * Trong danh sách các Flight của một Group, kiểm tra `PriorityFlyTime` của các thành viên.
    * Flight nào có thành viên sở hữu Priority thấp nhất (ví dụ 1) sẽ được đưa lên đầu danh sách của Group đó.
4.  **Gán Tee-time:**
    * Flight đầu tiên: `06:37`.
    * Các Flight tiếp theo: `+7 phút` mỗi flight.

### 3.2. Chức năng Export (Clipboard)
* **Trigger:** Nút bấm "Copy Excel".
* **Format:** TSV (Tab Separated Values).
- **Data Source:**
    - Google Sheets (CSV format).
    - Columns: `ID`, `Name`, `VGA`, `HDC`, `Group` (A/B/C), `Gender` (M/FM).
    - **Gender Logic:**
        - `M` = Male (Default).
        - `FM` = Female.
        - Used to generate appropriate avatars.
* **Schema:**
    `FlyNo` \t `TeeTime` \t `Group` \t `PlayerName` \t `VGA` \t `Handicap` \t `FlightHDC` \t **`AvatarURL`**
* **Yêu cầu đặc biệt:** Cột `AvatarURL` phải là link trực tiếp của ảnh đang hiển thị (từ DiceBear API) để đảm bảo tính nhất quán.

---

## 4. YÊU CẦU GIAO DIỆN (UI/UX REQUIREMENTS)

### 4.1. Hệ thống Design (Design System)
* **Font chữ:** Duy nhất **'Montserrat'** (Google Fonts).
    * Áp dụng cho cả Text thông thường và Số liệu kỹ thuật.
    * Trọng số: 400 (Regular), 600 (SemiBold), 800 (ExtraBold).
* **Màu sắc chủ đạo:**
    * Primary Green: `#064e3b` (Emerald 900) - Header Card, Background.
    * Accent Gold: `#fbbf24` (Amber 400) - Số Flight, Biểu tượng quan trọng.
    * Background: `#f0fdf4` (Mint Cream) - Nền trang web.
    * Badge Green: `#15803d` (Green 700) - Badge Handicap.

### 4.2. Chi tiết Thành phần (Components)
* **Flight Card:**
    * Style: Bo góc 16px, đổ bóng `shadow-md`, hiệu ứng Hover `translateY(-4px)`.
    * Header: Gradient xanh, chứa Số Flight (trong ô kính mờ) và Giờ Tee-time.
    * Body: Danh sách 4 golfer, ngăn cách bằng đường kẻ đứt (`border-dashed`).
    * Avatar: Tròn 44x44px, viền trắng. Dùng API DiceBear v9 (Style: Avataaars, tóc ngắn nam).
* **Magic Overlay (Màn hình chờ):**
    * Hiệu ứng HUD xoay tròn công nghệ.
    * Text chạy hiệu ứng giải mã ("Decipher") tên người chơi.
    * Tiến trình tải dữ liệu giả lập (Loading bar).

---

## 5. YÊU CẦU KỸ THUẬT & PHI CHỨC NĂNG (TECHNICAL & NON-FUNCTIONAL)

### 5.1. Tech Stack
* **Frontend Framework:** HTML5 thuần, không dùng React/Vue/Angular (để dễ deploy file tĩnh).
* **Styling:** Tailwind CSS (CDN v3.4).
* **Logic Script:** Vanilla JavaScript (ES6+).
* **Libraries:**
    * `PapaParse` (v5.4): Xử lý CSV.
    * `FontAwesome` (v6.4): Icon hệ thống.
* **Avatar Service:** DiceBear API v9.

### 5.2. Hiệu năng & Lưu trữ
* **Tốc độ:** Tải trang < 2s. Xử lý chia nhóm < 1s.
* **Persistence:** Dữ liệu sau khi chia nhóm phải được lưu vào `localStorage`. Khi user tải lại trang (F5), kết quả cũ phải giữ nguyên. Chỉ thay đổi khi user chủ động bấm "Chia Lại".

---

## 6. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA)

| Mã | Tiêu chí | Kết quả mong đợi (Pass) |
| :--- | :--- | :--- |
| **AC01** | **Strict Group Logic** | Các Flight của Group A phải luôn nằm trên/trước các Flight của Group B. Không có trường hợp xen kẽ (A-B-A). |
| **AC02** | **Fixed Player Sorting** | Nếu 4 người chơi `IsRandom=N` ở cùng Group, họ phải chung 1 Flight và thứ tự từ trên xuống dưới là HDC thấp -> cao. |
| **AC03** | **Priority Handling** | Nếu Flight X có thành viên Priority=1, Flight X phải nằm đầu tiên trong danh sách của Group đó. |
| **AC04** | **Data Export** | Khi paste dữ liệu copy vào Google Sheet, cột cuối cùng phải là Link ảnh Avatar (bắt đầu bằng `https://api.dicebear.com...`). |
| **AC05** | **Visual Consistency** | Không còn xuất hiện font chữ `Share Tech Mono`. Tất cả số liệu (Giờ, Flight No) phải là font `Montserrat`. |
| **AC06** | **Safety Avatar** | Avatar tạo ra không được có tóc dài/nữ tính (do filter tham số `shortFlat`, `theCaesar`...). |

---
*Tài liệu này được dùng làm chuẩn "Single Source of Truth" cho đội ngũ phát triển và kiểm thử.*