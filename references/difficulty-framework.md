# Four-Tier Reasoning Difficulty Framework for RAGAS Seed Questions

<!-- 四层推理难度框架：用于生成 RAGAS 种子问答对的抽象模板 -->
<!-- 所有模板使用 [槽位] 占位符，不含特定领域术语 -->

## Overview / 框架总览

| Tier | Name | Count | Reasoning Depth | Docs Required |
|------|------|-------|-----------------|---------------|
| T1 | Single-hop Retrieval | 2 | 单文档精确查找 | 1 |
| T2 | Multi-hop Retrieval | 3 | 跨文档信息整合 | 2–3 |
| T3 | Synthesis | 3 | 多源知识链重构 | 4–5 |
| T4 | Counterfactual & Meta-reasoning | 2 | 反事实推理 + 批判性分析 | 2–4 |

**Total: 10 questions per knowledge base.**

---

## Tier 1 — Single-hop Retrieval (2 questions)

<!-- 测试：检索精度，防止"差不多对"的幻觉 -->

### Reasoning Types / 推理类型

1. **Precise Fact Lookup / 精确事实查找**
   - 答案是文档中的一个确切事实（人名、年份、数值、公式）
   - 错误答案和正确答案只差一个细节

2. **Structured Detail Extraction / 结构化细节提取**
   - 答案需要从文档中提取一组结构化信息（列表、步骤、组成部分）
   - 不完整的列表 = 不合格

### Question Templates / 问题模板

```
T1-a (Precise Fact Lookup):
"[Person] 在 [Year] 提出的 [Theory/Method] 中，用什么方法解决了 [Problem]？
 具体包含哪些 [N] 个关键要素？"

T1-b (Structured Detail Extraction):
"根据 [Document X] 的描述，[Concept A] 由哪 [N] 个核心组件构成？
 请按 [Document X] 中的原始顺序列出。"
```

### Design Principles / 设计原则

- 答案必须**完全可追溯**到单一文档中的具体段落
- 设计"高混淆度"：相似概念、相邻年份、同一人的不同贡献中选择
- 验证标准：如果删除目标文档，模型应无法仅凭通用知识回答正确

---

## Tier 2 — Multi-hop Retrieval (3 questions)

<!-- 测试：跨文档信息整合，需要从 2-3 篇文档中拼接答案 -->

### Reasoning Types / 推理类型

1. **Cross-document Concept Integration / 跨文档概念整合**
   - 概念 A 出现在文档 X，概念 B 出现在文档 Y，答案需要关联两者

2. **Evolution Path Tracing / 演进路径追踪**
   - 追踪一个概念/方法从起源到当前形态的演变
   - 需要按时间线从多个文档中串联信息

3. **Dual-perspective Comparison / 双视角对照**
   - 同一主题在两个不同文档/领域中的表述差异
   - 需要提取两个视角并做结构化对比

### Question Templates / 问题模板

```
T2-a (Cross-document Integration):
"[Concept A]（见 [Document X]）和 [Concept B]（见 [Document Y]）之间
 存在什么核心联系？这种联系如何体现在 [Application/Method] 中？"

T2-b (Evolution Path Tracing):
"从 [Person A] 在 [Year1] 的 [Early Work] 到 [Person B] 在 [Year2] 的
 [Later Work]，[Core Idea] 经历了哪些关键演变？每一步的核心突破是什么？"

T2-c (Dual-perspective Comparison):
"[Field X] 和 [Field Y] 分别如何理解 [Shared Concept]？
 列出至少 [N] 个维度的异同。"
```

### Design Principles / 设计原则

- 答案中的**每个事实片段**必须可追溯到具体文档
- 单一文档不足以回答完整问题——需要信息整合
- 问题措辞不应暗示需要查看哪些文档

---

## Tier 3 — Synthesis (3 questions)

<!-- 测试：多文档谱系重构，需要从 4-5 篇文档中重建知识链条 -->

### Reasoning Types / 推理类型

1. **Multi-document Lineage Reconstruction / 多文档谱系重构**
   - 重建一个领域/方法从多个源头汇聚而成的完整脉络
   - 需要跨 4+ 篇文档串联因果链

