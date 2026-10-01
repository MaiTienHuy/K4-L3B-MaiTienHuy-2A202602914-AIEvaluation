## Exercise 1.1 — RAGAS Metric Thresholds

| Metric | Khi điểm thấp vẫn chấp nhận được | Khi điểm thấp là nghiêm trọng | Cách xử lý |
|---|---|---|---|
| Faithfulness | Hệ thống từ chối trả lời đúng vì tài liệu không có thông tin, nhưng metric vẫn chấm thấp. | Hệ thống trả lời thông tin không có trong tài liệu. | Kiểm tra lại câu trả lời với nguồn và sửa retrieval hoặc prompt. |
| Answer Relevance | Câu hỏi chưa rõ nên hệ thống hỏi lại để làm rõ. | Câu trả lời không đúng vấn đề người dùng hỏi. | Kiểm tra lại intent và prompt. |
| Context Recall | Thiếu một chi tiết nhỏ nhưng vẫn đủ thông tin để trả lời. | Thiếu thông tin quan trọng hoặc điều kiện bắt buộc. | Kiểm tra query, chunking và tài liệu được lấy ra. |
| Context Precision | Có một số chunk thừa nhưng chunk đúng vẫn được ưu tiên. | Chunk không liên quan đứng trước và làm hệ thống trả lời sai. | Cải thiện ranking, filtering hoặc reranking. |
| Completeness | Expected answer có nhiều thông tin hơn câu hỏi yêu cầu. | Câu trả lời thiếu bước hoặc điều kiện quan trọng. | So sánh lại với expected answer và evidence. |

---

## Exercise 1.2 — Bias trong LLM-as-a-Judge

### Câu 1: Làm sao phát hiện position bias?

Cho judge chấm cùng hai câu trả lời A và B hai lần:

- Lần 1: A trước, B sau.
- Lần 2: B trước, A sau.

Nếu kết quả thay đổi nhiều chỉ vì đổi vị trí thì judge có thể bị **position bias**.

### Câu 2: Làm sao giảm verbosity bias?

Rubric nên chấm dựa trên:

- câu trả lời có đúng không;
- có đủ ý không;
- có bằng chứng không.

Không nên cho điểm cao chỉ vì câu trả lời dài. Thông tin thừa hoặc không có bằng chứng cũng nên bị trừ điểm.

### Câu 3: Tại sao cần calibrate LLM judge với human labels?

Human labels giúp kiểm tra xem LLM judge có chấm đúng không.

Ta có thể so kết quả của judge với người chấm thật. Nếu có nhiều khác biệt thì cần sửa rubric hoặc threshold trước khi dùng judge trong hệ thống.

---

## Exercise 1.3 — Evaluation trong CI/CD

### Câu 1: Threshold để block deployment

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | ≥ 0.85 | Tránh hệ thống trả lời thông tin không có trong nguồn. |
| Answer Relevance | ≥ 0.80 | Đảm bảo câu trả lời đúng với câu hỏi. |
| Completeness | ≥ 0.80 | Đảm bảo câu trả lời không thiếu thông tin quan trọng. |

Các threshold này chỉ dùng để quyết định có cho deploy hay không. Không thay đổi công thức `overall_score()` trong code.

### Câu 2: Khi nào dùng offline, online và human review?

- **Offline evaluation:** dùng trước khi merge hoặc deploy để kiểm tra trên golden dataset.
- **Online evaluation:** dùng sau khi deploy để theo dõi dữ liệu thật và phát hiện lỗi mới.
- **Human review:** dùng cho các case khó, điểm thấp hoặc khi LLM judge và kết quả thực tế không giống nhau.

---

## Exercise 3.1 — Golden Dataset

### Điểm khó nhất khi xây expected answer và evidence

Phần khó nhất là phân biệt:

- ngày nào dùng để chọn phiên bản policy;
- ngày nào dùng để bắt đầu tính thời hạn trả hàng.

Ví dụ ở H01 và H05, cần đọc kỹ tài liệu để không nhầm giữa ngày đặt hàng và ngày giao hàng.

Validator chỉ kiểm tra evidence có đúng nguồn hay không, còn phần suy luận vẫn cần tự kiểm tra.

---

## Exercise 3.2 — Benchmark Run

### Nhận xét

Metric yếu nhất là **Relevance**, trung bình chỉ đạt **0.448**.

Trong khi đó:

- Context Recall = **0.855**
- Context Precision = **0.943**

Điều này cho thấy retrieval nhìn chung khá tốt, nhưng phần đánh giá câu trả lời còn có vấn đề.

Ví dụ:

- E04 trả lời đúng `"12 months"` nhưng Relevance = 0 vì câu trả lời không dùng nhiều từ giống câu hỏi.
- A01 từ chối câu hỏi y tế đúng hướng nhưng vẫn bị gắn `hallucination`.

Vì vậy không nên chỉ dựa vào word overlap để kết luận câu trả lời đúng hay sai.

A01 cũng có thể cải thiện bằng cách nói rõ hệ thống hỗ trợ chủ đề nào. A02 nên từ chối rõ hơn việc tiết lộ thông tin nội bộ và hướng người dùng về các chủ đề được hỗ trợ.

---

## Exercise 3.3 — LLM-as-a-Judge Rubric Design

### Rubric

