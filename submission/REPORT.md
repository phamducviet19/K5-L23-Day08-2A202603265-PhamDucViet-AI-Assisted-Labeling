# Báo cáo Day 08 — 2A202603265 - Phạm Đức Việt

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person=0.35, bicycle=0.25, motorbike=0.35, car=0.35, bus=0.40, truck=0.35 | Dẫn `r01` và `r02`: YOLO26m bao quát tốt 6 lớp COCO thông dụng khi conf ~0.25-0.35; ở conf=0.5 (r02) bị sót nhiều xe nhỏ ở xa |
| three_wheeler | YOLOE-26s, prompt: tuk-tuk, cyclo, cargo tricycle -> three_wheeler | 0.25 | Dẫn `r03`: YOLO26 hoàn toàn mù xe ba bánh (gán nhầm car/truck); YOLOE là model open-vocabulary nhận diện được 8 xe ba bánh trong pilot |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --conf 0.05 --thr "person=0.35,bicycle=0.25,motorbike=0.35,car=0.35,bus=0.40,truck=0.35,three_wheeler=0.25" --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1** (`sam2.1_b.pt`) vì chất lượng đường biên mask bám sát chi tiết phức tạp của xe và người, cân bằng tốt giữa kích thước mô hình (~160MB) và tốc độ suy luận; không đòi hỏi quyền phân phối như SAM 3 và vượt trội hơn hẳn MobileSAM về độ kín viền.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi closed-set misclassification khi YOLO26 không có nhãn xe ba bánh nên gọi tuk-tuk là `car`/`truck` với độ tin cậy rất cao (>0.85); tăng hay giảm conf đều không sửa được. Cách sửa: dùng model open-vocabulary YOLOE gán riêng cho `three_wheeler`, sau đó dùng `merge_prelabels.py` với ngưỡng IoU >= 0.6 để nhãn của YOLOE đè lên `car`/`truck` của YOLO26.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: Ở `img_002.jpg`, YOLO26 nhận diện xe tuk-tuk thành `car` với conf 0.85; ở `img_013.jpg`, box `car` điểm 0.84 trùm lên xe tuk-tuk.
  - Điểm thấp mà đúng: Ở `img_011.jpg`, YOLOE phát hiện xích lô/xe ba bánh với conf chỉ 0.28 nhưng bounding box đúng vị trí thực tế.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Việc sweep ngưỡng in-sample trên 4 ảnh gold (36 đối tượng) khiến các chỉ số Precision/Recall đo được trên gold có xu hướng cao hơn thực tế (optimistic bias); khi áp dụng bộ ngưỡng này vào 24 ảnh batch bên ngoài thì gặp nhiều trường hợp biến thể môi trường chưa thấy trong gold.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 375 | 375/433 = 86.6% | 433 box máy |
