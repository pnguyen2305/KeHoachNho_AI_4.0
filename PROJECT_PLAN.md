# EcoArcade Work Plan — Trạm Cân Phế Liệu Đổi Vé Arcade Cho Trẻ Em

> **Trạng thái tài liệu (04/11/2026):** Đây là bản kế hoạch kỹ thuật gốc (Master Engineering Plan) cho dự án phần cứng - nhúng cá nhân **EcoArcade**. Mọi quá trình thiết kế mạch, bản vẽ 3D CAD, firmware và kết quả kiểm thử thực tế sẽ được cập nhật vào `CHECKPOINTS.md` và `DECISIONS.md` thông qua các Pull Request trên nhánh `main`.

## 1. Mục tiêu, phạm vi và hạn chót (Goal, scope, and deadline)

Dự án **EcoArcade** hướng tới việc chế tạo một thiết bị tương tác vật lý hoàn chỉnh phục vụ sự kiện cộng đồng dành cho trẻ em. Khi trẻ em mang phế liệu (vỏ lon, chai nhựa, giấy báo cũ) đặt lên bàn cân và nhấn nút, hệ thống sẽ đo khối lượng chính xác, hiển thị hiệu ứng đèn/âm thanh vui nhộn và **nhả ra một dải vé vật lý (Arcade Ticket Strip)** tương ứng với số cân nặng đạt được.

Hệ thống phải thỏa mãn 5 mục tiêu kỹ thuật cốt lõi:

1. Đo lường khối lượng phế liệu trong dải từ $0.05\text{ kg}$ đến $15.00\text{ kg}$ với sai số tuyệt đối $\le \pm 0.02\text{ kg}$ ($20\text{ g}$);
2. Quy đổi khối lượng sang số lượng vé theo công thức tuyến tính có ngưỡng tối thiểu và nhả chính xác số lượng vé vật lý bằng cơ cấu **Arcade Ticket Dispenser** (có phản hồi quang học đếm từng vé);
3. Đảm bảo an toàn điện - cơ khí tuyệt đối cho trẻ em (điện áp hoạt động nội bộ $\le 12\text{ V DC}$, không có cạnh sắc, chống kẹt tay);
4. Lưu trữ nhật ký giao dịch (số lần cân, tổng khối lượng tái chế, tổng số vé phát ra) vào bộ nhớ Flash/SD hoặc xuất ra file `.csv`/`.json` để thống kê sau sự kiện;
5. Có cơ chế tự chẩn đoán lỗi (kẹt vé, hết vé, quá tải bàn cân) và cảnh báo rõ ràng cho tình nguyện viên vận hành.

- **Mốc thời gian chính thức (Event Ready Deadline):** **23:59, 25/03/2027**.
- **Mốc hoàn thành nội bộ (Internal Feature Freeze):** **21:00, 10/03/2027**, dành ra 15 ngày dự phòng để chạy thử nghiệm chịu tải (Stress Test), sơn trang trí hoàn thiện và sửa lỗi phát sinh trước ngày diễn ra sự kiện.

### Phạm vi các yêu cầu kỹ thuật (Requirements)

- **Requirement 1.1** — Khảo sát, chốt sơ đồ khối phần cứng và danh mục vật tư (BOM);
- **Requirement 1.2** — Đọc và lọc nhiễu tín hiệu Loadcell qua ADC 24-bit `HX711`;
- **Requirement 1.3** — Thuật toán tự động trừ bì (Auto-Tare), chống gian lận (Anti-cheat) và quy đổi vé;
- **Requirement 1.4** — Điều khiển động cơ nhả vé Arcade (Ticket Dispenser) kết hợp cảm biến quang học đếm vạch vé;
- **Requirement 1.5** — Giao diện tương tác trẻ em: Màn hình hiển thị, dải LED hiệu ứng `WS2812B`, âm thanh `DFPlayer Mini` và nút bấm Arcade khổng lồ;
- **Requirement 1.6** — Thiết kế mạch nguyên lý (Schematic), bố trí bảng điện và hệ thống cấp nguồn kép ($12\text{ V DC}$ và $5\text{ V DC}$);
- **Requirement 1.7** — Thiết kế mô hình 3D CAD (khung bàn cân + thùng máy Arcade) và gia công cơ khí;
- **Requirement 1.8** — Xử lý ngoại lệ phần cứng (hết vé, kẹt vé, quá tải $m > m_{\max}$);
- **Requirement 1.9** — Ghi nhận dữ liệu thống kê sự kiện (`event_metrics.json`, `transactions.csv`);
- **Requirement 1.10** — Quy trình kiểm thử độc lập (Calibration & Stress Testing) với quả cân chuẩn.

