# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 418; đối tượng sau duyệt: 422; ngưỡng accepted: IoU >= 0.9
- Đã **loại 28 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 292 | 292/418 = 69.9% | 418 box máy |
| edited (sửa) | 58 | 58/418 = 13.9% | 418 box máy |
| deleted (xoá) | 48 | 48/418 = 11.5% | 418 box máy |
| added (vẽ thêm) | 68 | 68/422 = 16.1% | 422 đối tượng sau duyệt |

Trong số deleted + added, **13 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 55.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 174 | 15 | 7 | 23 |
| bicycle | 2 | 2 | 1 | 7 |
| motorbike | 80 | 25 | 11 | 20 |
| car | 28 | 7 | 1 | 9 |
| bus | 2 | 0 | 1 | 2 |
| truck | 0 | 0 | 3 | 3 |
| three_wheeler | 6 | 9 | 24 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 41 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 39 | 15 | 2 | 72.2% (KTC 95%: 59–82%) | 95.1% (KTC 95%: 84–99%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 3/3
  - extra: 2/3
  - loose: 2/3
  - wrong_class: 3/3
  - lọt: img_009.jpg (loose), img_035.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
