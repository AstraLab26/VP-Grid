# VP-Grid (MT5) — README

EA: `VP-Grid.mq5`  
Phiên bản trong file: `2.12`

## Tổng quan
`VP-Grid` là EA lưới “pending ảo” cho MT5:

- Không gửi lệnh pending lên broker. EA tự lưu các mức giá lưới và **khi giá chạm mức** thì vào **lệnh Market** tương ứng.
- Hỗ trợ 4 “nhóm chiến lược” theo Magic:
  - **AA / BB / CC**: **Buy ảo phía trên** đường gốc (BUY STOP), **Sell ảo phía dưới** đường gốc (SELL STOP)
  - **DD**: **Sell ảo phía trên** đường gốc (SELL LIMIT), **Buy ảo phía dưới** đường gốc (BUY LIMIT)
- Có các cơ chế quản trị:
  - **Trailing theo tổng lợi nhuận lệnh đang mở** (gongLai mode)
  - **Balance** (đóng bớt lệnh lỗ phía đối diện) cho AA/BB/CC
  - **Lock profit (Save %)**: giữ lại % lợi nhuận TP (không dùng để balance)
  - **Scale theo tăng trưởng vốn**: nhân lot/TP/trailing theo % tăng trưởng so với vốn gốc
  - **Reset session** khi đạt điều kiện (trailing lock / trailing SL hit…)
  - **Daily stop**: cộng dồn theo RESET qua nhiều ngày; đạt ngưỡng thì dừng đến hết ngày (restart từ ngày sau)
  - **Trading hours**: chỉ cho EA chạy trong khung giờ; ngoài giờ thì chờ đến giờ bắt đầu
  - Thông báo **Notification** và **Telegram** (tùy chọn)

## Khái niệm chính

### 1) Đường gốc (Base line)
- `basePrice` được set khi EA khởi động và mỗi lần EA reset.
- Lưới được tạo **đối xứng** quanh đường gốc:
  - Phía trên: level +1, +2, …, +N
  - Phía dưới: level -1, -2, …, -N
- EA **không đặt lệnh tại đúng basePrice**.

### 2) “Pending ảo” là gì?
EA lưu danh sách `g_virtualPending[]` gồm (magic, loại lệnh, mức giá, level, TP, lot).  
Mỗi tick, EA kiểm tra nếu giá chạm mức thì vào Market bằng `trade.Buy()` / `trade.Sell()`.

### 3) Re-arm delay (sau khi TP)
Bạn có thể cấu hình thời gian **delay đặt lại** lệnh chờ ảo tại đúng **bậc vừa TP**.

Ví dụ: **AA level +1** chốt TP xong và `RearmDelayMinutesAA = 10` thì EA sẽ **đợi 10 phút** rồi mới đặt lại virtual pending AA tại level +1 (không ảnh hưởng các level khác).

## Cài đặt & chạy

### Bước 1: Copy file
- Chép `VP-Grid.mq5` vào:
  - `MQL5/Experts/` (hoặc một thư mục con tùy bạn)

### Bước 2: Compile
- Mở MetaEditor → mở file → Compile.

### Bước 3: Gắn lên chart
- Mở MT5 → Navigator → Experts → kéo EA vào chart mong muốn.
- Bật:
  - **Algo Trading**
  - (Nếu dùng Telegram) Tools → Options → Expert Advisors → Allow WebRequest và thêm `https://api.telegram.org`

## Các tham số (Inputs)

### 1) GRID
- **GridDistancePips**: khoảng cách giữa 2 mức lưới liên tiếp (đơn vị “pips” theo cách EA quy đổi).
- **MaxGridLevels**: số bậc tối đa mỗi phía (trên + dưới). Tổng mức = `2 * MaxGridLevels`.

### 2) ORDERS (pending ảo)
#### 2.1 Common
- **MagicNumber**: magic gốc. EA dùng:
  - AA = `MagicNumber`
  - BB = `MagicNumber + 1`
  - CC = `MagicNumber + 2`
  - DD = `MagicNumber + 3`
- **CommentOrder**: comment gắn cho lệnh Market khi kích hoạt pending ảo.
  - Mặc định hiện tại: `VPGrid`

