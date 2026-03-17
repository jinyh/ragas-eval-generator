---
name: ragas-eval-generator
description: Generate RAGAS seed QA pairs from any knowledge base. Use for RAGAS evaluation, seed questions, 评测问答, QA benchmark, RAG evaluation, ground truth generation, 检索评估, 种子问题生成, or building retrieval test sets. Invoke this skill whenever the user wants to create question-answer pairs for evaluating a RAG system.
---

<!-- RAGAS 种子问答生成器 — 内容无关的通用 Skill -->
<!-- 适用于任意知识库，生成 10 个四层难度的 RAGAS 评测问答对 -->

# RAGAS Seed QA Generator

## Purpose / 目标

从任意知识库中生成 **10 个高质量种子问答对**，用于 RAGAS（Retrieval Augmented Generation Assessment）评测。问答对覆盖四个推理难度层级，确保评测能全面检验 RAG 系统的检索精度和推理能力。

**输出格式**：RAGAS 兼容的 JSON 文件，包含 `question`、`ground_truth`、`contexts` 等标准字段。

---

## Phase 1: Scan Knowledge Base / 扫描知识库

<!-- 目标：了解知识库的规模、结构和主题分布 -->

### Steps

1. **Glob 扫描文档结构**
   - 使用 Glob 工具扫描项目中的文档文件（`**/*.md`, `**/*.txt`, `**/*.rst` 等）
   - 排除 node_modules、.git、.claude 等非内容目录

2. **识别文档元数据**
   - 统计文档数量和大小
   - 识别主要主题领域（通过文件名、目录结构、标题）
   - 如果文档量大（>20 篇），采样 Read 关键文档的开头和目录

3. **检测文档间引用关系**
   - Grep 搜索交叉引用（文件名出现在其他文件中、超链接、"参见"等）
   - 识别核心文档（被引用最多的）和边缘文档

4. **输出知识库概况表**

```markdown
| 维度 | 值 |
|------|-----|
| 文档总数 | N |
| 主题领域 | [领域1, 领域2, ...] |
| 核心文档 | [被引用最多的 3-5 篇] |
| 文档间引用密度 | 高/中/低 |
| 适合的问题覆盖范围 | [概述] |
```

---

## Phase 2: Plan Question Distribution / 规划问题分布

<!-- 目标：在动笔写问题之前，先确定每个问题的目标文档和推理类型 -->

### Steps

1. **加载难度框架**
   - Read 本 skill 的 `references/difficulty-framework.md`
   - 确认四层分布：T1(2) + T2(3) + T3(3) + T4(2) = 10

2. **规划分布草案**
   - 为每个问题槽位分配：
     - **Tier** 和 **推理类型**（从框架中选择）
     - **目标文档**（问题将基于哪些文档）
     - **初步角度**（问题大致方向）
   - 确保：
     - 知识库的主要主题都被覆盖
     - 单个文档不出现在 >3 个问题的 contexts 中
     - 推理类型有多样性（不要所有 T2 都是同一类型）

3. **输出分布草案表**

```markdown
| # | Tier | Reasoning Type | Target Docs | Angle |
|---|------|---------------|-------------|-------|
| 1 | T1 | Precise Fact Lookup | doc_a | ... |
| 2 | T1 | Structured Detail | doc_b | ... |
| 3 | T2 | Cross-doc Integration | doc_a, doc_c | ... |
| ... | ... | ... | ... | ... |
```

4. **请求用户确认**
   - 将分布草案展示给用户
   - 等待用户确认或调整后再进入 Phase 3

---

## Phase 3: Generate Questions / 生成问题

<!-- 目标：按 Tier 从低到高逐一生成问题和答案 -->

### Rules / 生成规则

- **必须 Read 验证**：生成每个问题前，必须 Read 目标文档的相关段落，从文档中提取内容
- **禁止凭知识编造**：答案中的每一个事实必须可追溯到具体文档的具体位置
- **标注 contexts**：每个问题必须附带源文档路径列表

### Generation Order / 生成顺序

按 Tier 从低到高生成，因为：
- T1 生成过程中会深入阅读文档细节，为后续 Tier 积累上下文
- T3/T4 需要跨文档联系，前面的阅读为此打基础

### Per-question Workflow / 每个问题的生成流程

```
For each question slot (Q1 → Q10):
  1. Read target document(s) — 找到与规划角度相关的具体段落
  2. Extract key facts — 提取用于构造问题的关键事实
  3. Craft question — 参考 difficulty-framework.md 中的模板，构造问题
  4. Write ground_truth — 基于提取的事实编写参考答案
  5. Record contexts — 记录源文档路径
  6. Write difficulty_analysis — 简要说明为什么这个问题属于该 Tier
```

