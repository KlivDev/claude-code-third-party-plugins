---
allowed-tools: Read, Write, AskUserQuestion, Bash, Grep, Glob
description: 搜索并继续之前保存的会话
---

智能搜索已保存的会话，支持按项目、时间、关键词过滤，选择后生成 Markdown 格式的会话摘要供在新会话中继续工作。

**核心特性**:
- 🔍 **智能搜索**: 支持按项目、时间范围、关键词搜索
- 📋 **可视化列表**: 清晰展示会话信息（标题、时间、进度）
- 📝 **完整导出**: 生成包含进度、方案、决策的 Markdown 文档
- 💡 **行动建议**: 自动生成后续工作建议

---

## 执行步骤

### 1. 扫描已保存的会话

a. 检查存储目录是否存在（项目本地）
   ```bash
   SUMMARY_BASE="./.claude-sessions/summaries"

   if [ ! -d "$SUMMARY_BASE" ]; then
       echo "❌ 错误：未找到已保存的会话"
       echo "提示：使用 /session-manager:save 保存当前会话"
       echo "📁 会话将保存在项目目录: ./.claude-sessions/"
       exit 1
   fi
   ```

b. 查找所有摘要文件（项目本地）
   ```bash
   # 查找当前项目的所有会话摘要
   find "$SUMMARY_BASE" -maxdepth 2 -name "summary.json" -type f
   ```

c. 读取并解析所有摘要文件
   - 使用 Read 工具读取每个 summary.json
   - 提取关键信息用于展示：
     * 会话标题
     * 项目名称
     * 保存时间
     * 进度信息（已完成/总数）
     * 会话 ID 和存储路径

d. 按保存时间排序（最新的在前）
   ```bash
   # 使用 Python 脚本读取和排序
   python3 << 'EOF'
   import json
   import glob
   import os
   from datetime import datetime

   summary_files = glob.glob(os.path.expanduser("~/.claude/session-summary/**/summary.json"), recursive=True)

   sessions = []
   for file_path in summary_files:
       with open(file_path, 'r', encoding='utf-8') as f:
           data = json.load(f)
           data['_file_path'] = file_path
           sessions.append(data)

   # 按 savedAt 时间排序
   sessions.sort(key=lambda x: x.get('savedAt', ''), reverse=True)

   # 输出 JSON
   print(json.dumps(sessions, ensure_ascii=False))
   EOF
   ```

### 2. 首次展示（最近 10 个）

a. 提取最近 10 个会话信息

b. 格式化展示选项
   ```
   每个选项格式：
   label: "[时间] 项目名 - 会话标题"
   description: "进度：X/Y 已完成 | 文件变更：Z 个"
   ```

c. 使用 AskUserQuestion 展示列表
   ```yaml
   question: "选择要继续的会话（显示最近 10 个）："
   header: "会话列表"
   multiSelect: false
   options:
     - label: "🔍 搜索和过滤"
       description: "按项目、时间或关键词搜索会话"

     - label: "[2025-11-03 16:00] claude-code-init - 添加会话管理插件"
       description: "进度：4/6 已完成 | 文件变更：3 个"

     - label: "[2025-11-03 14:30] my-project - 实现用户认证"
       description: "进度：5/8 已完成 | 文件变更：12 个"

     ... (最多 10 个)

     - label: "📚 查看更多..."
       description: "显示更多会话（第 11-20 个）"
   ```

### 3. 搜索和过滤功能

#### 如果用户选择"搜索和过滤"

a. 提供搜索选项
   ```yaml
   使用 AskUserQuestion:
     question: "选择搜索条件："
     header: "搜索选项"
     multiSelect: false
     options:
       - label: "按项目过滤"
         description: "只显示特定项目的会话"

       - label: "按时间范围"
         description: "选择时间范围（今天、本周、本月、自定义）"

       - label: "按关键词搜索"
         description: "在标题和描述中搜索关键词"

       - label: "返回会话列表"
         description: "返回主列表"
   ```

#### 场景 A: 按项目过滤

1. 提取所有唯一的项目名称
   ```python
   projects = list(set(s['projectName'] for s in sessions))
   projects.sort()
   ```

