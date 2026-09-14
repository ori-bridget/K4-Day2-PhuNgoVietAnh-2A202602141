# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Phù Ngô Việt Anh<br>
**MSSV:** 2A202602141<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: F7D99888F21440FB0374D84962B93213BD8C14E665D093CC8D37F4C61B71ED33
- Bốn mã ảnh:008, 022, 033, 038
- Số vật thể thực tế: 61
- Mã SHA-256 của gói YOLO của bạn: B0C2ED0B82105F71F8191B4C7CCFA8AC10E5C4B1DBE5A8D4318CE946F5149639
- Mã SHA-256 của gói CVAT gốc của bạn: 349B921849262E56BD9918DC6780786C85D583D7D1C4C5BE8B4C6250767F89C8
- Nguồn đối chiếu:bộ tham chiếu do người hướng dẫn thực hành cấp
- Mã SHA-256 của gói đối chiếu: C8BBC767D8BB9A29F4CA5ABF0C3516E5C2AF94C58143A980B0148CFE0B500D2B
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: day2-teaching-reference.zip

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:
Làm bài, chạy local hoàn toàn trên máy, không có sự trợ giúp từ học viên khác, chỉ có hướng dẫn từ lab coach

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| Các phương tiện trong bốn ảnh; tổng cộng 64 hộp | `car`, `truck`, `bus`, `van` | Nhìn hình dáng thân xe: ô tô con/SUV/taxi/bán tải dùng như xe con; thùng/ben hoặc thiết bị công vụ; thân xe khách dài nhiều cửa sổ; hoặc thân hộp nhỏ kín | Dùng đúng bốn lớp cố định `0 car, 1 truck, 2 bus, 3 van`; không gán người, xe máy, xe đạp, biển báo hay phần phản chiếu |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau: Ví dụ trong `drive_008`: một phương tiện có lớp `bus`, nhưng thuộc tính có thể là `visibility=occluded`, `boundary=inside`, `review_state=confident`. `bus` trả lời “đây là loại phương tiện gì”, còn các thuộc tính trả lời “nhìn thấy rõ đến đâu, có bị mép ảnh cắt không và quyết định đã đủ chắc chắn chưa”.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| `drive_008`, hộp xe buýt nhỏ ở gần mép trên ảnh; một hộp ô tô nhỏ có IoU đối chiếu thấp | phạm vi/lớp/hình học/thuộc tính | Rà ảnh ở mức 100%, xem `comparison_overlay.png`, đối chiếu bảng IoU; hộp xe buýt có dấu hiệu `visibility=unclear` và `review_state=needs_review` | Giữ hộp sát phần nhìn thấy, không ước lượng phần bị che/cắt; giữ trạng thái `needs_review` khi bằng chứng chưa đủ, và dùng quy tắc hình dáng thân xe để phân lớp |

