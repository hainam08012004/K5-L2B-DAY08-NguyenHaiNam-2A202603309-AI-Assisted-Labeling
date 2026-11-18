# Báo cáo Day 08 — 2A202603309 - Nguyễn Hải Nam

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | yolo26-n | car=0.45, các lớp khác mặc định 0.3 | r01, r02; trên tập pilot yolo26-n conf 0.25 bắt rất nhạy (148 box) nhưng có box thừa, conf 0.5 sạch nhưng mất recall (81 box). Ngưỡng 0.45 cho car giúp precision đạt 67% mà recall vẫn 100% trên gold. |
| three_wheeler | yoloe-s, prompt: tuk-tuk, cyclo, cargo tricycle | 0.15 | r03; YOLOE phát hiện chính xác xe ba bánh địa phương mà YOLO26 bỏ sót. Ngưỡng 0.15 cho recall 100% và precision 78% trên gold. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "car=0.45,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn SAM 2.1 Tiny (`sam2.1_t.pt`) làm phiên bản chính và đối chiếu với SAM 1 Mobile (`mobile_sam.pt`). Lý do: Dung lượng nhẹ (~78 MB), tốc độ suy luận nhanh trên CPU (~2-5s/ảnh), chất lượng đường viền polygon sắc nét vượt trội so với Mobile SAM, không yêu cầu cấp quyền HuggingFace phức tạp như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi nhận nhầm phương tiện địa phương (tuk-tuk, xe ba gác bị gán thành car/truck) do YOLO26 là closed-set model (COCO 80 lớp). Không thể sửa bằng cách tăng giảm ngưỡng. Cách khắc phục: Bổ sung model Open-Vocabulary (YOLOE-26s) với prompt chuyên biệt cho `three_wheeler`, sau đó dùng NMS gộp kết quả và ưu tiên nhãn của YOLOE.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  * Điểm cao mà sai: `img_013.jpg`, xe tuk-tuk bị YOLO26 nhận là `car` với độ tin cậy conf = 0.82.
  * Điểm thấp mà đúng: `img_002.jpg`, xe ba gác chở hàng được YOLOE nhận diện chính xác `three_wheeler` ở mức conf = 0.18.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Việc sweep ngưỡng và đánh giá trên cùng tập 4 ảnh gold (in-sample) dẫn đến hiện tượng quá lạc quan về precision/recall. Cỡ mẫu 4 ảnh quá nhỏ, không bao phủ hết các điều kiện thời tiết, ánh sáng và mật độ phương tiện của toàn bộ tập dữ liệu.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 194 | 194/325 = 59.7% | 325 box máy |
