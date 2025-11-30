Chào bạn, với vai trò là **Product Manager** và **Business Analyst**, tôi xin gửi đến bạn bản **Đặc tả Yêu cầu Sản phẩm (PRD)** phiên bản hoàn chỉnh nhất (Version 4.0 - Final Release).

Tài liệu này được cấu trúc chuẩn hóa để bạn có thể import vào các công cụ quản lý dự án (như Jira, Confluence, Google Docs, hoặc Notion). Nó bao gồm tất cả các thay đổi về giao diện (Card Style, Group Badge), logic tìm kiếm tiếng Việt và Responsive.

---

# PRODUCT REQUIREMENTS DOCUMENT (PRD)
**Dự án:** WOB Outing 2025 - Official Tournament Pairings  
**Phiên bản:** 4.0 (Final Release)  
**Ngày phát hành:** 29/11/2025  
**Trạng thái:** Ready for Production  
**Owner:** DuongLX  

---

## 1. TỔNG QUAN (EXECUTIVE SUMMARY)

### 1.1. Mục tiêu sản phẩm
Xây dựng Landing Page hiển thị thông tin chia nhóm (Pairings) cho giải golf **WOB Outing 2025**. Hệ thống đóng vai trò là "Bảng thông báo điện tử" giúp Golfer tra cứu nhanh lịch thi đấu, nhóm đấu và thông tin tiệc trưa ngay trên thiết bị di động.

### 1.2. Đối tượng sử dụng
* **Golfer:** Tra cứu giờ Tee Time, bạn cùng nhóm (Flight), Handicap và vị trí nhà hàng tiệc.
* **Ban tổ chức:** Quản lý và cập nhật dữ liệu thi đấu theo thời gian thực qua Google Sheets.

---

## 2. TRẢI NGHIỆM NGƯỜI DÙNG (UX FLOW & DESIGN)

### 2.1. Phong cách thiết kế (Design System)
* **Font chữ:** `Montserrat` (Toàn bộ trang web).
* **Màu sắc chủ đạo:**
    * **Primary Green:** `#065f46` (Header, Gradient).
    * **Accent Yellow:** `#facc15` (Icon cúp, Giờ Tee Time).
    * **Background:** `#f3f4f6` (Xám nhạt hiện đại).
* **Họa tiết nền:** Pattern sân golf (chấm bi/kẻ sọc mờ) màu xanh nhạt.

### 2.2. Bố cục Responsive (Mobile-First)
* **Meta Viewport:** Thiết lập `maximum-scale=1.0` để ngăn chặn lỗi tự động zoom input trên iOS.
* **Breakpoint:**
    * **Mobile:** Header xếp dọc (Logo -> Gala Button -> Search). Grid 1 cột.
    * **Tablet/Desktop:** Header xếp ngang (Logo - Search - Gala Button). Grid 3-4 cột.

### 2.3. Các thành phần giao diện chính

#### A. Màn hình chờ (Loading Overlay)
* **Hiệu ứng:** Màn hình tối màu, vòng tròn spinner xoay quanh icon quả bóng golf.
* **Thông báo:** "SYNCING DATA... Connecting to server..." tạo cảm giác công nghệ, chuyên nghiệp.
* **Thời lượng:** Tự động tắt sau khi dữ liệu từ Google Sheet được tải xong.

#### B. Header & Navigation
1.  **Logo & Thông tin giải:** Hiển thị tên giải "WOB OUTING 2025", ngày 28/11, sân Thủ Đức.
2.  **Nút thông tin Tiệc (Gala Lunch Button):**
    * *Mobile:* Thu gọn, hiển thị icon Tiệc + Giờ (12:30).
    * *Desktop:* Hiển thị đầy đủ Tên nhà hàng (Seasan) + Người Booking (Mr HảiLH).
    * *Hành động:* Click mở Google Maps.
3.  **Thanh tìm kiếm (Smart Search):**
    * Chiếm 100% chiều rộng trên Mobile.
    * Font size 16px (chuẩn Mobile).

#### C. Thẻ nhóm đấu (Flight Card) - **(Trọng tâm thiết kế)**
Mỗi thẻ Flight được thiết kế dạng "Scorecard" cao cấp:
* **Header Card:**
    * Nền: Gradient xanh đậm (`#065f46`).
    * Trái: Số Flight (trong ô vuông kính mờ) + Giờ Tee Time.
    * Phải: **Group Badge** (Nếu có dữ liệu).
        * Style: Nền trắng (`bg-white`), Chữ xanh (`text-green-800`), bo góc, font đậm.
        * Ví dụ hiển thị: "GROUP A".
