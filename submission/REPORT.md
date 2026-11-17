# Báo cáo Day 08 — 2A202604052 - Nguyễn Quốc Vượng

> Một file cho cả bài. Mọi con số kèm **mẫu số** và nguồn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng khi merge | Vì sao (`model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (r05, `--split all --conf 0.05`) | person 0.35, motorbike 0.5, car 0.35; còn lại `--conf 0.3` | COCO đủ tốt cho 6 lớp: r01 (conf 0.25) bắt cả vật nhỏ ở xa (img_007), r02 (conf 0.5) mất chúng (215 → 140 box) → chạy conf thấp rồi lọc theo lớp. Sau duyệt: person giữ 246/249, motorbike 88/90. |
| three_wheeler | YOLOE-26s, prompt tuk-tuk / cyclo / cargo tricycle (r06) | `--conf 0.3` | YOLO26 không có lớp xe ba bánh: tuk-tuk → `truck` 0.77 (img_011), `car` 0.34–0.41 (img_004, img_013). YOLOE bắt đúng tuk-tuk img_002 (0.93), xích lô img_032 (0.92). |

- Lệnh merge (chạy lại ra đúng 510 box, khớp `work/prelabel/merged_review/pre.coco.json`):
  ```bash
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json \
      --thr "person=0.35,motorbike=0.5,car=0.35" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
- **Sweep gợi ý** (`threshold_sweep.md`, recall 90%, 4 ảnh gold): `person=0.15,bicycle=0.2,motorbike=0.5,three_wheeler=0.15`; car / bus / truck không chọn được. **`--thr` đã dùng khác gợi ý** (chỉ motorbike trùng). Vì sao: ở 0.15 precision person 64%, three_wheeler 67% (1/3 box phải xoá), ảnh đông như img_030 (> 50 người) thì xoá box thừa tốn hơn vẽ thêm vật sót; gợi ý lại chọn trên chính 4 ảnh gold (27 người, 6 xe ba bánh) nên chỉ là ước lượng thô. Car không có gợi ý → 0.35, gần mặc định. Đánh đổi: added 22/432 = 5.1%, deleted 23/433 = 5.3%.
- **SAM**: SAM 2.1 (`sam2.1_b.pt`, r04: 2 ảnh, 16 mask, 3.66 s/ảnh), thêm SAM 1 (`mobile_sam.pt`) trên img_013 để so. Lý do: vừa GPU 6 GB, tải trọng số không cần xin quyền như SAM 3. Chất lượng không bản nào thắng hẳn (mục 5).
- **Không sửa được bằng ngưỡng**: sai lớp. Tuk-tuk → `truck` 0.77 (img_011) vẫn còn ở conf 0.5 (r02). Sửa: YOLOE riêng cho `three_wheeler`, merge IoU 0.6 để thay box `car` / `truck` trùng chỗ; còn sót thì sửa tay (10 ca đổi lớp). Quang gánh → `three_wheeler` 0.63 (img_014): thêm quy tắc CAL-01, xoá tay.
- **Điểm cao mà sai**: `truck` 0.77 trên tuk-tuk (img_011), `three_wheeler` 0.63 trên quang gánh (img_014), 0.61 trên xe tải xa (img_003). **Điểm thấp mà đúng**: tuk-tuk 0.28 (img_011), xích lô 0.27 (img_032), 0.38 (img_035).
- **Giới hạn**: ngưỡng chọn và chấm trên cùng 59 vật gold → P/R 78.7% / 81.4% là lạc quan; bus, truck mỗi lớp 1 vật, three_wheeler 6 vật → ngưỡng theo lớp gần như không có căn cứ.

## 2. AI error profile (từ `qc/review_stats.md`)

24 ảnh batch; đã loại 24 đối tượng quanh bẫy.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 406 | 406/433 = 93.8% | 433 box máy |
| edited | 4 | 4/433 = 0.9% | 433 box máy |
| deleted | 23 | 23/433 = 5.3% | 433 box máy |
| added | 22 | 22/432 = 5.1% | 432 đối tượng sau duyệt |

Trong deleted + added có 10 ca **đổi lớp**; vật máy sót thật ≈ 22 − 10 = 12.

| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 246 | 0 | 3 | 4 |
| bicycle | 6 | 0 | 3 | 2 |
| motorbike | 88 | 1 | 1 | 5 |
| car | 44 | 0 | 6 | 3 |
| bus | 2 | 0 | 1 | 0 |
| truck | 4 | 1 | 2 | 3 |
| three_wheeler | 16 | 2 | 7 | 5 |

- **Lớp tệ nhất**: `three_wheeler`, giữ 16/25 = 64%, xoá 7/25 = 28%, sót 5. YOLOE nhầm quang gánh (img_014), xe tải xa (img_003), tách 1 xích lô thành 2 box (img_035). `bicycle` cũng yếu: giữ 6/9 = 67%. Ngược lại person 246/249 = 98.8%, motorbike 88/90 = 97.8%.
- **So với gold**: precision 48/61 = 78.7% (KTC 95%: 67–87%), recall 48/59 = 81.4% (KTC 95%: 70–89%); TP 48, FP 13, FN 11, 4 ảnh.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **10/12 = 83.3%** (KTC 95%: 55–95%); loose 2/3, missing 2/3, wrong_class 3/3, extra 3/3. Lọt: `img_013` (loose), `img_024` (missing).
- **Lần đo đầu: 7/12 = 58.3%.** Vòng duyệt lại làm **sau khi** đã xem output `review_stats.py` lần đầu, trong đó có danh sách bẫy lọt (ảnh + loại). Vì vậy 10/12 chỉ đo số bẫy đã sửa được khi biết chỗ, không đo khả năng tự phát hiện; con số độc lập là **7/12 < 80%**. Nó cũng không nói gì về lỗi máy không cài bẫy (QC mục 7 cho thấy vẫn còn).
- **Vì sao lọt**: duyệt lại vẫn không tìm thấy. img_024 dày xe máy, mắt dò theo box có sẵn nên chiếc `car` không có box bị lẫn: thiếu **bước 0** (đếm vật trước khi bật nhãn máy). img_013: box tuk-tuk xa ~60 px, lệch vài px nên nhìn không phóng to vẫn "có vẻ đúng": **bước 4** làm chưa đủ.
- **Accepted cao, bắt bẫy thấp**: accepted 406/433 = 93.8% nhưng bắt bẫy lần đầu 7/12 = 58.3% → giữ box máy quá dễ (thiên kiến tự động hóa); accepted cao một phần do người duyệt ít nghi ngờ, không chỉ do máy tốt.

## 4. Máy có đáng dùng không?

- **Thời gian** (`time_log.csv`, CVAT analytics): gán tay 38.7 phút / 4 ảnh = 9.7 phút/ảnh; duyệt 54.7 phút / 24 ảnh = 2.3 phút/ảnh (gồm vòng duyệt lại). Thiên lệch: thời gian gán tay gồm cả làm quen CVAT và guideline; hai tập ảnh khác mật độ; duyệt nhanh một phần vì dễ dãi (mục 3) → nhanh ~4 lần là cận trên.
- **Kết luận**: **có, cho person / motorbike / car** (accepted 98.8% / 97.8% / 44/50 = 88%); **không tin máy với `three_wheeler` (64%) và `bicycle` (67%)**, phải duyệt từng box.
- **Ba quy tắc giảm thiên kiến**:
  1. **Đếm trước, xem sau**: ẩn nhãn máy, đếm vật theo lớp, lệch với số box máy thì phải tìm ra. Kiểm tra: missing 3/3 ở lần đo đầu.
  2. **Phóng to box ít tin** (≥ 200%): mọi box `three_wheeler` / `bicycle`, conf < 0.5, hoặc cạnh < 100 px; 2 box trên cùng một xe thì gộp. Kiểm tra: loose 3/3; QC không còn lỗi `thua` / `sai_vi_tri_kich_thuoc` ở 2 lớp này.
  3. **Chỉ báo số đo lần đầu**: chạy `review_stats.py` một lần sau khi duyệt xong; sửa sau đó ghi riêng "sau khi biết đáp án". Kiểm tra: báo cáo có đúng một số bắt bẫy lần đầu.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM | Số click | Gồm người lái / hàng? | Lỗi quan sát | File |
|---|---|---|---|---|---|
| img_004 | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm trên tuk-tuk xanh bên cạnh) | Không có người / hàng; gồm mui | Bám biên tốt, không lấn xe bên cạnh; biên dưới hơi răng cưa | `tuktuk_img004_sam2.jpg` |
| img_013 | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương: mui, thân; 1 âm trên người lái) | **Có**: lan vào người lái dù có click âm | Sai guideline (người là vật riêng) | `tuktuk_img013_sam2.jpg` |
| img_013 | SAM 1 (`mobile_sam.pt`) | 3 (cùng vị trí) | Không | Thủng ở sàn / bậc lên xuống, gầm xe | `tuktuk_img013_sam1.jpg` |
| img_017 | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm trên mui) | Không có người / hàng; gồm mui, ghế | Tách đúng khỏi tuk-tuk xanh; vùng nhìn xuyên khoang hở bị tô vào (4.2.2: phải bỏ) | `tuktuk_img017_sam2.jpg` |

