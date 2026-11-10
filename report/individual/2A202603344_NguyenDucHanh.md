# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Đức Hạnh
- MSSV: 2A202603344
- Email: duchanhdev@gmail.com
- Lớp: chưa có thông tin
- Link repo bài làm: https://github.com/duchanhdev/K5-L2-Day02-NguyenDucHanh-2A202603344-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1I_aDUcPpKYLehFJVP5jB3ULCuiMZbl9n

## 2. Schema và mini guideline

Định nghĩa lớp và ba ca mơ hồ nằm ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

Ba quyết định đã ghi:

1. `drive_001`: sedan là `car`, xe nóc cao là `van`, xe tải thùng kín là `truck`.
2. `drive_044`: hai xe sáng nóc cao ở làn gần là `van`; chỉ thân dài bị cắt mép mới xét `bus`; chấm xa không chắc lớp thì không gán.
3. `drive_018`: taxi vàng là `car`; mô tô không gán; xe xa không phân được loại thì không gán `truck`.

Điểm cần giảng viên nhìn lại là ngưỡng “đủ lớn để nhận lớp” với xe ở cuối đường. Bộ nhãn hiện gán nhiều box nhỏ hơn gold.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file `drive_001.txt` … `drive_050.txt` trong `dataset/labels/train/`.
- Kết quả checker: `python tools/check_yolo_labels.py` in `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`. Không có lỗi định dạng phải sửa.
- Ảnh đã mở và rà bằng mắt trong lần kiểm này: `drive_001`, `drive_018`, `drive_028`, `drive_044`. Chưa mở đủ 10 ảnh.
- Lỗi thấy khi rà: `drive_001` có xe tải và van đúng hướng schema, nhưng file có 17 box trong khi gold chỉ có 5. `drive_018` và `drive_028` gán thêm xe đỗ dọc lề và xe rất xa. `drive_044` có hai van gần đúng id `3`, đồng thời có box id `2` trên xe ở xa.
- Commit khóa nhãn ban đầu: `f6270aa` (`Add 50 File Label`).

## 4. Gold QC và rework

Nguồn: [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json). Chưa có vòng sửa nhãn sau gold, nên cột sau sửa để trống.

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 461 | |
| FP | 504 | |
| FN | 192 | |
| Precision | 0.4777 | |
| Recall | 0.7060 | |
| F1 | 0.5698 | |
| mean_iou_of_matches | 0.8480 | |
| passed | false | |

Mean IoU của các box đã ghép là 0.848, qua ngưỡng 0.70. F1 0.5698 chưa qua ngưỡng 0.80. Các box khớp thì sát cạnh. Phần thiếu chủ yếu là box thừa và lệch lớp: nhãn train có 98 box `bus` trong khi gold train có 20 `bus` và 54 `van`; khoảng 54 box id `2` đè lên vị trí gold gọi là `van`. Ảnh thừa nhiều box gồm `drive_044` (29 so với 8), `drive_018` (12 so với 2) và `drive_028` (18 so với 3). Chưa sửa file nhãn sau lần chấm này.

## 5. Kết quả YOLO

