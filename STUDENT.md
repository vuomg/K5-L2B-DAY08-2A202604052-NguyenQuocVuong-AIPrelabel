# Thông tin học viên

- Họ và tên: Nguyễn Quốc Vượng
- MSHV: 2A202604052
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: vuomg
- Repo bài nộp (URL): https://github.com/vuomg/K5-L2B-DAY08-2A202604052-NguyenQuocVuong-AIPrelabel
- Commit chốt bài (hash): <<>>
- Máy: Windows 11, NVIDIA GeForce RTX 4050 Laptop 6 GB (CUDA)
- CVAT (Docker): v2.81.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 406 / 4 / 23 (mẫu số: 433 box máy) = 93.8% / 0.9% / 5.3%; added: 22 (mẫu số: 432 đối tượng sau duyệt) = 5.1%
- So với gold: precision / recall của **máy** 48/61 = 78.7% / 48/59 = 81.4%
- Bắt bẫy: 10/12 = 83.3% (lần đo đầu trước khi duyệt lại: 7/12 = 58.3%)
- Thời gian: gán tay 9.7 phút/ảnh (38.7/4); duyệt có pre-label 2.3 phút/ảnh (54.7/24), working time từ CVAT analytics
- Lỗi QC: 4/5 = 80.0% theo ảnh, 8/93 = 8.6% theo đối tượng
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 4/5 = 80% > 2% và bắt bẫy lần đo đầu 7/12 = 58.3% < 80%