### Bảng tiêu chí đánh giá dự án (Self-Grading Rubric — 100 điểm)

| Tiêu chí | Điểm | Điều kiện vượt qua (Gate) |
|---|---:|---|
| Độ chính xác của cân (Weighing Accuracy) | 25 | Sai số $\le \pm 20\text{ g}$ trên toàn dải đo sau 50 lần đặt tải liên tiếp |
| Độ tin cậy nhả vé (Ticket Dispensing) | 25 | Nhả đúng 100% số vé tính toán, tự ngắt đúng vạch cắt, phát hiện được khi hết vé |
| Trải nghiệm & Độ bền cơ khí (UX & Enclosure) | 20 | Khung vỏ chắc chắn, chịu được lực tỳ đè của trẻ em, hiệu ứng đèn/âm thanh mượt mà |
| Kiến trúc phần mềm & Xử lý lỗi (Firmware) | 15 | Code chia module rõ ràng, không dùng `delay()` gây treo máy (Non-blocking State Machine) |
| Hồ sơ kỹ thuật & GitHub (Documentation) | 15 | Đầy đủ sơ đồ mạch, file 3D CAD, hướng dẫn lắp ráp và lịch sử commit rõ ràng |

---

## 2. Cấu trúc kho lưu trữ và phân định thành phần (Planned artifacts)

```text
eco-arcade-scale/
├── PROJECT_PLAN.md                      # File kế hoạch tổng thể này
├── BUILD_FROM_SCRATCH_GUIDE.md          # Cẩm nang linh kiện, đi dây và học tập
├── README.md                            # Giới thiệu dự án, hình ảnh, cách nạp code
├── DECISIONS.md                         # Ghi chép các quyết định thiết kế (Architectural Decisions)
├── CHECKPOINTS.md                       # Nhật ký tiến độ hàng tuần và minh chứng thực nghiệm
├── BOM.csv                              # Danh mục linh kiện, thông số, link mua và chi phí
│
├── hardware/
│   ├── schematic/                       # Sơ đồ nguyên lý (EasyEDA / KiCad)
│   │   ├── eco_arcade_schematic.pdf
│   │   └── wiring_diagram.png
│   └── cad_3d/                          # File thiết kế cơ khí (Fusion 360 / Onshape)
│       ├── scale_platform.step          # Bàn cân cơ khí
│       ├── arcade_cabinet_laser.dxf     # Bản vẽ cắt CNC/Laser vỏ thùng gỗ MDF/Mica
│       └── mounts_3dprint/              # Các file .stl in 3D gá đỡ linh kiện
│
├── firmware/
│   ├── platformio.ini                   # Cấu hình môi trường PlatformIO (hoặc Arduino IDE)
│   ├── include/
│   │   ├── config.h                     # Định nghĩa chân Pin, hệ số Calibration, quy tắc đổi vé
│   │   └── state_machine.h              # Các trạng thái hoạt động của máy
│   ├── src/
│   │   ├── main.cpp                     # Vòng lặp chính và điều phối State Machine
│   │   ├── scale_sensor.cpp             # Đọc HX711, lọc trung vị (Median Filter), Tare
│   │   ├── ticket_dispenser.cpp         # Điều khiển motor nhả vé và đọc xung optocoupler
│   │   ├── ui_feedback.cpp              # Điều khiển màn hình, LED WS2812B và âm thanh
│   │   └── logger.cpp                   # Lưu log giao dịch vào bộ nhớ LittleFS / thẻ SD
│   └── test/
│       ├── test_ticket_math.cpp         # Unit test công thức quy đổi cân nặng -> số vé
│       ├── test_scale_filter.cpp        # Kiểm thử bộ lọc nhiễu tín hiệu Loadcell
│       └── test_fault_handler.cpp       # Kiểm thử kịch bản kẹt vé và quá tải
│
├── media/
│   ├── sd_card_audio/                   # Các file âm thanh MP3 (0001_welcome.mp3, ...)
│   └── sticker_artwork/                 # File thiết kế Decal dán thùng máy cho trẻ em
│
└── data_logs/
    ├── sample_transactions.csv          # Dữ liệu mẫu xuất ra từ thiết bị
    └── event_metrics.json               # Thống kê tổng hợp sau sự kiện
```

