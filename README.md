# RAGAS Eval Generator

一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) Skill，从任意知识库自动生成 [RAGAS](https://docs.ragas.io/) 种子问答对，用于评测 RAG 系统的检索精度和推理能力。

## 特性

- **内容无关** — 适用于任意领域的知识库（技术文档、学术论文、百科全书等）
- **四层推理难度** — 从单跳事实查找到反事实推理，全面覆盖
- **文档可追溯** — 每个答案都锚定在源文档的具体段落上，杜绝幻觉
- **RAGAS 兼容** — 输出 JSON 直接对接 RAGAS 评测框架

## 难度框架

| Tier | 名称 | 数量 | 说明 |
|------|------|------|------|
| T1 | Single-hop Retrieval | 2 | 单文档精确查找 |
| T2 | Multi-hop Retrieval | 3 | 跨 2-3 篇文档信息整合 |
| T3 | Synthesis | 3 | 跨 4-5 篇文档知识链重构 |
| T4 | Counterfactual & Meta-reasoning | 2 | 反事实推理 + 批判性分析 |

## 生成流程

```
Phase 1: Scan    → 扫描知识库结构、主题、文档间引用
Phase 2: Plan    → 规划 10 个问题的分布（Tier + 推理类型 + 目标文档）
Phase 3: Generate → 按 Tier 从低到高逐一生成，Read 验证每个事实
Phase 4: Verify  → 并行 Agent 核对答案与文档的一致性
Phase 5: Output  → JSON 输出 + 格式验证 + 覆盖矩阵
```

## 输出格式

```json
[
  {
    "question": "问题文本",
    "ground_truth": "参考答案",
    "tier": 2,
    "reasoning_type": "Cross-document Concept Integration",
    "contexts": ["docs/topic_a.md", "docs/topic_b.md"],
    "difficulty_analysis": "需要从两篇文档中整合概念 A 和概念 B 的关系"
  }
]
```

## 安装

将 skill 目录放到 Claude Code 项目的 `.claude/skills/` 下：

```bash
# 方式一：直接克隆到 skill 目录
git clone https://github.com/jinyh/ragas-eval-generator.git \
  .claude/skills/ragas-eval-generator

# 方式二：作为 git submodule
git submodule add https://github.com/jinyh/ragas-eval-generator.git \
  .claude/skills/ragas-eval-generator
```

## 使用

在 Claude Code 对话中，用以下任意方式触发：

- "帮我生成 RAGAS 种子问题"
- "为这个知识库生成评测问答对"
- "Generate RAGAS seed QA pairs"
- "Create RAG evaluation ground truth"

Skill 会自动扫描当前项目的文档，引导你完成整个生成流程。

## 文件结构

```
ragas-eval-generator/
├── README.md                          # 本文件
├── SKILL.md                           # 主指令（五阶段流程）
└── references/
    └── difficulty-framework.md        # 四层推理难度框架 + 问题模板
```

## License

MIT
