# PolyClaw

**为 OpenClaw 设计的 Polymarket 交易技能工具。**

浏览预测市场、在链上执行交易，并使用 LLM 驱动的分析发现对冲机会。通过 Split + CLOB 机制在 Polygon 上实现完整的交易能力。

> **免责声明：** 本软件按原样提供，仅用于教育和实验目的。这不是投资建议。预测市场交易存在亏损风险。此代码未经审计。使用风险自负，仅使用您可以承受损失的资金。

📺 **[观看视频讲解](https://www.youtube.com/watch?v=s_uP802NVTE)**

## 功能特性

### 市场浏览
- `polyclaw markets trending` — 按24小时交易量排序的热门市场
- `polyclaw markets search "关键词"` — 按关键词搜索市场
- `polyclaw market <id>` — 查看市场详情和价格

### 交易执行
- `polyclaw buy <market_id> YES <金额>` — 买入 YES 仓位
- `polyclaw buy <market_id> NO <金额>` — 买入 NO 仓位
- Split + CLOB 执行机制（将 USDC 拆分为 YES+NO，卖出不需要的一方）

### 仓位追踪
- `polyclaw positions` — 列出持仓及实时盈亏
- `polyclaw position <id>` — 查看仓位详情
- 仓位数据本地存储在 `~/.openclaw/polyclaw/positions.json`

### 钱包管理
- `polyclaw wallet status` — 显示地址、POL/USDC.e 余额
- `polyclaw wallet approve` — 设置 Polymarket 合约授权（仅需一次）

### 对冲发现
- `polyclaw hedge scan` — 扫描热门市场寻找对冲机会
- `polyclaw hedge scan --query "主题"` — 扫描匹配查询的市场
- `polyclaw hedge analyze <id1> <id2>` — 分析特定市场对

使用 LLM 驱动的逆否命题逻辑来寻找覆盖组合。仅接受逻辑上必然的蕴含关系——相关性和"可能"的关系会被拒绝。

**覆盖等级：** T1 (≥95%), T2 (90-95%), T3 (85-90%)

## 快速开始

### 1. 安装技能

**方式 A：从 ClawHub 安装（推荐）**

```bash
clawhub install polyclaw
cd ~/.openclaw/skills/polyclaw
uv sync
```

**方式 B：手动安装**

```bash
cp -r polyclaw ~/.openclaw/skills/
cd ~/.openclaw/skills/polyclaw
uv sync
```

### 2. 配置环境变量

在 `openclaw.json` 的 `skills.entries.polyclaw.env` 下添加以下配置：

```json
"polyclaw": {
  "enabled": true,
  "env": {
    "CHAINSTACK_NODE": "https://polygon-mainnet.core.chainstack.com/YOUR_KEY",
    "POLYCLAW_PRIVATE_KEY": "0x...",
    "OPENROUTER_API_KEY": "sk-or-v1-..."
  }
}
```

**密钥获取方式：**
- **Chainstack 节点** — [在 Chainstack 注册](https://console.chainstack.com)（有免费套餐，支持 GitHub、X 或 Google 登录）
- **OpenRouter API 密钥** — [在 OpenRouter 创建密钥](https://openrouter.ai/settings/keys)

**安全警告：** 此钱包中仅保留少量资金。定期将资金转移到安全钱包。

> **需要独立 CLI 使用？** 此技能专为 OpenClaw 设计。如需不依赖 OpenClaw 的独立 CLI 使用，请参考 [polymarket-alpha-bot](https://github.com/chainstacklabs/polymarket-alpha-bot)。

### 3. 首次设置（交易必需）

首次交易前，需设置 Polymarket 合约授权（仅需一次，Gas 费约 0.01 POL）：

```bash
uv run python scripts/polyclaw.py wallet approve
```

这会向 Polygon 提交 6 笔授权交易。每个钱包只需执行一次。

### 4. 运行命令

```bash
# 浏览市场
uv run python scripts/polyclaw.py markets trending
uv run python scripts/polyclaw.py markets search "election"

# 发现对冲机会
uv run python scripts/polyclaw.py hedge scan --limit 10

# 查看钱包并交易
uv run python scripts/polyclaw.py wallet status
uv run python scripts/polyclaw.py buy <market_id> YES 50
```

## 示例提示词

可在 OpenClaw 中使用的自然语言提示：

### 1. 浏览热门市场
```
Polymarket 上有什么热门市场？
```
返回市场 ID、问题、价格和交易量。

### 2. 获取市场详情
```
显示市场 <market_id> 的详情
```
使用 Polymarket URL 中的市场 ID 或上面热门市场响应中的 ID。

返回完整市场信息及 Polymarket 链接。

### 3. 查看钱包状态
```
我的 PolyClaw 钱包余额是多少？
```
显示地址、POL 余额（用于 Gas）和 USDC.e 余额。

### 4. 直接交易
如果您对某个市场有自己的判断：
```
在市场 <market_id> 上买入 $50 的 YES
```
执行 Split + CLOB 流程并记录仓位。

### 5. 对冲发现流程
发现 LLM 分析的套利机会：
```
帮我在 Polymarket 上找一些对冲机会
```
或更具体地：
```
运行 hedge scan limit 10
```
> **注意：** 这需要几分钟时间。技能会获取开放市场并将市场对发送给 LLM 进行逻辑蕴含分析。

查看结果——您会看到覆盖等级（T1 = 95%+，T2 = 90-95%，T3 = 85-90%）以及可以建立对冲仓位的市场对。

### 6. 查看仓位
```
显示我的 PolyClaw 仓位
```
列出持仓及入场价、当前价和盈亏。

### 7. 提前卖出
在市场结算前退出仓位：
```
卖出我在市场 <market_id> 上的 YES 仓位
```
以当前市场价格在 CLOB 订单簿上卖出您的代币。

### 完整流程示例

1. **"Polymarket 上有什么热门市场？"** → 获取市场 ID
2. **"运行 hedge scan limit 10"** → 等待 LLM 分析
3. 查看带覆盖等级的对冲机会
4. **"在市场 abc123 上买入 $25 的 YES"** → 在目标市场建仓
5. **"在市场 xyz789 上买入 $25 的 NO"** → 在覆盖市场建仓
6. **"显示我的 PolyClaw 仓位"** → 验证入场并追踪盈亏

## 环境变量

| 变量 | 是否必需 | 说明 |
|------|---------|------|
| `CHAINSTACK_NODE` | 是（交易） | Polygon RPC URL |
| `OPENROUTER_API_KEY` | 是（对冲） | OpenRouter API 密钥（用于 LLM） |
| `POLYCLAW_PRIVATE_KEY` | 是（交易） | EVM 私钥（十六进制） |
| `HTTPS_PROXY` | 否 | 仅在 CLOB 订单失败时需要（见[故障排除](#clob-订单失败--被-cloudflare-封禁-ip)） |
| `CLOB_MAX_RETRIES` | 否 | CLOB 订单最大重试次数（默认：5） |

## 目录结构

```
polyclaw/
├── SKILL.md                     # OpenClaw 技能清单
├── README.md                    # 英文文档
├── README-zh.md                 # 中文文档（本文件）
├── pyproject.toml               # Python 依赖（uv）
│
├── scripts/
│   ├── polyclaw.py              # CLI 调度器
│   ├── markets.py               # 市场浏览（Gamma API）
│   ├── wallet.py                # 钱包管理
│   ├── trade.py                 # Split + CLOB 执行
│   ├── positions.py             # 仓位追踪 + 盈亏
│   └── hedge.py                 # LLM 对冲发现
│
└── lib/
    ├── __init__.py              # 包标记
    ├── clob_client.py           # py-clob-client 封装
    ├── contracts.py             # CTF ABI + 地址
    ├── coverage.py              # 覆盖率计算 + 等级
    ├── gamma_client.py          # Polymarket Gamma API 客户端
    ├── llm_client.py            # OpenRouter LLM 客户端
    ├── position_storage.py      # 仓位 JSON 存储
    └── wallet_manager.py        # 钱包生命周期
```

## 交易流程

1. **设置授权**（仅需一次）：`polyclaw wallet approve`
2. **执行交易**：`polyclaw buy <market_id> YES 50`
   - 将 $50 USDC.e 拆分为 50 YES + 50 NO 代币
   - 通过 CLOB 卖出 50 NO 代币 → 回收约 $15（按 30¢ 计）
   - 结果：持有 50 YES 代币，净成本约 $35
3. **追踪仓位**：`polyclaw positions`

### 理解拆分机制

Polymarket 使用**条件代币框架（CTF）**。您不能直接"购买 YES 代币"——而是：

1. **拆分**：将 USDC.e 存入 CTF 合约，该合约铸造等量的 YES + NO 代币
2. **卖出不需要的**：通过 CLOB 订单簿卖出您不想要的一方
3. **结果**：您持有所需的仓位，并通过卖出另一方回收了部分成本

**示例**（以 $0.65 购买 YES）：
```
拆分：$2 USDC.e → 2 YES + 2 NO 代币
卖出：2 NO 代币 @ $0.35 → 回收约 $0.70
净额：支付约 $1.30 获得 2 YES 代币（实际价格：$0.65）
```

### CLOB 订单 ID

执行交易时，CLOB 卖出会返回一个**订单 ID**，如：
```
0xc93d6214515b2436feb684854c98d314ad19111d7ab822a9c885d61588d5beaa
```

这**不是区块链交易哈希**——而是链下 Polymarket 订单簿标识符。CLOB 订单在链下匹配，然后批量在链上结算。

**您可以用订单 ID 做什么：**

通过 CLOB API 查询订单详情（需要钱包认证）：
```python
from py_clob_client.client import ClobClient

client = ClobClient("https://clob.polymarket.com", key=private_key, chain_id=137)
creds = client.create_or_derive_api_creds()
client.set_api_creds(creds)

order = client.get_order("0xc93d6214...")
# 返回：id, market, side, price, size_matched, status, created_at 等
```

**API 端点：** `GET https://clob.polymarket.com/data/order/<order_hash>`

**响应字段：** `id`、`market`、`asset_id`、`side`、`price`、`original_size`、`size_matched`、`status`（MATCHED/LIVE/CANCELLED）、`type`（FOK/GTC）、`created_at`、`maker_address`、`associate_trades`

**注意：** CLOB 订单 ID 没有公开的浏览器。要查看您的交易历史，请在 polymarket.com → Portfolio → Activity 连接您的钱包。

## 对冲发现流程

1. **扫描市场**：`polyclaw hedge scan --query "election"`
2. **查看输出**：表格显示 Tier、Coverage、Cost、Target、Cover
3. **分析市场对**：`polyclaw hedge analyze <id1> <id2>`
4. **如有利可图则执行**：手动买入两个仓位

**覆盖等级：**
- **Tier 1（高）：** ≥95% 覆盖率 — 接近套利
- **Tier 2（良好）：** 90-95% — 强对冲
- **Tier 3（中等）：** 85-90% — 尚可但有明显风险
- **Tier 4（低）：** <85% — 投机性（默认过滤）

## Polymarket 合约（Polygon 主网）

| 合约 | 地址 |
|------|------|
| USDC.e | `0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174` |
| CTF | `0x4D97DCd97eC945f40cF65F87097ACe5EA0476045` |
| CTF Exchange | `0x4bFb41d5B3570DeFd03C39a9A4D8dE6Bd8B8982E` |
| Neg Risk CTF Exchange | `0xC5d563A36AE78145C45a50134d48A1215220f80a` |
| Neg Risk Adapter | `0xd91E80cF2E7be2e162c6513ceD06f1dD0dA35296` |

## 故障排除

### "No wallet available"（无可用钱包）
设置 `POLYCLAW_PRIVATE_KEY` 环境变量：
```bash
export POLYCLAW_PRIVATE_KEY="0x..."
```

### "CHAINSTACK_NODE not set"（未设置 CHAINSTACK_NODE）
设置 Polygon RPC URL：
```bash
export CHAINSTACK_NODE="https://polygon-mainnet.core.chainstack.com/YOUR_KEY"
```

### "OPENROUTER_API_KEY not set"（未设置 OPENROUTER_API_KEY）
对冲命令需要此密钥。在 https://openrouter.ai/settings/keys 获取免费密钥：
```bash
export OPENROUTER_API_KEY="sk-or-v1-..."
```

### 对冲扫描找到 0 结果或错误结果
模型质量很重要。默认的 `nvidia/nemotron-nano-9b-v2:free` 效果不错。如果使用其他模型：
- 某些模型会发现虚假相关性（误报）
- 某些模型返回空响应（DeepSeek R1 使用 `reasoning_content`）
- 尝试明确指定 `--model nvidia/nemotron-nano-9b-v2:free`

### "Insufficient USDC.e"（USDC.e 不足）
检查余额——您需要 Polygon 上的 USDC.e（桥接的 USDC）：
```bash
uv run python scripts/polyclaw.py wallet status
```

### "CLOB order failed"（CLOB 订单失败）/ "IP blocked by Cloudflare"（被 Cloudflare 封禁 IP）

Polymarket 的 CLOB API 使用 Cloudflare 保护，会阻止来自许多 IP 的 POST 请求。解决方案是使用带重试逻辑的**轮换住宅代理**。

**推荐设置（IPRoyal 或类似服务）：**
```bash
export HTTPS_PROXY="http://user:pass@geo.iproyal.com:12321"
export CLOB_MAX_RETRIES=10
```

CLOB 客户端会自动使用新 IP 重试，直到找到未被封禁的 IP。通常在 5-10 次尝试内成功。

**替代方案：**
1. **手动卖出** — 您的拆分已成功。去 polymarket.com 卖出代币
2. **使用 `--skip-sell`** — 保留两种代币：`polyclaw buy <id> YES 50 --skip-sell`

### "Approvals not set"（未设置授权）
运行一次性授权设置：
```bash
uv run python scripts/polyclaw.py wallet approve
```

## 安全使用建议

### 必须遵守
1. **仅使用小额资金** — 这是实验性软件
2. **使用专用钱包** — 不要使用主钱包的私钥
3. **定期提款** — 不要在此钱包中长期存放大额资金
4. **保护环境变量** — 确保 `.env` 文件不被泄露

### 推荐做法
1. 考虑使用硬件钱包签名
2. 加密本地仓位存储文件
3. 添加交易金额上限检查
4. 实现交易双重确认机制

## 许可证

Apache 2.0

## 致谢

基于 Chainstack 的 [polymarket-alpha-bot](https://github.com/chainstacklabs/polymarket-alpha-bot)。

- **Chainstack** — Polygon RPC 基础设施
- **Polymarket** — 预测市场平台
- **OpenRouter** — 用于对冲发现的 LLM API