---

## 3. Các quy ước kỹ thuật cố định (Shared contracts to freeze)

Trước khi bắt tay vào hàn mạch và viết code, các thông số và quy tắc toán học sau được cố định làm chuẩn chung cho toàn bộ hệ thống:

### 3.1 Công thức quy đổi khối lượng sang số vé (Weight-to-Ticket Mapping)

Gọi $m$ (đơn vị: $\text{g}$) là khối lượng thực tế sau khi đã trừ bì và ổn định trên bàn cân:

- **Ngưỡng kích hoạt tối thiểu ($m_{\min}$):** $100\text{ g}$ ($0.1\text{ kg}$). Nếu $m < 100\text{ g}$, máy báo "Hãy bỏ thêm phế liệu nhé!" và không nhả vé.
- **Giới hạn an toàn tối đa ($m_{\max}$):** $15000\text{ g}$ ($15.0\text{ kg}$). Nếu $m > 15000\text{ g}$, máy phát cảnh báo quá tải để bảo vệ thanh Loadcell.
- **Định mức quy đổi cơ sở ($R$):** Mỗi $\Delta m = 200\text{ g}$ đổi được $1\text{ vé}$ (có thể cấu hình lại trong `config.h`).
- **Thưởng khuyến khích (Bonus Tier):** Nếu trẻ mang từ $2000\text{ g}$ ($2\text{ kg}$) trở lên trong một lượt, cộng thưởng thêm $B = 3\text{ vé}$.

Công thức tính tổng số vé $N_{\text{tickets}}$ cho một lượt cân hợp lệ ($m_{\min} \le m \le m_{\max}$):

$$N_{\text{tickets}}(m) = \left\lfloor \frac{m}{\Delta m} \right\rfloor + B \cdot \mathbb{I}(m \ge 2000)$$

Trong đó $\mathbb{I}(m \ge 2000)$ nhận giá trị bằng $1$ nếu $m \ge 2000\text{ g}$ và bằng $0$ nếu ngược lại.

### 3.2 Tiêu chuẩn ổn định cân (Weight Stability Contract)

Cân chỉ được coi là **đã ổn định (Stable)** để cho phép bấm nút nhận vé khi độ lệch chuẩn hoặc biên độ dao động giữa $K = 10$ mẫu đọc liên tiếp (trong khoảng thời gian $1.0\text{ s}$) nhỏ hơn ngưỡng $\epsilon = 10\text{ g}$:

$$\max_{1 \le k \le K} m_k - \min_{1 \le k \le K} m_k \le \epsilon$$

### 3.3 Chu trình máy trạng thái hữu hạn (Finite State Machine — FSM)

Hệ thống bắt buộc vận hành theo 6 trạng thái tuần tự, không cho phép nhận lệnh chồng chéo:

```text
[IDLE] -> (Phát hiện m >= m_min) -> [WEIGHING_STABILIZING]
       -> (Đạt độ ổn định)       -> [READY_FOR_BUTTON]
       -> (Trẻ nhấn nút Arcade)  -> [DISPENSING_TICKETS]
       -> (Nhả đủ N vé)          -> [WAIT_REMOVE_ITEMS]
       -> (Bàn cân về 0g + Tare) -> [IDLE]
```

*(Nếu xảy ra lỗi kẹt vé hoặc hết vé trong lúc `DISPENSING_TICKETS`, máy lập tức chuyển sang trạng thái `[ERROR_FAULT]` và khóa motor).*

### 3.4 Định dạng dữ liệu thống kê đầu ra (Telemetry Schema)

File `data_logs/transactions.csv` lưu trữ từng lượt tham gia với đúng các cột:

```text
transaction_id,timestamp_ms,weight_grams,tickets_dispensed,status
```

File `data_logs/event_metrics.json` tổng hợp kết quả cuối sự kiện:

```json
{
  "total_participants": 0,
  "total_weight_kg": 0.0,
  "total_tickets_issued": 0,
  "jam_or_empty_events": 0
}
```

---

