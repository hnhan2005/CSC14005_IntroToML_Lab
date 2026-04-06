# Walkthrough Notebook

Tài liệu này tóm tắt những gì đã được thực hiện trong thư mục notebooks, gồm ba notebook chính:

- **notebooks/collection_data.ipynb**: Thu thập dữ liệu thô từ Google Maps (Attraction), Booking (Hotel), Foody (Dining).
- **notebooks/exploration_data.ipynb**: Tổng hợp dữ liệu từ nhiều nguồn, tiền xử lý sâu, và phân tích khám phá dữ liệu (EDA).
- **notebooks/deep_learning.ipynb**: Xây dựng mô hình TextCNN dựa trên Deep Learning để xây dựng hệ thống gợi ý địa điểm dựa trên bình luận người dùng.

Lưu ý trạng thái hiện tại khi mở trong VS Code: các cell đều đang ở trạng thái chưa chạy trong phiên kernel hiện tại, nhưng notebook đã lưu sẵn đầy đủ code, mô tả quy trình, và nhiều output từ các lần chạy trước.

## 1. Tổng quan pipeline của nhóm

Pipeline chung trong notebooks được tổ chức theo thứ tự:

1. Thu thập dữ liệu từ 3 nguồn khác nhau theo 3 nhóm địa điểm.
2. Chuẩn hóa schema và gộp thành một bảng thống nhất.
3. Tiền xử lý sâu (kiểu dữ liệu, địa chỉ, giá, giờ mở cửa, bình luận).
4. Tạo dữ liệu đầu ra chuẩn để phân tích: data/processed_data.csv.
5. Phân tích khám phá dữ liệu (EDA): đơn biến, đa biến, không gian địa lý, văn bản bình luận.

Ba nhóm dữ liệu chính:

- Attraction (Google Maps)
- Hotel (Booking)
- Dining (Foody)

## 2. collection_data.ipynb đã làm gì

Notebook này tập trung vào phần Data Collection (thu thập dữ liệu thô).

### 2.1 Khởi đầu và kiểm tra robots.txt

- Mở đầu với thông tin nhóm, mục tiêu dữ liệu tại TP.HCM.
- Kiểm tra robots.txt của Google Maps, Booking, Foody bằng urllib.robotparser để xác nhận khả năng truy cập scraping ở mức URL kiểm tra.

### 2.2 Nhánh Google Maps (Attraction)

- Cài đặt thư viện chính: selenium, webdriver-manager, tqdm, ipywidgets.
- Khai báo CONFIG cho scraping:
  - max_places_per_query, max_reviews_per_place, years_back.
  - sleep ngẫu nhiên, checkpoint_interval, output_dir.
- Xây dựng danh sách SEARCH_QUERIES lớn (Anh + Việt) theo:
  - loại hình điểm tham quan.
  - theo khu vực/quận huyện TP.HCM.
- Cài đặt lớp GoogleMapsScraper gồm:
  - Khởi tạo driver và chống nhận diện automation.
  - Tìm kiếm, cuộn kết quả, lấy URL địa điểm.
  - Trích xuất từng trường: name, star, nums_comments, price, address, category, hours, comments, comment_scores, url.
  - Lấy review mới nhất, lọc theo khoảng thời gian years_back.
  - Lưu checkpoint định kỳ và lưu file cuối cùng .csv/.json.
- Thực thi vòng scrape chính với thanh tiến trình tqdm.

Kết quả đầu ra mô tả trong notebook: ggmaps.csv (và bản json).

### 2.3 Nhánh Booking (Hotel)

- Cài playwright và dùng Playwright Sync API để scraping.
- Khai báo danh sách PLACES theo nhiều quận/huyện TP.HCM.
- Khai báo ngày check-in/check-out để truy vấn kết quả phòng.
- Tổ chức thành các hàm:
  - save_to_csv: lưu từng đợt theo khu vực.
  - get_details_from_detail_page: vào từng khách sạn lấy score, review_count, address, sub-scores, comments, comment_scores.
  - scrape_booking: điều phối toàn bộ quy trình.
- Quy trình thực tế:
  - Vào trang searchresults.
  - Cuộn và bấm load more cho đến khi đủ hoặc hết dữ liệu.
  - Lấy dữ liệu card (name, url, price).
  - Truy cập trang chi tiết để bổ sung các trường còn lại.
  - Lưu file từng nơi và file tổng hợp.

