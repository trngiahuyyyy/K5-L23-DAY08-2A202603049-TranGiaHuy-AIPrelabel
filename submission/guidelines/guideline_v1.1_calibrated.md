# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603049 - Trần Gia Huy

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (và ghi rõ câu chưa chốt) và thêm quy tắc mới rút ra từ lỗi của máy / bẫy lọt / Part 5 SAM. Mỗi ca khó đều có ảnh ví dụ (`img_0xx`) và khớp 1 dòng trong `submission/qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc gần camera: gán từng xe hay `ignore_region`? | Xe nhìn rõ khung, tách được cá thể (cao ≥ 20 px, che ≤ 80%) vẫn gán từng `motorbike`; chỉ gom `ignore_region` cho cụm phía sau bị che khuất chồng khít ≥ 5 xe không tách nổi cá thể (bổ sung mục 4.1.6). | `img_005.jpg`, `img_018.jpg` | `CAL-03` |
| 3 & 5 | Hàng hoá nhô ra trên mask và ranh giới người lái/người đứng sau qua khung/kính xe ba bánh xử lý thế nào? | Mask `three_wheeler` **không** gồm người lái/người đứng sau (cắt tách riêng cho `person`, dùng click âm `--neg` hoặc sửa biên) và **không** gồm hàng hoá rời nhô ra ngoài thùng xe; chỉ gồm thân xe, khung mui, kính chắn gió và thùng hàng gắn cố định (bổ sung mục 4.2). | `img_013.jpg`, `img_003.jpg`, `img_010.jpg` | `CAL-02` |

*Các câu hỏi mở khác ở mục 6 v1.0 chưa chốt ở v1.1 (câu #1 xe đạp điện, #4 ngưỡng 20% người trong mui, #6 box chồng ảnh đêm, #7 ngưỡng `conf` chấp nhận): chưa gặp đủ ca rõ ràng trong lô ảnh đã duyệt nên giữ nguyên mặc định của v1.0 (không dùng `conf` thay mắt người; ca mơ hồ gắn `can_review`).*

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, Part 5 SAM)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_007.jpg`, `img_003.jpg` | Xích lô bị gán nhầm `motorbike`; nhiều vật nền/xe 2 bánh bị gọi nhầm `three_wheeler` | `YOLO26s` (closed-set) gọi xích lô là `motorbike` (`img_007`); `YOLOE-26s` (ngưỡng `0.15`) sinh nhiều box `three_wheeler` thừa (`img_003` xoá 6/8; toàn batch xoá 24/39). | Phân biệt theo số bánh + khung mui/thùng hàng gắn cố định: đúng xe ba bánh thì giữ `three_wheeler` và xoá box `motorbike` trùng; không đúng thì xoá box `three_wheeler`. *(Nguyên nhân chính: lỗi máy + ngưỡng YOLOE thấp.)* | Bổ sung **mục 2 & mục 5** (`CAL-01`). |
| `img_013.jpg`, `img_003.jpg`, `img_010.jpg` | Mask xe ba bánh ôm dính người lái, hàng hoá rời và người đứng sau xe | `SAM 2.1` (`sam2.1_b.pt`) khi dùng 2–3 click có xu hướng gộp người lái (`3/3` ảnh), hàng hoá (`img_013`) và người sau xe (`img_010`) vào chung 1 polygon `three_wheeler`. | Tách người lái/người sau xe sang `person` và loại hàng hoá rời nhô ra khỏi mask xe; chỉ giữ thân xe, khung mui, kính và thùng cố định. *(Nguyên nhân: guideline v1.0 mục 4.2 chưa nêu rõ cho mask + lỗi dính biên của SAM.)* | Bổ sung **mục 4.2** (`CAL-02`). |
| `img_035.jpg`, `img_009.jpg` | Bỏ lọt bẫy `extra` (box ma ở góc trên-phải ảnh đông 28 box) và bẫy `loose` trên `three_wheeler` | Bẫy nhân tạo từ `inject_traps.py`: box ma `motorbike` góc trên-phải (`img_035`) và box `three_wheeler` bị dịch lệch (`img_009`, sau duyệt không còn lớp `three_wheeler`). | Không sửa định nghĩa lớp; bổ sung quy trình kiểm duyệt: quét đủ 4 góc/vùng rìa trên của ảnh tìm box ma, và kiểm tra lại đúng lớp + 4 cạnh viền trước khi xoá hoặc đổi lớp box. *(Nguyên nhân gốc: thiên kiến tự động hóa / bỏ sót bước checklist, không phải guideline mơ hồ.)* | Bổ sung ghi chú **quy trình review / mục 4.3** (`CAL-04`). |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | Chốt quy tắc phân biệt `three_wheeler` vs `motorbike` khi 2 model đè nhau (`CAL-01`); bổ sung quy tắc tách người lái/người sau xe và hàng hoá rời trên mask `three_wheeler` (`CAL-02`); làm rõ ranh giới gán từng `motorbike` vs `ignore_region` ở dãy xe máy gần camera (`CAL-03`); thêm ghi chú thao tác quét 4 góc/kiểm lớp để tránh thiên kiến tự động hóa (`CAL-04`). | `CAL-01`, `CAL-02`, `CAL-03`, `CAL-04` (`submission/qc/calibration_decisions.csv`) |
