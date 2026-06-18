# 🚀 Dự án Tăng tốc Mô hình MobileNetV2 trên FPGA (CFU-Playground)

Dự án này thực hiện phương pháp **Đồng thiết kế Phần cứng - Phần mềm (Hardware-Software Co-design)** để tối ưu hóa tốc độ suy luận của mạng Neural MobileNetV2 (int8).

## 📊 Kết quả đạt được (Benchmark)
* **Tổng số chu kỳ (Baseline - Thuần phần mềm):** 500,387,190 cycles
* **Tổng số chu kỳ (Hardware-Accelerated):** 224,108,752 cycles
* **Hiệu năng:** Giảm **55.2%** thời gian thực thi, hệ thống tăng tốc **2.23 lần (Speedup 2.23x)**

## 🛠️ Chiến lược tối ưu hóa
1. **Phần cứng (Hardware RTL):** Thiết kế Cỗ máy trạng thái (FSM) với các trạng thái `WAIT_CMD`, `WAIT_INSTRUCTION`, `WAIT_TRANSFER` điều khiển 4 cụm MAC tính toán song song 16 phép tính trong 1 xung nhịp.
2. **Phần mềm (Software):** Áp dụng kỹ thuật **Pipelining (Xử lý luồng gối đầu)** trong file `src/mnv2_conv.cc` để CPU nạp dữ liệu pixel tiếp theo trong lúc phần cứng đang tính toán pixel hiện tại, triệt tiêu độ trễ chờ đợi.

👉 **Mã nguồn dự án của tôi nằm tại:** Thư mục `proj/mnv2_first/`
