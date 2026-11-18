# Báo cáo Day 08 — 2A202603047 - Đặng Thu Hương

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn run_id trong model_log.csv) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | bicycle 0.2, motorbike 0.25; person / car / bus / truck mặc định 0.3 | Chọn YOLO26m làm nguồn pre-label chính cho 6 lớp COCO. r01 cho thấy person, motorbike và bus có nhiều dự đoán rõ, confidence cao; ví dụ bus img_030 đạt 0.88, xe máy đạt 0.67–0.88. r02 cho thấy tăng conf lên 0.5 làm mất 76 box (35%), gồm 44 person và 23 motorbike. r05 cho thấy tuk-tuk img_002 có confidence 0.16 nhưng bị lọc ở ngưỡng 0.25. Vì vậy không dùng ngưỡng cao chung cho mọi lớp mà chọn ngưỡng riêng cho bicycle và motorbike. |
| three_wheeler | YOLOE-26s, prompt: tuk-tuk, cyclo, cargo tricycle | 0.15 | r03 phát hiện tuk-tuk img_002 (0.93) và xích lô img_032 (0.92), là các phương tiện YOLO26 khó gọi đúng tên. r06 phát hiện tuk-tuk ở img_004 với confidence 0.81 và 0.62, nhưng cũng có trường hợp chỉ đạt 0.07. Chọn ngưỡng 0.15 để giữ thêm phương tiện confidence thấp; box sai vẫn cần người duyệt kiểm tra và xóa. |

### Lệnh merge_prelabels.py đã dùng

