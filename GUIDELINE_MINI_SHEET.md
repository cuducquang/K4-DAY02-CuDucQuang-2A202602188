# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Cù Đức Quang  
**MSSV:** 2A202602188  
**Hình thức:** cá nhân  
**Mã cặp:** SOLO

## 1. Phạm vi và lớp

Chỉ gán phương tiện phân biệt được trong bốn lớp; mỗi xe một rectangle sát phần nhìn thấy. Không gán người, xe máy, xe đạp, biển báo hoặc hình phản chiếu. Nếu nguồn 640×640 quá nhỏ/mờ để phân lớp, bỏ box và ghi lý do, không đoán. Mục tiêu 40–60 là khối lượng tham khảo, không phải ngưỡng đạt.

| Mã | Lớp | Dấu hiệu quyết định |
| ---: | --- | --- |
| 0 | `car` | sedan, hatchback, SUV, taxi hoặc bán tải dùng như xe con; không có thân chở hàng rõ |
| 1 | `truck` | thùng/ben/sàn hàng hoặc thiết bị công vụ gắn trên xe rõ |
| 2 | `bus` | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế; có thể là xe khớp nối |
| 3 | `van` | thân hộp ngắn, kín, không có thùng tách rời hay thân xe buýt |

`visibility` ghi mức nhìn thấy: `clear`, `occluded`, `unclear`. `boundary` là `inside` hoặc `truncated` do **mép ảnh**, không phải do xe khác che. `review_state` là `confident` hoặc `needs_review`; lớp và ba thuộc tính là thông tin khác nhau. Box bị che chỉ ôm phần nhìn thấy, không tự vẽ phần ẩn.

## 2. Ba tình huống mơ hồ đã xét trước khi nhận nguồn đối chiếu

### A — Xe buýt hay van?

- **Vật thể:** `drive_038`, xe đỏ thân hộp tại khoảng `xyxy=(492,107,581,208)` pixel.
- **Dấu hiệu:** thân xe ngắn, một khối kín; không có thân dài và dãy cửa sổ liên tục như xe buýt ở mép trái cùng ảnh.
- **Quy tắc và quyết định:** gán `van`, không dựa riêng vào màu đỏ hoặc kích thước box. Giữ `visibility=occluded`, `boundary=inside`, `review_state=needs_review` do phần sau xe bị vật thể khác che và chi tiết nhỏ. Nếu Lab Coach có ảnh nguồn rõ hơn, kiểm lại lớp; không đổi theo phỏng đoán.

### B — Xe tải hay van/ô tô con?

- **Vật thể:** `drive_038`, xe cứu hộ trắng nửa dưới ảnh, khoảng `xyxy=(286,338,476,501)` pixel.
- **Dấu hiệu:** sau cabin là sàn và cần cẩu/thiết bị kéo xe rõ, không phải thân van kín một khối.
- **Quy tắc và quyết định:** `truck`; giữ `review_state=needs_review` cho ranh giới box quanh cần cẩu. Trường hợp `drive_008` có xe ben đỏ với thùng hàng rõ cũng theo cùng quy tắc. Nếu ảnh nguồn không cho thấy sàn/thiết bị, tạm đánh dấu xem lại và hỏi Lab Coach thay vì đoán theo kích thước.

### C — Bị che, bị cắt ở mép ảnh, hay không đủ bằng chứng?

- **Vật thể:** `drive_038`, xe con tối ở sát mép trái, khoảng `xyxy=(0,185,53,258)` pixel.
- **Dấu hiệu:** thân và kính xe còn nhìn được, đầu/đuôi kéo ra ngoài khung ảnh; phần ngoài ảnh không được suy diễn vào box.
- **Quy tắc và quyết định:** `car`, `visibility=clear`, `boundary=truncated`, `review_state=confident`. Tự kiểm phát hiện `boundary=inside` trước đó là sai; sửa trực tiếp trong CVAT và kéo cạnh trái box tới x=0. Nếu chỉ còn một mảng mờ không đủ phân lớp thì bỏ box, không gán `unclear` để hợp thức hóa một lớp đoán.

## 3. Tự kiểm tra

- [x] Rà cả bốn ảnh, đối chiếu đúng ZIP ảnh gốc bằng SHA-256.
- [x] Bỏ 39 box quá nhỏ/mờ hoặc trùng; sửa hai lỗi `boundary` có căn cứ.
- [x] Chuyển 105 track một keyframe thành rectangle riêng ảnh để không lan sang ảnh sau.
- [x] Sửa thứ tự `car, truck, bus, van` và bảo toàn lớp ngữ nghĩa của mọi box.
- [x] Mỗi box cuối cùng có đủ ba thuộc tính; hai gói xuất cùng 93 box.
- [x] Rà lại các box `needs_review`; giữ 64 box có bất định cần xác minh bằng nguồn độc lập/Lab Coach, không tự tăng mức tự tin.
- [x] Khóa gói YOLO cá nhân trước khi xem nguồn đối chiếu: SHA-256 `2f6fcb24acf8a71b03a7ef63ca4dfa5e7f527f143bbf7cf3190024c229092abc`.

**Số vật thể thực tế:** 93 trên bốn ảnh. Quyết định loại/giữ từng nhóm được ghi trong nhật ký tự kiểm tại máy local.

## 4. Phản hồi sau đối chiếu

Teaching reference `day2-reference-4img-v1` có 50 box. Đối chiếu sau khi khóa bài riêng: 48 ghép, IoU trung bình 0,872854, trung vị 0,883719, đồng thuận lớp 70,8333%, unmatched `mine=45`, `reference=2`. Với xe buýt lớn ở `drive_008`, reference dùng `van` dù hình ảnh cho thấy thân xe khách dài nhiều cửa sổ; giữ `bus` theo quy tắc quan sát được và ghi đây là khác biệt cần Lab Coach xác nhận. Không dùng IoU để tự động sửa lớp.
