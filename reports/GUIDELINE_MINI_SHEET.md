# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Nguyễn Minh Quân<br>
**MSSV:** 2A202602224<br>
**Hình thức:** Cá nhân<br>
**Mã cặp:** SOLO

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` (xe buýt) | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` (xe van) | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt; khoang hàng tách biệt như xe tải |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `visibility` (mức nhìn thấy) | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy |
| `boundary` (quan hệ mép ảnh) | `inside` (trong ảnh), `truncated` (bị cắt) | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại) | đánh dấu quyết định cần quay lại |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: `drive_008` (vật thể #3, tọa độ pixel `[261.2, 240.7, 367.0, 391.1]`)
- Dấu hiệu nhìn thấy: Thân xe dạng khối hộp kín một khối (monobox), trần cao phẳng, nhưng chiều dài thân xe ngắn (dưới 6m), chỉ có 1 cửa sổ phụ bên hông và cửa lùa trượt, không có dải nhiều hàng cửa sổ khách dài liên tục như xe buýt.
- Quy tắc áp dụng: Quy tắc 2 — Lớp `van` (xe thân hộp nhỏ kín chở người hoặc hàng); không gán `bus` vì thân ngắn, ít cửa sổ/hàng ghế.
- Quyết định: Gán nhãn `van`.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng to 200% để kiểm tra số hàng ghế và tỷ lệ chiều dài thân xe; nếu vẫn thiếu bằng chứng do bị che khuất thì tạm gắn cờ `needs_review` và ghi chú vào nhật ký để đối chiếu.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_022` (vật thể #5, tọa độ pixel `[570.9, 252.6, 634.9, 330.7]`)
- Dấu hiệu nhìn thấy: Buồng lái phía trước tách rời hoàn toàn với thùng chở hàng phía sau; có khoảng hở giữa cabin và thùng xe, khung gầm cao và có trục bánh kép chịu tải.
- Quy tắc áp dụng: Quy tắc 2 — Lớp `truck` (gán khi thấy rõ thùng, sàn chở hàng hoặc buồng lái tách biệt); không gán `van` hay `car` vì thân không phải một khối kín liền mạch.
- Quyết định: Gán nhãn `truck`.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng to 150% để quan sát kỹ khớp nối giữa cabin và thùng xe; nếu góc chụp chỉ thấy đầu xe mà không rõ thùng thì tạm gắn cờ `needs_review` và đối chiếu với luồng di chuyển.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_008` (vật thể #15, tọa độ pixel `[601.0, 548.9, 640.0, 640.0]`)
- Dấu hiệu nhìn thấy khi phóng 100%: Thân xe bị đường biên mép ảnh dưới và mép phải cắt ngang (chỉ thấy phần đầu xe và nắp capo), đồng thời có xe phía trước che khuất một phần góc cản xe.
- Giá trị `visibility`: `clear`
- Giá trị `boundary`: `truncated`
- Trạng thái `review_state`: `confident`
- Lý do: Mặc dù xe chạm mép ảnh và bị che một phần, nhưng các đặc trưng nhìn thấy được (đèn pha, nắp capo và kính chắn gió) đủ bằng chứng rõ ràng để xác định là xe con (`car`), và hộp bao được vẽ sát đúng phần nhìn thấy theo Quy tắc 3.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [x] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [x] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 111 — 40–60 là mục tiêu khối lượng, không phải điểm cắt.
