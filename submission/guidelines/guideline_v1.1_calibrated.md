# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603345 - Nguyễn Tuấn Dũng

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện có bàn đạp + motor: label `bicycle` hay `motorbike`? | **motorbike** — ưu tiên theo phương tiện di chuyển chính (motor điện); nếu không rõ → chọn `motorbike` | img_021 | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_033 | Xe 3 bánh chở hàng bị model label là `bicycle` | YOLO26 (conf=0.25) label sai class `bicycle` cho xe ba bánh nhỏ | `three_wheeler` — xe 3 bánh chở hàng/người luôn là three_wheeler dù kích thước nhỏ | Mục 3.3: Xe 3 bánh → three_wheeler; không nhầm với bicycle dù khung hình giống |
| img_006 | Box xe ô tô quá rộng, bao phủ cả phần khuất sau tường | YOLO26 extend box vào vùng bị khuất hoàn toàn | Chỉ label phần xe thực sự visible; không box vào vùng bị che khuất | Mục 4.1: Box phải bao đúng phần object nhìn thấy; không extend vào vùng bị khuất |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi xe đạp điện → motorbike; thêm quy tắc three_wheeler vs bicycle; thêm quy tắc box chỉ bao phần visible | CAL-01 (img_021), CAL-02 (img_033), CAL-03 (img_006) |
