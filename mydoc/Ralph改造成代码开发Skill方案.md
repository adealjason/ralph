# Ralph 改造成代码开发 Skill 方案

## 一、改造目标

将 Ralph 从**PRD 持续迭代工具**改造成**支持代码开发的 skill**，使其能够：
1. 在 IDE 中直接调用
2. 支持代码阅读、分析、修改
3. 保持 Ralph 的核心优势（迭代式执行、状态持久化）
4. 与现有 skill 系统兼容

---

## 二、核心改造思路

### 2.1 架构对比

| 维度 | 原 Ralph | 改造后 Skill |
|------|---------|-------------|
| 执行方式 | Bash 脚本循环 | Skill 触发 + 内部状态管理 |
| 状态存储 | 外部文件 (prd.json, progress.txt) | Skill 状态 + 外部文件 |
| 迭代控制 | 脚本循环 | Skill 内部状态机 |
| 工具调用 | 启动新 AI 实例 | 直接调用 MCP 工具 |
| 上下文 | 每次全新 | 可复用 + 增量更新 |

### 2.2 改造策略

```
┌─────────────────────────────────────────────────────────────┐
│                    改造策略                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 保留核心逻辑                                             │
│     └── 故事驱动、状态持久化、知识积累                       │
│                                                             │
│  2. 适配 Skill 框架                                           │
│     └── 触发条件、工具调用、状态管理                         │
│                                                             │
│  3. 增强代码能力                                             │
│     └── 代码阅读、修改、测试、验证                           │
│                                                             │
│  4. 优化交互体验                                             │
│     └── 进度展示、中断恢复、人工干预                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 三、详细设计方案

### 3.1 Skill 元数据

```yaml
---
name: ralph-dev
version: 1.0.0
description: "Autonomous code development using Ralph's iterative approach. Use when you need to implement features, refactor code, or complete multi-step development tasks. Triggers on: implement feature, develop this, build this feature, refactor code, complete this task."
user-invocable: true
---
```

### 3.2 核心模块设计

#### 3.2.1 状态管理模块

```typescript
// ralph-dev-state.ts
interface RalphDevState {
  // 任务定义
  task: {
    id: string;
    description: string;
    createdAt: string;
  };
  
  // 用户故事列表
  stories: Story[];
  
  // 当前执行状态
  currentStoryIndex: number;
  currentIteration: number;
  
  // 学习记录
  learnings: Learning[];
  
  // 执行历史
  history: ExecutionRecord[];
}

interface Story {
  id: string;
  title: string;
  description: string;
  acceptanceCriteria: string[];
  status: 'pending' | 'in_progress' | 'completed' | 'failed';
  attempts: number;
  notes: string;
}

interface Learning {
  timestamp: string;
  storyId: string;
  category: 'pattern' | 'gotcha' | 'context';
  content: string;
}
```

#### 3.2.2 执行引擎模块

```typescript
// ralph-dev-engine.ts
class RalphDevEngine {
  // 核心执行循环
  async executeStory(story: Story): Promise<ExecutionResult> {
    // 1. 读取相关代码
    const codeContext = await this.readCodeContext(story);
    
    // 2. 分析需求
    const plan = await this.analyzeStory(story, codeContext);
    
    // 3. 执行代码修改
    const changes = await this.makeChanges(plan);
    
    // 4. 验证结果
    const verification = await this.verifyChanges(story, changes);
    
    // 5. 记录学习
    await this.recordLearning(story, changes, verification);
    
    return {
      success: verification.passed,
      changes,
      learnings: verification.learnings
    };
  }
  
  // 代码阅读策略
  async readCodeContext(story: Story): Promise<CodeContext> {
    // 使用 code-reading-skill 的工具
    // - search_file_path: 查找相关文件
    // - get_single_file: 读取文件内容
    // - get_file_block: 读取特定行范围
    // - get_file_blame: 分析代码历史
  }
  
