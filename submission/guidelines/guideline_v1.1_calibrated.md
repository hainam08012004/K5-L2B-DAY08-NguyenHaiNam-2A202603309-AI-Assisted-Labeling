# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603309 - Nguyễn Hải Nam

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 và thêm quy tắc mới rút ra từ lỗi của máy / bẫy lọt trong quá trình review.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện: bicycle hay motorbike? | Phân loại theo bàn đạp vật lý: xe có bàn đạp trợ lực tính là `bicycle`; xe máy điện kiểu scooter, không bàn đạp tính là `motorbike`. | `img_011.jpg` | CAL-02 |
| 3 | Hàng hoá chở trên xe nhô ra khỏi xe ba bánh/xe đạp | Giỏ hàng, thùng hàng gắn cố định vào xe tính vào box xe; hàng chở nhô ra >20% chiều dài xe thì không vẽ box bao phần hàng nhô ra. | `img_004.jpg` | CAL-04 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Xe tuk-tuk / ba bánh có mui chở khách | YOLO26 gọi là `car` hoặc `truck` | Luôn phân loại là `three_wheeler`; tách riêng box `person` cho người lái và hành khách nhìn thấy | Bổ sung mục 2: xe ba bánh có mui vẫn thuộc `three_wheeler` (CAL-01) |
| `img_018.jpg` | Bãi đỗ / dòng xe máy ùn tắc dày đặc ở cự ly xa | YOLO26 vẽ sót hoặc box dính chùm | Với cụm ≥ 5 xe máy nằm sát nhau mà chiều cao mỗi xe < 20 px, gom thành một box `ignore_region` | Bổ sung mục 4.1.6: quy chuẩn vùng ignore cho xe máy ở xa (CAL-03) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | Khởi tạo baseline |
| v1.1 | Chốt phân loại xe đạp điện, quy định ranh giới hàng chở nhô ra, làm rõ xe ba bánh có mui và tiêu chuẩn gom cụm ignore_region cho xe máy ở xa | CAL-01, CAL-02, CAL-03, CAL-04 |
