# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 519; đối tượng sau duyệt: 471; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 406 | 406/519 = 78.2% | 519 box máy |
| edited (sửa) | 28 | 28/519 = 5.4% | 519 box máy |
| deleted (xoá) | 81 | 81/519 = 15.6% | 519 box máy |
| added (vẽ thêm) | 36 | 36/471 = 7.6% | 471 đối tượng sau duyệt |

Trong số deleted + added, **11 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 25.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 236 | 8 | 23 | 7 |
| bicycle | 4 | 1 | 2 | 3 |
| motorbike | 105 | 12 | 18 | 12 |
| car | 47 | 1 | 6 | 4 |
| bus | 3 | 0 | 0 | 2 |
| truck | 2 | 0 | 4 | 3 |
| three_wheeler | 9 | 6 | 28 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 70 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 51 | 20 | 19 | 71.8% (KTC 95%: 60–81%) | 72.9% (KTC 95%: 61–82%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - loose: 3/3
  - wrong_class: 2/3
  - missing: 3/3
  - lọt: img_009.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
