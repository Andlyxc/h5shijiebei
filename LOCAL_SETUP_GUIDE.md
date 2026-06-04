# 2026 World Cup Prediction Market - 本地开发指南

## 🚀 快速开始

### 前置要求

- **Node.js** 18+ (推荐 20 LTS)
- **npm** 9+ 或 **yarn**
- **Git**

### 1️⃣ 克隆项目

```bash
git clone https://github.com/dexorynlabs-betting/2026-worldcup-prediction-market.git
cd 2026-worldcup-prediction-market
```

### 2️⃣ 安装依赖

```bash
npm install --legacy-peer-deps
```

> **注意：** 使用 `--legacy-peer-deps` 是必须的，因为项目使用了一些兼容性需求。

### 3️⃣ 配置环境变量

创建 `.env.local` 文件：

```bash
cp .env.example .env.local
```

编辑 `.env.local` 并填入可选的 API 密钥：

```env
# The Odds API (可选) — 用于获取实时赔率
# 注册地址: https://the-odds-api.com (免费版: 500 请求/月)
ODDS_API_KEY=

# Polymarket API (无需密钥) — 用于获取预测市场数据
POLYMARKET_SLUG=fifa-world-cup-2026-winner
```

### 4️⃣ 启动开发服务器

```bash
npm run dev
```

打开浏览器访问：**http://localhost:3000**

---

## 📦 项目脚本

### 开发

```bash
npm run dev          # 启动开发服务器 (Turbopack 加速)
npm run build        # 构建生产版本
npm start            # 运行生产构建
```

### 测试

```bash
npm test             # 运行单次测试
npm run test:watch   # 监听模式运行测试
```

### 数据更新

```bash
# 更新 ELO 评分 (从 eloratings.net)
npm run scrape-elo

# 获取实时赔率 (需要 ODDS_API_KEY)
npm run fetch-odds

# 获取球队缺席名单 (伤病/停赛)
npm run fetch-absences

# 获取历史赛事数据 (用于回测)
npm run fetch-backtest

# 获取实时比赛结果 (ESPN)
npm run fetch-results

# 获取黄牌数据
npm run fetch-cards

# 创建快照运行
npm run snapshot-run
```

---

## 🏗️ 项目结构详解

