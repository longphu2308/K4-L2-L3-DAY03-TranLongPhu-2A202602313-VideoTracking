# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Trần Long Phú`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): Không có; chỉ gán xe bốn bánh nhìn thấy trong cảnh.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Đây vẫn là cùng một xe nếu còn bằng chứng liên tục trước và sau che. |
| Xe bị che lâu hơn ngưỡng trên | Mở track mới nếu không thể xác nhận chắc chắn identity | Tránh nối nhầm hai xe có appearance hoặc vị trí gần nhau. |
| Xe rời khung hình rồi quay lại | **track mới** | Khi đã ra khỏi khung, track cũ kết thúc theo luật lab. |
| Hai xe cắt nhau / chồng lên nhau | Giữ ID theo chuyển động và appearance trước/sau crossing; đặt keyframe dày hơn | Không đổi ID chỉ vì hai bbox tạm thời chồng lấn. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: thấy đủ hình dạng xe và không nhầm với người/xe máy |
| Xe đang đỗ, không di chuyển | Vẫn giữ bbox và track trong toàn bộ thời gian xe còn trong khung |
| Keyframe đặt dày ở đâu | Khi xe rẽ, phanh, bị che, crossing, sát rìa ảnh hoặc bbox đổi hình; đoạn đi đều có thể đặt thưa hơn |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 59 / gold track 4`
- Tình huống: ByteTrack đổi từ track 14 sang 15 ở vùng association khó.
- Quyết định: Giữ một ID cho cùng xe; không coi lần đổi của model là lý do đổi nhãn tay.
- Lý do: Diagnostics ghi nhận ID switch tại frame 59; cần ưu tiên continuity của xe qua crossing/che khuất.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 87 / gold track 5`
- Tình huống: ReID đổi từ track 17 sang 18.
- Quyết định: Kiểm tra ảnh sequence và giữ identity nếu xe vẫn liên tục; không merge chỉ dựa trên output model.
- Lý do: Đây là ID switch do diagnostics ghi nhận, cần đối chiếu motion và appearance trước/sau frame 87.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 113 / gold track 6`
- Tình huống: ReID đổi từ track 24 sang 31 trong đoạn bbox có IoU thấp.
- Quyết định: Đặt keyframe dày hơn quanh đoạn chuyển động/che khuất và giữ cùng ID khi evidence hình ảnh liên tục.
- Lý do: Diagnostics ghi nhận switch tại frame 113; IoU thấp không tự động chứng minh xe đã đổi.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Ghi rõ frame bắt đầu/kết thúc, đặc biệt với xe nhỏ ở rìa ảnh; luôn kiểm tra lại frame giữa hai keyframe.
- Khi model và annotation khác nhau, ghi frame–ID–evidence và xem ảnh trước khi sửa; không dùng model làm gold.
