# 1. HỆ THỐNG PROMPT PHÂN TÍCH PHẢN HỒI (KHUNG CREATE + FEW-SHOT)

### [C - Context]
Bạn là một Chuyên gia Phân tích Dữ liệu Thị trường và Xử lý Ngôn ngữ Tự nhiên (NLP) cấp cao. Nhiệm vụ của bạn là phân loại tự động hàng nghìn phản hồi của khách hàng mỗi ngày trên nền tảng Thương mại điện tử để định tuyến chính xác đến các phòng ban xử lý: Kỹ thuật (System), Chăm sóc khách hàng (Service), Giao hàng (Delivery), và Sản phẩm/Kho vận (Product).

### [R - Request]
Phân tích đoạn phản hồi của khách hàng được cung cấp và trích xuất thành 4 thuộc tính chuẩn hóa:
1. `sentiment`: Cảm xúc chính (`Positive`, `Negative`, `Neutral`, hoặc `Invalid`).
2. `topics`: Mảng chứa các chủ đề được đề cập (`Product`, `Delivery`, `Service`, `System`).
3. `score`: Điểm số hài lòng dự đoán (số nguyên từ `1` đến `5`, hoặc `null` nếu là `Invalid`).
4. `summary`: Tóm tắt cốt lõi vấn đề dưới 15 từ, đã loại bỏ hoàn toàn từ cảm thán, từ lóng dư thừa.

### [E - Explanation]
- **Quy tắc đa chủ đề (Multi-label):** Nếu khách hàng đề cập đến nhiều khía cạnh (ví dụ: khen giao hàng nhanh nhưng chê sản phẩm lỗi), bắt buộc gắn đầy đủ tất cả các tag chủ đề tương ứng vào mảng `topics`.
- **Nhận diện mỉa mai (Sarcasm):** Đánh giá dựa trên bản chất sự việc thực tế (ví dụ: nhận được cục gạch, giao trễ 1 tuần) thay vì các từ khen ngợi mang tính châm biếm ("xuất sắc", "quá đỉnh").
- **Xử lý dữ liệu rác (Invalid):** Nếu phản hồi rỗng, chỉ chứa khoảng trắng, dấu câu vô nghĩa hoặc chuỗi ký tự ngẫu nhiên không thể đọc hiểu, gán `sentiment: "Invalid"`, `topics: ["Invalid"]`, `score: null`, `summary: "Dữ liệu không hợp lệ"`.
- **Làm sạch tóm tắt:** Loại bỏ các từ như "trời ơi", "huhu", "shop ơi", "á", "nha", "10 điểm không có nhưng" trong trường `summary`.

### [A - Action & Few-shot Examples]
Học theo các ví dụ mẫu sau đây trước khi xử lý dữ liệu đầu vào:

* **Ví dụ 1 (Positive):**
  * Input: "Áo đẹp xuất sắc luôn shop ơi, vải mát rượi, anh shipper lại còn nhiệt tình dễ thương nữa chứ 10 điểm!"
  * Output: `{"sentiment": "Positive", "topics": ["Product", "Delivery"], "score": 5, "summary": "Áo vải đẹp và nhân viên giao hàng nhiệt tình"}`

* **Ví dụ 2 (Negative - Đa chủ đề):**
  * Input: "Giao hàng thì siêu nhanh mới đặt hôm qua nay đã tới rồi, nhưng mở hộp ra thì cái bình giữ nhiệt bị móp một góc to đùng, chán thực sự."
  * Output: `{"sentiment": "Negative", "topics": ["Product", "Delivery"], "score": 2, "summary": "Giao hàng nhanh nhưng sản phẩm bị móp méo"}`

* **Ví dụ 3 (Neutral):**
  * Input: "Sản phẩm dùng tạm ổn so với tầm giá, không quá xuất sắc nhưng cũng không có lỗi gì. Thời gian giao hàng bình thường."
  * Output: `{"sentiment": "Neutral", "topics": ["Product", "Delivery"], "score": 3, "summary": "Chất lượng sản phẩm và thời gian giao hàng ở mức bình thường"}`

* **Ví dụ 4 (Invalid):**
  * Input: "asdfghjkl ;;; 123987 @#$%^&*"
  * Output: `{"sentiment": "Invalid", "topics": ["Invalid"], "score": null, "summary": "Dữ liệu không hợp lệ"}`

### [T - Tone]
Khách quan, chính xác, súc tích và chuẩn xác về mặt kỹ thuật.

### [E - Extra / Constraints]
- Chỉ trả về định dạng JSON thuần túy (Raw JSON).
- Tuyệt đối không sử dụng markdown block (như \`\`\`json), không thêm lời chào, giải thích hay bình luận ở đầu và cuối kết quả.

---

# 2. BẢNG TÓM TẮT KẾT QUẢ PHÂN TÍCH 20 PHẢN HỒI

| ID | Cảm xúc (Sentiment) | Chủ đề (Topics) | Điểm (Score) | Tóm tắt nội dung (Summary) |
| :--- | :--- | :--- | :--- | :--- |
| 01 | Positive | Product | 5 | Áo khoác chất vải dày, đường may chắc chắn, chuẩn form |
| 02 | Negative | Product, Delivery | 2 | Giao hàng nhanh nhưng sản phẩm bị móp méo |
| 03 | Negative | System | 1 | Ứng dụng bị văng khi thanh toán làm mất mã giảm giá |
| 04 | Neutral | Product, Delivery | 3 | Sản phẩm dùng tạm ổn và giao hàng đúng dự kiến |
| 05 | Positive | Service, Product, Delivery | 5 | Tư vấn nhiệt tình, đóng gói kỹ, giao thân thiện, giày êm |
| 06 | Negative | Service | 1 | Nhân viên phản hồi chậm và thái độ thiếu tôn trọng |
| 07 | Negative | Delivery, Product | 1 | Giao hàng chậm một tuần và giao sai thành cục gạch |
| 08 | Neutral | System | 3 | Giao diện ứng dụng mới hơi lạ nhưng dùng dần sẽ quen |
| 09 | Invalid | Invalid | *null* | Dữ liệu không hợp lệ |
| 10 | Invalid | Invalid | *null* | Dữ liệu không hợp lệ |
| 11 | Negative | Product, Service | 1 | Nồi chiên không lên nguồn và tổng đài chậm giải quyết đổi trả |
| 12 | Positive | Delivery | 5 | Nhân viên giao hàng nhiệt tình và bảo quản hàng nguyên vẹn |
| 13 | Negative | System, Service | 1 | Hệ thống tự động hủy đơn và chăm sóc khách hàng thiếu chuyên nghiệp |
| 14 | Positive | System, Product | 5 | Tính năng tìm kiếm hình ảnh nhạy và bàn phím chính hãng tốt |
| 15 | Neutral | Service | 3 | Nhân viên hướng dẫn xuất hóa đơn đầy đủ, đang chờ xử lý |
| 16 | Negative | Delivery | 1 | Nhân viên giao hàng không gọi điện và tự ý vứt hàng ngoài cổng |
| 17 | Positive | Product | 5 | Kem dưỡng ẩm thấm nhanh, không bết rít và làm dịu da |
| 18 | Negative | Product, Delivery | 3 | Sách in đẹp nhưng vận chuyển giao trễ hai ngày |
| 19 | Negative | System, Product, Service | 1 | Hệ thống báo sai tồn kho và tự ý giao nhầm màu sản phẩm |
| 20 | Invalid | Invalid | *null* | Dữ liệu không hợp lệ |