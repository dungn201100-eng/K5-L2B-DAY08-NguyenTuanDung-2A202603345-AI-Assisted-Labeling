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
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
