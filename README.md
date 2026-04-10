# Hệ Thống Gợi Ý Địa Điểm Du Lịch Việt Nam
**Mô hình: Random Forest Recommendation System**

---

## Tổng Quan

Dự án này xây dựng một **hệ thống gợi ý địa điểm du lịch** dựa trên **Random Forest Classifier**, giúp dự đoán và gợi ý các địa điểm du lịch phù hợp nhất (nhà hàng, khách sạn, điểm tham quan) dựa trên các bình luận và thông tin từ người dùng. Hệ thống ra quyết định từ 200 cây quyết định (decision trees) và sử dụng **TF-IDF vectorization** để tiếp nhận và phân tích văn bản bình luận.

**Đầu ra chính:**
- Top 3 Nhà Hàng (Dining)
- Top 2 Khách Sạn (Hotel)  
- Top 1 Điểm Tham Quan (Attraction)

---

## Bài Toán

Bài toán được phát biểu dưới dạng **phân loại đa lớp (Multi-class Classification)**:

**Input:** Bình luận/mô tả từ người dùng + một số thông tin đặc trưng (điểm đánh giá, giá cả, v.v.)  
**Output:** Dự đoán địa điểm (lớp) phù hợp nhất + xác suất dự đoán

---

## Dữ Liệu

### Nguồn Dữ Liệu
Dữ liệu được thu thập từ các nền tảng du lịch phổ biến:
- **Google Maps** - Địa điểm, đánh giá, bình luận
- **Booking.com** - Khách sạn, giá cả, nhận xét
- **Foody.vn** - Nhà hàng, điểm đánh giá, bình luận

Tập trung vào **Thành Phố Hồ Chí Minh** với các danh mục:
- **Hotel** - Khách sạn lưu trú
- **Dining** - Nhà hàng, quán ăn  
- **Attraction** - Điểm tham quan du lịch

### Đặc Trưng (Features)

| Cột | Mô Tả | Kiểu Dữ Liệu |
|-----|-------|------------|
| **Name** | Tên địa điểm | String |
| **Address** | Địa chỉ | String |
| **Category** | Loại địa điểm (Hotel/Dining/Attraction) | Categorical |
| **Score** | Điểm đánh giá (0-5 sao) | Float |
| **Nums_comments** | Số lượng bình luận | Integer |
| **Price_min, Price_max** | Khoảng giá (VNĐ) | Integer |
| **Comments** | Danh sách bình luận | List[String] |


### Kích Thước Dữ Liệu
- ~6,000+ địa điểm ban đầu
- ~130,000+ bình luận sau khi phân tách (explode)
- 3 danh mục chính: Attraction, Hotel, Dining

---

## Pipeline - Quy Trình Xử Lý

### **Bước 1: Tiền Xử Lý Dữ Liệu**

**1.1 Tổng Hợp Dữ Liệu**
- Thu thập các CSV từ nhiều nguồn
- Gộp thành một tập dữ liệu thống nhất (`data.csv`)

**1.2 Chuẩn Hóa & Làm Sạch**
- **Tên & Địa chỉ:** Loại bỏ ký tự đặc biệt, chuẩn hóa Unicode (NFC)
- **Điểm (Score):** Chuyển thành float, thay thế giá trị lỗi bằng 0
- **Giá (Price):** Tách thành `Price_min` và `Price_max`
- **Giờ (Hours):** Tách thành `Start` và `End` 
- **Bình luận:** Chuẩn hóa định dạng, loại bỏ ký tự xuống dòng
- **Category:** Viết hoa, loại bỏ khoảng trắng

**1.3 Phi Bình Luận (Explode Comments)**
- Mỗi bình luận trở thành một dòng riêng biệt
- Tạo bình luận tổng hợp cho địa điểm thiếu dữ liệu

### **Bước 2: Chia Dữ Liệu & Label Encoding**

**2.1 Custom Train/Valid/Test Split**
- **Nếu ≥ 6 bình luận:** Chia 70% Train (A) / 15% Valid (B)/ 15% Test (C)
- **Nếu < 6 bình luận:** 100% Train (để không làm mẫu quá nhỏ)
- **3-Phase Training Strategy:**
  - Giai đoạn 1: Train trên A, đánh giá trên B
  - Giai đoạn 2: Train trên A+B, đánh giá trên C
  - Giai đoạn 3: Train trên A+B+C (mô hình cuối)
  Bộ siêu tham số được chọn để huấn luyện đã được chọn lọc qua nhiều lần thử nghiệm.