Kết quả đầu ra mô tả trong notebook: Ho_Chi_Minh_Hotels_*.csv.

### 2.4 Nhánh Foody (Dining)

- Dùng Playwright Sync API cho Foody.
- Khai báo DISTRICTS gồm 24 quận/huyện/TP Thủ Đức.
- Tổ chức hàm:
  - save_to_csv.
  - get_details_from_detail_page (có intercept API response để lấy review nhanh/chính xác hơn).
  - get_branches_from_chain_page (xử lý chuỗi có nhiều chi nhánh).
  - scrape_food (hàm điều phối).
- Xử lý nhiều tình huống đặc thù:
  - popup đăng nhập.
  - lazy-loading bình luận.
  - quán ngưng hoạt động/chưa có tương tác.
  - khử trùng lặp theo URL.
- Chuẩn hóa cột và lưu file theo quận + file tổng.

Kết quả đầu ra mô tả trong notebook: tphcm_food_*.csv.

## 3. exploration_data.ipynb đã làm gì

Notebook này bao trọn phần chuẩn bị dữ liệu phân tích và EDA.

### 3.1 Tổng hợp dữ liệu

- Đọc các file CSV trong thư mục data.
- Chuẩn hóa về một header chung 10 cột.
- Bỏ file data.csv khi lặp để tránh tự đọc lại file đầu ra.
- Gộp tất cả thành data/data.csv.

### 3.2 Mô tả dữ liệu và data dictionary

- Tạo bảng mô tả thuộc tính (ý nghĩa, đơn vị, kiểu dữ liệu).
- Hiển thị shape, head, info để kiểm tra tổng quan.

### 3.3 Tiền xử lý dữ liệu

Các bước tiền xử lý quan trọng đã triển khai:

- Chuẩn hóa cột tên, địa chỉ, Unicode, khoảng trắng.
- Chuyển kiểu số cho Score và Nums_comments.
- Tách Price thành Price_min và Price_max.
- Tách Hours thành Start và End.
- Chuẩn hóa Comments về format thống nhất dạng danh sách chuỗi trong ngoặc {}.
- Parse Comment_scores từ chuỗi sang list số.
- Rút gọn URL (loại query thừa).
- Xóa trùng theo cặp Name + Address.
- Kiểm tra tính hợp lệ dữ liệu (range score, giá, số bình luận).

### 3.4 Chuẩn hóa địa chỉ bằng Gemini

- Dùng Gemini API theo batch để chuẩn hóa địa chỉ về format nhất quán.
- Xử lý theo địa chỉ duy nhất để giảm số lượt gọi API.
- Có retry và fallback giữ nguyên địa chỉ gốc nếu thất bại.
- Map kết quả chuẩn hóa ngược lại toàn bộ DataFrame.

### 3.5 Lưu bộ dữ liệu đã xử lý

- Chọn thứ tự cột cuối cùng.
- Lưu thành data/processed_data.csv.

### 3.6 Phân tích khám phá dữ liệu (EDA)

Phần EDA được làm khá đầy đủ và có nhận xét ngay sau từng biểu đồ.

1. Phân tích đơn biến:

- Cơ cấu category (pie + bar).
- Phân bố Score/Price_min/Price_max/Nums_comments (hist + KDE).
- Outlier bằng boxplot.
- Top địa điểm theo score (lọc theo số bình luận tối thiểu).
- Top địa điểm theo nums_comments.

2. Phân tích đa biến:

- So sánh chỉ số trung bình theo category.
- Scatter score vs nums_comments (trục log + đường xu hướng).
- Chỉ số Value-for-Money theo category.
- Box/strip theo category cho score và price.
- Heatmap tương quan Pearson.
- Phân tích giờ mở/đóng cửa và heatmap cặp giờ.

3. Phân tích không gian địa lý:

- Tách District/Ward/Street từ Address.
- Geocoding bằng Nominatim, có cache tại data/coord_cache.json.
- Fallback tọa độ trung tâm quận khi geocode thiếu.
- Bản đồ scatter_mapbox theo category.
- Bản đồ nhiệt density_mapbox theo mức giá.
- Dashboard so sánh mật độ và giá theo quận.

4. Phân tích văn bản:

- WordCloud cho Comments.
- WordCloud cho Name.
- Dùng stopwords Việt + Anh để giảm nhiễu.

## 4. deep_learning.ipynb đã làm gì