- Số hộp `needs_review` trước và sau khi kiểm: gói xuất cuối có 1 hộp; không có bản xuất trước kiểm riêng để xác nhận số ban đầu, nên sau kiểm vẫn giữ 1 hộp. Hộp này là xe buýt nhỏ ở `drive_008` (`visibility=unclear`, `boundary=inside`), được giữ cờ xem lại thay vì đoán.
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: hộp xe buýt nhỏ ở gần mép trên `drive_008` khó phân biệt đầy đủ do kích thước và bị cắt. Tôi ghi `needs_review`, nêu rõ vị trí và dấu hiệu quan sát được, rồi xin Lab Coach xác nhận quy tắc trước khi đổi lớp hoặc bỏ hộp.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `2 0.385094 0.727867 0.445437 0.371672` (dòng đầu của `drive_022.txt`).
- Tên lớp và tọa độ điểm ảnh `xyxy`: lớp `2` là `bus`; với ảnh 640×640, hộp xấp xỉ `[103.92, 346.90, 389.00, 584.77]`.
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học? Vì YOLO chỉ kiểm tra cấu trúc năm số và tọa độ chuẩn hóa; nó không tự biết người gán có chọn đúng lớp, vẽ sát vật thể, loại phần bị che hay xử lý mép ảnh đúng hay không.

 

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`.
- Mã ảnh thẩm định: `drive_008`.
- Mô tả một dự đoán trong `detect_result.jpg`: ảnh thẩm định là cảnh giao thông đông, có xe buýt, xe tải và nhiều ô tô; file hiện không hiển thị hộp dự đoán rõ ràng, nên không thể coi việc không thấy hộp là dự đoán đúng hay âm tính thật.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Cần kiểm ngưỡng tin cậy/cách xuất ảnh dự đoán, độ bao phủ của dữ liệu huấn luyện và các trường hợp xe buýt–xe van–xe tải dễ nhầm; chỉ 8 epoch trên ba ảnh cũng chưa đủ để kết luận mô hình học tốt.
- Minh chứng nào có thể bác bỏ nhận định của bạn? Chạy lại cùng trọng số với log dự đoán thô, hạ và ghi rõ ngưỡng confidence, kiểm tra số hộp/điểm confidence; sau đó thử trên thêm ảnh độc lập có nhãn chuẩn.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế? Chỉ có bốn ảnh, một ảnh thẩm định, phân bố cảnh rất hẹp và ảnh thẩm định không đại diện cho dữ liệu triển khai; số lượng quá nhỏ để ước lượng recall, precision hay độ bền với cảnh mới.

 

## 6. Đối chiếu nhãn

- Số hộp ghép được: 41.
- IoU trung bình và trung vị: trung bình `0,8132`; trung vị `0,840614`.
- Mức đồng thuận lớp: `0,707317`, tương đương `70,73%` trên 41 hộp ghép.
- Số hộp phía bạn không ghép được: 23.
- Số hộp phía đối chiếu không ghép được: 9.
- Một điểm khác biệt cụ thể: ở `drive_022`, hộp của tôi là `bus` còn hộp đối chiếu là `van` dù IoU hình học là `0,86102`; ở `drive_008`, một cặp `car` có IoU chỉ `0,160564`, cho thấy còn khác biệt về hình học.
- Quy tắc hoặc hành động sửa phát sinh: khi phân biệt `bus`/`van`/`truck`, ưu tiên dấu hiệu thân xe (thân dài nhiều cửa sổ; thân hộp nhỏ kín; thùng/ben hoặc thiết bị công vụ), đồng thời rà lại các hộp có IoU thấp bằng ảnh phủ trước khi đổi nhãn.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng? Hai bên có thể cùng lặp lại một quy tắc sai hoặc cùng bỏ sót một vật thể; IoU chỉ đo độ chồng hình học, còn độ đúng của lớp, phạm vi, thuộc tính và tính đại diện dữ liệu vẫn cần kiểm độc lập.

 

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ: `GUIDELINE_MINI_SHEET.md`.
- [x] Có kết quả kiểm hai gói xuất: `my_export_audit.json` và `my_native_export_audit.json`.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán: `training_run.json` và `detect_result.jpg`.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu: `comparison_summary.json`, `comparison_iou.csv` và `comparison_overlay.png`.
- [ ] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình: chưa thể xác nhận trạng thái kho GitHub từ thư mục làm việc hiện tại; các tệp ZIP đầu vào/xuất và bộ tham chiếu vẫn đang có trong thư mục cục bộ.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập trong các tệp đầu ra đã kiểm tra.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach: Minh chứng mạnh nhất là `comparison_summary.json` đi kèm `comparison_iou.csv` và `comparison_overlay.png`, vì chúng cho thấy đồng thời số hộp ghép, IoU, đồng thuận lớp và các trường hợp khác biệt cụ thể. Câu hỏi còn lại cho Lab Coach: với các trường hợp hình học ghép tốt nhưng lớp khác nhau như `drive_022` (bus/van), nên ưu tiên dấu hiệu nào khi thân xe bị che hoặc nhìn từ xa?
