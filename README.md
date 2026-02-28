# iOS Publish Guide

使用 AI Agent + [asc CLI](https://github.com/rudrankriyam/App-Store-Connect-CLI) 完成 iOS App 从构建到上架 App Store 的全流程自动化。

```

## 包含什么

这个 skill 包含一份完整的 Agent 操作手册（`SKILL.md`），覆盖 9 个章节：

| 章节 | 内容 |
|------|------|
| 0. 前置准备 | asc 安装、API Key 配置、Bundle ID 注册、签名获取 |
| 1. 构建与上传 | xcodebuild archive → asc publish appstore |
| 2. 元数据配置 | 双语描述/关键词、年龄分级、Review Notes |
| 3. ASO 优化 | 关键词策略、描述撰写规则、本地化优先级 |
| 4. 截图自动化 | capture → frame(Koubou) → review → upload |
| 5. 定价设置 | 价格点查询、价格计划创建、地区可用性 |
| 6. 审核避坑 | 4.3 同质化、4.2 功能不足、隐私合规、提交节奏 |
| 7. 提交审核 | validate → submit → status 监控 |
| 8. 提交前检查清单 | 构建/隐私/元数据/功能/反同质化逐项确认 |

## 发布流程概览

```
配置认证 → 注册 Bundle ID → 构建归档 → 上传 IPA
    → 配置元数据（双语）→ ASO 优化 → 截图自动化
    → 定价设置 → 审核合规检查 → 提交审核 → 监控状态
```

## 致谢

- [App Store Connect CLI (asc)](https://github.com/rudrankriyam/App-Store-Connect-CLI)
- [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills) — ASO 优化（MIT）
- [claude-code-apple-skills](https://github.com/rshankras/claude-code-apple-skills) — App Store skill（MIT）
- [Koubou](https://github.com/nicklama/koubou) — 截图设备外框合成

## License

MIT
