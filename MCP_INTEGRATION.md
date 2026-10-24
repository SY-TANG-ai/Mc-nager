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

**接入方式**

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

## 二、调用的 Tool 
（只是列）
 `order-list`  - 历史点餐订单
 `mall-order-list` - 历史商城订单
 `campaign-calendar` — 活动日历
 `now-time-info` — 当前时间信息



## 三、调用流程

```
skill唤起

调用order-list，通过餐品订单了解顾客就餐偏好。

调用mall-order-list，通过商城订单了解顾客购买偏好。


调用campaign-calendar，根据最近两周的麦当劳活动日历。丰富角色的背景信息。

调用now-time-info，根据当前时间信息，让对话更真实。
组装开场

```

```
当用户要求实际的麦当劳服务时，会根据用户要求调用对应的mcd-mcp工具。

```


---

## 四、业务价值

### 1. 用户历史
更懂用户，聊得来
为后续个性化推荐提供数据

### 2. 角色历史
更真实，有话可讲。

### 3. 自由使用所有mcp工具
可以根据用户要求，自由调用所有mcp工具。

---

## 五、后续展望
### TODO

【】 不只是餐厅经理，也可以是客户经理。如果增加对应接口，可以上传用户对餐品/活动/商品的偏好信息，为新品开发提供参考。