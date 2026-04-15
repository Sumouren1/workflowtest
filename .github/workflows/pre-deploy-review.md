---
description: |
  Pre-deployment code review workflow. Before any deployment is triggered,
  this workflow performs automated code analysis and requires explicit human
  approval via a PR comment before the deployment label is applied.

on:
  pull_request:
    types: [opened, synchronize, ready_for_review]
  workflow_dispatch:

permissions:
  contents: read
  issues: write
  pull-requests: write

network: defaults

tools:
  github:
    lockdown: false
    min-integrity: none

safe-outputs:
  mentions: false
  allowed-github-references: []
  add-label:
    allowed-labels:
      - "review: pending"
      - "review: approved"
      - "review: blocked"
  create-pr-comment: {}

human-in-the-loop:
  enabled: true
  approval-required: true
  timeout: 48h
  trigger-comment: "/approve-deploy"
  reject-comment: "/block-deploy"

source: githubnext/agentics/workflows/pre-deploy-review.md
---

# Pre-Deployment Code Review

在每次 PR 合并到主分支、触发部署之前，对本次变更进行全面代码审查，并**等待人工确认**后才能放行部署。

## 审查步骤

### Step 1：分析变更内容
- 列出本次 PR 所有变更文件
- 识别高风险改动类型：
  - 数据库 migration 文件
  - 环境变量 / secrets 配置文件（`*.env*`）
  - 依赖版本变更（`package.json`、`requirements.txt`、`go.mod` 等）
  - CI/CD pipeline 文件（`.github/` 目录）
  - 权限与认证相关代码（auth、rbac、middleware）
  - 对外 API 接口变更

### Step 2：代码质量检查
- 检查是否存在明显 bug 或逻辑错误
- 检查是否有安全隐患（硬编码密钥、SQL 注入、未校验输入）
- 评估测试覆盖情况（是否有对应单测或 E2E 测试）
- 确认变更是否符合项目既有代码规范

### Step 3：生成结构化审查报告
在 PR 中发布一条评论，格式如下：

```
## 🔍 Pre-Deploy Code Review Report

**风险等级：** 低 / 中 / 高

### ✅ 通过项
- （列出审查通过的内容）

### ⚠️ 需关注项（不阻塞部署，建议后续修复）
- （列出需关注但不阻塞的内容）

### ❌ 阻塞项（必须修复后才可部署）
- （列出必须修复的问题，如有）

---
**如需批准部署，请审批人在评论中回复：** `/approve-deploy`
**如需阻塞部署，请回复：** `/block-deploy reason=<原因>`
```

### Step 4：根据审查结果打标签
- 如果存在"阻塞项" → 打上 `review: blocked` 标签，**停止流程，等待修复**
- 如果无阻塞项 → 打上 `review: pending` 标签，等待人工批准

### Step 5：等待人工确认
- 监听 PR 评论
- 当检测到授权审批人发出 `/approve-deploy` 评论时：
  - 将标签从 `review: pending` 更新为 `review: approved`
  - 在 PR 中回复确认信息，包含：审批人、审批时间
  - **此时部署流程可以继续**
- 当检测到 `/block-deploy` 评论时：
  - 将标签更新为 `review: blocked`
  - 记录阻塞原因

## 部署放行条件

**只有同时满足以下条件，才允许触发部署：**
1. PR 带有 `review: approved` 标签
2. 所有 CI 检查通过
3. 无未解决的阻塞项评论

## 备注

- 审批超时时间：48 小时（超时后需重新触发审查）
- 每次新的 commit push 后，审查状态自动重置为 `review: pending`
- 审查报告和审批记录将保留在 PR 评论历史中，便于追溯
