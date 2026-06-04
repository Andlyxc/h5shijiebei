# 快速运行指南 - 5 分钟启动项目

## 🎯 最快的开始方式

### 方法 1: 直接克隆 + 运行（推荐）

```bash
# 1️⃣ 克隆项目
git clone https://github.com/dexorynlabs-betting/2026-worldcup-prediction-market.git
cd 2026-worldcup-prediction-market

# 2️⃣ 安装依赖 (必须用 --legacy-peer-deps)
npm install --legacy-peer-deps

# 3️⃣ 启动开发服务器
npm run dev

# 4️⃣ 打开浏览器
# http://localhost:3000
```

**完成！** 🎉 现在你可以看到：
- 📊 **主页** - Monte Carlo 模拟器 + 仪表板
- 🎮 **Demo 标签** - 虚拟市场演示
- 📈 **Backtest** - 历史验证
- 📖 **Methodology** - 模型文档

---

## 🛠️ 环境要求

检查你的系统：

```bash
# 检查 Node.js 版本 (需要 18+)
node --version

# 检查 npm 版本 (需要 9+)
npm --version

# 如果版本太旧，更新 Node.js
# 访问 https://nodejs.org/
```

---

## 📦 安装失败？常见问题

### ❌ 问题: `npm ERR! peer dep missing`

**解决方案:**
```bash
npm install --legacy-peer-deps
```

这个项目需要这个标志，因为它使用了一些兼容性依赖。

---

### ❌ 问题: `npm ERR! code ENOENT`

**解决方案:**
```bash
# 清除缓存
npm cache clean --force

# 重新安装
rm -rf node_modules package-lock.json
npm install --legacy-peer-deps
```

---

### ❌ 问题: 端口 3000 已被占用

**解决方案:**
```bash
# 使用其他端口
npm run dev -- -p 3001
# 然后访问 http://localhost:3001
```