#### 2.2 AA (ảo): BUY trên gốc + SELL dưới gốc
- **EnableAA**: bật/tắt AA
- **LotSizeAA**: lot level 1
- **AALotScale**:
  - `LOT_FIXED`: các level dùng cùng lot
  - `LOT_GEOMETRIC`: level 2+ nhân theo cấp số nhân
- **LotMultAA**: hệ số nhân cho level 2+ (khi Geometric)
- **MaxLotAA**: chặn lot tối đa (0 = không chặn)
- **TakeProfitPipsAA**: TP theo pips (0 = không TP)
- **EnableBalanceAAByBB**: bật balance “AA by BB”

#### 2.3 BB (ảo)
- **EnableBB**, **LotSizeBB**, **BBLotScale**, **LotMultBB**, **MaxLotBB**
- **TakeProfitPipsBB**
- **EnableBalanceBB**

#### 2.4 CC (ảo)
- **EnableCC**, **LotSizeCC**, **CCLotScale**, **LotMultCC**, **MaxLotCC**
- **TakeProfitPipsCC**
- **EnableBalanceCC**

#### 2.5 DD (ảo): SELL trên gốc + BUY dưới gốc
- **EnableDD**, **LotSizeDD**, **DDLotScale**, **LotMultDD**, **MaxLotDD**
- **TakeProfitPipsDD**
- DD **không tham gia balance**.

### 3) SESSION: Trailing profit (open orders only)
- **EnableTrailingTotalProfit**: bật trailing theo tổng lợi nhuận các lệnh đang mở (session hiện tại).
- **TrailingThresholdUSD**: bắt đầu trailing khi floating P/L ≥ ngưỡng (USD).
- **TrailingDropMode**:
  - `TRAILING_MODE_LOCK`: nếu profit tụt xuống ngưỡng drop (khi SL chưa đặt) → đóng hết & reset
  - `TRAILING_MODE_RETURN`: nếu profit tụt xuống ngưỡng drop (khi SL chưa đặt) → thoát trailing, đặt lại pending
- **TrailingDropPct**: % tụt so với threshold để kích hoạt hành vi trên.
- **TrailingPointAPips**: khoảng cách Point A (pips) tính từ “grid level base” tại thời điểm vào trailing.
- **GongLaiStepPips**: bước trailing (pips): giá đi thêm 1 bước → SL dời 1 bước.

### 4) CAPITAL % SCALING
- **EnableScaleByAccountGrowth**: bật scale theo tăng trưởng vốn.
- **BaseCapitalUSD**:
  - `0`: lấy balance lúc attach EA làm vốn gốc
  - `>0`: dùng giá trị này làm vốn gốc
- **AccountGrowthScalePct**: tỉ lệ scale theo tăng trưởng (%). Ví dụ vốn +100%, scale 50% → nhân 1.5.
- **MaxScaleIncreasePct**: chặn mức tăng tối đa (0 = không chặn).

### 5) NOTIFICATIONS
- **EnableResetNotification**: gửi Notification khi EA reset/stop.

#### 5.1 Telegram
- **EnableTelegram**
- **TelegramBotToken**
- **TelegramChatID**

### 6) LOCK PROFIT (Save %)
- **EnableLockProfit**
- **LockProfitPct**: % lợi nhuận của mỗi lệnh đóng TP được “khóa” (cộng dồn) và **không dùng cho balance pool**.

### 7) DAILY STOP
- **EnableDailyStop**
- **DailyProfitTargetUSD**: ngưỡng USD (cộng dồn theo RESET).

**Cách tính Daily stop (cộng dồn qua nhiều ngày)**:
- Mỗi lần EA **RESET**, EA cộng dồn \( \Delta = Balance\_{now} - sessionStartBalance \) vào bộ đếm.
- Nếu **chưa đạt ngưỡng**, **sang ngày mới vẫn tiếp tục cộng dồn** (không reset).
- Chỉ khi bộ đếm **>= DailyProfitTargetUSD** (dù là do cộng dồn từ các ngày trước) → EA **dừng**.
- Khi EA đã dừng vì đạt ngưỡng:
  - Dù có vào **giờ chạy** trong **cùng ngày** → EA vẫn **dừng**.
  - Chỉ **sang ngày kế tiếp** và vào **đúng giờ chạy** (Trading hours) → EA mới được **khởi động lại** và bộ đếm được reset về **0** để bắt đầu chu kỳ mới.