2. **Multi-source Convergence Analysis / 多源汇聚分析**
   - 分析多个独立发展的思想如何汇聚到同一成果
   - 需要识别来自不同文档的贡献并整合

3. **Component-to-Source Mapping / 组件-来源映射**
   - 给定一个复杂系统/方法，追溯每个组件的学科来源
   - 需要在多个文档间建立精确的映射关系

### Question Templates / 问题模板

```
T3-a (Lineage Reconstruction):
"[Modern Method/System] 的核心思想可以追溯到哪些历史源头？
 请重建从 [Earliest Origin] 到 [Current Form] 的完整发展脉络，
 标注每个关键节点的贡献者、年份和核心突破。"

T3-b (Convergence Analysis):
"[Achievement/Breakthrough] 是多个学科交叉的产物。
 请分析来自 [Field A]、[Field B]、[Field C] 等领域的
 哪些具体思想/技术为其提供了关键基础？各自的贡献是什么？"

T3-c (Component-Source Mapping):
"[Complex System] 包含 [N] 个核心组件。请逐一追溯每个组件的
 学科起源和关键贡献者，并说明它们如何被整合到当前框架中。"
```

### Design Principles / 设计原则

- 答案需要**重构完整链条**，不是简单罗列
- 必须包含因果关系和时间顺序
- 遗漏任何关键节点 = 不完整答案
- 验证时检查：答案中的每个"因为X所以Y"是否都有文档支撑

---

## Tier 4 — Counterfactual & Meta-reasoning (2 questions)

<!-- 测试：超越检索的推理能力——反事实推理、批判性分析 -->

### Reasoning Types / 推理类型

1. **Counterfactual Causal Reasoning / 反事实因果推理**
   - "如果 X 没有发生，Y 会如何变化？"
   - 需要理解因果结构，而非仅仅记忆事实

2. **Argument Strength Evaluation / 论证强度评估**
   - 评估文档中某个论点的证据充分性
   - 需要识别论证的假设、前提和潜在弱点

3. **Philosophical Meta-analysis / 哲学元分析**
   - 从更高层次审视知识库中的方法论假设
   - 需要跨越具体事实进行抽象思考

### Question Templates / 问题模板

```
T4-a (Counterfactual Reasoning):
"如果 [Historical Event/Discovery] 没有发生（或推迟 [N] 年），
 [Downstream Development] 最可能沿什么替代路径发展？
 请基于文档中描述的因果关系进行推理，而非猜测。"

T4-b (Argument Evaluation / Meta-analysis):
"文档中关于 [Claim/Thesis] 的论证依赖哪些关键假设？
 这些假设中哪些有充分的文档证据支持，哪些是隐含的？
 如果最薄弱的假设被推翻，结论会如何改变？"
```

### Design Principles / 设计原则

- 答案仍须**锚定在文档事实上**，但需要推理延伸
- "合理的推理"和"瞎猜"的区分：推理的每一步前提都应可追溯
- 没有唯一正确答案，但有"可接受的推理质量标准"
- Ground truth 应标注：哪些部分是文档直接支持的，哪些是基于文档的合理推理

---

## Quality Checklist / 质量检查清单

### Coverage Check / 覆盖度检查

- [ ] 所有 10 个问题是否覆盖了知识库中的**主要主题领域**？
- [ ] 是否避免了同一文档被过度引用（单个文档不应出现在 >3 个问题的 contexts 中）？
- [ ] 四个 Tier 是否都有问题（T1:2, T2:3, T3:3, T4:2）？
- [ ] 推理类型是否有足够多样性（不应所有 T2 都是同一种推理类型）？

### Answer Traceability Check / 答案可追溯性检查

- [ ] T1 答案：是否能指向**精确的段落/句子**？
- [ ] T2 答案：是否标注了来自**每个文档的具体事实片段**？
- [ ] T3 答案：因果链中的**每个节点**是否都有文档支撑？
- [ ] T4 答案：是否区分了"文档直接支持"和"基于文档的推理"？

### Anti-hallucination Check / 反幻觉检查

- [ ] 答案中的所有人名、年份、数值是否与文档**完全一致**？
- [ ] 是否存在文档中没有的"常识性补充"被混入答案？
- [ ] 对于 T3/T4，推理步骤是否逻辑自洽且前提可追溯？
