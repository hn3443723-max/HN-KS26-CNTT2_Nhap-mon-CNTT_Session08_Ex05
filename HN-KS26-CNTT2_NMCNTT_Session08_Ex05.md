# Bài 5 : Xây dựng bộ tài liệu kỹ thuật cho dự án Smart Farm
## Yêu cầu đầu ra 
**Phần 1. Phân tích:** 
- Input:
- + Cảm biến độ ẩm đất: Đo độ ẩm của giá thể trồng lan.
- + Cảm biến nhiệt độ & độ ẩm không khí: Đo điều kiện môi trường xung quanh vườn lan.
- + Cảm biến ánh xạ/ánh sáng: Đo cường độ nắng tránh sốc nhiệt cho lan.
- + Tín hiệu điều khiển thủ công từ người nông dân 
- Output:
- + Van điện từ / Bơm mini (Relay): Đóng/mở nguồn nước tưới hoặc phun sương.
- + Màn hình OLED/LCD tại tủ điều khiển: Hiển thị thông số thời gian thực.
- + Còi báo động/Đèn LED cảnh báo: Báo lỗi mạng, lỗi cảm biến hoặc nước cạn.
- + Gửi dữ liệu và nhận lệnh từ Dashboard

-**Phần 2. Triển khai:**
#  Smart-Farm: Hệ Thống Tưới Thông Minh Cho Vườn Lan

*Hệ thống tự động hóa giám sát độ ẩm, nhiệt độ và điều khiển tưới nước thông minh, giúp vườn lan phát triển tối ưu, tiết kiệm nước và giải phóng sức lao động cho người nông dân.*

---

##  Sơ Đồ Hệ Thống

![Sơ đồ kiến trúc phần cứng và kết nối IoT Smart-Farm](https://rikkeiedu-storage.s3.ap-southeast-2.amazonaws.com/uploads/editor-images/1787648283613_687ec2e7-229c-4942-a347-080fae59560a.webp)

---

##  Hướng Dẫn Cài Đặt & Khởi Chạy

Thực hiện các lệnh dưới đây trên terminal/command line để cấu hình firmware và kết nối thiết bị:

```bash
# 1. Clone mã nguồn từ kho lưu trữ
git clone https://github.com
cd iot-controller

# 2. Cài đặt các thư viện phụ thuộc (cho PlatformIO hoặc Arduino IDE CLI)
pio lib install "adafruit/DHT sensor library"
pio lib install "knolleary/PubSubClient"

# 3. Biên dịch và nạp chương trình vào vi điều khiển ESP32
pio run --target upload

# 4. Giám sát Serial log để kiểm tra kết nối Wi-Fi và cảm biến
pio device monitor -b 115200
```

---

##  Tính Năng Của Hệ Thống

- [x] Tự động đo độ ẩm đất và kích hoạt tưới khi giá thể khô dưới ngưỡng cài đặt.
- [x] Giám sát nhiệt độ và độ ẩm không khí thời gian thực qua điện thoại.
- [x] Chế độ điều khiển thủ công (Manual) qua nút bấm vật lý tại tủ hoặc app từ xa.
- [x] Cảnh báo thông minh khi bơm gặp sự cố hoặc nguồn nước bị hụt.
- [x] Lưu trữ nhật ký hoạt động (Data Logger) lên bộ nhớ đệm và cloud.

---

##  Quy Trình Xử Lý Lệnh Tưới (Fetch – Decode – Execute)

> Khi CPU nhận được một lệnh kích hoạt tưới Lan từ hệ thống tự động hoặc từ người dùng:
> 1. **Fetch (Lấy lệnh):** CPU đọc mã lệnh hoặc gói tin MQTT từ bộ nhớ đệm/vùng đệm truyền thông (ví dụ: `CMD_WATER_ON_01`) đưa vào thanh ghi lệnh (Instruction Register).
> 2. **Decode (Giải mã lệnh):** Bộ giải mã lệnh trong CPU phân tích mã nhị phân, xác định đây là lệnh kích hoạt chân GPIO số 15 tương ứng với Module Relay của bơm nước.
> 3. **Execute (Thực thi):** CPU phát xung điện mức cao (HIGH) ra chân GPIO 15, đóng mạch Relay, dòng điện chạy qua kích hoạt máy bơm nước hoạt động tưới cho vườn lan trong khoảng thời gian đã định.

---
- **Phần 3. Kiểm thử:** 
## Lưu Ý & Xử Lý Sự Cố (Bao gồm phương án khi RAM đầy nhật ký)

* **Hiện tượng sự cố:** RAM thiết bị bị đầy do ghi dữ liệu nhật ký (Log) liên tục mà không được giải phóng hoặc chưa đẩy kịp lên Cloud.
* **Quy trình xử lý an toàn:**
  1. **Cơ chế Ring Buffer (Vòng đệm):** Cấu hình file log trên firmware hoạt động theo cơ chế bộ nhớ đệm vòng (`Circular Buffer`), tự động ghi đè dữ liệu cũ nhất khi dung lượng RAM đạt ngưỡng 85%.
  2. **Cơ chế Flush định kỳ:** Lập trình ép buộc đẩy toàn bộ dữ liệu log từ RAM xuống Flash nội bộ hoặc gửi batch qua MQTT mỗi 10 phút, sau đó thực hiện hàm `free()` giải phóng vùng nhớ RAM.
  3. **Khởi động lại an toàn (Safe Reboot):** Nếu phát hiện dung lượng RAM khả dụng xuống dưới 5%, hệ thống sẽ tự động kích hoạt ngắt an toàn (`ESP.restart()`), đóng tất cả van tưới khẩn cấp để tránh tràn bộ nhớ gây treo van (kẹt bơm gây cháy máy).
  4. **Thao tác thủ công cho người nông dân:** Nhấn giữ nút `RESET` cứng trên mạch 3 giây hoặc tắt/bật lại aptomat tổng của tủ điều khiển để xóa sạch trạng thái rác trên RAM khi cần bảo trì.
Nếu bạn cần bổ sung hoặc điều chỉnh thêm sơ đồ chân kết nối (Pinout diagram) hoặc mã nguồn C++ (Arduino/ESP32) chi tiết cho các cảm biến trên, hãy cho tôi biết nhé!
