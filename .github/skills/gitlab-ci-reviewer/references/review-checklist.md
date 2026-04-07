# GitLab CI/CD 審查清單

使用本清單對應 `SKILL.md` Step 2 中的六個分析面向，逐項確認。

---

## 1. 安全性 (Security)

### 機密與存取控制
- [ ] 是否把 password、Token、SSH key、registry credentials 寫死在 YAML 或 `script:` 中？（P0）
- [ ] 敏感變數是否應標記為 `masked` / `protected`，且只允許 protected branches / tags 使用？（P0 if exposed）
- [ ] production deploy job 是否缺少 manual approval、environment protection 或 branch/tag guard？（P0/P1）
- [ ] 是否將 `.env`、私密金鑰、部署憑證存成長期 artifact？（P0）

### 供應鏈與映像來源
- [ ] `image:` 是否使用 `latest` 或未釘選 digest/tag？（P1，若來自不可信 registry 可升 P0）
- [ ] `include:` 是否引用不受信任的遠端 template 或未固定 ref？（P0/P1）
- [ ] `curl | bash`、`wget | sh`、動態下載 script 後直接執行是否存在？（P0）
- [ ] third-party container / package repository 是否未驗證來源？

### Runner 與執行權限
- [ ] job 是否依賴 privileged Docker、`docker:dind`、host mount，但未限制使用範圍？（P0/P1）
- [ ] 是否使用 shared Runner 執行高權限 deploy 或簽章流程？（P1）
- [ ] 是否把 `CI_JOB_TOKEN`、cloud credential 直接 echo 到 log？（P0）

---

## 2. 錯誤處理與 Pipeline 正確性 (Error Handling & Reliability)

### 執行失敗控制
- [ ] shell step 是否缺少 `set -euo pipefail` 或等效嚴格模式？（P1）
- [ ] `allow_failure: true` 是否用在測試、掃描或 deploy 關鍵 job 上？（P1）
- [ ] 關鍵 job 是否缺少 `retry:` / `timeout:` / 明確失敗處理？（P1）
- [ ] `after_script` 是否可能覆蓋原始錯誤資訊或造成誤判？

### Pipeline 邏輯
- [ ] `rules:`、`only/except:`、`workflow: rules` 是否互相衝突或永遠不會觸發？（P1）
- [ ] stage 順序是否正確，是否存在 deploy 在 test 之前執行的風險？（P1）
- [ ] `needs:` / `dependencies:` 是否缺失，導致 artifact 取不到或 job 無法並行？（P1/P2）
- [ ] `when: manual`、`when: always`、`when: on_failure` 是否符合預期？
- [ ] 是否有可能產生 duplicate pipelines（push + merge_request_event 同時跑）？（P1/P2）

---

## 3. 效能 (Performance)

### Cache 與 Artifact
- [ ] `cache:` key 是否過於寬鬆，造成汙染或低命中率？（P1/P2）
- [ ] artifact 是否包含過大目錄（如整個 repo、`node_modules`、build cache）？（P1/P2）
- [ ] artifact retention (`expire_in`) 是否缺失，導致儲存成本增加？
- [ ] 是否把可由 cache 處理的資料誤放進 artifacts？

### Pipeline 速度
- [ ] 是否未使用 `needs:`、matrix、平行 job，導致 stage 串行等待過久？（P2）
- [ ] 是否每次都 full clone，而非設定 `GIT_DEPTH` / fetch strategy？（P2）
- [ ] 是否重複下載相同依賴或重跑相同測試？（P2）
- [ ] `rules:changes` 是否可用來避免不必要的 job？（P2/P3）

---

## 4. 程式碼風格 (Style)

### YAML 可讀性
- [ ] job 名稱是否清楚描述用途與環境？（P2）
- [ ] 縮排、list 格式、key 排列是否一致？（P2）
- [ ] 是否混用 `only/except` 與 `rules`，降低可讀性？（P2）
- [ ] 隱藏模板 job 是否使用 dot prefix（如 `.build_template`）？（P3 if missing）

### Script 風格
- [ ] `script:` 是否過長，應抽成 repo 內腳本檔案？（P2）
- [ ] 是否直接使用難懂的一行 shell command，缺少註解或拆分？
- [ ] environment、stage、variables 的命名是否一致？

---

## 5. 可維護性 (Maintainability)

### 重用與模組化
- [ ] 是否有重複 job 區塊可改用 `extends:`、YAML anchors 或 component template？（P2）
- [ ] 共用變數是否集中在 `variables:` 或模板層，而不是每個 job 各自複製？（P2）
- [ ] deploy/test/build 規則是否可共用 base template？

### 文件與治理
- [ ] 關鍵 pipeline（deploy、release、rollback）是否缺少註解或使用說明？（P2）
- [ ] 是否明確區分 dev/staging/production environment？（P2）
- [ ] 是否有 rollback / stop environment job？（P2/P3）
- [ ] 是否使用 `resource_group` 避免競態部署？（P1/P2 依風險）

---

## 6. 相容性 (Compatibility)

### GitLab / Runner 差異
- [ ] 是否使用只在特定 GitLab 版本支援的語法（如 `rules:changes:compare_to`、components）但未標示前提？（P2）
- [ ] Runner executor（shell、docker、kubernetes）假設是否與 job 指令相容？（P1/P2）
- [ ] Windows Runner 是否誤用 Bash-only 指令？Linux Runner 是否誤用 PowerShell-only 指令？（P1）
- [ ] `services:`、Docker-in-Docker、volume mount 是否符合 Runner 能力？

### 環境與部署目標
- [ ] image 內是否缺少必要工具，造成 job 在不同 Runner 表現不一致？（P1/P2）
- [ ] production/staging 使用的變數、URL、憑證是否正確隔離？（P1）
- [ ] 是否依賴 local path、host binary 或特定 shell，導致跨環境失敗？

---

## 快速評分標準

| Pipeline 狀態 | 建議 |
|--------------|------|
| 0 個 P0、0 個 P1 | ✅ 可合併 |
| 0 個 P0、有 P1 | ⚠️ 修復 P1 後合併 |
| 有任何 P0 | ❌ 必須修復 P0 後才能合併 |
