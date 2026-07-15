# ai-from-zero-blueprint

从零搭建「**领域陌生 + AI 可辅助实现**」应用的 Skill：即使你**完全不懂某个领域**，也能靠和 AI 的对话，一步步**从零做出**一个稳健、可验证、可溯源的应用（不是复刻别人的代码）。

> 核心理念：**计算与解释分离**。任何这类应用，真正产生数字/判定的是"确定性内核"（公式、规则表、查表、检索、轻量算法），LLM 只是把内核结果翻译成人话。你不用懂领域，只要和 AI 一起把"内核算什么"逼出来即可。

---

## 这个 Skill 能干什么

当你想从零做一个新应用（预测类、RAG 咨询类、AI 辅助决策/生成类……），调用它，它会驱动整个过程：

1. **第零步 · brainstorming** —— 先和用户确认真实产品目标（不是"复盘/复现"这类元任务），锁定形态/受众/成功标准/YAGNI。
2. **A 侦察** —— 逼 AI 当领域分析师，主动套三层架构、6 类陷阱、数据清单，列出概念表与可溯源依据。
3. **B 拆解** —— 把项目切成固定的 6 个 phase，给每个 phase 定义**四要素验收 Gate**（约束目标/完成标准/风险/检测方法）。
4. **C 建造** —— 逐 phase 验收推进，内核文件**根本不 import LLM**，每步可验证、可溯源、缺数据显式降级。

附带两个进阶能力：
- **反模式纠正**：识别"LLM 抢了硬计算"的脆弱薄壳，给出改造成"有内核"的具体方案。
- **并行多路线压力测试**：对陌生/高风险领域，让 2-3 个执行体用不同技术路线（纯确定性 / RAG+LLM / schema先行+诚实降级）各做一遍同一目标，汇总各路线特有失败模式，合流成一次 Skill 修订——这是 Skill 持续变强的飞轮。

## 文件结构

```
ai-from-zero-blueprint/
├── SKILL.md                  # 方法论主文件：三层骨架 + 四铁律 + 3段式 + 17项目证据表 + 反模式纠正 + 并行多路线
├── references/
│   ├── blueprint-template.md # 固化模板：三层骨架 + 6类陷阱 + 数据清单 + 标准6-phase + 四要素（AI 主动套用）
│   ├── evidence.md           # 17 个真实开源项目逐源码解剖（每条结论带具体文件路径）
│   ├── walkthrough.md        # 实战演练：从零做一个"张雪峰式高考志愿应用"
│   └── prompt-bank.md       # A / B / C 三段 AI 自查清单 + 第零步 brainstorming 调用方式
└── README.md
```

## 作为 WorkBuddy Skill 安装

把整个目录放到 WorkBuddy 的 skills 目录下即可（用户级 `~/.workbuddy/skills/` 或项目级 `.workbuddy/skills/`），然后在对话里用自然语言触发，例如：

- "从零做一个 AI 应用"
- "不懂这个领域怎么靠 AI 做"
- "怎么跟 AI 一步步沟通、这个项目一共多少步"
- "AI 帮我搭 XXX 系统"
- "如何验证 AI 做的对不对"

## 证据来源（17 个真实开源项目，均已读源码）

覆盖 9 种范式：预测（世界杯）、RAG 咨询（张雪峰高考志愿）、记账（maybe）、简历匹配（Resume-Matcher）、合同审查（ContractIntel_AI）、学习卡片（green-deck）、食谱（mealie）、健身（FitMate）、营养（LossWeightEasily）、旅行（ai-travel-planner）、口语陪练（BabelDuck）、学习计划（study-planner / AI-study-planner / dlai-roadmap）、习惯（habit-hero）、投资（AI_Portfolio_Analyzer / ai-investment-advisor）、时间记录（activity-timecard）等。

其中 11 个为 GOOD（确定性内核 + AI 解释），6 个为 WEAK（LLM 抢了硬计算或解释层空壳）——后者正好用于演示"薄壳如何被纠正成 robust 项目"。

## License

MIT
