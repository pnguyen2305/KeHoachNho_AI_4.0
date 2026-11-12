# EcoArcade Work Plan — Trạm Cân Phế Liệu Đổi Vé Arcade Cho Trẻ Em

> **Trạng thái tài liệu (12/11/2026):** Bản kế hoạch kỹ thuật gốc (Master Engineering Plan) đã được đồng bộ hóa với bản vẽ thiết kế cơ khí tổng thể (`900x740x1450 mm`), quy chuẩn quy đổi $1\text{ kg}/\text{vé}$ (sai số cho phép $\pm 50\text{ g}$) và hạn chót hoàn thành mới là **05/03/2027**. Mọi tiến độ thực hiện, sơ đồ mạch Proteus/STM32 và kết quả kiểm thử sẽ được ghi nhận vào `CHECKPOINTS.md` và `DECISIONS.md`.

## 1. Mục tiêu, phạm vi và hạn chót (Goal, scope, and deadline)

Dự án **EcoArcade** hướng tới việc chế tạo một trạm tương tác vật lý hoàn chỉnh phục vụ sự kiện cộng đồng dành cho trẻ em theo đúng quy cách bản vẽ thiết kế (`900 mm x 740 mm x 1450 mm`, tổng cao `1600 mm` tính cả đèn báo). Khi trẻ em mang phế liệu đặt lên khay cân (`340x340 mm`) và nhấn nút Arcade ($\phi 100\text{ mm}$), hệ thống sẽ hiển thị khối lượng và số vé lên bảng **2 x LED P10 (`640x160 mm`)**, phát âm thanh vui nhộn và **nhả ra một dải vé vật lý (Arcade Ticket Strip)** qua miệng phóng vé (`70x12 mm`).

Hệ thống phải thỏa mãn 5 mục tiêu kỹ thuật cốt lõi:

1. Đo lường khối lượng phế liệu trong dải từ $0.10\text{ kg}$ đến $20.00\text{ kg}$ với sai số tuyệt đối tối đa $\le \pm 0.05\text{ kg}$ ($\pm 50\text{ g}$);
2. Quy đổi khối lượng sang số lượng vé theo định mức chuẩn **$1\text{ kg} = 1\text{ vé}$** và nhả chính xác số lượng vé vật lý bằng cơ cấu **Arcade Ticket Dispenser** (có phản hồi quang học đếm từng vé);
3. Tích hợp hệ thống **Đèn tháp báo hiệu 3 màu (Đỏ - Vàng - Xanh)** trên đỉnh máy ($1600\text{ mm}$) để thông báo trạng thái hoạt động (Bình thường / Sắp hết vé / Lỗi kẹt vé hoặc quá tải) cho tình nguyện viên từ xa;
4. Đảm bảo an toàn điện - cơ khí tuyệt đối cho trẻ em (điện áp hoạt động nội bộ $\le 12\text{ V DC}$, có cửa hông có khóa để thay vé và cửa hậu bảo trì);
5. Lưu trữ nhật ký vận hành (tổng số lượt cân, tổng số $\text{kg}$ phế liệu thu gom, tổng số vé phát ra) để báo cáo kết quả sự kiện.

- **Mốc thời gian chính thức (Official Event Ready Deadline):** **23:59, 05/03/2027**.
- **Mốc hoàn thành nội bộ (Internal Feature Freeze):** **21:00, 22/02/2027**, dành ra 11 ngày dự phòng để chạy thử nghiệm chịu tải (Stress Test), hoàn thiện thẩm mỹ khung vỏ và xử lý lỗi phát sinh trước hạn chót 05/03/2027.

### Phạm vi các yêu cầu kỹ thuật (Requirements)