## 4. Phân rã gói công việc chi tiết (Work Packages)

Vì đây là dự án cá nhân (Solo Engineering Project), bạn sẽ đóng vai trò là **Kỹ sư chính (Lead Developer)** thực hiện lần lượt từ Gói A đến Gói E, và **AI (Gemini)** sẽ đóng vai trò là **Technical Reviewer (Người kiểm duyệt thiết kế, code và kết quả đo)**.

---

### 4.1 Work Package A — Phân hệ Đo lường Khối lượng (Weighing Subsystem)

- **Mục đích:** Xây dựng bộ đọc cảm biến lực Loadcell chính xác, chống nhiễu cơ học và tự động trừ bì (Tare).
- **Nội dung triển khai:**
  - Kết nối thanh Loadcell ($20\text{ kg}$) với module khuếch đại `HX711` và vi điều khiển `ESP32`.
  - Xây dựng bộ lọc trung vị kết hợp trung bình trượt (Median + Moving Average Filter) để loại bỏ nhiễu gai khi trẻ em ném mạnh đồ vật lên bàn cân.
  - Viết tiện ích hiệu chuẩn 2 điểm (Two-point Calibration) sử dụng vật chuẩn (ví dụ: chai nước suối $500\text{ ml} \approx 500\text{ g}$ và tạ $2\text{ kg}$).
  - Lưu hệ số `calibration_factor` vào bộ nhớ Flash (`Preferences` / `EEPROM`) để không bị mất khi rút điện.
- **Kiểm thử (Tests):**
  - Đặt vật chuẩn $500\text{ g}$, $1000\text{ g}$, $5000\text{ g}$ lên 4 góc khác nhau của mặt bàn cân; sai số giữa các góc không vượt quá $\pm 20\text{ g}$.
  - Kiểm tra độ trôi điểm không (Zero-drift) sau 30 phút bật máy liên tục.
- **Planned commits:**
  ```text
  wp-a: setup platformio project and hx711 raw reader (req 1.1, 1.2)
  wp-a: implement median filter and stability detector (req 1.2, 1.3)
  wp-a: add persistent calibration factor in flash memory (req 1.2)
  ```
- **Tiêu chí hoàn thành:** Đọc đúng khối lượng thực tế lên Serial Monitor, tự động về $0\text{ g}$ khi lấy vật ra.

---

### 4.2 Work Package B — Phân hệ Nhả vé Arcade & Xử lý Lỗi (Ticket Dispenser Subsystem)

- **Mục đích:** Điều khiển cơ cấu nhả vé chính xác từng vé một và bảo vệ phần cứng khi kẹt/hết giấy.
- **Nội dung triển khai:**
  - Thiết kế mạch điều khiển động cơ $12\text{ V DC}$ của máy nhả vé thông qua module MOSFET (`IRFZ44N` / `AOD4184`) hoặc Relay/Motor Driver, cách ly hoàn toàn với chân $3.3\text{ V}$ của `ESP32`.
  - Đọc tín hiệu ngắt ngoài (External Interrupt `attachInterrupt`) từ cảm biến quang học (Notch Sensor) tích hợp sẵn trên máy nhả vé mỗi khi 1 lỗ khuyết của vé đi qua.
  - Triển khai bộ đếm thời gian chờ (Watchdog Timeout): Nếu motor đã chạy quá $T_{\text{timeout}} = 1500\text{ ms}$ mà không nhận được xung vé mới $\rightarrow$ lập tức ngắt motor và bật cờ lỗi `ERROR_OUT_OF_TICKETS` hoặc `ERROR_JAM`.
- **Kiểm thử (Tests):**
  - Lệnh nhả lần lượt $1\text{ vé}$, $5\text{ vé}$, $20\text{ vé}$ (kiểm tra 10 lần liên tiếp không lệch quá $0\text{ vé}$).
  - Rút dải vé ra khỏi máy khi đang chạy để giả lập tình huống hết vé $\rightarrow$ xác nhận motor tự dừng sau đúng $1.5\text{ s}$.
- **Planned commits:**
  ```text
  wp-b: implement ticket dispenser motor control and notch interrupt (req 1.4)
  wp-b: add jam and out-of-ticket timeout protection (req 1.8)
  ```
- **Tiêu chí hoàn thành:** Nhả đúng số vé yêu cầu, dừng đúng vị trí rãnh xé vé, xử lý an toàn khi hết vé.

