### Bước 1: Đề xuất 2 prompt mẫu

#### Giải pháp 1: Prompt đơn giản, ngắn gọn

Phương án này tập trung vào ý chính, dùng câu lệnh trực diện để tiết kiệm thời gian viết và dung lượng prompt.

```text
Bạn là hệ thống xử lý đơn hàng. Hãy đọc tin nhắn khách hàng: "[ĐIỀN TIN NHẮN]".
Nhiệm vụ:
1. Trích xuất: Tên, SĐT, Sản phẩm, Số lượng, Địa chỉ.
2. Kiểm tra lỗi: Nếu thông tin nào thiếu hoặc không rõ, hãy điền chữ "invalid".
3. Trả về kết quả dưới dạng danh sách ngắn gọn.

```

#### Giải pháp 2: Prompt chi tiết, có nhiều ràng buộc (Kỹ thuật CoT & Xử lý đứt gãy)

Phương án này thiết lập khung tư duy tuần tự chặt chẽ, định nghĩa rõ ràng điều kiện gán nhãn lỗi và cấu trúc đầu ra nghiêm ngặt để tránh ảo giác dữ liệu (hallucination).

```text
Bạn là hệ thống tự động hóa đơn hàng thông minh của cửa hàng. Hãy áp dụng tư duy từng bước (Chain-of-Thought) và quy tắc xử lý đứt gãy để xử lý tin nhắn sau:

### DỮ LIỆU ĐẦU VÀO:
"[ĐIỀN TIN NHẮN]"

### QUY TRÌNH XỬ LÝ (BẮT BUỘC TUÂN THỦ):
1. Bước 1 (Extraction): Trích xuất chính xác các thực thể: Tên khách hàng, Số điện thoại, Sản phẩm, Số lượng, Địa chỉ nhận hàng.
2. Bước 2 (Validation & Error Handling): 
   - Kiểm tra từng trường dữ liệu. Nếu thiếu thông tin, thiếu chi tiết (như thiếu số nhà), hoặc sai định dạng, **bắt buộc** điền giá trị "invalid".
   - Tuyệt đối không tự suy diễn hoặc bịa đặt thông tin không có trong tin nhắn.
3. Bước 3 (Final Output): Tổng hợp trạng thái đơn hàng (Thành công hoặc Đứt gãy).

### ĐỊNH DẠNG ĐẦU RA:
- [Suy luận logic]: Giải thích ngắn gọn tại sao trường nào đó hợp lệ hoặc bị "invalid".
- [Bảng dữ liệu]: Hiển thị các trường kèm trạng thái.
- [Trạng thái hệ thống]: Trả về "Tạo đơn thành công" hoặc "Đứt gãy - Cần hỏi lại khách [Tên trường thiếu]".

```

---

### Bước 2: Lập bảng so sánh Trade-off

| Tiêu chí | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) |
| --- | --- | --- |
| **Độ dễ hiểu & Triển khai** | **Cao** (Dễ viết, nhanh chóng, thích hợp cho người mới). | **Trung bình** (Cần thiết kế cấu trúc logic và ràng buộc chặt chẽ). |
| **Chi phí Token** | **Ít** (Tốn ít token đầu vào cho mỗi lần gọi API). | **Nhiều hơn** (Tốn thêm token cho phần hướng dẫn chi tiết và ví dụ suy luận). |
| **Khả năng chặn lỗi biên (Edge-cases)** | **Trung bình** (Dễ bị AI bỏ sót hoặc tự ý "chế" thêm thông tin khi thiếu). | **Cực tốt** (Kiểm soát chặt chẽ bằng từ khóa `invalid` và luật cấm bịa đặt). |
| **Độ ổn định khi tích hợp hệ thống (Automation)** | **Thấp** (Định dạng đầu ra không cố định, dễ làm lỗi hệ thống phía sau). | **Cao** (Đầu ra tuân thủ cấu trúc chuẩn, dễ tích hợp với code backend). |

---

### Bước 3: Lựa chọn và chạy thử

#### 1. Lựa chọn phương án tối ưu

Phương án tối ưu nhất cho bài toán tự động hóa cửa hàng là **Giải pháp 2 (Chi tiết)**.

**Lý do:** Trong các hệ thống tự động hóa kinh doanh (e-commerce/CRM), việc để AI tự động "đoán mò" thông tin còn thiếu (như tự điền địa chỉ hoặc tự chế tên khách hàng) sẽ gây ra hậu quả nghiêm trọng khi giao hàng nhầm hoặc sai đơn. Giải pháp 2 sử dụng Chain-of-Thought kết hợp ràng buộc chặt chẽ giúp hệ thống nhận diện chính xác điểm đứt gãy (`invalid`) để kích hoạt kịch bản hỏi lại khách hàng thay vì làm hỏng toàn bộ quy trình.

---

#### 2. Chạy thử nghiệm thực tế với Giải pháp 2

* **Dữ liệu giả lập đầu vào:**
`"Đặt giúp mình 2 ly trà sữa trân châu đường đen, giao gấp qua chung cư ở Gò Vấp nhé, số đt 0912345678"`
* **Kết quả phản hồi thực tế từ mô hình:**

```markdown
- [Suy luận logic]: 
  + Tên khách hàng: Không xuất hiện trong tin nhắn -> Gán "invalid".
  + Số điện thoại: "0912345678" hợp lệ (đủ 10 số) -> Hợp lệ.
  + Sản phẩm: "2 ly trà sữa trân châu đường đen" -> Hợp lệ.
  + Số lượng: "2" -> Hợp lệ.
  + Địa chỉ: "chung cư ở Gò Vấp" quá chung chung, thiếu tên chung cư, số phòng, số nhà/tên đường cụ thể -> Gán "invalid".

- [Bảng dữ liệu]:
    | Trường thông tin | Giá trị trích xuất | Trạng thái |
    | :--- | :--- | :--- |
    | Tên khách hàng | invalid | Lỗi (Thiếu) |
    | Số điện thoại | 0912345678 | Hợp lệ |
    | Sản phẩm | Trà sữa trân châu đường đen | Hợp lệ |
    | Số lượng | 2 | Hợp lệ |
    | Địa chỉ | invalid | Lỗi (Thiếu chi tiết) |

- [Trạng thái hệ thống]: 
  Đứt gãy - Cần hỏi lại khách [Tên khách hàng, Địa chỉ chi tiết].

```