- **Requirement 1.1** — Khảo sát, chốt sơ đồ khối phần cứng (STM32), mô phỏng logic cốt lõi trên Proteus và chốt danh mục vật tư (BOM);
- **Requirement 1.2** — Đọc và lọc nhiễu tín hiệu Loadcell qua ADC 24-bit `HX711` đạt độ chính xác $\pm 50\text{ g}$;
- **Requirement 1.3** — Thuật toán tự động trừ bì (Auto-Tare), chống gian lận (Anti-cheat) và quy đổi $1\text{ kg}/\text{vé}$;
- **Requirement 1.4** — Điều khiển động cơ nhả vé Arcade (Ticket Dispenser) kết hợp ngắt ngoài (`EXTI`) đọc cảm biến quang học đếm vạch vé;
- **Requirement 1.5** — Giao diện hiển thị & âm thanh: Cụm `2 x LED P10 (640x160 mm)`, mạch âm thanh `DFPlayer Mini` + Loa $3\text{ W}$ ($\phi 100\text{ mm}$), nút bấm Arcade $\phi 100\text{ mm}$ và Đèn tháp 3 màu báo hiệu cho tình nguyện viên;
- **Requirement 1.6** — Thiết kế mạch nguyên lý (Schematic), bố trí bảng điện và hệ thống cấp nguồn ($12\text{ V DC}$ và $5\text{ V DC}$ công suất cao cho LED P10);
- **Requirement 1.7** — Thiết kế và chế tạo khung vỏ cơ khí theo đúng kích thước bản vẽ (`900x740x1450 mm`), khay cân (`340x340x25 mm`) và cơ cấu chống quá tải;
- **Requirement 1.8** — Xử lý ngoại lệ phần cứng (hết vé, kẹt vé, quá tải bàn cân) và kích hoạt đèn báo tình nguyện viên;
- **Requirement 1.9** — Ghi nhận dữ liệu thống kê sự kiện (`event_metrics.json`, `transactions.csv`);
- **Requirement 1.10** — Quy trình kiểm thử độc lập (Calibration & Stress Testing) với tải chuẩn.

### Bảng tiêu chí đánh giá dự án (Self-Grading Rubric — 100 điểm)

| Tiêu chí | Điểm | Điều kiện vượt qua (Gate) |
|---|---:|---|
| Độ chính xác của cân (Weighing Accuracy) | 25 | Sai số $\le \pm 50\text{ g}$ trên toàn dải đo sau 50 lần đặt tải liên tiếp trên khay $340\times 340\text{ mm}$ |
| Độ tin cậy nhả vé (Ticket Dispensing) | 25 | Nhả đúng 100% số vé theo quy tắc $1\text{ kg}/\text{vé}$, tự ngắt đúng vạch cắt, báo đèn đỏ khi hết/kẹt vé |
| Hiển thị P10 & Độ bền cơ khí (UX & Enclosure) | 20 | Bảng `2 x LED P10` hiển thị rõ nét khối lượng + số vé, khung vỏ chắc chắn đúng kích thước bản vẽ |
| Kiến trúc phần mềm & Mô phỏng (Firmware) | 15 | Code STM32 chia module rõ ràng, có file mô phỏng Proteus cho luồng State Machine & đếm xung vé |
| Hồ sơ kỹ thuật & GitHub (Documentation) | 15 | Đầy đủ sơ đồ mạch, bản vẽ cơ khí, hướng dẫn lắp ráp và lịch sử Git commit rõ ràng |

---

## 2. Cấu trúc kho lưu trữ và phân định thành phần (Planned artifacts)

