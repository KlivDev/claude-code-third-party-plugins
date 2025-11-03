---
allowed-tools: Read, Write, AskUserQuestion, Bash(date:*), Glob, Grep
description: 智能分析项目并生成 CLAUDE.md 规范文件（完全跨平台）
---

智能分析现有项目结构和配置文件,自动推断技术栈、架构模式等信息,**只对无法确定的信息进行询问**,最终生成项目的 CLAUDE.md 开发规范文件。

**核心特性**:
- 🔍 **智能分析**: 自动检测项目类型、语言、框架
- 💡 **按需询问**: 只对无法推断的信息询问用户
- 🎯 **数据优先**: 优先使用项目实际数据,避免猜测
- 🌐 **跨平台**: 完全兼容 Windows / macOS / Linux

---

## 内置模板

以下是用于生成 CLAUDE.md 的内置模板：

```template
# CLAUDE.md 模板

> 用于在新项目中快速建立开发规范和最佳实践。使用时请填充 `[占位符]` 部分。

---

## 项目概述

**[项目名称]** - [项目一句话描述]

### 核心技术栈
- **框架**: [框架名称和版本]
- **数据存储**: [数据库/缓存技术]
- **开发语言**: [语言和版本]
- **代码风格**: [命名规范]

### 项目架构
```
[绘制项目架构图]
例如：
API层 → Service层 → Logic层 → Data层 → Storage
```

**关键模块**：
- **[模块1]**: [路径] - [说明]
- **[模块2]**: [路径] - [说明]

---

## 测试规范
- 测试覆盖率 > 80%
- 使用 Mock 模拟外部依赖
- Mock 数据存放在 `test/data/` 目录
- 测试文件存放在 `test/` 目录

---

## 架构规则（强制）

### 层职责定义
- **[Layer 1]**: ✅ [应该做的] / ❌ [不应该做的]
- **[Layer 2]**: ✅ [应该做的] / ❌ [不应该做的]

### 开发规则
1. **命名规范** - [统一命名规范]
2. **依赖管理** - [依赖引入规则]
3. **错误处理** - [统一错误处理机制]
4. **版本控制** - Git提交必须使用 AskUserQuestion 工具询问用户同意
5. **禁止行为** - [明确禁止的操作]

### Claude Code 工具使用规范

**AI 必须使用的工具**：
- **AskUserQuestion**: 数据不足或需要决策时，必须主动询问用户
- **TodoWrite**: 跟踪多步骤任务的进度，及时更新任务状态
- **Read/Grep/Glob**: 修改前必须先分析现有代码，避免重复定义

---

## 核心开发实践（强制）

### 1. 代码修改前的分析（强制）

**所有代码修改前，必须先完成深度分析，理解项目架构后再设计方案。**

#### 调用链路分析要点
```
HTTP请求 → 路由 → Controller → Service → Logic → Data → Storage
```

必须分析：
- **入口点**: 请求从哪个API端点或界面进入
- **数据流转**: 数据在各层之间如何传递和转换
- **依赖关系**: 各组件之间的依赖
- **错误传播**: 错误如何在调用链中传播
- **缓存策略**: 缓存的读写时机
- **事务边界**: 数据一致性保证点

#### 方案设计原则：KISS

设计前必须回答4个问题：
1. **"这是真问题还是臆想的？"** - 拒绝过度设计
2. **"有更简单的方法吗？"** - 永远寻找最简方案
3. **"会破坏什么吗？"** - 向后兼容是铁律
4. **"真的需要这个功能吗？"** - 确认功能必要性

**性能要求**：
- 最小化数据库/缓存操作次数
- 优先使用批量操作
- 合理使用并发和异步
- 避免不必要的数据复制

**代码简洁性**：
- 能用30行解决，绝不写300行
- 复用现有代码
- 函数职责单一

#### 数据驱动的方案设计（强制）

**所有方案必须有充足的数据支撑**：

数据充足性判断：
- ✅ 是否完整理解现有代码实现逻辑？
- ✅ 是否掌握性能瓶颈数据？
- ✅ 是否了解用户实际使用场景？
- ✅ 是否分析业界最佳实践？
- ✅ 是否评估不同方案优劣？

**数据不足时的处理**：
```
⚠️ 我没有足够的信心设计最佳方案

