# Quét ngưỡng theo lớp

Mục tiêu recall: 90%. Ảnh dùng để quét: 4.

| Lớp | Ngưỡng gợi ý | Lý do |
|---|---|---|
| person | 0.15 | recall 93% (>= 90%), precision 64% (27 vật gold) |
| bicycle | 0.2 | recall 100% (>= 90%), precision 86% (6 vật gold) |
| motorbike | 0.5 | recall 90% (>= 90%), precision 90% (10 vật gold) |
| car | (mặc định) | không ngưỡng nào <= 0.5 đạt recall 90% (tối đa 88%, 8 vật): dùng ngưỡng mặc định, nói rõ trong báo cáo |
| bus | (mặc định) | chỉ 1 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| truck | (mặc định) | chỉ 1 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| three_wheeler | 0.15 | recall 100% (>= 90%), precision 67% (6 vật gold) |

Dán vào lệnh merge_prelabels (lớp không có trong chuỗi dùng `--conf` mặc định 0.3): `--thr "person=0.15,bicycle=0.2,motorbike=0.5,three_wheeler=0.15"`

Lưu ý: bạn chọn ngưỡng trên chính nhãn gold, nên precision / recall của máy trên gold ở ngưỡng đã chọn là con số **lạc quan** (chọn và chấm trên cùng dữ liệu); chỉ có vài ảnh -> ngưỡng là ước lượng thô; lớp có ít vật (xem n_gold) thì kết luận yếu.