```text
eco-arcade-scale/
├── PROJECT_PLAN.md                      # File kế hoạch tổng thể này
├── BUILD_FROM_SCRATCH_GUIDE.md          # Cẩm nang linh kiện, đi dây, làm vỏ thủ công & học tập
├── README.md                            # Giới thiệu dự án, hình ảnh 3D, cách nạp code STM32
├── DECISIONS.md                         # Ghi chép các quyết định thiết kế (Architectural Decisions)
├── CHECKPOINTS.md                       # Nhật ký tiến độ hàng tuần và minh chứng thực nghiệm
├── BOM.csv                              # Danh mục linh kiện, thông số, nơi mua và chi phí
│
├── simulation_proteus/                  # Mô phỏng mạch và logic trên Proteus
│   ├── eco_arcade_stm32_sim.pdsprj      # File project Proteus 8.x
│   └── README_SIM.md                    # Hướng dẫn chạy mô phỏng Loadcell giả lập & xung vé
│
├── hardware/
│   ├── schematic/                       # Sơ đồ nguyên lý & bố trí dây (EasyEDA / KiCad / Proteus)
│   │   ├── eco_arcade_schematic.pdf
│   │   └── wiring_diagram.png
│   └── mechanical_drawings/             # Bản vẽ cơ khí & cắt tấm (Fusion 360 / AutoCAD)
│       ├──overall_dimensions_2d.png     # Bản vẽ kích thước tổng thể 900x740x1450 mm
│       ├── render_3d.png                # Hình ảnh phối cảnh 3D của trạm cân
│       ├── scale_tray_340x340.step      # Thiết kế khay cân và đế gá Loadcell
│       └── panel_cutting_list.pdf       # Bảng kê kích thước cắt khung nhôm / gỗ / Formex
│
├── firmware/                            # Code nhúng cho vi điều khiển STM32 (STM32CubeIDE / PlatformIO)
│   ├── Core/
│   │   ├── Inc/
│   │   │   ├── config.h                 # Định nghĩa chân GPIO, hệ số Calibration, quy tắc 1kg/vé
│   │   │   ├── state_machine.h          # Các trạng thái hoạt động của máy
│   │   │   ├── hx711_driver.h           # Thư viện đọc ADC HX711 và lọc trung vị
│   │   │   ├── ticket_dispenser.h       # Điều khiển motor 12V và ngắt EXTI đếm vé
│   │   │   ├── p10_matrix.h             # Điều khiển 2 tấm LED P10 (640x160) qua chuẩn HUB12
│   │   │   └── tower_light_audio.h      # Điều khiển đèn tháp 3 màu & DFPlayer Mini
│   │   └── Src/
│   │       ├── main.c                   # Vòng lặp chính và điều phối State Machine
│   │       ├── hx711_driver.c
│   │       ├── ticket_dispenser.c
│   │       ├── p10_matrix.c
│   │       └── tower_light_audio.c
│   └── test/
│       └── test_ticket_conversion.c     # Kiểm thử công thức quy đổi 1kg/vé và biên độ +-50g
│
├── media/
│   ├── sd_card_audio/                   # Các file âm thanh MP3 (0001_welcome.mp3, ...)
│   └── decal_artwork/                   # File thiết kế Decal dán thùng máy
│
└── data_logs/
    ├── sample_transactions.csv          # Dữ liệu mẫu xuất ra từ thiết bị
    └── event_metrics.json               # Thống kê tổng hợp sau sự kiện
```

---

## 3. Các quy ước kỹ thuật cố định (Shared contracts to freeze)

### 3.1 Công thức quy đổi khối lượng sang số vé (Weight-to-Ticket Mapping)

Gọi $m$ (đơn vị: $\text{g}$) là khối lượng thực tế sau khi đã trừ bì và ổn định trên khay cân $340\times 340\text{ mm}$:

- **Độ phân giải hiển thị trên bảng LED P10:** Hiển thị theo đơn vị $\text{kg}$ với 2 chữ số thập phân (ví dụ: `2.35kg  14 VÉ` hoặc theo quy chuẩn mới `2.35kg  02 VÉ`).
- **Sai số cho phép tối đa ($e_{\max}$):** $\pm 50\text{ g}$ ($\pm 0.05\text{ kg}$).
- **Định mức quy đổi chuẩn ($\Delta m$):** **$1\text{ kg} = 1000\text{ g} = 1\text{ vé}$**.
- **Ngưỡng bù sai số có lợi cho trẻ em (Grace Margin):** Do sai số cho phép là $\pm 50\text{ g}$, để trẻ em không bị thiệt khi mang đúng túi $1\text{ kg}$ nhưng cân đo ra $960\text{ g}$, ngưỡng tính $1\text{ vé}$ đầu tiên được tính từ $m_{\min} = 950\text{ g}$ (tức cứ mỗi mốc $1000\text{ g}$ được cộng dung sai $+50\text{ g}$ khi làm tròn xuống):

$$N_{\text{tickets}}(m) = \left\lfloor \frac{m + 50}{1000} \right\rfloor \quad \text{với } m \ge 950\text{ g}$$

