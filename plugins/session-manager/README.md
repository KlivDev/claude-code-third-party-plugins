# Session Manager Plugin

> **会话管理插件** - 智能保存和恢复 Claude Code 会话，支持进度跟踪和工作连续性

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/your-repo/session-manager)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

---

## 📋 功能概述

Session Manager 是一个强大的 Claude Code 插件，解决了会话临时性和工作连续性问题。它提供了两个核心命令，让您可以轻松保存和恢复工作进度。

### 核心特性

- 🔄 **会话保存** - 智能总结并保存当前会话的所有关键信息
- 🔍 **智能搜索** - 支持按项目、时间、关键词搜索已保存的会话
- 📊 **进度跟踪** - 自动提取 TodoWrite 记录，清晰展示工作进度
- 💡 **决策记录** - 捕获所有 AskUserQuestion 交互，保留关键决策
- 📂 **文件追踪** - 记录所有文件变更，便于回顾
- 📝 **方案管理** - 关联实施方案文档，保持技术方案完整性
- 🌐 **跨平台** - 完全兼容 Windows / macOS / Linux
- 💾 **分层存储** - 摘要 + 详细日志分离，兼顾查询效率和数据完整性
- 🤖 **智能数据访问** - AI 自主决定是否读取完整对话，用户只看摘要

### 💡 智能设计

**双层信息架构**：
- **用户视角**：导出文件只显示精炼的摘要信息（进度、决策、文件变更）
- **AI 视角**：导出文件中嵌入隐藏指引，告知 AI 完整对话的位置
- **按需加载**：AI 根据任务复杂度自主决定是否读取完整对话记录

**项目本地存储**：
- 📁 **完全跨平台**：使用相对路径 `./.claude-sessions/`，Windows/Linux/macOS 通用
- 📦 **项目便携性**：会话数据跟随项目，可打包、归档、克隆
- 🤝 **团队协作**：可选择性提交到 Git，共享决策历史
- 🗂️ **项目隔离**：每个项目独立存储，自动隔离

**优势**：
- ✅ 用户体验：文件简洁易读，不被冗长对话淹没
- ✅ 数据完整：完整对话永久保存在 details.jsonl
- ✅ 智能决策：AI 在需要时自动获取详细上下文
- ✅ 灵活高效：避免不必要的大文件加载
- ✅ 跨平台兼容：无需特殊处理，完全通用

---

## 🚀 快速开始

### 安装

1. 克隆或下载此插件到 Claude Code 插件目录：
   ```bash
   cd ~/.claude/plugins
   git clone https://github.com/your-repo/session-manager.git
   ```

2. 重启 Claude Code 或重新加载插件配置

3. 验证安装：
   ```bash
   # 在 Claude Code 中输入
   /session-manager:save
   ```

### 基本使用

#### 保存当前会话

```bash
/session-manager:save
```

**输出示例**:
```
✅ 会话已成功保存！

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 保存位置（项目本地）
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
存储目录: ./.claude-sessions/summaries/abc-123/
  ├── summary.json   (2.3 KB)
  └── details.jsonl  (156 KB)

📊 会话信息
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
会话标题: 实现用户认证功能
项目名称: my-app
进度: 5/8 已完成

💡 使用 /session-manager:continue 在新会话中继续工作
```

#### 继续之前的工作

```bash
/session-manager:continue
```

系统会展示已保存的会话列表，您可以：
- 📋 直接选择最近的会话
- 🔍 搜索和过滤会话
- 📅 按时间范围筛选
- 🔎 按关键词搜索

选择后，系统会生成 Markdown 格式的会话摘要，包含：
- 工作概述
- 项目进度（任务列表）
- 实施方案要点
- 关键决策记录
- 文件变更列表
- 后续行动建议

---

## 📖 详细使用指南

### 命令说明

#### `/session-manager:save` - 保存会话

**功能**: 总结并保存当前会话到本地存储

**执行流程**:
1. 定位当前会话文件 (`~/.claude/projects/{project}/{session-uuid}.jsonl`)
2. 解析 JSONL 数据，提取关键信息：
   - 用户消息和 AI 响应
   - TodoWrite 记录（进度）
   - AskUserQuestion 记录（决策）
   - 文件操作记录
3. 生成结构化摘要
4. 保存到分层存储：
   - `summary.json` - 快速查询用的摘要
   - `details.jsonl` - 完整对话记录

**存储位置**:
```
项目根目录/
├── .claude-sessions/
│   └── summaries/
│       └── {session-uuid}/
│           ├── summary.json       # 摘要索引
│           └── details.jsonl      # 完整对话备份
└── ...
```

**最佳实践**:
- 🕐 完成重要阶段后立即保存
- 📝 第一条消息清楚描述要做什么
- 📋 使用 TodoWrite 跟踪任务
- 💬 重要决策使用 AskUserQuestion

