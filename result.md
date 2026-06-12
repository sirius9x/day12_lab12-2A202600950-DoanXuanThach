## Exercise 1.2 & 1.3: So sánh Basic vs Advanced

Đây là bảng so sánh sự khác biệt giữa hai phiên bản **Basic** (`develop/app.py`) và **Advanced** (`production/app.py`):

| Feature | Basic (`develop/app.py`) | Advanced (`production/app.py`) | Tại sao quan trọng? (Tại sao bản Advanced lại làm thế) |
|---------|-------------------------|-------------------------------|---------------------------------------------------------|
| **Config** | Hardcode trực tiếp các biến số và secret (như `OPENAI_API_KEY`) vào trong code. | Dùng Environment variables (`Env vars` - thông qua file `config.py`). | Tránh lộ lọt dữ liệu nhạy cảm (API Keys, Passwords) khi đẩy code lên GitHub. Dễ dàng đổi thông số cho từng môi trường (Dev, Staging, Prod) mà không cần sửa code. |
| **Health check** | ❌ Không có | ✅ Có (`/health` và `/ready`) | Các nền tảng Cloud (Render, Railway, Kubernetes) cần endpoint này để liên tục hỏi thăm xem server "còn sống không" hay "đã sẵn sàng nhận traffic chưa" để có thể tự động Restart nếu bị treo. |
| **Logging** | Dùng `print()` đơn giản (thậm chí in luôn cả secret key ra màn hình). | Dùng thư viện `logging` xuất ra chuẩn **Structured JSON**. | Ghi log dạng JSON giúp các công cụ theo dõi log chuyên nghiệp (Datadog, Elastic, Loki...) dễ dàng bóc tách, lọc, tìm kiếm và cảnh báo lỗi ở quy mô lớn. Không bao giờ log secret trong production. |
| **Shutdown** | Đột ngột (Bấm tắt là đứt gãy ngay lập tức). | **Graceful shutdown** (dùng `lifespan` và bắt tín hiệu `SIGTERM`). | Cho phép app có vài giây hoàn thành trả nốt kết quả cho những request đang xử lý dở dang và giải phóng tài nguyên trước khi tắt hẳn. Tránh làm client bị rớt kết nối đột ngột. |
| **Network & Port** | Bind cứng vào `localhost` và port cố định `8000`. | Bind vào `0.0.0.0` và nhận `PORT` linh hoạt từ biến môi trường. | Rất quan trọng! Để chạy được bên trong Docker container và triển khai lên Cloud thì bắt buộc phải bind ra `0.0.0.0`, và Cloud platform luôn cấp cho app một PORT ngẫu nhiên qua biến môi trường. |

<br/>

## Exercise 2.1: Dockerfile cơ bản

**1. Base image là gì?**
- Base image là **`python:3.11`**. Đây là phiên bản phân phối đầy đủ (full distribution) của môi trường Python 3.11, làm nền móng để chạy ứng dụng.

**2. Working directory là gì?**
- Working directory là **`/app`**. Đây là thư mục làm việc mặc định bên trong container. Các lệnh phía sau (`COPY`, `RUN`, `CMD`) đều sẽ được thực thi bên trong thư mục này.

**3. Tại sao COPY requirements.txt trước?**
- Để tận dụng **Docker Layer Cache**. Mã nguồn (`app.py`) thường xuyên thay đổi, còn danh sách thư viện (`requirements.txt`) thì ít thay đổi hơn. Việc copy và chạy `pip install` trước giúp Docker "lưu nháp" quá trình cài đặt thư viện. Khi sửa code, Docker chỉ build lại phần code mới, giúp giảm thời gian build từ vài phút xuống vài giây.

**4. CMD vs ENTRYPOINT khác nhau thế nào?**
- **`CMD`**: Là lệnh *mặc định* chạy khi khởi động container, **rất dễ bị ghi đè**. Nếu bạn chạy `docker run <image> bash`, lệnh mặc định sẽ bị hủy và thay bằng lệnh `bash`.
- **`ENTRYPOINT`**: Là lệnh *bắt buộc* và **khóa cứng** chương trình sẽ chạy. Những tham số phía sau lệnh `docker run` sẽ được truyền thêm vào chứ không ghi đè lệnh gốc (ví dụ: chạy `docker run <image> --help` thì Docker coi `--help` là tham số phụ).

<br/>

## Exercise 2.3: Multi-stage build

**1. Stage 1 (Builder) làm gì?**
- Stage 1 đóng vai trò là "xưởng mộc". Nó sử dụng image base (`python:3.11-slim`), cài đặt thêm các công cụ biên dịch (`gcc`, `libpq-dev`, v.v.), sau đó chạy `pip install` để cài đặt và biên dịch tất cả các thư viện Python (dependencies) vào thư mục của user. Nhiệm vụ duy nhất của nó là chuẩn bị xong xuôi các dependencies.

**2. Stage 2 (Runtime) làm gì?**
- Stage 2 là "ngôi nhà hoàn thiện". Nó khởi tạo từ một image `python:3.11-slim` mới, tạo một user không có quyền root (để bảo mật), và **CHỈ COPY** những thư viện đã cài đặt thành công từ Stage 1 sang. Sau đó nó copy mã nguồn `app.py` vào và chạy ứng dụng.

**3. Tại sao image lại nhỏ hơn?**
- Vì toàn bộ rác, file tạm (cache), và các công cụ biên dịch nặng nề (`gcc`, v.v.) chỉ tồn tại ở Stage 1 và **bị vứt bỏ hoàn toàn** khi Docker kết thúc quá trình build. Image cuối cùng (Stage 2) chỉ chứa đúng môi trường Python cơ bản + Code của bạn + Thư viện đã build sẵn, giúp image siêu nhẹ và an toàn.
