# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603047 - Đặng Thu Hương

> **Chỉ ghi phần thay đổi so với v1.0**, không sao chép lại toàn bộ guideline. Bản bổ sung này chốt các câu hỏi đã có quyết định calibration và thêm quy tắc rút ra từ lỗi của model trong quá trình gán nhãn.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? Ngưỡng cụ thể? | Vật thể có chiều cao **dưới 20 px** trong cụm dày được gộp vào **một** `ignore_region`; vật thể cao **từ 20 px trở lên** được vẽ box riêng. Ngưỡng 20 px áp dụng cả với cụm xe máy gần camera, không chỉ vật thể ở xa. Bổ sung vào mục 4.1.6 và mục 5. | img_018, img_024, img_027 | CAL-02 |
| 4 | Người ngồi trong xe ba bánh có mui che: ngưỡng "còn thấy ≥ 20%" có hợp lý không? | Giữ ngưỡng nhìn thấy **20%** theo mục 4.1.2: người còn nhìn thấy từ 20% trở lên thì vẽ box `person` riêng. **Mask của xe không bao gồm người lái hoặc hành khách**; phần ranh giới tiếp giáp được xử lý theo vật thể ở phía trước, bổ sung vào mục 4.2.4. | img_013, img_022 | CAL-03 |

**Các câu hỏi chưa chốt:**

- **Câu 1 – Xe đạp điện là `bicycle` hay `motorbike`:** Chưa có tiêu chí phân loại cụ thể được thống nhất và chưa có ca đại diện được đưa vào calibration để kiểm tra cách áp dụng. Nếu chỉ dựa vào hình dáng bên ngoài, người gán nhãn có thể phân loại khác nhau. Cần bổ sung ảnh ví dụ và thống nhất tiêu chí trước khi cập nhật taxonomy.
- **Câu 3 – Hàng nhô ra có tính vào box không:** Chưa có quyết định thống nhất về việc box chỉ bao quanh thân phương tiện hay bao gồm cả phần hàng hóa nhô ra. Cần làm rõ phạm vi đối tượng cần gán nhãn và cách xử lý hàng hóa trước khi đưa ra quy tắc chung.
- **Câu 5 – Kính xe có thuộc mask không:** Chưa có ví dụ calibration và kết luận cụ thể về việc phần kính trong suốt có được tính vào mask của phương tiện hay không. Đây là vấn đề về ranh giới mask, cần thống nhất cách xử lý giữa người gán nhãn trước khi chốt.
- **Câu 6 – Box chồng nhau trong ảnh đêm:** Chưa có đủ ca ảnh đêm được kiểm tra và thống nhất để phân biệt trường hợp hai phương tiện riêng biệt với trường hợp box chồng lấn do ảnh tối, mờ hoặc che khuất. Cần có ảnh đại diện và quyết định chung trước khi bổ sung quy tắc.
- **Câu 7 – Ngưỡng confidence để chấp nhận box máy:** Không chốt một ngưỡng confidence chung trong label contract vì confidence phụ thuộc vào model và từng lớp. Thử nghiệm cho thấy tăng ngưỡng lên 0.5 làm mất nhiều đối tượng thật; riêng `three_wheeler` có trường hợp đúng nhưng confidence rất thấp. Ngưỡng confidence được giữ trong model recipe để người duyệt tham khảo, không thay thế việc kiểm tra box và class bằng ảnh gốc.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt và box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của tôi | Quy tắc mới (bổ sung vào mục nào của v1.0) |
|---|---|---|---|---|
| img_002, img_019 | Xe bán tải ở img_002 và SUV ở img_019 | YOLO26m dự đoán cả hai là `truck`, confidence lần lượt 0.73 và 0.45. | Gán `car`; chỉ gán `truck` khi xe bán tải đang chở hàng. | **Mục 2 – Định nghĩa `car` / `truck`:** Bán tải, SUV và minivan được gán `car`; xe bán tải đang chở hàng được gán `truck`. (CAL-01) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản guideline do giảng viên cung cấp. | — |
| v1.1 | Chốt câu 2: vật thể dưới 20 px trong cụm xe máy dày được gộp vào một `ignore_region`, vật thể từ 20 px trở lên vẽ box riêng. Chốt câu 4: giữ ngưỡng nhìn thấy 20%, mask xe không bao gồm người lái và hành khách. Bổ sung quy tắc phân loại xe bán tải: đang chở hàng là `truck`, trường hợp còn lại là `car`. Các câu 1, 3, 5, 6, 7 tiếp tục để mở, chưa đưa thành quy tắc chính thức. | CAL-01 (`loi_may`), CAL-02 và CAL-03 (`cau_hoi_mo`). |