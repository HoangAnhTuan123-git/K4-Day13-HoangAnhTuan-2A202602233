# Thành viên nhóm — Day 13

Điền trong thư mục nhóm private, không commit bản có thông tin cá nhân lên repo public.

Mã nhóm/phòng: `K4-DAY13-HoangAnhTuan-2A202602233` (Bài cá nhân - Phòng thực hành Lab 13)

| Họ và tên | MSSV | Vai trò lượt A | Vai trò lượt B | Vai trò lượt C |
| --- | --- | --- | --- | --- |
| Hoàng Anh Tuấn | 2A202602233 | Operator, Config Inspector, Geometry Inspector, Log & Report Recorder | Operator, Config Inspector, Geometry Inspector, Log & Report Recorder | Operator, Config Inspector, Geometry Inspector, Log & Report Recorder |

---

## Hướng dẫn tra cứu evidence và cách phân công vai trò (Dành cho bản mẫu)

> [!NOTE]
> Phần hướng dẫn dưới đây giải thích nguồn dữ liệu (evidence) và phương pháp xác định vai trò để học viên hoàn thiện file này một cách chuẩn xác nhất.

### 1. Xem evidence ở đâu để điền thông tin?
- **Mã nhóm / Phòng thực hành:** Lấy theo định danh phân ca do Lab Coach (LC) công bố đầu buổi (ví dụ: `K4-DAY13-GROUP-XX` hoặc mã số bàn/máy phòng lab).
- **Họ tên & MSSV:** Đối chiếu với thẻ sinh viên hoặc danh sách điểm danh trên portal môn học.
- **Bằng chứng vai trò thực hiện:**
  - Lịch sử lệnh terminal (Shell History) trên máy thực hành: ghi nhận ai ngồi máy gõ lệnh (`Operator`).
  - Terminal output hoặc file log: hiển thị thời điểm chạy tương ứng từng lượt.
  - Phân công nội bộ nhóm trước khi bấm nút bắt đầu phiên trên portal.

### 2. Cách xác định và phân bổ vai trò thế nào?
Để đảm bảo tất cả thành viên đều nắm vững kỹ thuật, nhóm 4 người cần phân công 4 vai trò rõ ràng và luân chuyển qua 3 lượt chạy:

1. **Operator (Kỹ sư vận hành):**
   - *Nhiệm vụ:* Trực tiếp mở terminal, cấu hình biến môi trường, chạy câu lệnh Docker hoặc script runner `student-bundle.py`, kiểm tra mã thoát (exit code 0).
   - *Cách làm:* Kiểm tra `docker info`, gõ lệnh chạy tương ứng từng lượt, theo dõi tiến trình sinh file trong thư mục output.
2. **Config Inspector (Kỹ sư kiểm tra cấu hình):**
   - *Nhiệm vụ:* Rà soát các cờ tham số của lượt chạy trước khi gõ lệnh (kiểm tra `delta`, `voxel-size`, ROI, score threshold `0.3`).
   - *Cách làm:* Đọc kỹ đề bài trong `PRE-LABEL.md`, đối chiếu từng tham số dòng lệnh CLI xem đã đổi đúng duy nhất một biến mục tiêu chưa, tránh nhầm lẫn giữa các lượt A/B/C.
3. **Geometry Inspector (Kỹ sư kiểm tra hình học):**
   - *Nhiệm vụ:* Ngay khi có kết quả, mở file ảnh Side view (`side-*.png`) và file JSON dự đoán (`boxes-*.json`) để soi bounding box.
   - *Cách làm:* Kiểm tra độ bám mặt đường của đáy hộp, đếm số lượng hộp, phát hiện các hộp chìm dưới lòng đất hoặc bay lơ lửng, phát hiện hiện tượng chùm hộp ảo.
4. **Log & Report Recorder (Kỹ sư thư ký & Tổng hợp dữ liệu):**
   - *Nhiệm vụ:* Trích xuất số liệu thống kê từ `summary.csv` và `smoke.json`, ghi chép thời gian chạy, tính toán chênh lệch, đôn đốc các thành viên viết nhận xét cá nhân và chuẩn bị báo cáo nộp cho LC.
   - *Cách làm:* Mở `summary.csv` lấy `n_boxes`, `mean_z`; kiểm tra hash SHA256; điền dữ liệu vào `PRE-LABEL-REPORT.md`.

*Quy tắc luân chuyển:* Ba vai trò Operator, Config Inspector và Geometry Inspector hoán đổi vòng tròn qua các lượt A $\to$ B $\to$ C. Vai trò Log & Report Recorder có thể cố định cho một thành viên để bảo đảm tính nhất quán của dữ liệu báo cáo, hoặc hỗ trợ chéo các thành viên khác khi hoàn thành lượt chạy.
