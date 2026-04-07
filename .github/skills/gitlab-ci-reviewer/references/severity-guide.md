# GitLab CI/CD 嚴重等級定義指南

## P0 — Critical（🔴 必須立即修復）

> 直接影響機密安全、供應鏈信任、生產環境保護或造成高風險未授權部署。不得合併。

### 安全性問題
- **Secrets Exposure**: 在 YAML、`script:`、artifact、log 中暴露 Token、SSH key、password、cloud credential
- **Unprotected Production Deploy**: production deploy 未限制 protected branch/tag、無 manual approval、無 environment protection
- **Unsafe Remote Execution**: `curl | bash`、遠端下載 script 直接執行、`include:remote` 未固定可信來源
- **Privileged Runner Abuse**: 在 shared Runner 上使用 privileged mode 或 host mount 執行高權限操作
- **Supply Chain Risk**: 使用不可信 image / template，且未 pin version 或 digest
- **Artifact Leakage**: 將 `.env`、私密憑證、signed binaries、部署金鑰輸出成可下載 artifact

### 營運風險
- deploy job 可能在任何 branch 自動推上 production
- 缺少互斥 / 鎖定導致多次併發部署覆寫 production

---

## P1 — High（🟠 應在合併前修復）

> 會導致 pipeline 錯誤、部署失敗、可靠性下降或明顯資源浪費的問題。

### 邏輯與功能錯誤
- `rules` / `workflow: rules` 條件衝突，導致 job 永不執行或執行錯誤時機
- 關鍵測試、掃描或 deploy job 設定 `allow_failure: true`
- deploy stage 可在測試失敗後仍繼續執行
- `needs` / `dependencies` 配置錯誤，導致 artifacts 取不到或 stage 無法正確排序
- 缺少 `resource_group`，造成同一環境的併發部署競態

### 錯誤處理缺失
- shell script 缺少 `set -euo pipefail` 或等效嚴格模式
- 缺少 `timeout:`、`retry:`，使 pipeline 容易卡住或對 transient failure 無保護
- `after_script` 或 cleanup 流程掩蓋真正失敗原因

### 效能與成本問題
- 重複觸發 duplicate pipelines
- cache key 設計錯誤，造成 cache 汙染或完全無法命中
- artifact 過大、保留過久，導致儲存與傳輸成本明顯增加
- image 未固定版本，造成 build 結果不穩定或頻繁失效

---

## P2 — Medium（🟡 建議修復）

> 降低可讀性、可維護性或帶來中度風險的配置問題。

### 程式碼風格與可讀性
- job 名稱不清楚、縮排不一致、YAML 結構混亂
- 混用 `only/except` 與 `rules`，增加理解成本
- `script:` 區塊過長，建議抽成 repo 腳本

### 可維護性
- 相同 job 重複 3 次以上，未使用 `extends` / anchors
- 共用變數沒有集中管理
- 缺少對 release / rollback / manual gate 的說明註解
- 未使用 `rules:changes` 造成次要但可避免的資源浪費

### 相容性
- 使用新語法但未標明最低 GitLab 版本
- Runner shell / executor 假設不明確，可能在不同環境下出錯

---

## P3 — Low（🟢 參考優化）

> 非必要，但能提升長期品質與治理成熟度的建議。

### 最佳實踐建議
- 使用 hidden template jobs（如 `.base_job`）統一共用設定
- 為關鍵 environment 增加 `on_stop` / rollback job
- 補充註解、文件、pipeline 圖示說明
- 更精細地使用 `interruptible`、`resource_group`、`rules:changes`
- 補上 `expire_in`、`cache:policy`、`GIT_DEPTH` 等優化設定