Mặc định hiện tại: `EnableDailyStop = false`, `DailyProfitTargetUSD = 500`.

### 8) TRADING HOURS
- **EnableTradingHours**
- **TradingStartHour/TradingStartMinute** (mặc định 08:00)
- **TradingEndHour/TradingEndMinute** (mặc định 16:00)

**Hành vi theo giờ chạy**:
- **Ngoài giờ chạy**:
  - Nếu EA **đang dừng** → tiếp tục dừng cho tới khi vào giờ chạy, rồi EA **khởi động** và đặt **basePrice** theo giá lúc đó.
  - Nếu EA **đang chạy dở** mà vượt qua giờ kết thúc → EA **vẫn chạy tiếp**; khi xảy ra **RESET** tiếp theo thì EA mới **dừng**, sau đó chờ phiên sau.

Mặc định hiện tại: `EnableTradingHours = false`.

### 9) START FILTER (ADX)
- **EnableADXStartFilter**: chỉ cho EA “khởi động” khi ADX < ngưỡng.
- **ADXTimeframe**, **ADXPeriod**, **ADXStartThreshold**

**Nguyên tắc**:
- EA **đang chạy** thì vẫn chạy bình thường (không bị tắt).
- EA **đang dừng/chờ** (đợi giờ chạy hoặc vừa reset) thì chỉ khởi động lại khi **ADX < ADXStartThreshold**.

Mặc định hiện tại: `EnableADXStartFilter = false`, `ADXTimeframe = M15`, `ADXPeriod = 14`, `ADXStartThreshold = 25`.

### 10) RE-ARM DELAY (after TP)
- **RearmDelayMinutesAA/BB/CC/DD**

**Nguyên tắc**:
- Khi 1 lệnh **đóng bằng TP** tại bậc nào thì EA sẽ **delay đặt lại** virtual pending đúng **(loại + bậc)** đó trong X phút.
- Các bậc khác vẫn đặt lại bình thường.

Mặc định hiện tại: `RearmDelayMinutesAA/BB/CC/DD = 10`.

### 11) SESSION RESET (profit target)
- **EnableSessionProfitReset**
- **SessionProfitTargetUSD**

**Nguyên tắc**:
- Khi **không** ở `gongLaiMode`, nếu \( (Balance - sessionStartBalance) + floating \) đạt ngưỡng thì EA **Reset** (đóng hết, đặt base mới, đặt lại lưới).
- Nếu `gongLaiMode` đang kích hoạt thì **không áp dụng** reset theo ngưỡng phiên.

Mặc định hiện tại: `EnableSessionProfitReset = false`, `SessionProfitTargetUSD = 500`.

## Ví dụ cấu hình

> Lưu ý: đây là ví dụ để bạn tham khảo logic. Bạn vẫn cần tự tối ưu theo sản phẩm, spread, margin, và mức chịu DD.

### Ví dụ 1 — Lưới lớn, TP nhỏ, chỉ dùng BB/CC
Mục tiêu: chạy 2 nhóm BB/CC để lấy TP đều, tắt AA/DD.

- `GridDistancePips = 1000`
- `MaxGridLevels = 100`
- `EnableAA = false`
- `EnableBB = true`
  - `LotSizeBB = 0.01`
  - `BBLotScale = LOT_GEOMETRIC`
  - `LotMultBB = 1.3`
  - `TakeProfitPipsBB = 1000`
  - `EnableBalanceBB = true`
- `EnableCC = true`
  - `LotSizeCC = 0.01`
  - `CCLotScale = LOT_FIXED`
  - `TakeProfitPipsCC = 1000`
  - `EnableBalanceCC = true`
- `EnableDD = false`
- `EnableLockProfit = true`, `LockProfitPct = 25`
- `EnableScaleByAccountGrowth = true`, `BaseCapitalUSD = 50000`, `AccountGrowthScalePct = 50`

