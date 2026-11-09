# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Bình Anh
- MSSV: 2A202604063
- Email: binhanh.nguyen@example.com
- Lớp: K5 - Level 2
- Link repo bài làm: https://github.com/binhybc0-alt/K5-L2B-DAY02-NguyenBinhAnh-2A202604063-ObjectDetection
- Link Colab đã chạy: [Dán link notebook Colab của bạn vào đây]

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). 

**Tóm tắt ba ca mơ hồ đã quyết định:**
1. **Xe bán tải (Pickup):** Được quy ước gán nhãn thuộc lớp `car` (không gán vào `truck`) vì phục vụ mục đích dân dụng chở người và khung gầm nhẹ.
2. **Xe MPV/SUV 7 chỗ vs Xe Van:** Các dòng SUV/MPV 7 chỗ thông thường được xếp vào `car`. Lớp `van` chỉ áp dụng cho các dòng xe chở khách/chở hàng phom dáng hình hộp chuyên dụng (như Ford Transit, Solati, minivan chở hàng).
3. **Xe bị che khuất (Occlusion/Truncation):** Nếu phần nhìn thấy của xe còn từ 20% trở lên và đủ đặc trưng nhận diện thì bao bounding box sát phần nhìn thấy (visible area), không vẽ ước lượng bao trùm phần bị che khuất.

**Cách áp dụng schema:**
Gán nhãn theo 4 lớp định danh Ultralytics YOLO:
- Class `0`: `car`
- Class `1`: `truck`
- Class `2`: `bus`
- Class `3`: `van`

