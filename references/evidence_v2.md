# 证据库 v2：22 个跨领域开源项目的源码级解剖

> 全部来自 `参考项目分析V2/` 下的真实克隆仓库（浅克隆，已逐项目读源码核实；超大仓库 openbb/twenty 经 GitHub API 读取关键源码）。
> 判定标签：**GOOD** = 符合方法论（确定性内核 + AI 解释分离，可直接当模板）；**WEAK/ANTI** = 薄壳（LLM 抢硬计算 / 无降级），需按方法论纠正后复现。
> 与 v1（17 项目）合流后，本 Skill 证据库共 **39 个项目**，覆盖约 14 种内核形态范式。
> 每个项目的完整解剖见 `参考项目分析V2/analysis/<slug>.md`。

---

## 〇、总览与重大修正
- **22 项目中 21 GOOD + 1 WEAK**（ai-therapist）。
- **源码推翻搜索摘要**：原假设 D 类 3 个 WEAK，读源码后证伪 2 个——`ai-meal-planner`（BMR+0/1 背包 DP）、`ai-real-estate`（混合检索内核+Haversine）实为 GOOD。仅 `ai-therapist` 是真实薄壳。
- **结论**：GOOD/WEAK 必须靠**源码 grep 内核**判定，搜索摘要的"AI X 生成器"字眼会系统性误导。

## 一、范式分类（内核形态图鉴）
| 内核形态 | v2 项目 |
|---|---|
| 检索内核（RAG） | private-gpt, quivr, docsgpt, langchain-chatchat, ai-real-estate |
| 规则 / DSL 引擎 | semgrep, openfisca-core, languagetool |
| 最优化 | PyPSA, freqtrade, inventree |
| 符号计算 | sympy |
| 乐理计算 | music21 |
| 路由算法 | osrm-backend |
| 神经推理（确定性） | argos-translate |
| 数据聚合 | OpenBB |
| 领域模型 | twenty, medplum, wger, openfoodfacts(+inventree) |
| 算法内核 | ai-meal-planner |

---

## 二、22 个项目逐源码摘要

### 1. private-gpt（RAG / 私有文档问答）★57k — GOOD
- 确定性内核：`components/workflows/retrieval/retrieval.py` 的 `retrieve_raw_nodes()` → `self._retriever.aretrieve(query)`（向量相似度检索）；ingestion（切块+embedding 推理，非 LLM 生成）。
- AI 层：chat/completion 把 chunks 拼 context 生成答案。
- 金标准：retrieval.py 仅 import llama_index 检索组件，零 LLM import。拔掉 LLM 仍返回 chunks。✅
- 启示：补"检索内核"范式——RAG 是最强确定性内核之一。

### 2. quivr（RAG 库）★39k — GOOD
- 确定性内核：`core/quivr_core/rag/quivr_rag.py` 的 `retriever` 属性 → `vector_store.as_retriever()`（向量检索）+ reranker 重排。
- AI 层：`llm_endpoint` 只在 build_chain 末段合成答案（作者注释刻意解耦 LLM）。
- 金标准：retriever 零 LLM import。✅
- 启示：库级 RAG 与 private-gpt 应用级互补，共同证明"内核=检索、AI=解释"。

### 3. docsgpt（RAG / 文档客服）★18k — GOOD
- 确定性内核：向量检索 + 切块 + 嵌入（确定性）。
- AI 层：LLM 答案合成。
- 启示：RAG 范式第三例，工业级客服。

### 4. langchain-chatchat（RAG 框架）★38k — GOOD
- 确定性内核：知识库 vectorstore/retriever（向量检索）。
- AI 层：LLM 生成（中文场景）。
- 启示：RAG 范式第四例，本地知识库。

### 5. ai-real-estate（房产对话搜索）★— GOOD（原误判 WEAK）
- 确定性内核：`apps/api/vector_store/hybrid_retriever.py` —— 向量 + BM25 + 元数据过滤 + **Haversine 地理距离** + 价格/房间/年份/能耗过滤 + reranking。真实匹配 100% 确定性。
- AI 层：对话层只消费检索结果。
- 金标准：检索内核零 LLM import。✅
- 启示：推翻"LLM 包揽检索"假设；与 ai-travel-planner（真 WEAK）形成同领域正反对照。

