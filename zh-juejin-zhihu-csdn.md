（掘金 / 知乎 / CSDN，约 1 200 字）

## 标题
让 Claude Code 不再「失忆」：myc，一个完全本地、零 API Key 的编码 Agent 记忆层

## 正文

用过 Claude Code、Codex 或 opencode 超过一小时的人，大概都遇到过这种事：一开始和 Agent 商量好「用方案 A，不用方案 B，因为 C」，写了一小时代码之后，上下文被压缩，Agent 一脸认真地又提出了方案 B。原因它已经不记得了，因为原因根本不在它的上下文里了。

最近在 GitHub 上看到一个专门解决这个问题的开源项目：**myc**（取自 mycelium，菌丝）。它不是又一个云端记忆 API，而是一个放在项目目录旁边的本地任务与记忆层，一个 SQLite 文件搞定一切。

### 它做了什么

- **任务队列**：带依赖、阻塞关系和原子认领，多个 Agent 并行时不会抢同一个任务。
- **决策日志**：「为什么选 A 不选 B」以节点形式保存，支持混合检索（BM25 + 向量）。
- **上下文压缩前的钩子**：在 Claude Code 触发 PreCompact 的那一刻，把整个会话写到磁盘（自动脱敏），并把最关键的一段「救援包」写回存活的上下文。
- **候选而非事实**：从对话里自动提取的「决策」先标记为待确认，人工确认前不会进入检索结果。作者的理由很直接：会撒谎的记忆比没有记忆更危险。
- **记忆锚定到代码**：知识绑定在具体的代码片段上，代码重构、搬到别的文件，记忆跟着走；代码被删了，`recall` 会显示 `[code gone ×0.2]` 并降权，而不是把过时的东西当事实喂给 Agent。
- **内置代码索引**：tree-sitter，支持 TypeScript / JavaScript / Python，符号、调用关系、语义搜索。

### 性能是硬约束

作者把延迟当成设计约束而不是事后优化，每条热路径都有预算并有测试守着（10 万节点，p99）：

| 操作 | p99 | 预算 |
|---|---|---|
| 会话上下文包 `prime` | 0.6 ms | 30 ms |
| 混合检索 | 8.2 ms | 25 ms |
| 冷启动 | 21 ms | 60 ms |

所有数字可以用仓库里的 `bun run scripts/bench-latency.ts` 复现，网站构建时会校验每个数字，对不上就构建失败。测试 3 700+，每个防护都有对应的变异测试。

### 诚实地说缺点

- 只能跑在 **Bun** 上（依赖 `bun:sqlite` + `sqlite-vec`），Node / Deno 起不来。
- macOS 和 Linux；Windows 需要 WSL。
- 目前是单机单用户工具，没有服务端、团队模式和 ACL，这些在路线图里但还没实现。

### 安装

```bash
bun install -g @aistastudio/myc
cd your-project
myc init
myc wire        # 一条命令接入 Claude Code / Codex / opencode / Kimi
```

仓库：https://github.com/aistastudio/myc
网站（功能、路线图、可复现的性能图表，中文暂无，英/俄）：https://aistastudio.github.io/myc/

项目才发布不久，作者每天都在发版，欢迎去提 issue。

---
