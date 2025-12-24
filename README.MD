#  Mallorn Astronomical Object Classification

Dự án này tập trung xây dựng feature (Feature Engineering) và tối ưu hóa mô hình học máy để phân loại các đối tượng thiên văn dựa trên dữ liệu đường cong ánh sáng (light curves) và siêu dữ liệu (metadata) từ bộ dữ liệu Mallorn.

Mục tiêu là phân loại các đối tượng thiên văn thành:
- **TDE (Tidal Disruption Event – sao bị xé bởi hố đen)**  
- **non-TDE** (SN, AGN, SLSN, …)



##  Pipeline phát triển qua 4 phiên bản

Toàn bộ dự án được xây dựng theo 4 phiên bản tăng dần về độ phức tạp của feature và mô hình.


###  Phiên bản 1 

**Mục tiêu**
- Xây dựng pipeline ban đầu ổn định
- Sử dụng các đặc trưng thống kê cơ bản từ flux theo từng band
- Sử dụng tree-based model

**Feature chính**
- `band_agg_features`:
  - mean, std, min, max, median
  - percentiles (p05, p10, p25, p75, p90, p95)
  - amplitude (max-min)
  - MAD, IQR, skew, kurtosis
  - số điểm quan sát, thời gian kéo dài, khoảng cách giữa các lần đo
  - tỉ lệ flux âm
- `global_flux_features`:
  - thống kê flux trên toàn bộ band

**Mô hình**
- LightGBM baseline

**Ý nghĩa**
- Nắm bắt mức sáng trung bình, biên độ biến đổi và độ phân tán của đối tượng
- Phân biệt được:
  - TDE: thường có một đỉnh rõ, biên độ lớn
  - non-TDE: dao động nền hoặc nhiều đỉnh nhỏ

---

###  Phiên bản 2 

**Mục tiêu**
- Khai thác thông tin từ paper và notebook tạo dữ liệu MALLORN
- Đưa thêm các feature phản ánh:
  - tín hiệu/nhiễu
  - vùng quanh đỉnh sáng
  - hình dạng phân bố flux
- Thử nghiệm model tree-based mới

**Feature chính**
- `quality_features`, `snr_ratio_sets_features`: chất lượng tín hiệu
- `peak_count_features`, `peak_coverage_features`: mật độ điểm quanh đỉnh
- `window_duration_features`: độ dài sự kiện (rest-frame)
- `percentile_diff_features`, `percentile_tail_features`
- `robust_baseline_features`
- `curve_angle_features`
- `variability_indices_features`

**Mô hình**
- LightGBM + Optuna
- Catboost 


---

###  Phiên bản 3 

**Mục tiêu**
- Tận dụng đặc thù dữ liệu: quan sát không đều, có gap
- Mô tả hình dạng tổng thể của light curve
- Tối ưu đồng thời mô hình XGBoost, LightGBM

**Feature chính**
- `global_gap_features`:
  - gap_mean, gap_std, gap_max giữa các lần quan sát
- `fast_shape_features_allbands`:
  - slope tuyến tính
  - SNR trung bình
  - số đỉnh nhanh
  - rise/decline rate
  - tỉ lệ điểm vượt 1 std

**Mô hình**
- LightGBM
- XGBoost

**Ý nghĩa**
- Giúp mô hình biết:
  - dữ liệu có bị “đứt đoạn” quanh vùng quan trọng hay không
  - đối tượng là một flare đơn lẻ (TDE) hay dao động liên tục (AGN)

---

###  Phiên bản 4 (Final)

**Mục tiêu**
- Khai thác tối đa đặc trưng vật lý của TDE
- Kết hợp màu sắc, độ trễ đỉnh, suy giảm theo power-law
- Sử dụng ensemble kết hợp XGBoost + CatBoost

**Feature chính**
- `color_evolution_features`: tiến hóa màu (u-g, g-r)
- `peak_lag_features`: độ trễ đỉnh giữa các band
- `flare_baseline_features`: tách flare và nền
- `tde_shape_features`: suy giảm ~ t^{-5/3}, bất đối xứng rise/fade
- `tde_shape_features_per_band` + `gp_features_one_band`
- `blue_dominance_features_from_wide`: TDE sáng mạnh ở band xanh
- `z-proxy features`: chuẩn hóa theo (1+z)

**Mô hình**
- **XGBoost + CatBoost** 
  - Phương pháp: Blending (Weighted Average) giữa XGBoost và CatBoost.
  - Lý do chọn lựa: Phân tích tương quan (Correlation Analysis) cho thấy XGBoost và CatBoost có độ tương quan thấp hơn so với cặp XGBoost-LightGBM, tạo ra hiệu quả Ensemble tốt nhất.
  - Kết quả: Đạt F1-Score và Logloss tốt nhất trên tập Test.

**Ý nghĩa**
- Mô hình học được:
  - TDE thường xanh hơn, suy giảm theo quy luật, flare rõ trên nền
  - non-TDE có biến thiên ngẫu nhiên, nhiều đỉnh, màu ít đặc trưng

**=> Đây là phiên bản đạt kết quả tốt nhất.**

---

##  Kết quả tổng hợp

| Phiên bản | Ý tưởng chính                     | Model     | Public LB |
|-----------|----------------------------------|-----------|------------|
| V1        | Thống kê cơ bản                  | LightGBM  | 0.3400      |
| V2        | Theo quy trình sinh dữ liệu      | CatBoost  | 0.5103      |
| V3        | Gap + fast shape                 | XGB       | 0.5518     |
| V4        | Vật lý TDE + màu       | **XGBoost+Catboost** | **0.6010** |

---

##  Tài Liệu Tham Khảo (References)

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

## Slide và thuyết trình của nhóm
[Drive](https://drive.google.com/drive/folders/1RBgrwcAhsp87xaR9yWbozhulZUBXSy8L?usp=drive_link) - Link drive.