或者杀死占用 3000 的进程：
```bash
# macOS/Linux
lsof -i :3000
kill -9 <PID>

# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

---

## 🚀 可用命令

### 开发

```bash
npm run dev          # 启动开发服务器 (最常用)
npm run build        # 构建生产版本
npm start            # 运行生产构建
```

### 测试

```bash
npm test             # 运行测试
npm run test:watch   # 监听模式
```

### 数据更新

```bash
npm run scrape-elo       # 更新 ELO 评分
npm run fetch-odds       # 获取赔率
npm run fetch-absences   # 获取伤病数据
npm run fetch-results    # 获取实时结果
```

---

## 🎨 项目文件位置

```
2026-worldcup-prediction-market/
│
├── src/
│   ├── app/[locale]/page.tsx         ← 主页 (编辑这里!)
│   ├── components/                   ← UI 组件
│   ├── lib/sim/                      ← 模拟引擎 (核心)
│   └── data/teams.json              ← 球队数据
│
├── public/                           ← 图片资源
├── package.json                      ← 项目配置
└── README.md                         ← 完整文档
```

---

## 💻 编辑代码后会发生什么？

开发服务器支持 **热模块重载 (HMR)**：

1. **编辑 `.tsx` 或 `.ts` 文件** → 自动刷新浏览器 ✅
2. **编辑 `data/teams.json`** → 需要手动重启 (Ctrl+C, 再 npm run dev)
3. **编辑 `src/lib/sim/worker.ts`** → 需要手动重启 (Web Worker 不支持 HMR)

---

## 🔍 浏览器调试

### 打开开发者工具

```
Chrome/Edge: F12 或 Ctrl+Shift+I
Firefox: F12 或 Ctrl+Shift+I
Safari: Cmd+Option+I
```

### 查看 Web Worker 性能

1. 打开 DevTools → **Performance** 标签
2. 点击 Record
3. 在页面上点击 "Simulate 100K"
4. 点击 Stop
5. 查看时间线，找到 `simulateTournament` 函数

---

## 📊 第一次运行会看到什么？

```
┌─────────────────────────────────────┐
│  World Cup 2026 Simulator           │
├─────────────────────────────────────┤
│                                     │
│  [Hero Image] 谁将赢得2026世界杯？  │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ Simulations: [1K] [10K] [50K] │   │
│  │ Run Simulation               │   │
│  └──────────────────────────────┘   │
│                                     │
│  Champion Probabilities:            │
│  🇪🇸 Spain       28.5%              │
│  🇦🇷 Argentina   24.2%              │
│  🇫🇷 France      18.9%              │
│  ...                                │
│                                     │
│  [Dashboard Tabs]                   │
│  - Stage Matrix                     │
│  - Group Standings                  │
│  - Match Calendar                   │
│  ...                                │
└─────────────────────────────────────┘
```

---

## ⚡ 性能提示

### 为什么 100K 模拟需要几秒钟？

这是**正常的**！项目运行 Monte Carlo 模拟：
- 100K 次完整赛事模拟
- 每次 104 场比赛计算
- 共 ~1000 万 场比赛计算

**性能基准：**
- 1K 模拟: ~0.1 秒 ⚡
- 10K 模拟: ~1 秒 ⚡
- 50K 模拟: ~5 ��
- 100K 模拟: ~10 秒

### 如何加速？

1. **使用 Chrome/Edge** - 比 Firefox 快 20-30%
2. **关闭其他标签** - 减少系统负载
3. **避免同时运行其他任务** - 给浏览器分配 CPU

---

## 🌍 访问不同页面

| 页面 | URL | 说明 |
|------|-----|------|
| 主页 | http://localhost:3000 | 模拟器 + 仪表板 |
| 演示 | http://localhost:3000/demo | 虚拟市场交易 |
| 回测 | http://localhost:3000/backtest | 历史验证 |
| 方法论 | http://localhost:3000/methodology | 模型文档 |
| 西班牙语 | http://localhost:3000/es | Spanish 版本 |

---

## 📝 修改配置（可选）

### 修改 ELO 值

编辑 `src/data/teams.json`:
```json
{
  "teams": [
    {
      "id": "spain",
      "name": "Spain",
      "elo": 1850,  // ← 改这里
      "group": "A"
    },
    ...
  ]
}
```

重启服务器后生效。

### 修改小组分组

编辑 `src/data/groups.json`:
```json
{
  "A": ["Spain", "France", "Italy", "Germany"],
  "B": ["Argentina", "Brazil", "Uruguay", "Colombia"],
  ...
}
```

---

## 🆘 仍然有问题？

### 查看日志

```bash
# 开发服务器日志
npm run dev

# 看到这样的消息表示成功：
# ▲ Next.js 15.1.0
# - Local:        http://localhost:3000
# ✓ Ready in 1234ms
```

### 完全重置

```bash
# 删除所有缓存和依赖
rm -rf node_modules .next package-lock.json

# 重新安装
npm install --legacy-peer-deps

# 重新运行
npm run dev
```

### 获取帮助

- 📖 查看 [LOCAL_SETUP_GUIDE.md](./LOCAL_SETUP_GUIDE.md)
- 🤝 查看 [CONTRIBUTION_GUIDE.md](./CONTRIBUTION_GUIDE.md)
- 💬 [GitHub Issues](https://github.com/dexorynlabs-betting/2026-worldcup-prediction-market/issues)
- 📱 [Telegram](https://t.me/dexoryn777)

---

## ✅ 确认运行成功

看到以下信息表示一切正常：

```
✓ Ready in 1234ms

> Local: http://localhost:3000
```

然后：
1. 打开 http://localhost:3000 ✅
2. 看到 Hero 图像和模拟器 ✅
3. 点击 "Simulate 100K" 按钮 ✅
4. 等待 ~10 秒看到结果 ✅

**恭喜！你已成功运行项目！** 🎉

---

## 🎯 下一步

- 📖 读 [LOCAL_SETUP_GUIDE.md](./LOCAL_SETUP_GUIDE.md) 了解更多
- 🤝 读 [CONTRIBUTION_GUIDE.md](./CONTRIBUTION_GUIDE.md) 准备贡献
- 💡 在 `/demo` 页面玩虚拟市场
- 🔧 尝试修改代码和运行

**祝你开发愉快！** 🚀
