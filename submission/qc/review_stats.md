# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 532; đối tượng sau duyệt: 587; ngưỡng accepted: IoU >= 0.9
- Đã **loại 33 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 446 | 446/532 = 83.8% | 532 box máy |
| edited (sửa) | 33 | 33/532 = 6.2% | 532 box máy |
| deleted (xoá) | 52 | 52/532 = 9.8% | 532 box máy |
| added (vẽ thêm) | 108 | 108/587 = 18.4% | 587 đối tượng sau duyệt |

Trong số deleted + added, **8 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 100.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 249 | 6 | 12 | 46 |
| bicycle | 6 | 1 | 3 | 3 |
| motorbike | 124 | 16 | 8 | 31 |
| car | 49 | 2 | 3 | 18 |
| bus | 3 | 0 | 0 | 1 |
| truck | 2 | 0 | 4 | 4 |
| three_wheeler | 13 | 8 | 22 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 58 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 51 | 20 | 7 | 71.8% (KTC 95%: 60–81%) | 87.9% (KTC 95%: 77–94%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 2/3
  - wrong_class: 3/3
  - loose: 1/3
  - missing: 2/3
  - lọt: img_010.jpg (extra), img_005.jpg (loose), img_029.jpg (missing), img_018.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
