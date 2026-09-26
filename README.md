# Thiết kế, Tối ưu và Đánh giá Bộ Đa Hợp MUX 4:1 (2-bit)

> **Môn học:** Thiết kế Vi mạch số
> **Đơn vị:** Khoa Kỹ thuật Máy tính - Trường Đại học Công nghệ Thông tin, ĐHQG-HCM  
> **Thành viên thực hiện (Nhóm 10):**  
> - Nguyễn Đình Huy - MSSV: 23520624  
> - Nguyễn Gia Huy - MSSV: 23520629  

---

## 📌 Tổng quan đề tài

Đồ án tập trung vào việc nghiên cứu, hiện thực hóa và tối ưu mạch chọn dữ liệu **MUX 4:1 xử lý Bus dữ liệu 2-bit** bằng công nghệ **CMOS 90nm**. Đề tài thực hiện quy trình thiết kế vi mạch Custom chuẩn (Full-custom IC Design flow): từ đặc tả chức năng, vẽ mạch nguyên lý (Schematic), tối ưu kích thước Transistor ($W/L$), vẽ Layout, kiểm tra DRC/LVS, trích xuất ký sinh (Parasitic Extraction) đến mô phỏng hậu layout (Post-Layout Simulation).

### Đặc tả tín hiệu I/O
* **Ngõ vào dữ liệu (Data inputs):** 4 bus dữ liệu 2-bit: `A[1:0]`, `B[1:0]`, `C[1:0]`, `D[1:0]`.
* **Ngõ vào điều khiển (Select lines):** 2 tín hiệu 1-bit: `S1`, `S0`.
* **Ngõ ra dữ liệu (Data output):** 1 bus dữ liệu 2-bit: `Y[1:0]`.
* **Biểu thức logic cơ bản:**  
  $$Y = \overline{S_1}\ \overline{S_0} A + \overline{S_1} S_0 B + S_1 \overline{S_0} C + S_1 S_0 D$$

---

## 🛠️ Công cụ & Công nghệ sử dụng

* **Công nghệ sản xuất:** CMOS quy trình 90nm.
* **Bộ công cụ EDA (Synopsys):**
  * **Custom Compiler:** Thiết kế Schematic, Symbol và hoàn thiện Physical Layout.
  * **Custom WaveView:** Xem, phân tích dạng sóng mô phỏng (Waveform analysis).
  * **Hercules / IC Validator:** Kiểm tra luật vật lý (DRC) và đối chiếu mạch nguyên lý (LVS).
  * **StarRC:** Trích xuất tụ điện & điện trở ký sinh sau đóng gói Layout.
  * **HSPICE / FineSim:** Chạy mô phỏng Pre-layout và Post-layout.

---

## 🏗️ Ba phương án kiến trúc triển khai

Đề tài hiện thực và khảo sát định lượng qua 3 phương án cấu trúc khác nhau:

1. **Phương án 1 - Phân cấp (Hierarchical - Ghép tầng từ MUX 2:1):**
   * Xây dựng từ cell cơ sở MUX 2:1 tĩnh CMOS (gồm mạng Pull-Up PMOS và Pull-Down NMOS).
   * Ghép nối tầng theo cấu trúc cây (Tree structure): 2 cell MUX 2:1 ở tầng 1 điều khiển bởi `S0` và 1 cell MUX 2:1 ở tầng 2 điều khiển bởi `S1` để tạo thành MUX 4:1 1-bit, sau đó nhân bản cho Bus 2-bit.
2. **Phương án 2 - Logic tĩnh NAND-NAND (Gate-level):**
   * Sử dụng hoàn toàn cổng logic đảo 2 ngõ vào (NAND2) với 2 tầng chọn lọc và hợp nhất.
   * Tối ưu kích thước transistor ($W_p/W_n = 2$ với $W_n = 0.36\,\mu m$, $W_p = 0.72\,\mu m$) để cân bằng dòng chuyển mạch giữa mạng PUN và PDN.
3. **Phương án 3 - Mạch truyền dẫn (Transmission Gate - TG):**
   * Sử dụng cặp NMOS và PMOS mắc song song hoạt động như công tắc chuyển mạch song hướng (Bidirectional switch).
   * Khắc phục nhược điểm sụt áp ngưỡng ($V_{th}$), đảm bảo truyền mức logic Full-swing ($0\text{V} \leftrightarrow V_{DD}$) với số lượng linh kiện tối thiểu.

---

## 📊 Kết quả so sánh PPA (Power - Performance - Area)

Số liệu tổng hợp thực tế sau khi kiểm tra vật lý DRC/LVS sạch lỗi và chạy mô phỏng sau trích xuất ký sinh (Extracted/Post-Layout):

| Tiêu chí đánh giá (PPA) | Đơn vị | Phương án 1: Phân cấp MUX 2:1 | Phương án 2: Cổng NAND-NAND | Phương án 3: Transmission Gate |
| :--- | :---: | :---: | :---: | :---: |
| **Số lượng Transistor** | Con | 72 | 76 | **28** *(Ít nhất)* |
| **Diện tích Layout** | $\mu m^2$ / $nm^2$ | 273 | 109.12 | **54.2** *(Nhỏ nhất)* |
| **Độ trễ $t_{pd}$ (Schematic)** | ps | 305 / 226 | 187 / 200 | **58.5 / 105** |
| **Độ trễ $t_{pd}$ (Extracted)** | ps | 305 / 201 | 209 / 237 | **97.7 / 60.1** |
| **Công suất tiêu thụ ($P_{avg}$)** | $\mu\text{W}$ | 232.8 | 274.8 | **224.4** *(Thấp nhất)* |
| **Tần số tối đa ($f_{max}$)** | GHz | 2.0 | 2.24 | **6.337** *(Cao nhất)* |

### Nhận xét & Đánh giá
* **Transmission Gate (Tối ưu toàn diện):** Vượt trội về mọi mặt (chỉ 28 Transistor, diện tích chiếm dụng nhỏ nhất $54.2\,\mu m^2$, công suất thấp nhất $224.4\,\mu\text{W}$ và tần số hoạt động đạt tới $6.337\,\text{GHz}$). Đây là cấu trúc lý tưởng cho các hệ thống Ultra-low-power và SoC mật độ cao.
* **NAND-NAND:** Thích hợp triển khai theo thư viện cell tiêu chuẩn (Standard Cell Library) với độ trễ lan truyền hai sườn đối xứng, cân bằng.
* **Hierarchical:** Phù hợp với cách tiếp cận thiết kế module phân cấp, dễ kiểm soát nhiễu chéo (crosstalk) và mở rộng cho các bus dữ liệu rộng hơn.

---