```bash
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "bicycle=0.2,motorbike=0.25,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

### Phiên bản SAM được chọn

Chọn SAM 2.1 base (sam2.1_b.pt). Trong r04, pipeline YOLO26 → SAM chạy 4.26 giây/ảnh trên GTX 1650.

SAM tạo mask ôm sát khi box đầu vào đúng; ví dụ người ở img_004 và tuk-tuk xanh ở img_004. Tuy nhiên, mask phụ thuộc vào chất lượng box: box sai lớp, thiếu hoặc bị cụt sẽ truyền lỗi sang mask. Ở img_013, box car bị cụt nửa trái nên mask cũng cụt, viền răng cưa và có lỗ; tài xế/người ngồi trong xe chỉ được tạo thành một phần mask.

Tốc độ chậm khoảng 5 lần so với YOLO26 chạy riêng. Vì vậy chỉ dùng SAM cho phần thực hành tạo mask và chỉnh bằng click, không dùng để segment toàn bộ lô.

### Lỗi không sửa được chỉ bằng đổi ngưỡng

Lỗi sai lớp không thể giải quyết chỉ bằng tăng/giảm confidence. Tuk-tuk img_002 bị YOLO26 gọi thành truck; một số tuk-tuk img_004 bị gọi thành car. Đổi ngưỡng chỉ quyết định giữ hay loại box, không đổi được nhãn lớp. Cần kiểm tra ảnh và sửa class thủ công trong CVAT theo taxonomy.

### Box điểm cao mà sai hoặc điểm thấp mà đúng

- Điểm cao nhưng sai: bán tải img_002 bị gọi thành truck với confidence 0.73; SUV img_019 bị gọi thành truck với confidence 0.45 (r01).
- Điểm thấp nhưng vẫn là đối tượng thật: tuk-tuk img_002 có confidence 0.16 và bị lọc ở ngưỡng 0.25 (r05); tuk-tuk img_004 có trường hợp confidence 0.07 (r06).

### Giới hạn chọn ngưỡng

Ngưỡng được lựa chọn dựa trên tập gold chỉ gồm 4 ảnh, nên kết quả đo trên gold có thể lạc quan và chưa đại diện cho toàn bộ dataset. Model có thể hoạt động kém hơn trên ảnh mới, nhất là với phương tiện nhỏ, bị che khuất hoặc có hình dạng gần giống lớp khác.

## 2. AI error profile (nguồn: qc/review_stats.md)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 446 | 446/532 = 83.8% | 532 box máy |
| edited | 33 | 33/532 = 6.2% | 532 box máy |
| deleted | 52 | 52/532 = 9.8% | 532 box máy |
| added | 108 | 108/587 = 18.4% | 587 đối tượng sau duyệt |

### Thống kê theo lớp

Ở lớp `three_wheeler`, thống kê trong `qc/review_stats.md` ghi nhận:

| Lớp | accepted | edited | deleted | added | Tổng box máy (accepted + edited + deleted) | Tỷ lệ deleted |
|---|---:|---:|---:|---:|---:|---:|
| three_wheeler | 13 | 8 | 22 | 5 | 43 | 22/43 = 51.2% |

Như vậy, `three_wheeler` là lớp có vấn đề nổi bật: hơn một nửa số box máy của lớp này bị xóa khi duyệt. Kết quả này cũng phù hợp với các lỗi thấy trong model_log.csv: tuk-tuk bị gọi thành truck hoặc car, box lỏng/lấn sang người lái, xích lô có thể bị gộp box và phương tiện bị che khuất có thể bị bỏ sót (img_002, img_004, img_011 và img_032; r03–r06).

So với gold, precision của máy là 51/71 = 71.8% (KTC 95%: 60–81%) và recall là 51/58 = 87.9% (KTC 95%: 77–94%). Mẫu số lần lượt là 71 dự đoán máy được đối chiếu và 58 đối tượng gold trên 4 ảnh.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Kết quả bắt bẫy là 8/12 = 66.7% (KTC 95%: 39–86%), với 12 bẫy.

Các ảnh/loại bẫy được ghi nhận: extra ở img_010; loose ở img_005 và img_018; missing ở img_029. Theo thống kê loại: extra 2/3, wrong_class 3/3, loose 1/3, missing 2/3.

Kết quả cho thấy tôi chưa phát hiện được toàn bộ các trường hợp bẫy. Một nguyên nhân có thể là tôi vẫn chú ý nhiều đến box và nhãn AI đề xuất, thay vì kiểm tra độc lập toàn bộ ảnh theo checklist. Khi duyệt cần rà soát box thừa, sai lớp, box lỏng/chưa ôm sát và đối tượng bị bỏ sót.

Tỷ lệ accepted cao (446/532 = 83.8%) không đồng nghĩa QC đáng tin cậy. Bắt bẫy chỉ đạt 8/12 = 66.7%, cho thấy vẫn có lỗi mà quy trình duyệt chưa phát hiện. Không thể chỉ dựa vào tỷ lệ accepted để kết luận chất lượng nhãn tốt.

## 4. Máy có đáng dùng không?

Theo time_log.csv: gán tay 20 phút/4 ảnh = 5.0 phút/ảnh; duyệt pre-label 40 phút/24 ảnh ≈ 1.7 phút/ảnh.

Quy trình pre-label nhanh hơn về thời gian trung bình trên mỗi ảnh. Tuy nhiên, phép so sánh có thể bị thiên lệch vì tôi đã gán gold trước nên quen với cảnh và đối tượng trong dataset. Cần thử nghiệm trên nhóm ảnh mới, không quen thuộc để đo khách quan hơn.

**Kết luận: Có thể dùng AI làm công cụ pre-label để tiết kiệm thời gian, nhưng không dùng kết quả AI làm nhãn cuối cùng.** Cần người kiểm tra và sửa trước khi phát hành.

Cơ sở: accepted 446/532 = 83.8%; edited 33/532 = 6.2%; deleted 52/532 = 9.8%; added 108/587 = 18.4%. Trên 4 ảnh gold, precision là 71.8% (51/71) và recall là 87.9% (51/58).

### Ba quy tắc giảm thiên kiến tự động hóa

1. Rà soát toàn bộ ảnh theo thứ tự từ trái sang phải, từ trên xuống dưới; chủ động tìm đối tượng chưa có box, không chỉ nhìn các box AI đã tạo.
2. Kiểm tra từng box với ảnh gốc và taxonomy trước khi accepted; không dùng confidence cao làm bằng chứng duy nhất rằng box và class đúng.
3. Kiểm tra riêng vùng đông phương tiện, box chồng lấn, đối tượng bị che khuất và vật thể ở mép ảnh; ghi lại và xử lý từng trường hợp trước khi hoàn tất QC.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hóa? | Lỗi quan sát | File trong masks/ |
|---|---|---:|---|---|---|
| img_013 (tuk-tuk chụp cận, có tài xế) | SAM 2.1 base (sam2.1_b.pt) | 6 (4 dương + 2 âm), lần thử thứ 3 | Có dính lốm đốm chân/bàn chân tài xế dù có click âm; khách áo hồng ở nền sau (nhìn qua lan can) bị tô vào mask | Mask phủ thân, mái, đuôi và 2 bánh (41.8% ảnh, 1 polygon) nhưng lan sang xe máy đỏ bên phải. Lần 1 (3 click: 2 dương + 1 âm) dính tài xế và khách nền sau; lần 2 (6 click: 2 dương + 4 âm, âm đặt trong khung xe) chỉ còn mái (10.9% ảnh). | tuktuk_img013.jpg (lần 3), tuktuk_img013_v1_3click.jpg (lần 1) |
| img_004 (tuk-tuk xanh dương YOLO26 bỏ sót) | SAM 2.1 base (sam2.1_b.pt) | 3 (2 dương + 1 âm) | Không có người lái/hàng; người đứng bên phải được loại nhờ click âm | Phủ trọn mái, đầu xe, 3 bánh và ghế; không lấn sang tuk-tuk phía sau. | tuktuk_img004.jpg |
| img_032 (xích lô, xe phía sau bị che) | SAM 2.1 base (sam2.1_b.pt) | 3 (2 dương + 1 âm) | Không có người ngồi; người đứng cạnh được loại nhờ click âm | Phủ đủ đệm, khung, biển số và 2 bánh; không lấn sang xích lô bị che phía sau. | cyclo_img032.jpg |

### Đối chiếu guideline mục 4.2

Click dương giúp SAM chọn vùng cần tạo mask; click âm giúp loại vùng không thuộc đối tượng. img_004 và img_032 cho thấy click âm có thể loại người đứng cạnh.

Tuy nhiên, img_013 cho thấy click âm chưa loại hết tài xế/khách nền sau và mask vẫn lấn sang xe máy bên cạnh. Cần kiểm tra toàn bộ đường viền mask, đặt click âm đúng vùng bị dính và không chỉ đánh giá mask theo diện tích hoặc hình dáng tổng thể.

### Interactor SAM trong CVAT

Chưa có thông tin kết quả thử nghiệm Interactor SAM trong dữ liệu hiện tại nên không đưa ra kết luận đã sử dụng thành công hay gặp lỗi cụ thể.

## 6. Calibration (Part 6)

Có 3 quyết định trong calibration_decisions.csv: CAL-01 (loi_may), CAL-02 và CAL-03 (cau_hoi_mo).

**Quyết định quan trọng nhất:** CAL-01, vì làm rõ quy tắc phân biệt xe bán tải: không chở hàng gán car, đang chở hàng gán truck. Quy tắc này được cập nhật vào taxonomy.json v1.1.

CAL-02 quy định vật thể dưới 20 px trong cụm xe máy dày dùng ignore_region; CAL-03 giữ ngưỡng nhìn thấy 20% và mask xe loại người lái/khách. Các quy tắc được đưa vào guideline_v1.1_calibrated.md.

**Nguyên nhân gốc:** Có cả vấn đề guideline chưa đủ rõ ở một số trường hợp đặc biệt và giới hạn của model. Model nhầm tuk-tuk thành truck hoặc car, trong khi tình huống xe bán tải và vật thể nhỏ cần quy tắc gán nhãn thống nhất. Calibration bổ sung quy tắc cụ thể để người gán nhãn áp dụng nhất quán; lỗi nhận diện của model vẫn cần người sửa thủ công.

## 7. QC và quyết định (Part 6)

Mẫu QC gồm 8 ảnh và 152 đối tượng: 3 ca khó (img_013, img_022, img_023) và 5 ảnh ngẫu nhiên (img_003, img_006, img_014, img_017, img_035), seed 42.

Người kiểm là tôi; đây là hạn chế vì tôi cũng là người đã duyệt batch. Tôi trực tiếp xem ảnh, kiểm tra từng object_id và tự chốt error_type trong `qc_review.csv`. Tôi tự mở từng ảnh trong `work/qc/review/`, kiểm tra từng `object_id`, quyết định dòng nào là lỗi hay không lỗi, điền `error_type` và `note`, đổi reviewer thành `DangThuHuong` rồi hoàn thiện phiếu đang được chấm là `qc_review.csv`.

- Lỗi theo đối tượng: 7/152 = 4.6% (mẫu số: 152 đối tượng trong mẫu).
- Ảnh có lỗi: 4/8 = 50.0% (mẫu số: 8 ảnh trong mẫu).

### Top lỗi

| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---:|---|
| thừa | 5 | img_003.jpg, img_006.jpg, img_013.jpg |
| sai lớp | 2 | img_013.jpg, img_022.jpg |

Chi tiết trong `qc_review.csv` ghi 5 box thừa:
- img_003 ID 23.
- img_006 ID 73 và 78.
- img_013 ID 137 và 141.

Hai trường hợp sai lớp:
- img_013 ID 134.
- img_022 ID 293.

Các box còn lại trong phiếu được đánh dấu khong.

### Giới hạn mẫu nhỏ

Với 4/8 ảnh có lỗi, cận trên 95% Wilson của tỷ lệ ảnh lỗi là 78% theo qc_report_auto.md.

Mẫu chỉ có 8 ảnh nên độ bất định lớn; không thể dùng mẫu này để chứng minh tỷ lệ lỗi thật của toàn bộ dataset thấp hơn ngưỡng 2%.

### Script kiểm được / không kiểm được

Script có thể tổng hợp số lượng và trạng thái trong phiếu QC, đồng thời kiểm tra các lỗi cấu trúc mà validator được thiết kế để phát hiện.

Các nội dung cần quan sát ảnh như box có ôm sát vật thể không, hai box có thuộc cùng một đối tượng không, class có đúng ngữ nghĩa không, có đối tượng bị bỏ sót không và mask có dính nền/người/hàng hóa không vẫn cần người kiểm tra trực tiếp.

### Quyết định: REWORK — ngày 2026-11-18

Lý do: tỷ lệ ảnh có lỗi là 4/8 = 50.0%, cao hơn ngưỡng Release 2%; có 7/152 đối tượng lỗi, trong đó lỗi thừa chiếm 5/7 trường hợp. Khả năng bắt bẫy chỉ đạt 8/12 = 66.7%, thấp hơn mức 80% được dùng làm tiêu chí trong báo cáo.

Lỗi thừa lặp lại ở nhiều ảnh cho thấy cần rà soát quy trình và recipe, không chỉ sửa riêng từng box.

**Việc cần làm tiếp:**

- Sửa các box thừa và sai lớp đã xác định.
- Rà soát lại vùng đông phương tiện và box chồng lấn.
- Đối chiếu nhãn với guideline_v1.1_calibrated.md và taxonomy.json.
- QC lại trên mẫu ảnh mới, tốt nhất có người kiểm độc lập.
- Chỉ xem xét Release khi kết quả QC mới đáp ứng ngưỡng yêu cầu.

## 8. Ghi chú cá nhân

### Dùng AI để làm bài

Claude Code trong VS Code được dùng để hỗ trợ:

- Cài venv, PyTorch CUDA, tải model; đổi remote Git; điền thông tin máy vào STUDENT.md.
- Xem ảnh viz/ (pilot, không phải gold) và tổng hợp nhận xét cho model_log.csv (r01–r06). Tôi đã sửa/xác nhận và tự viết cột decision.
- Chạy threshold_sweep, review_stats, build_release, qc_sample, qc_report, validate_labels, check_submission; chạy lại SAM img_013 ba lần và bổ sung click.
- Tạo bảng so sánh pre-label với nhãn sau duyệt (work/review_diff.md, work/qc_detail.md) để tôi kiểm tra.
- Hỗ trợ soạn và chỉnh sửa REPORT.md / STUDENT.md; trình bày calibration_decisions.csv, guideline_v1.1_calibrated.md, taxonomy.json theo quyết định tôi chọn và commit Git. Tôi đọc lại, đối chiếu số liệu và chỉnh nội dung báo cáo trước khi nộp.
- Dùng ChatGPT trong cuộc trao đổi này để rà soát các điểm cần sửa và chỉnh lại REPORT.md theo số liệu, tên file và quy trình thực tế tôi cung cấp.

Không dùng AI để gán gold, duyệt batch trong CVAT hoặc tự xem ảnh để đưa ra kết luận QC. AI không tự quyết định error_type trong `qc_review.csv`; tôi trực tiếp mở ảnh, kiểm tra từng ID, đối chiếu box với đối tượng thực tế và tự ghi error_type, reviewer, note.

### Gold có bị nhiễm không?

**Không.** Tôi hoàn thành gán tay 4 ảnh gold và export lúc 10:22 ngày 17/11, trước lần chạy model đầu tiên (r01). Trong quá trình gán gold, tôi không xem pre-label, không sử dụng AI và không tham khảo kết quả model. Vì vậy, gold được tạo độc lập trước khi có kết quả dự đoán của model.

### Máy / GPU / CVAT

Windows 11, NVIDIA GTX 1650, PyTorch 2.14.1+cu126, CVAT v2.74.1 (Docker).

Tốc độ YOLO26 → SAM đo được là 4.26 giây/ảnh trong r04.

Trong lúc dọn dữ liệu, OneDrive có lúc khóa thư mục `viz` nên tôi không xóa được thư mục ngay. Tôi phải xử lý việc đồng bộ/khóa file trước rồi mới tiếp tục dọn. Ngoài ra, tôi đã cài venv, cấu hình PyTorch CUDA, tải model và chạy CVAT bằng Docker.

### Dùng cache hay tự chạy?

Không dùng cache data/cache/; tự chạy YOLO26, YOLOE và SAM 2.1 trên GPU. Các lần chạy được ghi trong work/runs.csv (r01–r06).

### Lỗi / bug script

Tôi chưa gặp và xác nhận được bug logic cụ thể trong các script đã chạy. Vấn đề thực tế tôi gặp khi làm bài là OneDrive khóa thư mục `viz` lúc dọn/xóa file; đây là vướng mắc do đồng bộ hoặc file đang bị giữ, không phải lỗi logic của script. Các lỗi box trùng, box lỏng, sai lớp và bỏ sót đối tượng là lỗi đầu ra model, tôi ghi nhận riêng chứ không quy thành bug code.

### Một điều làm khác lần sau

Lần sau tôi sẽ chuẩn bị checklist QC trước khi duyệt, kiểm tra độc lập ảnh gốc trước khi dựa vào pre-label, rà soát riêng các lớp khó và vùng đông phương tiện, đồng thời dùng một mẫu QC mới có người kiểm độc lập để đánh giá lại trước khi quyết định Release.