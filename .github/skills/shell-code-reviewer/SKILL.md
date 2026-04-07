---
name: shell-code-reviewer
description: "Review shell scripts for security, reliability, portability, and best practices. Categorizes findings by severity P0 (Critical), P1 (High), P2 (Medium), P3 (Low). Produces structured review reports with issue lists and actionable fix snippets. USE FOR: reviewing .sh .bash .zsh scripts; auditing CI shell steps; checking ShellCheck issues, quoting bugs, unsafe eval/rm/curl patterns, and portability problems. TRIGGERS: review shell, audit bash script, check sh, shell code review."
argument-hint: "Paste shell code directly, or provide a file path to review"
---

# Shell Code Reviewer

## 語言規範

> **所有回覆必須使用正體中文**。專業術語（如 shell builtins、ShellCheck rule、POSIX、OWASP 類別、Git 指令等）保留英文原文。

## 用途說明

對輸入的 shell 程式碼（`sh` / `bash` / `zsh`）進行全面審查，依嚴重等級 P0–P3 分類問題，並產出結構化報告、優先修復建議與可直接套用的修正片段。

## When to Use

- 審查 `.sh` / `.bash` / `.zsh` 腳本
- 稽核 CI/CD workflow 中的 shell step
- 找出 quoting、`eval`、`rm -rf`、暫存檔處理等高風險問題
- 針對 `ShellCheck` 常見警告、portable shell 問題與維護性風險做人工複核

---

## Review Procedure

### Step 1 — 接收程式碼

接受以下任一形式的輸入：

- 使用者直接貼上的 shell 程式碼片段
- 工作區內的檔案路徑（使用 `read_file` 讀取）
- 多個檔案路徑（平行讀取所有檔案）

如腳本未明確標示 shebang，需在報告中說明假設的 shell 方言（例如 Bash 或 POSIX `sh`）。

### Step 2 — 分析階段

依序檢查以下六個面向，並收集潛在問題：

1. **安全性 (Security)** — `eval`、未引用變數、危險刪除、暫存檔、秘密外洩、`curl|sh`
2. **錯誤處理 (Error Handling)** — `set -euo pipefail`、exit code、`trap`、pipeline failure、錯誤訊息
3. **效能 (Performance)** — 不必要的 subshell、重複外部命令、迴圈內 I/O、UUOC、過度 `grep|awk|sed`
4. **程式碼風格 (Style)** — quoting、一致命名、函式風格、`printf`、`ShellCheck` 規則
5. **可維護性 (Maintainability)** — 函式長度、重複邏輯、Magic Number、註解與用法說明
6. **相容性 (Compatibility)** — shebang、POSIX/Bash 差異、GNU/BSD 工具差異、Linux/macOS 行為差異

詳細檢查項目請參閱 [review-checklist.md](./references/review-checklist.md)。

### Step 3 — 嚴重等級分類

依據 [severity-guide.md](./references/severity-guide.md) 將每個問題分配到對應等級：

| 等級   | 名稱     | 定義                                                                           |
| ------ | -------- | ------------------------------------------------------------------------------ |
| **P0** | Critical | 安全漏洞、資料遺失風險、系統損壞 — **必須立即修復**                            |
| **P1** | High     | 邏輯錯誤、未處理例外、嚴重 portability / reliability 問題 — **應在合併前修復** |
| **P2** | Medium   | 程式碼風格違規、輕微 Bug、可讀性問題 — **建議修復**                            |
| **P3** | Low      | 改善建議、最佳實踐提示、可選優化 — **參考即可**                                |

### Step 4 — 產出報告

以下列格式輸出完整報告：

---

## 📋 審查報告 — `<檔案名稱或程式碼摘要>`

### 摘要

| 等級           | 數量  |
| -------------- | ----- |
| 🔴 P0 Critical | N     |
| 🟠 P1 High     | N     |
| 🟡 P2 Medium   | N     |
| 🟢 P3 Low      | N     |
| **合計**       | **N** |

---

### 🔴 P0 — Critical（必須立即修復）

> （若無則標示：無 P0 問題）

- **[P0-1]** `<問題標題>`
  - **位置**: 第 X 行 / 函式 `<name>`
  - **說明**: ＜問題的具體描述＞
  - **風險**: ＜若不修復會發生什麼事＞
  - **修復建議**:
    ```sh
    # 建議的修正程式碼
    ```

### 🟠 P1 — High（合併前修復）

- **[P1-1]** ...

### 🟡 P2 — Medium（建議修復）

- **[P2-1]** ...

### 🟢 P3 — Low（參考優化）

- **[P3-1]** ...

---

### ✅ 建議清單（優先順序）

依修復優先序列出所有行動項目：

1. `[ ]` **[P0-1]** ＜修復描述＞
2. `[ ]` **[P1-1]** ＜修復描述＞
3. `[ ]` **[P2-1]** ＜修復描述＞
4. `[ ]` **[P3-1]** ＜修復描述＞

---

### 📌 整體評分與總結

- **整體品質**: ⭐⭐⭐☆☆（3/5）
- **是否建議合併**: ✅ 可合併 / ❌ 需修復後再合併 / ⚠️ 建議修復後合併
- **摘要說明**: ＜2-3 句總結腳本整體狀況，並指出最關鍵的風險類型＞

---

### Step 5 — 自動修復建議

針對 P0 與 P1 的所有問題，以及 P2 中有明確修正方式的問題，額外輸出可直接套用的修正程式碼片段：

```sh
# [P1-1] 修正：啟用 strict mode 並安全處理暫存檔
# 原始碼（有問題）：
tmp=/tmp/data.$$
cp $src $tmp
rm -f $tmp

# 修正後：
set -euo pipefail
tmp="$(mktemp)"
trap 'rm -f "$tmp"' EXIT
cp -- "$src" "$tmp"
```

**格式規則**：

- 每個程式碼區塊前標示對應的問題編號（如 `[P0-1]`）
- 同時保留「原始碼（有問題）」與「修正後」，方便對照
- 若修正涉及大幅重構，只提供關鍵差異部分，並說明重構方向
- 若問題與 shell 方言有關，需標註修正適用於 Bash 或 POSIX `sh`

---

## Quality Completion Criteria

報告完成前確認：

- [ ] 所有回覆使用正體中文，專業術語保留英文
- [ ] 涵蓋全部六個分析面向
- [ ] 每個問題都標示了行號或函式名稱
- [ ] 每個 P0/P1 問題都附有修復程式碼範例
- [ ] Step 5 已對所有 P0/P1 及可修復 P2 輸出自動修復片段
- [ ] 建議清單按嚴重等級排序
- [ ] 整體評分與合併建議已填寫
- [ ] 已標明腳本假設的 shell 方言與相容性前提
