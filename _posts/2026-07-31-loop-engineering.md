---
layout: post
title: "Loop Engineering：从喂 Prompt 到设计让 Agent 自己干活的系统"
category: books
catalog: true
published: true
tags:
  - books
  - AI
time: '2026.07.31 16:28:00'
---

# Loop Engineering：从喂 Prompt 到设计让 Agent 自己干活的系统

> 本文是对花叔所著《Loop Engineering 橙皮书》(v260615) 的完整总结，原书发布于 2026 年 6 月。

---

## 一、什么是 Loop Engineering

### 一句话定义

> **Loop engineering is replacing yourself as the person who prompts the agent. You design the system that does it instead.**
>
> 循环工程，就是把「那个负责 prompt agent 的人」从你自己换成一套系统。你不再亲自一句句喂，而是设计那套替你喂的系统。

这个词由 Google Chrome 团队工程师 **Addy Osmani** 在 2026 年 6 月命名。但几乎同一周，三个人分别撞到了同一件事：

| 人 | 身份 | 说了什么 |
|---|---|---|
| **Peter Steinberger** | OpenClaw 作者 | "你不该再去 prompt 编程 agent 了。你该去设计那些会 prompt 你 agent 的循环。" |
| **Boris Cherny** | Anthropic / Claude Code 负责人 | "我不再 prompt Claude 了。我让一堆循环跑着，由它们去 prompt Claude。我的工作，就是写循环。" |
| **Addy Osmani** | Google Chrome 团队 | "循环工程，坐在 harness 的上一层楼。" |

三个人互不通气，一周内指向同一个词——不是巧合，是大家各自做着类似的事，做到一定程度发现需要一个共同的名字。

### 核心转变

**旧世界**：你坐在那儿，一句句 prompt agent。它干完一件，停下，等你下一句。你是循环里的人肉时钟。

**新世界**：你设计一套东西，让它自己一拍一拍地敲。它在定时器上跑、自己孵化小帮手去干活、自己把结果喂回给自己。你不在循环里了，你在循环外面看着它转。

---

## 二、四层栈：从 Prompt 到 Loop

过去两年 AI 圈造的那串「XX Engineering」，不是互相替代的，是叠起来的：

| 层 | 管什么 | 核心问题 |
|---|---|---|
| **Prompt Engineering** | 写好一次的提示词 | 我该告诉模型什么 |
| **Context Engineering** | 这一刻窗口里放什么 | 检索什么、摘要什么、清掉什么 |
| **Harness Engineering** | 单次运行的武装 | 给哪些工具、允许哪些动作、什么算完成 |
| **Loop Engineering** | 在 harness 之上调度 | 怎么让它自己一遍遍跑起来 |

每往上一层，你操心的东西就大一号。从「一句话」到「一个窗口」，再到「一次运行」，最后到「一整套自动跑的循环」。

**为什么非得分这么细？** 因为每一层的失败方式不一样。Prompt 错了你当场看见；Context 装错了模型答偏，也容易察觉；但到了 Loop 这层，它在你睡觉时自己跑、自己改你没看过的代码、自己把错误也一并喂给下一轮——出问题你可能几天后才发现。

> 层次越高，你离现场越远，犯的错就攒得越久。

---

## 三、一个循环的五个动作

Loop 里的「循环」不是空转。每转一圈，都在干一件具体的活：

### 1. 发现（Discovery）

搞清楚「这一圈该干什么」。关键是让 agent 自己去找活，而不是你把活喂给它。

Addy 的 triage loop 里，一个 triage skill 去读 CI 失败、open issues、最近 commits——合起来就是「昨天到今天，系统里出了哪些值得管的事」。

> 发现这一步做得好不好，决定了整个 loop 的质量上限。

### 2. 交付（Handoff）

把任务从调度系统交到干活的 agent 手里。不是简单喊一声「去修这个」，而是每个发现单独开一个隔离的 worktree，让多个 agent 各改各的，互不踩脚。

### 3. 验证（Verification）

**最不能省的一步。** 第一个子 agent 起草修复后，第二个子 agent 来审查——对照项目的 skill 和测试，instructions 跟第一个不一样，有时候连模型都换一个。

> 一个 loop 难的不是 loop 本身，难的是往里面塞一个能说「不」的东西。