  // 代码修改策略
  async makeChanges(plan: ChangePlan): Promise<CodeChanges> {
    // 使用 IDE 编辑工具
    // - file_replace: 替换文件内容
    // - create_file: 创建新文件
    // - edit_file: 编辑文件
  }
  
  // 验证策略
  async verifyChanges(story: Story, changes: CodeChanges): Promise<VerificationResult> {
    // 1. 语法检查
    const lintResult = await this.runLint(changes);
    
    // 2. 类型检查
    const typecheckResult = await this.runTypecheck(changes);
    
    // 3. 测试执行
    const testResult = await this.runTests(changes);
    
    // 4. 功能验证
    const functionalResult = await this.verifyFunctionality(story);
    
    return {
      passed: lintResult.ok && typecheckResult.ok && testResult.ok && functionalResult.ok,
      learnings: this.extractLearnings(changes, story)
    };
  }
}
```

#### 3.2.3 工具集成模块

```typescript
// ralph-dev-tools.ts
interface RalphDevTools {
  // 代码阅读工具 (复用 code-reading-skill)
  codeReading: {
    searchFile: (query: string) => Promise<File[]>;
    readFile: (path: string) => Promise<string>;
    readBlock: (path: string, start: number, end: number) => Promise<string>;
    getBlame: (path: string) => Promise<BlameInfo[]>;
  };
  
  // 代码编辑工具
  codeEditing: {
    replaceFile: (path: string, old: string, new: string) => Promise<void>;
    createFile: (path: string, content: string) => Promise<void>;
    editFile: (path: string, edits: Edit[]) => Promise<void>;
  };
  
  // 验证工具
  verification: {
    runLint: (paths: string[]) => Promise<LintResult>;
    runTypecheck: (paths: string[]) => Promise<TypecheckResult>;
    runTests: (paths: string[]) => Promise<TestResult>;
  };
  
  // 状态管理工具
  stateManagement: {
    saveState: (state: RalphDevState) => Promise<void>;
    loadState: () => Promise<RalphDevState>;
    appendLearning: (learning: Learning) => Promise<void>;
  };
}
```

### 3.3 执行流程设计

```
┌─────────────────────────────────────────────────────────────┐
│                    Ralph Dev 执行流程                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐                                           │
│  │  用户触发     │                                           │
│  │  "implement  │                                           │
│  │  feature X"  │                                           │
│  └──────┬───────┘                                           │
│         │                                                   │
│         ▼                                                   │
│  ┌──────────────┐                                           │
│  │  需求分析     │                                           │
│  │  - 理解需求   │                                           │
│  │  - 拆分故事   │                                           │
│  │  - 生成计划   │                                           │
│  └──────┬───────┘                                           │
│         │                                                   │
│         ▼                                                   │
│  ┌──────────────┐                                           │
│  │  故事循环     │◄─────────────────────────────────────────┤
│  │              │                                           │
│  │  ┌──────────┐│                                           │
│  │  │ 选择故事  ││                                           │
│  │  └────┬─────┘│                                           │
│  │       │      │                                           │
│  │       ▼      │                                           │
│  │  ┌──────────┐│                                           │
│  │  │ 代码阅读  ││                                           │
│  │  │ 理解上下文│                                           │
│  │  └────┬─────┘│                                           │
│  │       │      │                                           │
│  │       ▼      │                                           │
│  │  ┌──────────┐│                                           │
│  │  │ 代码修改  ││                                           │
│  │  │ 实现功能  │                                           │
│  │  └────┬─────┘│                                           │
│  │       │      │                                           │
│  │       ▼      │                                           │
│  │  ┌──────────┐│                                           │
│  │  │ 验证结果  ││                                           │
│  │  │ lint/test│                                           │
│  │  └────┬─────┘│                                           │
│  │       │      │                                           │
│  │       ▼      │                                           │
│  │  ┌──────────┐│                                           │
│  │  │ 记录学习  ││                                           │
│  │  │ 更新状态  │                                           │
│  │  └────┬─────┘│                                           │
│  │       │      │                                           │
│  │       ▼      │                                           │
│  │  ┌──────────┐│                                           │
│  │  │ 还有故事? ││                                           │
│  │  └────┬─────┘│                                           │
│  │       │      │                                           │
│  │       ▼      │                                           │
│  └──────────────┘                                           │
│         │                                                   │
│         ▼                                                   │
│  ┌──────────────┐                                           │
│  │  任务完成     │                                           │
│  │  总结报告     │                                           │
│  └──────────────┘                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3.4 状态持久化设计