```
2026-worldcup-prediction-market/
│
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── [locale]/                 # i18n 国际化路由 (en, es)
│   │   │   ├── page.tsx              # 主页 - 模拟器 + 仪表板
│   │   │   ├── demo/                 # 预测市场演示
│   │   │   │   └── page.tsx
│   │   │   ├── backtest/             # 历史回测 (2014/2018/2022)
│   │   │   │   └── page.tsx
│   │   │   ├── methodology/          # 模型文档
│   │   │   │   └── page.tsx
│   │   │   └── api/
│   │   │       └── odds/             # 实时赔率 API 端点
│   │   │           └── route.ts
│   │   └── layout.tsx                # 根布局
│   │
│   ├── components/                   # React 组件库
│   │   ├── demo/                     # 演示组件
│   │   │   ├── DemoHub.tsx           # 主演示容器
│   │   │   ├── MarketsTab.tsx        # 市场交易标签
│   │   │   ├── TicketsTab.tsx        # 门票二级市场
│   │   │   └── PortfolioTab.tsx      # 投资组合管理
│   │   ├── hero/                     # 主页 Hero 区域
│   │   │   ├── HeroGallery.tsx       # 图像库
│   │   │   ├── HeroDemoPromo.tsx     # 演示推广
│   │   │   └── MeshGradient.tsx      # 背景渐变
│   │   ├── layout/                   # 布局组件
│   │   │   ├── Header.tsx            # 页头
│   │   │   ├── HeaderProfile.tsx     # 用户资料卡
│   │   │   ├── Footer.tsx            # 页脚
│   │   │   └── SectionNav.tsx        # 粘性导航
│   │   ├── dashboard/                # 仪表板组件
│   │   │   ├── ChampionCard.tsx
│   │   │   ├── StageMatrix.tsx       # 赛段矩阵
│   │   │   ├── GroupStandings.tsx    # 小组积分
│   │   │   ├── MatchCalendar.tsx     # 比赛日历
│   │   │   └── ...
│   │   └── ...
│   │
│   ├── hooks/                        # 自定义 React Hooks
│   │   ├── useSimulation.ts          # 模拟器状态 + Web Worker
│   │   ├── useDemoWallet.ts          # 虚拟钱包 (localStorage)
│   │   └── ...
│   │
│   ├── lib/                          # 核心库代码
│   │   ├── sim/                      # 模拟引擎 ⭐ 核心
│   │   │   ├── engine.ts            # 主模拟运行器
│   │   │   ├── tournament.ts        # 赛事模拟
│   │   │   ├── group.ts             # 小组赛逻辑
│   │   │   ├── knockout.ts          # 淘汰赛 + 点球
│   │   │   ├── absences.ts          # 缺席处理
│   │   │   ├── rng.ts               # 随机数生成器 (xoshiro128**)
│   │   │   ├── scorers.ts           # 进球分配
│   │   │   ├── worker.ts            # Web Worker 封装
│   │   │   └── types.ts             # TypeScript 类型
│   │   │
│   │   ├── demo/                    # 演示系统
│   │   │   ├── markets.ts           # 市场定价逻辑
│   │   │   ├── tickets.ts           # 门票机制
│   │   │   ├── cache.ts             # 结果缓存
│   │   │   └── flags.ts             # 国旗工具函数
│   │   │
│   │   ├── backtest/               # 回测工具
│   │   │   ├── validator.ts        # 历史验证
│   │   │   └── ...
│   │   │
│   │   └── ...
│   │
│   ├── i18n/                        # 国际化
│   │   ├── messages/
│   │   │   ├── en.json             # 英文翻译
│   │   │   └── es.json             # 西班牙文翻译
│   │   └── routing.ts
│   │
│   └── data/                        # 静态数据
│       ├── teams.json              # 32 支队伍 ELO + 元数据
│       ├── groups.json             # 分组信息
│       ├── bracket.json            # 淘汰赛对阵
│       ├── absences/               # 伤病停赛数据
│       └── odds/                   # 历史赔率
│
├── public/                         # 静态资源
│   ├── banner.png                 # README 标题图
│   ├── worldcup1.jpg              # Hero 图像
│   ├── logo-worldcup2026.webp
│   └── ...
│
├── scripts/                        # 数据脚本
│   ├── scrape-elo.ts              # ELO 爬虫
│   ├── fetch-odds.ts              # 赔率获取
│   ├── fetch-absences.ts          # 缺席数据
│   ├── fetch-backtest.ts          # 历史数据
│   ├── fetch-results.ts           # 实时结果
│   └── ...
│
├── package.json
├── tsconfig.json
├── tailwind.config.mjs            # Tailwind 配置
├── next.config.mjs                # Next.js 配置
└── README.md
```

---

## 🧠 核心概念理解

### 1. 模拟引擎 (`src/lib/sim/`)

**流程：**
1. **初始化** - 加载 32 支队伍的 ELO、群体分组、缺席信息
2. **群体赛** - 每支队伍打 3 场小组赛，按积分排名
3. **淘汰赛** - R32 → R16 → QF → SF → Final
4. **点球** - 平局使用贝叶斯点球模型（非简单硬币）
5. **聚合** - 计算胜率、进球分布、球手助攻

**关键文件：**
- `engine.ts` - 主运行器，聚合结果
- `tournament.ts` - 赛事模拟逻辑
- `rng.ts` - xoshiro128** 随机数生成器（高性能）

### 2. ELO 胜率模型

```
胜率 = 1 / (10^(-dr/400) + 1)
dr = ELO_主队 - ELO_客队 + 主场加成(100分)
```

### 3. Poisson 进球模型

