# Báo Cáo Phản Ánh — Lab CI/CD for AI Systems

**Họ tên:** Vũ Đình Phượng
**Ngày:** 25/06/2026

---

## 1. Bộ Siêu Tham Số Đã Chọn

### Kết quả thực nghiệm (Bước 1)

Mô hình sử dụng **RandomForestClassifier**, thực hiện 4 lần chạy với các bộ tham số khác nhau, ghi nhận qua MLflow:

| Lần chạy | n_estimators | max_depth | min_samples_split | Accuracy | F1 Score |
|----------|-------------|-----------|-------------------|----------|----------|
| 1        | 200         | 20        | 5                 | 0.6640   | 0.6621   |
| 2        | 300         | 25        | 2                 | 0.6760   | —        |
| 3        | 500         | None      | 2                 | 0.6760   | —        |
| 4 (*)    | 300         | 25        | 2 (+ phase2)      | **0.7580** | **0.7572** |

(*) Lần chạy 4 sử dụng kết hợp `train_phase1.csv` và `train_phase2.csv` (5.996 mẫu).

### Bộ tham số được chọn

```yaml
n_estimators: 300
max_depth: 25
min_samples_split: 2
```

**Lý do:**
- `n_estimators=300` cho độ ổn định tốt hơn so với 200 mà không tốn quá nhiều thời gian huấn luyện như 500.
- `max_depth=25` cho phép cây đủ sâu để nắm bắt các pattern phức tạp trong dữ liệu Wine Quality mà không bị overfitting quá mức.
- `min_samples_split=2` (mặc định) giúp cây phân tách tối đa, phù hợp với dataset có nhiều lớp giao nhau.
- Quan trọng nhất: kết hợp thêm `train_phase2.csv` tăng tập huấn luyện từ 2.998 lên 5.996 mẫu, đây là yếu tố chính giúp accuracy vượt ngưỡng 0.70 (từ 0.664 lên **0.758**).

---

## 2. Khó Khăn Gặp Phải và Cách Giải Quyết

### Khó khăn 1: MLflow lỗi với SQLite + MLFLOW_ARTIFACT_ROOT

**Vấn đề:** Khi chạy `python src/train.py`, MLflow ném lỗi:
```
MlflowException: The configured tracking uri scheme: 'sqlite' is invalid
for use with the proxy mlflow-artifact scheme.
```

**Nguyên nhân:** File `.env` đặt đồng thời `MLFLOW_TRACKING_URI=sqlite:///mlflow.db` và `MLFLOW_ARTIFACT_ROOT=./mlartifacts`. Khi MLflow server không chạy, biến `MLFLOW_ARTIFACT_ROOT` khiến MLflow cố dùng scheme `mlflow-artifacts://` (chỉ hoạt động với HTTP server).

**Giải pháp:** Chạy training với tracking URI dạng file thay thế:
```bash
MLFLOW_TRACKING_URI=file:///tmp/mlruns python src/train.py
```
