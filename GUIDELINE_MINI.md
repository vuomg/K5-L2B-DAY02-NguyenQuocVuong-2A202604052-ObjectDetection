# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con chở người: sedan, hatchback, SUV, MPV gia đình, taxi, bán tải nhỏ | Thân thấp, có capo rõ |
| `truck` | Xe chở hàng có cabin tách thùng: tải, ben, bồn, cứu hộ | Cabin riêng + thùng/sàn phía sau |
| `bus` | Xe chở khách cỡ lớn, kể cả buýt nối toa | Thân dài, nhiều cửa sổ dọc thân |
| `van` | Xe thân hộp liền, mũi ngắn: minivan/van chở hàng hoặc người | Cao hơn `car`, ngắn hơn `bus` |

## Quy tắc chung

- Phạm vi gán: lớp mô tả loại xe, không mô tả trạng thái; xe đang chạy hay đang đỗ đều gán theo loại xe.
- Box bám sát vật thể nghĩa là: chạm mép ngoài thân xe nhìn thấy, không lấy bóng, mặt đường hay xe bên cạnh; mỗi xe một box.
- Vật thể bị che khuất: gán box quanh phần nhìn thấy nếu còn nhận ra lớp; bị cột che ngang vẫn là một box.
- Vật thể bị cắt bởi rìa ảnh: box chạm rìa ảnh, không đoán phần ngoài khung.
- Vật thể rất nhỏ/mơ hồ: xe ở xa quá nhỏ hoặc mờ đến mức không xác định chắc lớp thì không tự đoán, bỏ qua và ghi lại để hỏi giảng viên. Xe bị nhòe chuyển động nhưng vẫn nhận ra lớp thì vẫn gán.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_007` | Dãy xe đỗ ven đường bên trái, kể cả xe gần camera | `car`, mỗi xe một box | Xe đỗ vẫn gán theo loại xe; schema mô tả loại, không mô tả trạng thái |
| `drive_003` | Hàng xe chờ phía sau giao lộ, ở xa và nhỏ | Gán xe còn nhận ra lớp; bỏ qua xe quá nhỏ/mờ | Không tự đoán lớp khi không xác định chắc; ghi lại để hỏi giảng viên |
| `drive_014` | Xe cứu hộ có cabin + cẩu | `truck` | Xe chuyên dụng có cabin tách rời, không chở khách |
| `drive_042` | Xe tải chất hàng cao; buýt bị cắt rìa ảnh | `truck` gồm cả hàng; `bus` chạm rìa | Hàng là một phần xe; vật cắt khung gán phần nhìn thấy |
| `drive_024` | Xe ba bánh chở hàng | Không gán | Ngoài bốn lớp, giống mô tô/xe máy |
