# WorkBuddy 对话上下文

> 本文件是「麦当劳程序员创意开发大赛 · WorkBuddy 专项奖励」的提交材料。
> 赛事规则对它的定义是：**WorkBuddy 对话上下文，用于联动活动**。
>
> 所以这里不写安装步骤（在 `README.md`），也不写 MCP 接口细节（在 `MCP_INTEGRATION.md`）。
> 这里写的是**这个项目在 WorkBuddy 里真实发生过什么**，以及一段可以直接接着聊的上下文。

---

## 一、一句话说清

Mc-nager 从选题、探测 MCP、抓语料，到写出 `SKILL.md` 并实际压测开场，**全程在腾讯 WorkBuddy 里完成**，没有换过别的工具。

WorkBuddy 在这个项目里不是代码补全，是**一个能调 MCP 工具、能读飞书文档、能跑 Python、能写文件、并且会自己验证结果的执行者**。

---

## 二、真实使用记录

每一步都在开发工作区里留下了文件。

> 说明：表中 `data/`、`docs/`、`tools/` 是开发工作区的产物，属于分析过程，**不在本仓库内**。
> 仓库里可以直接核验的是 `SKILL.md`、`corpus/`、`scripts/`、`memory/`。

| 时间 | WorkBuddy 做的事 | 留下的东西 |
|---|---|---|
| 10-21 | 抓取全量报名 Issue，分类前 52 名 README，判断哪些方向已经挤满 | `data/all_issues.json`、`data/classified.json`、`data/digest_top52.md` |
| 10-22 | 逐个城市快照菜单与券，比对同一商品的跨城、跨渠道价 | `data/menu_snapshot.json`（20 城）、`data/city_prices.json` |
| 10-23 | 直连 `mcp.mcd.cn` 走完协议握手，脚本化探测 35 个工具，逐个记录入参与返回字段树 | `tools/mcp_client.py`、`docs/mcp-tools-inventory.md`（1074 行） |
| 10-23 | 真实下单一个麦辣鸡腿堡，核清券商品码与菜单商品码的差异 | 订单 `1030855370000808602806418214`，实付 11.9 元（原价 22.5） |
| 10-23 | 抓取 @麦当劳 官方微博 586 条公开博文并清洗，蒸馏说话方式 | `corpus/weibo-corpus.jsonl`、`scripts/sample_corpus.py` |
| 10-23 | 读取飞书知识库「麦门！！」下的 5 篇方案文档，写成 v1.0 | `SKILL.md`（v1.0，tag `v1.0`） |
| 10-24 | 按实测反馈重构 v1.1：删掉 references 层、接入历史订单、弱化点餐触发 | `SKILL.md`（v1.1，提交 `f187bef`） |

### 一次被推翻的设计

v1.0 的 `SKILL.md` 底下挂了三份 references：`persona.md`、`voice.md`、`opening.md`。

作者实测后判定"太复杂、效果不好"，要求全部删掉，只留一个入口。WorkBuddy 把三份文件备份到仓库外，删掉整层引用，并修掉了 `README.md` 和本文档里因此失效的 4 处指向。

**现在这个 skill 只有一个 `SKILL.md`。** 这条教训写进了项目记忆：单角色 chat-skill 不要铺多层 references。

---

## 三、只有拿着工具才看得到的东西

| 发现 | 影响 |
|---|---|
| 真实数据在 `result.structuredContent`，`content[0].text` 是字段说明文档 | 照常规解析会全军覆没 |
| 券绑定的商品码 ≠ 菜单商品码（`901440` vs `1440`） | 用菜单码配券报"促销规则不支持该商品" |
| `query-meals` 对已打烊门店返回 `600057`，不是参数错误 | 深夜调试会误以为是调用方式写错了 |
| 同城不同门店有 0.5~1 元差价 | 菜单价格只能当"参考价"，不能当承诺 |
| 「巨无霸指数」方向已被证伪：5 个独立项目做，最好的只有 9★ | 只产出数字的东西既不能省钱也不能晒，换方向 |

以上每一条都是调工具调出来的，不是读文档读来的。

---

## 四、可直接接着聊的上下文

把下面这段贴进一个 WorkBuddy 新会话，它就能接手这个项目。

```text
我在做一个麦当劳程序员创意开发大赛的参赛项目，叫 Mc-nager（职业"金"理人）。
仓库：https://github.com/SY-TANG-ai/Mc-nager

它是什么：一个 WorkBuddy Skill。装上后 Agent 会扮演麦当劳门店经理，
用麦当劳官方微博的语气陪用户聊天。不推销、不下单，当前阶段只做陪聊。

当前版本 v1.1。结构极简：只有 SKILL.md 一个入口，
配 corpus/weibo-corpus.jsonl（586 条官微语料）和 scripts/sample_corpus.py（按月加权抽样）。

设计上的硬约束：
- 只有一个入口文件。之前拆过 references/persona|voice|opening 三层，
  作者实测判定太复杂，已全部删除。不要往回加。
- 说话方式只学官微人格体（第一人称、短句、留白、把话头递回去），不学营销播报体。
  禁止直接引用微博原句。
- 活动信息只来自 MCP campaign-calendar 实时调用。取不到就不提，绝不编造。
- 可以装腔作势，但绝不许看不起客人。

已知的 MCP 坑：
- 真实数据在 result.structuredContent，content[0].text 是字段说明文档。
- 券绑定的商品码 ≠ 菜单商品码，901440 配券报"促销规则不支持该商品"。
- query-meals 对已打烊门店报 600057，深夜要换 24 小时店（成都 3450339）。
- 金额单位是分。

写作分工：README.md 和 MCP_INTEGRATION.md 由作者本人撰写，改动前先确认，
不要擅自重写。SKILL.md 的改动只做润色，不扩写、不新增指导性内容。
```

---

## 五、为什么是 Skill，不是 App

| | App | Skill（本项目） |
|---|---|---|
| 用户获取 | 要装、要注册、要授权 | 一句话唤起 |
| 交互成本 | 打开 → 找一个功能 | 直接说话 |
| 适合什么 | 有明确任务 | 有情绪、想聊天 |

Mc-nager 的全部价值在对话本身。做成 App 反而抬高了推门进来的门槛——而这个角色存在的意义，恰恰是让推门这件事变容易。

---

腾讯 WorkBuddy · <https://www.workbuddy.cn>
