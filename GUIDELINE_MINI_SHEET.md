# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Phù Ngô Việt Anh<br>
**MSSV:** 2A202602141<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** `SOLO`

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

- Ảnh và mã vật thể: `drive_022`, vật thể 1 (dòng đầu trong `drive_022.txt`, lớp ban đầu `bus`).
- Dấu hiệu nhìn thấy: Thân xe lớn và dài, có nhiều cửa sổ/khoang hành khách; hình dáng tổng thể nghiêng về xe buýt nhưng ảnh giao thông xa và có phần bị che.
- Quy tắc áp dụng: Gán `bus` khi thấy thân xe khách dài, nhiều cửa sổ hoặc hàng ghế; chỉ gán `van` khi là thân hộp nhỏ, kín.
- Quyết định: Gán lớp `bus`; nếu không nhìn đủ thân xe thì giữ `review_state=needs_review` thay vì đoán. Đây là trường hợp cần đối chiếu vì bộ tham chiếu ghi khác lớp (`van`) dù IoU hình học là `0,86102`.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Không đổi lớp theo cảm tính; ghi rõ dấu hiệu còn thiếu, giữ hộp và cờ `needs_review`, sau đó xin Lab Coach xác nhận quy tắc.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_038`, vật thể 16 (lớp ban đầu `van`, cặp đối chiếu là `truck`).
- Dấu hiệu nhìn thấy: Vật thể có dạng thân hộp/xe chở hàng ở giữa cảnh; phần thùng hoặc thiết bị công vụ không đủ rõ để khẳng định hoàn toàn, nên dễ nhầm giữa `van` và `truck`.
- Quy tắc áp dụng: Gán `truck` khi thấy rõ thùng, ben, sàn hàng hoặc thiết bị công vụ; gán `van` khi là thân hộp nhỏ kín và không có khoang hàng tách biệt.
- Quyết định: Giữ quyết định ban đầu là `van` khi chưa thấy thùng/ben tách biệt, nhưng đánh dấu cần xem lại; nếu thấy khoang hàng rõ thì sửa thành `truck`.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng ảnh 100%, kiểm tra phần sau thân xe và không suy đoán phần bị che; giữ `needs_review` và xin hỗ trợ nếu vẫn không thấy dấu hiệu phân lớp.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_008`, hộp xe buýt nhỏ gần mép trên, tọa độ xấp xỉ `(351.46, 0.32)–(377.87, 41.21)`.
- Dấu hiệu nhìn thấy khi phóng 100%: Vật thể rất nhỏ, chỉ thấy một phần thân xe ở sát phía trên ảnh; không đủ chi tiết để chắc chắn về lớp và mức nhìn thấy.
- Giá trị `visibility`: `unclear`.
- Giá trị `boundary`: `inside` (hộp vẫn nằm trong biên ảnh, dù sát mép trên).
- Trạng thái `review_state`: `needs_review`.
- Lý do: Bằng chứng hình ảnh yếu và vật thể gần mép ảnh; không được ước lượng phần bị che hoặc tự đổi lớp khi chưa có dấu hiệu đủ rõ.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh: `drive_008`, `drive_022`, `drive_033`, `drive_038`.
- [x] Đã kiểm vật thể thiếu và trùng qua audit xuất và bảng đối chiếu.
- [x] Đã kiểm lớp và hình học từng hộp bằng nhãn YOLO, CVAT và ảnh phủ.
- [x] Mỗi hộp có đủ ba thuộc tính: 64/64 hộp có `visibility`, `boundary`, `review_state`.
- [x] Đã xử lý mọi hộp `needs_review`: hộp còn chưa chắc chắn đã được ghi lý do và giữ cờ, không bị đoán lớp.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu, theo quy trình làm bài độc lập.
- [x] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi: không áp dụng vì làm cá nhân.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 61 — 40–60 là mục tiêu khối lượng, không phải điểm cắt.