需要以下数据：
1. [具体需要的数据1]
2. [具体需要的数据2]
...

请提供这些数据或告诉我如何获取。
```

**禁止**：
- ❌ 基于猜测或假设进行方案设计
- ❌ 数据不足时强行给出方案
- ❌ 使用"可能"、"也许"等不确定词汇描述关键决策

#### 方案审批流程（强制）

**在实施任何代码修改前，必须先获得用户审批**：

**工作流程**：
1. **分析阶段**：AI 使用只读工具（Read/Grep/Glob）深度分析代码
2. **方案设计**：制定详细的实施方案（见下方"方案文档要求"）
3. **展示方案**：向用户清晰展示方案，等待批准
4. **执行修改**：获得批准后才开始实际修改代码

**方案文档要求**：
1. **提交方案设计文档**，包含：
   - 完整调用链路分析
   - 问题诊断和根因分析
   - **支撑数据和分析**（必须包含）
   - 方案设计（架构图、时序图、代码片段）
   - KISS原则4问题的回答
   - 性能影响评估
   - 风险分析和缓解措施
   - **文档保存**: 保存到 `docs/todo/YYYY-MM-DD-HH-MM-功能名称-方案.md`

2. **等待用户确认** - 批准后方可开始实施

3. **实施** - 严格按照批准方案执行

#### 分析检查清单

开始编码前必须完成：
- [ ] 已完整追踪调用链路
- [ ] 已理解所有涉及层的职责
- [ ] 已识别所有数据转换点
- [ ] 已回答KISS原则4个问题
- [ ] 已收集充足数据支撑方案
- [ ] 已制定详细实施方案
- [ ] 已获得用户审批确认

### 2. 功能版本迭代限制

**禁止版本迭代**: 对于存量功能，严格禁止版本迭代（v1、v2、v3）

**核心原则**：
- **就地更新**: 所有改进必须直接修改现有代码
- **用户主导**: 只有用户明确要求才能创建新版本
- **单一实现**: 每个功能只能有一个实现版本
- **向后兼容**: 必须保持向后兼容
- **清理废弃**: 必须版本迭代时同时清理旧代码

### 3. 任务解析与影响分析

**任务前置分析原则**: 执行任何任务前，必须先进行全面分析

#### 文件影响评估
- **主要修改文件**: 直接需要编辑的核心文件
- **关联影响文件**: 可能受影响需同步更新的文件
- **测试相关文件**: 需要更新或新增的测试文件
- **配置文件**: 可能需要调整的配置

#### 依赖关系分析
- **文件依赖**: 识别导入/引用关系
- **功能依赖**: 识别模块间调用关系

#### 重复定义检查

添加任何新定义前，必须先检查：
```bash
# 检查函数/类/接口定义
grep -r "functionName\|className" --include="*.[ext]"
```

**原则**: 避免重复实现，优先复用现有代码

### 4. 前置条件信任原则

**避免重复验证**: 信任调用链的前置条件

```[语言]
// ❌ 过度防御：重复验证已验证数据
function process(data) {
    if (!data) return  // 前层已验证
    if (!data.field) return  // 前层已验证
    // ...
}

