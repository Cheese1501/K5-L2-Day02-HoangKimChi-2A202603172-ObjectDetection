# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Hoàng Kim Chi
- MSSV: 2A202603172
- Email: hoangkimchi46@gmail.com
- Lớp: C304
- Link repo bài làm: https://github.com/Cheese1501/K5-L2-Day02-HoangKimChi-2A202603172-ObjectDetection.git
- Link Colab đã chạy: TBD (notebook đã chạy ở môi trường local và trên Colab theo hướng dẫn lab)

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

Tôi đã dùng giá trị sau sửa để đối chiếu với điều kiện đạt: `F1 >= 0.80` và `mean_iou_of_matches >= 0.70`. Kết quả đạt cả hai chỉ tiêu, nên bộ nhãn đã sẵn sàng dùng cho bước train. File đánh giá có trong [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json).

## 5. Kết quả YOLO

- Runtime: CPU
- GPU/CPU: CPU
- Phiên bản Ultralytics: 8.4.208
- Phiên bản Torch: 2.15.0
- Cấu hình train: `yolov8n.pt`, `epochs=10`, `imgsz=640`, `batch=-1`, `device='cpu'`, `seed=20260909`.
- Đường dẫn best checkpoint trong phiên chạy: `runs/detect/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.214 |
| Recall | 0.635 |
| mAP50 | 0.337 |
| mAP50–95 | 0.222 |

Tôi đã chạy notebook theo hướng dẫn và thu được các kết quả trên trên tập eval. Vì tập `eval` chỉ có 10 ảnh nên chỉ là validation/model selection, không phải test set độc lập. Dạng xe phổ biến nhất là `car`, nên mô hình học tốt hơn ở lớp này hơn so với `truck`, `bus` và `van`.

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? `truck` có recall thấp nhất trong số các lớp, vì dữ liệu train rất ít và hình dạng xe tải dễ bị nhầm với `car`/`van` nếu không nhìn rõ phần thùng hàng. Các ảnh eval có xe tải ở xa hoặc bị che một phần là trường hợp khó nhất.
2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm tra độ chính xác của box. Nghĩa là box dự đoán có thể đâm đúng vị trí theo ngưỡng 0.5 nhưng không sát vật thể ở mức IoU cao hơn, dẫn đến sai lệch hình học hoặc box quá rộng/quá lỏng.
3. Lỗi class nào có thể liên quan guideline? Cặp lỗi dễ thấy nhất là `car` với `truck`/`pickup`, và `van` với `bus`/`minibus`. Nếu guideline không làm rõ rõ tiêu chí phân biệt xe bán tải/cabin nhỏ và xe tải chở hàng, model sẽ bị nhầm lớp ngay ở những trường hợp vừa đủ mơ hồ.
4. Vì sao không dùng 10 ảnh eval để train? Bởi vì `eval` là tập validation dùng để chọn best checkpoint và đánh giá mô hình; nếu dùng chúng để train thì metric sẽ bị inflated và không còn phản ánh khả năng khái quát của mô hình. Theo quy định lab, `eval` chỉ có vai trò kiểm tra, không dùng làm dữ liệu huấn luyện.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Chấm nhãn ban đầu | Box thừa, box quá rộng, nhiều xe bị gộp thành một box | Rà lại các ảnh khó và thu gọn box sát vật thể | Đã sửa và đánh giá đạt F1=1.0 |
| Train model | Class imbalance rõ rệt, `truck`/`bus`/`van` thấp hơn `car` nhiều | Kiểm tra lại class guideline và đánh giá trên eval | Model học tốt `car`, các lớp ít dữ liệu còn yếu |
| Chuẩn bị nộp | Chưa sẵn sàng cho GitHub nếu không có báo cáo và assets | Hoàn thiện report và chuẩn bị folder assets theo yêu cầu lab | Cần tạo thêm folder assets từ Colab khi nộp chính thức |

## 7. Tự đánh giá theo rubric

Theo [RUBRIC.md](../../RUBRIC.md), tôi đã đạt các phần chính sau:

- Có 50 file nhãn YOLO đúng basename và đúng cấu trúc kiểm tra.
- Có guideline có định nghĩa lớp và các ca mơ hồ.
- Có gold-set QC và kết quả đạt yêu cầu (`F1=1.0`, `mean_iou_of_matches=1.0`).
- Có pipeline YOLO chạy được đến eval và lưu checkpoint.

Một phần còn cần bổ sung khi nộp chính thức là folder `report/individual/assets/` sau khi giải nén zip Colab, vì hiện tại phần này chưa được upload vào repo. Ngoài ra, nếu cần nộp link GitHub và link Colab chính thức, tôi sẽ cập nhật sau khi push và chia sẻ link trên LMS.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ dùng: GitHub Copilot, để hỗ trợ đọc tài liệu, viết lại guideline, viết báo cáo và rà soát đoạn văn bản.
- Tài liệu và repo tham khảo: [README.md](../../README.md), [GUIDE.md](../../GUIDE.md), [RULES.md](../../RULES.md), [RUBRIC.md](../../RUBRIC.md), [SUBMISSION.md](../../SUBMISSION.md), [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md), và script [tools/evaluate_annotations.py](../../tools/evaluate_annotations.py).
- Tôi đã tự kiểm tra kết quả và không đưa gold train, token hay mật khẩu vào báo cáo hay repo.

