<h1 align="center">AppStore Publisher Guide</h1>

<p align="center">
  <strong>给你的 AI Agent 一键装上 App Store 发布能力</strong>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge" alt="MIT License"></a>
  <a href="https://github.com/rudrankriyam/App-Store-Connect-CLI"><img src="https://img.shields.io/badge/asc_CLI-v0.35-green.svg?style=for-the-badge" alt="asc CLI"></a>
  <a href="https://github.com/SDLLL/appstore-publisher/stargazers"><img src="https://img.shields.io/github/stars/SDLLL/appstore-publisher?style=for-the-badge" alt="GitHub Stars"></a>
</p>

<p align="center">
  <a href="#快速上手">快速开始</a> · <a href="#它能做什么">功能</a> · <a href="#发布流程概览">流程</a> · <a href="#常见问题">FAQ</a>
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README_CN.md">中文</a>
</p>

---

## 为什么需要这个？

你写完了一个 iOS App，想上架 App Store——然后发现：

- 📦 "帮我把 App 传到 App Store" → **不会用 xcodebuild 命令行**，只能手动在 Xcode 里点 Archive
- 📝 "帮我写 App Store 描述和关键词" → **不知道 ASO 怎么做**，关键词随便填了 100 个字符
- 📸 "帮我生成商店截图" → **手动截图再用 Figma 加框**，一套截图做半天
- 💰 "帮我设置定价" → **App Store Connect 后台点来点去**，找不到价格点在哪
- ❌ "提交审核被拒了" → **4.2 功能不足、4.3 同质化**，不知道怎么防，被拒了也不知道怎么办
- 🔒 "隐私政策、PrivacyInfo.xcprivacy 怎么填" → **每次都要查文档**，生怕漏了什么

**这些事情每发一个 App 都要重复一遍。**

**AppStore Publisher Guide 把这一切变成一句话：**

```
npx skills add SDLLL/appstore-publisher
```

安装后告诉你的 AI Agent「帮我发布 App」，它会自动完成从构建到上架的全部流程。

---

## 快速上手

复制这行命令，在终端运行：

```bash
npx skills add SDLLL/appstore-publisher
```

然后在你的项目目录启动 [Claude Code](https://claude.ai/download)：

```bash
claude
```

对它说「帮我发布 App 到 App Store」。Agent 会自动引导完成 asc 安装、认证配置等所有前置步骤。

> 兼容任何支持 Skills 的 AI Agent：Claude Code、Cursor、Windsurf 等。

---

## 它能做什么

一个 Skill 覆盖发布全流程：

| 阶段 | Agent 自动完成的事 |
|------|-------------------|
| **前置准备** | 安装 asc CLI、配置 API Key、注册 Bundle ID、获取签名证书 |
| **构建上传** | xcodebuild archive → 导出 IPA → `asc publish appstore` 上传 |
| **元数据配置** | 中英双语描述、关键词、年龄分级、Review Notes |
| **ASO 优化** | 关键词策略、描述撰写规则、本地化优先级（自动安装 [ASO 分析 skill](https://github.com/sickn33/antigravity-awesome-skills)） |
| **截图自动化** | `asc screenshots run` 自动截图 → `frame` 加设备外框 → `upload` 上传 |
| **定价设置** | 查询价格点、创建价格计划、管理地区可用性 |
| **审核避坑** | 4.2/4.3/5.1.1 检查、提交节奏建议、Review Notes 撰写指导 |
| **提交审核** | `asc validate --strict` 预检 → `asc submit` 提交 → `asc status` 监控 |
| **终极检查清单** | 构建质量、隐私合规、元数据、功能深度、反同质化逐项确认 |

---

## 发布流程概览

```
配置认证 → 注册 Bundle ID → 构建归档 → 上传 IPA
    → 配置元数据（双语）→ ASO 优化 → 截图自动化
    → 定价设置 → 审核合规检查 → 提交审核 → 监控状态
```

Agent 按顺序执行每个阶段，你只需要在关键节点确认。

---

## 常见问题

<details>
<summary><strong>需要提前安装什么吗？</strong></summary>

不需要。Skill 里包含了完整的前置准备步骤，Agent 会自动引导你安装 asc CLI、配置 API Key、获取签名证书。你只需要有一个 Apple Developer 账号和 Xcode。
</details>

<details>
<summary><strong>支持哪些类型的 App？</strong></summary>

主要针对纯付费 iOS App（无 IAP、无订阅、无广告、无后端）。如果你的 App 有内购或订阅，流程中的定价和元数据部分可能需要调整。
</details>

<details>
<summary><strong>截图自动化怎么用？</strong></summary>

asc CLI v0.29+ 内置了实验性截图流水线。Agent 会自动编写截图计划（JSON）→ 在模拟器中截图 → 用 [Koubou](https://github.com/nicklama/koubou) 加设备外框 → 上传到 App Store Connect。需要 `pip install koubou==0.14.0`。
</details>

<details>
<summary><strong>ASO 优化靠谱吗？</strong></summary>

Skill 内置了实战验证的 ASO 策略（关键词填充规则、描述撰写模板、本地化优先级等），同时会自动安装专业的 [ASO 分析 skill](https://github.com/sickn33/antigravity-awesome-skills) 提供数据驱动的关键词评分和竞品分析。
</details>

<details>
<summary><strong>怎么避免审核被拒？</strong></summary>

Skill 包含完整的审核避坑指南，覆盖最常见的拒审原因（4.2 功能不足、4.3 同质化、2.1 应用完整性、5.1.1 隐私合规），以及 2025-2026 年的新政策变化。提交前 Agent 会自动运行 `asc validate --strict` 进行预检。
</details>

<details>
<summary><strong>兼容哪些 AI Agent？</strong></summary>

任何支持 Claude Code Skills 的 AI Agent 都能用，包括 Claude Code、Cursor、Windsurf 等。
</details>

---

## 致谢

[App Store Connect CLI (asc)](https://github.com/rudrankriyam/App-Store-Connect-CLI) · [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills) · [claude-code-apple-skills](https://github.com/rshankras/claude-code-apple-skills) · [Koubou](https://github.com/nicklama/koubou)

## License

[MIT](LICENSE)
