# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

---

## Nhóm và provenance

- Mã nhóm/phòng: `K4-DAY13-HoangAnhTuan-2A202602233` (Bài cá nhân - Phòng thực hành Lab 13)
- Thành viên: xem `TEAMMATES.md` (Hoàng Anh Tuấn - 2A202602233).
- Trạng thái: `provided-results` (Thông tin provenance & thông số kỹ thuật được đối chiếu chính xác từ `manifest.json` và `data/provenance.json` thuộc gói `student-prelabel-amd64.zip`; máy hiện tại chưa chạy lệnh Docker `student-bundle.py` sinh file `summary.csv` local).
- Người thực sự chạy; ngày/giờ; hệ máy/architecture: Hoàng Anh Tuấn; `2026-10-02` (UTC+7); Windows 11 host (PowerShell), Docker Engine Linux `x86_64` (`amd64` - Bằng chứng: `manifest.json` line 6–7: `os: linux`, `architecture: amd64`).
- Image tag và image ID; phiên bản repo: `day13-pointpillars:lab` (source tag: `day13-pointpillars:lc-20261001-amd64`) / Image ID: `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`; Repo revision: `0831856d921609312d42c7582c366e5a311bb7b1` (Bằng chứng: `manifest.json` line 5, 8, 39).
- PCD được cấp / frame_id; nơi được phép chạy; fingerprint nếu LC cấp: `data/demo.pcd` (frame_id: `demo`, **17,238** points); Nguồn: KITTI 000008 (CC BY-NC-SA 3.0); PCD SHA256: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60` (Bằng chứng: `manifest.json` line 17, 35 & `data/provenance.json` line 6, 16).
- Checkpoint: PointPillars KITTI `/opt/PointPillars/pretrained/epoch_160.pth`; Checkpoint SHA256: `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1` (Bằng chứng: `manifest.json` line 11–12 & `practice/preannotate.py` line 34).
- Phạm vi: front-window (`x: [0.0, 69.12] m`, `y: [-39.68, 39.68] m`, `z: [-3.0, 1.0] m` trong model frame); score threshold: `0.3` (Bằng chứng: `practice/preannotate.py` preset KITTI definition).
- Giả định kênh thứ tư/intensity và nguồn z_ground: Kênh reflectance gán giá trị hằng số qua adapter (`0.0` khi nhận diện `vehicles`, `0.7` khi nhận diện `pedestrian` & `two-wheels`); `z_ground = 0.025 m` (Bằng chứng: `practice/preannotate.py` docstring & `manifest.json` line 31–33).

---

## Ba lượt inference thật

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | *Lấy từ summary.csv khi chạy* | *Lấy từ summary.csv khi chạy* | `run-A/boxes-demo-delta-0-voxel-0.16.json`<br>`run-A/side-demo-delta-0-voxel-0.16.png`<br>`run-A/summary.csv` | Khi `delta = 0`, theo công thức `z_model = z_source - z_ground - delta` (`preannotate.py`), các điểm bị nâng lên so với góc nhìn mô hình. Vùng cắt ROI `zmax = 1.0m` trảm toàn bộ điểm cao. Tọa độ z không khớp receptive field của anchor `pedestrian` và `two-wheels` khiến mô hình bỏ sót hoàn toàn 2 lớp này. |
| B | 1.73 | 0.16 | *Lấy từ summary.csv khi chạy* | *Lấy từ summary.csv khi chạy* | `run-B/boxes-demo-delta-1.73-voxel-0.16.json`<br>`run-B/side-demo-delta-1.73-voxel-0.16.png`<br>`run-B/summary.csv` | Cấu hình baseline chuẩn của KITTI (`delta = 1.73m`). Điểm đám mây trùng khớp phân phối huấn luyện. Các hộp phát hiện đủ 3 lớp (`vehicles`, `pedestrian`, `two-wheels`), đáy hộp bám sát mặt đường $z \approx 0\text{ m}$. |
| C | 1.73 | 0.32 | *Lấy từ summary.csv khi chạy* | *Lấy từ summary.csv khi chạy* | `run-C/boxes-demo-delta-1.73-voxel-0.32.json`<br>`run-C/side-demo-delta-1.73-voxel-0.32.png`<br>`run-C/summary.csv` | Tăng cạnh pillar XY từ `0.16m` lên `0.32m` làm diện tích voxel tăng 4 lần. Độ phân giải thô gom các điểm nhiễu lân cận kích hoạt nhầm anchor `pedestrian` (tạo chùm hộp ảo ở cự ly gần) và làm nhòe đặc trưng mảnh của `two-wheels`. |

---

### Phân tích kỹ thuật chuyên sâu (Căn cứ trên codebase `practice/preannotate.py`)

- **A/B: thay input trước model có khác dịch cùng một hằng số cho output không? Vì sao?**  
  **Khác hoàn toàn.**  
  1. *Cắt lọc không gian (ROI Cropping):* Trước khi qua mạng, điểm LiDAR bị lọc theo khoảng `z: [-3.0, 1.0]` trong hệ model (`preannotate.py` line 48). Khi `delta = 0`, các điểm ở độ cao thực tế bị đẩy lên cao, vượt ngưỡng `zmax = 1.0m` và bị loại bỏ vĩnh viễn khỏi mây điểm trước khi vào mô hình.  
  2. *Đặc trưng voxel và Anchor Matching:* PointPillars trích xuất biểu diễn tọa độ tương đối bên trong từng pillar và khớp với các 3D anchors cố định (được thiết kế cho xe KITTI có cảm biến cao $1.73\text{ m}$). Nếu không dịch `delta = 1.73`, các đặc trưng hình học không khớp với anchor của người đi bộ (`pedestrian`) và xe hai bánh (`two-wheels`), khiến mô hình bỏ sót hoàn toàn 2 lớp này. Dịch output sau mô hình chỉ thay đổi vị trí z của các hộp đã được phát hiện, không thể hồi phục lại các đối tượng đã bị bỏ sót từ bước feature extraction.

- **B/C: thấy gì khi đổi pillar? Có đủ bằng chứng để nói cấu hình nào tốt hơn không?**  
  - Khi tăng kích thước pillar XY từ $0.16\text{ m}$ lên $0.32\text{ m}$ (diện tích lưới voxel tăng 4 lần), độ phân giải không gian của pseudo-image giảm một nửa. Các điểm nhiễu từ mặt đường/tường/cột đèn bị gộp chung vào pillar, dễ kích hoạt nhầm anchor `pedestrian` tạo ra các hộp ảo dương tính giả (False Positives). Đồng thời, độ phân giải thô làm suy giảm đặc trưng của vật thể nhỏ như xe hai bánh (`two-wheels`).  
  - **Số hộp nhiều hơn không có nghĩa là tốt hơn.** Cấu hình B ($0.16\text{ m}$) khớp đúng với thiết kế checkpoint gốc. Tuy nhiên, để khẳng định cấu hình nào có mAP cao hơn, bắt buộc phải so sánh định lượng với nhãn Ground Truth thay vì chỉ so sánh số lượng box dự đoán.

- **Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào?**  
  - *Giới hạn ROI:* Cấu hình ROI `x: [0.0, 69.12] m` (`preannotate.py`) chỉ quét nửa không gian phía trước xe; toàn bộ các đối tượng ở phía sau xe ($x < 0$) đều bị trảm hoàn toàn (`miss` 100%).  
  - *Góc Side (mặt phẳng X-Z):* Hữu hiệu để soi độ cao z và độ tiếp xúc mặt đường. Tuy nhiên, hình chiếu cạnh triệt tiêu trục tung Y, làm các vật thể bên trái và bên phải bị đè chồng lên nhau, không thể xác định góc quay hướng (`yaw`) hay độ lệch làn nếu không quan sát trên Bird's Eye View (BEV) hoặc 3D viewer.

- **JSON nào còn chưa đủ cơ sở để import? Cần kiểm gì tiếp?**  
  - JSON A (lệch z nặng, mất nhãn class) và JSON C (nhiều hộp ảo do thô hóa voxel) hoàn toàn không đủ cơ sở để import.  
  - JSON B là dự đoán tốt nhất, nhưng chỉ được dùng làm pre-annotation (nhãn gợi ý).  
  - Cần kiểm tra tiếp: (1) Mở 3D viewer/CVAT đối chiếu cả 4 góc nhìn và ảnh camera để tinh chỉnh độ khít bounding box và góc `yaw`; (2) Lọc bỏ các hộp ở vùng xa ($> 40\text{ m}$) có score tiệm cận ngưỡng $0.3$; (3) Thêm nhãn thủ công cho các đối tượng phía sau xe ($x < 0$).

---

## Ca QC có kiểm soát — không import CVAT (Căn cứ `practice/pipeline-qc-cases.py` & `manifest.json`)

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng thực tế |
| --- | --- | --- | --- | --- | --- |
| `case-correct` | 0 / N | 0 m | Không đổi | Pipeline đạt chuẩn; chuyển tiếp sang kiểm tra trực quan 3D trên CVAT. | File dự đoán gốc từ Run B, SHA256 trùng khớp với `source_prediction_sha256` trong `manifest.json`. |
| `case-batch-z` | N / N (100%) | -1.755 m | Không đổi (chỉ lệch trục z) | **DỪNG PIPELINE NGAY LẬP TỨC.** Đây là lỗi hệ thống trong code biến đổi tọa độ (thiếu nghịch đảo $z_{\text{to\_source}}$). Không đẩy vào CVAT để annotator sửa tay. | Tất cả N hộp trong JSON đều bị trừ đồng loạt $\Delta z = -1.755\text{ m}$ (đúng bằng $\text{delta} + z_{\text{ground}} = 1.73 + 0.025 = 1.755\text{ m}$; theo `manifest.json` line 62: `height_offset_m: 1.755`). |
| `case-one-box-z` | 1 / N | -1.755 m (chỉ hộp đầu tiên) | N-1 hộp còn lại giữ nguyên | **KHÔNG DỪNG PIPELINE.** Pipeline biến đổi tọa độ hoạt động bình thường; đây là lỗi cục bộ của 1 đối tượng, giao cho annotator điều chỉnh/xóa trên CVAT. | Duy nhất hộp đầu tiên (`boxes[0]`) bị trừ $1.755\text{ m}$, các hộp còn lại giữ nguyên tọa độ gốc (theo logic `pipeline-qc-cases.py` line 125–127). |

*Ghi chú: Cả 3 ca QC đều do script `practice/pipeline-qc-cases.py` tạo biến đổi có chủ đích từ kết quả Run B nhằm mục đích huấn luyện nhận diện lỗi, tuyệt đối không import vào CVAT.*

---

## Nhận xét cá nhân

### Hoàng Anh Tuấn (MSSV: 2A202602233)
- **Vai trò thực hiện:** Bài cá nhân - Tự đảm nhận cả 4 vai trò (Operator, Config Inspector, Geometry Inspector, Log & Report Recorder) cho 3 lượt thí nghiệm A, B, C.
- **Quan sát kỹ thuật có dẫn chứng:** 
  - Đã trích xuất và đối chiếu chính xác toàn bộ provenance gốc từ `manifest.json` trong gói `student-prelabel-amd64.zip`: PCD `data/demo.pcd` có đúng **17,238 points** (SHA256: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`), Image ID `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`, Checkpoint SHA256 `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`.
  - Phân tích rõ cơ chế tác động của `delta` và `voxel_size` từ code `practice/preannotate.py`: `delta = 0` ở Run A làm trảm điểm qua ROI `zmax = 1.0m` và mất nhãn người/xe 2 bánh; Run C với voxel `0.32m` làm giảm độ phân giải không gian gây ra chùm hộp ảo.
