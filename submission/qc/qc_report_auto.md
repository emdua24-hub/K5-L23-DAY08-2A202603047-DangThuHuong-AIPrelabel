# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 152 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 7/152 = 4.6% | 152 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 4/8 = 50.0% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 446/532 = 83.8% / 33/532 = 6.2% / 52/532 = 9.8% | 532 box máy |
| Máy: added | 108/587 = 18.4% | 587 đối tượng sau duyệt |
| Bắt lỗi bẫy | 8/12 = 66.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 5 | img_003.jpg, img_006.jpg, img_013.jpg |
| sai_lop | 2 | img_013.jpg, img_022.jpg |

**Lỗi hệ thống**: 'thua' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 4 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **78%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 50.0% > 2%
- lỗi hệ thống: 'thua' chiếm 5/7 lỗi
- bắt bẫy 67% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
