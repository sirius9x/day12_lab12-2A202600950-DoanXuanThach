# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found
1. **Hardcode API Key**: Mật khẩu và API key (như `OPENAI_API_KEY`) bị ghi trực tiếp vào mã nguồn, gây rủi ro lộ lọt bảo mật cực lớn khi đẩy code lên GitHub.
2. **Hardcode Port và Host**: Ứng dụng bị gắn cứng vào `localhost` và port `8000`, khiến nó không thể chạy được trong container hoặc trên cloud (cloud thường tự cấp phát port ngẫu nhiên).
3. **Thiếu Health Check**: Không có các endpoint như `/health` hay `/ready` để hệ thống quản lý (như Docker/Kubernetes/Railway) biết trạng thái sống còn của ứng dụng.
4. **Sử dụng Print thay vì Logging chuẩn**: In log bằng `print()` không thể theo dõi phân tích tự động trên các hệ thống giám sát lớn, và code cũ thậm chí in lộ luôn cả secret key ra màn hình.
5. **Thiếu Graceful Shutdown**: Ứng dụng bị tắt đột ngột khiến các request đang xử lý bị ngắt ngang lập tức, sinh ra lỗi 502/503 cho client.

### Exercise 1.3: Comparison table
| Feature | Develop (Basic) | Production (Advanced) | Tại sao quan trọng? |
|---------|---------|------------|----------------|
| **Config**  | Hardcode trực tiếp các biến số và secret. | Dùng Environment variables. | Tránh lộ dữ liệu nhạy cảm trên GitHub. Dễ đổi thông số cho từng môi trường. |
| **Health check** | ❌ Không có | ✅ Có (`/health`, `/ready`) | Nền tảng Cloud cần endpoint này để liên tục hỏi thăm server, tự động restart nếu bị treo. |
| **Logging** | Dùng `print()` đơn giản. | Structured JSON Logging. | Giúp hệ thống log chuyên nghiệp dễ bóc tách, lọc, và tìm kiếm tự động. |
| **Shutdown** | Đột ngột | Graceful shutdown | Cho phép app hoàn thành nốt request đang chạy dở trước khi tắt hẳn, tránh rớt kết nối. |
| **Network** | Bind `localhost` và port cố định | Bind `0.0.0.0` và Port linh hoạt | Bắt buộc để nhận traffic khi chạy trong Docker container và triển khai lên Cloud. |

## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. **Base image**: `python:3.11` (Bản phân phối hệ điều hành và Python đầy đủ).
2. **Working directory**: `/app` (Thư mục làm việc mặc định mọi câu lệnh sẽ thực thi trong container).
3. **Tại sao COPY requirements.txt trước?**: Để tận dụng cơ chế Docker Layer Cache. Giúp quá trình build lại (rebuild) cực kỳ nhanh khi code thay đổi nhưng thư viện thì không đổi.
4. **CMD vs ENTRYPOINT**: `CMD` dễ dàng bị ghi đè (override) khi truyền lệnh khác vào cuối dòng `docker run`, còn `ENTRYPOINT` thì khóa cứng ứng dụng sẽ chạy và chỉ nhận lệnh thêm vào dạng tham số (arguments).

### Exercise 2.3: Image size comparison
- Develop: 1.66 GB
- Production (Advanced): 236 MB
- Difference: Giảm khoảng ~85.8% (Nguyên nhân do sử dụng kiến trúc Multi-stage build, loại bỏ hoàn toàn các build tools nặng nề như gcc và các file cache rác ở stage 2).

## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment
- URL: https://lab12-production-edb9.up.railway.app
- Screenshot: ![Demo](images/ex3_1.png)

## Part 4: API Security

### Exercise 4.1-4.3: Test results
[Chưa hoàn thành - Đang tiến hành làm]

### Exercise 4.4: Cost guard implementation
[Chưa hoàn thành - Đang tiến hành làm]

## Part 5: Scaling & Reliability

### Exercise 5.1-5.5: Implementation notes
[Chưa hoàn thành - Đang tiến hành làm]