### 4. 持久化（Persistence）

结果得落到一个能活过这次对话的地方。改动通过 connector 自动开 PR、更新 ticket；处理不了的进收件箱等人；还有一个贯穿始终的状态文件，记着「转到哪了」。

> Agent 会忘，仓库不会。记忆得待在磁盘上，不能待在上下文里。

### 5. 调度（Scheduling）

前四个动作合起来是「转一圈」，但转一圈不等于一个 loop。真正让它成为「循环」的，是调度——让它在定时器上不停地转，替你把这件事一直做下去。

> Automations are what make a loop an actual loop and not just one run you did once.

---

## 四、六个零件：搭一个 Loop 需要什么

| 零件 | 是什么 | 对应动作 | 核心原话 |
|---|---|---|---|
| **Automations** | 挂在时间表/触发器上自动跑 | 调度 | "make a loop an actual loop" |
| **Worktrees** | 隔离并行 agent 的工作目录 | 交付 | "same headache as two engineers" |
| **Skills** | 固化项目知识（SKILL.md） | 发现 | "fire $skill-name, not a wall of instructions" |
| **Connectors** | MCP 接外部系统 | 持久化/发现 | "only see the filesystem is a tiny loop" |
| **Sub-agents** | 生成者与评判者分离 | 验证 | "too nice grading its own homework" |
| **Memory** | 磁盘上的持久状态 | 持久化 | "the agent forgets, the repo doesn't" |

六个零件齐了，一个能自己转的 loop 就有了骨架：Automation 让它动，Worktree 让它不打架，Skill 让它不重复劳动，Connector 让它看得见外面，子 agent 让它能自我纠错，Memory 让它记得住。

---

## 五、生成器与评判器：为什么写代码的 AI 不能给自己打分

### 它总夸自己

Anthropic 工程师 Prithvi Rajasekaran 观察到一个稳定现象：

> When asked to evaluate work they've produced, agents tend to respond by confidently praising the work—even when, to a human observer, the quality is obviously mediocre.

让 agent 评价自己产出的东西，它往往会自信地夸一通，哪怕在人看来质量明显很一般。这不是模型不够聪明，是它在给自己的作业打分——写代码的上下文里已经塞满了「我为什么这么写」的理由。

### 改一个怀疑论者，比改一个谦虚的作者容易

> Tuning a standalone evaluator to be skeptical turns out to be far more tractable than making a generator critical of its own work.

调一个独立的评判器让它变得怀疑，比让生成器自我批判要容易得多。区别在结构，不在措辞——你没法靠一句「请严格一点」让作者跳出自己的视角，但你可以换一个 agent，给它完全不同的指令，让它从零开始看这段代码。

### 评判器要会动手，不只是会读

Rajasekaran 在前端任务中的做法：evaluator 接上 Playwright MCP，自己打开页面、点按钮、截图、查 DOM，像一个真人 QA 那样去用这个东西。判断依据从「我觉得这段 JSX 没问题」变成了「我点了登录按钮，页面跳转了，截图在这」。

### 产品落地：/goal

Claude Code 的 `/goal` 命令把 maker-checker 原则用在了停止条件上：

```
/goal all tests in test/auth pass and the lint step is clean
```

每跑完一轮，一个又小又快的模型来检查条件成立没有。不成立，Claude 就再跑一轮，而不是把控制权交还给你。**是否完成，由一个全新的模型判定，不是由干活的那个判定。**

> 一个 loop 的下限，是它的评判器。Generator 的水平决定 loop 能产出什么，Evaluator 的水平决定 loop 不会产出什么。

---

## 六、三个真实的 Loop

### 案例一：Addy 的早晨 — 个人 triage loop

每天早上 automation 自动醒来，读 CI 和 issue，开 worktree，子 agent 起草和审查，过了就自动开 PR，没把握的丢进收件箱等人，状态写进文件留给第二天。

关键细节：automation 调用的是 `$skill-name`，不是把一大段指令贴死在日程里。

> You fire $skill-name instead of pasting a giant wall of instructions into a schedule that nobody will ever update.

### 案例二：Stripe 的 Minions — 每周 1300 个 PR

每周合并 1300 多个 PR，没有一行是人手写的。触发方式很轻——Slack 里 @ 一下或加个 emoji 反应。

