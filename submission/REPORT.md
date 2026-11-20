# Báo cáo Day 08 — 2A202603049 - Trần Gia Huy

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26s | person 0.35, car 0.45, các lớp còn lại mặc định 0.3 (từ `threshold_sweep.md`; `motorbike` dùng mặc định 0.3 vì không ngưỡng nào `<= 0.5` đạt recall 90%, tối đa 89% trên 9 vật gold; `bicycle`/`bus`/`truck` dùng mặc định 0.3 vì `< 5` vật gold) | r03 (conf 0.25 trên 8 ảnh pilot): 195 box, 0.51 s/ảnh, không thiếu vật nào thuộc lớp COCO. r03 (conf 0.5): còn 131 box, mất 64/195 box (khoảng 33%), chủ yếu là vật ở xa. |
| three_wheeler | YOLOE-26s, prompt: "tuk-tuk", "cyclo", "cargo tricycle" | 0.15 | r02 (conf 0.25 trên 8 ảnh pilot): 8 box `three_wheeler` trong 3/8 ảnh. Đúng: tuk-tuk ở img_002 (0.93), xích lô ở img_032 (0.92). Sót: xích lô nhỏ ở vỉa hè img_007. Hai box 0.37 bên phải img_002 có vẻ là sạp hàng / xe bán tải. Không ra box ở img_015, img_016, img_019, img_030 là đúng vì các ảnh này không có xe ba bánh. YOLO26 không có lớp `three_wheeler` nên chỉ YOLOE gọi được tên lớp này. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,car=0.45,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Part 2 thử `mobile_sam.pt` (SAM 1, r04, 4.63 s/ảnh trên CPU); Part 5 chọn **SAM 2.1 (`sam2.1_b.pt`, r05)** vì trên `img_013.jpg` ra 1 polygon liền 45.4% ảnh so với `mobile_sam.pt` 2 polygon rời 35.2%, chấp nhận chậm hơn (14.61s vs 6.79s CPU); không dùng SAM 3 vì gated/chưa có HF token.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Ở `img_007.jpg`, xích lô bị gán `motorbike` ở cả conf 0.25 và conf 0.5 (r03), vì YOLO26 là detector closed-set, không có lớp `three_wheeler`. Cách sửa: chạy YOLOE cho lớp `three_wheeler`, gộp bằng `merge_prelabels.py`, và khi review xử lý box `motorbike` trùng với `three_wheeler` theo `CAL-01` (giữ `three_wheeler`, xoá `motorbike` trùng).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm thấp mà đúng: `img_032.jpg` (box 0.48 và 0.27 đều là xích lô thật). Điểm thấp mà sai hoặc không chắc: `img_002.jpg` (hai box 0.37 bên phải) và `img_011.jpg` (box 0.28 phủ dải xe xanh bị cắt ở mép trái). Điểm cao mà sai: chưa thấy trong 8 ảnh pilot.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan. Gold có 41 đối tượng (`person` 13, `motorbike` 9, `car` 8, `three_wheeler` 7, `bicycle` 3, `bus` 1, `truck` 0; nguồn `gold_manual.json`, không tính 5 vùng `ignore_region`). Lớp có dưới 5 vật (`bicycle`, `bus`, `truck`) giữ ngưỡng mặc định 0.3.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 292 | 69.9% | 418 box máy |
| edited | 58 | 13.9% | 418 box máy |
| deleted | 48 | 11.5% | 418 box máy |
| added | 68 | 16.1% | 422 đối tượng sau duyệt |

