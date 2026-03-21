# VP-Grid.mq5 — README (MT5)

- **File:** `VP-Grid.mq5`  
- **Phiên bản:** `2.12` (theo `#property version` trong code)

Tài liệu này **chỉ** mô tả EA **VP-Grid** trong file trên: tổng quan, input **nhóm 1–13**, giá trị mặc định trong source, và ví dụ cấu hình.

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
- Chép **`VP-Grid.mq5`** vào:
  - `MQL5/Experts/` (hoặc một thư mục con tùy bạn)

### Bước 2: Compile
- Mở MetaEditor → mở file → Compile.

### Bước 3: Gắn lên chart
- Mở MT5 → Navigator → Experts → kéo EA vào chart mong muốn.
- Bật:
  - **Algo Trading**
  - (Nếu dùng Telegram) Tools → Options → Expert Advisors → Allow WebRequest và thêm `https://api.telegram.org`

## Các tham số (Inputs)

### 0) Bảng mặc định — `VP-Grid.mq5` (tóm tắt)
Khi load EA lần đầu, các giá trị mặc định trong source (đồng bộ giao diện Inputs) như sau.

**Nhóm 1–2 — GRID + ORDERS:**

| Nhóm | Giá trị mặc định (tóm tắt) |
|------|----------------------------|
| **1. GRID** | `GridDistancePips = 1500`, `MaxGridLevels = 100` |
| **2.1 Common** | `MagicNumber = 123456`, `CommentOrder = "VPGrid"` |
| **2.2 AA** | Bật; lot 0.01; `LOT_GEOMETRIC`, mult 1.3; max 2; TP 0; balance bật |
| **2.3 BB** | Bật; lot 0.01; geometric 1.3; max 2; TP 1500; balance bật |
| **2.4 CC** | Tắt; lot 0.01; `LOT_FIXED`, mult 1.1; max 2; TP 1500; balance bật |
| **2.5 DD** | Tắt; lot 0.01; `LOT_FIXED`, mult 1.3; max 0.01; TP 1000 (không balance) |

**Nhóm 3–8.1** (trailing, scale, thông báo, lock profit, daily stop, giờ + ngày trong tuần):

| Nhóm | Giá trị mặc định (tóm tắt) |
|------|----------------------------|
| **3. Trailing** | Bật; ngưỡng 50 USD; `TRAILING_MODE_RETURN`; drop 20%; Point A 1000 pip; bước 500 pip |
| **4. Capital %** | Bật scale; base 50000 USD; nhân 50%; tăng tối đa 100% |
| **5. Notifications** | Bật khi reset/stop; Telegram tắt; token/chat rỗng |
| **6. Lock profit** | Bật; 25% |
| **7. Daily stop** | Tắt; mục tiêu 500 USD/ngày |
| **8. Trading hours** | Tắt; 08:00–16:00 server |
| **8.1 Weekdays** | **Bật** lọc ngày; T2–T6 bật; T7/CN tắt |

**Nhóm 9–13** (bộ lọc start/balance, re-arm, reset session, reset theo level, delay restart):

| Nhóm | Giá trị mặc định (tóm tắt) |
|------|----------------------------|
| **9. ADX** | Tắt; M15; period 14; bắt đầu khi ADX dưới 20 |
| **9.1 RSI cross** | Tắt; M15; period 14; cross 70 / 30 |
| **9.2 Balance RSI** | Bật; M5; lookback **20** bar; trên 70 / dưới 30 |
| **9.3 Across base** | Bật; `X=50 pip`; dư tối thiểu `50 pip` mỗi cặp đóng; tối thiểu 3 bậc lưới khỏi base; mode noTP->TP mặc định tắt |
| **10. Re-arm** | 20 phút (AA/BB/CC/DD) |
| **11. Session reset** | Bật; mục tiêu 500 USD |
| **12. Level match** | Bật; khoảng cách tối thiểu **4000** pip; session P/L ≥ 20 USD |
| **13. Restart delay** | 0 phút |

