# Thông tin học viên

- Họ và tên: Trần Gia Huy
- MSHV: 2A202603049
- Lớp (L2 / L3 / khác): K5-L23
- Tài khoản GitHub: trngiahuyyyy
- Repo bài nộp (URL): https://github.com/trngiahuyyyy/K5-L23-DAY08-2A202603049-TranGiaHuy-AIPrelabel
- Commit chốt bài (hash): xem commit cuối cùng trên nhánh `main` (commit Part 3 gần nhất: `8973b0d`)
- Máy: Windows, RTX 3050 4GB (cài PyTorch bản CPU nên toàn bộ model chạy trên CPU)
- CVAT (Docker): CVAT Docker local (`http://localhost:8080`), v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 292/418 = 69.9% / 58/418 = 13.9% / 48/418 = 11.5% (mẫu số: 418 box máy, đã loại 28 đối tượng quanh bẫy); added: 68/422 = 16.1% (mẫu số: 422 đối tượng sau duyệt; trong đó 13 đổi lớp, sót thật ≈ 55)
- So với gold: precision / recall của **máy**: Precision 39/54 = 72.2% (KTC 95%: 59–82%) / Recall 39/41 = 95.1% (KTC 95%: 84–99%) (4 ảnh gold, mẫu số 54 box máy / 41 đối tượng gold)
- Bắt bẫy: 10/12 = 83.3% (KTC 95%: 55–95%; lọt `img_009.jpg` loose, `img_035.jpg` extra)
- Thời gian: gán tay 10.0 phút/ảnh (40 phút / 4 ảnh); duyệt có pre-label 2.9 phút/ảnh (70 phút / 24 ảnh)
- Lỗi QC: 40.0% theo ảnh (2/5 ảnh, cận trên 95% Wilson: 77%), 2.1% theo đối tượng (2/94 đối tượng)
- **Quyết định:** Rework — lý do một dòng: tỉ lệ ảnh có lỗi 2/5 = 40.0% vượt ngưỡng Release ≤ 2% (2 lỗi `thua` ở `img_003.jpg` #18 và `img_035.jpg` #454), cần sửa nhãn và QC lại vòng sau.
