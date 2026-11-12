# Báo cáo thực hành Day 3 — Hoàng Thuận Thiên

- Mã học viên theo lớp: `2A202603270` — Hoàng Thuận Thiên — Lớp K5-L2
- Ngày / CVAT local: 11–12/11/2026 / CVAT local
- Công cụ đã dùng: CVAT local, `scripts/inspect_submissions.py`, `scoring/score.py`, `scoring/scorecard.py`

## 1. Bài đã nộp

Bài làm tập trung hoàn thành ba cấp độ thuộc nhóm `tiers`: `easy_semantic`, `medium_instance`, `hard_panoptic`. Sáu trạm `checkpoints` chưa thực hiện nên không tạo ZIP rỗng và được ghi nhận là `chưa có`.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `submissions/easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `submissions/medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `submissions/hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | chưa có | 0 / 1 | 3 |
| cp2_slice | chưa có | 0 / 1 | 3 |
| cp5_occlusion | chưa có | 0 / 1 | 3 |
| cp3_thin | chưa có | 0 / 1 | 3 |
| cp4_curb | chưa có | 0 / 1 | 3 |
| cp6_coverage | chưa có | 0 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: Ảnh `000000181542.jpg`, người đi bộ bên rìa phải ở vị trí `x = 612..633, y = 124..166`, là object đầu tiên với ID 1.
- Class và quy tắc tôi dùng để chọn biên: Tôi chọn lớp `person`, vẽ mặt nạ bám sát phần thân người nhìn thấy được, dừng đường biên trước cạnh cột biển báo phía trước và không tràn sang phần nền tường phía sau.
- Nếu dùng gợi ý sau đó: Tôi tự vẽ thủ công trên CVAT bằng công cụ mask mà không dùng SAM. Khi các xe đỗ nối đuôi nhau, ban đầu tôi xếp thứ tự lớp trên–dưới trong CVAT, sau đó phát hiện xuất `COCO 1.0` làm mất thứ tự lớp nên tôi đã chỉnh lại phần ranh giới cho các mask khớp sát phần nhìn thấy hơn.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: Lớp `sidewalk` trên hai ảnh `817bca71-00000000.jpg`, `81ae7cbb-6bc63a4a.jpg` và vùng `vegetation` trên ảnh `7ee6d192-89e2408b.jpg` thuộc `easy_semantic`, cùng với lỗi bỏ sót các đối tượng nhỏ và vùng che khuất ở `medium_instance` và ảnh `000000350023.jpg` thuộc `hard_panoptic`.
- Lỗi thuộc loại: thiếu-thừa vật, phủ vùng và biên.
- Bằng chứng tôi nhìn thấy: Ở bản xuất ban đầu, `easy_semantic` đạt `mIoU = 0.672` (`12.1 / 20`) do `sidewalk IoU = 0.000`; `medium_instance` đạt `0.483` (`5.9 / 32`, `TP = 46`, `R@0.5 = 0.65`) do các mask xe đỗ nối đuôi trên `000000458325.jpg` bị xuất chồng lấn xuống phần che khuất làm giảm IoU; `hard_panoptic` đạt `PQ = 0.335` (`9.0 / 30`) do thiếu một số người đi bộ, xe buýt góc phải và gộp đèn giao thông trên `000000350023.jpg`. Ngoài ra ở lần export đầu tiên tôi cũng từng chọn nhầm định dạng `YOLO 1.1` gây rỗng file trước khi chuyển sang `Segmentation mask 1.1` và `COCO 1.0`.
- Quy tắc và hành động sửa: Tôi mở lại từng task trong CVAT để tô bổ sung hai dải `sidewalk` và vùng cây bụi còn thiếu ở `easy_semantic`; cắt gọn biên và rà lại class ở `medium_instance`; tách riêng các `traffic light` và bổ sung `person`, `bus` bị sót trên ảnh `000000350023.jpg` của `hard_panoptic`. Với ảnh `000000460147.jpg` của `hard_panoptic`, tôi giữ nguyên phần gán đúng theo thực tế và đưa vào mục phản biện bên dưới.
- Sau sửa đã Save và export lại chưa? Đã Save trên CVAT, xuất lại đúng định dạng (`80` mask RLE ở `medium_instance`, `87` mask RLE ở `hard_panoptic`) và kiểm tra bằng `python3 scripts/inspect_submissions.py --dir submissions` đạt `[OK]` cho cả 3 task thuộc nhóm `tiers`.

Kết quả tự chạy `scoring/scorecard.py --group tiers` (`reports/tiers/SCORECARD.md`):
- Trước khi sửa mask: **27.0 / 82** (`easy_semantic`: `0.672` → `12.1 / 20`; `medium_instance`: `0.483` → `5.9 / 32`; `hard_panoptic`: `0.335` → `9.0 / 30`).
- Sau khi sửa và xuất lại: **65.9 / 82**, không có review signal bất thường:
  - `easy_semantic`: `mIoU = 0.929`, `coverage = 99.8%` (`sidewalk IoU = 1.000`, `road IoU = 0.988`, `sky IoU = 0.969`, `vegetation IoU = 0.894`, `building IoU = 0.793`) → **20.0 / 20**.
  - `medium_instance`: `metric = 0.629` (`mean matched IoU = 0.770`, `P@0.5 = 0.72`, `R@0.5 = 0.82`, `TP = 58 / FP = 22 / FN = 13`) → **16.3 / 32**.
  - `hard_panoptic`: `PQ = 0.643` (`SQ = 0.855`, `RQ = 0.764`) → **29.6 / 30**.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. Ảnh `000000460147.jpg` (`hard_panoptic`), mặt đường 4 làn bên trái dải phân cách, xe tải chở ô tô và xe đen bên lề trái | Đường có vạch kẻ làn và xe ô tô đang chạy: gán là `road`, phần bên ngoài hàng rào mới là `sidewalk`; hay gán luôn lòng đường thành `sidewalk` | Tôi gán 2 làn đường nhựa có vạch kẻ và xe chạy là `road`, phần vỉa hè ngoài hàng rào là `sidewalk`; gán đủ khung xe tải chở ô tô và xe đỗ lề trái. Khi đối chiếu reference thì đáp án lại tô toàn bộ 2 làn đường bên trái thành `sidewalk`, làm `sidewalk` và `road` trên ảnh này không khớp (`TP = 0`), đồng thời bỏ sót khung xe tải chở ô tô và xe đỗ bên trái | Tôi phản biện ca này và đề nghị coach kiểm tra lại ground truth trên ảnh `000000460147.jpg` vì ranh `road` và `sidewalk` của reference không khớp chức năng làn đường thực tế |
| 2. Ảnh `000000458325.jpg` (`medium_instance`), hàng xe ô tô đỗ nối đuôi nhau bên lề phải (`x = 350..590`) | Vẽ đủ dáng xe rồi xếp thứ tự layer trên–dưới trong CVAT, hay phải tự cắt bỏ phần bị che ngay lúc vẽ trước khi xuất `COCO 1.0` | Khi tôi dùng cách xếp thứ tự trên–dưới trong CVAT, file `COCO 1.0` xuất ra không giữ thứ tự layer nên diện tích mask bị tính luôn phần nằm chìm phía dưới, làm IoU bị kéo giảm dù vùng giao khớp rất cao | Nhờ coach xác nhận liệu khi chấm có xem xét trường hợp mất thứ tự layer khi xuất `COCO 1.0` từ CVAT hay luôn bắt buộc người gán phải cắt sát từng lát cắt nhìn thấy |
| 3. Ảnh `000000181542.jpg` (`medium_instance`), chiếc xe bán tải màu bạc ở làn giữa (`x = 270..520`) | Gán là `car` như xe cá nhân 4 chỗ hay gán là `truck` vì có thùng chở hàng phía sau | Ban đầu tôi gán là `car`, sau khi đối chiếu quy tắc và dữ liệu COCO thì xe bán tải có thùng hàng tách biệt được tính là `truck`, nên tôi đã đổi lại thành `truck` ở bản xuất mới | Nhờ coach thống nhất quy ước phân loại xe bán tải giữa `car` và `truck` xuyên suốt các buổi thực hành |