- Ghi chú mẫu số (nguồn `qc/review_stats.md`): đã **loại 28 đối tượng nằm quanh 12 bẫy** khỏi bảng trên vì lỗi nhân tạo không phản ánh chất lượng máy; trong `deleted + added` có **13 trường hợp đổi lớp**, nên số vật máy **sót thật** ≈ `68 − 13 = 55` (trên 422 đối tượng sau duyệt).
- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Nhãn `three_wheeler` (6 accepted, 9 edited, **24/39 = 61.5%** box máy bị xoá, 4 added; ví dụ `img_003.jpg` xoá 6/8, `img_014.jpg` xoá 3/3, `img_023.jpg` xoá 2/5) vì ngưỡng 0.15 khá thấp + prompt tiếng Anh của YOLOE dễ nhận nhầm xe khác/vật nền.
- So với gold (4 ảnh, nguồn `qc/review_stats.md`): **Precision** của máy là `39/54 = 72.2%` (KTC 95%: 59–82%; TP=39, FP=15, mẫu số 54 box máy); **Recall** của máy là `39/41 = 95.1%` (KTC 95%: 84–99%; TP=39, FN=2, mẫu số 41 đối tượng gold, không tính 5 `ignore_region`).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: `10/12 = 83.3%` (KTC 95%: 55–95% — khoảng tin cậy rộng do chỉ có 12 bẫy; `missing` 3/3, `wrong_class` 3/3, `extra` 2/3, `loose` 2/3); loại nào lọt: 1 `loose`, 1 `extra`; vì sao (bạn đã bỏ bước nào của checklist)? Tôi bỏ qua bước 3 (kiểm lớp & 4 cạnh viền — `loose`) với ảnh `img_009.jpg` và bước 1–2 (quét toàn ảnh & xoá box ma — `extra`) với ảnh `img_035.jpg`.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Duyệt tương đối kỹ, không duyệt qua loa (`accepted` 69.9%, bắt bẫy 83.3% đạt ngưỡng ≥ 80%), tuy nhiên vẫn bỏ sót một số bước trong checklist (vùng góc/mép trên ảnh đông và kiểm lại lớp/viền) đồng thời máy vẫn sót thật khoảng 55 vật.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay `10.0 phút/ảnh` (`40 phút / 4 ảnh`) vs duyệt `2.9 phút/ảnh` (`70 phút / 24 ảnh`). Thiên lệch (bạn gán gold trước nên đã quen cảnh): đã quen thao tác CVAT từ trước nên tốc độ giảm từ 10.0 xuống 2.9 phút/ảnh chủ yếu là nhờ máy vẽ sẵn phần lớn box (`accepted 292/418 = 69.9%`), chỉ rà soát và sửa lại. Tuy nhiên phép so sánh vẫn có hai điểm lệch:
  - Tập gold chỉ có 4 ảnh làm trước nên đã quen bối cảnh và guideline v1.0 trước khi sang batch.
  - Cảm giác “chỉ check lại cho nhanh” cũng dễ kéo theo thiên kiến tự động hóa.
- Kết luận: **Có**, dựa trên tốc độ nhanh hơn ~3.4 lần và `accepted` 69.9%, đổi lại vẫn phải kiểm kỹ vì máy sót thật ~55 vật, `three_wheeler` bị xoá 24/39, và 2 bẫy lọt vẫn có thể sửa tay được.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Trước khi bật nhãn máy, bấm ẩn toàn bộ box và quét mắt đủ 4 cạnh mép + 4 góc ảnh tối thiểu 10 giây.
  2. Với 100% box `three_wheeler` và mọi box có `conf < 0.4`, bắt buộc phóng to kiểm đủ 4 cạnh chạm mép vật.
  3. Giới hạn mỗi lượt duyệt tối đa 8–10 ảnh rồi nghỉ 3–5 phút, và sau mỗi lượt tự kiểm lại ngẫu nhiên 1–2 ảnh.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 click (1 âm + 2 dương) | Bao gồm người lái và hàng hoá | Mask dính 1 ít người lái và 1 ít hàng hoá | `tuktuk_img013.jpg` |