2. 展示项目列表供选择
   ```yaml
   question: "选择项目："
   header: "项目过滤"
   multiSelect: false
   options:
     - label: "项目名称 1"
       description: "{该项目的会话数} 个会话"
     - label: "项目名称 2"
       description: "{该项目的会话数} 个会话"
     ...
   ```

3. 过滤并展示该项目的会话列表

#### 场景 B: 按时间范围

1. 提供时间范围选项
   ```yaml
   question: "选择时间范围："
   header: "时间过滤"
   multiSelect: false
   options:
     - label: "今天"
       description: "只显示今天保存的会话"

     - label: "本周"
       description: "显示本周保存的会话"

     - label: "本月"
       description: "显示本月保存的会话"

     - label: "最近 7 天"
       description: "显示最近 7 天的会话"

     - label: "最近 30 天"
       description: "显示最近 30 天的会话"
   ```

2. 根据选择过滤会话
   ```python
   from datetime import datetime, timedelta

   now = datetime.now()

   # 根据选择计算时间范围
   if choice == "今天":
       start = now.replace(hour=0, minute=0, second=0)
   elif choice == "本周":
       start = now - timedelta(days=now.weekday())
   elif choice == "本月":
       start = now.replace(day=1, hour=0, minute=0, second=0)
   # ...

   # 过滤会话
   filtered = [s for s in sessions if datetime.fromisoformat(s['savedAt']) >= start]
   ```

3. 展示过滤后的会话列表

#### 场景 C: 按关键词搜索

1. 询问用户输入关键词
   ```yaml
   question: "请输入搜索关键词："
   header: "关键词搜索"
   multiSelect: false
   options:
     - label: "输入关键词"
       description: "在标题、描述、关键词中搜索"
   ```

2. 从用户输入中提取关键词（使用 Other 选项）

3. 在会话数据中搜索
   ```python
   keyword = user_input.lower()

   filtered = []
   for session in sessions:
       # 搜索标题
       if keyword in session['overview']['title'].lower():
           filtered.append(session)
           continue

       # 搜索描述
       if keyword in session['overview'].get('description', '').lower():
           filtered.append(session)
           continue

       # 搜索关键词列表
       if any(keyword in kw.lower() for kw in session['overview'].get('keywords', [])):
           filtered.append(session)
           continue
   ```

4. 展示搜索结果
   - 如果找到 0 个：提示未找到，返回主列表
   - 如果找到 1-10 个：直接展示
   - 如果找到 >10 个：展示前 10 个 + "查看更多"

### 4. 用户选择会话

#### 提取会话数据

1. 根据用户选择，定位到对应的 summary.json

2. 读取完整的摘要数据
   ```bash
   # 读取 summary.json
   Read tool: {SESSION_DIR}/summary.json
   ```

3. 可选：读取详细日志（如果需要提取更多上下文）
   ```bash
   # 读取 details.jsonl（可选）
   Read tool: {SESSION_DIR}/details.jsonl
   ```

### 5. 生成导出文件

#### 创建导出目录（项目本地）
```bash
EXPORT_DIR="./.claude-sessions/exports"
mkdir -p "$EXPORT_DIR"

# 生成文件名
TIMESTAMP=$(date +"%Y-%m-%d-%H-%M")
EXPORT_FILE="$EXPORT_DIR/${TIMESTAMP}-${SESSION_UUID}.md"
```

#### 生成 Markdown 内容

使用以下模板生成导出文件：