真正让它靠谱的，是 **LLM 醒来之前那一段**：一个确定性的 orchestrator 先把上下文备齐（扫链接、拉 Jira、找文档、用 Sourcegraph + MCP 搜代码），等 agent 上场，桌上已经摆好了它要的所有材料。

> 能用确定性逻辑解决的，绝不交给概率模型。

架构是六层，确定性的 gate 和 LLM 的创造步骤交替咬合。沙箱用 Devbox 跑在 EC2 上，"cattle not pets"，用完即弃。底层是开源工具 Goose 的一个 fork——**AI 的可靠性来自约束的质量，不是模型的大小。**

注意最后一行：人没退场，人换了工位。1300 个 PR 仍然由工程师 review，时间从「写」挪到了「审」。

### 案例三：「睡觉时跑」到底靠什么

| | Cloud Routines | Desktop 定时任务 | /loop |
|---|---|---|---|
| 跑在哪 | Anthropic 云 | 你的机器 | 你的机器 |
| 需要开机吗 | 否 | 是 | 是 |
| 最小间隔 | 1 小时 | 1 分钟 | 1 分钟 |
| 能看本地文件吗 | 否（fresh clone） | 能 | 能 |

本地 `/loop` 是「我在场时让它替我多跑几轮」，云端调度才是「我不在场它也照跑」。两件事，别混。

---

## 七、代价：四笔账

> A loop running unattended is also a loop making mistakes unattended.
>
> 一个没人看着的循环，也是一个没人看着犯错的循环。

### 1. 验证债（Verification Debt）

循环自动开 PR、改代码、合并，每一步都省了你的时间。但省下来的时间不是白来的，它变成了一堆「还没人验证过」的产出，堆在那儿等你还。

问题藏在测试没覆盖到的地方，藏在「能跑」和「对」之间的缝里。它们安静地躺着，攒到某个上线的早上一起爆。

### 2. 理解腐烂（Understanding Rot）

循环每天替你写一堆你没写过的代码。代码能跑、测试能过、PR 能合。但有个东西在背后慢慢变质：你对这个项目的理解。

> The faster the loop ships code you did not write, the bigger the gap between what exists and what you actually get.

代码库在长大，你脑子里的那张地图却停在三个月前。等出事的那天，你打开文件，发现自己像在看别人的项目。

### 3. 认知投降（Cognitive Surrender）

> When the loop runs itself it's very tempting to stop having an opinion and just take whatever it gives back.

循环越可靠，你越容易把判断这件事整个外包出去。验证债和理解腐烂还是技术问题，认知投降是态度问题。前两个是你没时间管，这个是你不想管了。

### 4. Token 失控（Token Runaway）

一个自主跑的循环，消耗有多少，很难提前算准。循环会自己孵化小帮手、自己重试、自己在定时器上一遍遍跑。你写的是逻辑，跑出来的是次数，而次数乘以单价才是钱。

| 代价 | 症状 | 防它 |
|---|---|---|
| 验证债 | 产出堆着没人验，错误安静积累 | 装一个跟干活的不是同一个的评判者 |
| 理解腐烂 | 代码在长，你脑里的地图停了 | 定期读产出，讲不出就是该更新 |
| 认知投降 | 循环给啥收啥，懒得有意见 | 执行可外包，拿主意不行 |
| Token 失控 | 用量剧烈波动，账单不可预测 | 上线前钉死预算和重试上限 |

> 循环工程最迷人的地方，是它让一个人能干一个团队的活。最危险的地方，也是同一处——团队会互相 review、互相提醒、互相吵架，一个人加一堆循环，很容易变成没人吵架的回音壁。

---

## 八、当工程师，不只是按下启动键

> Two people can build the same loop and get opposite outcomes.

同样的循环，两个人造，结果可以截然相反。区别不在循环里，在造它的人手上。

一个人用循环，是为了在自己已经吃透的事情上跑得更快。循环帮他扩大的，是他本来就有的判断。

另一个人用同样的循环，是为了不必再去理解。看不懂没关系，循环会写；判断不了没关系，循环会合。他用循环把「理解」这件麻烦事整个绕过去了。

**循环本身不偏向任何一边，它只是个忠实的乘号，乘的是你。**

### 不稀缺的和稀缺的

