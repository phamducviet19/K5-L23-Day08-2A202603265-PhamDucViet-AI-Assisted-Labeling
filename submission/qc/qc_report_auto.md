# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 6 ảnh (1 ca khó + 5 ngẫu nhiên), 100 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 6/100 = 6.0% | 100 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 6/6 = 100.0% | 6 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 375/433 = 86.6% / 29/433 = 6.7% / 29/433 = 6.7% | 433 box máy |
| Máy: added | 83/487 = 17.0% | 487 đối tượng sau duyệt |
| Bắt lỗi bẫy | 9/12 = 75.0% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 6 | img_003.jpg, img_006.jpg, img_013.jpg |

**Lỗi hệ thống**: 'bo_sot' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 6 ảnh lỗi trong 6 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **100%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 100.0% > 2%
- lỗi hệ thống: 'bo_sot' chiếm 6/6 lỗi
- bắt bẫy 75% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
