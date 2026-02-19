# AI 长期代理协议

一个用于AI代理在多个上下文窗口间工作的协议。灵感来源于有效的软件工程实践。

---

## 核心问题

AI代理在多个上下文窗口间工作面临两个关键失败模式：

1. **一次性做太多** - 代理试图在一次会话中完成所有事情，留下半实现的功能和未记录的进度
2. **过早宣布完成** - 代理在未经验证的情况下宣布项目完成

---

## 双代理解决方案

| 代理类型 | 角色 |
|----------|------|
| **初始化代理** | 首次会话：搭建环境、创建功能列表、进行初始 commit |
| **编码代理** | 后续会话：处理一个功能，为下一会话留下干净状态 |

---

## 阶段一：初始化代理

如果 `feature_list.json` 不存在，你是**初始化代理**。

### 你的任务

1. **分析提示词** 并创建完整的 `feature_list.json`：
```json
{
  "project": "项目名称",
  "total_features": 0,
  "features": [
    {
      "id": "F001",
      "category": "functional|ui|api|auth|etc",
      "description": "该功能的详细描述",
      "steps": [
        "Step 1: 具体验证操作",
        "Step 2: 另一个验证步骤",
        "Step 3: 预期结果"
      ],
      "priority": "high|medium|low",
      "passes": false
    }
  ]
}
```

**重要**：所有功能必须以 `passes: false` 开始。此文件使用 JSON 格式，因为相比 Markdown，模型更不容易意外损坏 JSON 文件。

2. **创建 `init.sh`**（Windows 系统用 `init.bat`）：
   - 启动开发服务器的脚本
   - 包含任何必要的设置命令

3. **创建 `claude-progress.txt`**：
```
# Project Progress
## Status: INITIALIZING
## Last Updated: YYYY-MM-DD HH:MM
## Completed Features: 0 / [total]
## Current Focus: Setup
## Last Commit: [commit hash]
## Notes: Initial environment setup complete
## Next Steps: Begin feature implementation
```

4. **进行初始 git commit**：
```
git add .
git commit -m "Initial project setup with feature list"
```

5. **停止** - 不要开始实现。让下一会话重新开始。

---

## 阶段二：编码代理

如果 `feature_list.json` 存在，你是**编码代理**。

### 会话开始序列

每次会话开始时必须遵循以下序列：

```
1. pwd                    # 确认工作目录
2. 读取 claude-progress.txt
3. 读取 feature_list.json
4. git log --oneline -20  # 了解最近的工作
5. 运行 init.sh/init.bat   # 启动开发服务器
6. 测试基础功能 # 验证环境是否正常
7. 在开始新工作前修复发现的任何 bug
```

### 会话工作流程

#### 步骤 1：选择一个功能
- 选择优先级最高的未完成功能（`passes: false`）
- 整个会话只处理这一个功能

#### 步骤 2：实现
- 为选定的功能编写代码
- 保持更改专注和最小化

#### 步骤 3：端到端测试验证
**必须**：你必须验证功能端到端工作正常：
- 对 Web 应用使用浏览器自动化工具（Puppeteer、Playwright）
- 对后端功能使用真实 API 调用
- 运行实际的用户类交互，不仅仅是代码审查
- 不要仅依赖 `curl` 或单元测试

#### 步骤 4：干净状态提交
会话结束前，确保：
- 代码能工作（没有损坏状态）
- 没有半实现的功能
- 代码已文档化和组织
- 适合合并到 main 分支

然后：
```
git add .
git commit -m "[Feature ID] 实现的功能描述"
```

#### 步骤 5：更新进度文件

更新 `claude-progress.txt`：
```
# Project Progress
## Status: IN_PROGRESS
## Last Updated: YYYY-MM-DD HH:MM
## Completed Features: [n] / [total]
## Current Focus: [刚完成的功能 id]
## Last Commit: [commit hash]
## Notes: [本次会话完成的工作]
## Next Steps: [建议下一个要处理的功能]
```

