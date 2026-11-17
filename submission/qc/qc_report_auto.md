# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 93 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 8/93 = 8.6% | 93 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 4/5 = 80.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 406/433 = 93.8% / 4/433 = 0.9% / 23/433 = 5.3% | 433 box máy |
| Máy: added | 22/432 = 5.1% | 432 đối tượng sau duyệt |
| Bắt lỗi bẫy | 10/12 = 83.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 3 | img_003.jpg, img_006.jpg |
| sai_vi_tri_kich_thuoc | 3 | img_031.jpg, img_035.jpg |
| thua | 2 | img_003.jpg, img_035.jpg |

## Giới hạn thống kê
- 4 ảnh lỗi trong 5 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **96%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 80.0% > 2%

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