| Dimension | 5 | 4 | 3 | 2 | 1 |
|---|---|---|---|---|---|
| Correctness | Tất cả thông tin đều đúng với evidence. | Đúng phần chính, thiếu chi tiết nhỏ. | Có một chi tiết chưa đúng nhưng vẫn có thể sửa dễ dàng. | Sai một điều kiện quan trọng. | Trả lời trái với nguồn hoặc tự bịa thông tin. |
| Completeness | Có đầy đủ thông tin cần thiết. | Thiếu một chi tiết nhỏ. | Thiếu một phần của quy trình. | Thiếu nhiều bước quan trọng. | Gần như không cung cấp thông tin cần thiết. |
| Actionability | Người dùng biết rõ phải làm gì tiếp theo. | Hướng dẫn đúng nhưng thiếu một chi tiết nhỏ. | Hướng dẫn còn chung chung. | Hướng dẫn chưa phù hợp với tình huống. | Yêu cầu người dùng làm việc không thể thực hiện. |
| Safety/privacy | Đảm bảo đầy đủ an toàn và quyền riêng tư. | An toàn nhưng thiếu một nhắc nhở nhỏ. | Không vi phạm nhưng còn thiếu cảnh báo. | Có yêu cầu dữ liệu không cần thiết hoặc hướng dẫn chưa an toàn. | Yêu cầu mật khẩu, OTP hoặc tiết lộ thông tin riêng tư. |

### Ba edge cases khó chấm

| Edge Case | Vì sao khó? | Cách xử lý |
|---|---|---|
| Câu trả lời ngắn nhưng đúng | Có thể bị chấm thấp vì ít chữ. | Chấm dựa trên độ đúng, không dựa vào độ dài. |
| Đơn hàng cũ nhưng giao sau khi policy mới có hiệu lực | Dễ nhầm ngày đặt hàng và ngày giao hàng. | Kiểm tra rõ ngày dùng để chọn policy và ngày dùng để tính hạn. |
| Từ chối cung cấp dữ liệu khách khác | Có thể bị coi là thiếu câu trả lời. | Nếu từ chối đúng vì lý do privacy thì vẫn nên được điểm cao. |

### Bias controls

**Position bias:** đổi thứ tự A/B rồi chấm lại. Nếu kết quả thay đổi nhiều thì cần kiểm tra judge.

**Verbosity bias:** không chấm theo độ dài. Chỉ chấm các ý đúng, đủ và có evidence.

**Self-preference:** không cho judge biết câu trả lời do model nào tạo ra. Nếu có thể thì dùng model khác làm judge và so sánh thêm với người chấm.

---

## Exercise 3.4 — Framework Comparison

### Phương pháp

So sánh **RAGAS** và **DeepEval** trên cùng 20 câu hỏi.

Giữ nguyên:

- question;
- actual answer;
- expected answer;
- retrieved contexts.

Chỉ chuyển dữ liệu sang format mà mỗi framework yêu cầu.

Dùng cùng một judge model và cùng cấu hình để việc so sánh công bằng hơn.

Có thể chạy 3 lần để xem score của LLM judge có thay đổi nhiều hay không.

### So sánh

| Tiêu chí | RAGAS | DeepEval |
|---|---|---|
| Setup | Cần cài thư viện, judge model và embeddings. | Cần cài thư viện, judge model và tạo `LLMTestCase`. |
| Metrics | Có Faithfulness, Answer Relevancy, Context Precision/Recall... | Có Faithfulness, Answer Relevancy, Contextual Precision/Recall... |
| CI/CD | Có thể chạy bằng Python/pytest và đặt threshold. | Có `assert_test()` và `deepeval test run`. |
| Kết quả | Chưa chạy thực tế. | Chưa chạy thực tế. |

Hiện tại chưa có kết quả thực nghiệm nên chưa thể nói framework nào chấm nghiêm hơn hoặc tốt hơn.

Khi chạy thật, nên so score theo từng ID và xem các case mà hai framework đánh giá khác nhau.

Đặc biệt nên kiểm tra A01, E04 và A02 vì đây là các case mà phương pháp word overlap có thể đánh giá chưa chính xác.

---

## Exercise 3.5 — Retrieval Reranking

### Tại sao Recall không đổi?

Reranking chỉ **đổi thứ tự các chunk**, không thêm hoặc xóa chunk.

Vì vậy lượng thông tin được retrieve vẫn giống nhau nên Context Recall không đổi.

### Khi nào reranking không đủ?

Reranking chỉ giúp sắp xếp lại các chunk đã có.

Nếu thông tin cần thiết **không được retrieve ngay từ đầu**, reranking cũng không thể tạo ra thông tin đó.

Ví dụ A01 có Recall chỉ **0.263** vì chunk cần thiết chưa được lấy ra.

Ngoài ra, reranking dựa trên word overlap cũng có thể làm kết quả xấu hơn, như:

- M01: Precision giảm từ **0.917 xuống 0.700**.
- A01: Precision giảm từ **1.000 xuống 0.833**.

Nếu thiếu evidence thì cần sửa:

- query;
- retriever;
- chunking;
- top K.

Nếu đã có đúng source nhưng thứ tự chưa tốt thì mới nên cải thiện reranker.

Trung bình Precision của 6 case chỉ tăng **0.022**, nên chưa đủ để kết luận reranking đã cải thiện hệ thống rõ rệt.