Notebook này tập trung vào phần xây dựng mô hình Deep Learning để giải quyết bài toán gợi ý địa điểm (Recommendation System).

### 4.1 Tổng quan bài toán và kiến trúc mô hình

- **Bài toán**: Phát biểu lại vấn đề gợi ý địa điểm dưới dạng phân loại văn bản đa lớp (Multi-class Text Classification).
- **Mô hình chính**: TextCNN (Convolutional Neural Networks) theo kiến trúc đề xuất bởi Yoon Kim (2014), kết hợp pre-trained FastText embedding (300 chiều) cho tiếng Việt.
- **Quy trình**: Từ dữ liệu thô → tiền xử lý → chia tập → huấn luyện → đánh giá → lưu checkpoint → suy luận.

### 4.2 Tiền xử lý dữ liệu cho mô hình

- **Trích xuất bình luận**: Phân rã cột dữ liệu bình luận gốc thành danh sách các câu văn bản (`comment_list`).
- **Tạo bình luận mặc định**: Với các địa điểm không có bình luận, nhóm tự động tạo mô tả dựa trên category + address + price với mức đánh giá mặc định 5.0.
- **Tái cấu trúc dữ liệu**: Sử dụng `explode()` để chuyển từ "1 địa điểm - nhiều bình luận" thành "1 địa điểm - 1 bình luận" trên mỗi dòng.
- **Tạo rank_score**: Đặc trưng xếp hạng kết hợp Score, Comment_score_mean, hoặc giá trị mặc định từ toàn bộ tập dữ liệu.

### 4.3 Chiến lược chia tập dữ liệu

- **Xử lý dữ liệu không cân bằng**: Tách riêng các địa điểm "hiếm" (< 6 bình luận) để tránh mất dữ liệu học tập.
- **Stratified split**: Chia dữ liệu còn lại theo tỷ lệ 70% Train - 15% Validation - 15% Test, đảm bảo phân bố đều của mỗi lớp trên ba tập.
- **Mã hóa nhãn**: Chuyển tên địa điểm (dạng chuỗi) thành label ID (số nguyên) để làm biến mục tiêu cho mô hình.

### 4.4 Xây dựng luồng dữ liệu

- **TextVectorization Layer**: Chuẩn hóa văn bản, chuyển từ thành token với `max_tokens = 20000`, độ dài chuỗi cố định `sequence_length = 100`.
- **Embedding Layer**: Sử dụng ma trận trọng số FastText pre-trained (300 chiều), được khởi tạo bằng cách duyệt từ điển và lưu trữ vector ngữ nghĩa tương ứng. Cố định trọng số (`trainable=False`) để giữ bảo toàn biểu diễn.
- **One-hot Encoding**: Mã hóa nhãn dạng số nguyên thành vector nhị phân tương thích với hàm mất mát `CategoricalCrossentropy`.
- **Batch processing**: Chia dữ liệu thành batch `batch_size = 64` để cân bằng tốc độ hội tụ và tài nguyên phần cứng.

### 4.5 Quá trình huấn luyện

- **Kiến trúc TextCNN**:
  - 3 nhánh Conv1D song song với kernel size 3, 4, 5 (128 filters mỗi nhánh).
  - GlobalMaxPooling1D sau mỗi nhánh để trích xuất đặc trưng quan trọng nhất.
  - Concatenate 3 nhánh + BatchNormalization + Dropout (0.5) + Dense (256, ReLU) + Output (Softmax).
- **Hàm mất mát & Tối ưu hóa**: `CategoricalCrossentropy` + Adam optimizer.
- **Chặn sớm (EarlyStopping)**: Dừng huấn luyện khi val_loss không giảm, lưu lại epoch tốt nhất.

### 4.6 Đánh giá và huấn luyện trên toàn bộ dữ liệu

- **Đánh giá trên tập kiểm thử**: Sử dụng `Top-5 Accuracy`, `Macro F1-Score`, và `Classification Report`.
- **Retraining trên toàn bộ dữ liệu**: Tích hợp Train + Validation + Test để tối ưu hóa hiệu suất trước triển khai.
  - Sử dụng `SparseCategoricalCrossentropy` và `SparseTopKCategoricalAccuracy` để tiết kiệm bộ nhớ khi làm việc với hàng chục ngàn mẫu.
  - Số epochs dừng được xác định từ best_epoch trong quá trình huấn luyện trước.