| img_003.jpg | SAM 2.1 (`sam2.1_b.pt`) | 2 click dương + 1 box | Bao gồm người lái | Mask dính người lái | `three_wheeler_img003.jpg` |
| img_010.jpg | SAM 2.1 (`sam2.1_b.pt`) | 2 click dương + 1 box | Bao gồm người lái | Mask dính 1 ít người lái và 1 ít người sau xe | `three_wheeler_img010.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu?
  - Quy tắc giúp: mục 4.2.4 (hai vật chạm nhau không chồng polygon, vùng chạm thuộc vật ở trước) giúp kết luận phần dính người lái (`img_013`, `img_003`, `img_010`) và người sau xe (`img_010`) là lỗi cần tách ra bằng click âm hoặc sửa tay.
  - Quy tắc còn thiếu: mục 4.2 chưa nói rõ hàng hoá chở trên xe (`img_013`) có nằm trong mask xe không (mục 4.1.5 mới nhắc ngưỡng 20% cho box, chưa có cho mask — trùng câu hỏi mở #3), và chưa nói rõ khi người lái ngồi trong khung/sau kính xe thì cắt ranh giới giữa mask xe và `person` thế nào (câu hỏi mở #4 & #5).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: chưa thử (chỉ chạy offline bằng `scripts/sam2_points.py` do CVAT Docker local chưa gắn Nuclio/SAM); khác biệt so với CVAT là chạy offline phải nhập toạ độ pixel thủ công cho `--pos`/`--neg` thay vì click trực tiếp và thấy mask đổi ngay.

## 6. Calibration (Part 6)

- Số quyết định: **4** (`submission/qc/calibration_decisions.csv`: `CAL-01` đến `CAL-04`); quan trọng nhất (dẫn `CAL-01`): xử lý `three_wheeler` vs `motorbike` khi 2 model đè nhau / YOLOE gán thừa `24/39` box trên `img_007.jpg`, `img_003.jpg`.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Phân biệt `three_wheeler` vs `motorbike` theo 3 bánh/khung mui/thùng cố định, xoá box trùng/thừa (`CAL-01`). Tách người lái, người đứng sau và hàng hoá rời ra khỏi mask `three_wheeler` (`CAL-02`). Hàng xe máy đỗ sát gần camera ưu tiên gán từng xe rõ (≥ 20 px, che ≤ 80%), chỉ gom `ignore_region` phần cụm phía sau ≥ 5 xe không tách nổi (`img_005.jpg`, `img_018.jpg`, `CAL-03`). Bổ sung bước quét 4 góc/mép ảnh và kiểm lớp trước khi xoá box (`img_035.jpg`, `img_009.jpg`, `CAL-04`).
- Nguyên nhân gốc: tách rõ 3 nhóm — **lỗi máy** (`CAL-01`: YOLO26 closed-set thiếu `three_wheeler`, YOLOE ngưỡng 0.15 sinh nhiều box thừa), **guideline mơ hồ** (`CAL-02`, `CAL-03`: chưa quy định rõ hàng hoá/người lái trên mask và ngưỡng tách cụm xe máy gần camera), và **thiên kiến / lỗi thao tác duyệt** (`CAL-04`: không cần đổi định nghĩa lớp, chỉ siết quy trình quét 4 góc/rìa ảnh).

## 7. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** (`0` ảnh `can_review` + `5` ngẫu nhiên `img_003, img_006, img_014, img_031, img_035`, seed 42; tổng `94` đối tượng); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt nên có thể lặp lại cùng điểm mù).
- Lỗi theo **đối tượng**: `2/94 = 2.1%` · theo **ảnh**: `2/5 = 40.0%` (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: `thua` — `2` đối tượng lỗi (`2/94 = 2.1%`): `img_003.jpg` (`#18`, box `three_wheeler` thừa sát mép phải) và `img_035.jpg` (`#454`, box ma `motorbike` ở góc trên-phải).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): với `2/5` ảnh lỗi, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **77%**; mẫu 5 ảnh quá nhỏ nên không thể chứng minh đạt ngưỡng `≤ 2%`.
- Script kiểm được / không kiểm được: script **kiểm được** tên file, lớp hợp lệ theo taxonomy, box nằm trong ảnh, đủ số ảnh, thống kê `accepted/edited/deleted/added`, tỉ lệ bắt bẫy và tính tỉ lệ lỗi từ `qc_review.csv`; script **không kiểm được** box có ôm sát vật thật hay không, lớp có đúng thực tế không, vật mà cả máy lẫn người cùng bỏ sót, và việc bạn có thực sự làm đúng checklist hay không.

**Quyết định: Rework**, ngày 2026-11-20. Lý do (đi theo Hold → Rework → Release): Không Hold vì không có ảnh hỏng (`data_problem = 0`) và các ca mơ hồ chính đã chốt vào guideline v1.1 (`guideline_gap = 0`). Dừng ở **Rework** vì tỉ lệ ảnh lỗi QC là `2/5 = 40.0%`, vượt xa ngưỡng Release `≤ 2%` (dù bắt bẫy đạt `10/12 = 83.3% ≥ 80%`).
Nếu Rework / Hold, việc cần làm tiếp: mở CVAT xoá 2 box thừa đã phát hiện (`img_003.jpg` `#18`, `img_035.jpg` `#454`), rà soát lại vùng mép/4 góc trên toàn bộ 24 ảnh (và kiểm lại `img_009.jpg`) theo quy tắc `CAL-04`, xuất `submission/labels/qc_fixed.json`, chạy lại `build_release.py` và `qc_sample.py --force` cho vòng QC sau.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Sử dụng `claude.ai` và Antigravity để giải thích lệnh, fix lỗi môi trường/CSV, đối chiếu số liệu và góp ý cấu trúc báo cáo; tự kiểm tra bằng cách chạy lại các lệnh script, đối chiếu trực tiếp với file output (`review_stats.md`, `threshold_sweep.md`, `qc_report_auto.md`), tự mở ảnh kiểm tra trực quan và tự chốt các quyết định/kết luận.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows / RTX 3050 4GB (cài PyTorch bản CPU nên toàn bộ model chạy trên CPU) / CVAT Docker v2.81.0 (`http://localhost:8080`); vướng mắc: máy không có `winget`, Execution Policy chặn kích hoạt `.venv`, và có lúc chạy nhầm Python 3.10 nên phải tạo lại `.venv` Python 3.12 rồi chạy lại pre-label.
- Dùng cache `data/cache/` thay vì tự chạy? Không, vì tự chạy local.
- Lỗi / bug script đã gặp (nếu có): Không gặp bug script; chỉ gặp lỗi `git push` bị từ chối lần đầu và đã xử lý.
- Một điều bạn làm khác lần sau: Sẽ kiểm tra kỹ hơn nữa (đặc biệt vùng mép/4 góc ảnh) để tránh để thừa box sau khi model gán.