```markdown
# 会话恢复：{会话标题}

> **会话 ID**: {uuid}
> **项目**: {项目名称} (`{项目路径}`)
> **会话时间**: {开始时间} - {结束时间}
> **保存时间**: {保存时间}
> **消息总数**: {消息数} ({用户消息数} 用户 + {AI消息数} 助手)

---

## 📋 工作概述

{从 summary.json 提取的描述}

**关键词**: {关键词1}, {关键词2}, {关键词3}

---

## 📈 项目进度

### ✅ 已完成任务 ({已完成数量})

{遍历 todos，筛选 status === "completed"}
- [x] {任务1内容}
- [x] {任务2内容}
...

### 🔄 进行中任务 ({进行中数量})

{遍历 todos，筛选 status === "in_progress"}
- [ ] {任务内容} - 🔄 {activeForm}

### 📋 待办任务 ({待办数量})

{遍历 todos，筛选 status === "pending"}
- [ ] {任务内容}

**进度总览**: {已完成}/{总数} 已完成 ({完成百分比}%)

---

## 📐 实施方案

{如果 implementation.hasDesignDoc === true}

**方案文档**: `{implementation.designDocPath}`

### 主要变更点

{遍历 implementation.mainChanges}
1. {变更点1}
2. {变更点2}
...

{如果没有方案文档}
_本会话未包含正式的实施方案文档_

---

## 💡 关键决策

{遍历 keyDecisions}

### 决策 {序号} - {格式化时间}

**问题**: {decision.question}

**选择**: {decision.answer}

{如果有多个选项，展示所有选项}
_可选项_:
- {选项1描述}
- {选项2描述}
...

---
{如果没有关键决策}
_本会话未记录关键决策_

---

## 📂 文件变更记录

{遍历 filesModified，按类型分组}

### ✨ 已创建文件 ({创建文件数量})

{筛选 operation === "created"}
- `{相对路径1}` - {格式化时间}
- `{相对路径2}` - {格式化时间}
...

### 📝 已修改文件 ({修改文件数量})

{筛选 operation === "modified"}
- `{相对路径1}` - {格式化时间}
- `{相对路径2}` - {格式化时间}
...

### 🗑️ 已删除文件 ({删除文件数量})

{筛选 operation === "deleted"}
- `{相对路径1}` - {格式化时间}
...

{如果没有文件变更}
_本会话未修改任何文件_

---

## 🎯 后续行动建议

根据当前进度和待办任务，建议：

{自动生成建议，基于：}
1. 继续完成进行中的任务：
   {如果有 in_progress 任务}
   - {任务内容} - 当前状态：{activeForm}

2. 开始下一个待办任务：
   {取第一个 pending 任务}
   - {任务内容}

3. 验证已完成功能：
   {基于已完成的任务生成验证建议}
   - 测试 {相关功能}
   - 验证 {相关文件} 的正确性

4. 更新文档：
   {如果有文档相关的待办}
   - 完善 {文档名称}

{如果所有任务都已完成}
🎉 **恭喜！所有任务已完成！**

建议：
- 进行全面测试
- 更新项目文档
- 准备提交代码

---

## 📚 详细对话记录

完整的对话记录已保存在：

```
{SESSION_DIR}/details.jsonl
```

如需查看完整上下文，可以：
1. 使用文本编辑器打开上述文件
2. 使用 `cat` 命令查看：`cat {SESSION_DIR}/details.jsonl`
3. 使用 `jq` 格式化查看：`jq -r '.content' {SESSION_DIR}/details.jsonl`

---

<!--
═══════════════════════════════════════════════════════════════
🤖 FOR AI ASSISTANT ONLY - 仅供 AI 助手参考
═══════════════════════════════════════════════════════════════

本部分专门提供给 AI 助手，用户通常不需要查看此内容。

## 📁 完整对话数据源

**文件路径**: `{SESSION_DIR}/details.jsonl`

**数据格式**: JSONL (每行一个 JSON 对象)

**包含内容**:
- 所有用户消息 (type: "user")
- 所有 AI 响应 (type: "assistant")
- 所有工具调用 (type: "tool_use")
- 时间戳和元数据

## 🎯 何时读取完整对话

**建议读取的场景**:
1. ✅ 用户要求了解详细的讨论过程
2. ✅ 需要理解某个决策的完整背景
3. ✅ 任务涉及之前讨论的具体技术细节
4. ✅ 当前摘要信息不足以完成任务
5. ✅ 需要验证之前的某个结论或约定

**无需读取的场景**:
1. ❌ 只需要了解当前进度状态
2. ❌ 只需要知道下一步做什么
3. ❌ 摘要信息已经足够清晰
4. ❌ 任务是全新的，不依赖历史细节

## 📖 如何读取

使用 Read 工具读取完整文件：

```
Read tool: {SESSION_DIR}/details.jsonl
```

**注意事项**:
- 文件可能较大 (50KB - 10MB)，建议只在必要时读取
- 可以使用 Grep 工具搜索特定内容而非读取全部
- 如果只需要某类消息，可以用 Python 过滤

## 💡 智能决策建议

**第一步**: 评估摘要信息是否充足
- 如果摘要中的进度、决策、文件变更信息足够 → 直接使用摘要
- 如果用户询问"为什么这样决定"、"当时怎么讨论的" → 读取完整对话

**第二步**: 按需读取
- 如果决定读取，使用 Read 工具读取 details.jsonl
- 可以结合 Grep 搜索关键词，避免读取全部

**第三步**: 向用户解释
- 如果读取了完整对话，告知用户："我查看了完整的对话记录..."
- 这样用户知道你的决策基于完整信息

═══════════════════════════════════════════════════════════════
END OF AI ASSISTANT SECTION
═══════════════════════════════════════════════════════════════
-->

---

## 🔗 如何继续工作

### 方式 1：粘贴到新会话（推荐）

1. 复制上述内容（从"工作概述"到"后续行动建议"）
2. 在新 Claude Code 会话中粘贴
3. 添加你的新请求，例如：
   ```
   继续之前的工作，{描述你要做什么}
   ```

### 方式 2：引用导出文件

在新会话中输入：
```
@./.claude-sessions/exports/{timestamp}-{uuid}.md

