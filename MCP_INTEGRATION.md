# MCP 集成说明（MCP_INTEGRATION）

本文档说明「一人食控卡助手」如何使用麦当劳 MCP（mcd-mcp），覆盖**用了哪些工具、调用流程、业务价值**，并明确只读边界。

## 1. 使用的 MCP 工具

mcd-mcp 共暴露 35 个工具，按读写性质分为两类。本 Skill **只使用只读查询类**。

### 1.1 本 Skill 使用的只读工具（✅）
| 工具 | 类别 | 在本 Skill 的作用 | 关键入参 |
| --- | --- | --- | --- |
| `query-nearby-stores` | 门店·读 | 到店 / 得来速查附近营业门店 | `searchType=2`,`city`,`keyword`,`beType` |
| `delivery-query-stores` | 门店·读 | 外送按地址查可配送门店 | `beType=2/6`,`addressId` |
| `delivery-query-addresses` | 地址·读 | 读取用户已有配送地址（不新建） | — |
| `list-nutrition-foods` | 营养·读 | 真实营养成分（能量等），卡路里数据源 | — |
| `query-meals` | 菜单·读 | 指定门店餐品列表（含 `productCode`、价格） | `storeCode`,`orderType`,`beType` |
| `query-meal-detail` | 菜单·读 | 套餐组成 / 可特调标记 | `code`,`storeCode`,`orderType`,`beType` |
| `query-store-coupons` | 券·读 | 指定门店 + 订单类型可用券 | `storeCode`,`orderType`,`beType` |
| `query-my-coupons` | 券·读 | 用户卡包已有券（展示用） | `page`,`pageSize` |
| `available-coupons` | 券·读 | 当前可领取麦麦省券（仅列举） | — |
| `query-promotions` | 促销·读 | 团餐满减 / 满折规则原始数据 | `storeCode`,`beCode`,`beType=6` |
| `calculate-price` | 计价·读 | 计算商品总价（含优惠**模拟**） | `items[]`,`couponId`(可选),`storeCode` |
| `now-time-info` | 时间·读 | 当前时间，判断营业 / 预约 | — |

### 1.2 明确禁止调用的写操作工具（⛔）
| 工具 | 风险 |
| --- | --- |
| `create-order` / `mall-create-order` / `party-order-create` | 创建订单（资金 / 履约） |
| `cancel-order` | 取消订单 |
| `auto-bind-coupons` | 自动绑券（改账户资产） |
| `draw-lottery` | 抽奖（消耗权益） |
| `delivery-create-address` | 新建配送地址（改账户） |

> 约束在 `skill/SKILL.md` 中以「硬性约束」固化，Agent 在任何情况下不得越界。

## 2. 调用流程

```
用户一句话
   │  解析：人数 / 场景 / 卡路里上限 / 预算 / 位置
   ▼
[1] 选场景定 beType/orderType
   │  到店(beType=1) 或 麦乐送(beType=2)
   ▼
[2] 查门店（只读）
   │  到店 → query-nearby-stores(searchType=2, city, keyword, beType=1)
   │  外送 → delivery-query-addresses → delivery-query-stores(beType=2, addressId)
   │  取 storeCode（及 beCode）
   ▼
[3] 拉菜单 + 营养（只读）
   │  list-nutrition-foods  → 卡路里基准
   │  query-meals(storeCode, orderType, beType) → productCode + 价格
   │  按名称对齐「卡路里 ↔ 价格 ↔ productCode」
   ▼
[4] 查券（只读·只查不领）
   │  query-store-coupons + query-my-coupons + available-coupons
   │  整理「可用券清单」
   ▼
[5] 核价（只读·模拟）
   │  calculate-price(items=[{productCode, quantity}], couponId?)
   │  结果 ÷100 得「元」
   ▼
[6] 产出 2-3 套方案
   │  每套：商品 / 卡路里 / 价格 / 用了哪张券(仅展示)
   │  标注「仅查询，未下单未领券」
```

### 关键传参规则（避免报错）
- 到店自取(`beType=1`)：`query-meals` / `calculate-price` 等**不要传 `beCode`**。
- 得来速 / 外送 / 团餐：`beCode` 必传，来源为 `query-nearby-stores` 或 `delivery-query-stores` 返回值。
- `calculate-price` 返回价格单位为「分」，展示前必须 ÷100。
- 优惠券在 `calculate-price` 中传 `couponId` 仅做**价格模拟**，不会真正绑券。

## 3. 业务价值

1. **真实数据驱动**：卡路里取自官方营养接口，价格取自官方计价接口，方案可信、可落地，避免「凭感觉搭配」导致的超标。
2. **优惠透明但不越权**：把可用券、卡包券、可领券都列出来帮用户省钱，但绝不自动领券 / 绑券，尊重用户对账户的完全控制权。
3. **决策降本**：把「查店 → 看营养 → 比价 → 用券」四步压缩成一句话，显著降低用户的决策成本与时间成本。
4. **安全合规的 MCP 范式**：以「只读查询」为硬性边界，为大赛提供一个**可复用、低风险**的 MCP Skill 设计范本——证明 MCP 既能创造体验价值，也能守住操作安全。
5. **场景可扩展**：当前聚焦「一人食控卡」，同一套只读工具链可平滑扩展到「家庭套餐搭配」「团餐满减优化」等场景。