```
λ = clamp(1.30 + 0.18 × (ELO差 + 主场加成) / 100, 0.15, 6.0)
进球数 ~ Poisson(λ)
```

### 4. 预测市场演示 (`src/components/demo/`)

- **虚拟钱包** - $1,000 play-money (localStorage 持久化)
- **市场** - YES/NO 二元市场，价格基于模拟概率
- **门票** - 二级市场，支持 face/fair/ask 价格
- **结算** - 随机抽样赛事结果来结算市场

---

## 💡 开发工作流

### 启动开发环境

```bash
npm run dev
```

开发服务器使用 **Turbopack**（快速刷新）：
- 编辑 `.tsx` / `.ts` 文件 → 自动刷新
- 编辑 `data/teams.json` → 需要重启
- 编辑 `src/lib/sim/worker.ts` → 需要重启 (Web Worker)

### 增加新功能的步骤

1. **创建组件** (`src/components/...`)
   ```bash
   touch src/components/dashboard/MyNewCard.tsx
   ```

2. **导入类型** (`src/lib/sim/types.ts`)
   ```typescript
   export interface MyData {
     // 你的数据结构
   }
   ```

3. **在页面中使用**
   ```typescript
   import MyNewCard from '@/components/dashboard/MyNewCard';
   ```

4. **样式** - 使用 Tailwind CSS v4 (OKLCH 调色板)
   ```tsx
   <div className="bg-teal-500 text-white rounded-lg p-4">
     内容
   </div>
   ```

---

## 🔍 调试技巧

### 查看 Web Worker 日志

编辑 `src/lib/sim/worker.ts`，添加日志：

```typescript
// 在 worker 中
self.onmessage = (e) => {
  console.log('[Worker]', e.data); // 不会显示，使用 self.postMessage 代替
  self.postMessage({
    type: 'log',
    message: 'Debug info'
  });
};
```

### 模拟性能分析

在浏览器 DevTools 的 Performance 标签运行模拟：
1. 打开 DevTools → Performance
2. 点击 Record
3. 点击 "Simulate 100K"
4. 点击 Stop

### 检查类型错误

```bash
npx tsc --noEmit
```

---

## 🚢 部署到 Vercel

项目已配置为在 Vercel 上部署：

```bash
# 已从 GitHub 连接到 Vercel
# 每次 push 到 main 分支自动构建和部署
```

也可以手动部署：

```bash
npm install -g vercel
vercel
```

---

## 📚 有用的资源

| 资源 | 链接 |
|------|------|
| ELO 评分 | https://www.eloratings.net/ |
| FIFA 官方赛程 | https://www.fifa.com/en/tournaments/mens/worldcup/canadamexicousa2026 |
| 赔率 API | https://the-odds-api.com |
| Polymarket | https://polymarket.com |
| Next.js 文档 | https://nextjs.org/docs |
| Tailwind CSS | https://tailwindcss.com |

---

## ❓ 常见问题

### Q: 为什么需要 `--legacy-peer-deps`?
A: 项目使用了一些较旧的依赖版本，需要绕过 npm 的对等依赖检查。

### Q: 模拟速度有多快？
A: 
- Node.js: ~35K 模拟/秒
- Browser Worker: ~15-25K 模拟/秒
- 100K 双通道运行: ~5-10 秒

### Q: 如何修改球队数据？
A: 编辑 `src/data/teams.json`，然后重启开发服务器。

### Q: Web Worker 出错了怎么办？
A: 检查 `src/lib/sim/worker.ts` 中的拼写和导入。Worker 不支持某些浏览器 API（如 localStorage）。

### Q: 如何添加新语言？
A: 
1. 创建 `src/i18n/messages/fr.json`
2. 编辑 `src/i18n/routing.ts` 添加 'fr' 到 locales
3. 翻译所有字符串

---

## 🤝 下一步

准备好贡献了？请查看 **[CONTRIBUTION_GUIDE.md](./CONTRIBUTION_GUIDE.md)**！
