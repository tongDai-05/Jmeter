# JMeter - Báo cáo kiểm tra hiệu năng trang web

## 1. Mục tiêu kiểm thử

- Sử dụng Apache JMeter để tạo và thực thi kịch bản kiểm thử tải (Stress / Performance Testing) mô phỏng người dùng truy cập trang web [VnExpress](https://vnexpress.net/).
- Đo lường và đánh giá khả năng chịu tải của hệ thống dưới áp lực lớn (10.000 người dùng đồng thời).
- Ghi nhận và phân tích các chỉ số hiệu năng: thời gian phản hồi (Response Time), tỷ lệ lỗi (Error Rate), thông lượng (Throughput) và băng thông mạng.
- Đưa ra nhận xét, đánh giá nguyên nhân và đề xuất phương án cải tiến kịch bản kiểm thử.

---

## 2. Kịch bản kiểm tra (Test Plan)

- **Thread Group (Nhóm người dùng ảo)**:
  - **Số lượng thread (Number of Threads - Users)**: 10.000 người dùng ảo
  - **Thời gian tăng tốc (Ramp-up period)**: 100 giây (trung bình khởi tạo 100 thread/giây)
  - **Thời gian chạy tối đa (Duration)**: 60 giây
  - **Số vòng lặp (Loop Count)**: 1
- **HTTP Request**:
  - **Giao thức (Protocol)**: HTTPS
  - **Tên miền (Server Name)**: `vnexpress.net`
  - **Đường dẫn (Path)**: `/`
  - **Phương thức (Method)**: GET
  - **Mã hóa (Content Encoding)**: UTF-8
- **Listeners (Bộ thu thập kết quả)**:
  - **Aggregate Report**: Thống kê số liệu hiệu năng tổng hợp.
  - **View Results Tree**: Kiểm tra trạng thái và thông tin chi tiết từng request (cấu hình Log Errors Only).

---

## 3. Kết quả kiểm tra

### Báo cáo tổng hợp (Aggregate Report)
![Aggregate Report](screenshots/vnexpress_aggregate_report.png)

### Bảng số liệu chi tiết
| Chỉ số (Metric) | Giá trị | Ý nghĩa |
| :--- | :--- | :--- |
| **Số mẫu kiểm thử (# Samples)** | 10.000 | Tổng số request gửi đến hệ thống |
| **Thời gian phản hồi TB (Average)** | 21.036 ms (~21,04 s) | Thời gian phản hồi trung bình của một request |
| **Thời gian trung vị (Median)** | 21.036 ms | 50% số request có thời gian phản hồi ≤ 21.036 ms |
| **90% Line** | 21.050 ms | 90% số request có thời gian phản hồi ≤ 21.050 ms |
| **95% Line** | 21.054 ms | 95% số request có thời gian phản hồi ≤ 21.054 ms |
| **99% Line** | 21.061 ms | 99% số request có thời gian phản hồi ≤ 21.061 ms |
| **Thời gian nhỏ nhất (Min)** | 21.006 ms | Thời gian phản hồi nhanh nhất |
| **Thời gian lớn nhất (Max)** | 21.115 ms | Thời gian phản hồi chậm nhất |
| **Tỷ lệ lỗi (Error %)** | 100.00% | 10.000 / 10.000 request thất bại |
| **Thông lượng (Throughput)** | 82,7 req/giây | Tốc độ hoàn tất xử lý request |
| **Băng thông nhận (Received KB/sec)** | 222,29 KB/s | Tốc độ truyền tải dữ liệu nhận về |
| **Băng thông gửi (Sent KB/sec)** | 0,00 KB/s | Tốc độ truyền tải dữ liệu gửi đi |

---

## 4. Hình ảnh chi tiết lỗi (View Results Tree)

![View Results Tree](screenshots/vnexpress_overview.png)

*Toàn bộ 10.000 request đều hiển thị trạng thái lỗi (icon đỏ) khi lọc `Log/Display Only: Errors`.*

---

## 5. Phân tích kết quả kiểm tra

1. **Tỷ lệ lỗi tuyệt đối (100% Error Rate)**:
   - Với mức tải cực đại 10.000 users trong vòng 100 giây từ một địa chỉ IP duy nhất, hệ thống VnExpress không phản hồi thành công bất kỳ request nào.
2. **Thời gian phản hồi đồng nhất quanh mốc ~21 giây**:
   - Biên độ chênh lệch thời gian giữa Min (21.006 ms) và Max (21.115 ms) chỉ khoảng 109 ms.
   - Điều này cho thấy các request không nhận được phản hồi nội dung thực tế từ ứng dụng mà bị treo kết nối cho đến khi chạm ngưỡng **Connection Timeout / Read Timeout** (khoảng 20–21 giây) rồi bị hủy.
3. **Nguyên nhân chính**:
   - **Hệ thống phòng thủ bảo vệ (WAF / Anti-DDoS / Rate Limiting)**: Hệ thống tường lửa ứng dụng (WAF) hoặc CDN bảo vệ của VnExpress đã phát hiện lưu lượng truy cập bất thường với tần suất quá cao từ cùng một nguồn IP và tự động kích hoạt cơ chế chặn kết nối (block IP / drop socket connection) để bảo vệ server gốc khỏi nguy cơ quá tải hoặc tấn công từ chối dịch vụ.
   - **Nghẽn tài nguyên máy Client**: Việc khởi tạo 10.000 kết nối đồng thời từ một máy kiểm thử cá nhân có thể gây cạn kiệt cổng kết nối cục bộ (Ephemeral Port Exhaustion) hoặc nghẽn hàng đợi TCP stack của hệ điều hành.

---

## 6. Kết luận & Đề xuất

### Kết luận
- Trang web [VnExpress](https://vnexpress.net/) có cơ chế bảo vệ an ninh và kiểm soát lưu lượng truy cập rất nghiêm ngặt và hiệu quả trước các đợt tải đột biến từ một nguồn duy nhất.
- Kịch bản kiểm thử tải 10.000 người dùng ảo từ một máy đơn lẻ giúp xác định được giới hạn chịu tải tối đa và cơ chế phòng vệ của hạ tầng mạng.

### Đề xuất
- **Kiểm thử phân tán (Distributed Testing)**: Áp dụng mô hình Master-Slave trong JMeter để sinh tải từ nhiều dải IP khác nhau, mô phỏng chân thực lưu lượng người dùng truy cập tự nhiên.
- **Tăng tải theo từng giai đoạn (Step-up / Ramp-up Testing)**: Bắt đầu từ mức tải nhỏ và tăng dần (ví dụ: 500 $\rightarrow$ 1.000 $\rightarrow$ 2.000 $\rightarrow$ 5.000 users) để tìm chính xác điểm bão hòa (Saturation Point) và ngưỡng kích hoạt bảo vệ của hệ thống.
- **Môi trường thử nghiệm**: Nên thực hiện trên môi trường Staging/Test có cấu hình tương đương Production và được whitelist IP kiểm thử để đánh giá chính xác năng lực xử lý mã nguồn và cơ sở dữ liệu.