*(Ví dụ: Nếu cân đo được $m = 960\text{ g} \rightarrow \lfloor 1010 / 1000 \rfloor = 1\text{ vé}$; nếu $m = 2350\text{ g} \rightarrow \lfloor 2400 / 1000 \rfloor = 2\text{ vé}$. Bạn cũng có thể cấu hình hệ số nhân sự kiện $k$ trong `config.h` nếu ban tổ chức muốn đổi tỉ lệ vào phút chót).*

- **Giới hạn an toàn tối đa ($m_{\max}$):** $20000\text{ g}$ ($20.0\text{ kg}$). Nếu $m > 20000\text{ g}$, máy phát còi cảnh báo quá tải và bật đèn Đỏ trên đỉnh tháp.

### 3.2 Tiêu chuẩn ổn định cân (Weight Stability Contract)

Với sai số cho phép $\pm 50\text{ g}$, điều kiện xác nhận cân đã ổn định (Stable) được nới lỏng hợp lý giúp máy phản hồi cực nhanh: Biên độ dao động giữa $K = 8$ mẫu đọc liên tiếp (trong khoảng $0.8\text{ s}$) nhỏ hơn ngưỡng $\epsilon = 30\text{ g}$:

$$\max_{1 \le k \le K} m_k - \min_{1 \le k \le K} m_k \le 30\text{ g}$$

### 3.3 Quy ước trạng thái Đèn tháp báo hiệu cho Tình nguyện viên (Tower Light Contract)

Đèn tháp 3 màu trên đỉnh máy (cao $1600\text{ mm}$) giúp tình nguyện viên quan sát từ xa mà không cần mở máy:

| Màu đèn tháp | Trạng thái hệ thống | Hành động của tình nguyện viên |
|---|---|---|
| **XANH LÁ (Sáng tĩnh)** | Máy đang hoạt động bình thường (`IDLE` / `WEIGHING` / `DISPENSING`) | Không cần can thiệp |
| **VÀNG (Nhấp nháy)** | Khối lượng trên cân chưa đủ $1\text{ kg}$ hoặc đang nhắc bé lấy phế liệu cũ ra khỏi khay cân | Hướng dẫn bé bỏ thêm phế liệu hoặc nhấc túi phế liệu đã cân ra |
| **ĐỎ (Nhấp nháy + Còi)** | **Lỗi phần cứng:** Hết vé trong khay, kẹt vé tại miệng `70x12`, hoặc quá tải bàn cân ($> 20\text{ kg}$) | Mở **Cửa hông thay vé (có khóa)** để nạp thêm vé mới hoặc gỡ vé kẹt |

### 3.4 Chu trình máy trạng thái hữu hạn (Finite State Machine — FSM)

```text
[IDLE (Đèn Xanh)]
   -> (Phát hiện m >= 100g)        -> [WEIGHING_STABILIZING]
   -> (Ổn định nhưng m < 950g)     -> [UNDER_MIN_WEIGHT (Đèn Vàng nhắc thêm đồ)]
   -> (Ổn định và m >= 950g)       -> [READY_FOR_BUTTON (Sáng đèn nút Arcade phi 100)]
   -> (Trẻ nhấn nút Arcade)        -> [DISPENSING_TICKETS (Nhả N vé qua miệng 70x12)]
   -> (Nhả đủ N vé)                -> [WAIT_REMOVE_ITEMS (Đèn Vàng nhắc lấy đồ ra)]
   -> (Khay cân về 0g +- 50g)      -> [IDLE (Auto-Tare)]
```

---

## 4. Phân rã gói công việc chi tiết (Work Packages)

### 4.1 Work Package A — Mô phỏng Proteus & Phân hệ Đo lường Khối lượng (STM32 + HX711)

