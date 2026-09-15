# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `SOLO - Nguyễn Việt Hoàng`
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

Bổ sung của nhóm (nếu có): `không có`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới ... frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `tránh đổi ID khi che khuất ngắn` |
| Xe bị che lâu hơn ngưỡng trên | `tạo track/ID mới` | `track cũ đã quá 2 giây` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `không đủ cơ sở xác nhận là xe cũ` |
| Hai xe cắt nhau / chồng lên nhau | `giữ ID theo hướng và vị trí dự đoán` | `tránh đổi ID giữa hai xe` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `bbox ≥ 10×10 px và nhận diện chắc chắn là xe` |
| Xe đang đỗ, không di chuyển | `vẫn giữ track và ID, bbox cố định theo xe` |
| Keyframe đặt dày ở đâu | `khi xe đổi hướng/tốc độ, bbox đổi mạnh hoặc bị che khuất` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

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

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

BBox trôi: cần thêm ngưỡng cảnh báo riêng, vì IoU 0.59 vẫn bị cảnh báo dù > 0.5.