// ✅ 信任前置条件
function process(data) {
    // 前置条件: 调用方已验证
    const value = data.field
    // 业务逻辑...
}
```

**分层职责**：
- **入口层**: 负责验证和防护
- **业务层**: 信任前置条件，专注业务
- **数据层**: 处理持久化，不做业务验证

---

## 交互规范（强制）

### 沟通原则
- **中文响应**: 所有响应使用中文
- **详细说明**: 提供清晰的操作步骤和说明
- **操作确认**: 重要操作前进行确认
- **透明度**: 如实汇报进度、问题和风险

### 主动提问机制（强制）
- **数据不足时提问**: 缺乏必要信息时，必须使用 `AskUserQuestion` 工具询问用户
- **明确性优先**: 遇到模糊需求时，通过提问明确用户意图
- **必须提问的场景**:
  - 技术方案有多种实现路径需权衡时
  - 需要删除或修改重要功能/数据时
  - 配置参数会显著影响系统行为时
  - 不确定用户具体需求时
  - 设计决策可能影响架构或性能时
  - 存在向后兼容性风险时
- **提问质量要求**:
  - 问题应具体、聚焦
  - 提供2-4个清晰选项
  - 每个选项附带详细说明
  - 说明各选项的影响和优缺点
- **等待确认**: 提问后必须等待用户回答

### 文件创建限制（强制）
- **禁止主动创建文档**: 不主动创建文档文件(*.md)或README，除非用户明确要求
- **优先编辑**: 优先编辑现有文件而非创建新文件
- **必要性原则**: 只创建绝对必要的文件
- **用户确认**: 创建新文件前必须征得用户同意

---

**最后更新**: [日期]
```

---

## 执行步骤

### 1. 前置检查

a. 检查当前目录是否已存在 CLAUDE.md 文件
   - 使用 Read 工具尝试读取 ./CLAUDE.md
   - 如果文件存在，使用 AskUserQuestion 询问用户是否要覆盖：
     - 选项 1: "备份后覆盖"（推荐） - 将现有文件备份为 CLAUDE.md.backup-{timestamp}
     - 选项 2: "直接覆盖" - 直接覆盖现有文件
     - 选项 3: "取消操作" - 退出命令
   - 如果用户选择取消，立即终止执行并输出提示信息

### 2. 项目分析（智能推断）

**核心原则**: 数据驱动，避免猜测。通过分析项目文件自动推断信息，减少用户输入负担。

#### a. 检测项目类型

使用 Read 工具检查项目配置文件：

**Node.js/TypeScript 项目检测**:
```bash
检测文件: package.json
提取信息:
  - 项目名称: name 字段
  - 项目描述: description 字段
  - 语言: 检查 devDependencies 中是否有 typescript
  - 框架: 从 dependencies 推断
    - "next" → Next.js
    - "react" → React
    - "vue" → Vue
    - "express" → Express
    - "nestjs" → NestJS
  - 测试框架: 从 devDependencies 推断 (jest, vitest, mocha 等)
  - 代码风格: 检查 eslintConfig, prettier 配置
```

**Go 项目检测**:
```bash
检测文件: go.mod
提取信息:
  - 项目名称: module 声明
  - 语言版本: go 版本行
  - 框架: 从 require 推断
    - "gin-gonic/gin" → Gin
    - "labstack/echo" → Echo
    - "gofiber/fiber" → Fiber

检测文件: go.sum (确认依赖)
```

**Python 项目检测**:
```bash
检测文件: pyproject.toml (优先) 或 requirements.txt 或 setup.py
提取信息:
  - 项目名称: [project].name 或从目录名推断
  - 语言版本: requires-python 字段
  - 框架: 从依赖推断
    - "fastapi" → FastAPI
    - "django" → Django
    - "flask" → Flask
  - 测试框架: pytest, unittest 等
```

**Java 项目检测**:
```bash
检测文件: pom.xml 或 build.gradle
提取信息:
  - 项目名称: artifactId 或 name
  - 语言版本: Java version
  - 框架: 从依赖推断
    - "spring-boot" → Spring Boot
    - "micronaut" → Micronaut
    - "quarkus" → Quarkus
```

#### b. 分析项目结构

使用 Glob 工具列举项目目录，推断架构模式：

