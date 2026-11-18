# Thông tin học viên

- Họ và tên: Phạm Đức Việt
- MSHV: 2A202603265
- Lớp (L2 / L3 / khác): L23
- Tài khoản GitHub: pdviet
- Repo bài nộp (URL): https://github.com/phamducviet19/K5-L23-Day08-2A202603265-PhamDucViet-AI-Assisted-Labeling.git
- Commit chốt bài (hash): 9bd01b8
- Máy: Windows 11, CPU
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 375 (86.6%) / 29 (6.7%) / 29 (6.7%) (mẫu số: 433 box máy); added: 83 (17.0%) (mẫu số: 487 đối tượng)
- So với gold: precision / recall của **máy**: 46.0% (KTC 95%: 34–58%) / 80.6% (KTC 95%: 65–90%)
- Bắt bẫy: 9/12 = 75.0%
- Thời gian: gán tay 6.25 phút/ảnh; duyệt có pre-label 3.75 phút/ảnh
- Lỗi QC: 100.0% theo ảnh (6/6), 6.0% theo đối tượng (6/100)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ ảnh có lỗi trong mẫu QC đạt 100% > 2% và lặp lỗi hệ thống bỏ sót đối tượng ở xa.
