---
name: gitlab-ci-reviewer
description: "Review GitLab CI/CD pipeline configurations for security, reliability, performance, and maintainability. Categorizes findings by severity P0 (Critical), P1 (High), P2 (Medium), and P3 (Low). USE FOR: reviewing .gitlab-ci.yml files, include templates, deploy jobs, runner settings, rules/needs/cache/artifacts configuration, and release pipelines. TRIGGERS: review gitlab ci, audit gitlab pipeline, check .gitlab-ci.yml, gitlab ci code review."
argument-hint: "Paste GitLab CI/CD YAML directly, or provide one or more file paths to review"
---

# GitLab CI/CD Reviewer

## 語言規範

> **所有回覆必須使用正體中文**。專業術語（如 `rules`、`needs`、`artifacts`、`cache`、`Runner`、`protected variables`、`OWASP` 類別等）保留英文原文。

## 用途說明

對輸入的 GitLab CI/CD 設定（例如 `.gitlab-ci.yml`、拆分的 `include` YAML、相關 deploy script）進行全面審查，依嚴重等級 P0–P3 分類問題，並產出結構化報告、優先修復建議與可直接套用的修正片段。

## When to Use

- 審查 `.gitlab-ci.yml`、`*.gitlab-ci.yml`、`include` 進來的 pipeline 模板
- 稽核 `image`、`services`、`before_script`、`script`、`rules`、`workflow`、`needs`、`artifacts`、`cache`
- 檢查 release / deploy job 的安全性、授權流程與 rollback 風險
- 在 Merge Request 前評估 pipeline 的可靠性、執行效率與維護性

---

## Review Procedure

### Step 1 — 接收設定內容

接受以下任一形式的輸入：

- 使用者直接貼上的 GitLab CI/CD YAML 片段
- 工作區內的檔案路徑（使用 `read_file` 讀取）
- 多個檔案路徑（平行讀取所有相關 YAML / script）

若設定使用 `include:`、YAML anchors、`extends:` 或引用外部 template，但內容未提供，需在報告中明確標示 **假設條件與未知風險**。

### Step 2 — 分析階段

依序檢查以下六個面向，並收集潛在問題：

1. **安全性 (Security)** — secrets、token 暴露、未受保護的 deploy job、未釘選 image、危險 shell 指令、第三方 template 信任邊界
2. **錯誤處理與 Pipeline 正確性 (Error Handling & Reliability)** — `rules` / `workflow` 邏輯、失敗傳遞、`allow_failure`、`retry`、`timeout`、依賴順序、deploy guardrail
3. **效能 (Performance)** — 重複 pipeline、cache key 設計、artifact 體積、clone 深度、`needs` / parallel 化、冗餘 stage
4. **程式碼風格 (Style)** — job 命名、YAML 一致性、縮排、hidden job / template 風格、key 排列與可讀性
5. **可維護性 (Maintainability)** — 重複 job、缺少 `extends` / anchors、環境變數集中化、說明註解、模組化程度
6. **相容性 (Compatibility)** — GitLab 版本功能差異、Runner executor 差異、Linux/Windows shell 差異、Docker / Kubernetes 環境假設

詳細檢查項目請參閱 [review-checklist.md](./references/review-checklist.md)。

### Step 3 — 嚴重等級分類

依據 [severity-guide.md](./references/severity-guide.md) 將每個問題分配到對應等級：

| 等級   | 名稱     | 定義                                                                         |
| ------ | -------- | ---------------------------------------------------------------------------- |
| **P0** | Critical | 安全漏洞、機密外洩、未受控生產部署、供應鏈高風險 — **必須立即修復**          |
| **P1** | High     | Pipeline 邏輯錯誤、錯誤處理缺失、部署風險、重大效能浪費 — **應在合併前修復** |
| **P2** | Medium   | 可讀性、維護性、次要配置缺陷 — **建議修復**                                  |
| **P3** | Low      | 改善建議、最佳實踐提示、可選優化 — **參考即可**                              |

### Step 4 — 產出報告

以下列格式輸出完整報告：

---

## 📋 審查報告 — `<檔案名稱或 Pipeline 摘要>`

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
  - **位置**: Job `<name>` / top-level key `<name>` / 第 X 行
  - **說明**: ＜問題的具體描述＞
  - **風險**: ＜若不修復會發生什麼事＞
  - **修復建議**:
    ```yaml
    # 建議的修正片段
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
- **摘要說明**: ＜2-3 句總結 pipeline 整體狀況，並指出最高風險區塊＞

---

### Step 5 — 自動修復建議

針對 P0 與 P1 的所有問題，以及 P2 中有明確修正方式的問題，額外輸出可直接套用的修正片段：

```yaml
# [P1-1] 修正：限制 production deploy 只允許 protected branch 且需 manual approval
# 原始碼（有問題）：
deploy_prod:
  stage: deploy
  script:
    - ./deploy.sh production
  only:
    - branches

# 修正後：
deploy_prod:
  stage: deploy
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
      when: manual
    - when: never
  environment:
    name: production
  script:
    - set -euo pipefail
    - ./deploy.sh production
```

**格式規則**：

- 每個程式碼區塊前標示對應的問題編號（如 `[P0-1]`）
- 同時保留「原始碼（有問題）」與「修正後」，方便對照
- 若修正涉及多個 job，優先提供最小可套用差異
- 若問題與 `include` 或外部 template 有關，需標示 **需同步修改上游模板**

---

## Quality Completion Criteria

報告完成前確認：

- [ ] 所有回覆使用正體中文，專業術語保留英文
- [ ] 涵蓋全部六個分析面向
- [ ] 每個問題都標示了 job 名稱、top-level key 或行號
- [ ] 每個 P0/P1 問題都附有 YAML 或 shell 修復範例
- [ ] Step 5 已對所有 P0/P1 及可修復 P2 輸出自動修復片段
- [ ] 建議清單按嚴重等級排序
- [ ] 整體評分與合併建議已填寫
- [ ] 若存在未知 `include` / Runner 前提，已清楚列出假設與限制