```bash
# 检测顶层目录
检测路径:
  - src/, lib/, pkg/ (源代码目录)
  - test/, tests/, __tests__/ (测试目录)
  - docs/ (文档目录)

架构模式推断:
  - 存在 src/controllers, src/services, src/models → 分层架构（MVC）
  - 存在 src/api, src/service, src/logic, src/data → 分层架构（四层）
  - 存在 packages/, services/, apps/ → 微服务架构
  - 存在 functions/, lambda/ → Serverless
  - 否则 → 单体应用
```

#### c. 分析代码风格

使用 Read 工具读取几个代码文件样本：

```bash
# 读取 2-3 个代码文件
# 分析变量命名模式

命名规范检测:
  - 检测变量/函数名中的模式
  - camelCase: 首字母小写 (userName, getUserData)
  - PascalCase: 首字母大写 (UserName, GetUserData)
  - snake_case: 下划线连接 (user_name, get_user_data)

Lint 配置检测:
  - .eslintrc.* (JavaScript/TypeScript)
  - .golangci.yml (Go)
  - .flake8, pyproject.toml (Python)
  - checkstyle.xml (Java)
```

#### d. 分析数据存储

使用 Grep 工具搜索数据库相关导入和配置：

```bash
# 搜索数据库导入语句
关键词:
  - "mysql", "pg", "postgres" → PostgreSQL/MySQL
  - "mongodb", "mongo" → MongoDB
  - "redis" → Redis
  - "elasticsearch", "es" → Elasticsearch

# 搜索配置文件
检查:
  - docker-compose.yml (服务定义)
  - .env.example (环境变量示例)
  - config/ 目录下的配置文件
```

#### e. 检测测试配置

```bash
检测测试目录:
  - test/, tests/, __tests__/ 是否存在
  - 测试文件数量和覆盖情况

检测测试配置:
  - jest.config.js, vitest.config.ts (JS/TS)
  - go.mod 中的测试依赖
  - pytest.ini, tox.ini (Python)
  - pom.xml 中的测试配置 (Java)

推断测试要求:
  - 存在完善测试配置 + 多个测试文件 → 建议 >80%
  - 存在基础测试 → 建议 >60%
  - 无测试 → 不强制要求
```

#### f. 记录推断结果

创建推断结果数据结构,记录每项信息的值和置信度:

```javascript
分析结果 = {
  projectName: {
    value: "从配置文件读取的名称",
    source: "package.json",
    confidence: "high"  // high: 确定 | medium: 较确定 | low: 需确认 | none: 未检测到
  },
  projectDescription: {
    value: "从配置文件读取的描述",
    source: "package.json",
    confidence: "medium"  // 描述可能不准确,建议用户确认
  },
  language: {
    value: "TypeScript 5.0+",
    source: "package.json devDependencies",
    confidence: "high"
  },
  framework: {
    value: "Next.js",
    source: "package.json dependencies",
    confidence: "high"
  },
  storage: {
    value: ["PostgreSQL", "Redis"],
    source: "grep 搜索和 docker-compose.yml",
    confidence: "medium"
  },
  architecture: {
    value: "分层架构",
    source: "目录结构分析",
    confidence: "medium"
  },
  namingStyle: {
    value: "驼峰命名（camelCase）",
    source: "代码文件分析",
    confidence: "high"
  },
  testCoverage: {
    value: ">80%",
    source: "测试配置和文件检测",
    confidence: "medium"
  },
  gitApproval: {
    value: null,  // 这个需要用户决策
    source: null,
    confidence: "none"
  }
}
```

#### g. 输出分析结果

输出分析摘要供用户查看:

```
🔍 项目分析完成！

📊 自动检测结果:
✅ 项目类型: TypeScript 项目 (来源: package.json)
✅ 项目名称: my-app (来源: package.json)
✅ 开发语言: TypeScript 5.0+ (来源: package.json)
✅ 主要框架: Next.js (来源: dependencies)
✅ 数据存储: PostgreSQL, Redis (来源: docker-compose.yml)
✅ 架构模式: 分层架构 (来源: 目录结构)
✅ 命名规范: 驼峰命名 (来源: 代码分析)
⚠️  测试要求: 建议 >80% (来源: 测试配置)

❓ 需要您确认的信息:
- 项目描述 (自动检测可能不准确)
- Git 提交流程 (需要您的决策)
```

