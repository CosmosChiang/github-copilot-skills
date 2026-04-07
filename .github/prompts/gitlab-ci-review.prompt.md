---
description: "Trigger a structured GitLab CI/CD review with P0-P3 severity grading, auto-fix suggestions, and a prioritized recommendation list. All output in Traditional Chinese."
---

使用 `gitlab-ci-reviewer` skill 對以下 GitLab CI/CD 設定進行完整審查。

**輸入內容**：

```yaml
${input:貼上你的 .gitlab-ci.yml 內容，或輸入檔案路徑}
```

**審查要求**：

1. 依序分析六個面向：Security、Error Handling & Reliability、Performance、Style、Maintainability、Compatibility
2. 每個問題分配 P0 / P1 / P2 / P3 嚴重等級
3. 輸出完整審查報告（摘要表格、分級問題列表、建議清單、整體評分）
4. 針對所有 P0、P1 及可修復的 P2 問題，額外輸出可直接套用的 YAML / shell 修正片段
5. **所有回覆使用正體中文，專業術語（如 `rules`、`needs`、`artifacts`、`cache`、`Runner`、OWASP 類別等）保留英文**
6. 若設定使用 `include:`、外部 template 或 repo 內 deploy script，請先說明你採用的假設與分析邊界
