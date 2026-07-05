# GRAFCET → VHDL 高階合成（MIAT 階段三：硬體合成）

## 合成總則

硬體高階合成與軟體合成概念幾乎完全相同：把每個模組分成**控制子系統（Control Subsystem）**與**資料子系統（Data Subsystem）**。

- 控制子系統：接收控制輸入（Control Inputs）、產生控制輸出；發出控制訊號（Control Signals）驅動資料子系統；接收資料子系統回傳的條件訊號（Conditions）決定狀態轉移。
- 資料子系統：接收資料輸入、執行運算、產生資料輸出。
- 每個 GRAFCET 狀態＝一個 `STD_LOGIC` SIGNAL（one-hot 正反器）。
- 每個功能模組可視為一台馮紐曼架構的獨立計算機（有自己的邏輯、狀態、記憶）。

## 固定架構：兩個 PROCESS ＋ ENTITY 整合

### 1. Grafcet 控制器 PROCESS

RST 高電位 → 只有初始狀態為 '1'；時脈正緣 → 依 if/elsif 鏈做狀態轉移：

```vhdl
grafcet : PROCESS(CLK, RST)
BEGIN
    IF RST = '1' THEN
        X0 <= '1'; X1 <= '0'; X2 <= '0';
    ELSIF CLK'EVENT AND CLK = '1' THEN
        IF    X0 = '1'              THEN X0 <= '0'; X1 <= '1';
        ELSIF X1 = '1'              THEN X1 <= '0'; X2 <= '1';
        ELSIF X2 = '1' AND I < 10   THEN X2 <= '0'; X1 <= '1';  -- Divergence OR
        ELSIF X2 = '1' AND I = 10   THEN X2 <= '0'; X0 <= '1';
        END IF;
    END IF;
END PROCESS grafcet;
```

（五大結構的對應與 C 合成規則相同：AND 分歧＝同時設多個狀態 '1'；AND 匯聚＝條件含所有並行狀態；OR 分歧＝互斥條件的 elsif；OR 匯聚＝多來源指向同一目標。）

### 2. Datapath PROCESS

只在時脈正緣觸發，依活躍狀態執行動作，與控制器完全分離：

```vhdl
datapath : PROCESS(CLK, RST)
BEGIN
    IF CLK'EVENT AND CLK = '1' THEN
        IF    X0 = '1' THEN TMP <= 0; I <= 0;      -- 狀態0動作：初始化
        ELSIF X1 = '1' THEN TMP <= TMP + I;        -- 狀態1動作：運算
        ELSIF X2 = '1' THEN I <= I + 1;            -- 狀態2動作：遞增
        END IF;
    END IF;
END PROCESS datapath;
```

### 3. ENTITY 整合

```vhdl
ENTITY SUM IS
    PORT (
        CLK, RST : IN  STD_LOGIC;
        S        : OUT INTEGER RANGE 0 TO 128
    );
END SUM;

ARCHITECTURE arch OF SUM IS
    SIGNAL X0, X1, X2 : STD_LOGIC;
    SIGNAL I          : INTEGER RANGE 0 TO 15;
    SIGNAL TMP        : INTEGER RANGE 0 TO 128;
BEGIN
    grafcet  : PROCESS(CLK, RST) ... ;   -- 控制器
    datapath : PROCESS(CLK, RST) ... ;   -- 資料路徑
    S <= TMP;                            -- 輸出接資料暫存器
END arch;
```

## 演算法 → GRAFCET → 電路：迴圈範例

C 演算法：

```c
int i, Sum = 0;
for (i = 0; i <= 10; i++) {
    Sum = Sum + i;
}
```

GRAFCET 建模（三狀態）：

- 狀態 0（初始）：動作 `Sum=0; i=0`；轉移條件 `=1`（永真）
- 狀態 1（運算）：動作 `Sum=Sum+i`；轉移條件 `=1`
- 狀態 2（遞增）：動作 `i=i+1`；Divergence OR：`i<10` → 回狀態 1；`i=10` → 回狀態 0

合成結果即上方三段 VHDL；EDA 工具模擬波形應顯示 S 最終為 55（1 加到 10）。這示範了無數位邏輯背景者也能把演算法轉成硬體電路。

## 階層式控制器（硬體）

- 上層控制器某狀態活躍（如 X2='1'）→ 觸發下層控制器啟動（A2 START）；下層完成訊號（A21 FINISH／結束狀態暫存器）→ 上層轉移條件。
- 各層溝通全靠狀態變數與轉移條件自動整合，**不需額外通訊協定**；多個子系統可同時運行實現平行處理。
- 效能瓶頸模組（如 LPCC）持續往下分解，直到每一步都可生成硬體電路，即可做成硬體加速器。

## 驗證

- 以 EDA 工具跑波形模擬：檢查 CLK/RST/狀態訊號/輸出值的時序是否符合 GRAFCET 模型預期。
- 硬體程式碼於 FPGA 上驗證；之後與軟體（MCU）進行軟硬體整合驗證。