更新 `feature_list.json`：
- **只更改 `passes: true`** 用于已完成的功能
- **永远不要删除或修改** 功能描述或测试步骤

---

## 关键规则

### 禁止操作

| 禁止 | 原因 |
|------|------|
| 从列表中删除功能 | 导致功能丢失和 bug |
| 修改测试步骤 | 破坏验证标准 |
| 同时处理多个功能 | 导致半实现状态 |
| 跳过端到端测试 | 遗漏真实世界的 bug |
| 留下损坏的代码 | 下一会话浪费时间修复 |

### 必须操作

| 必须 | 原因 |
|------|------|
| 首先读取进度文件 | 了解当前状态 |
| 开始新工作前测试 | 尽早发现 bug |
| 每次会话一个功能 | 保持专注和质量 |
| 端到端验证 | 确保功能真正工作 |
| 以干净状态提交 | 支持轻松回滚 |
| 更新两个进度文件 | 上下文连续性 |

---

## 失败模式与解决方案

| 问题 | 初始化代理 | 编码代理 |
|------|------------|----------|
| 过早宣布项目完成 | 创建包含所有功能的完整功能列表 | 首先读取功能列表；选择未完成的功能 |
| 留下损坏/未记录的状态 | 创建初始 git 仓库和进度文件 | 读取进度 + git log；测试基础功能；提交并更新 |
| 未经测试就标记功能完成 | 不适用 | 在标记 `passes: true` 前用 E2E 测试验证 |
| 浪费时间弄清楚如何运行应用 | 编写 `init.sh` 脚本 | 读取 `init.sh` 并运行 |

---

## 干净状态定义

"干净状态"意味着代码：
- 没有重大 bug 能正常工作
- 组织良好且有文档
- 适合合并到 main 分支
- 下一个开发者可以直接开始新功能而无需清理

---

## Quick Reference Card

```
╔══════════════════════════════════════════════════════════════╗
║                    SESSION START                              ║
║  1. pwd                                                       ║
║  2. Read claude-progress.txt                                  ║
║  3. Read feature_list.json                                    ║
║  4. git log --oneline -20                                     ║
║  5. Run init.sh/init.bat                                      ║
║  6. Test basic functionality                                  ║
║  7. Fix bugs BEFORE new work                                  ║
║  8. Pick ONE feature (highest priority, passes: false)        ║
║  9. Implement feature                                         ║
║  10. E2E test verification                                    ║
║  11. git commit (clean state)                                 ║
║  12. Update claude-progress.txt                               ║
║  13. Update feature_list.json (passes: true only)             ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Example Session

```
[Assistant] I'll start by getting my bearings and understanding the current state.

[Tool Use] <bash - pwd>

[Tool Use] <read - claude-progress.txt>

[Tool Use] <read - feature_list.json>

[Assistant] Let me check the git log to see recent work.

[Tool Use] <bash - git log --oneline -20>

[Assistant] Now let me check if there's an init.sh script to restart the servers.

[Tool Use] <bash - ./init.sh>

[Assistant] Excellent! Now let me verify that fundamental features are still working.

<Tests basic functionality>

[Assistant] Based on verification, the core features are working correctly. 
Let me review the feature list to find the next priority feature to implement.

<Selects F005 from feature_list.json>

<Implements feature F005>

<Runs E2E tests to verify>

[Assistant] Feature F005 is working. Let me commit and update progress.

[Tool Use] <bash - git add . && git commit -m "F005: Add user authentication">

[Tool Use] <write - claude-progress.txt>

[Tool Use] <write - feature_list.json - update passes: true for F005>

[Assistant] Session complete. F005 implemented and verified. 
Next suggested feature: F006 (User profile page).
```

---

## 致谢

基于 Anthropic 关于长期运行 AI 代理有效约束的研究。