循环让「生成」变得极其廉价。代码、方案、PR、修复，都能批量造出来。当一样东西可以被无限生成，它就不再值钱。

什么还稀缺？**判断力。** 知道哪个方案是对的、哪行代码该拦下来、哪个产出虽然能跑但根上错了——这件事循环替不了你。

> 循环工程不是让判断力贬值，恰恰相反。它把所有不需要判断的活都拿走了，剩下的全是判断。

### Addy 的收尾

> Build the loop. But build it like someone who intends to stay the engineer, not just the person who presses go.

造那个循环。但要像一个打算继续当工程师的人去造它，而不是一个只负责按下启动键的人。

---

## 九、今天就动手：搭你的第一个 Loop

### 五步走

**第一步：跑一个 /loop**

```bash
/loop 5m check the deploy          # 固定 5 分钟一次
/loop check the deploy             # Claude 自己决定节奏
/loop                              # 裸跑，执行 .claude/loop.md
```

时间单位 `s/m/h/d`，最小间隔 1 分钟。Session-scoped，7 天后过期，机器关了就停。

**第二步：让它读 CI 和 issue，先做 triage**

给它一个 prompt，让它每天早上去看 CI 失败、新开 issue、最近 commit，挑出值得处理的，列个清单。

**第三步：加一个状态文件，让它有记忆**

开一个 markdown 文件，把每次的发现、处理到哪一步都写进去。这就是 loop 的记忆——今天没处理完的，明天醒来还能接着干。

**第四步：加一个 evaluator，让它能说不**

```
/goal all tests in test/auth pass and the lint step is clean
```

判断条件成立的，是另一个 fresh 模型，不是干活的那个。

**第五步：加 worktree，让它并行**

```bash
--worktree  # 或 -w，给每个后台 agent 开独立 worktree
```

### 工具速查（2026 年 6 月）

| 能力 | Claude Code | Codex |
|---|---|---|
| 定时调度 | `/loop` | Automations 标签页 |
| 跑到条件满足 | `/goal` | 靠 automation 重跑 + 判断 |
| 并行隔离 | `--worktree / -w` | 专用 background worktree |
| 子 agent | `.claude/agents/` | `.codex/agents/` TOML |
| 外部连接 | MCP + Plugins | MCP connector |
| 显式调技能 | Skills (SKILL.md) | `$skill-name` |
| 关机也跑 | Cloud Routines | 规划中的 Codex Jobs |

### 第一个 Loop 检查清单

| 要素 | 问自己 |
|---|---|
| 发现源 | 它定时去读什么？ |
| 状态文件 | 跨轮的记忆落在哪个磁盘文件上？ |
| Evaluator | 有没有一个独立的、会说「不」的检查？ |
| 隔离 | 并行的 agent 是不是各自一个 worktree？ |
| Token 上限 | 设没设花费的天花板？跑飞了谁拦得住？ |
| 人工复核点 | 哪一步停下来等你看一眼？ |

前两条决定你的 loop 能不能跑，后四条决定它跑起来会不会闯祸。

> 第一个 loop，宁可小，也要把那个会说「不」的检查和人工复核点装齐。

---

## 总结

Loop Engineering 标记的是一次身份变化：从操作 agent 的人，变成调度 agent 的人。

- **Prompt** 管一句话，**Context** 管一窗户，**Harness** 管一次跑，**Loop** 管让它自己跑下去
- 一个循环 = 发现 + 交付 + 验证 + 持久化 + 调度
- 六个零件：Automations、Worktrees、Skills、Connectors、Sub-agents、Memory
- 最难的不是搭循环，是往里面放一个能说「不」的东西
- 四笔账：验证债、理解腐烂、认知投降、Token 失控——都不会在当下报警
- 造循环，但要像一个打算留下来的人去造

> 这本书到这儿就讲完了。原理、零件、案例、代价，能说的都说了。剩下的不在书里，在你的终端里。现在就去搭你的第一个 loop。

---

*原书作者：花叔 · AI Native Coder*  
*橙皮书免费下载：[huasheng.ai/orange-books](https://huasheng.ai/orange-books)*

# 参考文章
[loop-engineering-orange-book](https://github.com/alchaincyf/loop-engineering-orange-book)  
[loop-engineering](https://github.com/cobusgreyling/loop-engineering)
