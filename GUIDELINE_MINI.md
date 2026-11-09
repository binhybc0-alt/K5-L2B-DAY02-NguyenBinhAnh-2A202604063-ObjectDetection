# Guideline Gán Nhãn Mini - ADAS Object Detection

## 1. Bảng định nghĩa 4 lớp bắt buộc
- **0: car**: Ô tô con từ 4 đến 7 chỗ, sedan, hatchback, SUV, xe taxi, và **xe bán tải (pickup truck)**. Không gán cho xe buýt, xe tải lớn, xe máy.
- **1: truck**: Xe tải chở hàng thùng kín hoặc hở, xe tải ben, xe bồn, xe đầu kéo container. Không gán cho xe bán tải/cabin nhỏ thuộc `car`.
- **2: bus**: Xe buýt công cộng nội đô, xe khách liên tỉnh lớn, xe đưa đón học sinh/minibus nhiều chỗ. Không gán cho van chở hàng/chở người loại nhỏ.
- **3: van**: Xe van giao hàng, minivan chở người hoặc hàng hóa dạng thùng hộp liền khối (ví dụ: Ford Transit chở đồ, xe cứu thương, xe blind van). Không gán cho xe buýt cỡ lớn hoặc ô tô con thông thường.

## 2. Quy tắc vẽ Bounding Box
- **Bám sát viền vật thể**: Bounding box phải vẽ sát mép ngoài của thân xe (tight bounding box), không để thừa khoảng trống mặt đường, lề đường hoặc bầu trời để đảm bảo IoU >= 0.70.
- **Không gán nhãn cho xe máy/người**: Chỉ tập trung gán nhãn 4 lớp xe cơ giới nêu trên.

## 3. Quyết định cho 3 ca mơ hồ (Edge cases)
1. **Xe bị che khuất một phần (Occlusion)**:
   - Nếu phần nhìn thấy được chiếm từ **20% - 30% trở lên** và mắt người vẫn nhận diện chính xác được loại xe: Vẽ box bám sát phần thân xe nhìn thấy thực tế (visible boundary).
   - Nếu bị che khuất quá nhiều (< 20%) hoặc không thể phân biệt được lớp: Bỏ qua không gán box.
2. **Xe bị cắt bởi mép khung hình (Truncation)**:
   - Chỉ gán nhãn nếu phần thân lọt vào ảnh đủ đặc trưng để xác định lớp phương tiện.
   - Bounding box được kéo chạm sát mép viền ảnh (tọa độ giới hạn ở mép 0 hoặc 1), tuyệt đối không đoán vẽ tràn ra ngoài khung ảnh.
3. **Phân biệt xe bán tải (Pickup) và xe van chở khách 16 chỗ**:
   - **Xe bán tải (Pickup truck)**: Luôn luôn gán nhãn là `car` (Class 0), không đưa vào `truck`.
   - **Xe 16 chỗ (Ford Transit, Solati)**: Nếu dùng làm xe tuyến chở khách đông người thì xếp vào `bus` (Class 2); nếu dùng làm xe chở hàng, xe van chuyển phát bưu kiện thì gán `van` (Class 3).