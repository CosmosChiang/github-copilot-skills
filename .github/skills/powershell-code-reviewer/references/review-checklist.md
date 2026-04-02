# PowerShell 審查清單

使用本清單對應 SKILL.md Step 2 中的六個分析面向，逐項確認。

---

## 1. 安全性 (Security)

### 輸入驗證
- [ ] 使用者輸入是否通過 `[ValidatePattern]`、`[ValidateSet]`、`[ValidateRange]` 等驗證？
- [ ] `Invoke-Expression` / `& $cmd` 是否接受外部變數？（P0）
- [ ] 系統呼叫（`Start-Process`, `cmd.exe /c`）是否組合了未驗證字串？（P0）

### 憑證與機密
- [ ] 是否有硬編碼密碼、API Key 或 Token？（P0）
- [ ] `ConvertTo-SecureString -AsPlainText` 的來源是否安全？
- [ ] `PSCredential` 是否有可能被序列化或輸出至日誌？（P0）

### 網路安全
- [ ] 是否停用了 TLS 驗證？（P0）
- [ ] `Invoke-WebRequest` / `Invoke-RestMethod` 是否驗證了回應內容？

### 執行原則
- [ ] 是否無故使用 `-ExecutionPolicy Bypass`？
- [ ] 是否使用 `[System.Reflection.Assembly]::Load` 載入不明來源的組件？

---

## 2. 錯誤處理 (Error Handling)

- [ ] 檔案系統操作是否有 `try/catch`？（P1 if missing）
- [ ] 網路呼叫是否有 `try/catch` 重試機制？（P1 if missing）
- [ ] `$ErrorActionPreference` 是否在腳本開頭或關鍵區段設為 `'Stop'`？
- [ ] `catch` 區塊是否存在實質處理（非空白 catch）？（P1 if empty）
- [ ] 是否使用 `$?` 或 `$LASTEXITCODE` 檢查外部命令結果？
- [ ] `finally` 區塊是否適當清理資源（關閉連線、刪除暫存档）？
- [ ] 是否使用 `Write-Error` / `throw` 而非 `Write-Host` 回報錯誤？（P2 if Write-Host）

---

## 3. 效能 (Performance)

### 集合操作
- [ ] 陣列是否在迴圈內使用 `+=` 拼接？（P1 — 改用 `[List[T]]` 或 `@()` + 管線）
- [ ] 是否有大型集合可用 `Where-Object` 過濾後再處理？
- [ ] 是否已使用 `Select-Object -First N` 避免讀取不必要資料？

### I/O 操作
- [ ] 是否在迴圈內重複讀取同一檔案？（P1 — 應快取到變數）
- [ ] 是否在迴圈內重複開啟資料庫 / 網路連線？（P1 — 應連線池化）
- [ ] `Get-Content` 大檔案是否使用 `-ReadCount` 分批或 `StreamReader`？

### 管線與篩選
- [ ] 是否有可用管線取代 `foreach` 的操作（增加可讀性與效能）？
- [ ] 是否將篩選條件移到最早的管線步驟（左移過濾）？
- [ ] 是否大量使用 `ForEach-Object` 而非 `foreach` 語句（後者更快）？

---

## 4. 程式碼風格 (Style)

### 命名
- [ ] 函式是否遵循 `Verb-Noun` 格式？（使用 `Get-Verb` 確認動詞）（P2 if not）
- [ ] 變數名稱是否具有描述性？（P2 if `$x`, `$tmp`）
- [ ] 是否使用 PowerShell 別名（`ls`, `cat`, `%`, `?`）於非互動腳本中？（P2）

### 格式
- [ ] 縮排是否一致（建議 4 空格）？（P2 if inconsistent）
- [ ] 行長是否在 120 字元以內？（P2 if over）
- [ ] `{` 是否放在同行（One True Brace Style）？
- [ ] 是否有不必要的空白行或重複空行？

