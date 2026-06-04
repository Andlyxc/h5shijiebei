# 贡献指南 - 2026 World Cup Prediction Market

欢迎为这个项目做出贡献！本指南将帮助你通过所有可能的方式参与项目开发。

## 📋 贡献方式

### 1. 🐛 报告 Bug

**位置:** GitHub Issues  
**模板:** 使用 "Bug Report" 模板

**提交 Bug 时包含：**
- 清晰的标题（例如："模拟崩溃，当队伍缺席信息丢失时"）
- 重现步骤
- 实际行为 vs 预期行为
- 截图（如适用）
- 环境信息（浏览器、OS、Node 版本）

**示例:**
```
## Bug 描述
模拟器在运行 100K 次模拟时，显示的冠军概率总和不等于 100%

## 重现步骤
1. 打开 http://localhost:3000
2. 设置 100,000 次模拟
3. 点击 "Simulate"
4. 观察仪表板中的冠军概率

## 实际行为
概率总和 = 99.8%

## 预期行为
概率总和应该 = 100%

## 环境
- Browser: Chrome 131
- OS: macOS 14
- Node: v20.10
```

---

### 2. ✨ 功能请求

**位置:** GitHub Discussions 或 Issues  
**标签:** `enhancement`

**提交功能请求时包含：**
- 清晰的用例
- 预期用户流程
- 竞争产品中的类似例子
- 复杂度评估

**示例:**
```
## 功能: 实时比赛跟踪

## 用例
用户想在实际比赛进行中看到模拟器如何更新，
实时了解球队的晋级概率如何变化。

## 建议的实现
- 添加 `/live` 路由
- 每 15 分钟刷新一次 ESPN 数据
- 显示概率 delta 与赛前预测的对比
- 用绿色/红色指示符表示有利/不利
```

---

### 3. 📝 改进文档

**受欢迎的改进:**
- 添加代码注释
- 改进 README 或 methodology
- 添加类型注释
- 创建教程或指南

**提交方式:**
1. Fork 仓库
2. 编辑文档
3. 提交 Pull Request

---

### 4. 🔧 代码贡献

#### 高优先级功能

- [ ] **AI 预测器** - 使用机器学习模型预测未来概率
- [ ] **ELO 自动更新** - 实时从 eloratings.net 拉取
- [ ] **实时结果集成** - ESPN API 集成，自动更新模拟
- [ ] **用户账户系统** - 用户登录、保存模拟历史
- [ ] **移动应用** - React Native 版本
- [ ] **导出功能** - CSV/JSON 导出仪表板数据
- [ ] **性能优化** - WASM 加速模拟引擎
- [ ] **插件系统** - 允许社区添加自定义策略
- [ ] **A/B 测试框架** - 比较不同的 ELO 配置

#### 中优先级

- [ ] 德语/法语/日语翻译
- [ ] 深色/浅色模式切换
- [ ] 键盘快捷键
- [ ] PWA 支持（离线模式）
- [ ] 分享仪表板的社交功能

#### 小型改进

- [ ] 修复 UI 对齐问题
- [ ] 改进加载态
- [ ] 更多动画/过渡
- [ ] 改进表格排序
- [ ] 添加队伍统计详情

---

## 🔄 Pull Request 工作流

### 第 1 步: Fork 仓库

```bash
# 访问 https://github.com/dexorynlabs-betting/2026-worldcup-prediction-market
# 点击 "Fork" 按钮
```

### 第 2 步: 克隆你的 Fork

```bash
git clone https://github.com/YOUR_USERNAME/2026-worldcup-prediction-market.git
cd 2026-worldcup-prediction-market
```

### 第 3 步: 创建功能分支

```bash
git checkout -b feature/ai-predictor
# 或 fix/bug-name, docs/update-readme
```

**分支命名约定:**
- `feature/` - 新功能
- `fix/` - Bug 修复
- `docs/` - 文档更新
- `refactor/` - 代码重构
- `test/` - 测试改进

### 第 4 步: 进行更改

```bash
npm run dev
# 编辑文件...
npm test
npm run build  # 确保构建���功
```

### 第 5 步: 提交更改

**提交消息约定 (Conventional Commits):**

```bash
git commit -m "feat: add AI-powered predictor component

- Integrate TensorFlow.js for model inference
- Add new /predict route
- Update demo tab with predictions
- Tests: 45 new test cases

Closes #123"
```

**格式:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**类型:**
- `feat` - 新功能
- `fix` - Bug 修复
- `docs` - 文档
- `style` - 代码风格 (空格、分号等)
- `refactor` - 不改变行为的代码重组
- `test` - 测试
- `chore` - 构建、依赖等

**示例:**
```bash
git commit -m "feat(sim): add penalty shootout model

- Implement Bayesian shrinkage on historical PK rates
- Achieve 89% accuracy on 2022 validation set
- Add PenaltyKickout type to types.ts

Closes #45"
```

### 第 6 步: 推送到你的 Fork

```bash
git push origin feature/ai-predictor
```

### 第 7 步: 创建 Pull Request

1. 访问 https://github.com/dexorynlabs-betting/2026-worldcup-prediction-market
2. 点击 "New Pull Request"
3. 选择 base: `main` ← compare: `feature/ai-predictor`
4. 填写 PR 描述

**PR 模板:**