### 6. osrm-backend（路由引擎 / C++）★6k — GOOD
- 确定性内核：`src/engine/routing_algorithms/routing_base_ch.cpp`（双向 Dijkstra/CH/MLD），`src/contractor/*`（图压缩）。纯图算法。
- AI 层：无内置 LLM；`src/engine/guidance/*` 是"解释层只读内核"的最佳范本（把节点序列翻译成转向步骤）。
- 金标准：全仓无 LLM import（C++，仅 Boost/TBB 等）。✅
- 启示：补"路由算法"范式；guidance 模块示范"解释层绝不参与计算"。

### 7. openfisca-core（税务规则引擎 / Python）★— GOOD
- 确定性内核：`openfisca_core` 的 parameters/variables/formulas（立法规则有向图计算）。
- AI 层：无内置 LLM。
- 金标准：全仓 grep "openai|llm|gpt" 零命中。✅
- 启示：补"规则引擎"范式（规则图 vs LLM 猜）。

### 8. sympy（计算机代数 / Python）★15k — GOOD
- 确定性内核：`sympy/core`、`sympy/solvers`（符号求值/化简/求解方程）。
- AI 层：无。
- 金标准：纯 Python 符号计算，零 LLM。✅
- 启示：补"符号计算"范式——内核可以是数学推导。

### 9. music21（乐理计算 / Python）— GOOD
- 确定性内核：音程/和弦/调性分析算法。
- AI 层：无。
- 金标准：零 LLM。✅
- 启示：补"乐理计算"范式。

### 10. PyPSA（能源最优化 / Python）— GOOD
- 确定性内核：线性/网络最优化建模（发电机调度/潮流）。
- AI 层：无。
- 金标准：零 LLM。✅
- 启示：补"最优化"范式。

### 11. languagetool（语法规则引擎 / Java）— GOOD
- 确定性内核：规则 XML + 匹配器（语法/风格规则匹配）。
- AI 层：无。
- 金标准：零 LLM。✅
- 启示：补"规则引擎"范式（自然语言侧）。

### 12. argos-translate（神经翻译 / Python）— GOOD
- 确定性内核：`argostranslate/translate.py` 的 `LanguageModel`/`Translation`（CTranslate2 本地模型推理，确定性推理非 LLM 生成）。
- AI 层：无（它是翻译内核本身）。
- 金标准：grep 命中 torch/transformers 是模型推理依赖，非 LLM API 调用。✅
- 启示：补"神经推理（确定性）"范式——区分"本地模型推理"与"LLM 自由生成"。

### 13. inventree（库存管理 / Python）— GOOD
- 确定性内核：库存/物料/BOM 数据模型与计算（EOQ、库存校验）。
- AI 层：无（可选外挂）。
- 金标准：零 LLM。✅
- 启示：补"领域模型+最优化"范式；与 v1 FitMate（WEAK）对照（库存可确定性算，训练计划也应确定性）。

### 14. medplum（临床 FHIR 平台 / TS）— GOOD
- 确定性内核：FHIR 资源数据模型与校验（TypeScript/SQL）。
- AI 层：无（可选）。
- 金标准：零 LLM。✅
- 启示：补"领域模型"范式（医疗强数据内核）。

### 15. semgrep（代码模式匹配 / OCaml+Python）★16k — GOOD
- 确定性内核：`semgrep-core`（OCaml AST 合一匹配）+ 规则 DSL（YAML pattern/metavariable）。
- AI 层：无（云端 Assistant 仅解释 findings）。
- 金标准：Python 编排层 grep "import openai" 零命中。✅
- 启示：补"规则引擎"范式；直接反衬 v1"LLM 当分类器/判定器"的 WEAK（能写规则的别问 LLM）。

### 16. openfoodfacts（食品营养数据库）— GOOD
- 确定性内核：食品/营养数据模型与查表计算（营养分/标签）。
- AI 层：无。
- 金标准：零 LLM。✅
- 启示：补"领域模型+查表"范式；是 v1 LossWeightEasily（WEAK，LLM 编营养）的工业级对照版。

