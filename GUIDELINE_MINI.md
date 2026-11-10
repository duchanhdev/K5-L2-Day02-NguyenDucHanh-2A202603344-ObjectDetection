# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con chở người: sedan, SUV, taxi, xe bán tải cabin nhỏ. Xe đang đỗ vẫn là `car`. | Taxi vàng trong `drive_018`. Không dùng cho xe buýt, xe tải thùng hay mô tô. |
| `truck` | Xe tải chở hàng hoặc xe đầu kéo, có thùng hàng hoặc cabin tách khỏi khoang chở. | Xe tải thùng kín bên phải đường trong `drive_001`. Xe bán tải cabin nhỏ để ở `car`. |
| `bus` | Xe buýt hoặc minibus chở khách, thân dài, nhiều cửa sổ hành khách. | Xe buýt bị cắt mép trên trái trong `drive_044`, và xe buýt khớp nối trong `drive_058`. |
| `van` | Xe van hoặc minivan, nóc cao hơn sedan nhưng ngắn hơn xe buýt, dùng chở hàng hoặc chở người. | Hai xe sáng nóc cao ở làn gần trong `drive_044`. Không nâng van thành `bus` chỉ vì nóc cao. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: bốn cạnh ôm phần thân xe nhìn thấy, không lấy mặt đường, vạch kẻ, trời hay xe bên cạnh.
- Vật thể bị che khuất: vẫn vẽ một box quanh phần nhìn thấy nếu còn nhận ra lớp. Không gộp hai xe chồng nhau thành một box.
- Vật thể bị cắt bởi rìa ảnh: box chạm đúng mép ảnh, không đoán phần thân nằm ngoài khung.
- Vật thể rất nhỏ/mơ hồ: không gán khi không chắc lớp. Người đi bộ, xe đạp, mô tô và hình xe trên biển quảng cáo không thuộc bốn lớp.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001` | Cùng cảnh có sedan, xe van nóc cao gần dãy nhà và xe tải thùng kín. | Sedan là `car` (0). Van nóc cao là `van` (3). Xe tải thùng là `truck` (1). | Ba lớp mô tả loại xe, không mô tả xe đang đỗ hay đang chạy. |
| `drive_044` | Hai xe sáng nóc cao ở làn gần trông giống xe khách nhỏ; cuối đường có nhiều chấm xe. | Hai xe gần là `van` (3). Chỉ phần thân dài bị cắt ở mép trên trái mới xét `bus` (2). Chấm xa không chắc lớp thì bỏ. | Van khác bus ở chiều dài thân và số cửa sổ hành khách. Không đoán lớp khi vật thể chỉ còn vài pixel. |
| `drive_018` | Taxi vàng bị kéo mờ, vài mô tô ở lề, và một dãy xe rất nhỏ phía xa. | Taxi là `car` (0). Mô tô không gán. Xe xa không phân được cabin với thùng hàng thì không gán `truck`. | Schema chỉ có bốn lớp xe. Mô tô nằm ngoài scope. Không tự đoán lớp khi phần nhìn thấy không đủ. |