继续之前的工作，{描述你要做什么}
```

### 方式 3：手动选择性恢复

只复制你需要的部分：
- 如果只需要恢复进度：复制"项目进度"部分
- 如果需要方案细节：复制"实施方案"部分
- 如果需要决策上下文：复制"关键决策"部分

---

**导出时间**: {当前时间}
**导出工具**: session-manager v1.0.0

🤖 Generated by session-manager plugin
```

#### 写入导出文件

```bash
# 使用 Write 工具写入
Write tool: $EXPORT_FILE
```

### 6. 展示结果

输出格式化的成功信息：

```
✅ 会话已成功导出！

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 导出信息
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
会话标题: {标题}
项目名称: {项目名}
会话时间: {开始时间} - {结束时间}
进度状态: {已完成}/{总数} 已完成

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📁 文件位置（项目本地）
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
导出文件: ./.claude-sessions/exports/{timestamp}-{uuid}.md
文件大小: {文件大小}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💡 如何继续工作
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

方式 1（推荐）: 在新会话中输入

  继续之前的工作：{会话标题}

  @~/.claude/session-exports/{timestamp}-{uuid}.md

  {描述你要继续做的具体工作}

方式 2: 打开导出文件并复制相关内容

  cat ~/.claude/session-exports/{timestamp}-{uuid}.md

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 后续建议行动
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{列出 2-3 个具体的后续行动建议}
1. {建议1}
2. {建议2}
3. {建议3}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🤖 Generated by session-manager plugin
```

---

## 错误处理

### 场景 1: 没有已保存的会话

```
❌ 错误：未找到已保存的会话

存储目录: ~/.claude/session-summary/

可能原因：
1. 还没有保存过任何会话
2. 存储目录被删除或移动
3. 权限问题导致无法访问

建议：
- 使用 /session-manager:save 保存当前会话
- 检查存储目录是否存在：ls -la ~/.claude/session-summary/
- 检查目录权限
```

### 场景 2: 摘要文件读取失败

```
❌ 错误：无法读取会话摘要文件

文件: {文件路径}
错误: {错误信息}

可能原因：
1. 文件已损坏
2. 文件权限问题
3. JSON 格式错误

建议：
- 尝试读取其他会话
- 检查文件完整性：cat {文件路径}
- 如果多个文件都无法读取，可能需要重新保存会话
```

### 场景 3: 搜索无结果

```
ℹ️ 未找到匹配的会话

搜索条件: {条件描述}

建议：
- 尝试使用不同的关键词
- 检查时间范围是否过窄
- 查看所有会话：选择"返回会话列表"
```

### 场景 4: 导出文件创建失败

```
❌ 错误：无法创建导出文件

目标文件: {文件路径}
错误: {错误信息}

可能原因：
1. 磁盘空间不足
2. 导出目录不存在或无写入权限
3. 文件名包含非法字符

建议：
- 检查磁盘空间：df -h
- 创建导出目录：mkdir -p ~/.claude/session-exports
- 检查目录权限：ls -ld ~/.claude/session-exports
```

---

## 交互流程示例

### 示例 1: 快速选择最近的会话