### 17. wger（健身/营养追踪 / Python）— GOOD
- 确定性内核：训练/营养数据模型与计算（BMI/热量/动作库）。
- AI 层：无。
- 金标准：零 LLM。✅
- 启示：补"领域模型"范式；是 v1 FitMate（WEAK）的 GOOD 对照版。

### 18. OpenBB（金融数据聚合 / Python）★71k — GOOD
- 确定性内核：`openbb_core`（统一数据接口）+ `provider/`（registry/query_executor，多源确定性适配）+ `extensions`（各源 HTTP 拉取）。
- AI 层：`openbb-agent`（独立仓库，可选）把自然语言转 API 调用。
- 金标准：`openbb_platform/core` 无 LLM 目录，拔掉 AI 数据仍可查。✅
- 启示：补"数据聚合"范式；强化铁律 1/2（数据新鲜度、全链路溯源）。

### 19. freqtrade（加密回测 / Python）★52k — GOOD
- 确定性内核：`optimize/backtesting.py`（已读源码：import 全 freqtrade/numpy/pandas，**零 LLM**）+ `data` + `strategy`。
- AI 层：`freqai`（可选 ML，不进入回测核心）。
- 金标准：backtesting.py 零 LLM import。✅
- 启示：强化铁律——**防未来数据泄漏**（严格时间隔离）、**防过拟合**（样本外验证）。

### 20. twenty（CRM / TS）★25k — GOOD
- 确定性内核：`packages/twenty-server/src/modules` + `database` + `engine`（TypeORM 领域模型）。
- AI 层：`twenty-claude-skills` 等外围工具，不在核心数据路径。
- 金标准：核心数据模型零 LLM。✅
- 启示：补"领域模型"范式；直接反驳 v1"LLM 直接 CRUD"的 WEAK（先有领域模型，AI 只解释）。

### 21. ai-meal-planner（膳食规划）— GOOD（原误判 WEAK）
- 确定性内核：`calculate_bmr()`（Mifflin-St Jeor）+ `knapsack()`（0/1 背包 DP 选食物，满足热量/宏量约束）。
- AI 层：Llama-3 只写菜名/描述文案。
- 金标准：bmr/knapsack 零 LLM import。✅
- 启示：推翻"LLM 编菜"假设；与 v1 CookHero（WEAK，LLM 自由编菜）正反对照。

### 22. ai-therapist（AI 心理咨询）— WEAK（唯一真实薄壳）
- 薄壳表现：纯 LLM 通道 + 前端，无临床内核/分级/规则/降级。
- 内核文件 grep：无确定性内核；LLM 既算又解释（直接给临床/心理建议）。
- 违反铁律：诚实降级（无）、全链路溯源（无）、高危领域裸奔（无分级/免责/人工兜底）。
- 命中红旗：🚩 AI 既算又解释 + 🚩 高危领域裸奔。
- 纠正复现：把临床建议重写成"分级规则 + 危机关键词触发人工/热线 + 明确免责声明"，LLM 只做共情对话外壳，绝不给医疗判定。
- 启示：反模式清单从"生活类专属"升级为"通用警示 + 高危领域加倍护栏"。

---

## 三、红旗升级（回写 SKILL.md 用）
- 原红旗清单（v1）已含"AI 直接吐结论无内核""LLM 既算又解释"等，但示例偏生活类。
- **v2 升级**：
  1. 加 **ai-therapist** 作为"高危领域裸奔"典型（医疗/心理/法律/金融必须分级/免责/人工兜底）。
  2. 加 **金融特例红旗**：🚩 用未来数据回测（泄漏）/ 🚩 全历史拟合称效果好（过拟合）。
  3. 强调"**源码验证 GOOD/WEAK**"：搜索摘要的"AI X 生成器"会误导，必须 grep 内核再判定。
  4. 新增"加速是细节不是内核"护栏：先朴素版跑通再加速（CH/MLD/向量索引/缓存）。
