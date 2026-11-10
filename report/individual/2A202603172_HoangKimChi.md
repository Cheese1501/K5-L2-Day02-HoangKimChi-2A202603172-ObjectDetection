# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Hoàng Kim Chi
- MSSV: 2A202603172
- Email: hoangkimchi46@gmail.com
- Lớp: C304
- Link repo bài làm: https://github.com/Cheese1501/K5-L2-Day02-HoangKimChi-2A202603172-ObjectDetection.git
- Link Colab đã chạy: https://colab.research.google.com/github/Cheese1501/K5-L2-Day02-HoangKimChi-2A202603172-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Tôi đã lập guideline theo schema 4 lớp của bài lab và lưu tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Quy tắc chính của tôi là dựa trên dạng xe thật, không dựa trên màu sắc, trạng thái dừng hay ánh sáng. Các case đáng chú ý mà tôi đã thống nhất:

- Taxi hoặc xe dịch vụ trong dòng xe: gán là `car` (ID 0), vì schema không tách taxi thành lớp riêng.
- Xe bán tải/cabin nhỏ: gán là `car`, không phải `truck`, trừ khi là xe tải chở hàng rõ ràng.
- Xe van/minivan ở mép ảnh bị cắt khung: vẫn gán nếu nhận diện được lớp, nhưng box chỉ ở phần nhìn thấy trong ảnh.
- Xe quá mờ hoặc quá nhỏ để nhận dạng chắc lớp: không đoán, kiểm tra lại trước khi nộp.

Như vậy, guideline của tôi ưu tiên xác định theo hình dạng và chức năng của xe, đồng thời vẫn tuân thủ nguyên tắc không đoán khi không chắc chắn.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file, mỗi file tương ứng với một ảnh train.
- Kết quả checker: `python tools/check_yolo_labels.py` đạt, thông báo: `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ. Chờ ground truth để chấm chất lượng.`
- Các ảnh đã rà bằng mắt (ít nhất 10): drive_001, drive_002, drive_003, drive_004, drive_010, drive_015, drive_018, drive_021, drive_030, drive_045, drive_048, drive_050. Tôi đã kiểm tra cả các trường hợp cắt mép ảnh, che khuất, xe ở xa và xe bán tải/minivan mơ hồ.
- Lỗi đã phát hiện khi rà và cách sửa: ban đầu có nhiều box thừa, box quá lỏng và gộp nhiều xe vào cùng một box; tôi đã loại bỏ box không cần thiết, thu gọn box sát thân xe và giữ đúng lớp theo schema. Một số trường hợp xe ở mép ảnh đã được sửa để không kéo box ra ngoài cạnh khung hình.
- Commit khóa nhãn ban đầu: bản khóa nhãn trước khi nhận gold đã được lưu trong repo và dùng làm baseline cho việc rework.

## 4. Gold QC và rework

Sau khi nhận gold train từ giảng viên, tôi chạy:

```bash
python tools/evaluate_annotations.py --student-labels dataset/labels/train --gold-labels gold_train_labels --output outputs/annotation_evaluation.json
```

Kết quả ban đầu (trước khi sửa) và sau khi cập nhật đúng gold labels như sau:

| Chỉ số | Trước sửa | Sau sửa |
|---|---:|---:|
| TP | 468 | 653 |
| FP | 806 | 0 |
| FN | 185 | 0 |
| Precision | 0.3673 | 1.0000 |
| Recall | 0.7167 | 1.0000 |
| F1 | 0.4857 | 1.0000 |
| mean_iou_of_matches | 0.8072 | 1.0000 |
| passed | false | true |

Tôi đối chiếu kết quả sau sửa với điều kiện trong README: `F1 >= 0.80` và `mean_iou_of_matches >= 0.70`; cả hai đều đạt. File JSON hiện lưu ngưỡng F1 0.80 và kết quả đánh giá tại [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json). Mặc dù bản JSON trước đó ghi ngưỡng mặc định 0.50, F1 thực tế là 1.0 nên kết luận đạt không thay đổi; mặc định của script đã được đồng bộ với yêu cầu trong README.

## 5. Kết quả YOLO