- Runtime: Google Colab, GPU Tesla T4 (14913 MiB), `device` 0. Ultralytics 8.4.208, Python 3.13.16, Torch 2.14.1+cu130.
- Tham số: giữ nguyên notebook (`yolo26n.pt`, 50 epoch, `imgsz` 640, `batch=-1`, `seed=20260909`). Không đổi `data.yaml` để đưa eval vào train.
- Best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`
- Colab xác nhận `OK: 50 train + 10 eval` và commit được clone là `f6270aa`.

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4090642760207564 |
| Recall | 0.23393606659813174 |
| mAP50 | 0.2706670034721526 |
| mAP50–95 | 0.1995504032766255 |

Bốn số lấy từ JSON in ra sau cell `best_model.val` trên notebook Colab. Theo lớp ở cùng cell: `car` recall 0.652 (94 vật thể), `truck` 0.200 (5), `van` 0.083 (12), `bus` 0.000 (2).

Mười ảnh dự đoán đã giải nén trong `assets/eval_predictions/visuals/`: [drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg), [drive_052.jpg](assets/eval_predictions/visuals/drive_052.jpg), [drive_053.jpg](assets/eval_predictions/visuals/drive_053.jpg), [drive_054.jpg](assets/eval_predictions/visuals/drive_054.jpg), [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg), [drive_056.jpg](assets/eval_predictions/visuals/drive_056.jpg), [drive_057.jpg](assets/eval_predictions/visuals/drive_057.jpg), [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg), [drive_059.jpg](assets/eval_predictions/visuals/drive_059.jpg), [drive_060.jpg](assets/eval_predictions/visuals/drive_060.jpg).

Log train theo epoch nằm ở [assets/yolo26_runs/vehicle_annotation/results.csv](assets/yolo26_runs/vehicle_annotation/results.csv). Ảnh chỉ số của lần val nằm ở [confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) và [BoxPR_curve.png](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png). Bốn số trong bảng trên vẫn lấy từ JSON của cell `best_model.val`. Dòng epoch 50 trong `results.csv` là precision 0.20333, recall 0.27606, mAP50 0.24660, mAP50–95 0.16401, không dùng thay cho bảng đó.

Ảnh đã mở cho hai lớp này:

- [drive_052.jpg](assets/eval_predictions/visuals/drive_052.jpg): box `bus 0.49` nằm trên xe buýt hai tầng giữa ảnh. File eval không có `bus`; vật thể lớn giữa ảnh được ghi `van`.
- [drive_053.jpg](assets/eval_predictions/visuals/drive_053.jpg): box `bus` nằm trên xe buýt xanh phía gần và trên xe khách đỏ-trắng góc trên phải. File eval không có `bus`.
- [drive_056.jpg](assets/eval_predictions/visuals/drive_056.jpg): box `bus 0.53` nằm trên xe khách phía xa. Box `van 0.33` nằm trên sedan đỏ bên phải.
- [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg): hai box `bus` nằm trên hàng xe buýt bên trái. File eval chỉ có một `bus` ở mép phải; nhiều xe bên trái được ghi `van`.
- [drive_060.jpg](assets/eval_predictions/visuals/drive_060.jpg): box `bus 0.88` nằm trên xe buýt trắng-xanh phía xa. File eval không có `bus`.
- [drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg): không có box `bus`. Box `van 0.99` nằm trên sedan đỏ phía gần. File eval có một `bus` ở góc trên phải.
- [drive_054.jpg](assets/eval_predictions/visuals/drive_054.jpg): box `van 0.97` nằm trên sedan đỏ phía gần.
- [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg): năm box, tất cả đều là `car`.

[confusion_matrix_normalized.png](assets/yolo26_runs/vehicle_eval/confusion_matrix_normalized.png): 0.92 gold `van` bị dự đoán thành `bus`, không có ô đúng `van`–`van`. Gold `bus` thì 0.50 được dự đoán thành `bus` và 0.50 thành `car`.

Trả lời bốn câu hỏi cuối notebook:

1. Trong bảng cell val, recall thấp nhất vẫn là `bus` (0.000, 2 vật thể). Trên ảnh thì các box `bus` nằm trên xe buýt, như `drive_052`, `drive_053`, `drive_056`, `drive_058` và `drive_060`. Hai `bus` của nhãn eval không trùng các box đó: `drive_051` không có box `bus`, còn hai box `bus` của `drive_058` nằm bên trái trong khi nhãn `bus` nằm mép phải. Box vẽ sai lớp trên ảnh là `van`, thấy trên sedan ở `drive_051` và `drive_054`.
2. mAP50 toàn cục 0.271 và mAP50–95 0.200 đều thấp, nên đây không phải ca mAP50 cao. Khoảng cách rõ hơn nằm ở `car`: mAP50 0.740 so với mAP50–95 0.524. Cần kiểm độ sát của cạnh box, vì nhiều box qua IoU 0.50 nhưng không giữ được ở ngưỡng cao hơn.
3. Có lệch lớp, và trên ảnh eval lệch rõ ở `van`. Ma trận chuẩn hóa đưa 0.92 gold `van` sang dự đoán `bus`. `car` vẫn chiếm 94/113 vật thể và recall 0.652. Nhãn train lệch cùng hướng: nhiều box `bus` nằm trên xe gold gọi là `van`. Guideline phân biệt van nóc cao với bus thân dài; bộ nhãn đã nộp chưa theo đều quy tắc đó.
4. Mười ảnh eval có nhãn nhưng là tập `val` trong `data.yaml`. Notebook dùng chúng để chọn `best.pt` trong lúc train và để báo cáo mAP. Đưa chúng vào train làm checkpoint được chọn trên chính ảnh dùng để chấm. Split được giữ nguyên.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Self-QC trên PowerShell | `python3` không có; lệnh nhiều dòng với `\` trong GUIDE không chạy | Chạy `python tools/check_yolo_labels.py` một dòng | Checker đạt 50/50 |
| Gold QC | F1 0.5698, precision 0.4777 | Chạy `evaluate_annotations.py` với `gold_train_labels/` | Mean IoU đạt. Chưa sửa nhãn để kéo F1 lên 0.80 |
| Lấy kết quả Colab | Runtime đã ngắt trước khi tải zip trong phiên đầu | Giải nén hai folder vào `report/individual/assets/` | Có 10 ảnh `drive_051`–`drive_060`, `results.csv` và ảnh trong `vehicle_eval`. Bốn chỉ số chính vẫn lấy từ cell val |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):

- Định dạng YOLO: có 50/50 file và checker đạt.
- Đầy đủ vật thể và độ bám: mean IoU của box khớp là 0.848, nhưng FP 504 và FN 192. Chưa đạt mức bao phủ của gold.
- Phân lớp: `car` khớp phần lớn. `bus`, `van`, `truck` lệch nhau.
- Mini guideline: đã có định nghĩa bốn lớp, quy tắc box và ba ca có tên ảnh.
- Gold QC: có file JSON. F1 chưa đạt 0.80 và chưa có vòng rework.
- Pipeline: Colab chạy đủ 50 epoch trên T4, không đưa eval vào train, có bốn chỉ số và nhận xét. `assets/eval_predictions/` và `assets/yolo26_runs/` đã có trong thư mục báo cáo.

## 8. Công cụ AI và tài liệu tham khảo

Cursor.