| edited | 79 | 79/325 = 24.3% | 325 box máy |
| deleted | 52 | 52/325 = 16.0% | 325 box máy |
| added | 57 | 57/330 = 17.3% | 330 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` có tỉ lệ bị xóa nhiều nhất (25 box deleted, chỉ 6 accepted) do YOLOE tạo ra một số box lỏng hoặc phát hiện trùng lặp. Tiếp theo là `motorbike` với 31 box phải sửa vị trí/kích thước và 12 box bị xóa, nguyên nhân do xe máy đi san sát nhau trong dòng phương tiện đông đúc khiến box bị dính chùm (ví dụ `img_018.jpg`, `img_024.jpg`).
- So với gold: precision / recall của **máy** Precision 81.4% (35/43), Recall 79.5% (35/44) (mẫu số: 44 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 9/12 = 75.0% (KTC 95%: 47–91%); loại nào lọt: 3 bẫy lọt gồm `img_021.jpg` (missing), `img_010.jpg` (loose), `img_018.jpg` (missing); vì sao: Hai ảnh `img_021.jpg` và `img_018.jpg` có mật độ xe máy cực kỳ đông đúc, reviewer bị quá tải thông tin thị giác và bỏ qua bước quét mắt độc lập (Bước 0 trong checklist), dẫn đến việc chỉ tập trung sửa các box máy gợi ý mà bỏ sót vật máy không vẽ; ảnh `img_010.jpg` có box lỏng ở vùng bóng tối của bánh xe khó quan sát.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỉ lệ accepted đạt 59.7% cùng tỉ lệ bắt bẫy 75.0% cho thấy reviewer có xu hướng bị cuốn vào các gợi ý sẵn có của AI (Automation Bias / Thiên kiến tự động hóa). Người duyệt có tâm lý ỷ lại rằng AI đã bao quát được hầu hết vật thể, dẫn đến lơ là việc phát hiện các lỗi bỏ sót (missing objects).

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 8.75 phút/ảnh (35 phút / 4 ảnh gold) vs duyệt 2.92 phút/ảnh (70 phút / 24 ảnh batch). Thiên lệch: Có sự thiên lệch nhẹ do sau khi gán 4 ảnh gold, người duyệt đã làm quen với bối cảnh đường phố và các quy tắc gán nhãn, tuy nhiên mức tiết kiệm thời gian gần 3 lần (~66%) vẫn phản ánh ưu thế thực tế rất lớn của AI pre-labeling.
- Kết luận: RẤT ĐÁNG DÙNG, dựa trên số liệu thực tế: giảm 66.6% thời gian gán nhãn trên mỗi ảnh (từ 8.75 phút xuống 2.92 phút), giữ nguyên được 59.7% (194/325) số box máy mà không cần vẽ lại từ đầu. AI đặc biệt hiệu quả với các vật thể rõ ràng ở tiền cảnh (`person`, `car`, `motorbike`).
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quy tắc Blind-Scan: Luôn tắt lớp hiển thị nhãn máy trước khi xem ảnh mới, dùng mắt quét và nhẩm tính số lượng đối tượng trong 30 giây rồi mới bật hiển thị nhãn AI.
  2. Quy tắc Low-Confidence Review: Dùng bộ lọc trên CVAT lọc riêng toàn bộ các box có thuộc tính `conf < 0.4` để kiểm tra kỹ ranh giới và nhãn trước khi duyệt tổng thể.
  3. Quy tắc Pomodoro Breaks: Nghỉ ngơi mắt 5 phút sau mỗi 8 ảnh duyệt liên tục để tránh mỏi mắt và suy giảm sự tập trung đối với các chi tiết nhỏ.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 Tiny (`sam2.1_t.pt`) | 3 (2 pos, 1 neg) | Không gồm người lái (đã đặt click âm loại bỏ) | Mask bám sát thành xe tuk-tuk, phủ kín mui xe | `tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 Tiny (`sam2.1_t.pt`) | 1 pos + box prompt | Không gồm người đi bộ bên cạnh | Tách chính xác thân xe ba bánh, không bị lẹm vào nền | `three_wheeler_img004.jpg` |
