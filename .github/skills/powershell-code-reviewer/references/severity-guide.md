# PowerShell 嚴重等級定義指南

## P0 — Critical（🔴 必須立即修復）

> 直接影響安全性、資料完整性或系統穩定性。不得合併。

### 安全性問題
- **命令注入 (Command Injection)**: 直接使用未驗證的使用者輸入組合 `Invoke-Expression`、`& $cmd`
- **明文密碼**: 在腳本中硬編碼密碼、API Key、連線字串
- **憑證暴露**: 將 `PSCredential` 物件序列化或記錄到日誌
- **不安全的網路傳輸**: 使用 `[System.Net.ServicePointManager]::ServerCertificateValidationCallback` 停用 TLS 驗證
- **任意程式碼執行**: `Invoke-Expression`、`[scriptblock]::Create()` 接受外部輸入
- **路徑穿越 (Path Traversal)**: 未過濾 `../` 的動態路徑存取
- **過高權限**: 不必要地要求 `RunAs Administrator` 或修改系統安全原則

### 資料遺失風險
- 無備份機制直接使用 `Remove-Item -Recurse -Force` 於動態路徑
- 覆寫重要設定檔前未確認

---

## P1 — High（🟠 應在合併前修復）

> 功能性錯誤或嚴重效能問題，影響程式正確執行。

### 邏輯與功能錯誤
- 函式從不回傳值，但呼叫端依賴回傳值
- 比較運算子誤用（`=` 代替 `-eq`）
- 條件判斷永遠為真或永遠為假
- 迴圈索引錯誤（off-by-one）
- `switch` 缺少 `default:` / `Break` 導致貫穿

### 錯誤處理缺失
- 關鍵操作（檔案 I/O、網路呼叫、資料庫）缺少 `try/catch`
- `catch` 區塊為空或只記錄錯誤但繼續執行危險操作
- 未設定 `$ErrorActionPreference = 'Stop'` 導致錯誤被忽略
- 使用 `Write-Host` 而非 `Write-Error` / `throw` 報告錯誤

### 效能問題
- 在迴圈內使用 `+=` 拼接陣列（應使用 `[System.Collections.Generic.List]`）
- 在迴圈內重複開啟/關閉資料庫連線
- 大量資料未分頁直接載入記憶體
- ForEach 迴圈內呼叫緩慢的外部命令（應改用管線或批次呼叫）

---

## P2 — Medium（🟡 建議修復）

> 降低可讀性、可維護性或有輕微的功能風險。

### 程式碼風格
- 函式名稱未遵循 `Verb-Noun` 命名慣例（`Get-`, `Set-`, `New-`, `Remove-` 等）
- 變數命名不清晰（`$x`, `$temp`, `$data`）
- 縮排不一致（混用 Tab 與 Space）
- 行長超過 120 個字元
- 使用 PowerShell 別名（`ls`, `cat`, `%`, `?`）於腳本中（應使用完整 Cmdlet 名稱）

### 文件與可讀性
- 公開函式缺少 Comment-Based Help（`.SYNOPSIS`, `.PARAMETER`, `.EXAMPLE`）
- 複雜邏輯缺少內嵌註解
- Magic Number / Magic String 未定義為具名常數或參數

### 可維護性
- 函式超過 50 行（建議拆分）
- 重複程式碼區塊 3 次以上（應提取為函式）
- 輸出直接使用 `Write-Host`（阻擋管線傳遞，應改用 `Write-Output`）
- 混用 `param()` 與直接使用 `$args`

---

## P3 — Low（🟢 參考優化）

> 非必要但能提升程式碼品質的改善建議。

### 最佳實踐建議
- 可使用 `[CmdletBinding()]` 讓函式支援 `-Verbose`, `-WhatIf`, `-Confirm`
- 使用 `#Requires -Version X.X` 或 `#Requires -Modules` 明確聲明依賴
- 字串格式化建議使用 `-f` 格式運算子或 `$()` 替代多層引號
- 可用 `Where-Object` + `Select-Object` 的管線替代手工迴圈篩選
- 測試檔案是否存在建議使用 `Test-Path` 而非直接存取並 catch 例外
- 考慮加入 `Set-StrictMode -Version Latest` 提早發現未定義變數

### 相容性提示
- 字串中的反斜線路徑在 Linux 上可能失敗，建議使用 `Join-Path`
- `[System.DateTime]` 格式化字串可能受系統語系影響
- 依賴 Windows 特有 API（WMI / COM）時加入版本/平台檢查
