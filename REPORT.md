# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** LẠI HOÀNG DUY<br>
**MSSV:** 2A202602271<br>
**Hình thức:** CÁ NHÂN<br>
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: `drive_022`, `drive_033`, `drive_038`, `drive_008`
- Số vật thể thực tế: 59
- Mã SHA-256 của gói YOLO của bạn: `6057e26a7d3b0fb237ba45fca1009e746e2fb10b79782a14c81a78585be67a9f`
- Mã SHA-256 của gói CVAT gốc của bạn: `d3e1ec89fc1c80797d15da2a632f35b76d0dc6bc2a0479aef51295aa7cc778e7`
- Nguồn đối chiếu: bộ nhãn đối chiếu do Lab Coach cấp
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Thời điểm nhận bộ tham chiếu: 15:53 14/09/2026

Bài vẫn độc lập trước khi đối chiếu vì notebook thực hiện kiểm gói YOLO và gói CVAT gốc của chính mình trước, sau đó mới chọn nguồn đối chiếu và tải bộ nhãn của Lab Coach.

## 2. Quyết định phân lớp

Bộ lớp cố định gồm `car`, `truck`, `bus`, `van`, theo thứ tự YOLO `0 car, 1 truck, 2 bus, 3 van`.

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_022` — xe buýt lớn ở tiền cảnh | `bus` | Thân xe dài, nhiều cửa sổ, dạng xe buýt rõ | Thân xe khách dài, nhiều cửa sổ → `bus` |
| `drive_038` — xe cứu hộ ở khu vực giữa phía dưới | `truck` | Có thiết bị/khoang công vụ rõ ràng phía sau cabin | Có sàn hàng hoặc thiết bị công vụ rõ ràng → `truck` |
| `drive_022` — xe thân hộp nhỏ màu trắng | `van` | Thân xe nhỏ, kín, dạng hộp | Thân hộp nhỏ, kín → `van` |

Ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau: một xe vẫn có lớp `car` nhưng thuộc tính `visibility` có thể là `clear` hoặc `occluded`; `boundary` có thể là `inside` hoặc `truncated`. Lớp mô tả **xe gì**, còn thuộc tính mô tả mức nhìn thấy, quan hệ với mép ảnh và trạng thái xem lại.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| Hộp quanh xe cứu hộ ở `drive_038` có chênh nhẹ giữa hộp đỏ của bài mình và hộp xanh của bộ đối chiếu | hình học | Quan sát ảnh phủ đối chiếu; hai hộp cùng bao quanh vật thể nhưng kích thước/vị trí không hoàn toàn trùng | Rà lại hộp theo phần vật thể nhìn thấy; không ước lượng phần bị che. Nếu hộp chưa sát vật thể thì sửa trực tiếp trong CVAT |

- Số hộp `needs_review` trước và sau khi kiểm: notebook không ghi lại số lượng giá trị `needs_review` trước/sau; chỉ xác nhận cả 59 hộp đều có thuộc tính `review_state`. Không tự suy đoán con số.
- Một quyết định chưa đủ bằng chứng và cách xin hỗ trợ: khi phương tiện quá nhỏ, mờ hoặc phần đặc trưng quyết định lớp bị che đến mức không thể phân lớp có căn cứ, không đoán. Đánh dấu `review_state=needs_review`, ghi lý do và xin Lab Coach xác nhận quy tắc.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `2 0.380828 0.726664 0.434969 0.349641`
- Tên lớp và tọa độ điểm ảnh `xyxy`: lớp `2` là `bus`; `xyxy ≈ [104.5, 353.2, 382.9, 577.0]` trên ảnh `640×640`.
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học? Vì YOLO chỉ kiểm tra cấu trúc và các giá trị tọa độ; một dòng có đủ 5 số vẫn có thể dùng sai `class_id`, bao hộp quá rộng/hẹp, bao nhiều vật thể, bỏ sót vật thể hoặc ước lượng phần bị che. Do đó đúng định dạng không đồng nghĩa đúng nội dung.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định: `drive_008`
- Cấu hình: YOLO11n, Ultralytics `8.4.145`, `imgsz=640`, `batch=4`, `freeze=10`, `seed=42`, GPU Tesla T4. Đặt `epochs=8` nhưng early stopping dừng sau **4 epoch**, với kết quả tốt nhất ở epoch 1.
- Mô tả một dự đoán trong `detect_result.jpg`: ảnh thẩm định `drive_008` được chạy với `conf=0.25`; ảnh kết quả được notebook hiển thị không cho thấy các hộp dự đoán rõ ràng ở ngưỡng này.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Trước hết cần kiểm lại dữ liệu huấn luyện vì chỉ có **3 ảnh train với 40 hộp** và 1 ảnh validation với 19 hộp. Ngoài ra cần kiểm sự đa dạng kích thước, góc nhìn, mức che khuất và độ cân bằng giữa bốn lớp. Kết quả yếu không đủ để kết luận riêng rằng quy tắc gán nhãn sai.
- Minh chứng có thể bác bỏ nhận định trên: chạy lại trên tập ảnh lớn hơn, có train/validation/test độc lập và giữ cố định quy trình; nếu mô hình hoạt động tốt trên dữ liệu mới thì nhận định “dữ liệu quá ít/không đại diện” sẽ được xem xét lại.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế? Vì đây chỉ là bài huấn luyện chẩn đoán với 4 ảnh, trong đó 3 ảnh dùng train và 1 ảnh dùng validation. Tập quá nhỏ, không đại diện cho dữ liệu thực tế và không có test set độc lập. Chính notebook cũng ghi rõ mục đích là phản hồi/tìm lỗi dữ liệu, không phải benchmark sản xuất.

## 6. Đối chiếu nhãn

- Số hộp ghép được: **45**
- IoU trung bình và trung vị: **0.857294** và **0.870866**
- Mức đồng thuận lớp: **0.688889 = 68.89%**
- Số hộp phía bạn không ghép được: **14**
- Số hộp phía đối chiếu không ghép được: **5**
- Cách ghép: ghép tối ưu theo **IoU hình học, không dùng lớp khi ghép**; `IoU floor = 0.01` chỉ là tham số ghép kỹ thuật, **không phải ngưỡng đạt**.
- Một điểm khác biệt cụ thể: ở `drive_038`, hộp quanh xe cứu hộ ở khu vực giữa phía dưới có hộp đỏ và xanh chồng lấn lớn nhưng vẫn lệch nhẹ về vị trí/kích thước. Đây là dấu hiệu cần rà lại quy tắc “vẽ sát phần vật thể nhìn thấy”.
- Quy tắc hoặc hành động sửa phát sinh: kiểm lại hộp trong CVAT theo phạm vi vật thể thực sự nhìn thấy; nếu phần bị che không nhìn thấy thì không kéo hộp bao gồm phần ước lượng. Notebook không ghi nhận một lần sửa cụ thể đã hoàn tất, nên không khẳng định rằng hộp này đã được sửa.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng? Vì hai bộ nhãn có thể cùng mắc một lỗi, hoặc cùng áp dụng một cách hiểu chưa đúng quy tắc. Đồng thuận chỉ cho biết mức tái lập giữa hai nguồn; nó không thay thế quy tắc, kiểm tra bằng chứng trực quan hoặc một chuẩn tham chiếu đáng tin cậy.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [ ] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình — cần kiểm tra lại nội dung repository sau khi upload.
- [ ] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập — cần kiểm tra lại repository trước khi nộp.

**Minh chứng mạnh nhất trong bài:** hai định dạng xuất của cùng một bộ nhãn cá nhân có `object_count=59`, cùng lớp và cùng số hộp; kiểm tra chéo định dạng cho `same_annotation_state=true`, 59 hộp ghép được với IoU nhỏ nhất khoảng `0.999951`, cho thấy gói YOLO và CVAT gốc của mình nhất quán. Ngoài ra, đối chiếu độc lập với bộ tham chiếu cho 45 hộp ghép, IoU trung bình `0.857294`, trung vị `0.870866` và đồng thuận lớp `68.89%`.
