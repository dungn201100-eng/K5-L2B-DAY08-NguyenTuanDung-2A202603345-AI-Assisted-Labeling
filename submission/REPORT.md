# Báo cáo Day 08 — 2A202603345 - Nguyễn Tuấn Dũng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26n/m | 0.3 (mặc định) | Không có ngưỡng nào <= 0.5 đạt recall 90% (tối đa 66% trên 32 vật gold do nhiều người bị che khuất/ở xa); giữ conf mặc định 0.3 (dẫn r01, r02). |
| bicycle | YOLO26n/m | 0.2 | Đạt recall 100% (>= 90%), precision 83% trên 5 vật gold từ threshold_sweep (dẫn r01). |
| motorbike | YOLO26n/m | 0.15 | Đạt recall 93% (>= 90%), precision 78% trên 15 vật gold từ threshold_sweep (dẫn r01). |
| car | YOLO26n/m | 0.3 (mặc định) | Không ngưỡng nào <= 0.5 đạt recall 90% (tối đa 78% trên 9 vật gold); giữ conf mặc định 0.3 (dẫn r01, r02). |
| bus | YOLO26n/m | 0.3 (mặc định) | Chỉ có 1 vật trong 4 ảnh gold (< 5 mẫu tối thiểu): không đủ độ tin cậy thống kê để chọn ngưỡng riêng, giữ mặc định 0.3. |
| truck | YOLO26n/m | 0.3 (mặc định) | Chỉ có 1 vật trong 4 ảnh gold (< 5 mẫu tối thiểu): không đủ độ tin cậy thống kê để chọn ngưỡng riêng, giữ mặc định 0.3. |
| three_wheeler | YOLOE-26s (prompt: 'tuk-tuk', 'cyclo', 'cargo tricycle') | 0.15 | Đạt recall 100% (>= 90%), precision 78% trên 7 vật gold; bắt trúng xe ba bánh mà YOLO26 bỏ sót (dẫn r03). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "bicycle=0.2,motorbike=0.15,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn SAM 2.1 (`sam2.1_t.pt`) kết hợp SAM 1 (`mobile_sam.pt`). SAM 2.1 cho chất lượng biên viền sắc nét, bám sát cấu trúc khung xe hở nền; bản tiny và MobileSAM chạy nhẹ trên CPU (~4s/ảnh), không tốn RAM và không đòi hỏi quyền truy cập gated model như SAM 3 (dẫn r04).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi nhận nhầm phương tiện địa phương như xe ba bánh (`three_wheeler`: tuk-tuk, ba gác) thành `car` hoặc `truck` của detector COCO (YOLO26). Vì COCO không có nhãn này trong danh mục 80 lớp, nên dù tăng hay hạ ngưỡng thì YOLO26 vẫn gán sai lớp. Sửa bằng cách: dùng YOLOE với prompt text chuyên biệt để phát hiện `three_wheeler`, sau đó đặt file YOLOE lên trước file YOLO26 trong lệnh `merge_prelabels.py` với `--iou 0.6` để ghi đè (override) box sai.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: Ở `img_002.jpg`, YOLO26 nhận diện xe tuk-tuk thành `car` với conf cao (~0.85); hoặc YOLOE sinh box `three_wheeler` đè lên xe bán tải với conf 0.37.
  - Điểm thấp mà đúng: Người đi bộ bị khuất sau xe ở `img_001.jpg` hoặc tuk-tuk ở xa trong `img_012.jpg` chỉ có conf ~0.18–0.22 nhưng là đối tượng thật.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Việc chọn ngưỡng và đo lại Precision/Recall diễn ra trên cùng một tập 4 ảnh gold (in-sample evaluation). Cỡ mẫu 4 ảnh quá nhỏ (nhiều lớp như bus, truck chỉ có 1 vật), dẫn đến hiện tượng quá khớp (overfitting) ngưỡng và số đo trên gold luôn lạc quan hơn so với khi triển khai trên 24 ảnh batch.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 573 | 573/577 = 99.3% | 577 box máy |
