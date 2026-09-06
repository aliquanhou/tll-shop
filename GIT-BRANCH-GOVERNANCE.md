# TLL Shop Git Branch Governance

TLL Shop 仓库的 Git 分支治理规范。

## 分支体系

### main（稳定基线）
- **用途**：保存已验证、已验收、可回滚、可作为稳定基线的代码
- **准入条件**：
  1. develop 分支完整 CI 通过
  2. 三平台（Linux/macOS/Windows）编译通过
  3. 核心功能冒烟测试通过
  4. 总指挥验收通过
  5. Evidence 完整
- **禁止**：直接在 main 上施工、直接 push 到 main（除非紧急修复并经过审批）

### develop（施工主分支）
- **用途**：TLL Shop 开发的主分支，所有功能开发、Bug 修复先在这里集成
- **包含**：
  - 新功能开发
  - Bug 修复
  - 安全加固
  - 性能优化
  - 文档更新
- **规则**：
  - 所有施工必须先在 develop 或 feature 分支进行
  - develop 必须保持可编译状态
  - 重大改动前先创建 feature 分支

### feature/*（功能分支）
- **用途**：重大功能开发、底层改造、安全加固等隔离施工
- **命名规范**：
  - `feature/session-csprng`（Session ID 替换为 CSPRNG）
  - `feature/csrf-hmac`（CSRF 重新设计）
  - `feature/sql-parameterization`（SQL 参数化）
  - `feature/payment-redesign`（支付重新设计）
  - `feature/inventory`（库存管理）
- **规则**：
  - 从 develop 分支创建
  - 完成后合并回 develop
  - 合并前必须通过 CI 和验收

### fix/*（修复分支）
- **用途**：紧急 Bug 修复
- **命名规范**：`fix/issue-description`
- **规则**：
  - 从 main 或 develop 创建（根据严重程度）
  - 修复后同时合并到 develop 和 main（如果是生产环境 Bug）

## 工作流程

```
feature/*
    ↓ (开发完成 + CI 通过)
develop
    ↓ (完整回归 + 验收)
main
    ↓ (Release)
生产环境
```

### 功能开发流程
1. 从 develop 创建 feature 分支
2. 在 feature 分支开发
3. 本地测试通过
4. 推送 feature 分支
5. CI 通过
6. 合并到 develop
7. develop 完整 CI 通过
8. 验收通过
9. 合并到 main

### Bug 修复流程
1. 确认 Bug 影响范围
2. 从对应分支创建 fix 分支
3. 修复 Bug
4. 编写回归测试
5. CI 通过
6. 合并到 develop
7. 如果是生产环境 Bug，同时合并到 main

## Commit 规范

### 格式
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type
- `feat`: 新功能
- `fix`: Bug 修复
- `security`: 安全加固
- `refactor`: 重构
- `perf`: 性能优化
- `docs`: 文档
- `test`: 测试
- `ci`: CI/CD
- `chore`: 构建/工具/依赖
- `migrate`: 仓库迁移/分离

### 示例
```
feat(user-center): 补全用户中心缺失的6张数据库表

- coupons (优惠券定义表)
- user_coupons (用户优惠券表)
- user_collections (用户收藏表)
- point_logs (积分日志表)
- account_logs (账户余额日志表)
- cs_messages (客服消息表)

之前 us_ensureTables 只创建了5张表，导致优惠券、收藏、
积分明细、余额明细、联系客服等页面查询不存在的表而报错。

编译验证: main.tll 编译成功 (567 functions, 10003 constants)
```

## 回滚流程

### main 回滚
1. 确认需要回滚的 Commit
2. 创建回滚分支：`fix/rollback-<commit>`
3. 使用 `git revert` 或 `git reset` 回滚
4. 测试通过
5. 合并到 main
6. 记录回滚原因和影响

### develop 回滚
1. 确认需要回滚的 Commit
2. 直接在 develop 上 `git revert`
3. 推送
4. 通知团队

## 安全锚点

### 永久标签
- `v1.0.0-initial-migration`: 初始迁移基线（从 tllos 分离）
- 后续每个正式 Release 打标签

### 标签规则
- 正式版本：`v<major>.<minor>.<patch>`
- 预发布：`v<major>.<minor>.<patch>-rc<N>`
- 安全锚点：`v<version>-pre-<change>`
- 标签不可移动、不可删除

## 禁止事项

- ❌ 直接在 main 上施工
- ❌ 未经 CI 通过就合并到 main
- ❌ 跳过验收直接发布
- ❌ 强制推送（force push）到 main
- ❌ 删除已发布的标签
- ❌ 在 Commit 中包含密码、Token、密钥
- ❌ 将编译产物（*.tllbc）、日志（*.log）、数据库（*.db）提交到仓库

## 与 TLL OS 的关系

- TLL OS 仓库：`aliquanhou/tllos`（造工具）
- TLL Shop 仓库：`aliquanhou/tll-shop`（用工具造产品）
- 依赖方向：TLL Shop → TLL OS（单向）
- 当 Shop 发现 TLL 能力缺口时：
  1. 在 TLL OS 仓库修复/新增能力
  2. TLL OS 封板后更新 TLL Shop 的工具链版本
  3. 在 TLL Shop 中使用新能力
  4. 只做针对性验证，不重复全量回归
