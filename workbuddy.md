# WorkBuddy 开发过程记录（workbuddy.md）

> 本文件记录「一人食控卡助手」Skill 的关键开发过程，用于参赛证明：**本作品全程基于 WorkBuddy 完成**。

## 1. 任务背景
参加「麦当劳程序员创意开发大赛」，目标基于麦当劳 MCP 做一个 WorkBuddy Skill「一人食控卡助手」：
用户一句话描述需求（人数 / 卡路里 / 预算 / 位置），助手完成「查店 → 拉菜单营养 → 查券(只查) → 核价 → 给 2-3 套方案」，且**只查询不写操作**。

需交付：`skill/SKILL.md`、`README.md`、`MCP_INTEGRATION.md`、`mcp-config.example.json`、`workbuddy.md`；`CONTEST_DECLARATION.md` 由用户自行从官方仓库复制，不修改。

## 2. 用 WorkBuddy 做的事

### 2.1 工具盘点与只读确认（关键前置）
- 通过 WorkBuddy 的 **ToolSearch** 加载 `mcd-mcp` 相关工具 schema，逐一核对参数。
- 将 35 个 mcd-mcp 工具按读写分类：
  - ✅ 只读查询类（本 Skill 使用）：`query-nearby-stores`、`delivery-query-stores`、`delivery-query-addresses`、`list-nutrition-foods`、`query-meals`、`query-meal-detail`、`query-store-coupons`、`query-my-coupons`、`available-coupons`、`query-promotions`、`calculate-price`、`now-time-info`。
  - ⛔ 写操作类（本 Skill 禁止）：`create-order`、`cancel-order`、`auto-bind-coupons`、`mall-create-order`、`party-order-create`、`draw-lottery`、`delivery-create-address`。
- 明确「优惠券只查不领」「`calculate-price` 仅做价格模拟」的安全边界。

### 2.2 生成交付文件
全部文件由 WorkBuddy 在 Agent 模式下直接撰写到 `D:/mcd-calorie-helper/`：
- `skill/SKILL.md`：技能定义，含触发词、硬性约束、允许工具表、`beType/orderType` 映射、六步工作流、输出格式与示例。
- `README.md`：项目介绍、安装步骤（MCP 配置 + Skill 安装）、使用示例、目标用户、文件结构、安全说明。
- `MCP_INTEGRATION.md`：工具清单（读写表）、调用流程图、业务价值。
- `mcp-config.example.json`：仅含 `${...}` 环境变量占位符，无任何真实 Token。
- `workbuddy.md`：本记录。

### 2.3 关键决策（遵守用户纪律）
- **不瞎猜**：门店 / 场景 / 卡路里 / 预算缺失时，要求一次问清，而非默认假设。
- **只做必要修改**：仅新增交付文件，未改动任何现有代码或账户。
- **敏感信息隔离**：配置示例只用占位符，真实授权信息不落盘、不写入回复。

## 3. 交付物清单
| 文件 | 作用 |
| --- | --- |
| `skill/SKILL.md` | Agent 调用麦当劳 MCP 的规范 |
| `README.md` | 项目说明与安装指南 |
| `MCP_INTEGRATION.md` | MCP 工具 / 流程 / 价值 |
| `mcp-config.example.json` | 无密钥占位配置 |
| `workbuddy.md` | 开发过程记录（本文件） |
| `CONTEST_DECLARATION.md` | 大赛声明（用户自行复制，本工具未改动） |

## 4. 结论
本作品从工具盘点、约束确认到 5 个文件生成，均在 WorkBuddy 内完成，遵循「只读查询、只查不领、不下单不改账户」的安全设计，可直接作为大赛参赛 Skill 提交。
