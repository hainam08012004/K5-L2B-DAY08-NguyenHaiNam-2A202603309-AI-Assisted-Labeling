# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 325; đối tượng sau duyệt: 330; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 194 | 194/325 = 59.7% | 325 box máy |
| edited (sửa) | 79 | 79/325 = 24.3% | 325 box máy |
| deleted (xoá) | 52 | 52/325 = 16.0% | 325 box máy |
| added (vẽ thêm) | 57 | 57/330 = 17.3% | 330 đối tượng sau duyệt |

Trong số deleted + added, **7 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 50.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 125 | 31 | 9 | 20 |
| bicycle | 2 | 1 | 2 | 1 |
| motorbike | 37 | 31 | 12 | 11 |
| car | 23 | 4 | 3 | 14 |
| bus | 0 | 1 | 1 | 3 |
| truck | 1 | 1 | 0 | 3 |
| three_wheeler | 6 | 10 | 25 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 44 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 35 | 8 | 9 | 81.4% (KTC 95%: 67–90%) | 79.5% (KTC 95%: 65–89%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 9/12 = 75.0% (KTC 95%: 47–91%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - missing: 1/3
  - wrong_class: 3/3
  - loose: 2/3
  - lọt: img_021.jpg (missing), img_010.jpg (loose), img_018.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
