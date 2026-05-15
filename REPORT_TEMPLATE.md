# CSC4005 Lab 4 Report – CRNN for UrbanSound8K

## 1. Thông tin sinh viên

- Họ tên: Nguyễn Trung Thành
- Mã sinh viên: 1771040022
- Lớp: (chưa cập nhật)
- Link GitHub repo: https://github.com/FIT-DNU-CS-16-01/csc4005-lab4-thanh1771040022
- Link W&B project: https://wandb.ai/trungthanhk17/csc4005-lab4-urbansound8k-crnn

## 2. Mục tiêu thí nghiệm

Lab 4 tập trung phân loại 10 lớp âm thanh UrbanSound8K bằng CRNN trên biểu diễn log-mel spectrogram.  
Log-mel giữ được thông tin thời gian–tần số quan trọng của audio, giúp mô hình học ổn định hơn dữ liệu raw.  
Khác với 1D-CNN ở Lab 3 chủ yếu học pattern cục bộ, CRNN kết hợp CNN (trích đặc trưng) và RNN (học diễn biến theo thời gian).  
Mục tiêu đánh giá là cải thiện best validation accuracy/test accuracy, đồng thời phân tích lỗi bằng learning curves và confusion matrix.

## 3. Cấu hình dữ liệu

| Thành phần | Giá trị |
|---|---|
| Dataset | UrbanSound8K |
| Số lớp | 10 |
| Train folds | 1–8 |
| Validation fold | 9 |
| Test fold | 10 |
| Feature | log-mel spectrogram |
| Sampling rate | 16 kHz |
| Duration | 4 giây |

## 4. Cấu hình mô hình

| Thành phần | Giá trị |
|---|---|
| Model | CRNN (`crnn_small`) |
| CNN blocks | 3 block Conv2d-BN-ReLU-MaxPool, channels: 16 → 32 → 64 |
| RNN type | GRU (baseline) / BiLSTM (extension) |
| Hidden size | 96 |
| Dropout | 0.30 (baseline), 0.35 (extension) |
| Optimizer | AdamW |
| Learning rate | 0.001 (baseline), 0.0007 (extension) |
| Batch size | 32 |
| Epochs | 25 |

## 5. Kết quả huấn luyện

Điền kết quả tốt nhất từ W&B hoặc `metrics.json`.

| Run | best_val_acc | test_acc | Ghi chú |
|---|---:|---:|---|
| logmel_crnn_gru_baseline | 0.7463 | 0.7551 | Kết quả tốt nhất, generalization ổn định |
| logmel_crnn_bilstm_extension | 0.6348 | 0.6882 | BiLSTM tham số lớn hơn nhưng chưa vượt baseline |

## 6. Learning curves

Chèn hình `curves.png`.

![Learning curves](outputs/logmel_crnn_gru_baseline/curves.png)

Nhận xét:

- Có overfitting nhẹ ở cuối quá trình train (train_acc tăng cao hơn val_acc), nhưng mức chênh chưa lớn.
- Validation loss giảm theo xu hướng chung (từ ~1.92 xuống tốt nhất ~0.93), có dao động giữa các epoch.
- Nên giữ early stopping (patience=6) để tránh train quá lâu khi val loss bắt đầu plateau/dao động.

## 7. Confusion matrix

Chèn hình `confusion_matrix.png`.

![Confusion matrix](outputs/logmel_crnn_gru_baseline/confusion_matrix.png)

Nhận xét:

- Phân loại tốt: `gun_shot` (recall 1.00), `jackhammer` (recall 0.896), `drilling` (recall 0.80).
- Dễ bị nhầm: `siren` ↔ `children_playing`, `engine_idling` ↔ `jackhammer`, `air_conditioner` ↔ `children_playing`.
- Nguyên nhân hợp lý là các lớp này có phổ tần và cấu trúc lặp theo thời gian khá giống nhau, cộng thêm nhiễu nền đô thị trong clip ngắn 4 giây.

## 8. So sánh với Lab 3 1D-CNN

| Tiêu chí | Lab 3: 1D-CNN | Lab 4: CRNN |
|---|---|---|
| Feature chính | MFCC / log-mel | log-mel |
| Khả năng học pattern cục bộ | Có | Có |
| Khả năng học quan hệ thời gian | Hạn chế | Tốt hơn |
| Test accuracy | 0.6387 (best run log-mel 1D-CNN) | 0.7551 (CRNN baseline GRU) |
| Nhận xét | Mô hình gọn, train nhanh nhưng khó bắt quan hệ thời gian dài | Tăng +0.1164 accuracy tuyệt đối, đổi lại thời gian train/epoch cao hơn |

## 9. Kết luận

CRNN có cải thiện rõ so với 1D-CNN ở Lab 3 trong thí nghiệm này (0.7551 so với 0.6387 trên test set).  
Kết quả baseline GRU ổn định và cho khả năng tổng quát hóa tốt hơn bản BiLSTM extension.  
Learning curves cho thấy mô hình học tiến bộ đều, dù vẫn có dao động validation loss ở các epoch cuối.  
Confusion matrix cho thấy các lớp âm thanh nền liên tục vẫn là nhóm khó phân biệt.  
Nếu làm tiếp, em sẽ cải thiện bằng: (1) tăng cường SpecAugment/cân bằng lớp, (2) tune sâu hơn cho RNN hidden size + scheduler, và (3) thử attention pooling thay cho lấy trạng thái cuối để tận dụng tốt hơn thông tin chuỗi.

## 10. Link minh chứng

- GitHub commit cuối: https://github.com/FIT-DNU-CS-16-01/csc4005-lab4-thanh1771040022/commit/feb196dd0e3518e3929678a6fd368e0103ec476e
- W&B run baseline: https://wandb.ai/trungthanhk17/csc4005-lab4-urbansound8k-crnn/runs/d8vrgpud
- W&B run mở rộng: https://wandb.ai/trungthanhk17/csc4005-lab4-urbansound8k-crnn/runs/ol2tfdgh