### 3. 智能信息收集

根据项目分析结果,**只对置信度低或无法推断的信息进行询问**。

#### 询问策略:

**A. confidence = "high" 的信息**:
- ✅ 直接使用,不询问
- 在最终确认时展示给用户

**B. confidence = "medium" 的信息**:
- ⚠️ 作为推荐选项,但仍询问用户确认
- 在选项中标注 "(已检测)"

**C. confidence = "low" 或 "none" 的信息**:
- ❌ 必须询问用户
- 提供常见选项供选择

#### 询问示例:

**场景 1: 项目名称 confidence="high"**
```
不询问,直接使用检测到的值
```

**场景 2: 项目描述 confidence="medium"**
```yaml
使用 AskUserQuestion:
  question: "请确认项目描述："
  header: "项目描述"
  multiSelect: false
  options:
    - label: "使用检测到的描述 (已检测)"
      description: "Web application for user management"
    - label: "Web 应用后端服务"
      description: "提供 RESTful API 的后端服务"
    - Other: 自定义描述
```

**场景 3: 数据存储 confidence="medium"**
```yaml
使用 AskUserQuestion:
  question: "请确认项目使用的数据存储技术（可多选）："
  header: "数据存储"
  multiSelect: true
  options:
    - label: "PostgreSQL (已检测)"
      description: "关系型数据库"
    - label: "Redis (已检测)"
      description: "缓存系统"
    - label: "MySQL"
      description: "关系型数据库"
    - Other: 其他存储技术
```

**场景 4: Git 提交流程 confidence="none"**
```yaml
使用 AskUserQuestion:
  question: "Git 提交是否需要审批确认？"
  header: "提交规范"
  multiSelect: false
  options:
    - label: "需要（使用 AskUserQuestion）"
      description: "提交前必须询问用户确认"
    - label: "不需要"
      description: "允许直接提交"
```

#### 分批询问流程:

根据实际需要询问的问题数量,灵活分批:

**如果需要询问 0-2 个问题**: 一次性询问
**如果需要询问 3-5 个问题**: 分 2 批询问
**如果需要询问 6+ 个问题**: 分 3-4 批询问

**优先级排序**:
1. 高优先级: 项目描述、架构模式 (影响模板填充)
2. 中优先级: 数据存储、命名规范 (可能影响规范细节)
3. 低优先级: 测试要求、Git 提交规范 (配置项)

### 4. 生成 CLAUDE.md

根据分析结果和用户确认的信息，执行以下步骤：

a. 从"内置模板"部分提取模板内容（```template 和 ``` 之间的内容）

b. 创建变量映射表并进行字符串替换：
   - `[项目名称]` → 分析结果中的项目名称（或用户输入）
   - `[项目一句话描述]` → 分析结果中的项目描述（或用户输入）
   - `[框架名称和版本]` → 分析结果中的框架（或用户选择）
   - `[数据库/缓存技术]` → 分析结果中的数据存储（多个则用逗号连接）
   - `[语言和版本]` → 分析结果中的开发语言（或用户选择）
   - `[命名规范]` → 分析结果中的代码风格（或用户选择）

c. 添加生成时间戳：
   - 使用 Bash date 命令获取当前日期
   - 替换模板最后的 `[日期]` 为实际日期（格式：YYYY-MM-DD）

d. 根据分析结果和用户配置，调整对应章节：
   - 如果测试覆盖率选择"不强制要求"，修改"测试规范"章节，删除覆盖率要求
   - 如果 Git 提交选择"不需要"审批，修改"版本控制"规则，移除 AskUserQuestion 要求

### 5. 文件写入

