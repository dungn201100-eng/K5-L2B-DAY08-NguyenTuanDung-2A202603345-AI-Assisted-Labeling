# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 577; đối tượng sau duyệt: 576; ngưỡng accepted: IoU >= 0.9
- Đã **loại 36 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 573 | 573/577 = 99.3% | 577 box máy |
| edited (sửa) | 0 | 0/577 = 0.0% | 577 box máy |
| deleted (xoá) | 4 | 4/577 = 0.7% | 577 box máy |
| added (vẽ thêm) | 3 | 3/576 = 0.5% | 576 đối tượng sau duyệt |

Trong số deleted + added, **1 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 2.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 266 | 0 | 1 | 1 |
| bicycle | 9 | 0 | 1 | 0 |
| motorbike | 196 | 0 | 0 | 1 |
| car | 54 | 0 | 0 | 0 |
| bus | 3 | 0 | 0 | 0 |
| truck | 6 | 0 | 0 | 0 |
| three_wheeler | 39 | 0 | 2 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 70 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 55 | 16 | 15 | 77.5% (KTC 95%: 66–86%) | 78.6% (KTC 95%: 68–87%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 3/12 = 25.0% (KTC 95%: 9–53%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 1/3
  - wrong_class: 1/3
  - missing: 1/3
  - loose: 0/3
  - lọt: img_033.jpg (wrong_class), img_025.jpg (extra), img_034.jpg (missing), img_009.jpg (extra), img_006.jpg (loose), img_035.jpg (wrong_class), img_018.jpg (missing), img_017.jpg (loose), img_027.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