```markdown
## 描述
简单描述你的更改是什么。

Closes #(issue)

## 类型
- [ ] 新功能
- [ ] Bug 修复
- [ ] 文档更新
- [ ] 破坏性更改

## 更改列表
- 添加了 AI 预测器组件
- 集成了 TensorFlow.js
- 添加了 45 个新测试

## 测试
- [ ] 本地测试通过
- [ ] 单元测试编写
- [ ] 集成测试编写

## Checklist
- [x] 我已经自审了我的代码
- [x] 我已经注释了复杂区域
- [x] 我已经更新了相关文档
- [x] 我的更改不会产生新的警告
- [x] 我已添加了必要的测试
```

---

## 🎨 代码风格指南

### TypeScript

```typescript
// ✅ 好的
interface SimulationOptions {
  numSimulations: number;
  seed?: number;
  onProgress?: (completed: number, total: number) => void;
}

function runSimulations(opts: SimulationOptions): AggregateResult {
  // 实现
}

// ❌ 避免
function runSimulations(numSims, seed) {
  // 缺少类型
}
```

### React 组件

```typescript
// ✅ 好的
interface ChampionCardProps {
  champion: Team;
  probability: number;
  confidence: ConfidenceInterval;
}

export function ChampionCard({
  champion,
  probability,
  confidence,
}: ChampionCardProps) {
  return (
    <div className="bg-white rounded-lg p-4">
      {/* 内容 */}
    </div>
  );
}

// ❌ 避免
export default function ChampionCard(props) {
  return <div>{props.champion.name}</div>;
}
```

### Tailwind CSS

```tsx
// ✅ 好的 - 清晰的类名，使用语义颜色
<div className="bg-white rounded-lg shadow-md p-4 hover:shadow-lg transition-shadow">
  Champion: {champion.name}
</div>

// ❌ 避免 - 随意的样式，硬编码颜色
<div style={{ background: '#fff', padding: '16px', borderRadius: '8px' }}>
  Champion: {champion.name}
</div>
```

### 文件组织

```
// 一个组件 = 一个文件
src/components/dashboard/ChampionCard.tsx

// 相关逻辑分组在目录中
src/lib/sim/
  ├── engine.ts      # 导出
  ├── tournament.ts  # 内部
  ├── group.ts       # 内部
  └── types.ts       # 共享

// 测试与源码相邻
src/lib/sim/__tests__/
  ├── engine.test.ts
  ├── tournament.test.ts
  └── group.test.ts
```

---

## ✅ 测试指南

### 编写测试

```typescript
import { describe, it, expect } from 'vitest';
import { runSimulations } from '@/lib/sim/engine';

describe('Simulation Engine', () => {
  it('should produce 100,000 simulations', () => {
    const result = runSimulations({
      numSimulations: 100_000,
      seed: 12345,
    });

    expect(result.numSimulations).toBe(100_000);
  });

  it('champion probabilities should sum to 100%', () => {
    const result = runSimulations({
      numSimulations: 10_000,
      seed: 12345,
    });

    const total = result.teams.reduce((sum, _, i) => {
      return sum + result.stageCounts.champion[i];
    }, 0);

    // 允许浮点误差
    expect(total / result.numSimulations).toBeCloseTo(1.0, 2);
  });

  it('average goals per match should be 2.5-2.7', () => {
    // 测试实现...
  });
});
```

### 运行测试

```bash
npm test               # 运行全部
npm test -- engine    # 运行特定文件
npm run test:watch    # 监听模式
```

---

## 📦 发布流程（仅维护者）

```bash
# 1. 确保所有测试通过
npm test

# 2. 构建
npm run build

# 3. 合并 PR 到 main
git merge feature/xyz

# 4. Vercel 自动部署到 production
```

---

## 🎯 优先级指南

**立即开始:**
1. 选择一个标有 `good first issue` 的 issue
2. 在 issue 中评论 "我想处理这个"
3. Fork → 编辑 → 测试 → PR

**热门项目:**
- AI 预测器 (TensorFlow.js)
- 实时比赛追踪
- 用户账户系统
- 性能优化 (WASM)

**简单的获胜:**
- 文档更新
- 修复打字错误
- 改进 UI/UX
- 翻译

---

## 💬 社区

- **Telegram:** [https://t.me/dexoryn777](https://t.me/dexoryn777)
- **X/Twitter:** [@dexoryn](https://x.com/dexoryn)
- **GitHub Discussions:** 在仓库中提出想法
- **Issues:** 报告 Bug 或请求功能

---

## 📚 资源

| 资源 | 链接 |
|------|------|
| Contributing.md | [github.com/github/docs/.../contributing](https://github.com/github/docs/blob/main/CONTRIBUTING.md) |
| Conventional Commits | [conventionalcommits.org](https://www.conventionalcommits.org/) |
| Next.js 文档 | [nextjs.org/docs](https://nextjs.org/docs) |
| Tailwind CSS | [tailwindcss.com](https://tailwindcss.com) |
| TypeScript 最佳实践 | [typescriptlang.org/docs](https://www.typescriptlang.org/docs/) |
| Vitest | [vitest.dev](https://vitest.dev) |

---

## 🎉 致谢

感谢所有为这个项目做贡献的人！贡献者将被列入：
- README.md 的贡献者部分
- GitHub 贡献者图表
- 发布说明

---

**准备好了吗？选择一个问题，创建一个 Fork，并发送一个 PR！** 🚀

有问题？[在 GitHub 中开启讨论](https://github.com/dexorynlabs-betting/2026-worldcup-prediction-market/discussions)
