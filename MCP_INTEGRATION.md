# MCP 接入说明

本文档说明 Mc-nager 实际使用的麦当劳 MCP Server、Tool、调用流程，以及每个调用在产品里承担什么。

---

## 一、MCP Server

| 项 | 值 |
|---|---|
| 端点 | `https://mcp.mcd.cn` |
| 传输协议 | Streamable HTTP（JSON-RPC 2.0） |
| 认证方式 | `Authorization: Bearer ${MCD_MCP_TOKEN}` |
| Token 申请 | <https://open.mcd.cn/mcp> → 手机号登录 → 控制台 → 激活 |
| 工具总数 | 35 个 |
| 限流 | 600 次 / 分钟 |
| 使用范围 | 中国大陆，非商业用途 |

**接入方式（WorkBuddy）**

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamable-http",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer ${MCD_MCP_TOKEN}"
      }
    }
  }
}
```

> 仓库内 `mcp-config.example.json` 只含 `${MCD_MCP_TOKEN}` 占位符，无任何真实凭据。

---

## 二、Version 1.0 实际调用的 Tool

### `campaign-calendar` — 活动日历

**调用时机：每次开场前。**

这是 Version 1.0 唯一强依赖的工具。理由很简单：**角色要讲"我最近在忙什么"，这件事必须有真实来源。** 没有它，经理只能编"我们最近上了新品"，一句话就露馅。

| 项 | 内容 |
|---|---|
| 入参 | 无（不接受参数） |
| 关键返回路径 | `data.dailyList[].events[].articleDto.title` / `.content` |
| 数据形态 | 对象（非数组） |
| 时效性 | 返回 `currentTime` 与 12 天左右的活动窗口，含 `today: true` 标记当天 |

**调用流程**

```
① 读取 data.dailyList
       ↓
② 筛出 date == 今天 且 today == true 的那一天
       ↓
③ 遍历 events，取 articleDto.title / articleDto.content
       ↓
④ 按 title 去重（同一天同一活动会出现多条）
       ↓
