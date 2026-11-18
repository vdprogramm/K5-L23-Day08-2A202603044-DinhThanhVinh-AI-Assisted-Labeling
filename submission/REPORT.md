# Báo cáo Day 08 — 2A202603044 - Đinh Thành Vinh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | 0.30 (mặc định) | Các lớp `person`, `bicycle`, `motorbike`, `car` không đạt recall 90% ở mức ≤ 0.5 trên tập gold; `bus` (1 vật) và `truck` (0 vật) có < 5 mẫu gold nên giữ ngưỡng mặc định `0.30` (`threshold_sweep.md`). |
| three_wheeler | YOLOE (prompt: `tuk-tuk`, `cyclo`, `cargo tricycle`) | 0.15 | Đạt recall 100% (7/7 vật gold ≥ 90%) và precision 78% trên tập gold (`threshold_sweep.md`), khắc phục việc YOLO26 không có lớp xe ba bánh. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1 (`sam2.1_b.pt`)** vì cân bằng tốt giữa độ chính xác đường biên mask và tốc độ/RAM khi chạy trên máy tính dùng CPU (không có GPU rời).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi nhận nhầm phân lớp (như YOLO26 gọi tuk-tuk/xe ba gác là `car` hoặc `truck` vì bộ lớp COCO gốc không có `three_wheeler`) không thể sửa bằng cách tăng/giảm ngưỡng `conf`. Cách sửa là kết hợp thêm model open-vocabulary **YOLOE** với từ khóa mô tả xe 3 bánh (`tuk-tuk`, `cyclo`, `cargo tricycle`) và duyệt sửa thủ công trên CVAT.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Trên `img_009.jpg`, box `person` khoanh nhầm vào đống hàng/yên xe máy dưới mái bạt bên trái có điểm tin cậy khá cao nhưng sai hoàn toàn đối tượng; ngược lại một số xe ba gác ở `img_003.jpg` có điểm tin cậy thấp (~0.18–0.25) nhưng nhận diện đúng lớp `three_wheeler`.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Vì chọn ngưỡng và đánh giá cùng trên 4 ảnh `gold` (in-sample) với số lượng mẫu ít (70 đối tượng), các chỉ số precision/recall trên gold mang tính lạc quan và có khoảng tin cậy 95% rộng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 406 | 78.2% | 519 box máy |
| edited | 28 | 5.4% | 519 box máy |
| deleted | 81 | 15.6% | 519 box máy |
| added | 36 | 7.6% | 471 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? **`three_wheeler`** là lớp máy tệ nhất (chỉ giữ nguyên 9 box, phải xóa tới 28 box thừa/sai và sửa 6 box, ví dụ ở `img_003.jpg`), do hạ ngưỡng `0.15` giúp tăng độ phủ nhưng kéo theo nhiều box trùng lặp và nhận nhầm xe máy chở hàng cồng kềnh. Tiếp theo là **`motorbike`** (12 box phải sửa, 18 box xóa, 12 box vẽ bù) do xe máy đỗ san sát và bị người điều khiển che khuất một phần.
- So với gold: precision / recall của **máy** `71.8% (51/71, KTC 95%: 60–81%)` / `72.9% (51/70, KTC 95%: 61–82%)` (mẫu số: `70 đối tượng gold`).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: `11/12 (91.7%)`; loại nào lọt: `wrong_class` ở `img_009.jpg` (1/3 bẫy `wrong_class` bị lọt; các loại `extra` 3/3, `loose` 3/3, `missing` 3/3 đều bắt đủ); vì sao (bạn đã bỏ bước nào của checklist)? Ở lượt duyệt đầu tiên, do chưa ẩn nhãn máy trước khi nhìn ảnh nên còn lọt bẫy; sau khi rà soát lại kỹ từng ảnh theo checklist 4 bước thì bắt được 11/12 bẫy, chỉ còn sót 1 trường hợp `wrong_class` nhỏ ở vùng rìa tối của `img_009.jpg`.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Nếu `accepted` cao (~80%) mà bắt bẫy thấp chứng tỏ người duyệt mắc thiên kiến tự động hóa, chỉ bấm chấp nhận theo quán tính mà không kiểm tra kỹ ảnh gốc. Trong bài này, tỉ lệ `accepted` là 78.2% đi kèm tỉ lệ bắt bẫy đạt 91.7% (11/12) cho thấy quá trình rà soát đã phát hiện và loại bỏ được phần lớn lỗi cài sẵn.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay `5.0 phút/ảnh` (20 phút / 4 ảnh gold) vs duyệt `1.88 phút/ảnh` (45 phút / 24 ảnh batch). Thiên lệch (bạn gán gold trước nên đã quen cảnh): Việc gán tay 4 ảnh gold trước giúp làm quen giao diện CVAT và quy tắc lớp, đồng thời 24 ảnh batch có mật độ xe đông hơn nhưng nhờ có pre-label sẵn nên tốc độ duyệt vẫn nhanh gấp ~2.6 lần so với vẽ tay từ đầu.
- Kết luận: **Có, rất đáng dùng** cho tất cả các lớp, dựa trên tỉ lệ `accepted` đạt `78.2% (406/519)` và thời gian hoàn thành giảm từ `5.0 phút/ảnh` xuống còn `1.88 phút/ảnh` mà vẫn đảm bảo chất lượng sau duyệt.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Ẩn nhãn máy trước khi xem ảnh (phím `H`):** Dành 5–10 giây quét mắt toàn bộ người và xe trên ảnh gốc trước khi bật hiển thị box AI (áp dụng cho 24/24 ảnh).
  2. **Kiểm tra chéo riêng các phương tiện đặc thù (`three_wheeler` và cặp `person` + `motorbike`):** Rà soát 100% các xe 3 bánh và đảm bảo mỗi người ngồi trên xe đều có box `person` tách biệt với box xe.
  3. **Chia nhỏ phiên duyệt (nghỉ 3–5 phút sau mỗi 10 ảnh):** Tránh mỏi mắt gây bỏ sót các bẫy khung lỏng (`loose`) hoặc khung ảo (`extra`) ở rìa ảnh.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | `sam2.1_b.pt` | 3 (2 pos, 1 neg) | Có dính một phần vùng nền/người xung quanh do điểm pos rộng (chiếm 45.4% ảnh) | Vùng mask bị tràn rộng ra ngoài biên xe tuk-tuk | `tuktuk_img013.jpg` |