### 參數宣告
- [ ] 函式是否使用 `param()` 而非直接存取 `$args`？（P2 if using $args）
- [ ] 強制參數是否標記 `[Parameter(Mandatory)]`？

### PSScriptAnalyzer 規則對照（P2）

下列為 [PSScriptAnalyzer](https://github.com/PowerShell/PSScriptAnalyzer) 常見告警，對應 P2 等級：

| PSScriptAnalyzer Rule | 說明 |
|-----------------------|------|
| `PSAvoidUsingCmdletAliases` | 使用了別名（`ls`、`%`、`?`），應改為完整 cmdlet 名稱 |
| `PSAvoidUsingWriteHost` | 使用 `Write-Host` 而非 `Write-Output` / `Write-Verbose` |
| `PSUseDeclaredVarsMoreThanAssignments` | 變數宣告後從未使用（潛在 typo 或 dead code） |
| `PSAvoidGlobalVars` | 使用全域變數 `$global:X`，應改用參數傳遞 |
| `PSUseShouldProcessForStateChangingFunctions` | 變更狀態的函式缺少 `ShouldProcess` 支援 |
| `PSUsePSCredentialType` | 密碼參數應使用 `[PSCredential]` 而非 `[string]` |
| `PSReviewUnusedParameter` | 宣告的參數在函式中從未被使用 |
| `PSAvoidTrailingWhitespace` | 行尾有多餘空白字元 |
| `PSPlaceOpenBrace` | `{` 未放在同行（違反 One True Brace Style） |
| `PSUseConsistentIndentation` | 縮排不一致 |

> 建議在 CI/CD pipeline 中加入 `Invoke-ScriptAnalyzer` 自動執行以上規則。

---

## 5. 可維護性 (Maintainability)

### 函式設計
- [ ] 每個函式是否符合單一職責原則？
- [ ] 函式是否超過 50 行？（P2 — 考慮拆分）
- [ ] 是否有相同邏輯重複超過 3 次？（P2 — 提取為函式）
- [ ] 是否使用 `[CmdletBinding()]` 讓函式繼承公共參數？（P3 if missing）

### 文件
- [ ] 公開函式是否有 Comment-Based Help？（P2 if missing）
  ```powershell
  <#
  .SYNOPSIS  短描述
  .PARAMETER Name  參數說明
  .EXAMPLE   使用範例
  #>
  ```
- [ ] 複雜演算法是否有內嵌說明？（P2 if missing）
- [ ] Magic Number 是否已命名？（P2）

### 輸出
- [ ] 是否使用 `Write-Output` 而非 `Write-Host`（後者無法被管線捕捉）？（P2）
- [ ] 是否有不必要的 `return` 在函式最後一行？（P3 — 可省略）

---

## 6. 相容性 (Compatibility)

### 跨平台
- [ ] 路徑字串是否使用 `Join-Path` 而非硬編碼 `\`？（P2 for hardcoded）
- [ ] 是否依賴 `$env:COMPUTERNAME` 等 Windows 環境變數？（P3 — 加平台檢查）
- [ ] 是否使用了 Linux 不支援的 WMI / COM 物件？（P3）
- [ ] 是否使用了 `[System.Windows.Forms]` 等 Windows UI 元件？（P3）

### 版本相容性
- [ ] 是否使用了 PS 7+ 的新語法（三元運算子 `? :`、管線鏈 `&&`/`||`）但未標示版本需求？（P2）
- [ ] 是否有 `#Requires -Version` 或 `#Requires -Modules` 宣告？（P3 if missing）
- [ ] `[PSCustomObject]` 的建立方式是否相容於 PS 3+？

---

## 快速評分標準

| 程式碼狀態 | 建議 |
|-----------|------|
| 0 個 P0、0 個 P1 | ✅ 可合併 |
| 0 個 P0、有 P1 | ⚠️ 修復 P1 後合併 |
| 有任何 P0 | ❌ 必須修復 P0 後才能合併 |