* **Sub-Header:** Dòng tiêu đề nhỏ ghi "Golfer" và "HDC" để định danh cột.
* **Body Card:** Danh sách người chơi.
    * Mỗi người chơi là một hàng (Row) có đường kẻ nét đứt (`dashed`) ngăn cách.
    * Avatar: Hình tròn, có viền trắng và bóng đổ.
    * Thông tin: Tên (Bold), VGA (Badge xám nhỏ).
    * Handicap: Hiển thị trong ô vuông bo góc màu xanh lá (`badge-hdc-square`).

---

## 3. YÊU CẦU CHỨC NĂNG (FUNCTIONAL SPECS)

### 3.1. Quản lý dữ liệu (Data Integration)
* **Nguồn:** Google Sheet (Publish to CSV).
* **URL:** `https://docs.google.com/spreadsheets/d/e/2PACX-1vSl-c25LTQMr5-usqXvTRhhRXgpf7w9hzlX8JjXLVppuDcD8vIXg5TXAc31kWRe3aXBjRN4Ikpdukl3/pub?output=csv`
* **Dữ liệu dự phòng (Backup):** Hardcode sẵn một bộ dữ liệu mẫu trong code JS để đảm bảo web không bao giờ trắng trang nếu Google Sheet lỗi.

### 3.2. Tính năng Tìm kiếm (Smart Search Logic)
* **Input:** Nhập tên hoặc mã VGA.
* **Thuật toán:** `removeVietnameseTones` (Chuẩn hóa tiếng Việt).
    * Input: "Dương", "duong", "DƯƠNG" -> Output: "duong".
    * So sánh chuỗi đã chuẩn hóa để trả về kết quả chính xác không phân biệt hoa thường/dấu.
* **Kết quả:** Lọc danh sách Flight Card ngay lập tức (Real-time). Hiển thị số lượng kết quả tìm thấy.

### 3.3. Xử lý hình ảnh (Avatar Generation)
* **Cơ chế:** Sử dụng API **DiceBear v9**.
* **Fix lỗi v9:** Sử dụng danh sách style tóc mới (`shortFlat`, `theCaesar`, `sides`...) thay vì các mã cũ bị lỗi 400.
* **Fallback:** Nếu API lỗi, hiển thị Avatar ký tự (Initials) từ `ui-avatars.com`.

### 3.4. Tính năng cappture màn hình tất cả các card thành một tấm hình để lưu về máy 
* ** Xuất thẻ ** <main class="flex-grow container mx-auto px-4 py-6 relative z-0"> thành một tấm hình

---

## 4. CẤU TRÚC DỮ LIỆU (DATA SCHEMA)

Dữ liệu đầu vào từ file CSV phải tuân thủ cấu trúc sau:

| Tên Cột | Kiểu | Bắt buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `FlyNo` | Number | Có | Số thứ tự nhóm (1, 2, 3...) |
| `TeeTime` | String | Có | Giờ phát bóng (06:37, 06:44...) |
| `Group` | String | Không | Tên bảng đấu (A, B, C...). Nếu trống sẽ ẩn Badge. |
| `PlayerName` | String | Có | Họ và tên Golfer |
| `VGA` | String | Có | Mã VGA (VD: 12345) |
| `Handicap` | Number | Có | Điểm chấp hiện tại |
| `AvatarURL` | String | Không | Link ảnh riêng (nếu có). Để trống tự sinh. |

---

## 5. MÔ TẢ KỸ THUẬT (TECHNICAL STACK)

* **Frontend Framework:** Không sử dụng (Vanilla JS) để tối ưu tốc độ tải.
* **Styling:** **Tailwind CSS (CDN)**.
* **CSS Custom:** Viết riêng cho class `.flight-card`, `.flight-header`, `.group-badge` để đạt độ chi tiết cao về đổ bóng và gradient.
* **Data Parser:** **PapaParse 5.4** (Xử lý CSV).
* **Icons:** FontAwesome 6.4.

---

## 6. KIỂM THỬ & NGHIỆM THU (ACCEPTANCE CRITERIA)

1.  **Hiển thị:** Trang web tải dưới 2 giây. Hiệu ứng Loading xuất hiện và biến mất mượt mà.
2.  **Dữ liệu:** Dữ liệu hiển thị đúng với file Google Sheet (bao gồm cả cột Group mới).
3.  **Giao diện Card:**
    * Header màu xanh Gradient.
    * Có Badge "GROUP X" màu trắng chữ xanh nằm bên phải Header.
    * Các golfer được ngăn cách bởi đường kẻ nét đứt.
4.  **Tìm kiếm:** Gõ "duong" phải ra "Lê Xuân Dương".
5.  **Responsive:**
    * Trên iPhone: Ô tìm kiếm không bị zoom cận cảnh khi nhấn vào.
    * Nút Gala Lunch hiển thị gọn gàng, không vỡ layout.

---
*Tài liệu này xác nhận cấu hình cuối cùng của hệ thống. Mọi thay đổi sau thời điểm này cần được cập nhật vào phiên bản PRD tiếp theo.*