| `img_003.jpg` | `sam2.1_b.pt` | 3 (2 pos, 1 neg) | Tách được phần lớn người ngồi trên xe nhờ điểm neg phía trên | Mask ôm gọn phần thân và mái xe ba gác (chiếm 11.6% ảnh), hơi khuyết nhẹ ở bánh xe | `tuktuk_img003.jpg` |
| `img_035.jpg` | `sam2.1_b.pt` | 2 (1 pos, 1 neg) | Tách riêng được người đạp xích lô ngồi phía sau | Mask bám sát thân xe xích lô (chiếm 7.8% ảnh), mất một phần nan hoa bánh xe mảnh | `cyclo_img035.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc 4.2.1 (bám đường biên phần nhìn thấy, không cần chi tiết nhỏ hơn bánh xe/tay lái) và 4.2.4 (hai vật chạm nhau không chồng lên nhau) giúp xác định ranh giới khi đặt điểm `neg` tách người lái khỏi xe. Tuy nhiên, guideline còn thiếu hướng dẫn cụ thể về số lượng điểm `pos`/`neg` tối thiểu khi xe có khung mái hở nhìn xuyên qua nền đường.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Chưa sử dụng trực tiếp trên CVAT (chạy offline bằng script `sam2_points.py`); cách chạy offline giúp tái lập chính xác tọa độ click nhưng khó tinh chỉnh trực quan từng điểm như khi click trực tiếp trên giao diện CVAT.

## 6. Calibration (Part 6)

- Số quyết định: `4` (`submission/qc/calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-01`): Quy định bắt buộc phân loại mọi phương tiện 3 bánh (tuk-tuk, xích lô, xe ba gác chở hàng) vào lớp `three_wheeler` và kết hợp `YOLOE` ngưỡng `0.15` để khắc phục lỗi `YOLO26m` gọi nhầm thành `car`/`truck`.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Chốt toàn bộ 7 câu hỏi mở (tiêu chí bàn đạp/sàn để chân phân biệt xe điện `CAL-04`, gom `ignore_region` khi cụm xe che khuất > 50% `CAL-02`, loại bỏ hàng hóa rời nhô > 20% chiều dài xe `CAL-03`, giữ ngưỡng che khuất 20%, gộp kính chắn gió vào mask xe, xóa box trùng lặp và rà kỹ `conf < 0.45`).
- Nguyên nhân gốc: Kết hợp cả hai yếu tố **guideline v1.0 còn để mở một số tình huống thực tế đường phố châu Á** và **giới hạn danh mục của model pretrained (YOLO26)**.

## 7. QC và quyết định (Part 6)

- Mẫu: `5` ảnh (`0` ảnh `can_review` + `5` ngẫu nhiên: `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`, seed 42); người kiểm: Đinh Thành Vinh (**hạn chế**: người kiểm QC cũng chính là người trực tiếp duyệt nhãn ở Part 4 nên có thể lặp lại cùng thiên kiến chủ quan).
- Lỗi theo **đối tượng**: `0/93 = 0.0%` · theo **ảnh**: `0/5 = 0.0%` (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: Không ghi nhận lỗi nào trên 5 ảnh mẫu QC (`0/5` ảnh lỗi); trên toàn lô duyệt trước đó lỗi phổ biến nhất là sai lớp ở `img_009.jpg` (`wrong_class`).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 0 ảnh lỗi trên mẫu nhỏ 5 ảnh, cận trên khoảng tin cậy 95% (Wilson) của tỉ lệ ảnh lỗi thực tế lên tới **43%**, do đó mẫu 5 ảnh chưa đủ lực thống kê để chứng minh tuyệt đối ngưỡng ≤ 2%.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): Script kiểm tra tự động (`validate_labels.py`, `qc_report.py`) kiểm được tính hợp lệ định dạng COCO, kích thước box tối thiểu `< 20px`, phân bố lớp và thống kê phiếu QC; nhưng **không kiểm được** độ ôm sát thực tế của box quanh vật thể, việc người duyệt có tuân thủ bước ẩn nhãn máy (`H`) hay không, hoặc trường hợp cả người và máy cùng bỏ sót một đối tượng.

**Quyết định: Release**, ngày `2026-11-18`. Lý do (đi theo Hold → Rework → Release): Không vướng Hold (guideline v1.1 đã chốt rõ mọi ca mở, dữ liệu hợp lệ); không phải Rework (tỉ lệ ảnh lỗi mẫu QC là `0.0% ≤ 2%`, không có lỗi hệ thống lặp lại và tỉ lệ bắt bẫy đạt `11/12 = 91.7% ≥ 80%`); do đó đủ điều kiện **Release** lô dữ liệu `submission/release/`.
Nếu Rework / Hold, việc cần làm tiếp: Không áp dụng vì đã đạt điều kiện Release.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Sử dụng AI trợ lý (Gemini) để giải thích cấu trúc lệnh Python, rà soát lỗi quy trình trên CVAT và tổng hợp cấu trúc báo cáo; mọi con số thống kê đều được đối chiếu trực tiếp từ kết quả chạy script thực tế (`review_stats.md`, `threshold_sweep.md`, `qc_report_auto.md`).