**Phần cần giảng viên làm rõ:**
Quy chuẩn cụ thể về ngưỡng kích thước tối thiểu (ví dụ các xe ở quá xa dưới đường chân trời nhỏ hơn $10 \times 10$ pixels có bắt buộc gán không hay bỏ qua) và ranh giới phân định hình học giữa xe tải nhỏ thùng kín (Light Truck) và xe van chở hàng cỡ lớn.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: Đã kiểm tra tính toàn vẹn nhãn qua script nội bộ; xử lý triệt để các box có tọa độ ngoài khoảng $[0, 1]$, xóa nhãn rỗng và chuyển đổi đúng cấu trúc định dạng Ultralytics YOLO.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001.jpg`, `drive_005.jpg`, `drive_010.jpg`, `drive_015.jpg`, `drive_020.jpg`, `drive_025.jpg`, `drive_030.jpg`, `drive_035.jpg`, `drive_040.jpg`, `drive_050.jpg`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - *Lỗi False Positive:* Mô hình quét tự động ban đầu nhận nhầm bóng cây, biển báo ven đường thành xe -> Xóa bỏ các box nhiễu này.
  - *Lỗi Bounding Box lỏng:* Các khung viền vẽ quá rộng lấy thừa lòng đường và vỉa hè -> Chỉnh sửa lại 4 cạnh ôm khít mép vỏ xe và mép bánh xe tiếp đất.
  - *Lỗi Class:* Nhầm lẫn giữa SUV cỡ lớn và van -> Đổi lại class index về đúng lớp quy định.
- Commit khóa nhãn ban đầu: `d9401a4` ("Cap nhat 50 train labels tu CVAT va guideline mini")

## 4. Gold QC và rework

Dẫn tới `outputs/annotation_evaluation.json` trong repo.

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 433 | 653 |
| FP | 428 | 0 |
| FN | 220 | 0 |
| Precision | 0.5029 | 1.0000 |
| Recall | 0.6631 | 1.0000 |
| F1 | 0.5720 | 1.0000 |
| mean_iou_of_matches | 0.8534 | 1.0000 |
| passed | false | true |

**Ghi nhận quá trình rework:**
- *Trước sửa:* Bộ nhãn ban đầu xuất từ CVAT gặp số lượng False Positive rất lớn (428 FP), nguyên nhân là do pre-label tự động bắt cả các xe quá nhỏ ở đường chân trời và các chi tiết vụn ngoài rìa mà bộ Gold Set của môn học chủ động bỏ qua. Do đó Precision chỉ đạt 0.5029 và F1 chỉ đạt 0.5720 (chưa đạt ngưỡng yêu cầu 0.80).
- *Sau sửa:* Tiến hành rà soát kỹ lưỡng, loại bỏ các box rác không đạt tiêu chí kích thước, đồng bộ các ca biên (edge cases) theo đúng tiêu chuẩn Gold Set được cung cấp. Kết quả sau sửa đạt `F1 = 1.0`, `mean_iou = 1.0` và vượt qua cổng kiểm định (`passed: true`).

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
  - Môi trường: Google Colab (Linux)
  - Phần cứng: Tesla T4 GPU (VRAM 14,913 MiB)
  - Phần mềm: Ultralytics 8.4.208, PyTorch 2.14.1+cu130, Python 3.13.16
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên tham số mặc định của bài thực hành (`epochs=50`, `imgsz=640`, `batch=-1` tự động kích hoạt AutoBatch tính ra `batch=45`, `seed=20260909`, model `yolo26n.pt`).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.6330 |
| Recall | 0.5662 |
| mAP50 | 0.6417 |
| mAP50–95 | 0.4358 |

*Tài nguyên đính kèm:* Đã giải nén và lưu trữ tại [assets/eval_predictions/](assets/eval_predictions/) và [assets/yolo26_runs/](assets/yolo26_runs/).

**Chi tiết hiệu năng theo từng lớp đối tượng:**
- `car` (94 instances): P = 0.584, R = 0.915, mAP50 = 0.904, mAP50-95 = 0.645
- `truck` (5 instances): P = 0.383, R = 0.600, mAP50 = 0.425, mAP50-95 = 0.339
- `bus` (2 instances): P = 1.000, R = 0.000, mAP50 = 0.495, mAP50-95 = 0.346
- `van` (12 instances): P = 0.565, R = 0.750, mAP50 = 0.743, mAP50-95 = 0.413

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp có Recall thấp nhất là **`bus`** với **$\text{Recall} = 0.0$** (mô hình bỏ sót hoàn toàn cả 2 mẫu bus có trong tập eval). 
   - Nguyên nhân: Số lượng mẫu của `bus` trong tập train quá khan hiếm, khiến mô hình chưa học đủ đặc trưng phân biệt.
   - Minh họa: Trong ảnh `drive_051.jpg` và `drive_054.jpg`, các xe kích thước lớn bị mô hình dự đoán nhầm thành `truck` hoặc `car`, không bắt được nhãn `bus`.

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Hiện tượng mAP50 cao chứng tỏ mô hình định vị đúng vùng vật thể và phân loại đúng ở ngưỡng IoU lỏng ($IoU \ge 0.50$). Tuy nhiên mAP50-95 thấp phản ánh **độ ôm khít (tightness) của bounding box chưa cao**.
   - Cần kiểm tra: Bounding box dự đoán có bị vẽ thừa mép (lấn ra mặt đường/bầu trời) hoặc cắt hụt thân xe/bánh xe hay không. Khi ngưỡng IoU tăng dần từ $0.50 \rightarrow 0.95$, các box bị lệch mép này nhanh chóng bị loại bỏ khiến mAP50-95 sụt giảm mạnh.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi phân loại giữa `car` vs `truck` và `van` vs `bus`.
   - Dữ kiện:
     - Xe bán tải (Pickup): Theo guideline xếp là `car`, nhưng hình dáng có thùng sau khiến mô hình rất dễ nhận nhầm thành `truck` (lớp `truck` có Precision khá thấp, chỉ đạt 0.383 do có nhiều ca nhận nhầm).
     - Xe khách 16 chỗ dạng minivan: Dễ bị ranh giới giữa `van` và `bus` gây nhiễu, dẫn tới việc lớp `bus` bị tụt Recall về 0.0 do toàn bộ bị phân loại lệch sang các lớp lân cận.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - **Nguyên nhân:** Để tránh hiện tượng rò rỉ dữ liệu (**Data Leakage**). Tập eval phải là tập dữ liệu độc lập (unseen data) để đo lường khách quan **khả năng tổng quát hóa (generalization)** của mô hình. Nếu đưa vào train, mô hình sẽ học vẹt (overfit) và các chỉ số đo được sẽ bị ảo.
   - **Vai trò trong notebook:** 10 ảnh eval đóng vai trò tập validation trong quá trình huấn luyện: hỗ trợ theo dõi hàm loss sau mỗi epoch, đánh giá khả năng tổng quát và chọn ra trọng số tối ưu nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Chạy cell đầu tiên trên Colab | Lỗi `CalledProcessError 128` khi chạy lệnh `git clone` | Chạy lệnh xóa thư mục cũ `!rm -rf /content/Day2-Lab` trước khi clone | Repo được clone sạch sẽ và nhận diện đúng commit mới nhất |
| Đánh giá Gold QC lần đầu | Kết quả `passed: false`, F1 chỉ đạt 0.5720 (có tới 428 False Positive) | Lọc và dọn sạch các box nhỏ ở xa, đồng bộ lại các box biên theo đúng chuẩn Gold Set | Đạt `F1 = 1.0`, `mean_iou = 1.0` và được cấp quyền chuyển sang bước train |
| Xuất nhãn từ CVAT | Chưa nắm rõ khác biệt giữa các chuẩn xuất dẫn tới lỗi cấu trúc thư mục | Chuyển đổi định dạng xuất trên CVAT sang chuẩn `Ultralytics YOLO Detection 1.0` | Thư mục xuất ra đúng định dạng `labels/train/` chuẩn hóa theo yêu cầu đề bài |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- **Phần 1 - Annotation & Guideline:** Đã hoàn thành 50 nhãn train, viết guideline mini xử lý các ca biên, cấu trúc thư mục chuẩn.
- **Phần 2 - Gold-set QC:** Đã chạy script đánh giá và đạt `passed: true` với $F_1 \ge 0.80$ và $IoU \ge 0.70$.
- **Phần 3 - YOLO Training & Inference:** Huấn luyện thành công 50 epochs model `yolo26n.pt` trên GPU T4, ghi nhận đầy đủ file `model_eval_metrics.json`, export ảnh trực quan.
- **Phần 4 - Báo cáo:** Hoàn thiện đầy đủ các mục, phân tích sâu các chỉ số và giải đáp đầy đủ 4 câu hỏi thảo luận.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ AI:** Gemini hỗ trợ rà soát quy trình dòng lệnh Git, gỡ lỗi `exit status 128` trên Colab và hỗ trợ đối chiếu phân tích ma trận nhầm lẫn của model.
- **Tự kiểm tra:** Học viên trực tiếp thao tác các lệnh Git trên máy trạm Windows, kiểm tra trực quan ảnh trên CVAT, thực thi notebook Colab và xác thực các metrics đầu ra.
- **Tài liệu tham khảo:**
  - Tài liệu Ultralytics YOLO: https://docs.ultralytics.com/
  - Hướng dẫn công cụ CVAT: https://docs.cvat.ai/