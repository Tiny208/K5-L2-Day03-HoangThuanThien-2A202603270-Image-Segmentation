# Báo cáo thực hành Day 3 — Hoàng Thuận Thiên

- Mã học viên theo lớp: `2A202603270` — Hoàng Thuận Thiên — Lớp K5-L2
- Ngày / CVAT local: 11–12/11/2026 / CVAT local
- Công cụ đã dùng: CVAT local, `scripts/inspect_submissions.py`, `scoring/score.py`, `scoring/scorecard.py`

## 1. Bài đã nộp

Bài làm tập trung hoàn thành đầy đủ ba cấp độ thuộc nhóm `tiers` (`easy_semantic`, `medium_instance`, `hard_panoptic`). Sáu trạm `checkpoints` chưa thực hiện nên không tạo ZIP rỗng và được ghi nhận là `chưa có`.

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
- Nếu dùng gợi ý sau đó: Tôi không sử dụng gợi ý tự động SAM mà vẽ thủ công trực tiếp trên CVAT cho toàn bộ các object; đối với các xe di chuyển gần nhau, tôi đối chiếu lại danh sách Objects để giữ mỗi phương tiện là một instance riêng biệt.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: Toàn bộ `easy_semantic` và `medium_instance` ở lần xuất file đầu tiên từ CVAT.
- Lỗi thuộc loại: khác — lỗi định dạng khi export từ CVAT.
- Bằng chứng tôi nhìn thấy: Lần xuất đầu tiên cho hai job 4 và job 5 được chọn nhầm sang định dạng `YOLO 1.1`, làm cho tất cả file nhãn `.txt` bên trong đều có kích thước 0 byte và không chứa mặt nạ segmentation.
- Quy tắc và hành động sửa: Tôi mở lại các job đã lưu trên CVAT và xuất lại đúng theo định dạng quy định là `Segmentation mask 1.1` cho `easy_semantic` và `COCO 1.0` cho `medium_instance`, `hard_panoptic`, sau đó đổi tên file vào thư mục `submissions/`.
- Sau sửa đã Save và export lại chưa? Đã Save trên CVAT, xuất lại đúng định dạng và kiểm tra bằng `python3 scripts/inspect_submissions.py --dir submissions` đạt trạng thái `[OK]` cho cả 3 task thuộc nhóm tiers (`73` mask RLE ở Medium và `75` mask RLE ở Hard).

Kết quả tự chạy script đánh giá nhóm `tiers` tại máy cá nhân (`reports/tiers/SCORECARD.md`): **27.0 / 82** (`easy_semantic`: `mIoU = 0.672`, `12.1 / 20`; `medium_instance`: `metric = 0.483` với `mean matched IoU = 0.746` và `R@0.5 = 0.65`, `5.9 / 32`; `hard_panoptic`: `PQ = 0.335` với `SQ = 0.446` và `RQ = 0.447`, `9.0 / 30`). Qua kết quả đối chiếu với reference, tôi ghi nhận các thiếu sót thật còn tồn tại trong bản export hiện tại: lớp `sidewalk` bị bỏ trống trên hai ảnh của `easy_semantic` làm `sidewalk IoU = 0.000`, dãy xe đỗ nối đuôi trên `000000458325.jpg` có mask bị vẽ chồng lấn qua phần khuất của xe trước làm IoU giảm xuống `0.42–0.48`, và ranh `road`–`sidewalk` trên `000000460147.jpg` bị nhầm ở phần đường phụ bên hông. Các lỗi mask này chưa kịp chỉnh lại trên CVAT trước hạn nộp nên tôi giữ nguyên trạng thái trung thực như bản đã xuất.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. Ảnh `000000181542.jpg` (`medium_instance`), chiếc xe bán tải màu bạc ở làn giữa (`x ≈ 270..520`) | Có thể gán `car` vì kích thước và đầu xe giống xe con, hoặc gán `truck` vì có thùng chở hàng hở phía sau | Trong bài làm tôi gán là `car`; khi đối chiếu reference COCO thì mẫu này thuộc lớp `truck` với IoU vùng khớp đạt `0.533` | Nhờ coach xác nhận quy ước phân biệt ranh giới giữa `car` và `truck` đối với xe bán tải pickup |
| 2. Ảnh `000000458325.jpg` (`medium_instance`), hàng xe ô tô đỗ nối đuôi nhau bên lề phải (`x ≈ 350..590`) | Có thể vẽ bao trùm theo phom đầy đủ cả phần xe bị che, hoặc chỉ vẽ đúng lát cắt thân xe còn nhìn thấy phía sau xe đứng trước | Trong bài tôi vẽ rộng lấn sang vùng bị che khiến nhiều cặp mask trùng nhau; đối chiếu quy tắc chỉ vẽ phần nhìn thấy thì phải cắt sát ngay tại biên xe phía trước | Rút kinh nghiệm cắt biên sát phần nhìn thấy để tránh giảm IoU xuống dưới ngưỡng `0.50` khi chấm instance |
| 3. Ảnh `000000460147.jpg` (`hard_panoptic`), xe tải hai tầng chở ô tô ở giữa đường và phần mặt đường phụ bên lề | Với xe chở ô tô: tách riêng từng `car` trên sàn chở hay gộp chung vào `truck`; với phần làn phụ bên ngoài hàng rào: gán `road` hay `sidewalk` | Tôi đã tách từng `car` trên thùng xe nên bị tính lệch class so với `truck` ở reference; phần làn phụ bên rìa ngoài vỉa tôi tô `road` làm chênh diện tích với `sidewalk` | Nhờ coach làm rõ quy ước gán nhãn với xe chuyên dụng chở nhiều xe con trên thùng và cách tách ranh `sidewalk` ở làn đường phụ |