- **Đối chiếu 4.2**: giúp: 4.2.4 (vật chạm nhau, img_004 / img_017), 4.2.2 (bỏ lỗ lớn). Thiếu: chưa nói **người ngồi trong xe ba bánh** có thuộc mask xe không (img_013); câu hỏi 5 (kính chắn gió) chưa chốt.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thử; mask làm offline bằng `scripts/sam2_points.py`.

## 6. Calibration (Part 6)

- **4 quyết định** (`calibration_decisions.csv`: 1 `loi_may`, 1 `cau_hoi_mo`, 2 `bay_lot`). Quan trọng nhất: **CAL-01**, quang gánh không phải `three_wheeler`. Đây là lỗi lặp của YOLOE (xoá 7/25) và 2/3 lỗi QC lần đầu.
- **Quy tắc mới trong v1.1**: (1) `three_wheeler` phải thấy bánh, không gồm quang gánh / xe đẩy / thúng (CAL-01); (2) xe máy đỗ dày gần camera: từng xe nếu tách được, phần còn lại 1 `ignore_region` (CAL-02, img_006); (3) đếm vật trước khi bật nhãn máy (CAL-03); (4) phóng to kiểm box vật < 100 px (CAL-04).
- **Nguyên nhân gốc**: CAL-01 do máy + guideline mơ hồ; CAL-02 do guideline mơ hồ; CAL-03 / 04 do người duyệt chưa theo quy trình (thêm quy tắc kiểm tra, không đổi định nghĩa lớp).