---

#### `/session-manager:continue` - 继续工作

**功能**: 搜索并恢复之前保存的会话

**执行流程**:
1. 扫描所有已保存的会话
2. 展示最近 10 个会话列表
3. 提供搜索和过滤选项：
   - 按项目过滤
   - 按时间范围（今天、本周、本月）
   - 按关键词搜索
4. 用户选择后，生成 Markdown 导出文件
5. 保存到 `~/.claude/session-exports/`

**导出文件内容**:
```markdown
# 会话恢复：{标题}

> 会话信息、时间、进度等元数据

## 工作概述
{会话内容概述}

## 项目进度
- [x] 已完成任务
- [ ] 待办任务

## 实施方案
{方案文档要点}

## 关键决策
{用户做出的重要选择}

## 文件变更
{创建/修改/删除的文件}

## 后续行动建议
{根据进度生成的建议}

## 详细对话记录
完整对话保存在：{路径}/details.jsonl
(包含 AI 可读的隐藏指引)
```

**智能数据访问机制**:

导出文件中包含一个 HTML 注释部分（用户不可见），专门指引 AI：
- 📁 完整对话文件的位置
- 🎯 何时应该读取完整对话
- 📖 如何读取和使用

AI 会根据以下场景自主决策：

**AI 会读取完整对话**：
- ✅ 用户询问"当时为什么这样决定？"
- ✅ 任务涉及之前讨论的具体技术细节
- ✅ 需要理解某个决策的完整背景
- ✅ 摘要信息不足以完成当前任务

**AI 不会读取完整对话**：
- ❌ 只需要了解当前进度
- ❌ 只需要知道下一步做什么
- ❌ 摘要信息已经足够清晰
- ❌ 任务是全新的，不依赖历史

**使用导出文件**:

方式 1（推荐）:
```
在新会话中输入：

继续之前的工作：{标题}

@~/.claude/session-exports/2025-11-03-16-30-abc123.md

{描述你要继续做的具体工作}
```

方式 2:
```
打开导出文件，复制需要的部分粘贴到新会话
```

---

## 🎯 使用场景

### 场景 1: 长期项目管理

```bash
# 第一天
开始项目，使用 TodoWrite 规划任务
/session-manager:save

# 第二天
/session-manager:continue
→ 选择昨天的会话
→ 在新会话中继续工作
```

### 场景 2: 多项目切换

```bash
# 项目 A
cd ~/project-a
工作一段时间...
/session-manager:save

# 切换到项目 B
cd ~/project-b
工作一段时间...
/session-manager:save

# 回到项目 A
cd ~/project-a
/session-manager:continue
→ 选择"按项目过滤" → "project-a"
→ 恢复项目 A 的工作
```

### 场景 3: Token 限制处理

```bash
# 当前会话接近 token 限制
/session-manager:save

# 开启新会话
/session-manager:continue
→ 选择刚才的会话
→ 在新会话中继续，重置 token 计数
```

### 场景 4: 紧急任务插入

```bash
# 正在做任务 A
工作中...
/session-manager:save

# 紧急任务 B 来了
处理任务 B...

# 完成任务 B，回到任务 A
/session-manager:continue
→ 搜索任务 A 的会话
→ 恢复上下文继续
```

---

## 🔧 技术实现

### 数据格式

#### summary.json 结构

```json
{
  "version": "1.0",
  "sessionId": "abc-123-def-456",
  "projectPath": "/Users/user/project",
  "projectName": "my-app",
  "savedAt": "2025-11-03T16:00:00Z",
  "startTime": "2025-11-03T14:00:00Z",
  "endTime": "2025-11-03T16:00:00Z",
  "messageCount": 45,
  "userMessageCount": 15,
  "assistantMessageCount": 30,

  "overview": {
    "title": "实现用户认证功能",
    "description": "添加 JWT 认证，实现登录、注册、权限验证",
    "keywords": ["认证", "JWT", "登录"]
  },

  "progress": {
    "todos": [
      {
        "content": "设计认证架构",
        "status": "completed",
        "activeForm": "设计认证架构中"
      }
    ],
    "completedCount": 5,
    "totalCount": 8
  },

  "keyDecisions": [
    {
      "timestamp": "2025-11-03T15:00:00Z",
      "question": "选择认证方案",
      "answer": "JWT"
    }
  ],

  "filesModified": [
    {
      "path": "src/auth/jwt.ts",
      "operation": "created",
      "timestamp": "2025-11-03T15:30:00Z"
    }
  ],

  "implementation": {
    "hasDesignDoc": true,
    "designDocPath": "docs/todo/2025-11-03-认证功能-方案.md",
    "mainChanges": [
      "添加 JWT 中间件",
      "实现登录 API",
      "添加权限验证"
    ]
  }
}
```

#### details.jsonl 结构