**2.2 Mã Hóa Nhãn (Label Encoding)**
- Chuyển tên địa điểm thành số nguyên (0, 1, 2, ...)
- Lưu mapping để sử dụng lại

### **Bước 3: Trích Xuất Đặc Trưng (Feature Engineering)**

**3.1 Đặc Trưng Văn Bản - TF-IDF**
- **Tiền xử lý văn bản:** Chuyển thành chữ thường, loại bỏ ký tự đặc biệt
- **Vectorization:** Sử dụng TF-IDF với:
  - **Max features:** 5000 từ/cụm từ quan trọng nhất
  - **ngram_range=(1,2):** Bắt từ đơn và cụm 2 từ
  - **min_df=2:** Loại bỏ từ xuất hiện quá ít
  - **max_df=0.95:** Loại bỏ từ xuất hiện quá phổ biến

**3.2 Đặc Trưng Số - Normalization**
- **4 đặc trưng số học:** Score, Nums_comments, Price_min, Price_max
- **Chuẩn hóa:** StandardScaler để đưa về cùng tỷ lệ

**3.3 Kết Hợp Đặc Trưng (Feature Stacking)**
- Gộp TF-IDF (5000 chiều) + số học (4 chiều) = **5004 đặc trưng tổng**
- Sử dụng sparse matrix để tiết kiệm bộ nhớ

### **Bước 4: Huấn Luyện Mô Hình**

Xem mục **Mô Hình** dưới đây.

### **Bước 5: Đánh Giá & Inference**

**5.1 Metrics Đánh Giá**
- **Accuracy:** Tỷ lệ dự đoán đúng
- **Macro F1:** F1 trung bình cho tất cả lớp
- **Top-5 Accuracy:** Xác suất địa điểm đúng nằm trong Top 5 dự đoán

**5.2 Inference (Dự Đoán)**
- Tiếp nhận query từ người dùng
- Vectorize & dự đoán xác suất cho từng loại địa điểm
- Lọc theo danh mục (**lấy riêng Top 3 Dining, Top 2 Hotel, Top 1 Attraction**)
- Trả về địa điểm với xác suất cao nhất

---

## Mô Hình

### Kiến Trúc Random Forest

```
Input Data (5004 features)
    ↓ (distributed to 200 trees)
┌─────────────────────────────────┐
│   200 Decision Trees (Parallel) │
├─────────────────────────────────┤
│  Tree 1 │ Tree 2 │ ... Tree 200 │
└─────────────────────────────────┘
    ↓ (voting/averaging)
Majority Vote (Classification)
    ↓
Probability Distribution
    ↓
Top-K Recommendations
```

### Tham Số Mô Hình

```python
RandomForestClassifier(
    n_estimators=200,          # 200 cây quyết định
    max_depth=14,              # Độ sâu tối đa, kiểm soát overfitting
    min_samples_split=30,      # Số min để split node
    min_samples_leaf=5,        # Số min ở lá của cây
    oob_score=True,            # Đánh giá tự động bằng Out-Of-Bag samples
    n_jobs=-1,                 # Dùng tất cả CPU cores
    random_state=42            # Tái tạo được kết quả
)
```

### Lý Do Chọn Random Forest

1. **Xử lý dữ liệu hỗn hợp:** Văn bản + số học hiệu quả
2. **Giải thích được (Interpretable):** Cung cấp feature importance
3. **Nhanh:** Training & inference nhanh, không cần GPU
4. **Robust:** Xử lý dữ liệu không cân bằng tốt
5. **OOB Evaluation:** Tự động đánh giá mô hình
6. **Dễ deploy:** Đơn giản, dễ maintain, dễ debug


### 3. Dự Đoán Cho Query Mới

```python
# Query từ người dùng
user_query = "Tôi muốn ăn cơm tấm ngon, giá rẻ, tại quận 1"

# Load model đã train
model = joblib.load('models/rf_model.pkl')
vectorizer = joblib.load('models/tfidf_vectorizer.pkl')

# Vectorize query
query_vec = vectorizer.transform([user_query])

# Dự đoán xác suất
proba = model.predict_proba(query_vec)

# Lấy Top-3 gợi ý
top_3_indices = proba.argsort()[0][-3:][::-1]

# Trả về tên địa điểm
places = label_encoder.inverse_transform(top_3_indices)
print("Gợi ý địa điểm:", places)
```


