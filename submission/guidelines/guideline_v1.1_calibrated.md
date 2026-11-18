# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603044 - Đinh Thành Vinh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Phân loại theo bàn đạp và sàn để chân: xe có bàn đạp rõ ràng gán `bicycle`; xe điện dáng xe ga có sàn để chân và không có bàn đạp gán `motorbike`. | `img_035.jpg` | `CAL-04` |
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Nếu các xe đỗ chồng chéo che khuất nhau > 50% diện tích thân xe và không xác định được ranh giới từng chiếc thì gom thành 1 khung `ignore_region`. | `img_018.jpg` | `CAL-02` |
| 3 | Hàng nhô ra trên xe đạp / ba gác: box xe và mask có gồm hàng không? | Box xe chỉ ôm khung xe và giỏ gắn cố định; hàng hóa rời nhô ra quá 20% chiều dài xe không tính vào box và mask xe. | `img_009.jpg` | `CAL-03` |
| 4 | Người ngồi trong xe ba bánh có mui che: ngưỡng "còn thấy ≥ 20%"? | Giữ ngưỡng ≥ 20% diện tích cơ thể (tương đương nhìn thấy rõ đầu và phần vai/ngực); chỉ thấy tay hoặc bóng mờ dưới 20% thì không gán `person`. | `img_003.jpg` | `CAL-01` |
| 5 | Kính chắn gió / cửa sổ xe: có thuộc mask xe không? | Kính chắn gió và cửa sổ trong suốt thuộc phần thân vỏ của mask xe; chỉ tách riêng khi có người ngồi lộ hẳn ra ngoài khung cửa sổ. | `img_013.jpg` | `CAL-01` |
| 6 | Box máy chồng nhau quá nhiều trong ảnh đêm / phố đông: gộp hay giữ? | Giữ box riêng cho từng đối tượng tách biệt (IoU < 0.6); xóa các box trùng lặp cùng lớp đè lên nhau trên cùng một vật thể. | `img_003.jpg` | `CAL-01` |
| 7 | Mức tin cậy tối thiểu để chấp nhận box máy không cần sửa? | Không tự động chấp nhận theo điểm `conf` thuần túy; vẫn phải nhìn mắt kiểm tra độ ôm sát cạnh box, nhưng ưu tiên soi kỹ các box có `conf < 0.45`. | `img_009.jpg` | `CAL-03` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_003.jpg` | Xe ba gác / tuk-tuk có thùng hàng hoặc mui che lớn | `YOLO26m` nhận nhầm thành `car` hoặc `truck` | Gán `three_wheeler` dựa theo cấu trúc 3 bánh và dùng `YOLOE` ngưỡng `0.15` | Mục 2 & 4.3: Mọi xe 3 bánh chở khách/hàng đều gán `three_wheeler`, không gán `truck`/`car`. |
| `img_018.jpg` | Hàng xe máy đỗ san sát nhau gần vỉa hè chỉ lộ đầu xe/tay lái | `YOLO26m` vẽ nhiều box nhỏ chồng chéo từng phần đầu xe | Xóa các box lẻ bị che > 50% và vẽ 1 khung `ignore_region` bao cụm | Mục 4.1.6: Cụm xe đỗ dày đặc che khuất > 50% thân xe không tách được từng chiếc thì dùng `ignore_region`. |
| `img_009.jpg` | Xe máy chở hàng cồng kềnh và bạt che dễ nhầm thành người/xe ba bánh | `YOLO26m` khoanh nhầm phần yên/bạt xe máy ở góc trái thành `person` | Xóa box `person` ảo; giữ box `motorbike` ôm thân xe chính, loại bỏ phần hàng nhô > 20% | Mục 4.1.5: Không gán `person` cho đống hàng/vải phủ; box xe loại bỏ phần hàng rời nhô quá 20%. |
| `img_035.jpg` | Xe đạp điện và xe máy điện lưu thông lẫn trên phố | `YOLO26m` gán lẫn lộn giữa `bicycle` và `motorbike` | Căn cứ vào bàn đạp (có bàn đạp → `bicycle`; dáng xe ga sàn phẳng → `motorbike`) | Mục 2 (Bảng lớp): Bổ sung tiêu chí bàn đạp và sàn để chân để phân biệt xe điện. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt 7 câu hỏi mở (phân biệt xe điện, cụm xe đỗ dày đặc, hàng nhô > 20%, ngưỡng che khuất 20%, kính xe, trùng box, ngưỡng duyệt `conf`) và bổ sung quy tắc phân loại `three_wheeler`. | Rút ra từ lỗi nhận nhầm lớp của YOLO26m, lỗi che khuất và các ca phân vân khi duyệt (`CAL-01`, `CAL-02`, `CAL-03`, `CAL-04`). |