- **Mục đích:** Làm chủ vi điều khiển `STM32F103C8T6` (BluePill), mô phỏng luồng hoạt động trên **Proteus** và đọc chính xác cảm biến Loadcell thực tế với sai số $\le \pm 50\text{ g}$.
- **Nội dung triển khai:**
  - Khởi tạo project trên **STM32CubeIDE**, cấu hình GPIO, Timer, EXTI và UART.
  - Dựng mạch mô phỏng trên **Proteus 8**: Mô phỏng STM32 đọc tín hiệu cân giả lập, nhận nút nhấn Arcade, xuất xung điều khiển motor nhả vé, nhận xung phản hồi vé qua chân ngắt `EXTI` và điều khiển 3 đèn báo Xanh - Vàng - Đỏ.
  - Lắp mạch thật trên Breadboard: Kết nối thanh Loadcell ($20\text{ kg}$ hoặc $30\text{ kg}$) + `HX711` vào STM32, viết hàm lọc trung bình trượt và hàm tự động trừ bì (`Tare`).
- **Kiểm thử (Tests):**
  - Đặt các mức tải $1\text{ kg}$, $2\text{ kg}$, $5\text{ kg}$, $10\text{ kg}$ lên khay cân thử nghiệm; độ lệch hiển thị không quá $\pm 50\text{ g}$.
- **Planned commits:**
  ```text
  wp-a: create stm32cubeide project and proteus simulation schematic (req 1.1)
  wp-a: implement hx711 driver, moving average filter and auto-tare (req 1.2, 1.3)
  ```

---

### 4.2 Work Package B — Phân hệ Nhả vé Arcade & Đèn báo Tình nguyện viên

- **Mục đích:** Điều khiển máy nhả vé Arcade $12\text{ V DC}$ nhả đúng $1\text{ vé}$ cho mỗi $1\text{ kg}$ và kích hoạt đèn tháp báo lỗi khi hết/kẹt vé.
- **Nội dung triển khai:**
  - Đấu nối module MOSFET kích motor $12\text{ V DC}$ của máy nhả vé và mạch chia áp đưa xung cảm biến quang (`Notch Sensor`) về chân ngắt ngoài `EXTI` của STM32.
  - Đấu nối module Relay/MOSFET 3 kênh điều khiển **Đèn tháp 3 màu ($12\text{ V}$)** trên đỉnh máy.
  - Cài đặt bộ đếm thời gian bảo vệ (Watchdog Timeout $1.5\text{ s}$): Nếu motor quay nhưng không có xung vé đi qua miệng `70x12 mm` $\rightarrow$ dừng motor ngay lập tức và bật đèn Đỏ gọi tình nguyện viên.
- **Planned commits:**
  ```text
  wp-b: implement ticket dispenser motor control and exti notch counter (req 1.4)
  wp-b: add timeout jam detection and 3-color volunteer tower light logic (req 1.5, 1.8)
  ```

---

### 4.3 Work Package C — Hiển thị Bảng `2 x LED P10 (640x160)`, Âm thanh & Nút Arcade $\phi 100$

- **Mục đích:** Hoàn thiện toàn bộ giao diện mặt trước đúng như hình render 3D (`2.35kg  02 VÉ`), âm thanh `Loa 3W` và nút bấm Arcade $\phi 100\text{ mm}$.
- **Nội dung triển khai:**
  - Ghép nối 2 tấm module ma trận **LED P10 ($320\times 160\text{ mm}$ mỗi tấm)** thành bảng **`640x160 mm`** (64x16 điểm ảnh) qua cổng `HUB12`.
  - Lập trình hiển thị font chữ lớn bên trái là số cân (`XX.XXkg`) và bên phải là số vé (`XX VÉ`).
  - Kết nối module `DFPlayer Mini` qua cổng `UART` của STM32 ra loa $3\text{ W}$ (lưới $\phi 100\text{ mm}$) và đèn nền trong nút bấm Arcade $\phi 100\text{ mm}$.
  - Tích hợp hoàn chỉnh 6 trạng thái của State Machine.
- **Planned commits:**
  ```text
  wp-c: implement hub12 driver for dual p10 led matrix 640x160 display (req 1.5)
  wp-c: integrate dfplayer mini audio, 100mm arcade button and full fsm (req 1.3, 1.5)
  ```

---

### 4.4 Work Package D — Chế tạo Khung vỏ Cơ khí (`900x740x1450 mm`) & Khay cân (`340x340 mm`)

