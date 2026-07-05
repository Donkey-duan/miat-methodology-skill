# MIAT Methodology Skill

讓 AI 編碼代理（Claude Code 等）正確運用 **MIAT 系統設計方法論** 的 Agent Skill：IDEF0 階層式架構設計 → GRAFCET 離散事件建模 → 高階合成（C / VHDL）→ 軟硬體整合驗證。

An [Agent Skill](https://agentskills.io) that teaches AI coding agents the **MIAT system design methodology** (Prof. Ching-Han Chen 陳慶瀚, MIAT Lab, National Central University, Taiwan): IDEF0 hierarchical architecture design → GRAFCET discrete-event modeling → high-level synthesis to C / VHDL → HW/SW integration verification.

## 為什麼需要這個 Skill？

沒有這個 skill 時，AI 對「用 MIAT 方法論設計」的請求會**即興發揮**：用 `switch-case` 寫控制器、自創合成規則、不知道 `main()/grafcet()/action()` 三層架構與階層式狀態編號。這個 skill 把課程的精確規則固化下來：

- IDEF0 語法（ICOM、動詞方塊/名詞箭頭、A0→An→Anm 編號、原子模組）
- GRAFCET 語法（狀態/轉移交替、禁自迴圈、AND 雙線/OR 單線、Sub-GRAFCET 階層編號）
- **五大基本結構的 C 合成規則**＋Sub-Grafcet 函式呼叫＋父子控制器握手＋Reset
- **VHDL 雙 PROCESS 合成範本**（grafcet 控制器＋datapath）
- 八步驟工作流程與設計報告模板

## 安裝

### Claude Code（全機可用，所有專案）

```bash
# macOS / Linux
git clone https://github.com/Donkey-duan/miat-methodology-skill.git ~/.claude/skills/miat-methodology

# Windows (PowerShell)
git clone https://github.com/Donkey-duan/miat-methodology-skill.git "$env:USERPROFILE\.claude\skills\miat-methodology"
```

### 單一專案

```bash
git clone https://github.com/Donkey-duan/miat-methodology-skill.git .claude/skills/miat-methodology
```

安裝後對 Claude 說「用 MIAT 方法論設計一個…」即會自動觸發；也可明確要求「使用 miat-methodology skill」。

本 skill 遵循 [agentskills.io](https://agentskills.io) 規格，任何支援 Agent Skills 的代理（Claude Code、Claude.ai、相容工具）皆可使用。

## 檔案結構

```
miat-methodology-skill/
├── SKILL.md                          # 入口：方法論總覽、鐵律、八步驟、速查表
├── references/
│   ├── idef0.md                      # 階段一：IDEF0 架構設計規則
│   ├── grafcet.md                    # 階段二：GRAFCET 建模規則＋常見錯誤
│   ├── synthesis-c.md                # 階段三：GRAFCET→C 合成規則＋完整範例
│   └── synthesis-vhdl.md             # 階段三：GRAFCET→VHDL 合成範本
└── templates/
    └── design-report-template.md     # 八步驟設計報告模板
```

## 使用範例

```
用 MIAT 方法論設計一個循跡機器人：兩組 IR 感測器、左右兩個直流馬達，
請完成 IDEF0 架構、GRAFCET 離散事件模型、並合成 C 系統程式。
```

Skill 會引導代理產出：實體/功能架構圖 → 每模組 GRAFCET（含狀態/動作表）→ 依五大規則合成的 `grafcet()/action()` 控制器程式 → 狀態轉移模擬驗證。

## 出處與致謝

本 skill 內容整理自國立中央大學資工系 **陳慶瀚教授**（機器智慧與自動化技術 MIAT 實驗室）之「MIAT 系統設計方法論」課程（嵌入式系統設計：系統設計方法論、系統架構設計、離散事件建模、高階合成）。方法論之學術原創屬於陳慶瀚教授與 MIAT 實驗室；本專案為方法論規則的重新表述與工具化，不包含課程原始教材檔案。

## License

MIT — see [LICENSE](LICENSE).
