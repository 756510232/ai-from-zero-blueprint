# 证据库：17 个真实开源项目的源码级解剖

> 全部来自 `D:\参考项目分析\` 下的真实克隆仓库（浅克隆，已逐项目读源码核实）。
> 判定标签：**GOOD** = 符合方法论（确定性内核 + AI 解释分离，可直接当模板）；**WEAK/ANTI** = 薄壳或畸形（LLM 抢硬计算 / 解释层空壳），需按方法论纠正后复现。
> 无论 GOOD 还是 WEAK，都印证同一句话：**外行能靠 AI 做出来、且做得稳的项目，内核一定是"能算/能查的确定性东西"。**

---

## 一、GOOD —— 符合方法论，可直接当模板（11 个）

### 1. maybe（记账 / 个人理财）★52k
- 成品：个人记账、资产负债表、损益表。
- 确定性内核：`app/models/balance_sheet.rb`、`income_statement.rb`、`transaction.rb` —— 纯算术/SQL 聚合，**不 import 任何 LLM**。
- AI 层：`app/models/*/assistant/function/` 下 `GetBalanceSheet` 等**只读 function 工具**消费数据；独立控制器 + 用户 consent 开关 + `provider/openai.rb`（gpt-4.1）。AI 只能读、不能写。
- 复现要点：先写死会计模型（借贷/余额/损益），AI 助手只通过只读接口问答。三层隔离的教科书。

### 2. Resume-Matcher（求职 / 简历优化）★26k
- 成品：简历 vs JD 匹配打分 + 优化建议。
- 确定性内核：`apps/backend/app/services/ats.py` —— **加权关键词匹配** `overall = 0.55*关键词 + 0.25*技能 + 0.20*完整度`，`ats.py` **完全不 import LLM**。
- AI 层：`apps/backend/app/services/llm.py`（支持本地 Ollama）只抽关键词/改写/生成 diff。
- 护栏：`validate_master_alignment()` 会删掉 AI 虚构的技能/公司（防编造闸门）。
- 复现要点：打分用确定性加权公式，AI 只解释差距。这是"内核不必是向量库"的铁证。

### 3. ContractIntel_AI（法律 / 合同审查）
- 成品：抽条款 + 风险分级 + 谈判点。
- 确定性内核：`clause_extractor.py`（Legal-BERT 抽条款）+ `config/risk_rules.py`（规则表打分）。
- AI 层：`llm_interpreter.py` 只解读风险/给谈判点，不参与抽取；失败走 fallback（置信度 0.50）；结构化 JSON 溯源。
- 复现要点：抽取与分级用模型+规则，AI 只做"人话翻译"。决策与解释分离。

### 4. green-deck（教育 / 学习卡片）
- 成品：从文档自动生成 Anki 卡片。
- 确定性内核：**源校验质量流水线** —— 卡片带 `src` 字段能映回原文、三重过滤防幻觉（本仓不自己实现间隔重复，调度委托给 Anki `app/api/anki.py`）。
- AI 层：LLM 生成卡片（带源校验）。
- 复现要点：生成类应用的护栏 = "生成物必须能映回原文"。内核可委托给成熟外部系统。

### 5. mealie（食谱 / 膳食规划）★11k
- 成品：食谱库管理 + 膳食规划。
- 确定性内核：纯确定性（存储/URL 解析/膳食规划），`services/` 下逻辑不依赖 LLM。
- AI 层：`services/openai/openai.py` 是**可选外挂**，没配 provider 就抛异常 —— **拔掉 AI 核心照常工作**。
- 复现要点：反证"骨架本质是先有可靠内核，AI 只是锦上添花"。

### 6. habit-hero（习惯 / 打卡）
- 成品：打卡、连续天数 streak、完成率、心情分析。
- 确定性内核：`backend/crud.py` 的 `calculate_streak()` / `compute_analytics()` —— streak 纯 Python 日期运算（`log_date` 排序相邻差 1 天累加，断天归零），`success_rate = 打卡天数/跨度天数`。
- AI 层：`backend/ai.py`（Gemini `gemini-2.5-flash`）只产鼓励语 `generate_motivation()`、心情分类 `analyze_notes()`（输出 JSON）。
- 复现要点：统计全确定性，AI 只写文本。缺笔记返 `neutral` 占位（柔性降级）。

### 7. ai-investment-advisor（金融 / 多 AI 投委会）
- 成品：多模型独立分析同标的后取共识，给投资建议。
- 确定性内核：`scripts/fetch_market_data.py`（AKShare 取数）+ `AGENTS.md` 数据原则"价格必须来自脚本、禁止估算/编造"；`committee/SKILL.md` 共识规则（3/3 强共识、2/3 弱共识、权重=置信度×数据完整度）。
- AI 层：Claude / Codex / Gemini 各自分析，人工串联执行规则（仓库内无聚合代码，规则写在 SKILL.md 供 LLM 读）。
- 复现要点：客观数据抓取 + 共识规则，模型只做"分析员"。规则可文档化由 LLM 执行。

### 8. LossWeightEasily（营养 / 减重）★替代 AI-Calories-Calculator
- 成品：食物营养查询、体重记录、BMR/TDEE 计算、智能食谱规划。
- 确定性内核：`backend/src/services/user_service.py` 的 `calculate_bmr`（Mifflin-St Jeor：`10w+6.25h-5a+5/-161`）、`calculate_tdee`（×1.2~1.9）；营养库 PostgreSQL 表 `Food`（`food_repository.py` 按 `fdc_id` 查表）；向量检索 Milvus（确定性）。
- AI 层：`LoseWeightAgent`（qwen/dashscope）用于食谱规划、图识别；两处 init 均 `try/except` 置 None —— **无 agent 时 BMR/TDEE 照常算**。
- 复现要点：公式 + 查表是确定性内核，AI 仅可选增强。识别失败返 fallback（热量 0）。

### 9. activity-timecard（时间记录 / 活动分类）★替代 Tracker
- 成品：定期抓活动窗口标题，按规则判类别，追加到 Excel 时间卡。
- 确定性内核：`activity_tracker.py` 的 `classify_activity(title)` —— 纯规则：窗口标题依次与 `.env` 关键词表（`WORKS_N_KEYWORDS`/`RESEARCH_KEYWORDS`/`BREAK_KEYWORDS`/`WORK_KEYWORDS`）做 `in` 子串匹配，按优先级返类别。
- AI 层：**无**。grep 全仓库无 openai/llm/http。100% 确定性规则 + cron 调度。
- 复现要点：分类用关键词规则表，零模型。证明"轻内核 + 无 AI 也能自洽"。

### 10. study-planner（课程拆解 / 学习计划）★替代 course_pilot
- 成品：输入课程大纲 JSON + 每日分钟 + 起始日，输出均衡 Markdown 每日学习表。
- 确定性内核：`study_planner.py` 纯函数排程：`schedule_course()`（按章节线性装入天、超 daily_limit 换天）、`generate_weekday_sequence()`（只排周一~五）、`handle_oversized_item()`（超长视频拆多天）、`validate_daily_limit()`（20~480 分钟）。
- AI 层：**无**。imports 仅标准库（argparse/json/sys/re/datetime），零网络/AI。
- 复现要点：排程是纯算法，AI 完全可选。骨架里的 AI 层可被证明彻底省略。

### 11. dlai-roadmap（课程拆解 / 学习路线）★替代 course_pilot 之二
- 成品：基于 100+ 门课数据集，按问卷生成个性化学习路线 + 周计划 + PDF/日历导出。
- 确定性内核：`src/data/courses.json`（121 门课，含 `difficulty`/`estimated_hours`/`prerequisites`/`career_paths`）+ `src/utils/pathwayGenerator.js` 的 `generatePathway(answers)` —— 查表式映射把课程分到阶段/周，按 `estimated_hours/weeklyHours` 算 `startWeek/endWeek`。
- AI 层：**无**。grep 无 apiKey/openai。路线 100% 规则驱动。
- 复现要点：数据集 + 映射规则即确定性内核，导出（jsPDF/.ics）也确定性。

---

## 二、WEAK / ANTI —— 薄壳或畸形，需纠正后复现（6 个）

> 这些项目"外行也能靠 AI 写出来"，但正因为内核被 LLM 吃掉，它们**脆弱、不可复现、易编造**。Skill 的价值之一，就是教你怎么把这类薄壳重构成 GOOD 版本。

### 12. FitMate-Workout-Planner（健身 / 训练计划）★反例
- 成品：生成 12 次个性化训练。
- 问题：选动作/组数/次数由 `server/app/groq_client.py` 的 Groq `llama3-8b`（temperature=0.7）黑盒决定；`server/exercises.csv` 仅当文本喂给 LLM，**无"目标/经验/器械→动作"规则映射**。确定性只在 `utils.py` 的 `apply_progression()`（渐进超负荷）+ `generate_dates()`（排期）。LLM 失败直接抛 500，无回退。
- 纠正复现：把"动作选择"抽成确定性规则（目标×经验×器械 → 动作库子集），LLM 只生成计划文案/解释。内核文件必须不 import LLM。

### 13. CookHero（食谱 / 膳食生成）★反例
- 成品：按目标/过敏生成周食谱。
- 问题：周计划由 `app/agent/subagents/builtin/diet_planner.py` 用 LLM + `web_search` **自由生成**，直接参与"挑哪道菜"；RAG（Milvus）只用于问答，未接周计划；"营养计算"只是事后 `sum()` 求和，全文无 constraint/solve。
- 纠正复现：建"营养约束求解器"（给定每日热量/宏量目标 → 从食谱库挑达标组合），LLM 只解释。先有求解器，再谈生成。

### 14. ai-travel-planner（旅行 / 行程）★反例
- 成品：按偏好生成行程。
- 问题：唯一确定性计算是 `trip_duration`（天数）；`travel_app.py` 的 `LLMChain`(Ollama llama3) 包揽"哪天去哪"的硬排程；Wikipedia RAG 仅注入背景摘要，**不防具体景点编造**（Google Places 代码被注释）。
- 纠正复现：建约束求解（按天分配、控预算/通勤），拉真实 POI 数据（地图 API）进确定性层，LLM 只解说。删掉注释、喂真数据。

### 15. AI-study-planner（学习计划）★反例
- 成品：按 deadline 生成学习计划。
- 问题：唯一计算是 `app.py:102` 的 `days_left`；排程完全交给 OpenRouter 的 `llama-4-maverick`，prompt 直接要求 LLM "分配小时数、加休息"。无时间分块算法，无降级（空字段直接 stop）。
- 纠正复现：照 `study-planner`（#10）的纯算法重做排程内核，LLM 只写导语。

### 16. AI_Portfolio_Analyzer（投资 / 组合分析）★半畸形
- 成品：组合 β/波动率/买卖点 + 解读。
- 问题：确定性内核成立（`ai_portfolio.py` 加权 β、`streamlit_app.py` 波动率=`returns.std()` 从 yfinance 取数，全程无 LLM）；但 `ai_portfolio.py` **import openai 却从不调用**，LLM 解释层是空壳。无 MA/夏普（README 暗示的没实现）。
- 纠正复现：保留确定性内核，补上解释层（把指标拼成结构化输入给 LLM 写报告），并补 MA/夏普指标计算。解释层缺失比内核错误更隐蔽。

### 17. BabelDuck（口语 / 对话陪练）★外壳型
- 成品：多场景口语对话 + 侧边语法/翻译/润色 + 本地存储 + 多供应商。
- 内核（确定性外壳）：`store.tsx` Redux 状态机 + `chat-persistence.ts` localStorage + `llm-service.ts`/`intelligence.ts` 供应商路由（OpenAI/SiliconFlow/302AI，TTS WebSpeech/Azure，STT 路由）。
- 问题：对话与纠错（revise/generate）均由 LLM 现编，**无语法/词典规则层**；无 key 抛错、free trial 兜底。
- 纠正复现：外壳已是 GOOD 模板（状态/存储/路由可抄）；但把"纠错"升级为"确定性语法/词典校验 + LLM 润色"双轨，可防编造。对话类应用也能有确定性内核。

---

## 三、跨项目规律（写进 Skill 的结论）

1. **确定性内核 ≠ 必须复杂 ML**。可以是：加权公式（Resume-Matcher）、纯算术（maybe/habit-hero）、规则表（activity-timecard）、约束求解（CookHero 应有）、时间分块算法（study-planner）、数据集+映射（dlai-roadmap）、查表（LossWeightEasily）。**越轻越好**。
2. **真分离最简自检 = 内核文件 `grep` 不到任何 LLM import**。FitMate/CookHero/ai-travel-planner/AI-study-planner 都过不了这条。
3. **内核可委托给成熟外部系统**（green-deck → Anki；activity-timecard 甚至零 AI）。
4. **生成类护栏 = 源校验流水线**（green-deck 的 `src` 映回原文）。
5. **解释层缺失比内核错更隐蔽**（AI_Portfolio_Analyzer 空壳）—— 验收时要把"解释层是否真接上"列为 Gate。
6. **17 个里 6 个是薄壳**：说明"外行靠 AI 写出来"和"写出 robust 项目"之间有方法论鸿沟。本 Skill 就是填这条沟的。
