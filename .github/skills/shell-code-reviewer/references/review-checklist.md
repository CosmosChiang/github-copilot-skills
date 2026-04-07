# Shell 審查清單

使用本清單對應 `SKILL.md` Step 2 中的六個分析面向，逐項確認。

---

## 1. 安全性 (Security)

### 輸入與命令執行
- [ ] 是否使用 `eval`、`bash -c`、`sh -c` 執行外部輸入？（P0）
- [ ] 命令替換 `$(...)` / 反引號是否直接包入未驗證資料？（P0）
- [ ] `find | xargs` 是否使用 `-print0` / `-0` 處理特殊字元？
- [ ] 參數是否正確引用，如 `"$file"`, `"$@"`？未引用且用於 destructive command 視為 P0。

### 機密與網路
- [ ] 是否有硬編碼 password、token、API key？（P0）
- [ ] 是否使用 `curl|sh`、`wget|bash`、`curl -k` 等不安全模式？（P0）
- [ ] 是否將機密寫入 log、`set -x` trace 或 console？
- [ ] `ssh`, `scp`, `rsync` 是否停用 host key 驗證？

### 檔案系統風險
- [ ] 是否在動態路徑上使用 `rm -rf`, `chmod -R`, `chown -R`？（P0）
- [ ] 是否使用 `mktemp` 建立暫存檔？未使用且檔名可預測視情況列 P0/P1。
- [ ] `source` / `.` 是否載入不可信或未驗證路徑？（P0）

---

## 2. 錯誤處理 (Error Handling)

- [ ] 腳本開頭是否使用適當 strict mode，例如 `set -euo pipefail`？（P1 if missing on critical script）
- [ ] 關鍵命令失敗時是否明確 `exit 1` 或回傳錯誤？
- [ ] 是否檢查 `$?`、`PIPESTATUS` 或以 `if ! cmd; then ... fi` 處理失敗？
- [ ] `trap` 是否清理暫存檔、鎖檔、背景程序？（P1 if cleanup missing）
- [ ] 是否濫用 `|| true`, `2>/dev/null` 掩蓋錯誤？（P1）
- [ ] 錯誤訊息是否輸出到 `stderr`（例如 `printf ... >&2`）而非一般輸出？

---

## 3. 效能 (Performance)

### 外部命令與 subshell
- [ ] 是否有 Useless Use of Cat（`cat file | grep ...`）？（P2）
- [ ] 是否在迴圈內重複呼叫 `grep`, `awk`, `sed`, `cut`, `basename`？（P1/P2）
- [ ] 是否建立過多 subshell / command substitution，可改用 shell builtin？
- [ ] 是否讀取整個檔案到記憶體，但其實只需要 `head`, `tail`, `grep -m1`？

### I/O 與批次處理
- [ ] 是否在迴圈內反覆開啟同一檔案或重複執行同一網路請求？（P1）
- [ ] 是否可把多個 `sed` / `awk` 操作合併？
- [ ] 是否有不必要的 `sort | uniq`，其實可以用 `sort -u`？

---

## 4. 程式碼風格 (Style)

### 命名與 quoting
- [ ] 變數、函式、常數命名是否一致且可讀？
- [ ] 參數展開是否一律加引號，除非明確需要 word splitting？（P2 if inconsistent）
- [ ] 是否使用 `local` 宣告函式內變數（Bash/Zsh）？
- [ ] 是否使用 `printf` 取代容易有 portability 問題的 `echo -e`？

### ShellCheck 對照（P2）

| ShellCheck Rule | 說明 |
|-----------------|------|
| `SC2086` | 未引用變數展開，可能造成 word splitting / globbing |
| `SC2046` | 未引用 command substitution |
| `SC2164` | `cd` 後未檢查失敗 |
| `SC2155` | 宣告與指派混在一起可能掩蓋 exit code |
| `SC2034` | 變數已宣告但未使用 |
| `SC2181` | 建議直接檢查命令本身，而非只檢查 `$?` |
| `SC2002` | Useless Use of Cat |
| `SC2016` | 單引號內變數不會展開 |

> 建議在 CI/CD pipeline 中加入 `ShellCheck` 自動檢查以上規則。

---

## 5. 可維護性 (Maintainability)

### 函式設計
- [ ] 每個函式是否符合單一職責原則？
- [ ] 函式是否過長（例如超過 40–60 行）？（P2）
- [ ] 是否有重複程式碼可抽成共用函式？（P2）
- [ ] 是否把設定值寫成具名常數或參數，而不是散落的 Magic Number？

### 文件與用法
- [ ] 是否有 shebang 與 usage/help 說明？（P2 if missing）
- [ ] 複雜邏輯是否有簡短註解說明 intent 而非逐行翻譯？
- [ ] 是否標示必要環境變數、外部依賴與預期輸入輸出？

---

## 6. 相容性 (Compatibility)

### Shell 方言
- [ ] `[[ ... ]]`、arrays、`function name()`、`mapfile`、`${var^^}` 是否需要 Bash/Zsh？若 shebang 為 `/bin/sh`，則為 P1/P2。
- [ ] 若腳本聲稱 POSIX 相容，是否誤用了 Bash-only 語法？（P1）
- [ ] 是否依賴 `source` 而非 POSIX 的 `.`？

### 工具與平台差異
- [ ] `sed -i`, `date`, `mktemp`, `stat`, `readlink` 是否考慮 GNU/BSD 差異？（P2）
- [ ] 路徑、暫存目錄、權限行為是否兼容 Linux/macOS？
- [ ] 是否有 `#!/usr/bin/env bash` 或明確 `# shellcheck shell=bash` 標示需求？（P3 if missing）

---

## 快速評分標準

| 程式碼狀態 | 建議 |
|-----------|------|
| 0 個 P0、0 個 P1 | ✅ 可合併 |
| 0 個 P0、有 P1 | ⚠️ 修復 P1 後合併 |
| 有任何 P0 | ❌ 必須修復 P0 後才能合併 |
