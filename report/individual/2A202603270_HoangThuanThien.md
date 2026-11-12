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

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: Ảnh `000000181542.jpg`, người đi bộ bên rìa phải ở vị trí khoảng `x = 612..633, y = 124..166`, là object đầu tiên với ID 1.
- Class và quy tắc tôi dùng để chọn biên: Tôi chọn lớp `person`, vẽ mặt nạ bám sát phần thân người nhìn thấy được, dừng đường biên trước cạnh cột biển báo phía trước và không tràn sang phần nền tường phía sau.
- Nếu dùng gợi ý sau đó: Tôi không sử dụng gợi ý tự động SAM mà vẽ thủ công trực tiếp trên CVAT cho toàn bộ các object. Khi các phương tiện đỗ sát nhau hoặc nối đuôi nhau, tôi sắp xếp thứ tự lớp trên–dưới trong CVAT để xe phía trước che lên xe phía sau trên khung hình.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: Toàn bộ `easy_semantic` và `medium_instance` ở lần xuất file đầu tiên từ CVAT.
- Lỗi thuộc loại: khác — lỗi định dạng khi export từ CVAT.
- Bằng chứng tôi nhìn thấy: Lần xuất đầu tiên cho job 4 và job 5 được chọn nhầm sang định dạng `YOLO 1.1`, làm cho tất cả file nhãn `.txt` bên trong đều có kích thước 0 byte và không chứa mặt nạ segmentation.
- Quy tắc và hành động sửa: Tôi mở lại các job đã lưu trên CVAT và xuất lại đúng theo định dạng quy định là `Segmentation mask 1.1` cho `easy_semantic` và `COCO 1.0` cho `medium_instance`, `hard_panoptic`, sau đó đổi tên file vào thư mục `submissions/`.
- Sau sửa đã Save và export lại chưa? Đã Save trên CVAT, xuất lại đúng định dạng và kiểm tra bằng `python3 scripts/inspect_submissions.py --dir submissions` đạt trạng thái `[OK]` cho cả 3 task thuộc nhóm tiers (`73` mask RLE ở Medium và `75` mask RLE ở Hard).

Kết quả tự chạy script đánh giá nhóm `tiers` tại máy cá nhân (`reports/tiers/SCORECARD.md`): **27.0 / 82** (`easy_semantic`: `mIoU = 0.672`, `12.1 / 20`; `medium_instance`: `metric = 0.483` với `mean matched IoU = 0.746` và `R@0.5 = 0.65`, `5.9 / 32`; `hard_panoptic`: `PQ = 0.335` với `SQ = 0.446` và `RQ = 0.447`, `9.0 / 30`).

Qua đối chiếu trực quan giữa bài xuất ra và reference, tôi ghi nhận các điểm thực tế chưa sửa lại trên CVAT trước hạn nộp cũng như các điểm cần phản biện với coach:

1. **Hạn chế định dạng `COCO 1.0` khi CVAT export ở `medium_instance`:** Khi vẽ trên CVAT, tôi đã thiết lập thứ tự lớp trước–sau để xe phía trước nằm trên và che phần khuất của xe phía sau. Tuy nhiên, định dạng `COCO 1.0` khi xuất ra không giữ thứ tự lớp này và không tự cắt phần mặt nạ nằm chìm bên dưới. Hệ quả trên ảnh `000000458325.jpg`, phần giao mask của tôi khớp vùng nhìn thấy của reference tới `93%–99.8%`, nhưng tổng diện tích mask bị tính gần gấp đôi làm `IoU` rơi xuống `0.42–0.48`, ngay dưới ngưỡng `0.50`. Theo [`GUIDE.md`](../../GUIDE.md) và [`RUBRIC.md`](../../RUBRIC.md), tôi giữ nguyên file xuất từ CVAT thay vì sửa tay JSON bên trong ZIP và trình bày rõ vấn đề này để coach xem xét lại trực tiếp file làm bài.
2. **Sai sót còn tồn tại trong bài làm:** Ở `easy_semantic`, hai dải `sidewalk` trên ảnh `817bca71-00000000.jpg` và `81ae7cbb-6bc63a4a.jpg` bị bỏ trống dẫn đến `sidewalk IoU = 0.000`; ở `7ee6d192-89e2408b.jpg`, phần sườn đồi đất khô chưa được phủ hết vào `vegetation`. Ở `hard_panoptic` ảnh `000000350023.jpg`, xe buýt ở góc dưới phải và một số người đi bộ nhỏ bị bỏ sót, ba đèn giao thông bị gộp chung thành một mask.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. Ảnh `000000460147.jpg` (`hard_panoptic`), mặt đường 4 làn bên trái dải phân cách, xe tải chở ô tô và xe đen bên lề trái | Đường có vạch kẻ làn và xe ô tô đang chạy: gán là `road`, còn phần bên ngoài hàng rào mới là `sidewalk`; hay gán luôn lòng đường thành `sidewalk` | Tôi gán 2 làn đường nhựa có vạch kẻ và xe chạy là `road`, còn phần vỉa hè sau hàng rào là `sidewalk`; đồng thời gán cả khung xe tải chở ô tô và chiếc xe đỗ bên lề trái. Khi đối chiếu reference thì chính bộ đáp án lại tô toàn bộ lòng đường 4 làn bên trái thành `sidewalk`, làm `sidewalk PQ = 0.000`, đồng thời bỏ trống khung xe tải chở ô tô và xe đỗ bên trái | Tôi phản biện ca này và đề nghị coach kiểm tra lại ground truth trên ảnh `000000460147.jpg` vì ranh `road` và `sidewalk` của reference không khớp với chức năng đường giao thông thực tế |
| 2. Ảnh `000000458325.jpg` (`medium_instance`), hàng xe ô tô đỗ nối đuôi nhau bên lề phải (`x = 350..590`) | Vẽ đủ dáng xe rồi xếp thứ tự layer trên–dưới trong CVAT, hay tự cắt bỏ hoàn toàn phần bị che trước khi xuất `COCO 1.0` | Trong bài làm tôi dùng cách xếp layer trên–dưới trong CVAT; nhưng lúc xuất `COCO 1.0`, các mask vẫn bị tính nguyên diện tích nằm chìm phía dưới. Nếu mô phỏng cắt đúng theo thứ tự lớp tôi đã xếp thì `medium_instance` tăng từ `46` TP lên `53` TP và từ `5.9 / 32` lên `13.6 / 32` | Nhờ coach xác nhận khi chấm bài có xem xét trường hợp mất thứ tự layer do định dạng `COCO 1.0` của CVAT hay bắt buộc phải cắt sát phần nhìn thấy khi vẽ từ đầu |
| 3. Ảnh `000000181542.jpg` (`medium_instance`), chiếc xe bán tải màu bạc ở làn giữa (`x = 270..520`) | Gán là `car` như các xe cá nhân thông thường hay gán là `truck` vì có thùng hàng phía sau | Tôi gán là `car`, còn reference COCO gán là `truck` với IoU vùng khớp đạt `0.533` | Nhờ coach thống nhất quy ước phân loại xe bán tải pickup giữa `car` và `truck` cho các bài sau |
