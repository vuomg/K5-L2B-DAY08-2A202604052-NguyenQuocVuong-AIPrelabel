# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202604052 - Nguyễn Quốc Vượng

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc **gần camera** (không phải ở xa): từng xe hay `ignore_region`? Ngưỡng cụ thể? | Gán **từng xe** nếu tách được: thấy riêng bánh, biển số hoặc yên của xe đó **và** phần nhìn thấy ≥ 20%, box cao ≥ 20 px. Phần còn lại của khối không tách được thành từng xe → **một** `ignore_region` bao đúng phần đó. Xe rõ ở hàng đầu **luôn** có box riêng, không được gộp vào ignore. | img_006 (hàng xe đỗ bên trái: #74, #79, #73 có box; 2–3 xe phía sau #73 phải xử lý theo quy tắc này) | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_014 (CAL-01) | Người gánh hàng rong: hai thúng hàng treo trên đòn gánh, không có bánh xe | YOLOE-26s (prompt tuk-tuk / cyclo / cargo tricycle) gán 2 box `three_wheeler` (conf 0.33, 0.63) | Xoá cả 2 box; quang gánh không thuộc lớp nào → không gán, không vẽ `ignore_region`; người gánh chỉ có một box `person` | **Mục 2**, dòng `three_wheeler`, cột "Không gồm": quang gánh, xe đẩy tay, giỏ / thúng hàng không có bánh. Chỉ gán `three_wheeler` khi **thấy xe có bánh** (đếm được 3 bánh hoặc thấy rõ khung xe ba bánh, xem 4.3); không thấy bánh → không gán |
| img_024 (CAL-03) | Ảnh dày xe máy; bẫy missing lọt (một `car` bị che giữa đám xe máy) | (bẫy cài) | Vẽ thêm box `car` bị bỏ sót. Đây là lỗi của tôi (bỏ bước 0 của checklist), guideline không mơ hồ | **Mục 4.1** (quy trình duyệt, thêm điểm 8): trước khi bật lớp nhãn máy, quét ảnh và đếm vật theo lớp; ảnh > 30 vật thì quét theo 4 góc phần tư và tìm riêng vật lớn (`car` / `bus` / `truck` / `three_wheeler`) bị che giữa đám xe máy; so số đếm với số box máy |
| img_013 (CAL-04) | Bẫy loose lọt: box `three_wheeler` của tuk-tuk ở xa bị lệch / thu nhỏ | (bẫy cài) | Kéo lại box ôm sát tuk-tuk. Lỗi của tôi (chỉ liếc qua box nhỏ), mục 4.1.1 đã đủ rõ | **Mục 4.1.1** (bổ sung): box của vật nhỏ (cạnh < 100 px) phải phóng to ≥ 200% và kiểm từng cạnh; với vật ~60 px, lệch 3–4 px ở một cạnh đã dưới IoU 0.9, không chấp nhận bằng mắt thường |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở số 2 (xe máy đỗ dày gần camera: từng xe nếu tách được, phần còn lại 1 `ignore_region`); thêm loại trừ quang gánh / xe không bánh khỏi `three_wheeler`; thêm bước đếm vật trước khi xem nhãn máy (4.1.8) và bắt buộc phóng to kiểm box vật nhỏ (4.1.1). Câu hỏi mở 1, 3–7 **chưa chốt**: không gây lỗi nào trong mẫu QC, giữ nguyên quy tắc v1.0 | CAL-02 (img_006), CAL-01 (img_014, lỗi máy), CAL-03 (img_024, bẫy lọt), CAL-04 (img_013, bẫy lọt) |
