# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Cù Đức Quang<br>
**MSSV:** 2A202602188<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** `SOLO`

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: `drive_008`, `drive_022`, `drive_033`, `drive_038` (đều 640×640).
- Số vật thể thực tế: **93** (`car=70`, `truck=5`, `bus=10`, `van=8`).
- Mã SHA-256 của gói YOLO của bạn: `2f6fcb24acf8a71b03a7ef63ca4dfa5e7f527f143bbf7cf3190024c229092abc`
- Mã SHA-256 của gói CVAT gốc của bạn: `344881dd6c8b96362872feb1fb6cd0f4f2640a04c8ecee46ec132f110225aadb`
- Nguồn đối chiếu: bộ tham chiếu teaching do Lab Coach cấp (`day2-reference-4img-v1`).
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: `day2-reference-4img-v1`, nhận từ `day2-teaching-reference.zip` ngày 14/09/2026.

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

Mình tự gán cả bốn ảnh, tự kiểm tra CVAT, khóa SHA-256 gói YOLO và gói CVAT trước khi mở ZIP tham chiếu. Hai gói của mình được xuất từ cùng một job sau QC; không dùng reference để tạo hoặc sửa bản riêng.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_008`, xe khách lớn | `bus` | thân dài, nhiều cửa sổ | thân xe khách dài là `bus` |
| `drive_008`, xe ben đỏ | `truck` | cabin và thùng ben tách biệt | thùng/ben rõ là `truck` |
| `drive_038`, xe cứu hộ | `truck` | sàn chở và cần cẩu/thiết bị công vụ | thiết bị công vụ rõ là `truck` |
| `drive_038`, xe đỏ thân hộp | `van` | thân ngắn, kín, không có dãy cửa sổ dài | thân hộp nhỏ, không phải bus/truck |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Xe con ở mép trái `drive_038` vẫn có lớp `car`, nhưng `boundary=truncated` vì bị mép ảnh cắt. Lớp là loại xe; thuộc tính là mức nhìn thấy, quan hệ mép ảnh và trạng thái xem lại.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| 105 track một keyframe lan sang frame sau | phạm vi | API job và overlay cho thấy export trước có 275 box thay vì 132 đối tượng | chuyển thành rectangle shape riêng từng frame |
| class map là `car,truck,van,bus` | lớp | đọc `data.yaml` | sửa thứ tự thành `car,truck,bus,van`, bảo toàn lớp ngữ nghĩa |
| 39 box rất nhỏ/mờ hoặc trùng | phạm vi/lớp | xem ảnh gốc 100% và crop | bỏ box không đủ bằng chứng, còn 93 box |
| Xe mép trái `drive_038` có `boundary=inside` | thuộc tính/hình học | box bắt đầu tại x≈0,97 và xe bị cắt | kéo x trái về 0, đặt `truncated` |
| Xe nằm trọn `drive_008` có `boundary=truncated` | thuộc tính | cách mép trên khoảng 15 px | đặt `inside` |

- Số hộp `needs_review` trước và sau khi kiểm: `102/132` → `64/93`.
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: các xe rất nhỏ dưới cầu và vùng xa; giữ `needs_review`, ghi trong guideline và hỏi Lab Coach, không tự đoán lớp.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `2 0.758523 0.350859 0.375547 0.355844` từ `drive_008.txt`.
- Tên lớp và tọa độ điểm ảnh `xyxy`: `2=bus`; xấp xỉ `(365.3, 110.7, 605.6, 338.4)` trên ảnh 640×640.
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học? Vì cú pháp chỉ kiểm tra năm trường và miền giá trị; nó không biết box có ôm đúng xe, có bỏ sót xe hay lớp có phù hợp dấu hiệu ảnh hay không.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`.
- Mã ảnh thẩm định: `drive_008`.
- Mô tả một dự đoán trong `detect_result.jpg`: ở confidence 0,25, mô hình không sinh box nào dù ảnh có xe buýt, xe ben và ô tô rõ.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Kiểm phân bố lớp lệch (`car=70`), số ảnh quá ít và ngưỡng confidence; không kết luận ngay nhãn sai.
- Minh chứng nào có thể bác bỏ nhận định của bạn? Một nguồn nhãn độc lập nhất quán hoặc validation lớn hơn cho cùng vùng ảnh.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế? Chỉ có một ảnh validation, tập quá nhỏ và lượt chạy là chẩn đoán pipeline.

## 6. Đối chiếu nhãn

- Số hộp ghép được: `48`.
- IoU trung bình và trung vị: `0.872854` và `0.883719`.
- Mức đồng thuận lớp: `70.8333%`.
- Số hộp phía bạn không ghép được: `45`.
- Số hộp phía đối chiếu không ghép được: `2`.
- Một điểm khác biệt cụ thể: xe buýt lớn `drive_008` có IoU `0.952466`, mình gán `bus` còn reference gán `van`.
- Quy tắc hoặc hành động sửa phát sinh: xem lại ảnh gốc; thân dài và nhiều cửa sổ phù hợp `bus`, nên giữ `bus` và ghi khác biệt để Lab Coach xác nhận, không sửa chỉ để tăng đồng thuận.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng? Hai người hoặc hai bộ nhãn có thể cùng bỏ sót hoặc cùng hiểu sai một vật thể; IoU chỉ mô tả độ khớp hình học của các box đã ghép.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Minh chứng mạnh nhất là hai export từ cùng job có 93/93 box khớp, IoU tối thiểu giữa format là `0.99994466`, và mỗi box CVAT có đủ ba thuộc tính. Câu hỏi còn lại là vì sao reference gán xe buýt lớn `drive_008` thành `van`; mình đã giữ `bus` theo dấu hiệu quan sát được và ghi lại để Lab Coach xác nhận.