---

### 4.3 Work Package C — Giao diện Trẻ em: Đèn, Âm thanh & Máy trạng thái (UX & FSM)

- **Mục đích:** Tạo sự hào hứng như máy game thùng thực thụ cho các bé và ghép nối toàn bộ luồng hoạt động.
- **Nội dung triển khai:**
  - Lập trình hiển thị số cân nặng và số vé dự kiến lên màn hình LCD/TFT hoặc LED 7 đoạn cỡ lớn.
  - Lập trình dải LED `WS2812B` chạy hiệu ứng dâng cột sáng (Power Bar) tỷ lệ thuận với khối lượng đặt lên cân.
  - Tích hợp module `DFPlayer Mini` phát hiệu ứng âm thanh: tiếng chào mừng khi đặt đồ lên, tiếng đếm số, tiếng nhạc chiến thắng khi nhấn nút Arcade và tiếng chuông báo khi hoàn tất.
  - Ghép nối hoàn chỉnh 6 trạng thái của FSM (Mục 3.3), bao gồm cơ chế **Anti-cheat** (bắt buộc phải lấy hết phế liệu cũ ra khỏi cân, đợi cân về $0\text{ g}$ mới cho phép lượt cân tiếp theo).
- **Kiểm thử (Tests):**
  - Thử nhấn nút Arcade liên tục (Button Mashing) khi máy đang nhả vé $\rightarrow$ hệ thống phải bỏ qua các lần bấm thừa, không nhả dư vé.
  - Thử giữ nguyên phế liệu cũ trên bàn cân và bấm nút lần 2 $\rightarrow$ hệ thống từ chối và nhắc lấy đồ ra.
- **Planned commits:**
  ```text
  wp-c: integrate lcd display, ws2812b progress bar and dfplayer audio (req 1.5)
  wp-c: implement full non-blocking finite state machine and anti-cheat (req 1.3, 1.5)
  wp-c: add local transaction logging to flash filesystem (req 1.9)
  ```
- **Tiêu chí hoàn thành:** Toàn bộ mạch chạy mượt trên bàn thí nghiệm (Benchtop Prototype), âm thanh - ánh sáng - nhả vé đồng bộ hoàn hảo.

---

### 4.4 Work Package D — Thiết kế 3D CAD & Gia công Khung vỏ Cơ khí (Mechanical & Enclosure)

- **Mục đích:** Đóng gói toàn bộ hệ thống thành một mô hình cứng cáp, bắt mắt và an toàn cho trẻ nhỏ.
- **Nội dung triển khai:**
  - Vẽ 3D CAD trên **Fusion 360** hoặc **Onshape**:
    - **Khung chịu lực bàn cân:** Gồm 2 tấm đế (tấm dưới cố định, tấm trên làm mặt bàn đặt phế liệu) gắn qua thanh Loadcell ở tâm, có 4 ốc giới hạn hành trình ở 4 góc (Overload Stop Bolts) để tránh gãy Loadcell nếu có bé bước chân lên bàn cân.
    - **Thùng điều khiển (Arcade Head):** Chứa màn hình, loa, nút bấm Arcade $60\text{ mm}$ hoặc $100\text{ mm}$, khe nhả vé và khay chứa cuộn vé dự trữ.
  - Xuất bản vẽ `.dxf` để đi cắt Laser/CNC gỗ MDF phủ Melamine (dày $6\text{ mm}$ – $9\text{ mm}$) hoặc Mica (Acrylic), kết hợp in 3D các gá đỡ mạch điện.
- **Kiểm thử (Tests):**
  - Kiểm tra tải trọng tĩnh $25\text{ kg}$ đè lên bàn cân: các ốc chống quá tải phải chạm nhau trước khi thanh Loadcell bị biến dạng dẻo.
  - Kiểm tra tháo lắp nhanh cửa hậu (Service Door) để người vận hành thay xấp vé mới trong vòng $< 60\text{ s}$.
- **Planned commits:**
  ```text
  wp-d: add 3d cad models for loadcell platform with overload stops (req 1.7)
  wp-d: add laser-cut dxf files for arcade cabinet enclosure (req 1.7)
  ```
- **Tiêu chí hoàn thành:** Khung vỏ lắp ghép vừa khít, không bị cạ cơ khí làm sai lệch kết quả cân.

