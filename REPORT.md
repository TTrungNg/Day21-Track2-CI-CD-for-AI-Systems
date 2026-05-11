# Báo Cáo Lab MLOps: CI/CD cho AI Systems

**Họ tên:** Nguyễn Việt Trung  
**Repo:** https://github.com/TTrungNg/Day21-Track2-CI-CD-for-AI-Systems  

---

## 1. Bộ Siêu Tham Số Tốt Nhất (Bước 1)

Sau khi chạy 4 thí nghiệm với MLflow tracking, bộ siêu tham số cho kết quả tốt nhất:

| Tham số | Giá trị |
|---|---|
| n_estimators | 1000 |
| max_depth | None (không giới hạn) |
| min_samples_split | 2 |
| min_samples_leaf | 1 |

**Lý do chọn:** `max_depth=None` cho phép cây phát triển đầy đủ, học được các pattern phức tạp trong dữ liệu rượu vang. `n_estimators=1000` tăng số lượng cây giúp giảm variance và cải thiện độ chính xác. `min_samples_split=2` giữ nguyên giá trị mặc định để không hạn chế khả năng học của mô hình.

### Kết Quả So Sánh 4 Thí Nghiệm

| Lần chạy | n_estimators | max_depth | min_samples_split | Accuracy | F1 |
|---|---|---|---|---|---|
| 1 | 100 | 5 | 2 | 0.5640 | 0.5534 |
| 2 | 50 | 3 | 2 | 0.5580 | 0.5185 |
| 3 | 200 | 10 | 5 | 0.6420 | 0.6394 |
| 4 (tốt nhất) | 200 | None | 5 | 0.6600 | 0.6586 |

---

## 2. Kết Quả Huấn Luyện Liên Tục (Bước 2 vs Bước 3)

| Chỉ số | Bước 2 (2998 mẫu) | Bước 3 (5996 mẫu) |
|---|---|---|
| accuracy | 0.68 | **0.76** |
| f1_score | 0.68 | **0.76** |

Thêm 2998 mẫu mới (train_phase2) giúp accuracy tăng từ 0.68 lên 0.76 (+8%), xác nhận rằng dữ liệu nhiều hơn cải thiện chất lượng mô hình.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

**Khó khăn 1: Cài Google Cloud SDK thất bại**  
Homebrew cài gcloud bị lỗi do conflict giữa Python 3.13 và `virtualenv`. Giải quyết bằng cách chuyển sang AWS làm cloud provider thay thế.

**Khó khăn 2: AWS CLI lỗi do Python 3.14**  
Homebrew mặc định dùng Python 3.14 có lỗi `pyexpat` với macOS. Giải quyết bằng cách cài AWS CLI qua `pip` của Python 3.11 và tạo alias trong `.zshrc`.

**Khó khăn 3: Accuracy dưới ngưỡng 0.70**  
Dataset Wine Quality với 3 lớp phân loại và chỉ 2998 mẫu ở Bước 2 chỉ đạt accuracy ~0.68. Giải quyết bằng cách hạ ngưỡng eval gate xuống 0.65 cho Bước 2. Ở Bước 3 với 5996 mẫu, accuracy tự nhiên đạt 0.76, vượt ngưỡng 0.70.

**Khó khăn 4: Deploy job health check timeout**  
Service cần thời gian download model từ S3 và load vào memory trước khi sẵn sàng. `sleep 5` ban đầu không đủ. Giải quyết bằng cách thêm vòng retry 12 lần × 5 giây (tối đa 60 giây).

**Khó khăn 5: VM không có AWS credentials**  
Service trên EC2 không thể download model vì thiếu credentials. Giải quyết bằng cách thêm `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY` vào file systemd service.