- **Mục đích:** Hiện thực hóa mô hình vật lý theo đúng bản vẽ 2D/3D với phương pháp thi công tối ưu nhất cho sinh viên.
- **Nội dung triển khai:**
  - **Khung chịu lực bên trong:** Lắp ghép khung xương bằng **nhôm định hình $20\times 20\text{ mm}$** (bắt ke góc vuông không cần hàn) hoặc khung gỗ/sắt hộp chia làm 3 khối module tháo rời được để dễ vận chuyển:
    1. Khối bệ cân trái (`rộng 490 mm, cao 459 mm`) + Khay cân `340x340 mm, vành 25 mm`.
    2. Khối trụ nhả vé phải (`rộng 410 mm, cao 878 mm`) có mặt vát gắn nút Arcade $\phi 100$, miệng vé `70x12 mm`, khay hứng vé và **cửa hông có khóa** để thay vé.
    3. Khối thân chính & trán đèn P10 phía sau (`rộng 900 mm, cao 1450 mm, sâu 320 mm`) có lưới loa $\phi 100$, khe thông gió, cửa hậu và cột đèn báo cao `1600 mm`.
  - **Vỏ bao che bên ngoài:** Ốp tấm **Formex (PVC Foam) dày $5\text{ mm} - 8\text{ mm}$** hoặc gỗ MDF cắt CNC, dán Decal màu Trắng - Xanh lá - Vàng đúng như phối cảnh 3D.
- **Planned commits:**
  ```text
  wp-d: finalize structural frame andpanel cutting dimensions for 900x740x1450 cabinet (req 1.7)
  wp-d: assemble 340x340 scale tray with overload protection bolts (req 1.7)
  ```

---

### 4.5 Work Package E — Lắp ráp Tổng thể, Kiểm chuẩn & Nghiệm thu trước 05/03/2027

- **Mục đích:** Đi dây hoàn thiện vào thùng máy, kiểm tra độ bền 100 lượt cân và đóng gói tài liệu GitHub phục vụ sự kiện & đưa vào CV.
- **Nội dung triển khai:**
  - Cố định bộ nguồn tổ ong $5\text{ V} - 10\text{ A}$ (nuôi 2 tấm LED P10) và nguồn $12\text{ V} - 5\text{ A}$ (nuôi máy nhả vé, đèn tháp, đèn nút bấm) vào trong thùng máy.
  - Đi dây gọn gàng qua ống gen, kiểm tra khóa cửa hông thay vé và cửa hậu.
  - Chạy kiểm thử thực tế 100 lượt cân phế liệu, quay video quá trình vận hành và hoàn thiện hồ sơ GitHub.
- **Planned commits:**
  ```text
  wp-e: complete internal power distribution and cabinet wiring harness (req 1.6, 1.7)
  wp-e: pass 100-cycle validation test and finalize github documentation (req 1.9, 1.10)
  ```

---

## 5. Lịch trình chi tiết từ 12/11/2026 đến 05/03/2027 (Planned Calendar)