```
用户: /session-manager:continue

系统: 显示最近 10 个会话列表

用户: 选择第二个会话

系统: ✅ 会话已成功导出！
      导出文件: ~/.claude/session-exports/2025-11-03-16-30-abc123.md
```

### 示例 2: 按项目搜索

```
用户: /session-manager:continue

系统: 显示最近 10 个会话列表

用户: 选择"搜索和过滤"

系统: 显示搜索选项

用户: 选择"按项目过滤"

系统: 显示所有项目列表
      - my-app (3 个会话)
      - another-project (5 个会话)

用户: 选择"my-app"

系统: 显示 my-app 的 3 个会话

用户: 选择第一个

系统: ✅ 会话已成功导出！
```

### 示例 3: 按关键词搜索

```
用户: /session-manager:continue

系统: 显示最近 10 个会话列表

用户: 选择"搜索和过滤"

系统: 显示搜索选项

用户: 选择"按关键词搜索"

系统: 请输入搜索关键词

用户: 输入"认证"（通过 Other 选项）

系统: 找到 2 个匹配的会话
      - [2025-11-03] my-app - 实现用户认证功能
      - [2025-11-02] api-service - 添加 JWT 认证

用户: 选择第一个

系统: ✅ 会话已成功导出！
```

---

## 跨平台兼容性

### 路径处理
- ✅ 使用 `$HOME` 而非 `~`
- ✅ 使用绝对路径
- ✅ 使用 `find` 命令递归搜索（标准工具）

### JSON 处理
- ✅ 优先使用 Python 3（系统自带）
- ✅ 处理 UTF-8 编码
- ✅ 使用 `ensure_ascii=False` 保留中文

### 时间处理
- ✅ 解析 ISO 8601 格式
- ✅ 格式化为本地可读格式
- ✅ 计算时间范围（今天、本周等）

### 文件操作
- ✅ 使用 `mkdir -p` 创建目录
- ✅ 使用 `date` 生成时间戳
- ✅ 使用 `cat` 展示文件内容

---

## 性能优化

### 会话列表加载
- 只读取 summary.json（不读取 details.jsonl）
- 默认只展示最近 10 个（减少交互负担）
- 按需加载更多（"查看更多"选项）

### 搜索性能
- 在内存中进行搜索（Python 脚本）
- 不使用磁盘索引（保持简单）
- 对于大量会话（>100），可能需要 2-3 秒

### 导出文件生成
- 一次性生成完整内容
- 不进行额外的文件读取
- 使用模板字符串拼接

---

## 使用技巧

### 最佳实践

1. **定期保存会话**
   - 完成重要阶段后立即保存
   - 结束工作前保存当天进度

2. **使用清晰的标题**
   - 第一条消息描述清楚要做什么
   - 便于后续搜索和识别

3. **利用搜索功能**
   - 项目较多时，先按项目过滤
   - 记不清时间时，使用关键词搜索

4. **选择性恢复**
   - 不一定要恢复全部内容
   - 只复制需要的部分到新会话

5. **保持存储整洁**
   - 定期清理旧会话（手动）
   - 备份重要会话

### 高级用法

**多项目管理**:
```bash
# 在不同项目间快速切换
cd ~/project-a
/session-manager:continue  # 恢复 project-a 的工作

cd ~/project-b
/session-manager:continue  # 恢复 project-b 的工作
```

**长期项目跟踪**:
```bash
# 每周五保存
/session-manager:save

# 下周一恢复，按时间过滤"本周"
/session-manager:continue
→ 选择"按时间范围" → "本周"
```

**任务切换**:
```bash
# 紧急任务插入时
/session-manager:save  # 保存当前任务

# 完成紧急任务后
/session-manager:continue  # 恢复之前的任务
```

---

## 注意事项

- ⚠️ 搜索只在摘要数据中进行（不搜索详细日志）
- ⚠️ 导出文件不会自动清理，需要手动管理
- ⚠️ 大量会话（>50）时，列表加载可能稍慢
- ⚠️ 导出文件可能包含敏感信息，注意安全

---

## 未来增强

以下功能计划在未来版本中添加：

- [ ] 会话标签功能
- [ ] 会话笔记添加
- [ ] 自动清理旧会话
- [ ] 导出格式自定义
- [ ] 会话对比功能
- [ ] 团队共享支持

---

🤖 Generated with Claude Code
