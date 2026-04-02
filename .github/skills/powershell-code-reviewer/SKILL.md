---
name: powershell-code-reviewer
description: "Review PowerShell code for quality, security, and best practices. Categorizes findings by severity P0 (Critical), P1 (High), P2 (Medium), P3 (Low). Produces structured review reports with issue lists and actionable recommendations. USE FOR: reviewing .ps1 .psm1 .psd1 files; auditing PowerShell modules; checking security vulnerabilities, error handling, performance, coding style, and maintainability. TRIGGERS: review powershell, audit script, check ps1, powershell code review."
argument-hint: "Paste PowerShell code directly, or provide a file path to review"
---

# PowerShell Code Reviewer

## 語言規範

> **所有回覆必須使用正體中文**。專業術語（如 cmdlet 名稱、pattern 名稱、OWASP 類別、Git 指令等）保留英文原文。

## 用途說明

對輸入的 PowerShell 程式碼進行全面審查，依嚴重等級 P0–P3 分類問題，並產出結構化報告與建議清單。

## When to Use

- 審查 `.ps1` / `.psm1` / `.psd1` 檔案
- 提交 Pull Request 前的自動化品質檢查
- 稽核 PowerShell 模組的安全性與最佳實踐
- 找出潛在的效能瓶頸或邏輯錯誤

---

## Review Procedure

### Step 1 — 接收程式碼

接受以下任一形式的輸入：

- 使用者直接貼上的 PowerShell 程式碼片段
- 工作區內的檔案路徑（使用 `read_file` 讀取）
- 多個檔案路徑（平行讀取所有檔案）

### Step 2 — 分析階段

依序檢查以下六個面向，並收集潛在問題：

1. **安全性 (Security)** — 參考 [severity-guide.md](./references/severity-guide.md) 的 P0 清單
2. **錯誤處理 (Error Handling)** — `try/catch`、`$ErrorActionPreference`、`-ErrorAction`
3. **效能 (Performance)** — 管線使用、迴圈內 I/O、大量物件建立
4. **程式碼風格 (Style)** — 命名慣例、縮排、Verb-Noun 格式
5. **可維護性 (Maintainability)** — 函式長度、重複程式碼、Magic Number
6. **相容性 (Compatibility)** — PS 版本差異、Windows/Linux 路徑、別名使用

詳細的檢查項目請參閱 [review-checklist.md](./references/review-checklist.md)。

### Step 3 — 嚴重等級分類

依據 [severity-guide.md](./references/severity-guide.md) 將每個問題分配到對應等級：

| 等級   | 名稱     | 定義                                                    |
| ------ | -------- | ------------------------------------------------------- |
| **P0** | Critical | 安全漏洞、資料遺失風險、系統損壞 — **必須立即修復**     |
| **P1** | High     | 邏輯錯誤、未處理例外、嚴重效能問題 — **應在合併前修復** |
| **P2** | Medium   | 程式碼風格違規、輕微 Bug、可讀性問題 — **建議修復**     |
| **P3** | Low      | 改善建議、最佳實踐提示、可選優化 — **參考即可**         |

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
    ```powershell
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
- **摘要說明**: ＜2-3 句總結程式碼整體狀況＞

---

### Step 5 — 自動修復建議

針對 P0 與 P1 的所有問題，以及 P2 中有明確修正方式的問題，額外輸出可直接套用的修正程式碼片段：

```powershell
# [P0-1] 修正：移除硬編碼密碼，改用 SecureString 參數
# 原始碼（有問題）：
$password = "MyP@ssw0rd"

# 修正後：
param(
    [Parameter(Mandatory)]
    [securestring] $Password
)
```

**格式規則**：

- 每個程式碼區塊前標示對應的問題編號（如 `[P0-1]`）
- 同時保留「原始碼（有問題）」與「修正後」，方便對照
- 若修正涉及大幅重構，只提供關鍵差異部分，並說明重構方向

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
