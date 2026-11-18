# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603047 - Đặng Thu Hương

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? Ngưỡng cụ thể? | Vật cao < 20 px trong cụm dày gộp vào **một** `ignore_region`; vật cao ≥ 20 px vẽ box riêng. Ngưỡng 20 px áp dụng cả cụm gần camera, không chỉ vật ở xa (bổ sung mục 4.1.6, mục 5). | img_018, img_024, img_027 | CAL-02 |
| 4 | Người ngồi trong xe ba bánh có mui che: ngưỡng "còn thấy ≥ 20%" có hợp lý không? | Giữ ngưỡng 20% (mục 4.1.2): người còn thấy ≥ 20% vẽ box `person` riêng. **Mask** xe không gồm người lái / khách; vùng chạm thuộc vật ở phía trước (mục 4.2.4). | img_013, img_022 | CAL-03 |

Câu 1, 3, 5, 6, 7 chưa chốt: <<lý do>>

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_002, img_019 | Xe bán tải (img_002), SUV (img_019) | YOLO26m gọi `truck` (0.73, 0.45) | Gán `car`; chỉ gán `truck` khi bán tải đang chở hàng | Mục 2, định nghĩa `car` / `truck`: bán tải, SUV, minivan là `car`; bán tải đang chở hàng là `truck` (CAL-01) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 2 (ngưỡng 20 px cho cụm xe máy dày gần camera → `ignore_region`), câu 4 (giữ 20%, mask xe loại người lái / khách); bán tải chở hàng là `truck`, còn lại là `car` | CAL-01 (loi_may), CAL-02, CAL-03 (cau_hoi_mo) |
