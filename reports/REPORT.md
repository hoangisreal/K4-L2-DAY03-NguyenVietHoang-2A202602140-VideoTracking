# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Việt Hoàng - SOLO`
Ngày: `3`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `20` phút |
| Thời gian gán `clip_01` | `50` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `67` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

### Ca 1
- Clip / frame / ID: `clip01/78/4`
- Tình huống: `xe bus có gương thì có nên bbox cả gương xe bus không`
- Quyết định: `box luôn cả gương`
- Lý do: `gương là vật đi liền với xe bus nên bbox luôn`

### Ca 2
- Clip / frame / ID: `clip01/189/1`
- Tình huống: `xe đứng yên thì có nên bbox không`
- Quyết định: `bbox xe đứng yên`
- Lý do: `xe đứng yên vẫn cần phải track`

### Ca 3
- Clip / frame / ID: `clip01/88/5`
- Tình huống: `xe mới đi ra vừa từ xe khác che tầm nhìn, nên track từ frame này không`
- Quyết định: `track là xe bốn bánh`
- Lý do: `vì xe lộ ra 10x10 px và nhận diện đó là xe bốn bánh`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `track không bị mất id khi tracking`
- Lượt 2: `frame đầu và frame cuối có xuất hiện và có outside không bị lệch bbox`
- Lượt 3: `đã chỉnh bbox vừa track`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Chấm với gold — trước và sau rework

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Lần chấm đầu | 0.8414 | 0.8309 | 0.8531 | 0.8879 | 0.9751 | 0.9492 | 0.8780 | 25 | 3 | 0 |
| Sau rework | 0.8414 | 0.8309 | 0.8531 | 0.8879 | 0.9751 | 0.9511 | 0.8780 | 3 | 25 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**. Sau rework: IDF1 = 0.9751, MOTA = 0.9511, MOTP = 0.8780; cả ba đều vượt ngưỡng.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox xuất hiện sớm | 79–87 | 5 | Rà lại mốc bắt đầu track và chỉ gán bbox từ khi xe đủ điều kiện nhận diện. |
| Bbox xuất hiện sớm | 106–111 | 7 | Rà lại mốc bắt đầu track và chỉnh outside/first frame cho đúng. |
| Bbox lỏng | 89 | 5 | Thêm keyframe, kéo bbox sát phần xe nhìn thấy; IoU với gold là 0.592. |

## 4. Kết quả model và so sánh ba chiều

Cấu hình: model `yolo26n.pt`, tracker `bytetrack.yaml`, conf `không có trong JSON`, imgsz `không có trong JSON`

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| me vs gold | 0.8414 | 0.8309 | 0.8531 | 0.8879 | 0.9751 | 0.9511 | 0.8780 | 3 | 25 | 0 |
| model vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| model vs me | 0.7538 | 0.6889 | 0.8267 | 0.8734 | 0.8808 | 0.7532 | 0.8570 | 95 | 39 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`mota thấp hơn idf1: 0.9511 so với 0.9751. nếu mota cao mà idf1 thấp thì phát hiện vẫn có nhưng id bị sai; mota chỉ phạt mỗi lần đổi id một lần nên không phạt theo thời gian sai.`

**2. DetA và AssA của model lệch nhau bao nhiêu? Cái nào kéo HOTA xuống — model không tìm ra xe, hay tìm ra rồi nhưng đánh mất ID?**

`deta thấp hơn assa 0.1274 (0.6487 so với 0.7761). deta kéo hota xuống nhiều hơn: model còn bỏ sót và tạo bbox dư, không chủ yếu là mất id.`

**3. Một chỗ bạn đúng và model sai (frame, ID, vì sao):**

`frame 17, id model 10: model tạo track ma không khớp gold, không gán bbox thừa.`

**4. Một chỗ model đúng và bạn sai (frame, ID, vì sao):**

`frame 56, id gold 4: model có track 14 khớp xe, track muộn nên thiếu bbox ở frame này.`

**5. Trong ba loại bất đồng giữa bạn và model, loại nào nhiều nhất? Nó nói gì về clip này?**

`bất đồng detection là nhiều nhất, đặc biệt bbox dư: model có 88 fp và 54 fn, trong khi chỉ có 2 id switch. clip có nhiều xe nhỏ, che khuất và nền dễ gây nhầm.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`bổ sung luật bắt đầu/kết thúc track, outside và xe bị che khuất vào guideline.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_clip_01.txt`
- [x] `outputs/eval_model_vs_gold.json`, `outputs/eval_model_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