```typescript
// ralph-dev-persistence.ts
class RalphDevPersistence {
  // 状态文件结构
  private stateFile = '.ralph-dev/state.json';
  private learningsFile = '.ralph-dev/learnings.md';
  
  // 保存状态
  async saveState(state: RalphDevState): Promise<void> {
    await fs.writeFile(this.stateFile, JSON.stringify(state, null, 2));
  }
  
  // 加载状态
  async loadState(): Promise<RalphDevState | null> {
    if (!await fs.exists(this.stateFile)) {
      return null;
    }
    const content = await fs.readFile(this.stateFile);
    return JSON.parse(content);
  }
  
  // 追加学习记录
  async appendLearning(learning: Learning): Promise<void> {
    const markdown = `## ${learning.timestamp} - ${learning.storyId}

**Category:** ${learning.category}

${learning.content}

---
`;
    await fs.appendFile(this.learningsFile, markdown);
  }
  
  // 获取学习记录
  async getLearnings(): Promise<Learning[]> {
    if (!await fs.exists(this.learningsFile)) {
      return [];
    }
    const content = await fs.readFile(this.learningsFile);
    return this.parseLearnings(content);
  }
}
```

### 3.5 中断与恢复设计

```typescript
// ralph-dev-resume.ts
class RalphDevResume {
  // 检查是否有未完成的任务
  async hasPendingTask(): Promise<boolean> {
    const state = await this.loadState();
    return state && state.currentStoryIndex < state.stories.length;
  }
  
  // 恢复任务
  async resumeTask(): Promise<void> {
    const state = await this.loadState();
    if (!state) {
      throw new Error('No pending task found');
    }
    
    // 加载学习记录
    const learnings = await this.getLearnings();
    
    // 从当前故事继续
    const currentStory = state.stories[state.currentStoryIndex];
    await this.executeStory(currentStory, learnings);
  }
  
  // 暂停任务
  async pauseTask(): Promise<void> {
    const state = await this.loadState();
    await this.saveState(state);
    console.log('Task paused. Use "resume ralph dev" to continue.');
  }
}
```

---

## 四、Skill 文档结构

### 4.1 SKILL.md 模板

```markdown
---
name: ralph-dev
version: 1.0.0
description: "Autonomous code development using Ralph's iterative approach. Use when you need to implement features, refactor code, or complete multi-step development tasks. Triggers on: implement feature, develop this, build this feature, refactor code, complete this task."
user-invocable: true
---

# Ralph Dev - Autonomous Code Development

## Overview

Ralph Dev is an autonomous code development skill that uses an iterative approach to implement features, refactor code, and complete multi-step development tasks. It combines the power of AI with systematic code reading, modification, and verification.

## Core Capabilities

- **Story-driven development**: Break down complex tasks into manageable user stories
- **Iterative execution**: Execute stories one at a time with verification
- **Knowledge accumulation**: Learn from each iteration to improve future work
- **Code reading**: Deep analysis of existing codebase
- **Code modification**: Safe and verified code changes
- **Quality assurance**: Lint, typecheck, and test verification

## Usage

### Basic Usage

```
implement feature: [feature description]
```

### Advanced Usage

```
develop this: [detailed requirements]
refactor code: [refactoring goals]
complete this task: [task description]
```

## Execution Flow

1. **Requirement Analysis**: Understand the request and break it into stories
2. **Story Execution**: For each story:
   - Read relevant code
   - Plan changes
   - Make modifications
   - Verify results
   - Record learnings
3. **Completion**: Summarize results and provide report

## State Management

- **State file**: `.ralph-dev/state.json` - Tracks current progress
- **Learnings file**: `.ralph-dev/learnings.md` - Accumulates knowledge
- **History file**: `.ralph-dev/history.md` - Execution history

## Commands

- `resume ralph dev` - Resume paused task
- `pause ralph dev` - Pause current task
- `status ralph dev` - Check current status
- `reset ralph dev` - Reset all state

## Integration with Other Skills

- **code-reading-skill**: Used for deep code analysis
- **pua-private**: Applied when stuck or making poor progress
- **e2e-testing**: Used for end-to-end verification

## Best Practices

1. **Small stories**: Keep stories small enough to complete in one iteration
2. **Clear acceptance criteria**: Define verifiable success conditions
3. **Thorough verification**: Always lint, typecheck, and test
4. **Record learnings**: Document patterns and gotchas for future iterations
5. **Regular saves**: State is saved after each story completion

## Example

### Input
```
implement feature: Add user authentication with JWT tokens
```

### Output
```
## Ralph Dev Started

