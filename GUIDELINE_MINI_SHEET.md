# Phiếu quy tắc gán nhãn — Ngày 2

Họ và tên: LẠI HOÀNG DUY<br>
MSSV: 2A202602271<br>
Hình thức: CÁ NHÂN<br>
Mã cặp: SOLO

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp: `car`, `truck`, `bus`, `van`.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do và đánh dấu `needs_review`.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt; khoang hàng tách biệt như xe tải |

Thứ tự lớp cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.
- Khi hai hộp có khác biệt, ưu tiên kiểm lại phạm vi vật thể nhìn thấy trước khi sửa theo cảm tính.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `visibility` | `clear`, `occluded`, `unclear` | mức bằng chứng nhìn thấy |
| `boundary` | `inside`, `truncated` | vật thể có bị mép ảnh cắt hay không |
| `review_state` | `confident`, `needs_review` | trạng thái cần xem lại |

YOLO không lưu ba thuộc tính này; vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

### Tình huống A — xe buýt hay xe van?

- Ảnh/vật thể: `drive_022`, xe thân lớn ở tiền cảnh.
- Dấu hiệu nhìn thấy: thân xe dài, nhiều cửa sổ, hình dạng xe buýt rõ.
- Quy tắc áp dụng: xe khách dài, nhiều cửa sổ → `bus`; không dùng `van` cho thân xe buýt.
- Quyết định: `bus`.
- Nếu vẫn thiếu bằng chứng: đánh dấu `needs_review`, ghi lý do và xin Lab Coach xác nhận; không đoán.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh/vật thể: `drive_038`, xe cứu hộ ở khu vực giữa phía dưới.
- Dấu hiệu nhìn thấy: cabin và thiết bị công vụ/khoang phía sau rõ.
- Quy tắc áp dụng: có sàn hàng hoặc thiết bị công vụ rõ → `truck`; không dùng `van` cho khoang hàng tách biệt như xe tải.
- Quyết định: `truck`.
- Nếu vẫn thiếu bằng chứng: phóng ảnh 100%, kiểm dấu hiệu cấu trúc; nếu vẫn không đủ thì `needs_review` và xin hỗ trợ.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh/vật thể: `drive_008`, các xe ở sát mép ảnh/phía bên phải và các xe bị phương tiện khác che.
- Dấu hiệu nhìn thấy khi phóng 100%: một phần xe có thể bị vật thể khác che; một số xe nằm sát hoặc chạm mép khung ảnh.
- Giá trị `visibility`: `occluded` nếu xe bị phương tiện khác che nhưng vẫn có đủ bằng chứng; `unclear` nếu không đủ bằng chứng để quyết định.
- Giá trị `boundary`: `truncated` nếu mép ảnh thực sự cắt qua vật thể; nếu xe nằm hoàn toàn trong khung → `inside`.
- Trạng thái `review_state`: `confident` nếu có đủ bằng chứng; `needs_review` nếu không chắc.
- Lý do: ba thuộc tính mô tả ba khía cạnh khác nhau: mức nhìn thấy, quan hệ với mép ảnh và độ chắc chắn của quyết định.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà bộ bốn ảnh theo output notebook.
- [x] Đã kiểm vật thể theo số lượng 59.
- [x] Đã kiểm lớp và hình học qua gói YOLO.
- [x] Gói CVAT gốc có đủ ba thuộc tính cho cả 59 hộp.
- [ ] Đã xử lý mọi hộp `needs_review`
- [x] Đã hoàn thành ba tình huống quy tắc trong phiếu này.
- [x] Bài riêng được kiểm trước khi nhận bộ tham chiếu.
- [x] Nguồn đối chiếu là bộ nhãn do Lab Coach cấp.
- [x] Số vật thể thực tế: 59; khoảng 40–60 chỉ là mục tiêu khối lượng, không phải điểm cắt.
