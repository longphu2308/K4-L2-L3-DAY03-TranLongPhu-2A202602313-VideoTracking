# Báo cáo Ngày 3 — Tracking Annotation

Báo cáo nộp bài Ngày 3.

Họ tên / nhóm: `TRẦN LONG PHÚ`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | 45 phút |
| Thời gian gán `clip_01` | 2 tiếng |
| Số track đã vẽ trong `clip_01` | 6 |
| Số keyframe trung bình mỗi track | 100 |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Giữ đúng identity khi xe bị che hoặc hai xe đi gần nhau; dùng cùng một `track_id` khi xe vẫn còn trong cảnh.
2. Xác định frame vào/ra của xe để không tạo bbox thừa sau khi xe rời khung.
3. Giữ bbox bám phần xe nhìn thấy, đặc biệt ở vùng rìa ảnh và các đoạn chuyển động nhanh.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Đã kiểm tra file MOT có 6 ID trong `clip_01`; chưa phát hiện ID trùng trong cùng frame bằng thống kê file.
- Lượt 2: Đã kiểm tra đủ 190 frame và đối chiếu phạm vi track sau khi có teaching reference; phát hiện thiếu track 7, 8 theo diagnostics.
- Lượt 3: Đã kiểm tra các đoạn có IoU thấp; diagnostics ghi nhận bbox lỏng ở các frame 84, 87, 88, 91, 104–107 và 185–190.

Kiểm chéo với: Chưa ghi nhận reviewer độc lập. File `reports/review_partner.md` chưa có trong workspace.
Số lỗi bạn tìm được trong bản của bạn ấy: Chưa có dữ liệu. Số lỗi bạn ấy tìm được trong bản của bạn: Chưa có dữ liệu.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Chưa có biên bản kiểm chéo để kết luận khác biệt giữa hai người. Sau khi đối chiếu gold, guideline cần ghi rõ tiêu chí bắt đầu/kết thúc track và cách xử lý xe bị che lâu hơn 25 frame.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `080a77e2c7929ab37c9f3f287683e81a72f480c0d07359ce1df663460c7ca22d` |
| Thời điểm khóa | `2026-09-15T09:28:25.293133+00:00` |
| Số row / frame / track trước khi mở reference | `480 / 190 / 6` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | Chưa chạy riêng | Chưa chạy riêng | Chưa chạy riêng | Chưa chạy riêng | Chưa chạy riêng | Chưa chạy riêng | Chưa chạy riêng | Chưa chạy riêng | Chưa chạy riêng | Chưa chạy riêng |
| Sau rework | 0.5457 | 0.4319 | 0.6907 | 0.8390 | 0.6363 | 0.3316 | 0.8415 | 145 | 238 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **chưa**. Chỉ MOTP đạt ngưỡng; IDF1 và MOTA chưa đạt.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bỏ sót track | 7, 8 | 7, 8 | Bổ sung/kiểm tra lại hai track bị gold tham chiếu nhưng chưa có trong annotation. |
| Bao phủ chưa đủ | 16/60, 23/56, 53/95 | 4, 5, 6 | Rà lại điểm bắt đầu/kết thúc và các đoạn bị che của các track này. |
| Bbox/entry-exit | 79–100, 69–78, 149–151; 84–107, 185–190 | 4, 5, 6 | Kiểm tra bbox thừa, bbox treo và thêm keyframe ở các đoạn IoU thấp. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / ByteTrack, BoT-SORT + ReID |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / [2, 5, 7] |
| device | CUDA `0`; `persist=True`; 190 frame |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.5457 | 0.4319 | 0.6907 | 0.8390 | 0.6363 | 0.3316 | 0.8415 | 145 | 238 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.5349 | 0.4016 | 0.7134 | 0.8617 | 0.5975 | 0.0646 | 0.8674 | 303 | 145 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA (0.3316) thấp hơn IDF1 (0.6363). Kết quả chính bị kéo xuống bởi 145 FP và 238 FN; diagnostics cũng ghi nhận thiếu track 7, 8 và nhiều track chỉ được bao phủ một phần. IDSW bằng 0 nên chưa có bằng chứng về lỗi đổi ID trong phép chấm này. MOTA không chỉ đo identity mà cộng FP, FN và IDSW, vì vậy cần đọc cùng IDF1/AssA để tách lỗi detection khỏi lỗi association.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID tốt hơn ByteTrack trên clip này: HOTA tăng 0.7085 → 0.7635, IDF1 tăng 0.8746 → 0.9001, AssA tăng 0.7761 → 0.8204 và MOTA tăng 0.7487 → 0.7923. FN giảm 54 → 26, nhưng FP tăng nhẹ 88 → 91; IDSW giữ nguyên ở 2. ReID treatment đạt cả ba ngưỡng, còn ByteTrack thiếu MOTA rất ít (0.7487 so với 0.75). Đây vẫn là system comparison vì hai tracker implementation khác nhau, không cô lập riêng tác động của ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

So với gold, ByteTrack có DetA 0.6487, FP 88 và FN 54; ReID có DetA 0.7110, FP 91 và FN 26. ReID giảm FN mạnh và tăng DetA, nhưng tạo thêm 3 FP. AssA/IDF1 cũng tăng, nên association và độ bao phủ đều cải thiện. Hai model đều có 2 IDSW, vì vậy phần cải thiện chính ở run này đến từ detection/coverage và chất lượng association tổng thể, không phải giảm số IDSW.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Một finding có thể xem là model cần kiểm tra lại là ReID sinh ghost track 29 trong frame 108–190 và ghost track 7 trong frame 16–116 theo diagnostics. Đây là các đoạn model có bbox nhưng không khớp track gold; chưa đủ bằng chứng để kết luận nhãn tay đúng ở từng frame, nên cần xem ảnh sequence trước khi sửa annotation.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID làm annotation đáng xem lại ở các đoạn mà model phủ được gold tốt hơn: ReID không bỏ sót toàn bộ track nào, FN chỉ còn 26 so với 238 của annotation; đặc biệt cần kiểm tra các track 7 và 8 mà annotation ban đầu không có. Tuy vậy, không sửa nhãn chỉ vì model khác; phải xác nhận bằng frame ảnh và rule CVAT.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Ghi rõ ngưỡng giữ ID khi che khuất, tiêu chí xác định frame xuất hiện đầu tiên và frame rời khung, cùng quy tắc đặt keyframe ở đoạn xe rẽ/chồng lấn. Quy trình nên có một lượt kiểm riêng cho entry/exit và một lượt kiểm các frame giữa keyframe trước khi export; reviewer cũng cần ghi frame–ID–evidence cho mọi finding.

### Stretch: ngưỡng appearance của ReID

| Cấu hình | HOTA | DetA | AssA | IDF1 | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| appearance = 0.70 | 0.763 | 0.711 | 0.820 | 0.900 | 91 | 26 | 2 |
| appearance = 0.80 | 0.763 | 0.711 | 0.820 | 0.900 | 91 | 26 | 2 |
| appearance = 0.90 | 0.763 | 0.710 | 0.820 | 0.899 | 91 | 27 | 2 |

Ngưỡng 0.70 và 0.80 cho cùng kết quả trên clip này. Ở 0.90, DetA và IDF1 giảm nhẹ, FN tăng từ 26 lên 27, còn IDSW không đổi. Vì vậy chưa có bằng chứng ngưỡng cao hơn giúp association tốt hơn; nên giữ cấu hình 0.80 đã khóa cho run chính.

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
