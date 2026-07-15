# 复现蓝图：用本 Skill 一个一个做出来

> 配套 `SKILL.md` 的三段式方法论使用。每个项目给三样东西：
> 1. **确定性内核规格**（这套项目"真正算什么"，外行和 AI 一起逼出来）
> 2. **验收闸门 Gate**（客观可验证，过不了就说明内核没立住）
> 3. **参考仓库**（`D:\参考项目分析\` 下真实代码，照猫画虎）
> 标记 ★ 的项目建议**先照 GOOD 版复现**；标记 ⚠ 的项目原版是薄壳，按"纠正"复现（产出优于原版）。

---

## ★1. 记账 / 个人理财（参考 maybe）
- 内核：会计模型（账户、交易、借贷、余额、损益表），纯算术/SQL 聚合。
- Gate：给定一组交易，损益表数字可手算复核；**内核文件零 LLM import**；AI 助手只能读、不能写。
- 仓库：`maybe/`（看 `app/models/balance_sheet.rb`、`income_statement.rb`、只读 `assistant/function/`）

## ★2. 简历优化（参考 Resume-Matcher）
- 内核：ATS 加权打分 `0.55*关键词 + 0.25*技能 + 0.20*完整度`，确定性。
- Gate：同份简历同 JD，分数可复现；`ats.py` 不 import LLM；AI 改写的简历不引入原文没有的技能（有校验闸门）。
- 仓库：`Resume-Matcher/`（看 `apps/backend/app/services/ats.py`、`llm.py`）

## ★3. 合同审查（参考 ContractIntel_AI）
- 内核：条款抽取（模型）+ `risk_rules.py` 规则打分。
- Gate：抽出的条款可溯源到原文片段；风险分级由规则表决定、不随 LLM 随机变；LLM 只出"人话解读" JSON。
- 仓库：`ContractIntel_AI/`（看 `clause_extractor.py`、`config/risk_rules.py`、`llm_interpreter.py`）

## ★4. 学习卡片（参考 green-deck）
- 内核：源校验质量流水线（卡片带 `src` 映回原文）+ 三重过滤；调度可委托 Anki。
- Gate：每张卡能定位回原文句子；AI 不能编造原文没有的内容。
- 仓库：`green-deck/`（看卡片生成 + `src` 字段 + `app/api/anki.py`）

## ★5. 食谱管理（参考 mealie，作反证）
- 内核：食谱库存储 + URL 解析 + 膳食规划，纯确定性。
- Gate：拔掉 AI 配置后，增删查改/规划全部正常；AI 是可选外挂。
- 仓库：`mealie/`（看 `services/openai/openai.py` 为可选外挂）

## ★6. 习惯打卡（参考 habit-hero）
- 内核：streak（日期差累算、断天归零）+ 完成率（打卡天数/跨度天数），纯 Python。
- Gate：给一组打卡日期，streak/完成率可手算复核；AI 只产鼓励语/心情分类 JSON，不参与统计。
- 仓库：`habit-hero/`（看 `backend/crud.py` 的 `calculate_streak`/`compute_analytics`、`backend/ai.py`）

## ★7. 多 AI 投委会（参考 ai-investment-advisor）
- 内核：AKShare 客观取数（禁估算/编造）+ 共识规则（3/3 强、2/3 弱、权重=置信度×完整度）。
- Gate：数据全部来自脚本抓取、可标注"缺失"；共识判定可由规则复核；模型只做分析员。
- 仓库：`ai-investment-advisor/`（看 `scripts/fetch_market_data.py`、`AGENTS.md`、`committee/SKILL.md`）

## ★8. 营养减重（参考 LossWeightEasily）
- 内核：Mifflin-St Jeor 算 BMR/TDEE（公式写死）+ 食物营养库查表 + 向量检索（可选）。
- Gate：给定身高体重年龄性别活动量，TDEE 可手算复核；无 LLM 时 BMR/TDEE/查表照常；识别失败有 fallback。
- 仓库：`LossWeightEasily/`（看 `backend/src/services/user_service.py`、`food_repository.py`）

## ★9. 时间记录分类（参考 activity-timecard）
- 内核：窗口标题 → `.env` 关键词规则表匹配类别，100% 确定性。
- Gate：给一组标题，分类结果可预判；**全仓库零 LLM/网络调用**；输出追加到 Excel。
- 仓库：`activity-timecard/`（看 `activity_tracker.py` 的 `classify_activity`、`.env.example`）

## ★10. 课程排程 CLI（参考 study-planner）
- 内核：时间分块算法（按章节装天、超 daily_limit 换天、只排工作日、超长拆分、每日 20~480 分钟校验）。
- Gate：给课程 JSON + 每日分钟，输出 Markdown 每日表，可手算验证天数/分钟分配；**零依赖、零 LLM**。
- 仓库：`study-planner/`（看 `study_planner.py` 的 `schedule_course`/`handle_oversized_item`）

## ★11. 学习路线图（参考 dlai-roadmap）
- 内核：课程数据集（难度/时长/前置）+ 问卷→阶段/周 的查表映射。
- Gate：给问卷答案，路线可由映射规则复核；PDF/.ics 导出确定；**无 LLM**。
- 仓库：`dlai-roadmap/`（看 `src/data/courses.json`、`src/utils/pathwayGenerator.js`）

---

## ⚠12. 训练计划（原版 FitMate 薄壳 → 纠正复现）
- 纠正内核：动作库（目标×经验×器械 → 候选子集）+ 渐进超负荷公式（周间加重/加组）+ 排期。LLM 只写计划文案。
- Gate：`exercises` 选择由规则决定（不随 LLM 变）；内核文件不 import LLM；LLM 失败有回退计划（非 500）。
- 参考（看薄壳错在哪）：`FitMate-Workout-Planner/`（看 `server/exercises.csv`、`utils.py`、`groq_client.py`）

## ⚠13. 周食谱生成（原版 CookHero 薄壳 → 纠正复现）
- 纠正内核：营养约束求解器（给定每日热量/宏量 → 从食谱库挑达标组合）+ 可选 RAG 拉真实食谱。
- Gate：生成的周食谱总营养≈目标（偏差可算）；LLM 不凭空编菜；先求解器后生成。
- 参考（看薄壳错在哪）：`CookHero/`（看 `app/agent/subagents/builtin/diet_planner.py`、`app/diet/service.py`）

## ⚠14. 旅行行程（原版 ai-travel-planner 薄壳 → 纠正复现）
- 纠正内核：约束求解（按天分配景点、控预算/通勤）+ 真实 POI 数据（地图 API）进确定性层。
- Gate：行程满足天数/预算硬约束；景点来自真实数据可核查；LLM 只解说不编景点。
- 参考（看薄壳错在哪）：`ai-travel-planner/`（看 `travel_app.py` 的 `LLMChain`、被注释的 Places 代码）

## ⚠15. 学习计划（原版 AI-study-planner 薄壳 → 纠正复现）
- 纠正内核：照 #10 study-planner 的纯算法重做排程，LLM 只写导语。
- Gate：同 #10 的时间分块 Gate；内核零 LLM。
- 参考（看薄壳错在哪）：`AI-study-planner/`（看 `app.py` 的 LLM 排程）

## ⚠16. 组合分析（原版 AI_Portfolio_Analyzer 半畸形 → 纠正复现）
- 纠正内核：保留 yfinance 取数 + 波动率/β/收益计算（确定性）；**补上解释层**（指标拼结构化输入给 LLM 写报告）+ 补 MA/夏普指标。
- Gate：指标可手算复核；解释层真接上（非空壳）；取数失败有降级。
- 参考（看空壳错在哪）：`AI_Portfolio_Analyzer/`（看 `ai_portfolio.py` 的 openai 空调用、`streamlit_app.py` 指标）

## ⚠17. 口语陪练（原版 BabelDuck 外壳型 → 升级复现）
- 纠正内核：抄其 GOOD 外壳（Redux 状态 + localStorage + 多供应商路由）；把"纠错"升级为"确定性语法/词典校验 + LLM 润色"双轨。
- Gate：外壳与 AI 解耦（无 key 有兜底）；纠错有规则层可复核，不只靠 LLM 现编。
- 参考：`BabelDuck/`（看 `store.tsx`、`chat-persistence.ts`、`llm-service.ts`、`intelligence.ts`）

---

## 复现通用验收清单（每项目收尾必过）
- [ ] 确定性内核文件 `grep` 不到任何 LLM/AI import（或经论证确实不需要）。
- [ ] 核心数字/决定可手算或独立脚本复核（可复现，固定种子/无随机）。
- [ ] AI 层只解释/生成，不参与硬计算；失败有降级而非崩溃。
- [ ] 缺数据/外部失败有明确 fallback（返回占位/默认值/标注缺失）。
- [ ] 生成内容可溯源（引用原文/数据来源），无凭空编造。