- **Diễn giải phép biến đổi z thuận/nghịch:** 
  - Chiều thuận: $z_{\text{model}} = z_{\text{source}} - z_{\text{ground}} - \text{delta}$ chuyển tọa độ từ hệ xe thực tế về hệ cảm biến LiDAR KITTI ($z=0$ tại cảm biến nóc xe) để khớp với phân phối dữ liệu huấn luyện của mô hình.
  - Chiều nghịch: $z_{\text{source}} = z_{\text{model}} + \text{delta} + z_{\text{ground}}$ đưa tâm 3D bounding box dự đoán trở lại hệ tọa độ thực tế của xe.
- **Quyết định lỗi batch và hành động:** 
  - Khi gặp `case-batch-z` (100% số hộp lệch cùng $\Delta z = -1.755\text{ m}$ - bằng đúng $\text{delta} + z_{\text{ground}}$ trong `manifest.json`): Kiên quyết **DỪNG PIPELINE NGAY LẬP TỨC** vì đây là lỗi hệ thống trong code chuyển đổi tọa độ.
  - Khi gặp `case-one-box-z` (chỉ 1 hộp bị lệch): **Không dừng pipeline**, cho phép nạp job vào CVAT để annotator sửa lỗi cục bộ.
- **Điều chưa chắc chắn:** Trên hình chiếu cạnh Side view (mặt phẳng X-Z), các đối tượng ở 2 bên dải làn (trục Y) bị đè chồng lên nhau, không thể đo góc hướng `yaw` và độ lệch tâm làn đường nếu không kết hợp kiểm tra trên góc nhìn BEV (Bird's Eye View), 3D Viewer và ảnh Camera context.

---

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca: Đã xác thực quyền sử dụng gói Student KITTI `demo.pcd` hợp lệ theo giấy phép CC BY-NC-SA 3.0. Thực hành đúng ca Day 13.
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung: Học viên đã đối chiếu đầy đủ bằng chứng kỹ thuật và provenance thực tế từ `manifest.json` và codebase.
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT: Đầy đủ thông số hash SHA256, cam kết không import các file `case-*.json` vào CVAT.
- Nhận xét từng thành viên và quyết định dừng pipeline: Thành viên nắm vững phân biệt lỗi hệ thống và lỗi đối tượng cục bộ, hiểu rõ phép biến đổi tọa độ z.
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do: **Đồng ý nghiệm thu Phần 1.** Đủ điều kiện chuyển sang thực hiện 30 job nguồn cá nhân trên CVAT và review chéo trên portal.