| edited (sửa) | 0 | 0/577 = 0.0% | 577 box máy |
| deleted (xoá) | 4 | 4/577 = 0.7% | 577 box máy |
| added (vẽ thêm) | 3 | 3/576 = 0.5% | 576 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? **three_wheeler**: bị xoá 2/39 box (5.1%), do máy YOLO26 không có lớp này trong COCO 80 nên nhận nhầm thành `car`; YOLOE bù một phần nhưng vẫn tạo box thừa ở một số ảnh (ví dụ img_033.jpg). `bicycle` cũng bị xoá 1/9 box (11.1%) — xe đạp điện bị nhầm class.
- So với gold (4 ảnh, 70 đối tượng): Precision = 77.5% (KTC 95%: 66–86%), Recall = 78.6% (KTC 95%: 68–87%). Lạc quan do ngưỡng được chọn in-sample trên 4 ảnh gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **3/12 = 25%** (KTC 95%: 9–53%); loại lọt: extra (2/3), wrong_class (2/3), missing (2/3), loose (3/3). Lọt nhiều nhất là `loose` (box rộng) vì box vẫn bao đúng vật — mắt người dễ chấp nhận box hơi rộng mà không tightening. `missing` lọt do các vật rất nhỏ (img_018 motorbike ~23×22px) hoặc bị che khuất nặng khó phát hiện bằng mắt thường.
- accepted = 99.3% nhưng bắt bẫy chỉ 25%: nghĩa là phần lớn bẫy bị qua do accepted rate cao (box máy gần đúng) khiến người duyệt không cẩn thận kiểm tra từng box. Phải quét ảnh TRƯỚC khi nhìn nhãn máy để tránh thiên kiến xác nhận (confirmation bias).

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **~5.5 phút/ảnh** (22 phút / 4 ảnh gold) vs duyệt **~0.63 phút/ảnh** (15 phút / 24 ảnh batch). Thiên lệch: đã quen cảnh sau khi gán gold, nên duyệt batch nhanh hơn thực tế; thời gian gold có thể còn bao gồm cả việc đọc guideline lần đầu.
- Kết luận: **Có, đáng dùng** — đặc biệt cho `person`, `motorbike`, `car` (accepted > 99%). Với `three_wheeler` nên kết hợp YOLOE, không dùng YOLO26 đơn thuần vì COCO không có lớp này. Tiết kiệm ~88% thời gian (0.63 so với 5.5 phút/ảnh).
- **Ba quy tắc giảm thiên kiến của bạn:**
  1. Quét toàn bộ ảnh bằng mắt từ góc trái sang phải **trước** khi nhìn nhãn máy — đếm sơ bộ số vật để phát hiện bỏ sót.
  2. Kiểm tra riêng lớp `three_wheeler` trong mỗi ảnh (nhìn các xe 3 bánh) vì đây là lớp máy tệ nhất — không để lọt do xen kẽ với `car`.
  3. Dành 1 lượt cuối kiểm chỉ `loose boxes`: phóng to từng box, kéo sát viền vật để box không rộng hơn 10% so với bounding box thực.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_001.jpg | SAM 1 (MobileSAM) | 2 pos + 1 neg | Có (tài xế trong cabin) | Mask bao cả phần bóng xe dưới bánh (~10% diện tích thừa) | tuktuk_img001_mobilesam.jpg |
| img_001.jpg | SAM 2.1-tiny | 2 pos + 1 neg | Có | Sắc nét hơn SAM 1, ít thừa hơn, 3 polygon do xe hở viền | tuktuk_img001_sam2.jpg |
| img_012.jpg | SAM 1 (MobileSAM) | 2 pos + 1 neg | Có | Mask hơi dư vào xe bên cạnh (~8px) ở cạnh phải | tuktuk_img012_mobilesam.jpg |
| img_020.jpg | SAM 2.1-tiny | 2 pos + 1 neg | Có | Mask nhỏ (0.9% ảnh) do xe three_wheeler ở xa, khó bám viền chính xác | tuktuk_img020_sam2.jpg |

- Đối chiếu guideline mục 4.2: Quy tắc "box chỉ bao phần visible" giúp xác định vùng click. Quy tắc còn thiếu: hướng dẫn rõ có nên include bóng (shadow) của xe trong mask hay không — hiện SAM bao cả bóng và chưa có quy tắc loại trừ.
- Interactor SAM trong CVAT: Chưa thử do CVAT Interactor cần GPU model server (nuclio). Offline script (`sam2_points.py`) đơn giản hơn, chạy CPU được với MobileSAM (~2–4s/ảnh).

## 6. Calibration (Part 6)

