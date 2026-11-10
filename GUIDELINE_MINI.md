# Mini guideline — bài làm cá nhân cho 50 ảnh train

## 1. Định nghĩa lớp

| ID | Lớp | Định nghĩa | Ví dụ / điểm phân biệt |
| --- | --- | --- | --- |
| 0 | `car` | Xe con, sedan, SUV, taxi, xe bán tải/cabin nhỏ. | Nếu là xe hơi cá nhân, taxi hoặc xe bán tải cỡ nhỏ thì gán `car`; không gán `truck` nếu không phải xe tải chở hàng rõ ràng. |
| 1 | `truck` | Xe tải chở hàng, xe đầu kéo, xe container. | Phân biệt với xe bán tải/cabin nhỏ thuộc `car`. |
| 2 | `bus` | Xe buýt, minibus chở khách, xe khách lớn. | Nếu phần thân có hình dạng xe khách dài và chở nhiều người, gán `bus`. |
| 3 | `van` | Xe van, minivan, xe chở hàng hoặc chở người dạng van. | Khi xe có hình dạng van/minivan nhưng không phải xe buýt lớn, gán `van`. |

## 2. Quy tắc chung khi gán nhãn cho 50 ảnh train

- Mỗi xe được gán một bounding box duy nhất, vừa ôm phần thân xe nhìn thấy được.
- Với xe bị che khuất một phần, tôi vẫn giữ nhãn nếu có thể nhận diện rõ lớp xe; không vẽ quá rộng hoặc bao gồm cột, chắn hoặc vật cản khác.
- Với xe cắt mép ảnh, tôi vẽ box chỉ trong phạm vi thật của ảnh; không đoán phần nằm ngoài khung hình.
- Nếu xe quá nhỏ, mờ hoặc bị che hoàn toàn đến mức không thể xác định lớp, tôi không đoán; tôi lưu ảnh đó và kiểm tra lại trước khi nộp.
- Xe đang đỗ, đứng ở bãi đỗ hay dừng đèn đỏ vẫn được gán theo lớp xe tương ứng.
- Không gán người đi bộ, mô tô, xe đạp, biển báo, đèn giao thông, phản chiếu, hoặc các đối tượng không phải xe thật trong cảnh.
- Giữ nguyên mã lớp YOLO: `0=car`, `1=truck`, `2=bus`, `3=van`.

## 3. Các trường hợp dễ nhầm và quyết định của tôi

| Ảnh | Tình huống | Quyết định cuối | Lý do |
| --- | --- | --- | --- |
| `drive_001` | Xe con màu bạc ở bên phải bị cột che một phần. | `car` (ID 0) | Phần thân xe còn nhìn thấy đủ để nhận diện là xe con; không bao gồm cột hoặc phần che khuất. |
| `drive_002` | Xe van/mini-van ở mép phải bị cắt khung ảnh. | `van` (ID 3) nếu hình dạng là van; còn nếu không đủ đảm bảo thì bỏ qua và kiểm tra lại | Xe cắt mép ảnh vẫn được gán nếu nhận diện được lớp; box chạm rìa nhưng không đoán phần nằm ngoài ảnh. |
| `drive_003` | Taxi hoặc xe dịch vụ nằm trong hàng xe. | `car` (ID 0) | Theo schema, taxi là `car`; không tách thành lớp riêng. |
| `drive_010` | Xe ở xa, thân xe mờ nhưng rõ là phương tiện con người. | `car` (ID 0) nếu phần nhìn thấy đủ để xác định; nếu quá mờ thì không đoán | Không gán nhãn theo ánh đèn hay độ sáng; chỉ gán khi hình dạng xe đủ rõ. |
| `drive_030` | Xe tải cabin nhỏ hoặc pickup; dễ nhầm với truck. | `car` (ID 0) | Xe bán tải/cabin nhỏ thuộc `car`, không phải `truck`, trừ khi là xe tải chở hàng rõ ràng. |
| `drive_045` | Xe ở mép khung ảnh, chỉ thấy một nửa thân. | Gán nếu có thể nhận diện lớp; box chạm mép ảnh | Không cần mở rộng box ra ngoài ảnh; chỉ vẽ phần thật sự nhìn thấy. |

## 4. Quy tắc phân loại đối với các đối tượng mơ hồ trong dataset

- `car`: xe con, taxi, SUV, hatchback, sedan, xe bán tải/cabin nhỏ; nếu không rõ là xe tải chở hàng thì ưu tiên `car`.
- `truck`: xe tải chở hàng, thùng hàng, đầu kéo, container; không gán cho xe pickup/cabin nhỏ.
- `bus`: xe khách dài lớn, minibus/chở khách; nếu nhìn như xe khách hay xe buýt thì ưu tiên `bus`.
- `van`: xe van, minivan, xe chở hàng/xe người dạng van; nếu xe nhỏ hơn xe buýt nhưng lớn hơn xe con và hình dạng giống van thì chọn `van`.

## 5. Nhiệm vụ của tôi với toàn bộ 50 ảnh

- Kiểm tra từng ảnh trong `dataset/images/train` và khớp với file `.txt` tương ứng trong `dataset/labels/train`.
- Một ảnh có thể chứa nhiều xe; mỗi xe phải có một box riêng và không gom thành một box chung.
- Box cần sát thân xe, không bao gồm lề đường, vệt sáng, đường viền, hay đèn chắn.
- Mức độ chặt và nhất quán phải giữ như nhau trên 50 ảnh để giảm sai lệch khi đánh giá gold set.

## 6. Kết luận

Tôi sẽ giữ nguyên schema 4 lớp, ưu tiên xác định theo dạng thân xe và chức năng vận tải chứ không theo màu sắc, trạng thái dừng, hay ánh sáng. Nếu một xe quá nhỏ hoặc quá mờ để xác định chắc lớp, tôi sẽ không đoán mà sẽ kiểm tra lại trước khi nộp.