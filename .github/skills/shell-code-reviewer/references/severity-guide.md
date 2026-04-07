# Shell 嚴重等級定義指南

## P0 — Critical（🔴 必須立即修復）

> 直接影響安全性、資料完整性或系統穩定性。不得合併。

### 安全性問題
- **Command Injection**：使用未驗證輸入組合 `eval`、`bash -c`、`sh -c`、反引號或 `$(...)`
- **危險刪除 / 覆寫**：對動態路徑執行 `rm -rf`, `mv`, `cp -f`, `chmod -R`, `chown -R` 且缺少 guard
- **未引用變數造成路徑展開**：在刪除、搬移、解壓或上傳指令中直接使用 `$file`, `$dir`, `$*`
- **硬編碼機密**：腳本內直接包含 password、API key、token、SSH private key
- **不安全下載執行**：`curl ... | sh`、`wget ... | bash`、`curl -k`、`wget --no-check-certificate`
- **暫存檔競態 / Symlink 攻擊**：使用可預測檔名（例如 `/tmp/app.$$`）而非 `mktemp`
- **載入不可信內容**：`source` / `.` 載入來自外部輸入或未驗證路徑的腳本
- **停用 SSH / TLS 驗證**：如 `StrictHostKeyChecking=no` 用於正式流程且未說明風險

### 資料遺失風險
- 在根目錄、使用者家目錄或可空字串路徑上執行 destructive command
- 未備份即覆寫設定檔、憑證或資料庫 dump

---

## P1 — High（🟠 應在合併前修復）

> 功能性錯誤、可靠性問題或嚴重 portability 問題，影響腳本正確執行。

### 邏輯與功能錯誤
- 關鍵命令未檢查 exit code，仍繼續後續流程
- 缺少 `set -e` / `pipefail` 導致 pipeline 前段失敗被忽略
- 使用 `for x in $(command)` 解析檔名或多行輸出，造成 split/glob 錯誤
- 依賴未宣告的環境變數，腳本在空值時產生錯誤結果
- `trap` 缺失，導致暫存檔或鎖檔未清理
- `read` / `xargs` / `find` 未處理空白、換行或特殊字元

### 錯誤處理缺失
- 關鍵 I/O、下載、部署、資料庫操作缺少明確錯誤處理
- 發生錯誤只 `echo` 訊息但不 `exit 1`
- `|| true`、`2>/dev/null` 被濫用而掩蓋失敗

### 效能與穩定性問題
- 在大型迴圈內重複呼叫 `grep`、`sed`、`awk`、`cat` 等外部命令
- 重複讀寫同一檔案而未快取結果
- 以非 portable 方式依賴 GNU-only 參數但未標示環境需求

---

## P2 — Medium（🟡 建議修復）

> 降低可讀性、可維護性，或有輕微功能風險。

### 程式碼風格
- 變數未加雙引號（雖目前輸入看似安全，但易產生 split/glob 問題）
- 使用過時反引號 `` `cmd` `` 而非 `$(cmd)`
- `echo` 用於格式化輸出而非 `printf`
- 函式名稱、常數、區域變數命名不一致
- 縮排、空白與區塊風格不一致
- 違反常見 `ShellCheck` 規則（如 `SC2086`, `SC2046`, `SC2164`）

### 文件與可讀性
- 缺少 shebang 或 usage 說明
- 複雜條件分支缺少註解
- Magic Number / Magic String 未抽為常數或參數

### 可維護性
- 單一函式過長、重複邏輯超過 3 次
- 全域變數過多，缺少 `local` 或 `readonly`
- 沒有將 reusable 邏輯抽成函式

---

## P3 — Low（🟢 參考優化）

> 非必要但能提升程式碼品質與可攜性的改善建議。

### 最佳實踐建議
- 可加入 `set -u`、`readonly`、`umask`、`trap cleanup EXIT`
- 使用 `command -v` 檢查依賴工具是否存在
- 使用 `case` 取代冗長的 `if/elif`
- 在 CI 中加入 `ShellCheck` 與格式檢查
- 對長選項與環境需求補充 usage 文件

### 相容性提示
- `[[ ... ]]`、arrays、process substitution、`mapfile` 為 Bash 特性，POSIX `sh` 不支援
- `sed -i`、`date`, `mktemp`, `readlink` 在 GNU 與 BSD 上參數可能不同
- 若目標平台包含 macOS 或 BusyBox，需明確標示支援範圍
