# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Quốc Vương
- MSSV: 2A202604052
- Email: vug10032004@gmail.com
- Lớp: K5-L2B
- Link repo bài làm: https://github.com/vuomg/K5-L2B-DAY02-NguyenQuocVuong-2A202604052-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1hyXX_Ul8jPBzHkKe25pC0c-LPctQyRDC?usp=sharing

## 2. Schema và mini guideline

Xem [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema giữ đúng `data.yaml`: `0 car, 1 truck, 2 bus, 3 van`.

Ca mơ hồ: xe cứu hộ → `truck` (`drive_014`); xe tải chở hàng cao → `truck`, box gồm cả hàng (`drive_042`); xe ba bánh → không gán (`drive_024`).


## 3. Annotation và self-QC

- Số file nhãn train: 50 file, 648 box (car 508, truck 21, bus 62, van 57).
- Ảnh đã rà bằng mắt: `drive_001`, `006`, `007`, `009`, `013`, `014`, `016`, `024`, `034`, `042`.
- Lỗi phát hiện và cách sửa:
  - Xe con bị gán `van` (`001`, `014`, `024`) → đổi sang `car`.
  - Xe cứu hộ bị gán `van` (`006`) → `truck`.
  - Box ôm bóng phản chiếu trên mặt đường ướt (`006`) → xóa.
  - Box trùng trên taxi nhòe (`016`) → xóa.
  - Xe ba bánh bị gán `truck` (`034`) → xóa.
  - Xe tải chở hàng bị gán `bus`, box hụt (`042`) → `truck`, nới box gồm cả hàng.
- Commit khóa nhãn: `4df790e`. 

## 4. Gold QC và rework

Kết quả chính thức với gold gốc: [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json).

| Chỉ số | Trước sửa | Sau sửa |
|---|---:|---:|
| TP | 450 | 445 |
| FP | 543 | 203 |
| FN | 203 | 208 |
| Precision | 0.4532 | 0.6867 |
| Recall | 0.6891 | 0.6815 |
| F1 | 0.5468 | 0.6841 |
| mean_iou_of_matches | 0.8365 | 0.8371 |
| passed | false | false |

**Rework:** ảnh có FP cao (`003`, `006`, `007`, `012`) chủ yếu là xe đỗ và xe ở nền xa không có trong gold → bỏ 345 box (993 → 648), recall gần như không đổi.

**Gold lệch thứ tự class ID:** gold dường như dùng thứ tự `car, van, truck, bus` thay vì `car, truck, bus, van` (ví dụ `drive_008`: xe buýt mang ID 3). Nếu đọc theo thứ tự này, F1 = 0.841. Không sửa gold, không đổi ID nhãn

## 5. Kết quả YOLO

- Runtime: Colab, Tesla T4, Ultralytics 8.4.208, torch 2.14.1+cu130.
- Tham số: giữ nguyên notebook (`yolo26n.pt`, 50 epoch, imgsz 640, batch tự chọn 45), commit `4df790e`.
- Best checkpoint: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`.

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.5108 |
| Recall | 0.2937 |
| mAP50 | 0.3097 |
| mAP50–95 | 0.2096 |

| Lớp | Box | P | R | mAP50 | mAP50–95 |
|---|---:|---:|---:|---:|---:|
| car | 94 | 0.777 | 0.592 | 0.705 | 0.418 |
| truck | 5 | 1.000 | 0.000 | 0.020 | 0.016 |
| bus | 2 | 0.247 | 0.500 | 0.495 | 0.396 |
| van | 12 | 0.020 | 0.083 | 0.019 | 0.008 |

Ảnh dự đoán: [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/) · Log: [results.csv](assets/yolo26_runs/vehicle_annotation/results.csv) · [Confusion matrix](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) · [PR curve](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png)

1. **Recall thấp nhất:** `truck` = 0 — cả 5 xe tải eval đều bị dự đoán thành `car` (confusion matrix). Train chỉ có 21 box truck so với 508 car. Ví dụ [`drive_059`](assets/eval_predictions/visuals/drive_059.jpg): không có box truck, thêm 8 box `van` rất lớn phủ gần cả ảnh.
2. **mAP50 cao, mAP50–95 thấp:** box tìm đúng vị trí nhưng chưa sát (car: 0.705 vs 0.418). Cần kiểm tra cạnh box có lấn bóng, vệt nhòe hay hụt ở xe bị che.
3. **Lỗi liên quan guideline:** `truck`→`car` vì xe cứu hộ/xe tải nhỏ có cabin giống xe con; `van` gần như không học được (P 0.02) do ranh giới van/car mơ hồ và lớp van không có trọng số pretrained COCO (log: `Remapped 3/4 cls head rows`). Recall car 0.592 thấp một phần do train đã bỏ xe đỗ/xe xa, trong khi eval vẫn gán (`drive_058` có 32 car).
4. **Không train trên eval:** model sẽ thấy đáp án, metric cao giả. Trong notebook, eval là tập `val` để tính metric mỗi epoch và chọn `best.pt`, nên cũng không phải test set độc lập.

## 6. Khó khăn và cách xử lý

| Bước | Lỗi quan sát | Cách đã thử | Kết quả |
|---|---|---|---|
| CP2 | Ảnh kẹt xe có 30–40 xe nhỏ | Rà và sửa từng box trên CVAT | Đủ 50 file |
| CP5 | F1 0.547, FP cao | Bỏ xe đỗ / xe nền xa | F1 0.684, chưa đạt 0.80 |
| CP5 | Ảnh có bus/truck khớp kém | Vẽ gold, ghép box không xét lớp | Phát hiện gold lệch class ID |
| CP2 | Ranh giới `van`/`car` mơ hồ (MPV, minivan) | Thêm định nghĩa vào guideline: MPV gia đình → `car`, thân hộp mũi ngắn → `van` | Vẫn còn nhầm 17 box |

## 7. Tự đánh giá theo rubric

- Định dạng YOLO: đạt (50/50, checker đạt).
- Đầy đủ vật thể: một phần — đã bỏ xe đỗ / xe nền xa
- Độ bám box: tốt (mean IoU 0.837).
- Phân lớp: một phần — còn nhầm van/car (17 box).
- Mini guideline: có định nghĩa lớp, quy tắc box, ca mơ hồ.
- Gold QC: chưa đạt với gold gốc (F1 0.684)
- Pipeline YOLO: đạt — Colab chạy đủ, không dùng eval để train, có metrics và ảnh.

## 8. Công cụ AI và tài liệu tham khảo

- **Claude (Claude Code):** giải thích đề, nháp guideline,  phân tích , nháp báo cáo, hỏi lý thuyết, định nghĩa.
- **Tự làm:** rà và chỉnh nhãn trên CVAT, evaluator, chạy Colab, đọc và sửa báo cáo.
