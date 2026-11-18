# Thông tin học viên

- Họ và tên: Đặng Thu Hương
- MSHV: 2A202603047
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: emdua24-hub
- Repo bài nộp (URL): https://github.com/emdua24-hub/K5-L23-DAY08-2A202603047-DangThuHuong-AIPrelabel
- Commit chốt bài (hash): <<>>
- Máy: Windows 11, GPU NVIDIA GeForce GTX 1650 (CUDA 12.6, torch 2.14.1+cu126)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 446/532 = 83.8% / 33/532 = 6.2% / 52/532 = 9.8% (mẫu số: 532 box máy); added: 108/587 = 18.4% (mẫu số: 587 đối tượng)
- So với gold: precision / recall của **máy** 51/71 = 71.8% / 51/58 = 87.9% (4 ảnh gold, 58 đối tượng)
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 5.0 phút/ảnh (20 phút / 4 ảnh); duyệt có pre-label ≈ 1.7 phút/ảnh (40 phút / 24 ảnh)
- Lỗi QC: 50.0% theo ảnh (4/8), 4.6% theo đối tượng (7/152)
- **Quyết định:** Rework — lý do một dòng: Rework do cần rà soát và chuẩn hóa bounding box, loại bỏ nhãn trùng, sửa nhãn sai lớp và xác minh các đối tượng nhỏ hoặc bị khuất để đảm bảo độ chính xác của dataset.