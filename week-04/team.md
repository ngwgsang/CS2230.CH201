# Tuần 3

Báo cáo này trình bày tiến độ đánh giá các phương pháp control trong sinh paraphrase tiếng Việt theo ba chiều chính:

- Lexical similarity (tương đồng từ vựng)

- Syntactic similarity (tương đồng cấu trúc)

- Semantic similarity (bảo toàn ngữ nghĩa)

Mục tiêu của thí nghiệm là kiểm tra mức độ phù hợp giữa các estimator tự động và đánh giá của con người, từ đó lựa chọn phương pháp đo lường đáng tin cậy cho pipeline control paraphrase tiếng Việt.

# Dữ liệu

Thí nghiệm được thực hiện trên một tập 600 cặp paraphrase được gán nhãn thủ công, bao gồm:

| Dataset  | Số lượng |
| -------- | -------- |
| ViSP     | 200      |
| ViQP     | 200      |
| vnPara   | 200      |
| **Tổng** | **600**  |


# Đánh giá

Mỗi cặp câu được 3 annotator độc lập đánh giá theo ba chiều:

- Lexical similarity

- Syntactic similarity

- Semantic preservation

Điểm được chấm trên thang 0–100.

Để kiểm tra độ ổn định của việc gán nhãn, nhóm thực hiện 3 vòng calibration trên 60 mẫu pilot. Sai lệch trung bình giữa các annotator (MAE) đạt:

# Lexical Control

| Lexical Estimator           | Pearson r ↑ | MAE ↓    |
| --------------------------- | ----------- | -------- |
| Word Tokenize (pyvi)        | **0.9755**  | **2.98** |
| Word Tokenize (underthesea) | 0.7857      | 19.05    |
| Whitespace Tokenize         | 0.8868      | 7.43     |

Nhận xét

Pyvi segmentation cho kết quả tốt nhất.

Tokenization theo khoảng trắng vẫn đạt tương quan khá cao nhưng sai số lớn hơn.

Underthesea có tương quan thấp hơn đáng kể.

Điều này cho thấy chất lượng tách từ ảnh hưởng rất lớn tới việc đo lexical similarity trong tiếng Việt, đặc biệt với các từ ghép.

# Syntactic Control

Đối với chiều cú pháp, chúng tôi sử dụng Tree Edit Distance (TED) trên cây constituency và thử nghiệm các mức max depth khác nhau.

| Syntactic Estimator | Pearson r ↑ | MAE ↓    |
| ------------------- | ----------- | -------- |
| TED (max_depth=1)   | 0.1792      | 18.03    |
| TED (max_depth=2)   | 0.7767      | 9.87     |
| TED (max_depth=3)   | **0.9615**  | **2.87** |
| TED (max_depth=4)   | 0.8724      | 7.51     |

Nhận xét

max_depth = 3 cho kết quả tốt nhất.

Depth quá nhỏ không thể capture sự khác biệt cấu trúc.

Depth quá sâu dễ bị nhiễu do lỗi parser.

Kết quả cho thấy con người có xu hướng đánh giá similarity cấu trúc ở mức phrase-level, thay vì ở mức quá thô hoặc quá chi tiết.

# Semantic Control

Kết quả đánh giá Semantic Control

Thí nghiệm semantic sử dụng BERTScore với các contextual encoder khác nhau.

| Semantic Estimator | Pearson r ↑ | MAE ↓    |
| ------------------ | ----------- | -------- |
| PhoBERT-base       | **0.9747**  | **2.71** |
| PhoBERT-large      | 0.9040      | 21.93    |
| mBERT              | 0.8487      | 5.67     |
| XLM-R-base         | 0.8476      | 28.22    |


Nhận xét

PhoBERT-base đạt tương quan gần như hoàn hảo với đánh giá của con người.

Các mô hình multilingual có hiệu năng thấp hơn.

PhoBERT-large có correlation khá cao nhưng sai số lớn hơn.

Điều này cho thấy encoder đơn ngữ phù hợp hơn cho việc đánh giá semantic preservation trong paraphrase tiếng Việt.

# Kết luận tạm thời

Từ các thí nghiệm ablation, pipeline control được lựa chọn như sau:

| Dimension | Method                           |
| --------- | -------------------------------- |
| Lexical   | Pyvi tokenization                |
| Syntactic | Tree Edit Distance (max_depth=3) |
| Semantic  | BERTScore với PhoBERT-base       |

Các phương pháp trên cho độ tương quan rất cao với đánh giá của con người, do đó được chọn làm thành phần kiểm soát chính trong hệ thống paraphrase control cho tiếng Việt.