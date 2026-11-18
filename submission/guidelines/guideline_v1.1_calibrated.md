# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603265 - Phạm Đức Việt

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Gán `bicycle` nếu xe có bàn đạp trợ lực; Gán `motorbike` nếu xe chạy hoàn toàn bằng điện/tay ga, có sàn để chân và không có bàn đạp. | `img_004.jpg` | `CAL-02` |
| 2 | Khối xe máy đỗ dày đặc gần camera hoặc ở xa: từng xe hay `ignore_region`? | Nếu xe rõ và tách được viền: gán từng xe riêng; Nếu cụm từ 5 xe trở lên đỗ san sát nhau ở xa hoặc che khuất nhau khiến chiều cao/chiều rộng nhìn thấy < 20px thì dùng 1 box `ignore_region` bao quanh cả cụm. | `img_006.jpg` | `CAL-03` |
| 3 | Vật bị cắt mép ảnh chỉ lộ một phần nhỏ: gán hay bỏ? | Vật bị cắt mép ảnh hoặc bị che khuất mà diện tích nhìn thấy < 20% hoặc không đủ đặc trưng nhận diện lớp (như chỉ thấy 1 góc gương, 1 mẩu cản xe) thì không gán nhãn. | `img_023.jpg` | `CAL-04` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Xe tuk-tuk / xe ba bánh có người lái bên trong | YOLO26-m nhận nhầm thành `car` hoặc `truck` | Bắt buộc đổi nhãn thành `three_wheeler`; Người lái gán riêng 1 box `person`. Khi chạy tự động dùng YOLOE chuyên biệt hoặc merge prelabel. | Mục 2 & Mục 4.1.4 (bổ sung phân loại xe ba bánh) |
| `img_003.jpg` | Nhóm người đi bộ đứng ở hậu cảnh rất xa, chen chúc | Máy bỏ sót hoặc gán box chồng lấn lung tung | Với các cá nhân có chiều cao >= 20px: gán từng `person`; Nhóm người quá đông đúc ở xa không thể tách biên cá nhân thì vẽ `ignore_region`. | Mục 4.1.6 (mở rộng ignore_region cho người đi bộ ở xa) |
| `img_033.jpg` | Cụm phương tiện đỗ trên vỉa hè bị cây cối che khuất một phần | YOLOE và YOLO26 bỏ sót nhiều xe ở góc khuất | Quét kiểm tra bằng mắt: các xe còn lộ rõ bánh và thân xe phải vẽ thêm `motorbike`/`bicycle`; phần bị khuất sâu không đoán. | Mục 4.1.2 & Mục 4.1.6 (kiểm tra góc khuất) |
| `img_017.jpg` | Bẫy loose: box máy vẽ quá rộng bao cả khoảng trống xung quanh | Prelabel bẫy vẽ box rộng hơn 20% | Thu nhỏ cạnh box bám sát phần nhìn thấy của thân xe (sai số <= 2px theo quy tắc 4.1.1). | Mục 4.1.1 (siết chặt biên box) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | Khung tiêu chuẩn ban đầu |
| v1.1 | Chốt phân loại xe đạp điện (`bicycle`/`motorbike`), quy định ngưỡng vẽ cụm `ignore_region` cho đám đông/xe đỗ xa, tiêu chuẩn loại bỏ vật cắt mép <20%, và khắc phục lỗi YOLO26 nhận nhầm tuk-tuk. | Rút ra từ lỗi máy, câu hỏi mở và thực tế review (`CAL-01`, `CAL-02`, `CAL-03`, `CAL-04`) |