- Số quyết định: **3** (`calibration_decisions.csv`); quan trọng nhất: **CAL-01** (xe đạp điện → motorbike) vì đây là câu hỏi mở từ v1.0 ảnh hưởng nhiều ca nhất.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: (1) xe có motor điện → `motorbike` không phải `bicycle`; (2) xe 3 bánh chở hàng/người → `three_wheeler` dù kích thước nhỏ; (3) box chỉ bao phần object visible, không extend vào vùng bị khuất.
- Nguyên nhân gốc: **guideline mơ hồ** (v1.0 chưa phân biệt rõ xe đạp điện vs xe máy) + **lỗi máy** (YOLO26 không có lớp three_wheeler trong COCO 80 nên gán sai class liên tục).

## 7. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: Nguyễn Tuấn Dũng (**hạn chế**: vừa là người duyệt lô — không độc lập).
- Lỗi theo **đối tượng**: 2/119 = 1.7% · theo **ảnh**: 2/5 = 40.0% (ngưỡng Release ≤ 2% theo đối tượng, nhưng mẫu quá nhỏ)
- Top lỗi: `sai_vi_tri_kich_thuoc` (1 ca, img_006 — box xe car rộng hơn vùng visible) + `sai_lop` (1 ca, img_035 — YOLO label `bicycle` nhầm cho người đi xe). Hai lỗi này đã được biết từ trước (bẫy lọt qua duyệt).
- **Giới hạn mẫu nhỏ**: 2 ảnh lỗi / 5 ảnh → cận trên 95% (Wilson) của tỉ lệ lỗi thật là **77%**. Mẫu 5 ảnh hoàn toàn không đủ để chứng minh ngưỡng 2%; chỉ mang tính tham khảo.
- Script kiểm được: đếm loại lỗi, tính tỉ lệ, in cận trên Wilson. Script không kiểm được: box có ôm sát vật không (phải mở ảnh thực), bỏ sót cùng máy (nếu máy bỏ sót thì người duyệt cũng không thấy trong ảnh đã có label), duyệt theo đúng checklist hay không.

**Quyết định: REWORK**, ngày 2026-11-19. Lý do: tỉ lệ ảnh lỗi 40% > 2%; bắt bẫy 25% < 80% → lô duyệt chưa đáng tin cậy.
Nếu Rework: cần duyệt lại 24 ảnh lần 2 theo checklist ketat hơn (quét ảnh trước nhìn nhãn, tighten loose box, kiểm tra three_wheeler riêng); tăng mẫu QC lên ≥ 10 ảnh để thu hẹp khoảng tin cậy.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Dùng Antigravity (Gemini) để hỗ trợ cài môi trường (`.venv` Python 3.11, CLIP), chạy script Part 2–3 (tour models, threshold sweep), sinh SAM masks Part 5, và điền hướng dẫn các bước. Tất cả số liệu (gold labels, model runs, review_stats) đều từ chạy thực — không fabricate. Gold labels gán tay tự làm trong CVAT (Part 1). Batch review (Part 4) tự làm thủ công trong CVAT.
- Gold có bị nhiễm không? **Có một phần**: khi gán gold đã biết các lớp cần dùng (person, bicycle, motorbike, car, bus, truck, three_wheeler) từ guideline. Chưa nhìn pre-label trước khi gán xong.
- Máy / GPU / CVAT: CPU (không có GPU), CVAT chạy Docker Desktop local. Vướng mắc: YOLOE cần CLIP library riêng; SAM 2.1 lần đầu tải mobileclip2_b.ts ~242MB cần internet.
- Dùng cache `data/cache/`: **có** — Part 2 dùng cache `yolo26m_all_c005.coco.json` và `yoloe26s_three_wheeler_all_c005.coco.json` để bỏ qua bước inference nặng trên CPU (mỗi lần chạy 24 ảnh mất 10–20 phút không GPU).
- Lỗi / bug script đã gặp: `python` mặc định là Python 3.12 không có packages → phải dùng `.venv\Scripts\python.exe`. Git trên sandbox bị block `index.lock` → chạy git với BypassSandbox. YOLOE lần đầu cần internet để tải mobileclip.
- Một điều làm khác lần sau: Duyệt batch review kỹ hơn (quét ảnh trước nhìn nhãn, tighten all loose boxes), và tăng ngưỡng trap detection bằng cách đọc danh sách checklist sau mỗi ảnh thay vì duyệt nhanh toàn lô.