### 1) GRID
- **GridDistancePips**: khoảng cách giữa 2 mức lưới liên tiếp (đơn vị “pips” theo cách EA quy đổi).
- **MaxGridLevels**: số bậc tối đa **mỗi phía** (trên + dưới). Tổng mức = `2 * MaxGridLevels`.

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

### 8.1) WEEKDAYS (run days)
- **EnableWeekdaySchedule**, **RunMonday..RunSunday**

**Nguyên tắc**:
- Nếu hôm nay là **ngày không chạy**:
  - EA **đang chạy dở** vẫn tiếp tục chạy bình thường.
  - Khi xảy ra **RESET** tiếp theo (trailing lock / trailing SL hit / session reset / levels match…) thì EA sẽ **dừng** và chuyển sang WAITING.
  - EA sẽ **đợi đến ngày được chạy** rồi khởi động lại (đặt base tại giá lúc khởi động lại).

Mặc định hiện tại: `EnableWeekdaySchedule = true`, `RunMonday..RunFriday = true`, `RunSaturday = false`, `RunSunday = false`.

### 9) START FILTER (ADX)
- **EnableADXStartFilter**: chỉ cho EA “khởi động” khi ADX < ngưỡng.
- **ADXTimeframe**, **ADXPeriod**, **ADXStartThreshold**

**Nguyên tắc**:
- EA **đang chạy** thì vẫn chạy bình thường (không bị tắt).
- EA **đang dừng/chờ** (đợi giờ chạy hoặc vừa reset) thì chỉ khởi động lại khi **ADX < ADXStartThreshold**.

Mặc định hiện tại: `EnableADXStartFilter = false`, `ADXTimeframe = M15`, `ADXPeriod = 14`, `ADXStartThreshold = 20`.

### 9.1) START FILTER (RSI cross)
- **EnableRSIStartFilter**: chỉ cho EA “khởi động” khi RSI có tín hiệu cắt ngưỡng.
- **RSITimeframe**, **RSIPeriod**, **RSIUpperCross**, **RSILowerCross**

**Nguyên tắc**:
- EA **đang chạy** thì vẫn chạy bình thường (không bị tắt).
- EA **đang dừng/chờ** (đợi giờ chạy hoặc vừa reset) thì chỉ khởi động lại khi:
  - RSI **cắt lên** `RSIUpperCross` (mặc định 70), **hoặc**
  - RSI **cắt xuống** `RSILowerCross` (mặc định 30).
- EA dùng **2 nến đã đóng gần nhất** (shift 2 → shift 1) để xác định “cắt” cho ổn định.

Mặc định hiện tại: `EnableRSIStartFilter = false`, `RSITimeframe = M15`, `RSIPeriod = 14`, `RSIUpperCross = 70`, `RSILowerCross = 30`.

### 9.2) BALANCE FILTER (RSI)
- **EnableRSIBalanceFilter**, **RSIBalanceTimeframe**, **RSIBalanceLookbackBars**, **RSIBalanceUpper**, **RSIBalanceLower**

**Nguyên tắc** (chỉ áp dụng cho đóng lệnh âm ngược phía trong Balance AA/BB/CC):
- Nếu giá hiện tại **trên** đường gốc: chỉ cho phép đóng balance khi RSI nến đóng gần nhất **> RSIBalanceUpper** và trong `RSIBalanceLookbackBars` nến gần nhất có RSI vượt ngưỡng này.
- Nếu giá hiện tại **dưới** đường gốc: chỉ cho phép đóng balance khi RSI nến đóng gần nhất **< RSIBalanceLower** và trong `RSIBalanceLookbackBars` nến gần nhất có RSI dưới ngưỡng này.

Mặc định hiện tại: `EnableRSIBalanceFilter = true`, `RSIBalanceTimeframe = M5`, `RSIBalanceLookbackBars = 20`, `RSIBalanceUpper = 70`, `RSIBalanceLower = 30`.

### 9.3) BALANCE ACROSS BASE (open, no TP)
- **EnableBalanceOpenAcrossBaseNoTP**
- **BalanceOpenAcrossBaseNoTP_XPips** (X pip)
- **BalanceClosePairMinSurplusPips** (pip dư tối thiểu cho mỗi cặp đóng dương+âm)
- **BalanceOpenAcrossBaseNoTP_MinDistanceLevels** (N): giá hiện tại phải cách `base` ít nhất N bậc lưới (default N=3)
- **EnableBalanceNoTPCloseTP**: bật mode dùng lệnh dương **không TP** để đóng lệnh âm **có TP** ở phía đối diện