### 4.7 Lưu checkpoint

- Lưu trữ tại `checkpoints/textcnn.pkl` dưới dạng dictionary chứa:
  - Trọng số mô hình
  - Nhãn lớp (label_encoder.classes_)
  - Từ vựng (vectorizer_vocab)
  - Siêu tham số (max_tokens, sequence_length, embedding_dim, v.v.)
- Cho phép hệ thống gợi ý hoạt động độc lập mà không cần huấn luyện lại.

### 4.8 Suy luận và hệ thống gợi ý hoàn chỉnh

- **Đọc checkpoint**: Khôi phục mô hình từ file `.pkl` để suy luận nhanh.
- **Bảng lookup**: Tạo bảng siêu dữ liệu của các địa điểm (Name, Address, Score, Price, Category, v.v.).
- **Công thức điểm cuối cùng**: Kết hợp xác suất từ mô hình (70%) + rank_score_normalized (30%).
- **Phân tích câu truy vấn**: Trích xuất bộ lọc (district, price range) từ câu mô tả tự nhiên của người dùng.
- **Trả về kết quả**: Danh sách gợi ý bao gồm 3 attraction, 2 dining, 1 hotel, được sắp xếp theo final_score giảm dần.

### 4.9 Đánh giá độ phức tạp

- **Kích thước checkpoint**: Khoảng vài MB (tùy thuộc kích thước mô hình).
- **Số tham số tổng cộng**: Thể hiện qua `model.summary()` và `count_params()`.
- **Thời gian suy luận**: Trung bình vài chục milliseconds trên một câu truy vấn.

## 5. Dữ liệu đầu ra chính trong thư mục data

Theo luồng notebook, các file dữ liệu chính đã/đang được sử dụng:

- data/ggmaps.csv: dữ liệu attraction từ Google Maps.
- data/booking.csv: dữ liệu hotel từ Booking.
- data/foody.csv: dữ liệu dining từ Foody.
- data/data.csv: dữ liệu gộp từ nhiều nguồn.
- data/processed_data.csv: dữ liệu sau tiền xử lý, dùng cho EDA/mô hình.
- data/coord_cache.json: cache geocoding để tiết kiệm số lần gọi API.
- checkpoints/textcnn.pkl: checkpoint mô hình TextCNN cho hệ thống gợi ý.

## 6. Điểm mạnh của quy trình hiện tại

### 6.1 Pipeline dữ liệu

- Thiết kế pipeline end-to-end rõ ràng từ thu thập → xử lý → phân tích → mô hình.
- Có chiến lược chống gián đoạn khi scraping (checkpoint, backup theo khu vực).
- Có xử lý dữ liệu thực tế khá sâu: chuẩn hóa địa chỉ, parse bình luận, kiểm tra hợp lệ.
- EDA đa chiều: thống kê mô tả, tương quan, không gian, văn bản.
- Có nhận xét sau từng phân tích giúp kết nối dữ liệu với insight nghiệp vụ.

### 6.2 Mô hình Deep Learning

- Lựa chọn TextCNN phù hợp với bản chất dữ liệu (bình luận ngắn có mật độ thông tin cao).
- Sử dụng pre-trained FastText embedding, tiết kiệm chi phí học đặc trưng từ đầu.
- Chiến lược xử lý dữ liệu không cân bằng (rare_classes, stratified split).
- Hệ thống gợi ý hoàn chỉnh với bộ lọc thông minh từ câu truy vấn tự nhiên.
- Lưu checkpoint cho phép triển khai nhanh mà không cần huấn luyện lại.

## 7. Gợi ý cải thiện tiếp theo

- **Code organization**: Tách code scraping, preprocessing, model training thành các module .py riêng để tái sử dụng và dễ test.
- **Data management**: Chuẩn hóa tên file đầu ra cố định (tránh phụ thuộc số lượng trong tên).
- **Logging & Monitoring**: Thêm logging chuẩn và thống kê lỗi chi tiết theo từng nguồn dữ liệu.
- **Model improvement**: Thử các mô hình khác (RNN, LSTM, Transformer) để so sánh hiệu suất; điều chỉnh siêu tham số trong công thức Final_Score.
- **Production**: Xây dựng API server để phục vụ yêu cầu gợi ý từ front-end; add caching để tăng tốc độ đáp ứng.
- **Validation**: Thực hiện user study để đánh giá chất lượng gợi ý thực tế.
