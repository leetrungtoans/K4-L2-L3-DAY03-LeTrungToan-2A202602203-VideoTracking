# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lê Trung Toán`
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

Bổ sung của nhóm (nếu có): chỉ giữ xe bốn bánh; không gán xe máy, người hoặc vật thể phản chiếu.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che dưới 25 frame | cùng một xe vẫn giữ identity trong khoảng occlusion ngắn |
| Xe bị che lâu hơn ngưỡng trên | mở track mới | tránh nối nhầm khi không còn đủ bằng chứng identity |
| Xe rời khung hình rồi quay lại | track mới | ra khỏi khung là kết thúc track |
| Hai xe cắt nhau / chồng lên nhau | mỗi xe giữ ID cũ; tách bbox theo phần nhìn thấy | ưu tiên continuity của chuyển động và vị trí trước/sau crossing |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu từ frame đầu tiên có thể xác định là xe bốn bánh; không đoán ở các frame chưa đủ bằng chứng |
| Xe đang đỗ, không di chuyển | vẫn giữ track trong toàn bộ thời gian còn nhìn thấy; kiểm tra để tránh cảnh báo bbox đứng im là lỗi giả |
| Keyframe đặt dày ở đâu | đặt dày khi xe rẽ, phanh, bị che hoặc gần mép ảnh; thưa hơn khi chuyển động đều |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / 80-100 / 6`
- Tình huống: pre-gold có bbox trước khi track tham chiếu 6 xuất hiện.
- Quyết định: cần kiểm tra frame bắt đầu; không tự nối nếu chưa chứng minh cùng xe.
- Lý do: diagnostic ghi nhận đây là ghost track ở đầu đoạn.

### Ca 2
- Clip / frame / ID: `clip_01 / 149-151 / 4`
- Tình huống: bbox còn sau khi track tham chiếu 4 rời khung.
- Quyết định: kết thúc track bằng outside đúng frame xe rời khung.
- Lý do: tránh bbox treo sau exit.

### Ca 3
- Clip / frame / ID: `clip_01 / 84-90 / 5`
- Tình huống: các bbox cùng ID có IoU thấp với gold; thấp nhất trong nhóm diagnostic là frame 84, IoU 0.532.
- Quyết định: đặt keyframe dày hơn ở đoạn xe bị che/đổi hướng và ôm phần nhìn thấy.
- Lý do: lỗi hình học không phải ID switch.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Khi xe ra khỏi khung, phải kết thúc track; không giữ bbox thêm các frame sau exit.
- Với đoạn bbox đứng im, phải phân biệt xe đỗ thật với quên outside bằng kiểm tra frame đầu/cuối và ảnh gốc.
