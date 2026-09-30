# Parameterized Instruction ROM - RISC-V RV32I

[![Language](https://img.shields.io/badge/Language-SystemVerilog-blue.svg)](https://en.wikipedia.org/wiki/SystemVerilog)
[![Standard](https://img.shields.io/badge/Standard-IEEE%201800--2012%2F2017-brightgreen.svg)]()
[![Target ISA](https://img.shields.io/badge/ISA-RISC--V%20RV32I-red.svg)](https://riscv.org/)
[![Tool](https://img.shields.io/badge/Verified%20with-Vivado%202022.2-orange.svg)]()
[![License](https://img.shields.io/badge/License-MIT-green.svg)]()

Bộ nhớ lệnh **Instruction ROM (`instr_rom`)** là thành phần cung cấp mã máy cho tầng nạp lệnh (**Instruction Fetch - IF stage**) theo kiến trúc bộ nhớ Harvard độc lập. Module được tham số hóa dung lượng linh hoạt, hỗ trợ nạp chương trình trực tiếp bằng file hex qua `$readmemh`.

---

## 📌 Đặc tả Thiết kế & Nguyên lý Đọc Bộ nhớ

```
                       +------------------------+
                       |       instr_rom        |
      addr [31:0] ---->|     (Parameterized)    |-----> instr [31:0]
                       |     DEPTH = 256 words  |
                       +------------------------+
```

### ⚡ Đặc điểm Kỹ thuật Nổi bật:
1. **Truy xuất căn chỉnh Word (Word-Aligned Addressing):**
   - Địa chỉ byte `addr[31:0]` do Program Counter cung cấp được dịch 2 bit sang phải (`addr[31:2]`) để định địa chỉ word trong mảng ROM.
2. **Khởi tạo an toàn (Safe Initialization):**
   - Toàn bộ ROM được mặc định khởi tạo bằng mã lệnh `NOP` (`32'h00000013` = `ADDI x0, x0, 0`).
   - Có thể nạp mã máy chương trình từ file nhị phân/hex bằng `$readmemh("program.hex", rom)`.
3. **Bảo vệ Vùng nhớ (Out-of-Bounds Protection):**
   - Khi địa chỉ vượt quá phạm vi `DEPTH` cấu hình, module tự động trả về lệnh `NOP`, ngăn ngừa việc CPU thực thi dữ liệu rác ngoài tầm kiểm soát.

---

## 🔌 Đặc tả Cổng Giao tiếp & Tham số

### Parameter:
| Tên Parameter | Kiểu dữ liệu | Giá trị mặc định | Ý nghĩa |
| :--- | :---: | :---: | :--- |
| `DEPTH` | `int` | `256` | Số lượng từ lệnh 32-bit trong ROM (mặc định 256 words = 1 KB) |

### Ports:
| Tên cổng | Hướng (Direction) | Độ rộng bit | Ý nghĩa |
| :--- | :---: | :---: | :--- |
| `addr` | Input | `[31:0]` | Địa chỉ byte từ Program Counter (PC) |
| `instr` | Output | `[31:0]` | Lệnh 32-bit lấy ra cấp cho tầng giải mã ID |

---

## 🧪 Kiểm chứng & Mô phỏng (Verification)

Testbench `testbench/tb_instruction_rom.sv` nạp chuỗi lệnh mẫu (`ADDI`, `ADD`, `SW`), kiểm tra việc đọc tuần tự, đọc ngẫu nhiên và kiểm tra cơ chế bảo vệ trả về `NOP` khi truy cập ngoài dải địa chỉ ROM.

### Lệnh chạy mô phỏng:

```bash
xvlog -sv rtl/instruction_rom.sv testbench/tb_instruction_rom.sv
xelab instr_rom_tb -s rom_sim
xsim rom_sim -R
```

---

## 📂 Cấu trúc Thư mục Repo

```
.
├── rtl/
│   └── instruction_rom.sv # RTL Instruction ROM
├── testbench/
│   └── tb_instruction_rom.sv # Self-checking testbench
├── .gitignore
└── README.md
```

---

## 👨‍💻 Thông tin Tác giả & Đồ án

- **Sinh viên thực hiện:** Nguyễn Thành Trung
- **Học phần:** Đồ án Môn học 2 (Capstone Project II) – Ngành Kỹ thuật Máy tính
- **Tên đề tài:** Thiết kế, kiểm chứng và triển khai FPGA lõi vi xử lý RISC-V RV32I 32-bit pipeline 5 tầng ở mức RTL bằng SystemVerilog
- **GitHub cá nhân:** [@thanhchun2005-blip](https://github.com/thanhchun2005-blip)
- **Email:** thanhchun2005@gmail.com