| edited | 29 | 29/433 = 6.7% | 433 box máy |
| deleted | 29 | 29/433 = 6.7% | 433 box máy |
| added | 83 | 83/487 = 17.0% | 487 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` tệ nhất khi bị deleted 9, edited 9 và chỉ accepted 5 (tỉ lệ giữ nguyên < 25%), do máy thường dự đoán box lỏng và nhầm lẫn với car/truck ở `img_002.jpg`. Lớp `truck` cũng bị xóa 4/7 box do nhận diện nhầm đuôi xe buýt ở `img_008.jpg`.
- So với gold: precision / recall của **máy**: Precision = 46.0% (KTC 95%: 34–58%), Recall = 80.6% (KTC 95%: 65–90%) (mẫu số: 36 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 9/12 = 75.0%; loại nào lọt: `img_026.jpg` (missing), `img_029.jpg` (wrong_class), `img_017.jpg` (loose); vì sao (bạn đã bỏ bước nào của checklist)? Do lướt nhanh qua các ảnh đông đúc, nhìn box máy tưởng đã đủ nên bỏ qua việc quét mắt tìm đối tượng bị sót (missing ở img_026) và chưa soi kỹ nhãn xe máy vs xe đạp điện (wrong_class ở img_029), cũng như không phóng to kiểm tra độ khít biên cạnh box (loose ở img_017). Đã bỏ bước quét toàn bộ ảnh bằng mắt trước khi bật layer pre-label.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỉ lệ Accepted cao (86.6%) trong khi bắt bẫy chưa đạt ngưỡng tối thiểu (75.0% < 80%) phản ánh hiện tượng Automation Bias rõ rệt: người duyệt tin tưởng vào đề xuất của mô hình AI, lười can thiệp vào các box có sẵn nên dễ dàng bỏ lọt các bẫy tinh vi.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 6.25 phút/ảnh (25 phút / 4 ảnh) vs duyệt 3.75 phút/ảnh (90 phút / 24 ảnh). Thiên lệch (bạn gán gold trước nên đã quen cảnh): Người gán đã quen mắt với bối cảnh giao thông và định nghĩa taxonomy từ bước gán gold nên khi chuyển sang batch review thao tác có phần nhanh hơn nhờ hiệu ứng học tập.
- Kết luận: **Có, nhưng chỉ đáng dùng cho các lớp phổ biến (person, motorbike, car)**; dựa trên việc accepted đạt 86.6% và tiết kiệm gần 40% thời gian mỗi ảnh so với vẽ tay hoàn toàn. Đối với các lớp đặc thù như `three_wheeler`, máy hỗ trợ kém và cần người sửa nhiều.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. *Quét mù trước*: Luôn tắt layer nhãn máy trong 15 giây đầu tiên khi mở ảnh để quan sát bằng mắt thường và phát hiện các đối tượng bị bỏ sót.
  2. *Kiểm tra biên zoom-in*: Với mọi box lớn > 100px, phóng to 200% kiểm tra 4 cạnh chạm sát viền ngoài cùng của vật trước khi chấp nhận.
  3. *Soi kỹ ca phân loại nhạy cảm*: Dừng lại tối thiểu 20 giây đối chiếu guideline v1.1 với mọi trường hợp xe ba bánh, xe đạp điện hoặc xe cắt mép ảnh.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_004.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 2 click (1 dương, 1 âm) | Không gồm người lái (đã tách người/nền) | Bám biên tốt phần nhìn thấy, có bỏ qua vùng lỗ thủng nhìn xuyên qua xe | `tuktuk_img004_sam2.jpg` |
| `img_013.jpg` | SAM 1 (`mobile_sam.pt`) | 3 click (2 dương, 1 âm) | Cố tách người nhưng chưa sạch, vẫn dính phần trên mặt | Bám biên và tách người ngồi trên xe nhưng vẫn chưa sạch và vẫn dính phần trên mặt, khung xe cũng chưa kín lắm, có bỏ qua vùng lỗ thủng không thuộc về xe | `tuktuk_img013_sam1.jpg` |
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 dương, 1 âm) | Dính cả tài xế trên xe (chưa tách được người) | Nhiều nhiễu, bị lem ra các phần ngoài xe và tài xế trên xe, có bỏ qua vùng lỗ thủng không thuộc về xe nhưng có vùng vẫn dính nhiều | `tuktuk_img013_sam2.jpg` |
| `img_017.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 2 click (1 dương, 1 âm) | Không gồm người lái | Tương đối sát viền nhưng phần lỗ vẫn bị đè nhiều và chưa cover hết vùng xe | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc 4.2.2 về việc loại bỏ lỗ thủng lớn (> 5% diện tích) giúp định hướng rõ cho việc kiểm tra mask của SAM khi gặp kết cấu hở của xe ba bánh; Quy tắc còn thiếu hướng dẫn phân tách ranh giới tiếp giáp giữa người lái và thân xe trong phân đoạn instance (khiến SAM 2.1 dễ lem mask xe trùm lên tài xế) và chưa cụ thể hóa trường hợp kính chắn gió nhìn xuyên thấu nội thất/người ngồi bên trong.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Đã thử nghiệm offline qua script `sam2_points.py` để chủ động kiểm soát chính xác toạ độ click dương/âm và so sánh trực tiếp giữa SAM 1 (`mobile_sam.pt`) và SAM 2.1 (`sam2.1_b.pt`); so với Interactor CVAT online, bản offline chạy trực tiếp bằng PyTorch local không bị nghẽn mạng/timeout và lưu trực tiếp file mask minh chứng kèm chấm toạ độ trực quan.

## 6. Calibration (Part 6)

- Số quyết định: 4 (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): `CAL-01` (khắc phục lỗi YOLO26 gán nhầm tuk-tuk thành car/truck trên `img_013`, `img_002`) và `CAL-02` (chốt phân loại xe đạp điện có bàn đạp là `bicycle`, không bàn đạp là `motorbike` trên `img_004`).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Tiêu chuẩn bàn đạp cho xe đạp điện, quy tắc gom cụm `ignore_region` cho nhóm người/xe máy đỗ xa dày đặc (>= 5 vật < 20px), và loại bỏ vật cắt mép nhìn thấy < 20% diện tích.
- Nguyên nhân gốc: Kết hợp giữa giới hạn danh mục mô hình máy (`loi_may`) và sự mơ hồ chưa bao quát hết của guideline v1.0 (`cau_hoi_mo`).

## 7. QC và quyết định (Part 6)

- Mẫu: 6 ảnh (1 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 6/100 = 6.0% · theo **ảnh**: 6/6 = 100.0% (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: `bo_sot` chiếm 6/6 lỗi (ví dụ: nhóm người ở xa trên `img_003.jpg`, xe máy chưa gán trên `img_006.jpg`, xe cắt mép trên `img_013.jpg` và `img_023.jpg`, cụm xe lề đường trên `img_033.jpg`).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 6 ảnh lỗi trong 6 ảnh mẫu cho cận trên 95% Wilson là **100%**. Mẫu nhỏ không thể chứng minh lô nhãn đạt dưới ngưỡng 2%, chỉ có giá trị cảnh báo lỗi lặp lại.
- Script kiểm được / không kiểm được: Script kiểm tra được số lượng đối tượng, tỷ lệ IoU trùng lặp; script không kiểm tra được độ khít viền ngữ nghĩa theo mắt người và không phát hiện được những đối tượng mà cả người duyệt lẫn AI đều cùng bỏ sót.

**Quyết định: REWORK**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): Guideline đã được bổ sung v1.1 sạch không còn khoảng trống (không Hold), dữ liệu ảnh hoàn chỉnh (không Hold); tuy nhiên tỉ lệ ảnh có lỗi đạt 100.0% (> 2%), xuất hiện lỗi hệ thống bỏ sót lặp lại (`bo_sot` 6/6) và tỉ lệ bắt bẫy 75% < 80%. Theo cây quyết định, bắt buộc phải đưa vào **REWORK**.
Nếu Rework / Hold, việc cần làm tiếp: Mở lại toàn bộ 24 ảnh batch trên CVAT, áp dụng guideline v1.1 để vẽ `ignore_region` cho các cụm người/xe ở xa, rà soát lại các góc khuất và các bẫy bị lọt trước khi build lại release.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Dùng trợ lý Antigravity hỗ trợ điều hướng chạy lệnh CLI, đồng bộ bảng dữ liệu và kiểm tra cấu trúc; toàn bộ kết quả gán nhãn và nhận xét lỗi đều được tự tay kiểm chứng bằng mắt.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không; tự gán tay hoàn toàn 4 ảnh gold trước khi chạy bất kỳ script pre-label nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Chạy trên CPU máy cá nhân Windows (Python 3.12); CVAT chạy qua Docker local; quá trình chạy SAM trên CPU mất ~8.6s/ảnh nhưng chạy trơn tru.
- Dùng cache `data/cache/` thay vì tự chạy? Không dùng cache; tự chạy pre-label trực tiếp trên máy bằng lệnh `prelabel.py`.
- Lỗi / bug script đã gặp (nếu có): Lỗi sai tên file `batch_review.json` khi xuất từ CVAT thiếu hậu tố `ed`, đã giải quyết bằng lệnh copy đồng bộ.
- Một điều bạn làm khác lần sau: Sẽ luôn tuân thủ nguyên tắc "quét mù" bằng mắt thường trước khi nhìn vào gợi ý của máy để không bị ảnh hưởng bởi thiên kiến tự động hóa.
