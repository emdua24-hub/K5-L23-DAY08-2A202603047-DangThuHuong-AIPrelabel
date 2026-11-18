# Báo cáo Day 08 — 2A202603047 - Đặng Thu Hương

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | bicycle 0.2, motorbike 0.25; person / car / bus / truck mặc định 0.3 (`qc/threshold_sweep.md`) | <<>> |
| three_wheeler | YOLOE-26s, prompt: tuk-tuk, cyclo, cargo tricycle | 0.15 (`qc/threshold_sweep.md`) | <<>> |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  ```
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "bicycle=0.2,motorbike=0.25,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 b (`sam2.1_b.pt`); tốc độ đo được: box YOLO26 → SAM 4.26 s/ảnh trên GTX 1650 (r04). Lý do: <<chất lượng mask, tốc độ, RAM, quyền truy cập>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? <<>>
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): <<>>
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: <<>>

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 446 | 446/532 = 83.8% | 532 box máy |
| edited | 33 | 33/532 = 6.2% | 532 box máy |
| deleted | 52 | 52/532 = 9.8% | 532 box máy |
| added | 108 | 108/587 = 18.4% | 587 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** 51/71 = 71.8% (KTC 95%: 60–81%) / 51/58 = 87.9% (KTC 95%: 77–94%) (mẫu số: 58 đối tượng gold, 4 ảnh)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 8/12 = 66.7% (KTC 95%: 39–86%); loại nào lọt: extra img_010, loose img_005, missing img_029, loose img_018 (extra 2/3, wrong_class 3/3, loose 1/3, missing 2/3); vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 20 phút / 4 ảnh = 5.0 phút/ảnh vs duyệt 40 phút / 24 ảnh ≈ 1.7 phút/ảnh. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk chụp cận, có tài xế) | SAM 2.1 b (`sam2.1_b.pt`) | 6 (4 dương + 2 âm), lần thử thứ 3 | Có: dính lốm đốm chân / bàn chân tài xế dù có click âm; khách áo hồng ở nền sau (nhìn qua lan can) bị tô vào mask | Mask phủ đủ thân, mái, đuôi, 2 bánh (41.8% ảnh, 1 polygon) nhưng lan sang xe máy đỏ bên phải. Lần 1 (3 click: 2 dương + 1 âm) dính tài xế + khách nền sau; lần 2 (6 click: 2 dương + 4 âm, âm đặt trong khung xe) chỉ còn mái (10.9% ảnh) | `tuktuk_img013.jpg` (lần 3), `tuktuk_img013_v1_3click.jpg` (lần 1) |
| img_004 (tuk-tuk xanh dương YOLO26 bỏ sót) | SAM 2.1 b (`sam2.1_b.pt`) | 3 (2 dương + 1 âm) | Không có người lái / hàng; người đứng bên phải được loại nhờ click âm | Phủ trọn mái, đầu xe, 3 bánh, ghế; không lấn sang tuk-tuk phía sau | `tuktuk_img004.jpg` |
| img_032 (xích lô, xe phía sau bị che) | SAM 2.1 b (`sam2.1_b.pt`) | 3 (2 dương + 1 âm) | Không có người ngồi; người đứng cạnh được loại nhờ click âm | Phủ đủ đệm, khung, biển số, 2 bánh; không lấn sang xích lô bị che phía sau | `cyclo_img032.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh, 152 đối tượng (3 ca khó: img_013, img_022, img_023 + 5 ngẫu nhiên: img_003, img_006, img_014, img_017, img_035, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 7/152 = 4.6% · theo **ảnh**: 4/8 = 50.0% (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): cận trên 95% (Wilson) của tỉ lệ ảnh lỗi = 78% (4/8). <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, NVIDIA GTX 1650 (torch 2.14.1+cu126), CVAT v2.74.1 (Docker). <<vướng mắc>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