Mặc định hiện tại:
- `EnableBalanceOpenAcrossBaseNoTP = true`
- `BalanceOpenAcrossBaseNoTP_XPips = 50`
- `BalanceClosePairMinSurplusPips = 50`
- `BalanceOpenAcrossBaseNoTP_MinDistanceLevels = 3`
- `EnableBalanceNoTPCloseTP = false`

**Nguyên tắc chung** (áp dụng cho AA/BB/CC, không áp dụng DD):
- Xét giá hiện tại so với `base` để xác định **cùng phía** và **bên kia phía** qua đường gốc.
- EA chỉ chạy chế độ này khi khoảng cách hiện tại đến `base` >= `N * gridStep` (tức ít nhất N bậc lưới).
- Ngưỡng kích hoạt theo pip: tổng điều kiện phải đạt `X` pip.
- Mỗi cặp đóng phải đảm bảo còn dư tối thiểu `BalanceClosePairMinSurplusPips` pip.
- Lệnh dương được đóng **toàn bộ**; lệnh âm được phép đóng **toàn phần hoặc một phần**.
- Nếu khối lượng partial tính ra quá nhỏ, EA thử lần lượt `0.02 lot`, rồi `0.01 lot`; nếu vẫn không đủ thì chờ tick sau.

**Mode A — Không TP đóng Không TP** (`EnableBalanceOpenAcrossBaseNoTP = true`)
- Chỉ xét lệnh **không TP** (`TP <= 0`).
- Khi giá hiện tại ở trên base:
  - chọn lệnh dương không TP phía trên base.
  - đóng lệnh âm không TP phía dưới base.
- Khi giá hiện tại ở dưới base thì ngược lại.
- Điều kiện kích hoạt: `sumPosPips + sumNegAbsPips >= X`.

**Mode B — Không TP đóng TP** (`EnableBalanceNoTPCloseTP = true`)
- Lệnh nguồn: lệnh dương **không TP** cùng phía với giá hiện tại.
- Lệnh mục tiêu: lệnh âm **có TP** phía đối diện qua base.
- Guard: chỉ chạy khi **không còn lệnh âm không TP ở phía đối diện** với giá hiện tại.
- Điều kiện kích hoạt: `sumSrcPips + sumTgtNegAbsPips >= X`.

**Thứ tự ưu tiên khi bật đồng thời cả 2 mode**
- EA ưu tiên chạy **Mode B (noTP -> TP)** trước; nếu vào nhánh này thì tick đó không chạy Mode A.

**Ghi chú**:
- Nếu `EnableRSIBalanceFilter` đang bật thì cơ chế này vẫn có thể bị chặn theo điều kiện RSI balance (vì nó nằm trong `DoBalanceAll()`).

### 10) RE-ARM DELAY (after TP)
- **RearmDelayMinutesAA/BB/CC/DD**

**Nguyên tắc**:
- Khi 1 lệnh **đóng bằng TP** tại bậc nào thì EA sẽ **delay đặt lại** virtual pending đúng **(loại + bậc)** đó trong X phút.
- Khi hết thời gian delay (hoặc nếu delay = 0), EA **chỉ bổ sung lại** virtual pending tại bậc đó khi **giá hiện tại cách bậc đó tối thiểu 1 bậc lưới** (khoảng cách ≥ `gridStep`).
- Các bậc khác vẫn đặt lại bình thường.

Mặc định hiện tại: `RearmDelayMinutesAA/BB/CC/DD = 20`.

### 11) SESSION RESET (profit target)
- **EnableSessionProfitReset**
- **SessionProfitTargetUSD**

**Nguyên tắc**:
- Khi **không** ở `gongLaiMode`, nếu \( (Balance - sessionStartBalance) + floating \) đạt ngưỡng thì EA **Reset** (đóng hết, đặt base mới, đặt lại lưới).
- Nếu `gongLaiMode` đang kích hoạt thì **không áp dụng** reset theo ngưỡng phiên.