### Task Analysis
- Feature: User authentication with JWT tokens
- Estimated stories: 4

### Stories Generated
1. US-001: Add user schema with password hashing
2. US-002: Implement JWT token generation and validation
3. US-003: Create authentication middleware
4. US-004: Add login/logout API endpoints

### Starting Execution...
[Progress will be shown as each story completes]
```

## Troubleshooting

### Task Stuck
If a story fails multiple times, the skill will:
1. Apply pua-private for systematic debugging
2. Try alternative approaches
3. Request human intervention if needed

### State Corruption
If state file is corrupted:
```
reset ralph dev
```

## Learnings Format

```markdown
## 2024-01-15 10:30 - US-001

**Category:** pattern

**Content:**
This codebase uses bcrypt for password hashing with salt rounds of 10.
Always use the `hashPassword` utility function instead of calling bcrypt directly.

---
```
```

---

## 五、实施步骤

### 5.1 第一阶段：基础框架

1. 创建 Skill 目录结构
2. 实现状态管理模块
3. 实现执行引擎基础
4. 集成代码阅读工具

### 5.2 第二阶段：核心功能

1. 实现代码修改功能
2. 实现验证功能（lint、typecheck、test）
3. 实现学习记录功能
4. 实现中断恢复功能

### 5.3 第三阶段：优化增强

1. 集成 pua-private 处理卡壳
2. 优化故事拆分策略
3. 增强错误处理
4. 添加进度展示

### 5.4 第四阶段：测试完善

1. 编写测试用例
2. 实际项目验证
3. 性能优化
4. 文档完善

---

## 六、技术栈选择

| 模块 | 技术选择 | 理由 |
|------|---------|------|
| 状态管理 | JSON 文件 | 简单、可版本控制 |
| 学习记录 | Markdown | 人类可读、可搜索 |
| 代码阅读 | code-reading-skill | 复用现有能力 |
| 代码编辑 | IDE 工具 | 直接操作文件 |
| 验证 | 项目工具链 | 使用项目现有工具 |
| 日志 | 控制台 + 文件 | 实时反馈 + 历史记录 |

---

## 七、风险评估

| 风险 | 影响 | 缓解措施 |
|------|------|---------|
| 状态文件损坏 | 任务中断 | 定期备份、校验机制 |
| 代码修改错误 | 项目破坏 | 验证机制、回滚能力 |
| 无限循环 | 资源浪费 | 迭代限制、超时机制 |
| 学习记录污染 | 误导后续 | 人工审核、版本控制 |

---

## 八、总结

这个方案将 Ralph 的核心优势（迭代式执行、状态持久化、知识积累）与 Skill 框架的能力（工具调用、状态管理、用户交互）相结合，创造出一个强大的代码开发 skill。

**关键创新点**：
1. 保留 Ralph 的迭代式执行模式
2. 适配 Skill 框架的工具调用机制
3. 增强代码阅读和修改能力
4. 提供中断恢复和状态管理

**下一步**：
如果你认可这个方案，我可以开始实施第一阶段的基础框架。
