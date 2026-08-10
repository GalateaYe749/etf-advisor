# ETF Advisor · 长线+短线双策略投资系统

基于东方财富/新浪等数据源的 ETF 智能分析与模拟交易平台。支持 22 只 ETF（A股宽基/行业/跨境/商品）的长线定投与短线波段双策略，包含技术指标计算、AI 深度分析、持仓个性化建议、策略回测与参数优化。

## 功能概览

| 模块 | 说明 |
|---|---|
| ETF 看板 | 22 只 ETF 实时行情、净值走势、技术指标（MA/RSI/MACD/布林）、短线买卖信号 |
| 每日报告 | 当天市场概况 + 长线持仓盈亏 + 定投建议（具体金额）+ 短线信号 |
| 长线仓位 | 模拟组合持仓明细、资金曲线、个性化操作建议 |
| 短线仓位 | 买入/卖出观察信号、评分和触发原因 |
| AI 分析 | 四时段深度分析（晨间宏观/午间/收盘/晚间反思） |
| 回溯检验 | 持仓反馈实验室：真实持有 vs 如果买了别的会怎样 |
| 进化引擎 | 192 组参数交叉验证，找出最优止损/止盈/定投组合 |
| 投资百科 | 常见金融术语解释 |

## 架构

```
etf-advisor/
├── www/                    # 网站根目录（部署到 GitHub Pages）
│   ├── index.html          # 单页应用（纯前端渲染）
│   └── data/               # JSON 数据文件（由脚本生成）
│       ├── etf-data.json        # 22 只 ETF 完整数据
│       ├── indices.json         # 5 个大盘指数
│       ├── news.json            # 财经新闻
│       ├── simulated-portfolio.json  # 模拟组合
│       ├── daily-report.json    # 每日报告
│       ├── advice.json          # 分仓建议
│       ├── insights.json        # 投资洞察
│       ├── ai-analysis.json     # AI 四时段分析
│       ├── history.json         # 历史净值记录
│       ├── meta.json            # 元数据
│       ├── backtest.json        # 回溯检验
│       ├── evolution.json       # 进化引擎参数优化
│       ├── glossary.json        # 投资百科
│       └── etf-news-analysis.json  # 每只ETF的新闻情绪分析
│
├── scripts/                # 后端脚本
│   ├── pipeline.mjs        # 主数据管线（净值/指数/新闻/技术指标/模拟组合/每日报告）
│   ├── intraday.mjs        # 盘中实时数据更新（场内价+指数）
│   ├── healthcheck.mjs     # 全站13项健康检查+自愈
│   ├── backtest.py         # 回溯检验引擎
│   ├── evolve.py           # 进化引擎参数优化器
│   ├── deploy.sh           # 一键部署（pipeline + Git push）
│   └── enrich-news.py      # 新闻增强
│
└── MEMORY.md               # 系统知识库（数据源特性、配置说明）
```

## 快速开始

### 1. 克隆仓库

```bash
git clone git@github.com:GalateaYe749/etf-advisor.git
cd etf-advisor
```

### 2. 安装依赖

```bash
# Node.js >= 18
# 无需 npm install — 脚本使用 Node 内置模块（fs/path/child_process）

# Python >= 3.9（用于回溯和进化引擎）
pip install none  # 无额外依赖，使用标准库
```

### 3. 配置

无需额外配置文件。如果需要修改跟踪的 ETF 列表或策略参数，编辑 `scripts/pipeline.mjs` 顶部的 `CONFIG` 对象：

```js
const CONFIG = {
  trackedETFs: [
    { code: '510300', name: '沪深300ETF', type: '宽基', tradeMode: 'long_only' },
    // ... 添加或删除
  ],
  sim: {
    totalCash: 10000,        // 初始本金
    longRatio: 0.70,         // 长线比例
    shortRatio: 0.30,        // 短线比例
    longRules: {
      monthlyInvest: 700,    // 月定投上限
      maxPerETFPerMonth: 350,// 单只每月上限
    },
    shortRules: {
      stopLoss: -0.05,       // 止损 -5%
      takeProfit: 0.08,      // 止盈 +8%
      maxSinglePosition: 0.5,// 单笔最多占短线资金50%
    },
  },
};
```

### 4. 运行脚本

```bash
# 完整数据管线（净值+指数+新闻+技术指标+模拟组合+每日报告，约3-5分钟）
cd ~/projects/etf-advisor
node scripts/pipeline.mjs

# 盘中实时更新（指数+ETF场内实时价，约30秒）
node scripts/intraday.mjs

# 数据健康检查
node scripts/healthcheck.mjs

# 回溯检验（对比你的持仓 vs 其他ETF）
python3 scripts/backtest.py

# 进化引擎（192组参数交叉测试）
python3 scripts/evolve.py

# 一键部署（pipeline + git push）
bash scripts/deploy.sh
```

### 5. 部署网站

```bash
# 网站是纯静态的，直接推送到 GitHub Pages
cd www
git add data/*.json index.html
git commit -m "Update"
git push origin main
```

GitHub Pages 会在 `galateaye749.github.io/etf-advisor/` 自动生效（1-2分钟）。

如果部署到其他服务器（Nginx/Apache），直接把 `www/` 目录作为静态站根目录即可。

### 6. 定时任务（可选）

建议配置 cron 实现四时段自动更新：

| 时间 | 任务 | 命令 |
|---|---|---|
| 8:00 | 深度分析 | `node scripts/pipeline.mjs` |
| 12:00 | 盘中更新 | `node scripts/intraday.mjs` |
| 17:00 | 收盘更新 | `node scripts/intraday.mjs` |
| 22:00 | 晚间净值 | `node scripts/pipeline.mjs` |
| 每30分钟 | 健康监控 | `node scripts/healthcheck.mjs` |

## 数据源

| 数据 | 来源 | 备用 |
|---|---|---|
| ETF 净值 | `fund.eastmoney.com/pingzhongdata/` | `api.fund.eastmoney.com/f10/lsjz` |
| 大盘指数 | `push2.eastmoney.com` | `push2delay.eastmoney.com` |
| ETF 实时价 | `push2delay.eastmoney.com` | — |
| 新闻 | 财联社 / 东财API / 东财HTML / 新浪财经 | 四源独立，全失败保留旧数据 |

## 注意事项

1. **基金净值 T+1 公布**：当天净值最早晚上才出，盘中用 ETF 场内实时价替代
2. **QDII（标普500/纳指）**：净值公布滞后 1-2 天，属正常
3. **pingzhongdata 日期偏差**：该接口的净值日期恒比官方晚 1 天，脚本已自动 +1 天修正
4. **f10 接口每页最多 20 条**：翻页时需注意
5. **push2 高频请求会被限流**：脚本自动切换镜像域名
6. **GitHub Pages 强制 HTTPS**：网站文件必须通过 HTTPS 访问
7. **服务器需要能访问东方财富和新浪的 API**（中国大陆服务器）

## 技术栈

- **前端**：原生 HTML/CSS/JS + Chart.js（图表）
- **后端脚本**：Node.js（数据管线）+ Python（回测/优化）
- **数据存储**：JSON 文件（按日更新，推送到 GitHub Pages）
- **AI 分析**：DeepSeek V4（通过 OpenClaw agentTurn）
- **定时任务**：OpenClaw cron

## 维护者

GalateaYe749 · 自动更新时间 2026-08-10
