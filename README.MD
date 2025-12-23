# 🌌 Mallorn Astronomical Object Classification

Dự án này tập trung xây dựng và tối ưu hóa mô hình học máy để phân loại các đối tượng thiên văn dựa trên dữ liệu đường cong ánh sáng (light curves) và siêu dữ liệu (metadata) từ bộ dữ liệu Mallorn.

## Phát Triển Mô Hình (Model Evolution)

Dự án được phát triển qua 3 giai đoạn chính, tương ứng với các notebook trong repository:

### 1. Giai đoạn Khởi tạo & Tinh chỉnh Baseline (LightGBM)
Giai đoạn này tập trung vào việc xây dựng pipeline xử lý dữ liệu và thiết lập baseline nhanh chóng.

* **`lightgbm-optunav1.ipynb`**:
    * Phiên bản khởi tạo.
    * Thiết lập pipeline xử lý dữ liệu cơ bản.
* **`lightgbm-optunav2.ipynb`**:
    * **Feature Engineering:** Bổ sung các đặc trưng thống kê từ chuỗi thời gian (time-series statistics) của light curves.
    * Tích hợp **Optuna** để tìm kiếm siêu tham số (hyperparameters) sơ bộ.
    * Cải thiện độ chính xác so với v1.
* **`lightgbm-optunav3.ipynb`**:
    * Phiên bản LightGBM tối ưu với dữ liệu xử lý thêm.

### 2. Đa dạng và thử nghiệm thêm Mô hình (Model Diversity)
Để chuẩn bị cho Ensemble, dự án mở rộng sang các kiến trúc Boosting khác nhau nhằm tạo ra sự đa dạng trong dự đoán (prediction diversity).

* **`xgboost-optunav3.ipynb`**:
    * Mô hình **XGBoost**.
    * **Đặc điểm:** Sử dụng cây bất đối xứng (Asymmetric trees) với độ sâu lớn, đóng vai trò là mô hình(Low Bias), khai thác sâu các mẫu dữ liệu phức tạp.
* **`catboost-optunav2.ipynb`**:
    * Mô hình **CatBoost**.
    * **Đặc điểm:** Sử dụng cây đối xứng (Symmetric/Oblivious trees) và xử lý biến phân loại (Categorical features) vượt trội. Đóng vai trò là mô hình(High Stability), giúp giảm nhiễu và chống Overfitting.

### 3. Giai đoạn Tổng hợp (Final Ensemble) - Best Solution
* **`xgboost-catboost.ipynb`**:
    * Đây là notebook chứa giải pháp cuối cùng.
    * **Phương pháp:** Blending (Weighted Average) giữa XGBoost và CatBoost.
    * **Lý do chọn lựa:** Phân tích tương quan (Correlation Analysis) cho thấy XGBoost và CatBoost có độ tương quan thấp hơn so với cặp XGBoost-LightGBM, tạo ra hiệu quả Ensemble tốt nhất.
    * **Kết quả:** Đạt F1-Score và Logloss tốt nhất trên tập Test.
## 📚 Tài Liệu Tham Khảo (References)

Dự án được thực hiện dựa trên sự hỗ trợ của các thư viện mã nguồn mở và các nghiên cứu sau:

1.  **Dữ liệu (Dataset):**
    * [Cuộc thi MALLORN](https://www.kaggle.com/competitions/mallorn-astronomical-classification-challenge/data) - Nguồn dữ liệu chính của dự án.

2.  **Thuật toán & Thư viện (Algorithms & Libraries):**
    * **XGBoost:** Chen, T., & Guestrin, C. (2016). [XGBoost: A Scalable Tree Boosting System](https://arxiv.org/abs/1603.02754).
    * **CatBoost:** Prokhorenkova, L., et al. (2018). [CatBoost: unbiased boosting with categorical features](https://arxiv.org/abs/1706.09516).
    * **Optuna:** Akiba, T., et al. (2019). [Optuna: A Next-generation Hyperparameter Optimization Framework](https://optuna.org/).

3.  **Kỹ thuật liên quan (Techniques):**
    * [Stratified K-Fold Cross Validation (Scikit-learn Documentation)](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.StratifiedKFold.html)
    * [Kaggle Ensembling Guide](https://www.kaggle.com/code/vipulgandhi/a-comprehensive-guide-to-ensemble-learning) - Cơ sở lý thuyết cho việc kết hợp (Blending) các mô hình.
4.  **Các tài liệu học thuật về thiên văn học:**
    * [MALLORN: Many Artificial LSST Lightcurves based on Observations of Real Nuclear transients](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.StratifiedKFold.html)

## SLide và thuyết trình của nhóm
[Drive](https://drive.google.com/drive/folders/1RBgrwcAhsp87xaR9yWbozhulZUBXSy8L?usp=drive_link) - Link drive.

---