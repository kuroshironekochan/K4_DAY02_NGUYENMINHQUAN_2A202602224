# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Nguyễn Minh Quân <br>
**MSSV:** 2A202602224<br>
**Hình thức:** Solo <br>
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: 2d26949a8419e3b877e3633fc51d4090e3360565511cf501abf401d4e93b6900
- Bốn mã ảnh: drive_022, drive_033, drive_038, drive_008
- Số vật thể thực tế: 442
- Mã SHA-256 của gói YOLO của bạn: 9959063d4b4748d9cd3a84a6347f95a1e54ce29a27529e35696e88cc5886c83a
- Mã SHA-256 của gói CVAT gốc của bạn: d1265f372eb0a78949120f7d6fe3cd08edfc567a962e6d42042d7cc0e0af902a
- Nguồn đối chiếu:SQiFeng/traffic-vehicle-detection
- Mã SHA-256 của gói đối chiếu: 710d2c157c5750f71271354d0d31b24457c64e7c
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: 17, 16:32, 14/9/2026

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

- Tôi làm việc cá nhân và tự mình gán nhãn độc lập toàn bộ cả 4 bức ảnh mà không phân chia ảnh hay phân chia lớp với người khác. Tôi hoàn thành phần việc của mình mà không trao đổi hay xem xét dữ liệu với bất kì cá nhân nào khác cho đến khi bước đối chiếu.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| drive_008 (xe làn giữa) | car | Kiểu dáng sedan 4 chỗ, nắp capo và đuôi xe dốc thấp, kính chắn gió trước nghiêng, không có thân khối hộp cao và không có khoang chở hàng hở. | Thuộc lớp `car` vì có dáng phổ thông của sedan, đồng thời không có đặc điểm nhận dạng của xe tải (thân khối cao, khoang hàng mở) hay xe khách (thân dài, nhiều cửa sổ dọc). |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Ví dụ, lớp `truck` chỉ phân biệt loại xe tải, còn thuộc tính `occlusion` (che khuất) cho biết mức độ che phủ, do đó giúp mô hình phân biệt xe tải bị che khuất với xe tải không bị che khuất.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| Xe con ở mép phải dưới ảnh `drive_008` có hộp vẽ trùm ra ngoài biên ảnh | phạm vi/lớp/hình học/thuộc tính | quan sát thấy hộp giới hạn xe con vượt ra ngoài khung hình, trong khi quy tắc yêu cầu giữ hộp trong phạm vi ảnh | sửa lại bounding box cho xe con sao cho nằm gọn trong khung hình |

- Số hộp `needs_review` trước và sau khi kiểm: trước khi kiểm tra có 26 hộp tập chung ở các hộp chị che/cắt mép/ mờ, sau kiểm tra còn 17 hộp

- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: trong ảnh drive 008 có rất nhiều xe bị che khuất, cắt mép và mờ nên tôi không chắc có nên gán nhãn với các xe này hay không, sau khi hỏi người hướng dẫn thì được hỗ trợ thêm về việc gán nhãn

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: 0, 0.693125, 0.519977, 0.168125, 0.162453
- Tên lớp và tọa độ điểm ảnh `xyxy`: car [389.8, 280.8, 497.4, 384.8]
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?
1. Sai lớp: Dòng ghi lớp `0` (car) nhưng thực tế vật thể trong ảnh là xe van hoặc xe tải nhỏ; parser chỉ thấy số 0 hợp lệ chứ không đối chiếu với hình ảnh thực tế.
2. Sai phạm vi: Hộp có thể bao quanh một đối tượng không thuộc phạm vi (người, xe máy, bóng phản chiếu) hoặc gộp chung 2 xe vào một hộp lớn; dòng nhãn vẫn đúng 5 trường nhưng vi phạm quy tắc mỗi phương tiện một hộp.
3. Sai hình học: Tọa độ vẫn thuộc [0, 1] nhưng hộp có thể vẽ lỏng chứa nhiều nền, vẽ chệch vào phần bị che khuất hoặc cắt lẹm vào thân xe. Định dạng YOLO cũng không lưu thuộc tính `visibility` hay `boundary` để kiểm tra độ tin cậy của hộp.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: drive_022, drive_033, drive_038
- Mã ảnh thẩm định: drive_008
- Mô tả một dự đoán trong `detect_result.jpg`: Xe van màu trắng ở góc trên bên phải được phát hiện đúng, nhưng một xe tải màu đỏ ở phía dưới bên trái lại bị bỏ sót.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào?
 1. Xe tải bị bỏ sót cho thấy mô hình chưa học tốt cách nhận dạng xe tải hoặc khoảng cách quá xa, cần kiểm tra lại quy tắc gán nhãn xe tải và bổ sung thêm dữ liệu huấn luyện có xe tải ở các điều kiện tương tự.

- Minh chứng nào có thể bác bỏ nhận định của bạn?
1. Hình ảnh drive_008 được gán nhãn với bounding box chứa thân xe tải màu đỏ, chứng tỏ nó có thể được nhận dạng khi có đủ ngữ cảnh và độ phân giải.

- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?
1.  kết quả chỉ đánh giá mô hình trên 4 ảnh với các điều kiện ánh sáng, góc chụp, khoảng cách và bố cục cố định, không bao gồm các trường hợp đa dạng như đường phố thực tế, điều kiện thời tiết khác, các loại xe và kích thước khác nhau, hoặc các vật thể che khuất một phần

## 6. Đối chiếu nhãn

- Số hộp ghép được: 111
- IoU trung bình và trung vị: IoU trung bình là 0.9035, IoU trung vị là 0.9167
- Mức đồng thuận lớp: 100%
- Số hộp phía bạn không ghép được: 0
- Số hộp phía đối chiếu không ghép được: 0
- Một điểm khác biệt cụ thể: Hộp cho xe con ở phía trước bên phải bị che bởi xe tải
- Quy tắc hoặc hành động sửa phát sinh: Xe con bị che bởi xe tải nên hộp của xe con bị cắt 
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?
1.  mức đồng thuận chỉ cho biết hai người có cùng chọn một nhãn hay không, chứ không đảm bảo nhãn đó phù hợp với định nghĩa của lớp, bao phủ đúng đối tượng hoặc được gán nhãn theo quy tắc.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

1. Quy trình gán nhãn độc lập được kiểm soát chặt chẽ bằng mã băm SHA-256 trước khi tiếp xúc với nguồn đối chiếu, cùng với việc duy trì đầy đủ 3 thuộc tính (`visibility`, `boundary`, `review_state`) trên toàn bộ các hộp nhãn trong gói CVAT gốc.
