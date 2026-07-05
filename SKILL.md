---
name: miat-methodology
description: Use when designing, modeling, reviewing, or grading an embedded/control system with the MIAT methodology (國立中央大學 陳慶瀚教授) — triggers include MIAT、IDEF0 系統架構/功能分解、GRAFCET 離散事件建模、高階合成（GRAFCET 合成 C 或 VHDL）、控制器/資料路徑分離、嵌入式系統設計方法論課程作業（自走車、循跡機器人、語者辨識等）, or any request to take requirements through architecture, discrete-event models, and synthesized controller code.
---

# MIAT 系統設計方法論

## Overview

MIAT 方法論（國立中央大學資工系 機器智慧與自動化技術實驗室 MIAT Lab，陳慶瀚教授；課程中亦稱 Methods for Integrated Architecture and Design）是一套 Top-Down 嵌入式系統設計方法論，四個階段環環相扣：

1. **IDEF0 階層式模組化架構設計** — 把系統功能分解到「原子模組」
2. **GRAFCET 離散事件建模** — 每一個功能模組建一張 GRAFCET 描述行為
3. **高階合成（High-Level Synthesis）** — 依固定合成規則將 GRAFCET 機械式轉為 C（軟體）或 VHDL（硬體）
4. **軟硬體整合驗證** — MCU/FPGA 上驗證，最終整合

核心原則：**控制器（Controller）與資料路徑（Datapath）分離**。狀態轉移邏輯由合成規則全自動產生（嚴謹、可驗證、不可即興）；動作（Action/Datapath）的具體實作交給工程師手寫或 LLM 生成。這種解耦從根本上避免「整合地獄（Integration Hell）」。

## 鐵律 — 違反任何一條就不是 MIAT 合成

寫出來的程式若違反以下規則，必須整段重寫，不得以「等效寫法」辯護：

1. **狀態 = 全域布林狀態暫存器 X0, X1, …（one-hot）**。禁止 `enum + switch(state)` 單一狀態變數寫法。
2. **程式架構固定三層**：`main(){ while(1){ grafcet(); } }`、`grafcet(){ 狀態轉移 if 鏈; action(); }`、`action(){ if(Xn==1){…} … }`。
3. **合成規則逐條照搬** [references/synthesis-c.md](references/synthesis-c.md)（五大基本結構 + Sub-Grafcet + Reset）。不得自創或改編規則。
4. **階層編號**：狀態 N 的子 GRAFCET 狀態編號 N0, N1, N2…（狀態 2 → 20, 21…；狀態 34 → 340, 341…）。父子控制器以狀態暫存器握手，不設計額外通訊協定。
5. **GRAFCET 語法**：狀態與轉移條件嚴格交替、禁止自迴圈箭頭、永真條件明寫 `=1`、平行（AND）用雙線、分支（OR）用單線且條件互斥。

## 工作流程（八步驟）

依序執行，每步驟有明確產出物：

1. **功能模組原理（演算法）說明** — 定義功能與演算法原理
2. **IDEF0 系統架構** — 階層式功能分解圖 → [references/idef0.md](references/idef0.md)
3. **FBD 功能方塊圖定義** — 每個模組的輸入/輸出/控制
4. **GRAFCET 離散事件建模** — 每模組一張圖＋狀態/動作說明表 → [references/grafcet.md](references/grafcet.md)（若無軟體模組，跳到步驟 7）
5. **系統程式合成** — [references/synthesis-c.md](references/synthesis-c.md)（C）或 [references/synthesis-vhdl.md](references/synthesis-vhdl.md)（VHDL）
6. **系統程式驗證** — 模擬狀態暫存器轉移序列；記錄 code size、記憶體用量、效能
7. **Driver 程式設計 / AI 生成** — 硬體周邊驅動，手寫或 LLM 生成
8. **軟硬體整合驗證** — 控制器軟體＋Driver＋硬體平台（MCU/FPGA）

完整報告骨架見 [templates/design-report-template.md](templates/design-report-template.md)。

## Quick Reference

| 階段 | 圖形語言/工具 | 產出物 | 參考文件 |
|---|---|---|---|
| 架構設計 | IDEF0（另繪實體架構圖） | A0 頂層圖＋逐層分解圖（A1…、A31…） | references/idef0.md |
| 行為建模 | GRAFCET | 每模組一張圖＋狀態/動作表 | references/grafcet.md |
| 軟體合成 | 合成規則 → C | main/grafcet/action 三層程式 | references/synthesis-c.md |
| 硬體合成 | 合成規則 → VHDL | grafcet PROCESS＋datapath PROCESS | references/synthesis-vhdl.md |
| 驗證 | 模擬/EDA 波形 | 狀態序列、code size、記憶體、效能 | synthesis-*.md 驗證節 |

## Common Mistakes

| 錯誤 | 修正 |
|---|---|
| `switch(state)` + enum 寫控制器 | 布林狀態暫存器 + if 鏈（鐵律 1、2） |
| 自創合成規則（如「R1–R8」） | 只用課程五大結構規則 + Sub-Grafcet + Reset |
| GRAFCET 狀態畫自迴圈箭頭 | 停留在狀態即可，不畫迴圈 |
| 兩轉移條件相鄰、或兩狀態相鄰 | 狀態與轉移必須交替 |
| 一條轉移線連到多個狀態 | 只有 AND 分歧（雙線）可同時啟動多狀態 |
| 把實體架構和功能架構混在一張圖 | 兩張圖分開獨立繪製 |
| 子系統完成用回呼/旗標協定 | 用子控制器結束狀態暫存器（如 X14==1）當父轉移條件 |
| 忘記轉移時復位子控制器 | 父轉移須 `x1=0; x14=0; x10=1; x2=1;` 樣式 |

## When NOT to Use

- 純資料流/演算法優化問題（無離散事件行為）— 只取 IDEF0 分解即可
- 對方明確要求其他方法論（UML/SysML 等）— 勿混用符號
