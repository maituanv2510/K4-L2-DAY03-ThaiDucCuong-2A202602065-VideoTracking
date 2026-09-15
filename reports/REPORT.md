# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Thái Đức Cường`
Ngày: `15/09`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `chưa ghi lại` phút |
| Thời gian gán `clip_01` | `chưa ghi lại` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `chưa thống kê` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che hoặc hai xe chồng lên nhau: giữ ID theo vị trí, hình dạng và hướng chuyển động; không đổi ID chỉ vì bbox giao nhau.
2. Xe ở gần rìa ảnh: đặt bbox chạm đúng rìa ảnh và không suy đoán phần xe nằm ngoài khung.
3. Xe nhỏ hoặc mờ khi mới xuất hiện: chỉ bắt đầu track từ frame đầu tiên có đủ bằng chứng đó là xe bốn bánh, không gán theo bóng hoặc vài pixel rời rạc.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: kiểm tra tính liên tục của 8 ID trong `clip_01`; không phát hiện ID trùng trong cùng frame.
- Lượt 2: kiểm tra frame bắt đầu/kết thúc và bbox tại rìa ảnh; annotation có 637 row trên frame 1–190.
- Lượt 3: kiểm tra các vùng xe cắt nhau, che khuất và chuyển động; kết quả evaluator không phát hiện fragmentation hoặc ID switch trong annotation.

Kiểm chéo với: `chưa cung cấp review partner`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `chưa có dữ liệu`. Số lỗi bạn ấy tìm được trong bản của bạn: `chưa có dữ liệu`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Chưa có biên bản peer review để xác định ca bất đồng. Quy tắc cần làm rõ đã được bổ sung trong `GUIDELINE_MINI.md`: giữ ID qua che khuất ngắn, dùng ngưỡng 25 frame, và tạo ID mới sau khi xe ra khỏi khung hoặc bị che quá ngưỡng.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `chưa có manifest` |
| Thời điểm khóa | `chưa có manifest` |
| Số row / frame / track trước khi mở reference | `chưa có pre-gold`; bản cuối có `637 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có |
| Sau rework | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 0 | 0 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Không có lỗi annotation theo evaluator | — | — | Annotation cuối khớp gold; không thực hiện sửa dựa trên model output. |
| — | — | — | Không có evidence pre-gold để đối chiếu thay đổi trước/sau. |
| — | — | — | Cần bổ sung review partner nếu có sửa thủ công cụ thể. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml / botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / 2, 5, 7` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 0 | 0 | 0 |
| ByteTrack control vs gold | 0.6782 | 0.6137 | 0.7518 | 0.8675 | 0.8344 | 0.6845 | 0.8461 | 84 | 114 | 3 |
| BoT-SORT + ReID vs gold | 0.7563 | 0.6984 | 0.8194 | 0.9101 | 0.8627 | 0.7268 | 0.9049 | 87 | 86 | 1 |
| ReID vs bạn | 0.7563 | 0.6984 | 0.8194 | 0.9101 | 0.8627 | 0.7268 | 0.9049 | 87 | 86 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Với annotation của tôi, MOTA và IDF1 đều bằng `1.0000`. Ở model, MOTA thấp hơn IDF1: ByteTrack là `0.6845` so với `0.8344`, còn ReID là `0.7268` so với `0.8627`. Điều này cho thấy model vẫn giữ identity tương đối tốt nhưng còn bỏ sót và có false positive. MOTA tập trung vào FP, FN và ID switch; một lỗi ID không làm MOTA giảm mạnh như nhiều detection miss, trong khi IDF1 phản ánh trực tiếp độ đúng của identity.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

ReID tốt hơn ByteTrack ở cả ba chỉ số: IDF1 tăng từ `0.8344` lên `0.8627`, AssA tăng từ `0.7518` lên `0.8194`, và IDSW giảm từ `3` xuống `1`. Diagnostics ghi nhận ByteTrack switch tại frame `59` của gold ID `4`, frame `94` của ID `5` và frame `169` của ID `8`; ReID còn một switch tại frame `87` của ID `5`. Vì hai cấu hình dùng hai tracker implementation khác nhau, kết quả này không cô lập causal effect của riêng ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Từ ByteTrack sang ReID, DetA tăng `0.6137 -> 0.6984`, FN giảm `114 -> 86`, nhưng FP tăng nhẹ `84 -> 87`. AssA cũng tăng `0.7518 -> 0.8194` và IDSW giảm `3 -> 1`, nên ReID treatment cải thiện association rõ rệt. Tuy nhiên FN và một phần FP vẫn cho thấy lỗi còn lại thuộc detection/coverage; IDSW và fragmentation cho thấy vẫn còn lỗi association. Không nên quy toàn bộ khác biệt cho ReID vì tracker implementation cũng thay đổi.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Tại frame `87`, gold và annotation giữ cùng identity `ID 5`, trong khi diagnostics của ReID ghi nhận switch từ model ID `17` sang `18`. Vì annotation cuối khớp gold hoàn toàn và có `IDSW = 0`, đây là evidence cho thấy model sai association chứ không phải annotation cần đổi.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Tại frame `115`, ReID có bbox khớp gold ID `6` với IoU khoảng `0.507`, là vùng cần xem lại vì hình học của model khá lỏng. Tuy nhiên annotation của tôi khớp gold với `LocA = 1.0000`, `FP = 0`, `FN = 0` và `IDSW = 0`; vì vậy evidence hiện có ủng hộ kết luận model sai bbox, không đủ lý do để sửa annotation.

## 6. Nếu phải gán thêm 10 clip nữa

Tôi sẽ giữ ngưỡng che khuất 25 frame, nhưng viết rõ hơn cách xử lý xe cắt nhau, xe ra khỏi khung và xe xuất hiện lại. Quy trình sẽ gồm: đặt keyframe dày tại lúc vào/ra khung và vùng che khuất; chạy kiểm tra ID trùng, bbox ngoài ảnh và gap trước khi export; lưu pre-gold cùng manifest trước khi xem gold hoặc model; sau đó mới đối chiếu metrics và ghi frame/ID của từng lần sửa.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