| Giai đoạn / Thời gian | Mục tiêu bắt buộc cần hoàn thành (Required Outcome) | Checkpoint Tag |
|---|---|---|
| **12/11 – 22/11/2026** | Khởi tạo GitHub Repo, dựng xong mạch mô phỏng **Proteus + STM32**, đặt mua linh kiện điện tử (STM32, HX711, Loadcell, Máy nhả vé, 2 tấm LED P10, Đèn tháp) | `milestone-01-sim-bom` |
| **23/11 – 08/12/2026** | Hoàn thành **WP-A**: Đấu nối Loadcell + HX711 với STM32 trên mạch thật, lọc nhiễu và hiệu chuẩn đạt sai số $\le \pm 50\text{ g}$ | `milestone-02-scale` |
| **09/12 – 25/12/2026** | Hoàn thành **WP-B**: Điều khiển máy nhả vé Arcade $12\text{ V}$, đếm vé bằng ngắt `EXTI`, xử lý kẹt/hết vé và điều khiển Đèn tháp 3 màu | `milestone-03-dispenser` |
| **26/12/2026 – 15/01/2027** | Hoàn thành **WP-C**: Điều khiển bảng `2 x LED P10 (640x160)`, âm thanh `DFPlayer Mini`, nút bấm $\phi 100$; ghép nối toàn bộ State Machine chạy mượt trên bàn thử | `milestone-04-firmware` |
| **16/01 – 05/02/2027** | Hoàn thành **WP-D (Phần 1)**: Lên bảng kê cắt vật liệu, lắp dựng phần khung xương chịu lực và khay cân `340x340 mm` có ốc chống quá tải | `milestone-05-frame` |
| **06/02 – 22/02/2027** | **Mốc đóng băng nội bộ (Internal Freeze):** Ốp hoàn thiện vỏ thùng máy, sơn/dán màu theo bản render 3D, lắp toàn bộ linh kiện và đi dây vào trong thùng | `milestone-06-cabinet` |
| **23/02 – 01/03/2027** | Hoàn thành **WP-E**: Hiệu chuẩn lại cân lần cuối với khay `340x340`, chạy Stress Test 100 lượt cân ($1\text{ kg}/\text{vé}$), quay video Demo và hoàn tất GitHub | `v1.0-release` |
| **02/03 – 05/03/2027** | Thời gian dự phòng (Contingency): Kiểm tra an toàn lần cuối, chuẩn bị xấp vé dự phòng và bàn giao thiết bị đúng hạn **05/03/2027** | `event-ready` |

---

## 6. Quy trình kiểm chứng và Lệnh kiểm thử (Verification Procedures)

1. **Kiểm thử trên phần mềm mô phỏng Proteus (`simulation_proteus/`):**
   - Thay đổi giá trị biến trở giả lập Loadcell ứng với các mức $0.5\text{ kg}$ (Đèn Vàng, $0\text{ vé}$), $1.0\text{ kg}$ ($1\text{ vé}$), $2.35\text{ kg}$ ($2\text{ vé}$).
   - Nhấn nút Arcade $\rightarrow$ Motor quay, xung từ bộ phát xung đi vào chân `EXTI` đủ số vé thì motor tự ngắt.
   - Ngắt bộ phát xung khi motor đang chạy $\rightarrow$ Sau $1.5\text{ s}$ motor tự dừng và Đèn Đỏ sáng nhấp nháy.
2. **Kiểm thử trên phần cứng thực tế (Hardware Benchtop & Full Cabinet Check):**
   - Đặt quả cân/vật chuẩn $1\text{ kg}$ ở tâm và 4 góc khay cân $340\times 340\text{ mm}$: Bảng LED P10 hiển thị trong khoảng `0.95kg` đến `1.05kg` và báo `01 VÉ`.
   - Kiểm tra rút giắc cảm biến vé hoặc rút hết vé giấy: Máy tự khóa motor sau $1.5\text{ s}$ và bật đèn tháp Đỏ báo cho tình nguyện viên.

---

## 7. Định nghĩa Hoàn thành Dự án (Definition of Done)

Dự án **EcoArcade** chỉ được công nhận là **Hoàn thành (Done)** vào ngày **05/03/2027** khi tất cả các điều kiện sau đều đúng:

- [ ] Kích thước hình học và bố cục mặt trước của máy tuân thủ thiết kế (`900x740x1450 mm`, khay cân `340x340`, nút $\phi 100$, miệng vé `70x12`, bảng `2 x LED P10 640x160`, đèn tháp `1600 mm`);
- [ ] Cân đo đạt độ chính xác $\le \pm 50\text{ g}$, quy đổi đúng $1\text{ kg} = 1\text{ vé}$ và nhả đúng số lượng vé vật lý;
- [ ] Có mô phỏng Proteus hoạt động tốt kèm mã nguồn STM32 sạch, rõ ràng, không bị treo máy;
- [ ] Đèn báo tình nguyện viên phản ánh chính xác trạng thái máy (Xanh / Vàng / Đỏ), cửa hông thay vé đóng mở thuận tiện;
- [ ] Toàn bộ tài liệu, sơ đồ mạch, bản vẽ cơ khí và video thực tế đã được đẩy lên GitHub để sẵn sàng phục vụ sự kiện và đính kèm vào CV ứng tuyển.