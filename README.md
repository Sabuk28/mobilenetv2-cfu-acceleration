<img width="624" height="275" alt="image" src="https://github.com/user-attachments/assets/29abbc37-2ae0-4b6a-b044-e0e4f674ea64" /># 🚀 Dự án Tăng tốc Mô hình MobileNetV2 trên FPGA (CFU-Playground)

Dự án này thực hiện phương pháp **Đồng thiết kế Phần cứng - Phần mềm (Hardware-Software Co-design)** để tối ưu hóa tốc độ suy luận của mạng Neural MobileNetV2 (int8).

## 📊 Kết quả đạt được (Benchmark)
* **Tổng số chu kỳ (Baseline - Thuần phần mềm):** 500,387,190 cycles
* **Tổng số chu kỳ (Hardware-Accelerated):** 224,108,752 cycles
* **Hiệu năng:** Giảm **55.2%** thời gian thực thi, hệ thống tăng tốc **2.23 lần (Speedup 2.23x)**

## 🛠️ Chiến lược tối ưu hóa
1. **Phần cứng (Hardware RTL):** Thiết kế Cỗ máy trạng thái (FSM) với các trạng thái `WAIT_CMD`, `WAIT_INSTRUCTION`, `WAIT_TRANSFER` điều khiển 4 cụm MAC tính toán song song 16 phép tính trong 1 xung nhịp.
2. **Phần mềm (Software):** Áp dụng kỹ thuật **Pipelining (Xử lý luồng gối đầu)** trong file `src/mnv2_conv.cc` để CPU nạp dữ liệu pixel tiếp theo trong lúc phần cứng đang tính toán pixel hiện tại, triệt tiêu độ trễ chờ đợi.

👉 **Mã nguồn dự án của tôi nằm tại:** Thư mục `proj/mnv2_first/`


**CÁC BƯỚC MÔ PHỎNG**
**1. Chuẩn bị môi trường và lấy mã nguồn (Environment Setup)**
	Để triển khai dự án Hardware-Software Co-design, nhóm sử dụng môi trường hệ điều hành Ubuntu 22.04 LTS. Quá trình tải mã nguồn và khởi tạo môi trường được thực hiện tuần tự qua các bước sau:

**Bước 1:** Cài đặt các công cụ hệ thống cơ bản
Trước khi tải mã nguồn, hệ thống cần được cài đặt các công cụ biên dịch và quản lý phiên bản cơ bản. Nhóm sử dụng Terminal của Ubuntu để thực thi:
<img width="451" height="128" alt="image" src="https://github.com/user-attachments/assets/559aa63a-f84c-44d5-be66-e5b6b645d793" />

**Bước 2: **Tải mã nguồn CFU-Playground từ GitHub
Mã nguồn toàn bộ dự án được cung cấp dưới dạng mã nguồn mở bởi Google. Nhóm tiến hành nhân bản (clone) kho lưu trữ từ GitHub về máy tính cục bộ:
<img width="559" height="120" alt="image" src="https://github.com/user-attachments/assets/e401ebad-15fb-4553-b0a0-567233748284" />

**Bước 3: **Thiết lập môi trường và Toolchain tự động
Dự án CFU-Playground đòi hỏi rất nhiều công cụ phức tạp (Trình biên dịch chéo RISC-V GCC, máy ảo Renode, phần mềm tổng hợp Verilator, và Yosys). Thay vì cài đặt thủ công, nhóm chạy kịch bản tự động do Google cung cấp:
<img width="248" height="105" alt="image" src="https://github.com/user-attachments/assets/f1927d00-defd-4829-a95f-6c8d5df2cb3a" />

	Giải thích: Lệnh này sẽ tự động tải các toolchain cần thiết và tạo ra một môi trường biệt lập tên là cfu-common thông qua hệ thống quản lý gói Conda. Điều này đảm bảo các phiên bản thư viện phần cứng không bị xung đột với hệ thống máy tính của nhóm.
  <img width="624" height="275" alt="image" src="https://github.com/user-attachments/assets/bff6e10b-a8cd-4100-8500-8631354097ab" />

**2. Chuẩn bị trước khi Mô phỏng (Pre-Simulation Configuration)**

Tại đây, trước khi khởi chạy mô phỏng, nhóm cần cấu hình file Makefile để quyết định việc hệ thống sẽ chạy bằng Phần mềm (Baseline) hay chạy bằng Phần cứng (Co-design). Nhóm mở file Makefile và thực hiện bật/tắt (uncomment/comment) cờ biên dịch:


**3. Tiến hành Khởi chạy Mô phỏng chạy thuần phần mềm **
Sau khi lưu cấu hình Makefile, nhóm thực thi lệnh tổng hợp và khởi chạy trình giả lập Renode:

Quá trình xử lý ngầm của hệ thống sau lệnh make renode:
  **1.Phần cứng (Hardware): **Trình biên dịch sẽ gọi công cụ Amaranth để dịch các thiết kế viết bằng Python (cfu.py) sang chuẩn RTL Verilog (cfu.v). Sau đó, công cụ Verilator sẽ biên dịch file Verilog này thành các mô hình nhúng C++ để đưa vào giả lập.

  **2.Phần mềm (Software): **Trình biên dịch chéo (RISC-V Toolchain) sẽ biên dịch mô hình AI (TensorFlow Lite Micro) cùng file nhân tùy chỉnh (mnv2_conv.cc) thành file nhị phân .elf.

 ** 3.Hợp nhất (Co-simulation):** Máy ảo Renode được bật lên, nạp file .elf vào vi xử lý VexRiscv (CPU) và kết nối với khối Verilator (CFU) để mô phỏng một hệ thống SoC hoàn chỉnh.
Khi Renode khởi động thành công, hệ thống mở ra 2 cửa sổ: Cửa sổ Monitor (theo dõi log hệ thống) và Cửa sổ UART (giao diện tương tác của CPU).
 
**Các bước lấy kết quả đánh giá:**
Tại cửa sổ UART, nhóm nhập phím 1 (TfLM Models menu) rồi tiếp tục chọn phím 2 (Mobile Net v2 models).
Hệ thống yêu cầu chọn phương thức kiểm thử, nhóm chọn phím 1 (Run test 1) để chạy một vòng suy luận mô hình hoàn chỉnh.
Bộ đếm hiệu năng (Performance Counter) của hệ thống sẽ bắt đầu ghi nhận số chu kỳ (Ticks) cho từng lớp của mạng nơ-ron.
Sau khi phân loại xong, CPU in ra dòng cuối cùng tổng kết số chu kỳ xung nhịp (cycles total). Nhóm lấy con số này làm thước đo (Benchmark) để so sánh hiệu năng.
Để kết thúc, nhóm chuyển sang cửa sổ Renode Monitor và gõ lệnh quit để tắt mô phỏng an toàn, dọn dẹp bộ nhớ đệm.
 
**4. Tiến hành Khởi chạy Mô phỏng Hardware-Accelerated (Kích hoạt bộ tăng tốc CFU)**
Nhóm gỡ chú thích để trình biên dịch GCC liên kết các lệnh gọi Custom Instruction xuống phần cứng FPGA:

Sau khi lưu cấu hình Makefile, nhóm thực thi lệnh tổng hợp và khởi chạy trình giả lập Renode:

Làm lại các bước lấy kết quả đánh giá ở trên:
