Họ tên / nhóm: `Lê Trung Toán`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `30` phút |
| Thời gian gán `clip_01` | `90` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `chưa ghi nhận` |

Ba tình huống khó nhất:

1. Xe bị che hoặc gần mép ảnh: giữ ID khi còn đủ bằng chứng, bbox chỉ ôm phần nhìn thấy.
2. Xe rời khung: phải kết thúc track đúng frame, không để bbox treo ở các frame sau.
3. Đoạn xe gần như đứng im: kiểm tra ảnh gốc để phân biệt xe đỗ thật với quên bấm `outside`.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua chưa được ghi nhận riêng trong artifact hiện có. Kiểm tra tự động cho thấy:
`clip_01` có 611 bbox, 190 frame, 8 ID và 0 lỗi định dạng; `clip_02` có 238 bbox,
60 frame, 6 ID và 0 lỗi định dạng. Validator còn cảnh báo bbox gần như đứng im ở
`clip_01`, track 2, các đoạn frame `32-46`, `145-159`, `167-181`.

Kiểm chéo: **chưa có `reports/review_partner.md`**. Vì vậy chưa thể xác nhận reviewer,
Pair ID, số lỗi hai chiều hoặc ca hai người bất đồng. Đây là artifact còn thiếu, không
phải kết luận rằng không có lỗi.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `1a438358f2ae3c63e02e2c69c5ab85300dafa55f1acc59064ba160328a3d7fb4` |
| Thời điểm khóa | `2026-09-15T12:25:33.820546+00:00` |
| Số row / frame / track trước khi mở reference | `611 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.8019 | 0.7845 | 0.8205 | 0.8743 | 0.9443 | 0.8848 | 0.8647 | 52 | 14 | 0 |
| Sau rework | chưa chạy | chưa chạy | chưa chạy | chưa chạy | chưa chạy | chưa chạy | chưa chạy | chưa chạy | chưa chạy | chưa chạy |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có ở vòng pre-gold**.

Diagnostic pre-gold ghi nhận ghost/endpoint cần xem lại ở `clip_01`: track 6 frame
`80-100`, track 4 frame `149-151`, track 5 frame `76-78`, track 8 frame `169-171`.
Các bbox lỏng đáng chú ý gồm track 5 frame `84-90`, track 6 frame `110-111`,
track 7 frame `106`, và track 1/2 frame `190`.

Chưa có bằng chứng rework sau khi đọc diagnostic. File annotation hiện tại vẫn trùng
`evidence/pre-gold/clip_01/gt.txt` theo SHA-256.

## 4. Kết quả model: ByteTrack control vs ReID treatment

Notebook Colab không truy cập được từ môi trường hiện tại vì liên kết chuyển sang
Google Cloud xác thực. Các file `outputs/model_run_config.json`, hai file model và
ba file evaluation tương ứng cũng chưa có trong workspace, nên không thể điền số liệu
hoặc kết luận so sánh model một cách đáng tin cậy.

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.8019 | 0.7845 | 0.8205 | 0.8743 | 0.9443 | 0.8848 | 0.8647 | 52 | 14 | 0 |
| ByteTrack control vs gold | chưa có artifact | | | | | | | | | |
| BoT-SORT + ReID vs gold | chưa có artifact | | | | | | | | | |
| ReID vs bạn | chưa có artifact | | | | | | | | | |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1?**

MOTA `0.8848` thấp hơn IDF1 `0.9443`. Cả hai đều qua cổng. Về nguyên tắc, MOTA chủ
yếu phạt FN, FP và ID switch; lỗi hình học và sai identity không được phản ánh mạnh
như IDF1/AssA. Vì vậy một nhãn có MOTA cao vẫn có thể giữ identity kém.

**2. ByteTrack và BoT-SORT + ReID khác nhau thế nào?**

Chưa thể kết luận vì chưa có hai file evaluation model. Khi chạy notebook, cần báo cáo
IDF1, AssA, IDSW và một chuỗi frame cụ thể; đây là so sánh hai implementation tracker,
không phải causal ablation cô lập riêng tác động của ReID.

**3. DetA, FP và FN đổi thế nào?**

Với annotation so với gold, DetA là `0.7845`, FP `52`, FN `14`; chênh lệch giữa hai
model chưa có artifact. Các diagnostic hiện có gợi ý lỗi còn lại gồm cả endpoint/ghost
và hình học bbox, chưa đủ cơ sở quy toàn bộ cho detector hay association.

**4. Một chỗ bạn đúng và ReID sai:** chưa có output ReID nên chưa xác định được frame/ID.

**5. Một chỗ ReID làm xem lại annotation:** chưa có output ReID nên chưa xác định được.

## 6. Nếu phải gán thêm 10 clip nữa

Giữ ba luật đã ghi trong `GUIDELINE_MINI.md`: che dưới 25 frame thì giữ ID, ra khỏi
khung rồi quay lại mở ID mới, và bắt đầu ở frame đầu tiên xác định được xe bốn bánh.
Quy trình sẽ luôn gồm ba lượt tua, kiểm tra frame đầu/cuối, kiểm tra giữa các keyframe,
chạy validator, khóa pre-gold rồi mới xem reference/model. Đặc biệt phải ghi lại
reviewer và các finding theo frame-ID ngay trong lúc kiểm chéo.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md`