### Ví dụ 2 — Chạy đủ AA/BB/CC/DD, DD lot nhỏ để “đệm”
Mục tiêu: DD lot nhỏ, TP đều; AA/BB/CC dùng balance để giảm lệnh lỗ đối diện.

- `EnableAA = true`, `TakeProfitPipsAA = 0`, `EnableBalanceAAByBB = true`
- `EnableBB = true`, `TakeProfitPipsBB = 1000`, `EnableBalanceBB = true`
- `EnableCC = true`, `TakeProfitPipsCC = 1000`, `EnableBalanceCC = true`
- `EnableDD = true`, `LotSizeDD = 0.01`, `MaxLotDD = 0.01`, `TakeProfitPipsDD = 1000`

### Ví dụ 3 — Dùng trailing tổng lợi nhuận để “khóa” session
Mục tiêu: khi tổng floating đạt ngưỡng thì hủy pending, bỏ TP, bắt đầu trailing SL.

- `EnableTrailingTotalProfit = true`
- `TrailingThresholdUSD = 50`
- `TrailingDropMode = TRAILING_MODE_RETURN` (an toàn hơn để thoát trailing nếu chưa kịp đặt SL)
- `TrailingDropPct = 20`
- `TrailingPointAPips = 1000`
- `GongLaiStepPips = 500`

## Lưu ý quan trọng về “pip” và khoảng cách lưới

EA hiện quy đổi bước lưới bằng:
- `gridStep = GridDistancePips * _Point * 10`

Điều này **chỉ đúng** kiểu giá phổ biến:
- **5 digits** (EURUSD dạng 1.23456) hoặc **3 digits** (USDJPY dạng 123.456)

Nếu bạn chạy symbol có **4 digits / 2 digits** thì công thức trên có thể làm **lưới bị giãn 10 lần** so với “pips” bạn nhập.

Khuyến nghị khi sử dụng:
- Nếu symbol của bạn không phải 5/3 digits, hãy kiểm tra lại khoảng cách thực tế trên chart.
- Nếu cần chuẩn hóa cho mọi symbol, nên đổi cách tính “pipSize” giống phần trailing (dựa trên `Digits`) rồi dùng `gridStep = GridDistancePips * pipSize`.

## Cách EA đặt lệnh theo lưới (tóm tắt)
- Trên base:
  - AA/BB/CC: đặt BuyStop ảo ở các level cao hơn giá hiện tại
  - DD: đặt SellLimit ảo ở các level cao hơn giá hiện tại
- Dưới base:
  - AA/BB/CC: đặt SellStop ảo ở các level thấp hơn giá hiện tại
  - DD: đặt BuyLimit ảo ở các level thấp hơn giá hiện tại

## Câu hỏi thường gặp

### 1) “Khoảng cách lưới có đều không?”
- **Đều theo công thức** (mỗi level = base ± n * gridStep).
- Nhưng “đều theo pips input” còn phụ thuộc cách EA quy đổi pip ↔ point (xem mục Lưu ý pip).

### 2) Vì sao không thấy pending trên broker?
Vì EA dùng pending ảo (`g_virtualPending[]`). Khi chạm mức, EA vào lệnh Market.

### 3) Balance dùng tiền nào?
- Pool balance = TP đã chốt trong session (sau khi trừ lock profit) và dùng để đóng/đóng một phần các lệnh lỗ đối diện cho AA/BB/CC.

### 4) Thông báo có hiển thị tiến độ daily stop và giờ chạy không?
Có. Trong Notification/Telegram sẽ có:
- `Daily reset-profit progress: X / Target USD`
- `Trading hours: 08:00-16:00`
- Trạng thái: `WAITING`, hoặc `stop pending (will stop on next reset)` khi vừa hết giờ chạy.

## Ghi chú hiệu năng (EA nhẹ hơn)
EA đã được tối ưu để “bảo trì lưới” (quét level/duplicate/ensure) chạy tối đa **1 lần/giây** thay vì mỗi tick, giúp nhẹ hơn khi tick dày hoặc chạy nhiều chart.

---

Nếu bạn muốn, mình có thể cập nhật code để `GridDistancePips` hoạt động đúng trên mọi symbol (5/3/4/2 digits) theo cùng cách tính `pipSize` như phần trailing.

