# 竞猜功能实现指南

本指南将教你如何为 2026 World Cup Prediction Market 项目添加完整的竞猜/预测市场功能。

## 📋 目录

1. [功能概述](#功能概述)
2. [核心架构](#��心架构)
3. [逐步实现](#逐步实现)
4. [数据结构](#数据结构)
5. [API 集成](#api-集成)
6. [高级功能](#高级功能)

---

## 🎯 功能概述

### 什么是竞猜功能？

竞猜功能允许用户在虚拟市场上交易"YES/NO"二元期权，基于 Monte Carlo 模拟的概率定价。

### 核心特性

✅ **虚拟钱包** - $1,000 play-money，localStorage 持久化  
✅ **三类市场** - Winner (冠军) | Group Winner (小组冠军) | H2H (对阵)  
✅ **实时交易** - 买入/卖出 YES/NO  
✅ **投资组合** - 追踪开放头寸和收益  
✅ **市场结算** - 基于随机抽样的赛事结果  
✅ **国际化** - 支持英文和西班牙文  

### 用户流程

```
1. 用户运行模拟 (100K sims)
   ↓
2. 系统生成市场 (基于模拟概率)
   ↓
3. 用户用 $1,000 虚拟资金交易
   ↓
4. 用户点击"Settle Markets"
   ↓
5. 系统随机抽样一个赛事结果
   ↓
6. 根据结果结算市场，更新收益
```

---

## 🏗️ 核心架构

### 三层设计

```
┌─────────────────────────────────────┐
│  UI 层 (Components)                  │
│  MarketsTab, PortfolioTab           │
│  DemoHub, TicketsTab                │
└──────────────────┬──────────────────┘
                   │
┌──────────────────▼──────────────────┐
│  业务逻辑层 (Hooks + Lib)            │
│  useDemoWallet                      │
│  markets.ts, tickets.ts             │
└──────────────────┬──────────────────┘
                   │
┌──────────────────▼──────────────────┐
│  数据层 (localStorage + types)       │
│  DemoWalletState                    │
│  DemoMarket, DemoPosition           │
└─────────────────────────────────────┘
```

### 关键文件

```
src/
├── components/demo/
│   ├── DemoHub.tsx              ← 主容器
│   ├── MarketsTab.tsx           ← 市场交易 UI
│   ├── PortfolioTab.tsx         ← 投资组合 UI
│   ├── TicketsTab.tsx           ← 门票交易 UI
│   └── DemoTeamFlags.tsx        ← 队旗显示
│
├── hooks/
│   └── useDemoWallet.ts         ← 钱包状态管理
│
├── lib/demo/
│   ├── markets.ts               ← 市场生成/结算逻辑
│   ├── tickets.ts               ← 门票机制
│   ├── types.ts                 ← TypeScript 定义
│   ├── cache.ts                 ← 缓存管理
│   └── flags.ts                 ← 国旗工具函数
```

---

## 🔧 逐步实现

### 第 1 步：定义类型 (`src/lib/demo/types.ts`)

```typescript
// 市场类型
export type MarketType = 'winner' | 'group_winner' | 'h2h';

// 单个市场
export interface DemoMarket {
  id: string;
  type: MarketType;
  title: string;
  subtitle: string;
  yesPrice: number; // 0.01 - 0.99
  
  // 根据市场类型的可选字段
  teamId?: string;
  groupId?: string;
  slotKey?: string;
  yesTeamId?: string;
  homeTeamId?: string;
  awayTeamId?: string;
}

// 用户头寸
export interface DemoMarketPosition {
  id: string;
  marketId: string;
  side: 'yes' | 'no'; // 买哪一方
  shares: number;     // 购买的份数
  avgPrice: number;   // 平均购买价格
  createdAt: number;  // 时间戳
  settled?: boolean;
  payout?: number;    // 结算后的收益
}

// 钱包状态
export interface DemoWalletState {
  balance: number;           // 现金
  positions: DemoMarketPosition[]; // 开放头寸
  tickets: DemoTicketHolding[];
  lastSettledAt: number | null;
  settlementLabel: string | null;
}
```

### 第 2 步：生成市场 (`src/lib/demo/markets.ts`)

```typescript
// 从模拟结果生成市场
export function buildDemoMarkets(
  result: SerializedResult,
  locale: 'en' | 'es' = 'en'
): DemoMarket[] {
  const markets: DemoMarket[] = [];
  const N = result.numSimulations;

  // 1. Winner 市场 - 前 12 支概率最高的队伍
  const topTeams = result.teams
    .map((t, i) => ({
      id: t.id,
      prob: result.stageCounts.champion[i] / N,
    }))
    .sort((a, b) => b.prob - a.prob)
    .slice(0, 12);

  for (const team of topTeams) {
    markets.push({
      id: `winner-${team.id}`,
      type: 'winner',
      title: `${team.name} wins the World Cup`,
      subtitle: 'Outright winner market',
      yesPrice: Math.max(0.01, Math.min(0.99, team.prob)),
      teamId: team.id,
    });
  }

  // 2. Group Winner 市场 - 每个小组的冠军
  for (const [letter, teamIds] of Object.entries(GROUPS)) {
    const best = teamIds.reduce((a, b) =>
      getGroupWinnerProb(result, a) > getGroupWinnerProb(result, b)
        ? a
        : b
    );
    markets.push({
      id: `group-${letter}-${best}`,
      type: 'group_winner',
      title: `${best} wins Group ${letter}`,
      subtitle: `Group ${letter}`,
      yesPrice: getGroupWinnerProb(result, best),
      teamId: best,
      groupId: letter,
    });
  }

  // 3. H2H 市场 - 前 10 场预期出现次数最多的对阵
  const fixtures = Array.from(result.fixtures.values())
    .sort((a, b) => {
      const freqA = a.winsHome + a.winsAway + a.draws;
      const freqB = b.winsHome + b.winsAway + b.draws;
      return freqB - freqA;
    })
    .slice(0, 10);

  for (const fix of fixtures) {
    markets.push({
      id: `h2h-${fix.home}-${fix.away}`,
      type: 'h2h',
      title: `${fix.home} beats ${fix.away}`,
      subtitle: fix.stage.toUpperCase(),
      yesPrice: fix.winsHome / (fix.winsHome + fix.winsAway + fix.draws),
      homeTeamId: fix.home,
      awayTeamId: fix.away,
    });
  }

  return markets;
}
```

### 第 3 步：管理钱包 (`src/hooks/useDemoWallet.ts`)

```typescript
export function useDemoWallet() {
  const [wallet, setWallet] = useState<DemoWalletState>(() =>
    defaultWalletState() // $1,000 初始资金
  );

  // 持久化到 localStorage
  useEffect(() => {
    localStorage.setItem(DEMO_STORAGE_KEY, JSON.stringify(wallet));
  }, [wallet]);

  // 买入 YES/NO
  const buySide = useCallback(
    (market: DemoMarket, side: 'yes' | 'no', amount: number) => {
      const price = marketPriceForSide(market, side);
      const shares = amount / price;

      setWallet((w) => ({
        ...w,
        balance: w.balance - amount,
        positions: [
          ...w.positions,
          {
            id: newId(),
            marketId: market.id,
            side,
            shares,
            avgPrice: price,
            createdAt: Date.now(),
          },
        ],
      }));
    },
    []
  );

  // 卖出头寸
  const sellPosition = useCallback(
    (positionId: string, market: DemoMarket) => {
      setWallet((w) => {
        const pos = w.positions.find((p) => p.id === positionId);
        if (!pos) return w;

        const currentPrice = marketPriceForSide(market, pos.side);
        const proceeds = pos.shares * currentPrice;

        return {
          ...w,
          balance: w.balance + proceeds,
          positions: w.positions.filter((p) => p.id !== positionId),
        };
      });
    },
    []
  );

  // 结算市场
  const settleMarkets = useCallback(
    (result: SerializedResult, sample: SampleSim, locale: 'en' | 'es') => {
      setWallet((w) => {
        let balance = w.balance;
        const positions = w.positions.map((p) => {
          // 根据 sample 计算是否赢了
          const yesWon = resolveMarketYes(
            getMarketById(p.marketId),
            sample,
            result.teams
          );
          const won = p.side === 'yes' ? yesWon : !yesWon;
          const payout = won ? p.shares * 1 : 0; // 赢则获利 1:1
          balance += payout;

          return {
            ...p,
            settled: true,
            payout,
          };
        });

        return {
          ...w,
          balance,
          positions,
          lastSettledAt: Date.now(),
        };
      });
    },
    []
  );

  return {
    wallet,
    buyYes: (m, amt) => buySide(m, 'yes', amt),
    buyNo: (m, amt) => buySide(m, 'no', amt),
    sellPosition,
    settleMarkets,
    // ... 其他方法
  };
}
```

### 第 4 步：构建 UI (`src/components/demo/MarketsTab.tsx`)

```tsx
interface Props {
  markets: DemoMarket[];
  result: SerializedResult;
  wallet: Wallet;
}

export function MarketsTab({ markets, result, wallet }: Props) {
  const [filter, setFilter] = useState<'all' | MarketType>('all');
  const [amounts, setAmounts] = useState<Record<string, string>>({});

  const filtered = useMemo(
    () => (filter === 'all' ? markets : markets.filter(m => m.type === filter)),
    [markets, filter]
  );

  return (
    <div className="space-y-6">
      {/* 过滤按钮 */}
      <div className="flex gap-2">
        {(['all', 'winner', 'group_winner', 'h2h'] as const).map(f => (
          <button
            key={f}
            onClick={() => setFilter(f)}
            className={filter === f ? 'bg-gold' : 'border'}
          >
            {t(`filter_${f}`)}
          </button>
        ))}
      </div>

      {/* 市场卡片 */}
      <div className="grid gap-3 lg:grid-cols-2">
        {filtered.map(market => {
          const amt = parseFloat(amounts[market.id] ?? '25');
          const noPrice = 1 - market.yesPrice;

          return (
            <article key={market.id} className="rounded-2xl border p-4">
              <h3>{market.title}</h3>
              
              {/* 价格显示 */}
              <div className="flex justify-between">
                <div>
                  <p>YES</p>
                  <p className="text-gold">
                    {(market.yesPrice * 100).toFixed(1)}¢
                  </p>
                </div>
                <div>
                  <p>NO</p>
                  <p className="text-red-500">
                    {(noPrice * 100).toFixed(1)}¢
                  </p>
                </div>
              </div>

              {/* 下注金额输入 */}
              <input
                type="number"
                value={amounts[market.id] ?? '25'}
                onChange={e => setAmounts(a => ({
                  ...a,
                  [market.id]: e.target.value,
                }))}
                placeholder="$25"
              />

              {/* 买入按钮 */}
              <div className="flex gap-2">
                <button
                  onClick={() => {
                    wallet.buyYes(market, amt);
                    fireConfetti();
                  }}
                  disabled={wallet.wallet.balance < amt}
                  className="flex-1 bg-gold"
                >
                  Buy YES
                </button>
                <button
                  onClick={() => {
                    wallet.buyNo(market, amt);
                    fireConfetti();
                  }}
                  disabled={wallet.wallet.balance < amt}
                  className="flex-1 border border-red-500"
                >
                  Buy NO
                </button>
              </div>
            </article>
          );
        })}
      </div>

      {/* 开放头寸列表 */}
      {wallet.wallet.positions.filter(p => !p.settled).length > 0 && (
        <section className="border rounded-2xl p-4">
          <h3>Open Positions</h3>
          <ul className="space-y-2">
            {wallet.wallet.positions
              .filter(p => !p.settled)
              .map(pos => {
                const m = markets.find(m => m.id === pos.marketId);
                const mark = pos.shares * marketPriceForSide(m, pos.side);
                const cost = pos.shares * pos.avgPrice;
                const pnl = mark - cost;

                return (
                  <li key={pos.id} className="flex justify-between p-2 border">
                    <div>
                      <span>{m?.title}</span>
                      <span className={
                        pos.side === 'yes' ? 'text-gold' : 'text-red-500'
                      }>
                        {pos.side.toUpperCase()}
                      </span>
                    </div>
                    <div>
                      <span className="font-mono">
                        {pos.shares.toFixed(1)} @ 
                        {(pos.avgPrice * 100).toFixed(1)}¢
                      </span>
                      <span className={pnl >= 0 ? 'text-green' : 'text-red'}>
                        {pnl >= 0 ? '+' : ''}${pnl.toFixed(2)}
                      </span>
                    </div>
                    <button onClick={() => wallet.sellPosition(pos.id, m)}>
                      Sell
                    </button>
                  </li>
                );
              })}
          </ul>
        </section>
      )}
    </div>
  );
}
```

---

## 📊 数据结构

### DemoMarket（市场）

```typescript
interface DemoMarket {
  id: string;                    // 唯一标识
  type: 'winner' | 'group_winner' | 'h2h';
  title: string;                 // 市场标题
  subtitle: string;              // 副标题
  yesPrice: number;              // YES 的价格 (0-1)
  
  // 可选字段（根据类型）
  teamId?: string;               // winner / group_winner 的队伍 ID
  groupId?: string;              // group_winner 的小组字母
  homeTeamId?: string;           // h2h 的主队 ID
  awayTeamId?: string;           // h2h 的客队 ID
  slotKey?: string;              // h2h 的比赛槽位
}
```

### 价格计算

```typescript
// YES 的价格 = 队伍赢的概率
yesPrice = probability
noPrice = 1 - yesPrice

// 计算份数
shares = investment / price
// 例：投入 $25，YES 价格 0.30，则获得 83.33 份
shares = 25 / 0.30 = 83.33

// 结算时收益
payout = shares * 1 = 83.33 (如果赢了)
payout = 0 (如果输了)

// 损益
pnl = current_mark_price - cost
```

### 市场结算逻辑

```typescript
function resolveMarketYes(
  market: DemoMarket,
  sample: SampleSim,  // 随机抽样的赛事结果
  teams: Team[]
): boolean {
  if (market.type === 'winner') {
    // 检查样本中的冠军是��与市场的 teamId 匹配
    const champion = teams[sample.champion];
    return champion.id === market.teamId;
  }

  if (market.type === 'group_winner') {
    // 计算样本中该小组的冠军
    const groupChampion = calculateGroupWinner(sample, market.groupId);
    return groupChampion === market.teamId;
  }

  if (market.type === 'h2h') {
    // 检查样本中的对阵结果
    const match = findMatchInSample(sample, market.slotKey);
    if (!match) return null;
    const homeWon = match.goalsHome > match.goalsAway;
    return market.yesTeamId === (homeWon ? market.homeTeamId : market.awayTeamId);
  }

  return false;
}
```

---

## 🔌 API 集成

### 集成实时赔率

```typescript
// src/lib/demo/odds.ts

export async function fetchPolymarketOdds(
  marketSlug: string
): Promise<{ yes: number; no: number }> {
  const response = await fetch(
    `https://api.polymarket.com/markets/${marketSlug}`
  );
  const data = await response.json();
  
  // 使用 Polymarket 的实时价格而不是模拟概率
  return {
    yes: data.prices[0],
    no: data.prices[1],
  };
}
```

### 集成 ESPN 实时结果

```typescript
// src/scripts/fetch-live-results.ts

export async function fetchLiveResults() {
  const response = await fetch(
    'https://site.api.espn.com/en/site/api/site/worldcup/events'
  );
  const data = await response.json();
  
  // 更新市场基于实时比分
  for (const event of data.events) {
    if (event.competitions[0].status.type.completed) {
      const score = event.competitions[0].competitors;
      console.log(`${score[0].displayName} ${score[0].score} - ${score[1].score} ${score[1].displayName}`);
    }
  }
}
```

---

## 🚀 高级功能

### 1. 套利检测

```typescript
// 检测价格不一致的套利机会
function findArbitrage(markets: DemoMarket[]): string[] {
  const arbs: string[] = [];
  
  for (const market of markets) {
    const yesPrice = market.yesPrice;
    const noPrice = 1 - yesPrice;
    
    // 理想情况下 YES + NO = 1.0
    // 如果 < 1.0，存在套利机会
    if (yesPrice + noPrice < 1.0) {
      arbs.push(
        `Arb: ${market.title} (YES: ${(yesPrice*100).toFixed(1)}¢ + NO: ${(noPrice*100).toFixed(1)}¢)`
      );
    }
  }
  
  return arbs;
}
```

### 2. 智能下注建议

```typescript
// 凯利公式：最优下注比例
function kellyFraction(
  probability: number,
  odds: number // 赢的倍数
): number {
  // Kelly: f = (p*odds - (1-p)) / (odds - 1)
  const f = (probability * odds - (1 - probability)) / (odds - 1);
  return Math.max(0, Math.min(f, 0.25)); // 限制在 25% 以内
}

// 使用示例
const prob = 0.60;        // 60% 胜率
const odds = 1 / 0.30;    // 价格 30¢ = 3.33:1 赔率
const fraction = kellyFraction(prob, odds); // 0.18 = 投注 18% 的资金
```

### 3. 风险管理

```typescript
interface RiskMetrics {
  maxDrawdown: number;      // 最大回撤
  sharpeRatio: number;      // 夏普比率
  winRate: number;          // 胜率
  profitFactor: number;     // 利润因子
}

function calculateRiskMetrics(positions: DemoMarketPosition[]): RiskMetrics {
  const settled = positions.filter(p => p.settled);
  const profits = settled.map(p => (p.payout ?? 0) - (p.shares * p.avgPrice));
  
  const totalProfit = profits.reduce((a, b) => a + b, 0);
  const totalWins = profits.filter(p => p > 0).length;
  const winRate = totalWins / settled.length;
  
  const grossProfit = profits.filter(p => p > 0).reduce((a, b) => a + b, 0);
  const grossLoss = Math.abs(profits.filter(p => p < 0).reduce((a, b) => a + b, 0));
  const profitFactor = grossProfit / (grossLoss || 1);
  
  return {
    maxDrawdown: 0, // TODO: 计算最大回撤
    sharpeRatio: 0, // TODO: 计算夏普比率
    winRate,
    profitFactor,
  };
}
```

### 4. 社交分享

```typescript
// 分享投资组合成绩
function sharePortfolio(wallet: DemoWalletState): string {
  const roi = ((wallet.balance - 1000) / 1000) * 100;
  const message = `🏆 I made ${roi.toFixed(1)}% ROI on the World Cup 2026 Prediction Market! 
  
💰 Final balance: $${wallet.balance.toFixed(2)}
🎯 Open positions: ${wallet.positions.filter(p => !p.settled).length}
📊 Settled markets: ${wallet.positions.filter(p => p.settled).length}

Join me and predict the future: https://worldcup2026-prediction-market.vercel.app`;

  return encodeURIComponent(message);
}

// Twitter 分享
const twitterUrl = `https://twitter.com/intent/tweet?text=${sharePortfolio(wallet)}`;

// Telegram 分享
const telegramUrl = `https://t.me/share/url?url=...&text=${sharePortfolio(wallet)}`;
```

---

## 🧪 测试竞猜功能

### 单元测试

```typescript
import { describe, it, expect } from 'vitest';
import { resolveMarketYes, buildDemoMarkets } from '@/lib/demo/markets';

describe('竞猜功能', () => {
  it('应该生成正确数量的市场', () => {
    const result = runSimulations({ numSimulations: 1000 });
    const markets = buildDemoMarkets(result);
    
    expect(markets.length).toBe(12 + 8 + 10); // 12 winner + 8 groups + 10 h2h
  });

  it('市场价格应该在 0-1 之间', () => {
    const result = runSimulations({ numSimulations: 1000 });
    const markets = buildDemoMarkets(result);
    
    for (const market of markets) {
      expect(market.yesPrice).toBeGreaterThanOrEqual(0.01);
      expect(market.yesPrice).toBeLessThanOrEqual(0.99);
    }
  });

  it('应该正确结算市场', () => {
    const sample = result.sampleSims[0];
    const market = markets[0];
    
    const won = resolveMarketYes(market, sample, result.teams);
    expect(won).toBe(true || false);
  });
});
```

### 集成测试

```typescript
// 模拟完整用户流程
it('应该完成完整的交易流程', () => {
  // 1. 生成市场
  const markets = buildDemoMarkets(result);
  
  // 2. 用户买入
  wallet.buyYes(markets[0], 50);
  expect(wallet.wallet.balance).toBe(950);
  expect(wallet.wallet.positions).toHaveLength(1);
  
  // 3. 用户卖出
  wallet.sellPosition(wallet.wallet.positions[0].id, markets[0]);
  expect(wallet.wallet.balance).toBeGreaterThanOrEqual(950);
  
  // 4. 用户结算
  wallet.settleMarkets(result, result.sampleSims[0], 'en');
  expect(wallet.wallet.positions[0].settled).toBe(true);
});
```

---

## 📈 性能优化

### 缓存市场

```typescript
const marketCache = new Map<string, DemoMarket[]>();

export function buildDemoMarketsWithCache(result: SerializedResult): DemoMarket[] {
  const key = `result-${result.numSimulations}`;
  
  if (marketCache.has(key)) {
    return marketCache.get(key)!;
  }
  
  const markets = buildDemoMarkets(result);
  marketCache.set(key, markets);
  
  return markets;
}
```

### 虚拟化长列表

```tsx
import { FixedSizeList } from 'react-window';

export function MarketsTabVirtualized({ markets }: Props) {
  return (
    <FixedSizeList
      height={600}
      itemCount={markets.length}
      itemSize={150}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>
          <MarketCard market={markets[index]} />
        </div>
      )}
    </FixedSizeList>
  );
}
```

---

## 🎯 后续步骤

### 第 1 周：基础实现
- [ ] 定义数据类型
- [ ] 实现市场生成
- [ ] 实现钱包管理
- [ ] 构建基础 UI

### 第 2 周：交易功能
- [ ] 买入/卖出逻辑
- [ ] 头寸追踪
- [ ] 市场结算
- [ ] 收益计算

### 第 3 周��高级功能
- [ ] 实时赔率集成
- [ ] 智能下注建议
- [ ] 社交分享
- [ ] 风险分析

### 第 4 周：优化和测试
- [ ] 性能优化
- [ ] 完整测试覆盖
- [ ] 部署到生产环境
- [ ] 用户反馈

---

## 💡 常见问题

### Q: 如何修改初始资金？
A: 编辑 `src/lib/demo/types.ts`
```typescript
export function defaultWalletState(): DemoWalletState {
  return {
    balance: 10000, // 改成你想要的金额
    positions: [],
    tickets: [],
  };
}
```

### Q: 如何添加新的市场类型？
A: 
1. 在 `MarketType` 中添加新类型
2. 在 `buildDemoMarkets()` 中添加生成逻辑
3. 在 `resolveMarketYes()` 中添加结算逻辑

### Q: 如何持久化用户数据到数据库？
A: 替换 localStorage 为 API 调用
```typescript
// 保存到数据库
async function saveWallet(state: DemoWalletState) {
  await fetch('/api/wallet', {
    method: 'POST',
    body: JSON.stringify(state),
  });
}
```

---

## 📚 相关资源

- [Markets.ts 源代码](./src/lib/demo/markets.ts)
- [Hooks 源代码](./src/hooks/useDemoWallet.ts)
- [UI 组件源代码](./src/components/demo/MarketsTab.tsx)
- [预测市场理论](https://en.wikipedia.org/wiki/Prediction_market)
- [Kelly 公式](https://en.wikipedia.org/wiki/Kelly_criterion)

---

**现在你已经有了完整的竞猜功能实现指南！** 🎉

有任何问题？在 GitHub Issues 中提问或查看 [CONTRIBUTION_GUIDE.md](./CONTRIBUTION_GUIDE.md)！