a. 如果用户在步骤 1 中选择了"备份后覆盖"：
   - 使用 Bash date 命令生成时间戳（格式：YYYYMMDDHHmmss）
   - 使用 Read 工具读取现有的 CLAUDE.md
   - 使用 Write 工具将内容写入 CLAUDE.md.backup-{timestamp}
   - 输出备份成功信息

b. 使用 Write 工具将生成的内容写入 ./CLAUDE.md 文件

### 6. 确认完成

输出成功信息，展示完整的配置信息：

```
✅ 项目规范文件生成成功！

📄 文件位置: ./CLAUDE.md
📅 生成时间: {当前时间}

📊 项目配置摘要:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ 项目名称: {项目名称}
✅ 项目描述: {项目描述}
✅ 开发语言: {语言和版本}
✅ 主要框架: {框架}
✅ 数据存储: {数据存储技术}
✅ 架构模式: {架构模式}
✅ 命名规范: {命名规范}
✅ 测试要求: {测试覆盖率}
✅ Git 提交: {是否需要审批}

🎯 自动检测项数: {检测成功的项数}/{总项数}

💡 下一步操作:
1. 查看生成的 CLAUDE.md 文件
2. 根据项目实际情况调整细节
3. 提交到版本控制系统

🤖 Generated with Claude Code
```

---

## 跨平台兼容性说明

此命令已优化为完全跨平台兼容：

### ✅ 跨平台特性

1. **内置模板** - 不需要读取外部文件，避免路径问题
2. **Claude Code 工具** - 仅使用跨平台的内置工具
   - Read/Write: 文件读写
   - Glob: 文件查找（跨平台模式匹配）
   - Grep: 内容搜索（跨平台正则表达式）
   - AskUserQuestion: 用户交互
   - Bash date: 时间戳生成（Windows/Mac/Linux 均支持）
3. **最小依赖** - 仅依赖标准工具，无需额外安装
4. **相对路径** - 使用 `./CLAUDE.md` 相对路径，适配所有系统

### 🖥️ 系统支持

- ✅ **Windows**: 完全支持（包括 CMD 和 PowerShell）
- ✅ **macOS**: 完全支持
- ✅ **Linux**: 完全支持
- ✅ **WSL**: 完全支持

### 📝 核心改进

1. **智能分析优先**：
   - 自动检测项目配置文件（package.json, go.mod, pyproject.toml 等）
   - 智能推断技术栈、架构模式、代码风格
   - 减少用户输入负担，提升体验

2. **数据驱动决策**：
   - 基于实际项目数据进行推断
   - 避免猜测和假设
   - 置信度评估确保准确性

3. **按需询问**：
   - 只对无法确定的信息询问用户
   - 提供检测到的值作为推荐选项
   - 灵活的分批询问策略

4. **错误处理**：
   - 文件读写失败时，提供清晰的错误信息
   - 权限不足时，建议用户检查目录权限
   - 用户输入验证（避免空值）

5. **灵活性**：
   - 支持空项目（使用传统询问模式）
   - 支持已有项目（智能分析模式）
   - 允许用户覆盖自动检测的结果

6. **向后兼容**：
   - 备份机制确保不会丢失现有配置
   - 提供清晰的操作提示和确认

7. **遵循 KISS 原则**：
   - 不进行过度验证
   - 不添加不必要的功能
   - 专注于核心任务：分析项目 → 收集信息 → 生成文件

### 🎯 最佳实践

**对于新项目（空目录）**:
- 命令会检测到没有配置文件
- 自动切换到询问模式
- 引导用户完成所有配置

**对于已有项目**:
- 命令会自动分析项目文件
- 提取尽可能多的信息
- 只对不确定的信息进行询问
- 展示分析结果供用户确认

**最佳工作流程**:
1. 在项目根目录运行命令
2. 查看自动分析结果
3. 确认或修改推断的信息
4. 生成规范文件
5. 根据实际需求微调细节