### Output Format / 每个问题的输出格式

```json
{
  "question": "问题文本",
  "ground_truth": "参考答案（完整、准确、可追溯）",
  "tier": 1,
  "reasoning_type": "Precise Fact Lookup",
  "contexts": ["path/to/doc_a.md"],
  "difficulty_analysis": "该问题需要从 doc_a 中精确提取 [具体信息]，属于单跳检索"
}
```

---

## Phase 4: Verify Ground Truth / 验证答案

<!-- 目标：确保答案与文档完全一致，防止幻觉 -->

### Steps

1. **启动并行验证**
   - 使用 Agent 工具（Explore subagent）并行验证每个答案
   - 每个验证 agent 的任务：
     - 阅读 contexts 中列出的所有文档
     - 逐一核对答案中的事实（人名、年份、数值、公式、因果关系）
     - 报告任何不一致

2. **验证检查项**
   - [ ] 人名拼写与文档一致
   - [ ] 年份/日期与文档一致
   - [ ] 数值/公式与文档一致
   - [ ] T2/T3 的跨文档信息整合逻辑正确
   - [ ] T4 的推理前提都有文档支撑
   - [ ] contexts 列表完整（没有遗漏必要的源文档）

3. **修正**
   - 对验证发现的问题，回到 Phase 3 修正
   - 修正后需重新验证

---

## Phase 5: Output / 输出

### 5.1 生成 JSON 文件

将 10 个问答对写入 JSON 文件（默认 `ragas_seed_questions.json`，可由用户指定）：

```json
[
  {
    "question": "...",
    "ground_truth": "...",
    "tier": 1,
    "reasoning_type": "...",
    "contexts": ["..."],
    "difficulty_analysis": "..."
  },
  ...
]
```

### 5.2 JSON 格式验证

使用 Bash 运行 Python 验证 JSON 格式：

```bash
python -c "
import json
with open('ragas_seed_questions.json', 'r') as f:
    data = json.load(f)
print(f'Loaded {len(data)} questions')
for i, q in enumerate(data):
    assert 'question' in q, f'Q{i+1} missing question'
    assert 'ground_truth' in q, f'Q{i+1} missing ground_truth'
    assert 'contexts' in q, f'Q{i+1} missing contexts'
    assert isinstance(q['contexts'], list), f'Q{i+1} contexts not a list'
    assert q.get('tier') in [1,2,3,4], f'Q{i+1} invalid tier'
print('All validations passed!')
"
```

### 5.3 覆盖矩阵摘要

输出一个覆盖矩阵，展示文档×推理类型的覆盖情况：

```markdown
## Coverage Matrix / 覆盖矩阵

### By Tier
| Tier | Count | Reasoning Types Used |
|------|-------|---------------------|
| T1 | 2 | ... |
| T2 | 3 | ... |
| T3 | 3 | ... |
| T4 | 2 | ... |

### By Document
| Document | Used in Questions |
|----------|------------------|
| doc_a.md | Q1, Q3, Q7 |
| ... | ... |

### Coverage Gaps (if any)
- [列出未被覆盖的主题领域或文档]
```

---

## Output JSON Schema

<!-- RAGAS 兼容的输出字段定义 -->

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "array",
  "items": {
    "type": "object",
    "required": ["question", "ground_truth", "tier", "reasoning_type", "contexts", "difficulty_analysis"],
    "properties": {
      "question": {
        "type": "string",
        "description": "评测问题文本"
      },
      "ground_truth": {
        "type": "string",
        "description": "参考答案，完整且可追溯到源文档"
      },
      "tier": {
        "type": "integer",
        "minimum": 1,
        "maximum": 4,
        "description": "难度层级：1=单跳检索, 2=多跳检索, 3=综合, 4=反事实/元推理"
      },
      "reasoning_type": {
        "type": "string",
        "description": "推理类型（参见 difficulty-framework.md）"
      },
      "contexts": {
        "type": "array",
        "items": { "type": "string" },
        "description": "源文档路径列表"
      },
      "difficulty_analysis": {
        "type": "string",
        "description": "难度分析：为什么这个问题属于该 Tier"
      }
    }
  }
}
```

---

## Notes / 注意事项

- 这是一个**内容无关**的 skill，适用于任意知识库
- 默认生成 10 个问题（T1:2 + T2:3 + T3:3 + T4:2）
- 所有答案必须基于文档内容，不得凭 AI 通用知识编造
- 输出的 JSON 可直接用于 RAGAS 评测框架的 `EvaluationDataset`
- 如果知识库文档数量少于 5 篇，T3 和 T4 的跨文档问题可能需要调整策略
