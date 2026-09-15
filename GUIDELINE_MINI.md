# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Thái Đức Cường`
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

Bổ sung của nhóm (nếu có): chỉ gán xe bốn bánh nhìn thấy trong vùng ảnh; không suy đoán xe chỉ xuất hiện qua phản chiếu hoặc hình quảng cáo.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Giữ liên tục identity khi vẫn có đủ bằng chứng đó là cùng một xe. |
| Xe bị che lâu hơn ngưỡng trên | kết thúc track cũ; khi xuất hiện lại thì tạo ID mới | Sau khoảng che dài, không đủ bằng chứng để nối identity một cách chắc chắn. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Lần xuất hiện lại được xem là một quan sát mới, tránh nối nhầm với xe khác. |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo vị trí, hình dạng và hướng chuyển động trước khi chồng; không đổi ID chỉ vì bbox giao nhau | Ưu tiên continuity của từng xe qua vùng che khuất ngắn. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: bbox phải có phần thân xe nhìn thấy rõ, không gán chỉ từ một vài pixel hoặc bóng |
| Xe đang đỗ, không di chuyển | vẫn giữ track nếu xe còn nhìn thấy trong frame; cập nhật bbox theo biên nhìn thấy, không bỏ track vì xe đứng yên |
| Keyframe đặt dày ở đâu | đặt dày ở lúc xe vào/ra khung, bắt đầu hoặc kết thúc che khuất, hai xe cắt nhau và khi hình dạng thay đổi nhanh; đoạn chuyển động ổn định có thể thưa hơn |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 59 / ID 4`
- Tình huống: điểm chuyển identity trong kết quả ByteTrack; model đổi từ ID 14 sang 15.
- Quyết định: annotation giữ nguyên ID 4.
- Lý do: ID annotation phải theo cùng xe qua đoạn giao nhau/che khuất ngắn; model switch không phải lý do để đổi nhãn.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 94 / ID 5`
- Tình huống: ByteTrack đổi từ ID 23 sang 32 và chỉ phủ một phần track tham chiếu.
- Quyết định: giữ một track ID 5 xuyên suốt khi xe vẫn còn quan sát được.
- Lý do: ưu tiên continuity của annotation; không tách track chỉ vì tracker bị mất association trong một frame.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 169 / ID 8`
- Tình huống: ByteTrack đổi từ ID 54 sang 70 ở vùng cuối track; ReID chỉ còn một switch tại frame 87 của ID 5.
- Quyết định: giữ ID annotation theo xe thật, không dùng ID của model làm nhãn tham chiếu.
- Lý do: model output là baseline để phát hiện điểm cần xem lại; annotation cuối đối chiếu gold không có ID switch.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Khi hai xe chồng bbox, phải ghi rõ tiêu chí nối ID: vị trí, hướng chuyển động và hình dạng trước/sau vùng che; không đổi ID chỉ vì detector hoặc tracker đổi ID.
- Khi xe ra khỏi khung hoặc bị che quá 25 frame, phải kết thúc track cũ và tạo ID mới khi xuất hiện lại; không dùng appearance/model để tự động nối nếu không có bằng chứng hình ảnh liên tục.