Mặc định hiện tại: `EnableSessionProfitReset = true`, `SessionProfitTargetUSD = 500`.

### 12) RESET KHI LEVELS MATCH
Reset EA khi **đủ đồng thời** 4 điều kiện sau:

1. Có **ít nhất 1 lệnh mở phía trên đường gốc** với khoảng cách đến base **>= X pip** (X cấu hình bởi **LevelMatchMinDistancePips**).
2. Có **ít nhất 1 lệnh mở phía dưới đường gốc** với khoảng cách đến base **>= X pip**.
3. **Tổng P/L phiên (đã trừ tiết kiệm từ TP)**: (lệnh đóng + lệnh đang mở) của phiên hiện tại ≥ **LevelMatchSessionTargetUSD** (USD).
   - Nếu bật `EnableLockProfit` thì phần `LockProfitPct` đã được “lock” từ các TP lãi sẽ bị trừ vào P/L dùng để xét reset.
4. **Chế độ gồng lãi tổng chưa kích hoạt** (không ở gongLaiMode).

**Input:**
- **EnableResetWhenLevelsMatch**: bật/tắt chế độ reset này.
- **LevelMatchMinDistancePips** (X pip): điều kiện (1) và (2) — khoảng cách tối thiểu từ base cho lệnh phía trên và phía dưới.
- **LevelMatchSessionTargetUSD**: điều kiện (3) — ngưỡng P/L phiên (USD).
- Nếu `LevelMatchSessionTargetUSD` **> 0**: reset khi `P/L phiên >= ngưỡng`.
- Nếu `LevelMatchSessionTargetUSD` **< 0**: reset khi `P/L phiên >= ngưỡng` (ví dụ `-100` nghĩa là lỗ không tệ hơn -100 USD; ví dụ -10 cũng reset).

**Ví dụ (X = 300 pip, ngưỡng -20 USD):**
- Đường gốc = 1.10000, bước lưới = 0.00100 (100 pips).
- **Trên gốc:** có ít nhất 1 lệnh cách base từ **300 pip trở lên** ✓
- **Dưới gốc:** có ít nhất 1 lệnh cách base từ **300 pip trở lên** ✓
- **P/L phiên:** (Balance − balance đầu phiên) + floating = **+20 USD** ≥ -20 ✓
- **Gồng lãi tổng:** chưa kích hoạt ✓  
→ EA **reset**: đóng hết lệnh/pending, đặt base mới = giá hiện tại, khởi tạo lại lưới.

Nếu thiếu một trong bốn điều kiện trên thì **không** reset.

Khi đủ cả 4 điều kiện, EA thực hiện reset (đóng hết, base mới, daily/trading/ADX xử lý như các reset khác).

Mặc định: `EnableResetWhenLevelsMatch = true`, `LevelMatchMinDistancePips = 4000`, `LevelMatchSessionTargetUSD = 20`.

### 13) RESTART DELAY (after RESET)
- **RestartDelayMinutesAfterReset**: sau mỗi lần EA **RESET** (đóng hết lệnh/pending), EA sẽ **chờ X phút** rồi mới khởi động lại (đặt base mới + khởi tạo lưới). `0 = off`.

**Lưu ý**:
- Chỉ áp dụng cho các lần **RESET** (trailing lock / trailing SL hit / session reset / levels match…).
- Khi đến thời điểm restart, EA vẫn phải thỏa các điều kiện đang bật như **Trading hours**, **ADX start filter**, **RSI start filter**; nếu chưa thỏa thì EA tiếp tục WAITING theo đúng điều kiện đó.

Mặc định hiện tại: `RestartDelayMinutesAfterReset = 0`.

> **`VP-Grid.mq5` chỉ có input đến nhóm 13.** Không có engine phụ trong file gốc.

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
- `TrailingThresholdUSD = 100`
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

Nếu cần, có thể chỉnh code để `GridDistancePips` quy đổi đúng trên mọi symbol (5/3/4/2 digits) bằng `pipSize` giống phần trailing.