标准的 Claude Code 会话 JSONL 格式，每行一个 JSON 对象：

```jsonl
{"type":"user","content":"实现用户认证","timestamp":"..."}
{"type":"assistant","content":"我来帮你实现","timestamp":"..."}
{"type":"tool_use","name":"Write","input":{...},"timestamp":"..."}
...
```

---

## ⚙️ 配置和定制

### 存储路径

默认存储路径：
- 会话摘要：`~/.claude/session-summary/`
- 导出文件：`~/.claude/session-exports/`

如需修改，可以编辑命令文件中的路径变量。

### 搜索选项

默认展示最近 10 个会话，可以在 `continue-session.md` 中修改：

```markdown
# 修改此值以改变默认展示数量
DEFAULT_DISPLAY_COUNT = 10
```

---

## 🛠️ 故障排查

### 问题 1: 找不到会话文件

**错误信息**:
```
❌ 错误：找不到当前会话文件
```

**可能原因**:
- 当前不在有效的 Claude Code 会话中
- 项目路径编码错误

**解决方案**:
1. 确保在 Claude Code 会话中执行命令
2. 检查 `~/.claude/projects/` 目录是否存在
3. 尝试在新会话中重试

---

### 问题 2: 存储目录创建失败

**错误信息**:
```
❌ 错误：无法创建存储目录
```

**可能原因**:
- 磁盘空间不足
- 权限问题

**解决方案**:
1. 检查磁盘空间：`df -h`
2. 检查目录权限：`ls -ld ~/.claude`
3. 手动创建目录：`mkdir -p ~/.claude/session-summary`

---

### 问题 3: 搜索无结果

**提示信息**:
```
ℹ️ 未找到匹配的会话
```

**可能原因**:
- 关键词不匹配
- 时间范围过窄
- 没有保存过会话

**解决方案**:
1. 尝试不同的关键词
2. 扩大时间范围
3. 返回主列表查看所有会话

---

## 🔐 安全和隐私

### 数据安全

- ✅ 所有数据仅保存在本地（`~/.claude/` 目录）
- ✅ 不会上传到任何远程服务器
- ✅ 不会修改原始会话数据

### 隐私注意事项

⚠️ **重要提醒**:

保存的会话可能包含敏感信息：
- API 密钥和令牌
- 数据库连接字符串
- 密码和凭证
- 项目源代码

**建议**:
1. 定期清理旧会话
2. 不要共享导出文件
3. 备份时注意加密
4. 在公共设备上使用后及时清理

---

## 📊 性能和限制

### 性能表现

| 操作 | 数据量 | 性能 |
|------|--------|------|
| 保存会话 | < 1MB | < 1 秒 |
| 保存会话 | 1-10MB | 1-3 秒 |
| 搜索会话 | < 50 个 | < 1 秒 |
| 搜索会话 | 50-100 个 | 1-2 秒 |
| 导出文件 | 任意 | < 1 秒 |

### 已知限制

1. **大文件处理**
   - 超过 10MB 的会话可能需要更长时间
   - 建议分阶段保存

2. **搜索范围**
   - 仅在摘要数据中搜索
   - 不搜索详细日志内容

3. **存储空间**
   - 每个会话约占 100KB - 2MB
   - 建议定期清理

---

## 🧪 测试

运行测试脚本验证功能：

```bash
# 运行保存命令测试
bash test/session-manager/test-save.sh

# 运行继续命令测试
bash test/session-manager/test-continue.sh
```

查看测试报告：
```
docs/test/2025-11-03-会话管理插件测试.md
```

---

## 🤝 贡献

欢迎贡献代码、报告问题或提出建议！

### 开发流程

1. Fork 本仓库
2. 创建功能分支：`git checkout -b feature/amazing-feature`
3. 提交更改：`git commit -m 'Add amazing feature'`
4. 推送分支：`git push origin feature/amazing-feature`
5. 提交 Pull Request

### 报告问题

在 [Issues](https://github.com/your-repo/session-manager/issues) 页面报告问题。

提供以下信息：
- Claude Code 版本
- 操作系统
- 错误信息
- 复现步骤

---

## 📝 更新日志

### v1.0.0 (2025-11-03)

**首次发布**

- ✨ 实现会话保存功能
- ✨ 实现会话恢复功能
- ✨ 支持智能搜索和过滤
- ✨ 分层存储架构
- ✨ 跨平台支持
- 📚 完整文档和示例

---

## 📄 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件

---

## 💬 联系方式

- **作者**: Wang Xuecheng
- **邮箱**: ahut17353766123@gmail.com
- **项目**: [claude-code-third-party-plugins](https://github.com/your-repo)

---

## 🙏 致谢

感谢以下资源和项目的启发：
- [Claude Code](https://claude.com/code) - 官方 CLI
- Claude Code 社区的反馈和建议

---

**🤖 Generated with Claude Code**