⑤ 作为"我最近在忙什么"的唯一素材来源
```

**真实返回片段**（2026-10-23 实采，已精简）

```json
{
  "success": true,
  "code": 200,
  "datetime": "2026-10-23 00:31:09",
  "traceId": "87c54dd21c51ab5eb1325fa1f06e1268",
  "data": {
    "currentTime": "2026-10-23 00:31:09",
    "dailyList": [
      {
        "date": "2026-10-23",
        "dateText": "2026年10月23日 今日",
        "today": true,
        "events": [
          {
            "activityCode": "...",
            "articleDto": {
              "title": "麦当劳「我就喜欢」限定丝巾正式开售🔥",
              "content": "丝巾搭配棒球帽的潮流谁懂😎\n任意餐品消费加 39.9 元即可得\n颈间、包袋、手腕、头饰一巾多玩🤔\n10月22日10:45开售",
              "highlights": "..."
            }
          }
        ]
      }
    ]
  }
}
```

**踩过的坑**

| 坑 | 说明 |
|---|---|
| `activityTitle` 是空的 | 真正有内容的标题在 `articleDto.title`，不在 `activityTitle` |
| 同日重复 | 同一个活动在当天的 `events[]` 里可能出现多条，需按 `title` 去重 |
| 有"往期回顾" | `date` 早于今天的条目是历史活动，不能当"最新"讲 |
| 拿不到就不要提 | 调用失败或当天无文章时，降级为讲店里的事（修冷冻柜、数库存），**不编活动** |

**配套调试脚本**

仓库外提供了 `fetch_today_campaign.py`（见项目主仓库 `tools/`），可直接把返回压成可读摘要，用于核对语料是否过期：

```bash
python tools/fetch_today_campaign.py --days 3
```

输出：

```
## 2026-10-23 【今天】
### 麦当劳「我就喜欢」限定丝巾正式开售🔥
- 丝巾搭配棒球帽的潮流谁懂😎
- 任意餐品消费加 39.9 元即可得
...
---
工具: campaign-calendar  traceId: 87c54dd21c51ab5eb1325fa1f06e1268
```

---

## 三、Version 1.0 明确不调用的 Tool

这一点是刻意设计，不是遗漏。Version 1.0 是**纯 chat 版本**，目标是先把"人"演好，再考虑"卖东西"。

| 工具 | 为什么不用 |
|---|---|
| `calculate-price` | 涉及价格核算，聊天阶段不需要报价 |
| `create-order` | 不替用户下单 |
| `query-my-coupons` / `query-store-coupons` | 不发券、不推荐优惠 |
| `query-meals` | 不主动推商品。客人问具体商品时，按 `campaign-calendar` 里出现的价格回答 |
| `draw-lottery` / `mall-*` | 抽奖与积分商城属后续版本 |
| `party-*` | 派对活动属后续版本 |

---

## 四、已实测可用的工具池（为后续版本准备）

以下 28 个只读工具已在开发过程中实测通过，返回结构与字段均已记录，可直接用于 1.1 及以后：

| 分类 | 工具 | 后续用途 |
|---|---|---|
| 基础 | `now-time-info` | 更精确的时段判断 |
| 门店 | `query-nearby-stores`、`delivery-query-stores` | "你在哪家店附近" |
| 菜单 | `query-meals`、`query-meal-detail` | 客人问具体商品时查真实在售与价格 |
| 菜单 | `query-meal-assistance` | 企业团餐场景 |
| 营养 | `list-nutrition-foods` | 热量查询（注意：返回逗号分隔字符串，不是 JSON） |
| 营销 | `campaign-calendar` | **Version 1.0 正在使用** |
| 营销 | `query-promotions` | 团餐折扣矩阵与不参与优惠商品 |
| 券 | `available-coupons`、`query-my-coupons`、`query-store-coupons` | 1.2 的"自然推荐"素材 |
| 积分 | `mall-points-products`、`mall-product-detail` | 低金额兑换品 |
| 交易 | `calculate-price` | 1.2 报价 |
| 活动 | `query-party-city`、`query-party-store`、`query-party-store-date`、`query-party-store-session` | 亲子派对场景 |
| 查询 | `query-order`、`order-list`、`query-my-account` | 客人问"我上次点了什么" |

**写操作均已实测但 Version 1.0 不用**：`create-order`（生成待支付订单，返回 `payH5Url`，15 分钟超时）、`cancel-order`、`delivery-create-address`、`auto-bind-coupons`、`draw-lottery`、`mall-create-order`、`party-order-create`。

---

## 五、业务价值

### 1. 让角色"有据可依"

角色扮演类应用最容易死在两件事上：**编事实**和**说不出细节**。

`campaign-calendar` 同时解决了这两个问题：

- **不编事实**：所有活动、价格、发售时间都来自官方接口，有 `traceId` 可查
- **有细节**：接口返回的文案本身就带具体信息（"10月22日10:45开售"、"200目精细研磨"），这些数字直接变成角色的台词燃料

### 2. 把"接口"翻译成"人话"

官方文案是营销体（一行一个卖点 + emoji）。角色要把它转成**一个人的第一人称叙述**：

| 接口原文 | 角色台词 |
|---|---|
| 丝巾…10月22日10:45开售 / 更有APP专享积分抢先购10:40开抢 | "丝巾昨儿上午十点四十五开的售，帽子早没了" |
| 韩式烟熏芝士风味酱 / 浓郁芝香邂逅韩式香辛风味 | "后厨那锅烟熏芝士，闻久了衣服上全是味" |
| 龙焰鸡腿堡三件套 26.9 元起 | "就是辣——你能吃辣吗？不能吃我給你换" |

这不是"读接口"，这是**用接口的数据演一个人**。这个转译层是本项目的核心工作量所在。

### 3. 为后续变现预留了干净的接口

Version 1.0 不推销，但**技术路径已经铺好**：

- 需要推荐时 → `query-meals` + `query-promotions`
- 需要报价时 → `calculate-price`
- 需要转化时 → `create-order`（返回 H5 支付页）

也就是说，1.1/1.2 不需要重构，只需要在合适的情节点上接上对应工具。**先做人，再做买卖**——这个顺序是产品设计，不是技术妥协。

---

## 六、真实调用凭证

开发过程中的全部调用均留有 `traceId` 记录，可按需复核。示例：

| 工具 | traceId |
|---|---|
| `campaign-calendar` | `87c54dd21c51ab5eb1325fa1f06e1268` |

> 本项目在开发阶段完成过完整的真实下单链路验证（`create-order` 生成待支付订单并返回 H5 支付链接），但 Version 1.0 的 Skill 不包含该能力。