---

### 4.5 Work Package E — Tích hợp Tổng thể, Kiểm chuẩn & Chạy thử nghiệm (Integration & Validation)

- **Mục đích:** Đi dây chuẩn công nghiệp, hoàn thiện thẩm mỹ và nghiệm thu toàn bộ dự án trước sự kiện.
- **Nội dung triển khai:**
  - Hàn mạch lên PCB đục lỗ (Perfboard) hoặc PCB đặt in, bấm đầu cosse dây điện, bọc gen co nhiệt và cố định dây bằng dây rút để chống tuột khi di chuyển đi sự kiện.
  - Dán Decal trang trí chủ đề Bảo vệ Môi trường / Arcade cho thùng máy.
  - Thực hiện kịch bản kiểm thử 100 lượt cân giả lập liên tiếp và xuất file `event_metrics.json`.
  - Quay video Demo hoạt động thực tế và hoàn thiện `README.md` trên GitHub.
- **Planned commits:**
  ```text
  wp-e: finalize wiring harness, power distribution and enclosure assembly (req 1.6, 1.7)
  wp-e: run 100-cycle endurance test and record validation metrics (req 1.9, 1.10)
  docs: complete github portfolio documentation and demo media
  ```

---

## 5. Sơ đồ phụ thuộc các giai đoạn (Dependency Graph)

```text
[Chốt BOM & Mua linh kiện]
            |
            +-------------------------------+
            v                               v
[WP-A: Đọc Loadcell & Lọc nhiễu]   [WP-B: Điều khiển Máy nhả vé]
            |                               |
            +---------------+---------------+
                            |
                            v
       [WP-C: Giao diện Đèn/Loa & Ghép nối State Machine]
                            |
                            v
       [WP-D: Vẽ 3D CAD, Cắt CNC/In 3D & Làm khung cơ khí]
                            |
                            v
       [WP-E: Lắp ráp tổng thể, Hiệu chuẩn, Trang trí & Nghiệm thu]
                            |
                            v
                 [Tag Release v1.0 -> Sự kiện]
```

---

## 6. Lịch trình chi tiết từ Tháng 11/2026 đến Tháng 03/2027 (Planned Calendar)

| Giai đoạn / Thời gian | Mục tiêu bắt buộc cần hoàn thành (Required Outcome) | Checkpoint Tag |
|---|---|---|
| **04/11 – 15/11/2026** | Khởi tạo GitHub Repo, chốt phương án phần cứng, đặt mua toàn bộ linh kiện cốt lõi (ESP32, Loadcell, HX711, Máy nhả vé Arcade, Nguồn 12V) | `milestone-01-init` |
| **16/11 – 30/11/2026** | Hoàn thành **WP-A**: Hàn header, đấu nối Loadcell + HX711 trên Breadboard, viết xong bộ lọc nhiễu và hiệu chuẩn cân chính xác | `milestone-02-scale` |
| **01/12 – 20/12/2026** | Hoàn thành **WP-B**: Cấp nguồn $12\text{ V}$, điều khiển thành công máy nhả vé Arcade bằng MOSFET, đếm chính xác từng vé qua cảm biến quang | `milestone-03-dispenser` |
| **21/12/2026 – 10/01/2027** | Hoàn thành **WP-C**: Kết nối màn hình, LED `WS2812B`, loa `DFPlayer Mini`, nút bấm Arcade; hoàn thiện State Machine từ đầu đến cuối trên bàn thử | `milestone-04-firmware` |
| **11/01 – 31/01/2027** | Thiết kế 3D CAD (**WP-D** phần 1): Học Fusion 360/Onshape, đo kích thước thật của linh kiện, vẽ xong bàn cân và vỏ thùng Arcade | `milestone-05-cad` |
| **01/02 – 21/02/2027** | Gia công cơ khí (**WP-D** phần 2): Đặt cắt Laser gỗ/Mica, in 3D các gá đỡ, lắp thử phần khung thô và kiểm tra cơ cấu chống quá tải bàn cân | `milestone-06-mech` |
| **22/02 – 10/03/2027** | Hoàn thành **WP-E**: Hàn mạch cố định, đi dây vào thùng máy, dán Decal trang trí, hiệu chuẩn lại cân sau khi gắn mặt bàn chính thức | `milestone-07-assembly` |
| **11/03 – 20/03/2027** | Chạy Stress Test 100 lượt cân, tinh chỉnh âm lượng/ánh sáng, chuẩn bị cuộn vé dự phòng và viết hướng dẫn vận hành 1 trang cho sự kiện | `v1.0-release` |
| **21/03 – 25/03/2027** | Thời gian dự phòng (Contingency): Khắc phục các lỗi nhỏ phát sinh, đóng gói vận chuyển tới địa điểm tổ chức sự kiện | `event-ready` |

