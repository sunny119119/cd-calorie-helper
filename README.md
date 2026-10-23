# 麦当劳一人食控卡助手（mcd-solo-calorie-helper）

> 麦当劳程序员创意开发大赛参赛作品 · 基于麦当劳 MCP 的 WorkBuddy Skill

一个人想吃麦当劳，又想控制卡路里和钱包？一句话描述需求，助手帮你查附近营业门店、拉真实菜单营养、看可用优惠券（只查不领），并给出 2-3 套「卡得住、钱包也守得住」的方案。

## ✨ 功能特性
- 🔍 **附近门店**：基于位置关键词查到店 / 外送营业门店。
- 🥗 **真实营养**：调用官方营养数据，卡路里来自真实餐品成分，不靠估算。
- 🎟️ **优惠券只查不领**：列举门店可用券、卡包券、可领券，绝不自动绑券。
- 🧮 **精准核价**：用官方计价接口核算每套方案到手价（含优惠模拟）。
- 📋 **2-3 套方案**：每套明确标注总卡路里与总价格，是否符合你的上限一目了然。
- 🔒 **纯查询安全边界**：只做查询与核价，不下单、不领券、不修改账户。

## 🎯 目标用户
- 想减脂 / 控卡但又馋麦当劳的「一人食」用户。
- 对预算敏感、希望用优惠券但不想被自动操作的用户。
- 想快速决策「今天吃啥组合最划算」的忙碌打工人。

## 🚀 安装

### 1. 启用麦当劳 MCP
参考官方大赛仓库，把麦当劳 MCP Server 配置进 WorkBuddy 的 `~/.workbuddy/mcp.json`。
本仓库提供无密钥的占位配置：`mcp-config.example.json`，把里面的 `${...}` 占位符替换成你的大赛授权信息即可（**不要提交真实 Token**）。

```bash
cp mcp-config.example.json ~/.workbuddy/mcp.json
# 然后编辑 ~/.workbuddy/mcp.json，填入你的 MCD_MCP_* 环境变量
```

### 2. 安装 Skill
把 `skill/` 目录放到 WorkBuddy 技能目录之一：
- 用户级：`~/.workbuddy/skills/mcd-solo-calorie-helper/`
- 项目级：`<你的项目>/.workbuddy/skills/mcd-solo-calorie-helper/`

```bash
# 例如（Windows Git Bash）
mkdir -p ~/.workbuddy/skills/mcd-solo-calorie-helper
cp -r skill/* ~/.workbuddy/skills/mcd-solo-calorie-helper/
```

安装后，在对话中直接说需求即可触发，例如：
> 「一个人，想吃麦当劳，控制在 600 卡以内，预算 30 元，附近有店吗」

## 💡 使用示例

**示例 1 — 到店控卡**
```
用户：一个人，想吃麦当劳，控制在 500 卡以内，预算 25 元，上海人民广场附近有店吗
助手：
  门店：人民广场店（营业中）
  方案A  麦香鸡汉堡×1 + 玉米杯×1 + 零度可乐×1  ≈ 480 kcal / ¥22
  方案B  板烧鸡腿堡(去酱)×1 + 苹果片×1          ≈ 460 kcal / ¥24
  （含可用券：满20减3，仅展示，未领取未绑定）
```

**示例 2 — 外送控卡**
```
用户：在家吃，一个人，600 卡以内，30 元，麦乐送能送到吗
助手：先读取你的已有配送地址 → 查可配送门店 → 拉菜单营养 → 列券 → 核价 → 给方案
```

## 📁 文件结构
```
mcd-calorie-helper/
├── skill/
│   └── SKILL.md              # 技能定义（Agent 调用规范）
├── README.md                 # 项目介绍、安装、示例、目标用户
├── MCP_INTEGRATION.md        # MCP 工具清单、调用流程、业务价值
├── mcp-config.example.json   # 无密钥占位配置
├── workbuddy.md              # 本次开发过程记录（证明使用 WorkBuddy）
└── CONTEST_DECLARATION.md    # 大赛声明（从官方仓库复制，勿改）
```

## ⚠️ 安全说明
本 Skill 严格限定在「只读查询」边界内：仅调用门店、菜单营养、优惠券查询、计价等读接口；**不调用任何下单、取消、绑券、抽奖、建地址等写接口**。所有优惠仅做展示与价格模拟，绝不会自动领券或下单。

详见 [MCP_INTEGRATION.md](./MCP_INTEGRATION.md) 与 [skill/SKILL.md](./skill/SKILL.md)。
