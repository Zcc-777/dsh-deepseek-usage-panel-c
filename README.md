<h1 align="center">DSH-USAGE-PANEL-C</h1>

<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-4d6bfe" height="20" alt="license-MIT" />&ensp;
  <img src="https://img.shields.io/badge/version-0.2.4--c.1-7c5cff" height="20" alt="version" />&ensp;
  <img src="https://img.shields.io/badge/DeepSeek%20Harness-Web-blue" height="20" alt="platform" />&ensp;
  <img src="https://img.shields.io/badge/node-%E2%89%A520-30d158" height="20" alt="node" />&ensp;
  <img src="https://img.shields.io/badge/UI-%E4%B8%AD%E6%96%87-ff9f0a" height="20" alt="ui" />
</p>

> 📌 版本更新：见 [doc/ver/0.2.4-c.1.md](doc/ver/0.2.4-c.1.md)

> 🍴 本仓库是 [WeiyangPro/dsh-deepseek-usage-panel](https://github.com/WeiyangPro/dsh-deepseek-usage-panel) 的定制分支，仓库名为 `dsh-deepseek-usage-panel-c`。包名与 DSH 插件 ID 仍保持 `dsh-deepseek-usage-panel`，便于直接替换原版并沿用现有配置与数据目录。本分支新增峰谷价提示、今日消费和今日 Token 明细。

> 🪟 DeepSeek Harness（DSH）Web 悬浮用量面板 —— macOS 毛玻璃 · 中文界面 · 官方余额 · 今日消费 · 峰谷价倒计时 · Token 用量曲线。

Floating macOS-glass **usage panel** for DeepSeek Harness Web: official
balance + today's spend (balance-ledger) + peak/off-peak countdown + detailed
token usage (today / session / 24 h smooth trend / per-model).
`Token` 等有歧义名词未做翻译。

## 📸 效果预览

<p align="center">
  <img src="img/dark.png" alt="深色主题" width="240" />&nbsp;<img src="img/light.png" alt="亮色主题" width="240" />
</p>

> 截图为上游版本；本分支新增的「峰价/谷价」「今日消费」「今日 Token 明细」见下方功能说明。

## ✨ 功能

- 💰 **官方余额**：轮询 `GET /user/balance`，只显示官方字段（当前余额、充值余额）。
- 💸 **今日消费**：胶囊显示余额观测口径的今日消费（`¥x.xx`）；展开窗保留今日 Token 明细（缓存命中/未命中、输出、合计）。
- 🕒 **峰谷价提示**：胶囊与展开窗显示当前「峰价 / 谷价」及距下次切换倒计时；峰价红、谷价绿。规则按官方价格页（周一至周五 09:00–12:00、14:00–18:00 峰价，其余谷价，谷价 5 折），内置 2026 年法定节假日表。
- 📊 **Token 用量**：今日 / 会话 / 近 24 小时平滑曲线（输入·输出，含缓存命中/未命中明细）/ 按模型占比。
- 🖱️ **交互**：胶囊 ⇄ 面板拖动；点击 / 红绿灯 / `ESC` 收起（带过渡动画）；面板按胶囊位置自动选择展开方向；↻ 一键刷新（旋转 + 「已刷新」反馈）。
- 🚫 无需频繁打开 platform.deepseek.com/usage。

## 📦 安装与卸载

### 安装

```bash
# 如果已安装原版，先卸载（本分支沿用同一包名/插件 ID）
dsh plugin --profile web remove dsh-deepseek-usage-panel

# 方式一：从本定制分支在线安装（推荐）
dsh plugin --profile web add "github:Zcc-777/dsh-deepseek-usage-panel-c"

# 方式二：从源码安装
dsh plugin --profile web add "link:/你的路径/dsh-deepseek-usage-panel-c"
```

装完重启 `dsh web` 并硬刷新浏览器（`Ctrl/Cmd + Shift + R`）。若未自动加入
bundle，把 `dsh-deepseek-usage-panel` 追加到 profile `package.json` 的
`dsh.profile.bundles`。

**API Key（余额必需）** —— 在 `~/.dsh/.credentials.yaml` 配置（插件直读，不注入浏览器）：

```yaml
version: 1
refs:
  DEEPSEEK_API_KEY: "<你的 DeepSeek API Key>"
```

> 不要把真实 Key 写入仓库或提交到 Git；只放在本机 `~/.dsh/.credentials.yaml` 或环境变量里。

也可设置环境变量 `DEEPSEEK_API_KEY` 或 `DSH_USAGE_HUD_API_KEY`。

### 卸载

```bash
# 方式一：CLI 卸载（移除依赖与 bundles 项）
dsh plugin --profile web remove dsh-deepseek-usage-panel

# 方式二：手动清理
pnpm --dir <profile-dir> remove dsh-deepseek-usage-panel
# 并手动把 package.json 的 dsh.profile.bundles 中的 "dsh-deepseek-usage-panel" 移除
```

重启 `dsh web` 即完全生效。（可选）删除本地数据目录
`$DSH_HOME/dsh-deepseek-usage-panel/` 或 `<profile>/.dsh-usage/` 以清理历史用量记录。

## 🖱️ 使用

- 右下角胶囊显示 余额 / 今日消费 / 峰谷倒计时，可拖动、位置自动记忆。
- 轻点胶囊展开面板：今日消费、峰谷时段条、余额卡、今日 Token 明细、近 24 小时曲线、按模型统计、DeepSeek 用量页链接。
- 三个 MAC 圆点或 `ESC` 收起；右上 ↻ 立即刷新。

## ⚙️ 配置项

在 profile 的 `cordis.patch.yml` 中按行 id 覆盖：

```yaml
- id: usage-panel
  config:
    pollIntervalMs: 300000   # 余额轮询间隔（毫秒，最小 60000）
    historyHours: 72         # 历史小时桶保留数
    baseURL: https://api.deepseek.com   # 覆盖接口地址
    apiKeyRef: DEEPSEEK_API_KEY         # 凭据引用名
    dataDir: null            # 覆盖数据目录（默认 $DSH_HOME/dsh-deepseek-usage-panel）
```

## 🔒 数据与隐私

- 🔑 API Key 仅存在于宿主（Node）进程，浏览器只收到解析后的余额快照。
- 🗄️ 用量历史写入数据目录 `state.json`（原子写入、自动裁剪），更换安装方式自动迁移旧数据。
- 🌐 网络请求：每轮询周期一次官方余额接口；无其它外发流量。

### 📏 数据口径（请一定了解）

「今日 / 近 24 小时 / 按模型 / 会话」的 Token 数据是**本插件对 Harness 会话的本地测量**，
**不是 DeepSeek 平台的官方账目**，因此可能与 platform.deepseek.com/usage 的数字存在差异。原因如下：

1. **观测窗口**：插件从「它运行起来之后的时刻」开始统计；当天若**安装较晚、中途重装/重启过 dsh web**，
   在这些时间点**之前**产生的请求，本地拿不到（DeepSeek 未提供官方用量 API，无法补齐）。
   —— 表现为“今日/近 24h”可能**偏小于**官方全天金额。
2. **口径单条一致**：每条响应使用 DeepSeek 返回的 `usage`（未命中输入 + 缓存命中输入 + 输出含推理），
   与官方计费同源；所以**单条记录的 token 计算是准的**，偏差主要来自“**统计时间窗口不完整**”，而非算错。
3. **模型占比可能失真**：如果某段时间（例如上午）未被插件覆盖，而那段恰好以某个模型/缓存命中为主，
   就会让“按模型”占比与实际全天不同。
4. **“会话”≠“今日”**：会话是**当前会话的累计值（可跨天/多轮）**，不是自然日口径，不能与日账单直接比较。
5. **按本地时区零点切分“今日”**：若你与官方账单的时区口径不同，临近午夜会差一截。
6. 缓存命中（`cacheReadTokens`）单独列出；输出含推理 token，均与官方分桶一致。
7. **今日消费**是余额观测账本口径（余额下降累计），不是 token × 单价的估算；余额接口有轮询间隔，数字可能略有滞后，且插件启动前发生的消费无法补记。

> 结论：**数字可信的方向是“相对趋势/构成”，绝对总量请以官方 usage 页为准。**
> 我们计划在后续版本中加入「统计自插件启用时间 / 完整性提示（完整 or 可能不完整）/ 手动重置」，
> 让口径更透明。若你对某天数据差异有疑问，可按“统计窗口”理解，而非视为错误。


## 🧩 本分支的定制内容

本仓库基于 [WeiyangPro/dsh-deepseek-usage-panel](https://github.com/WeiyangPro/dsh-deepseek-usage-panel)
修改，仓库名为 `dsh-deepseek-usage-panel-c`；包名与 DSH 插件 ID 保持
`dsh-deepseek-usage-panel`，可直接替换原版并沿用现有配置与数据目录。

- 🕒 **峰谷价提示**：胶囊与展开窗显示当前「峰价 / 谷价」及距下次切换倒计时；峰价红、谷价绿。规则见 `client.js` 的 `dsh-usage pricing-policy` 区块。
- 💸 **今日消费**：胶囊显示余额观测口径的今日消费金额（`¥x.xx`）。
- 📊 **今日 Token 明细**：缓存命中输入、缓存未命中输入、输出（含推理）、合计，以及响应次数与缓存命中率。
- 📅 2027 年放假安排公布后，更新 `client.js` 里的 `CN_HOLIDAYS` 与 `CN_HOLIDAY_YEARS`。

上游项目：[WeiyangPro/dsh-deepseek-usage-panel](https://github.com/WeiyangPro/dsh-deepseek-usage-panel)

## 📄 许可

MIT License —— 见 [LICENSE](LICENSE)。
