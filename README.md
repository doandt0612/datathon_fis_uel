# 📊 Dự báo Doanh thu & Tối ưu Vận hành (Datathon FIS@UEL_K24416)

**Đội thi:** FIS@UEL_K24416  
**Chủ đề:** Phân tích rủi ro vận hành và xây dựng mô hình dự báo doanh thu dài hạn (548 ngày).

---

## 📖 1. Tổng quan dự án (Project Overview)
Dự án giải quyết bài toán cốt lõi của doanh nghiệp bán lẻ: **Làm thế nào để cân bằng giữa việc thúc đẩy tăng trưởng doanh thu và tối ưu hóa chi phí vận hành (Logistics & Tồn kho)?** Dự án được chia làm 2 phần chính:
1. **Phân tích Kinh doanh (Business Analytics):** Bóc tách rủi ro từ phương thức thanh toán COD, giải mã "nghịch lý tồn kho" (vừa thiếu vừa thừa hàng), và đánh giá hiệu quả thực sự của các chiến dịch khuyến mãi.
2. **Dự báo Học máy (Machine Learning Forecasting):** Xây dựng kiến trúc mô hình **Hybrid Ensemble (Ridge + Prophet + LightGBM)** dự báo Doanh thu và Giá vốn (COGS), tích hợp 3 lớp rào chắn kiểm soát Data Leakage nghiêm ngặt.

---

## 📂 2. Cấu trúc dự án (Project Structure)
```
datathon_fis_uel/
├── data/
│   ├── raw
│   ├── features
│   └── processed/
│       ├── master
│       └── pre_processed
│
├── notebooks/
│   ├── 01_preprocessing.ipynb
│   ├── 02_data_merging.ipynb
│   ├── 03_eda_cancle_rate.ipynb
│   ├── 03_eda_inventory.ipynb
│   ├── 03_eda_promotion.ipynb
│   ├── 04_feature_engineering.ipynb
│   ├── 05_model.ipynb
│   ├── baseline.ipynb
│   └── MCQs.ipynb
│
├── output/
│   ├── figures
│   ├── params # Chứa tham số tối ưu Optuna của LightGBM
│   └── submission.csv (file submit chính thức)
│
├── .gitignore
├── requirements.txt
└── README.md
```

---

## ⚙️ 3. Hướng dẫn Cài đặt (Installation)

Dự án được phát triển trên Python 3.9+. Để chạy mã nguồn mà không gặp lỗi xung đột thư viện, vui lòng làm theo các bước sau:

- **Bước 1:** Clone repository

```
git clone https://github.com/doandt0612/datathon_fis_uel.git

cd datathon_fis_uel

```

- **Bước 2:** Tạo môi trường ảo (Virtual Environment)

```
python -m venv venv

# Active trên Windows:
venv\Scripts\activate

# Active trên MacOS/Linux:
source venv/bin/activate

```

- **Bước 3:** Cài đặt thư viện

```
pip install -r requirements.txt
```

---

## 🚀 4. Quy trình Thực thi (Execution Workflow)

Để tái tạo lại toàn bộ kết quả của dự án, vui lòng chạy các file Jupyter Notebook trong thư mục notebooks/ theo thứ tự đánh số:

1. **Chuẩn bị dữ liệu:** Chạy 01_preprocessing.ipynb và 02_data_merging.ipynb để làm sạch và tổng hợp dữ liệu thành bảng Master (Dữ liệu nặng nên không up lên github).

2. **Khám phá Insight:** Mở các file 03_eda_... để xem quá trình trực quan hóa Dashboard Kinh doanh và các kết luận chiến lược.

3. **Feature Engineering:** Chạy 04_feature_engineering.ipynb để sinh ra đặc trưng phục vụ mô hình dự báo.

4. **Huấn luyện và Dự báo:** Chạy 05_model.ipynb. File này chứa toàn bộ Pipeline từ việc cắt tập Validation (Out-of-Time), tối ưu SLSQP, cho đến xuất file submission.csv (file chính thức, các file submission_... khác để đánh giá kết quả base model) và vẽ biểu đồ giải thích mô hình SHAP.

---

## 🧠 5. Phương pháp Tiếp cận (Methodology)
### A. Business Insights & Hành động Chiến lược
- **Hiệu ứng COD:** Thanh toán COD tạo ra mức độ rủi ro cực lớn (tỷ lệ hủy/hoàn 25%), gây đóng băng hàng hóa trên đường vận chuyển. Đề xuất: Thiết lập cơ chế lọc rủi ro thanh toán và chuyển dịch ưu đãi sang nhóm Prepaid.

- **Nghịch lý Tồn kho:** 50.6% bản ghi tồn kho vừa báo Stockout vừa Overstock. Nhóm Fast Mover luôn thiếu hàng trong khi Slow Mover tích trữ đủ bán trong 4.7 năm. Đề xuất: Tái cấu trúc tồn kho theo cơ chế Demand-Driven, tăng Safety Stock cho nhóm AX.

- **ROI Khuyến mãi:** GP_ROI đạt -0.90x (Mỗi đồng chiết khấu làm bào mòn lợi nhuận gộp). Đề xuất: Bỏ khuyến mãi đại trà, thiết lập giá sàn (Price Floor) và chuyển sang tối ưu AOV.

### B. Machine Learning Pipeline
- **Chiến lược Feature:** Loại bỏ hoàn toàn Lag/Rolling features do Horizon dự báo quá dài. Chỉ sử dụng Calendar Features và Time-Series Trigonometry (Sin/Cos) để bắt tính lặp lại của mùa vụ.

- **Kiến trúc Hybrid Ensemble:** Kết hợp sự vững chãi của Hồi quy tuyến tính (Ridge), khả năng nhận diện chu kỳ của Prophet, và sức mạnh phi tuyến của LightGBM. Trọng số kết hợp được tối ưu hóa tự động bằng thuật toán SLSQP.

- **Kiểm soát Data Leakage:** Xây dựng 3 rào chắn bảo vệ:

    - Phân tách Train/Validation tuyến tính tuyệt đối.

    - Cô lập bộ tham số chuẩn hóa (Mean/Std) nội bộ trong tập Train.

    - Sử dụng cơ chế dự báo chéo (Out-of-Fold) để tối ưu trọng số tầng Ensemble.

---

*Dự án được thực hiện trong khuôn khổ cuộc thi Datathon được tổ chức bởi VinTelligence - VinUni Data Science & AI Club của đội thi FIS@UEL_K24416.*