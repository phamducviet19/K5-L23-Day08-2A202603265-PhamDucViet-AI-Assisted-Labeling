# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 433; đối tượng sau duyệt: 487; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 375 | 375/433 = 86.6% | 433 box máy |
| edited (sửa) | 29 | 29/433 = 6.7% | 433 box máy |
| deleted (xoá) | 29 | 29/433 = 6.7% | 433 box máy |
| added (vẽ thêm) | 83 | 83/487 = 17.0% | 487 đối tượng sau duyệt |

Trong số deleted + added, **4 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 79.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 231 | 11 | 7 | 24 |
| bicycle | 6 | 0 | 3 | 5 |
| motorbike | 82 | 8 | 1 | 31 |
| car | 45 | 1 | 5 | 14 |
| bus | 3 | 0 | 0 | 1 |
| truck | 3 | 0 | 4 | 1 |
| three_wheeler | 5 | 9 | 9 | 7 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 36 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 29 | 34 | 7 | 46.0% (KTC 95%: 34–58%) | 80.6% (KTC 95%: 65–90%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 9/12 = 75.0% (KTC 95%: 47–91%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 2/3
  - extra: 3/3
  - missing: 2/3
  - wrong_class: 2/3
  - lọt: img_026.jpg (missing), img_029.jpg (wrong_class), img_017.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