| `img_004.jpg` | SAM 1 Mobile (`mobile_sam.pt`) | 1 pos + box prompt | Không gồm xe | Biên mask quanh chân người hơi răng cưa và thô | `person_img004_mobilesam.jpg` |
| `img_004.jpg` | SAM 2.1 Tiny (`sam2.1_t.pt`) | 1 pos + box prompt | Không gồm xe | Biên mask bám sát đường viền quần áo và nếp gấp | `person_img004_sam2.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc 4.2.4 (hai vật chạm nhau tách riêng polygon, người lái tách riêng xe) rất hữu ích khi đặt điểm click âm để loại bỏ người lái ra khỏi mask xe. Tuy nhiên guideline còn thiếu hướng dẫn về mức độ chi tiết khi tách các nan hoa bánh xe và khoảng trống nhỏ xuyên qua khung xe.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Interactor SAM trực tiếp trên CVAT yêu cầu cấu hình serverless Nuclio; việc chạy offline qua script `sam2_points.py` giúp kiểm soát chính xác tọa độ click, lặp lại được kết quả và không phụ thuộc vào tải của server CVAT.

## 6. Calibration (Part 6)

- Số quyết định: 4 quyết định (`calibration_decisions.csv`); quan trọng nhất: `CAL-01` (khẳng định xe ba bánh có mui chở khách vẫn thuộc `three_wheeler`) và `CAL-03` (quy chuẩn hóa vùng `ignore_region` cho các cụm xe máy ùn tắc ở cự ly xa).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Tiêu chuẩn phân biệt xe hai bánh điện theo bàn đạp cơ học, quy định ranh giới cho hàng chở nhô ra quá 20% chiều dài xe, và ngưỡng gom cụm ignore cho xe máy.
- Nguyên nhân gốc: Sự phối hợp giữa giới hạn của guideline v1.0 (chưa dự liệu hết phương tiện đặc thù Đông Nam Á) và hạn chế của mô hình COCO detector đóng.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: Nguyễn Hải Nam (**hạn chế**: người duyệt tự kiểm tra lại sản phẩm của chính mình, có thể có điểm mù chủ quan).
- Lỗi theo **đối tượng**: 0/65 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%).
- Top lỗi: Không phát hiện lỗi nghiêm trọng trong mẫu 5 ảnh ngẫu nhiên (tất cả 65 box đều đúng lớp và ôm sát vật thể).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Dù mẫu 5 ảnh đạt 0 lỗi (0%), khoảng tin cậy Wilson 95% cho thấy cận trên của tỉ lệ ảnh lỗi thật trong toàn bộ dataset có thể lên tới 43%. Do đó con số 0% trên mẫu nhỏ không đủ chứng minh chất lượng lô đạt chuẩn ≤ 2%.
- Script kiểm được / không kiểm được: Script kiểm tra được định dạng COCO, id nhãn hợp lệ, tọa độ box không vượt khung hình, tỉ lệ accepted/deleted; script không kiểm tra được mức độ ôm sát thực tế so với vật thể thật, không phát hiện được trường hợp cả người và máy cùng bỏ sót đối tượng ngoài đời thực.

**Quyết định: REWORK**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): Căn cứ vào cây quyết định chất lượng, tỉ lệ bắt bẫy đạt 75.0% (9/12 bẫy) < ngưỡng tối thiểu 80%, cho thấy lô duyệt chưa đạt độ tin cậy tuyệt đối về kiểm soát bẫy. Theo nguyên tắc trung thực số liệu, quyết định đưa ra là **REWORK**.
Nếu Rework / Hold, việc cần làm tiếp: Rà soát lại 3 ảnh bị lọt bẫy (`img_010.jpg`, `img_018.jpg`, `img_021.jpg`), bổ sung các box còn thiếu, siết lại quy trình Blind-Scan và xuất bản lại release candidate.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng Antigravity Assistant hỗ trợ thiết lập môi trường, tự động hóa tương tác CVAT API và thực hiện các script thống kê; tự tay thực hiện toàn bộ việc gán nhãn 4 ảnh gold và review 24 ảnh batch trên CVAT.
- Gold có bị nhiễm không: Không. 4 ảnh gold được gán tay từ đầu trên CVAT trước khi chạy bất kỳ mô hình AI nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 10 Pro, Intel Core i7-6820HQ, GPU NVIDIA Quadro M1000M 2GB (chạy chế độ CPU trên PyTorch), CVAT Docker v2.74.1. Môi trường chạy CPU mượt mà với các model bản nano/tiny.
- Dùng cache `data/cache/` thay vì tự chạy? Không dùng cache, tự chạy toàn bộ các model trên máy local.
- Lỗi / bug script đã gặp (nếu có): Không gặp bug lớn; chỉ có cảnh báo FutureWarning deprecation từ PyTorch khi load script torchscript.
- Một điều bạn làm khác lần sau: Sẽ thực hiện bước quét mắt độc lập kỹ lưỡng hơn đối với các ảnh có mật độ đông đúc trên 30 đối tượng để tránh lọt bẫy missing của AI.