---

## 7. Quy trình kiểm chứng và Lệnh kiểm thử (Verification Procedures)

Trong quá trình phát triển, mọi nhánh code và bản lắp ráp phải vượt qua các bước kiểm tra sau trước khi cập nhật vào `CHECKPOINTS.md`:

1. **Kiểm thử công thức toán học và State Machine trên máy tính (Unit Tests):**
   ```bash
   pio test -e native
   ```
2. **Chế độ chẩn đoán phần cứng độc lập (Hardware Diagnostic CLI qua Serial 115200 baud):**
   - Gửi lệnh `TARE` $\rightarrow$ Cân phải trả về `0.00 kg` trong vòng $500\text{ ms}$.
   - Gửi lệnh `CAL 1000` (khi đặt quả cân $1\text{ kg}$) $\rightarrow$ Tự động tính và lưu `calibration_factor` mới.
   - Gửi lệnh `DISPENSE 5` $\rightarrow$ Máy nhả đúng $5\text{ vé}$ và in ra thời gian thực thi trung bình trên mỗi vé (kỳ vọng $\approx 180\text{ ms} - 250\text{ ms}/\text{vé}$).
   - Gửi lệnh `EXPORT` $\rightarrow$ Xuất toàn bộ dữ liệu `transactions.csv` và `event_metrics.json` qua cổng Serial.

---

## 8. Kỷ luật Git và Quy chuẩn Commit (Git Discipline)

Để hồ sơ GitHub đạt chuẩn chuyên nghiệp như một dự án kỹ thuật thực tế:

- Kiểm tra định danh Git trước khi làm việc:
  ```bash
  git config --get user.name
  git config --get user.email
  ```
- Mỗi tính năng hoặc bản vẽ được thực hiện trên một nhánh riêng (ví dụ: `feat/wp-a-loadcell`, `feat/wp-b-dispenser`, `cad/enclosure-v1`) và gộp vào `main` qua Pull Request.
- Định dạng thông điệp Commit thống nhất:
  ```text
  <phân-hệ>: <hành động cụ thể> (req 1.x)
  ```
  *Ví dụ:* `wp-a: add 5-sample median filter for hx711 readings (req 1.2)`
- Không đẩy các file rác biên dịch (`.pio/`, `build/`, file backup của phần mềm CAD) lên kho lưu trữ; quản lý chặt chẽ qua file `.gitignore`.

---

## 9. Định nghĩa Hoàn thành Dự án (Definition of Done)

Dự án **EcoArcade** chỉ được công nhận là **Hoàn thành (Done)** khi tất cả các điều kiện sau đều đúng:

- [ ] Cân đo chính xác từ $100\text{ g}$ đến $15\text{ kg}$ với sai số $\le \pm 20\text{ g}$, có ốc bảo vệ chống sập bàn cân khi trẻ em dẫm lên;
- [ ] Máy nhả vé Arcade xuất đúng 100% số vé theo công thức quy đổi, vết cắt nằm đúng rãnh xé vé;
- [ ] Cơ chế chống gian lận (Anti-cheat) và chống kẹt/hết vé (Watchdog Timeout) hoạt động chính xác trong thực tế;
- [ ] Toàn bộ dây điện được bọc cách điện, đi gọn trong thùng máy, sử dụng một phích cắm nguồn duy nhất bên ngoài;
- [ ] Thùng máy được trang trí hoàn thiện, thân thiện và thu hút trẻ em, có cửa thay vé nhanh trong 1 phút;
- [ ] Kho lưu trữ GitHub có đầy đủ code, sơ đồ nguyên lý, file 3D CAD, bảng BOM, hình ảnh/video thực tế và tất cả các mốc `milestone-01` đến `v1.0-release` đều được gắn tag đầy đủ.