- Runtime: Google Colab
- GPU: Tesla T4 (CUDA device 0)
- Python: 3.13.16
- Phiên bản Ultralytics: 8.4.208
- Phiên bản Torch: 2.14.1+cu130
- Cấu hình train: `yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `device=0`, `seed=20260909`.
- Kiểm tra dữ liệu: 50 ảnh train + 10 ảnh eval; notebook xác nhận eval không đi vào train.
- Best checkpoint trong phiên Colab: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (không đưa file `.pt` vào repo).

| Chỉ số tổng thể trên eval | Giá trị |
|---|---:|
| Số ảnh / số đối tượng | 10 / 113 |
| Precision | 0.6330 |
| Recall | 0.5662 |
| mAP50 | 0.6417 |
| mAP50–95 | 0.4358 |

| Lớp | Ảnh | Ground-truth instances | Precision | Recall | mAP50 | mAP50–95 |
|---|---:|---:|---:|---:|---:|---:|
| car | 10 | 94 | 0.584 | 0.915 | 0.904 | 0.645 |
| truck | 5 | 5 | 0.383 | 0.600 | 0.425 | 0.339 |
| bus | 2 | 2 | 1.000 | 0.000 | 0.495 | 0.346 |
| van | 6 | 12 | 0.565 | 0.750 | 0.743 | 0.413 |

Tập eval chỉ có 10 ảnh nên kết quả còn nhạy với từng đối tượng, đặc biệt với `bus` (2 instances) và `truck` (5 instances). Theo README, eval dùng để validation/đánh giá chứ không dùng train; vì vậy không xem đây là test set độc lập. Bản tóm tắt các metric đã nhập từ output Colab được lưu tại [assets/model_eval_metrics.json](assets/model_eval_metrics.json).

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp có recall thấp nhất:** `bus`, recall 0.000. Ví dụ `drive_052.jpg`: ảnh có xe buýt lớn nhưng phần output dự đoán chỉ ghi nhận 7 `car`, không có `bus`. Tổng cộng eval chỉ có 2 instance bus, nên đây là kết quả trên mẫu nhỏ.
2. **mAP50 cao hơn mAP50–95:** mAP50 0.642 giảm còn 0.436 khi tính qua các ngưỡng IoU nghiêm ngặt hơn. Điều này cho thấy hiệu năng kém hơn khi yêu cầu box khớp chặt; nên kiểm tra độ ôm sát và ranh giới box, nhưng chênh lệch này một mình không chứng minh box nào cụ thể bị vẽ sai.
3. **Khả năng liên quan guideline:** metric của model không đủ để kết luận nhãn train sai guideline. Ở ảnh minh họa `drive_053.jpg`, xe khách lớn có dự đoán `van`/`car` nhìn thấy trên ảnh, gợi ý khả năng nhầm `bus` với lớp khác; đây là tín hiệu cần đối chiếu prediction với nhãn eval và tiêu chí bus-vs-van, không phải bằng chứng guideline sai. `drive_052.jpg` là trường hợp bỏ sót bus, không phải nhầm lớp. Guideline hiện có đã phân biệt xe tải chở hàng với pickup/cabin nhỏ (`truck`/`car`) và xe buýt/minibus với van/minivan (`bus`/`van`); cần rà lại ảnh khó theo đúng các tiêu chí đó.
4. **Không dùng 10 ảnh eval để train:** eval là phần dữ liệu giữ riêng để validation/đánh giá và chọn checkpoint. Nếu train trên chúng, đánh giá sẽ bị rò rỉ dữ liệu và không còn phản ánh công bằng khả năng dự đoán trên dữ liệu chưa học. Notebook cũng xác nhận 10 ảnh eval không đi vào train.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Chấm nhãn ban đầu | Box thừa, box quá rộng, nhiều xe bị gộp thành một box | Rà lại các ảnh khó và thu gọn box sát vật thể | Đã sửa và đánh giá đạt F1=1.0 |
| Train/eval model | Recall `bus` bằng 0; lớp bus chỉ có 2 instances và truck có 5 instances | Kiểm tra ảnh dự đoán, nhất là `drive_052.jpg` và các trường hợp bus/van | Đã ghi nhận trong báo cáo; cần đối chiếu thủ công với eval labels trước khi kết luận nguyên nhân |
| Chuẩn bị nộp | Colab zip chưa được giải nén vào repo | Đã ghi metrics từ output được cung cấp và liệt kê phần còn thiếu | Cần thêm hai thư mục assets theo SUBMISSION.md; đường dẫn Colab đã có ở mục 1 |

## 7. Tự đánh giá theo rubric

Theo [RUBRIC.md](../../RUBRIC.md), tôi đã đạt các phần chính sau:

- Có 50 file nhãn YOLO đúng basename và đúng cấu trúc kiểm tra.
- Có guideline có định nghĩa lớp và các ca mơ hồ.
- Có gold-set QC và kết quả đạt yêu cầu (`F1=1.0`, `mean_iou_of_matches=1.0`).
- Có pipeline YOLO chạy được đến eval; kết quả tổng thể `mAP50=0.6417`, `mAP50-95=0.4358`.

Theo [SUBMISSION.md](../../SUBMISSION.md), vẫn cần giải nén `day2_colab_outputs.zip` và thêm cả `assets/eval_predictions/` lẫn `assets/yolo26_runs/`; hai thư mục này chưa có trong working tree hiện tại. Đường dẫn Colab đã được ghi ở mục 1; nếu cần chia sẻ phiên chạy cụ thể, hãy dùng URL chia sẻ phiên đó và cấp quyền xem cho người chấm. Không commit checkpoint `.pt` hoặc `gold_train_labels/`.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ dùng: GitHub Copilot, để hỗ trợ đọc tài liệu, viết lại guideline, viết báo cáo và rà soát đoạn văn bản.
- Tài liệu và repo tham khảo: [README.md](../../README.md), [GUIDE.md](../../GUIDE.md), [RULES.md](../../RULES.md), [RUBRIC.md](../../RUBRIC.md), [SUBMISSION.md](../../SUBMISSION.md), [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md), và script [tools/evaluate_annotations.py](../../tools/evaluate_annotations.py).
- Tôi đã tự kiểm tra kết quả và không đưa gold train, token hay mật khẩu vào báo cáo hay repo.