## 7. QC và quyết định (Part 6)

- **Mẫu**: 5 ảnh ngẫu nhiên (seed 42, 0 ảnh `can_review`): img_003, img_006, img_014, img_031, img_035; 93 đối tượng; người kiểm: chính người duyệt (**hạn chế**).
- **Lỗi**: theo đối tượng **8/93 = 8.6%**, theo ảnh **4/5 = 80.0%** (ngưỡng ≤ 2%); chỉ img_014 sạch.
- Lần QC trước vòng duyệt lại: 3/94 đối tượng = 3.2%, 2/5 ảnh = 40.0% (quang gánh img_014 × 2, một box `three_wheeler` img_003).
- **Top lỗi**: **bỏ sót** 3/8 (img_003: xe đạp của #4, xe máy sau #16; img_006: khối xe đỗ sau #73); **sai vị trí / kích thước** 3/8 (img_031: #380 thiếu mâm hàng, #382 thiếu chân; img_035: #444 thiếu mui); **thừa** 2/8 (box `three_wheeler` trùng: #12 img_003, #443 img_035). Không có sai lớp.
- **Giới hạn mẫu nhỏ**: 4/5 ảnh lỗi → KTC 95% (Wilson) ≈ 38%–96%. 5 ảnh không chứng minh được ngưỡng 2% (0/5 vẫn có cận trên 43%), nhưng cận dưới 38% ≫ 2% nên "không đạt" là chắc. Mỗi dòng bỏ sót không có id tính 1 vật, nên 8/93 có thể đếm thiếu.
- **Script kiểm được**: tên file, lớp, box trong ảnh, accepted / edited / deleted / added, bẫy, tỉ lệ từ phiếu QC. **Không kiểm được**: box có ôm sát không (img_031, img_035), vật cả máy lẫn người cùng bỏ sót (img_003, img_006), 2 box cho 1 vật, có làm bước 0 thật không.

**Quyết định: Rework**, ngày 2026-11-17. Không Hold: ảnh dùng được, các lỗi đều đã có quy tắc (4.1.1, 4.1.5, mục 2) hoặc đã chốt ở v1.1; câu hỏi mở 1, 3–7 không gây lỗi trong mẫu. Rework: ảnh lỗi 4/5 = 80% > 2%, đối tượng lỗi 8/93 = 8.6%, bắt bẫy lần đầu 7/12 = 58.3% < 80%.
Việc tiếp theo: (1) sửa 8 lỗi QC và 2 bẫy còn lọt (img_013, img_024) trong CVAT; (2) duyệt lại 24 ảnh theo 3 quy tắc ở mục 4, ưu tiên `three_wheeler` / `bicycle` và khối xe đỗ dày (CAL-02); (3) export `submission/labels/qc_fixed.json`, chạy `build_release.py`, `qc_sample.py --force` (seed mới), QC vòng 2.

## 8. Ghi chú cá nhân

- **Dùng AI**: Claude (Claude Code): cài môi trường (torch CUDA, SAM 3, cvat-cli), đưa YOLO26 / SAM 3 vào CVAT bằng Nuclio, sửa lỗi cổng CVAT, giải thích quy trình, kiểm tra định dạng file
- **Gold có bị nhiễm không**: Không. Gold gán tay xong ở job 19 (17:09–18:08) trước job review 20; `pre_traps_import.coco.json` chỉ có 24 ảnh batch.
- **Máy**: Windows 11, RTX 4050 Laptop 6 GB (torch 2.14.1+cu130), Python 3.12 venv; CVAT 2.81.1 (Docker). Vướng mắc: cài torch bản CUDA khớp driver.
- **Dùng cache `data/cache/`?** Không, tự chạy YOLO26m và YOLOE-26s (r01–r06).
- **Bug script**: không gặp.
- **Lần sau**: tách tập chọn ngưỡng khỏi tập chấm (chọn trên pilot, chấm trên gold), vì lần này chọn và chấm cùng 4 ảnh gold nên số đo lạc quan.
