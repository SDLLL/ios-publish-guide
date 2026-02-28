---
name: ios-publish-guide
description: End-to-end iOS App Store publishing workflow using asc CLI. Covers build & upload, metadata, ASO optimization, screenshot automation, pricing, review guidelines, and submission. This skill should be used when publishing, submitting, or uploading an iOS app to the App Store, or when configuring App Store metadata, screenshots, and pricing via CLI.
---

# ASC CLI App Store 发布完全指南

> Agent 操作手册 — 按顺序执行每个阶段，完成从构建到上架的全流程。
> 基于 asc v0.35.2，适用于纯付费 iOS App（无 IAP、无订阅、无后端）。

---

## 0. 前置准备

### 0.1 安装 asc

```bash
# 方式一：Homebrew（推荐）
brew install asc

# 方式二：源码编译
git clone https://github.com/rudrankriyam/App-Store-Connect-CLI.git
cd App-Store-Connect-CLI
go build -o /usr/local/bin/asc .

# 验证
asc --version
```

### 0.2 配置 API Key

引导用户前往 [App Store Connect → Keys](https://appstoreconnect.apple.com/access/integrations/api) 创建 API Key，下载 `.p8` 文件。

```bash
# 登录（交互式，将凭证存入系统钥匙串）
asc auth login

# 或通过环境变量配置（适合 CI / Agent）
export ASC_KEY_ID="你的KeyID"
export ASC_ISSUER_ID="你的IssuerID"
export ASC_PRIVATE_KEY_PATH="/path/to/AuthKey_XXXXXX.p8"

# 验证认证状态
asc auth status
asc doctor --output table
```

**注意事项：**
- API Key 需要 Admin 或 App Manager 权限
- `.p8` 文件只能下载一次，妥善保存
- 如果认证出问题，运行 `asc doctor` 诊断

### 0.3 注册 Bundle ID

```bash
asc bundle-ids create \
  --identifier "com.yourcompany.appname" \
  --name "AppName" \
  --platform IOS
```

**验证：** `asc bundle-ids list` 确认已创建。

### 0.4 获取签名证书和描述文件

```bash
asc signing fetch \
  --bundle-id com.yourcompany.appname \
  --profile-type IOS_APP_STORE \
  --output ./signing
```

产出文件：`./signing/` 目录下的 `.cer` 和 `.mobileprovision` 文件。

**注意事项：**
- 如果使用 xcodebuild 云端签名（`-allowProvisioningUpdates -authenticationKeyPath`），可跳过此步
- `ExportOptions.plist` 中的 `teamID` 必须与开发者团队 ID 一致

---

## 1. 构建与上传

### 1.1 在 App Store Connect 网页端创建 App

**这一步无法通过 CLI 完成，必须手动操作。**

前往 [App Store Connect → Apps → +](https://appstoreconnect.apple.com/apps) 创建新 App：
- 选择已注册的 Bundle ID
- 填写 App 名称（后续可通过 CLI 修改元数据）
- 选择主要语言
- 选择 SKU（建议用 bundle ID 后缀）

创建后记录 **App ID**（数字），后续命令需要：

```bash
# 查找 App ID
asc apps --output table
```

### 1.2 构建归档

```bash
# Archive
xcodebuild archive \
  -project AppName.xcodeproj \
  -scheme AppName \
  -archivePath ./build/AppName.xcarchive \
  -destination "generic/platform=iOS" \
  -allowProvisioningUpdates \
  -authenticationKeyPath "$ASC_PRIVATE_KEY_PATH" \
  -authenticationKeyID "$ASC_KEY_ID" \
  -authenticationKeyIssuerID "$ASC_ISSUER_ID"

# Export IPA
xcodebuild -exportArchive \
  -archivePath ./build/AppName.xcarchive \
  -exportPath ./build \
  -exportOptionsPlist ExportOptions.plist \
  -allowProvisioningUpdates \
  -authenticationKeyPath "$ASC_PRIVATE_KEY_PATH" \
  -authenticationKeyID "$ASC_KEY_ID" \
  -authenticationKeyIssuerID "$ASC_ISSUER_ID"
```

`ExportOptions.plist` 最小配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>app-store-connect</string>
    <key>teamID</key>
    <string>你的TeamID</string>
    <key>uploadSymbols</key>
    <true/>
    <key>destination</key>
    <string>upload</string>
</dict>
</plist>
```

### 1.3 上传 IPA

```bash
asc publish appstore \
  --app "APP_ID" \
  --ipa ./build/AppName.ipa \
  --version "1.0.0" \
  --wait
```

`--wait` 会等待 Apple 处理构建（通常 5-15 分钟）。

**验证：** `asc builds list --app "APP_ID" --limit 1 --output table`

**注意事项：**
- 上传超时可设置 `ASC_UPLOAD_TIMEOUT=30m`
- 如果 IPA 较大，耐心等待，不要重复上传
- Info.plist 必须包含 `ITSAppUsesNonExemptEncryption = NO`，否则每次上传都会弹出加密合规问卷

---

## 2. 元数据配置

### 2.1 App 基本信息（双语）

```bash
# 中文元数据
asc app-info set \
  --app "APP_ID" \
  --locale "zh-Hans" \
  --description "你的 App 中文描述..." \
  --keywords "关键词1,关键词2,关键词3" \
  --support-url "https://yoursite.com/support" \
  --marketing-url "https://yoursite.com"

# 英文元数据
asc app-info set \
  --app "APP_ID" \
  --locale "en-US" \
  --description "Your App English description..." \
  --keywords "keyword1,keyword2,keyword3" \
  --support-url "https://yoursite.com/support" \
  --marketing-url "https://yoursite.com"
```

**验证：** `asc app-info get --app "APP_ID" --output table`

### 2.2 年龄分级

```bash
asc age-rating set \
  --app "APP_ID" \
  --violence-cartoon NONE \
  --violence-realistic NONE \
  --sexual-content-nudity NONE \
  --profanity-humor NONE \
  --alcohol-tobacco-drug-use NONE \
  --gambling false \
  --horror-fear NONE \
  --unrestricted-web-access false \
  --user-generated-content false
```

根据 App 实际内容调整每项的值（`NONE` / `INFREQUENT_OR_MILD` / `FREQUENT_OR_INTENSE`）。

### 2.3 审核信息

```bash
# 获取版本 ID
asc versions list --app "APP_ID" --output table

# 创建审核详情
asc review details-create \
  --version-id "VERSION_ID" \
  --contact-email "your@email.com" \
  --contact-first-name "YourName" \
  --contact-last-name "YourLastName" \
  --contact-phone "+86xxxxxxxxxx" \
  --notes "审核说明：描述 App 用途、测试步骤、特殊功能说明"
```

**Review Notes 撰写要点：**
- 第一句话说清楚 App 做什么
- 如果功能较简单，主动解释独特价值
- 如需特殊操作才能体验完整功能，写明步骤
- 不要留空——空白 Review Notes 会增加被拒概率

---

## 3. ASO 优化

ASO（App Store Optimization）需要在编写元数据前完成。

### 3.0 安装 ASO 分析工具

在执行 ASO 优化前，Agent 必须先检查并安装专业的 ASO 分析 skill：

```bash
# 检查是否已安装
ls ~/.claude/skills/app-store-optimization/SKILL.md 2>/dev/null || \
  npx skills add sickn33/antigravity-awesome-skills --skill app-store-optimization --all -y
```

安装后可获得完整的 ASO 分析能力：关键词评分模型、竞品分析框架、元数据优化器、ASO 健康分评估等。**在进行关键词研究和元数据优化时，应调用 `app-store-optimization` skill 获得数据驱动的分析结果。**

> 致谢：ASO 分析 skill 来自 [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills)（MIT License）

### 3.1 关键词策略

**100 字符限额，每个字都要精打细算：**

- 不要重复 App 名称和副标题中已有的词（Apple 会自动索引）
- 用逗号分隔，**逗号后不加空格**（浪费字符）
- 优先放搜索量高但竞争度低的长尾词
- 中文关键词按单字拆分可获得更多组合（如"记账本"被拆为"记""账""本"）
- 英文关键词用单数形式（Apple 自动匹配复数）
- 不要使用竞品品牌名（违反规则且无效）

**关键词研究方法：**
1. App Store 搜索联想（输入核心词看下拉建议）
2. 竞品 App 的关键词分析（查看它们的名称、副标题、描述）
3. Apple Search Ads 的搜索热度参考

### 3.2 描述撰写规则

**前三行是生命线**——用户在商店只能看到前三行，必须点击"更多"才能展开。

```
结构模板：
第 1 行：一句话说清 App 核心价值（解决什么问题）
第 2-3 行：最重要的 2-3 个卖点
---
第 4-8 行：功能列表（用 emoji + 短句）
---
最后：隐私声明 / 支持信息
```

**禁忌：**
- 不要以"欢迎使用..."开头
- 不要堆砌关键词
- 不同 App 的描述必须完全独立撰写，禁止复制粘贴
- 不使用未经验证的声明（"最好的""第一名"）

### 3.3 App 名称与副标题

- **名称**（30 字符限制）：简洁有辨识度，包含一个核心关键词
- **副标题**（30 字符限制）：补充说明功能或价值，放入次要关键词
- 不使用 Apple 商标（如"for iPhone"）
- 名称 + 副标题 + 关键词三者互补，不重复

### 3.4 本地化优先级

付费 App 的重点市场（按收入排序）：
1. 🇺🇸 en-US（必须）
2. 🇨🇳 zh-Hans（必须）
3. 🇯🇵 ja（推荐）
4. 🇬🇧 en-GB（可复用 en-US）
5. 🇩🇪 de-DE / 🇫🇷 fr-FR（可选）

每个语言的关键词和描述都应该是**原生撰写**，不是机器翻译。不同语言市场的用户搜索习惯完全不同。

---

## 4. 截图自动化

asc v0.29+ 新增了实验性截图自动化流水线，依赖 [Koubou](https://github.com/nicklama/koubou) 进行设备外框合成。

### 4.1 安装依赖

```bash
pip install koubou==0.14.0
```

### 4.2 编写截图计划

创建 `.asc/screenshots.json`：

```json
{
  "bundleId": "com.yourcompany.appname",
  "udid": "booted",
  "outputDir": "./screenshots/raw",
  "steps": [
    { "action": "launch" },
    { "action": "wait", "duration": 2 },
    { "action": "screenshot", "name": "01-home" },
    { "action": "tap", "x": 200, "y": 400 },
    { "action": "wait", "duration": 1 },
    { "action": "screenshot", "name": "02-detail" },
    { "action": "tap", "x": 50, "y": 50 },
    { "action": "screenshot", "name": "03-settings" }
  ]
}
```

支持的 action：`launch`、`tap`、`type`、`wait`、`wait_for`（轮询）、`screenshot`。

### 4.3 自动截图

```bash
# 确保模拟器已启动且 App 已安装
# 执行截图序列
asc screenshots run --plan .asc/screenshots.json
```

或单独截图：

```bash
asc screenshots capture \
  --bundle-id "com.yourcompany.appname" \
  --name "home" \
  --output-dir ./screenshots/raw
```

### 4.4 加设备外框

```bash
# 单张截图加框
asc screenshots frame \
  --input ./screenshots/raw/01-home.png \
  --device iphone-air \
  --output-dir ./screenshots/framed

# 支持的设备：
# iphone-air（默认）、iphone-17-pro、iphone-17-pro-max
# iphone-16e、iphone-17、mac
```

高级用法——使用 Koubou YAML 配置自定义标题和背景：

```bash
asc screenshots frame \
  --config ./koubou.yaml \
  --watch  # 监听文件变化，自动重新生成
```

### 4.5 审核预览

```bash
# 生成 HTML 对比报告（原图 vs 加框图）
asc screenshots review-generate \
  --framed-dir ./screenshots/framed \
  --raw-dir ./screenshots/raw \
  --output-dir ./screenshots/review

# 在浏览器中打开审核
asc screenshots review-open --output-dir ./screenshots/review

# 批准所有就绪的截图
asc screenshots review-approve --all-ready --output-dir ./screenshots/review
```

### 4.6 上传截图

```bash
# 获取版本本地化 ID
asc localizations list --version-id "VERSION_ID" --output table

# 上传 iPhone 截图（6.7 英寸）
asc screenshots upload \
  --version-localization "LOC_ID" \
  --path "./screenshots/framed" \
  --device-type "IPHONE_65"
```

**截图规则：**
- 必须与实际 App UI 一致
- 可以有文字叠加，但主体必须是真实截图
- 每个 App 的截图完全独立制作
- 对于大多数 iOS 提交，一组 iPhone（`IPHONE_65`）+ 一组 iPad（`IPAD_PRO_3GEN_129`）就够了
- 使用 `asc screenshots sizes` 查看支持的尺寸，`--all` 查看完整矩阵

---

## 5. 定价设置

### 5.1 查看价格点

```bash
# 查看可用价格点
asc pricing price-points --app "APP_ID" --territory "USA" --output table

# 按价格筛选
asc pricing price-points --app "APP_ID" --territory "USA" --price 0.99
```

### 5.2 创建价格计划

```bash
# 设置价格（需要先获取 price-point ID）
asc pricing schedule create \
  --app "APP_ID" \
  --price-point "PRICE_POINT_ID" \
  --base-territory "USA"
```

### 5.3 管理地区可用性

```bash
# 查看当前可用地区
asc pricing availability get --app "APP_ID" --output table

# 设置特定地区可用
asc pricing availability set \
  --app "APP_ID" \
  --territory "USA,CHN,JPN,GBR,DEU" \
  --available true
```

**验证：** `asc pricing schedule get --app "APP_ID" --output table`

---

## 6. 审核避坑指南

### 6.1 致命风险：Guideline 4.3 — 垃圾应用 / 同质化

**最严重的后果：开发者账号永久封禁。**

会触发 4.3 的行为：
- 多个 App 共享相同源码或资源
- 使用模板批量生产 App
- 相似的二进制文件、元数据、设计
- 同一功能拆成多个 App

**防御措施：**
- 每个 App 解决**不同的问题**，属于**不同的分类**
- 每个 App 独立 UI、独立元数据、独立截图
- 不同 App 使用**不同的系统框架组合**（一个用 MapKit，另一个用 Charts）
- 永远不要同一天提交多个 App，每次间隔至少 3-5 天
- 永远不要复制粘贴 App 描述、关键词、截图文案

### 6.2 高风险：Guideline 4.2 — 最低功能要求

**审核员评估时间约 5 分钟。** 他们不会深入使用你的 App，需要在很短的时间内判断这个 App 是否有存在价值。

**必须具备的"原生深度"信号（按重要程度）：**

| 信号 | 说明 |
|------|------|
| Widget（WidgetKit） | 通过 4.2 审核的最强信号 |
| App Intents / Siri | 展示深度系统集成 |
| 多页面交互（≥3 页） | 证明不是玩具 App |
| SwiftData 数据持久化 | 有状态体验 |
| 设置页面 | 用户可自定义 |
| 引导流程 | 帮助审核员快速理解价值 |

**被拒应对策略：**
1. 不要立刻修改，先理解拒绝理由
2. 如果理由含糊（模板回复），直接**重新提交**——可能换到不同审核员
3. 如果理由具体，针对性添加功能后重新提交
4. 被拒后**专业回复**，不要争论

**4.2 的执行高度不一致**——同样的 App 可能被 A 审核员拒绝却被 B 审核员通过。不要滥用加急审核（Expedited Review）——过度使用会标记账号。

### 6.3 必需：Guideline 2.1 — 应用完整性

占所有拒审的 **40%**，但完全可以预防：
- App 启动崩溃
- 残留占位符文本（"Lorem ipsum"、"TODO"、"即将推出"）
- 死链接
- 功能不完整（按钮无响应、空页面）
- 需要登录但未提供测试账号
- 在无网络 / 弱网环境下崩溃

**预防措施：**
- 在**真机**上测试，不能只用模拟器
- 在无网络 / 弱网环境下测试
- 搜索项目中的所有 "TODO"、"FIXME"、"placeholder"
- 验证所有 URL 可访问
- Review Notes 中提供清晰的测试说明

### 6.4 必需：Guideline 5.1.1 — 隐私合规

每个 App 都必须有（即使不收集任何数据）：
- `PrivacyInfo.xcprivacy` — 声明 API 使用原因
- Privacy Policy URL — 在 App Store Connect 中设置
- App 内隐私政策入口 — 设置页面中提供链接
- App Privacy 营养标签 — 如实填写

如果使用 `UserDefaults`、文件时间戳 API、磁盘空间 API 等，必须在 `PrivacyInfo.xcprivacy` 中声明 Required Reasons。

### 6.5 提交节奏

| 阶段 | 策略 |
|------|------|
| 建立信誉期（前 4 周） | 每周 1 个，提交最独特最精致的 App |
| 稳步加速期（5-12 周） | 每周 2 个，前提是无拒审 |
| 正常节奏（13 周起） | 每周 2-3 个，根据审核反馈调整 |
| 遇到拒审 | 先解决再提交下一个，不要赌运气 |

**关键原则：**
- **永远不要同一天提交多个 App**
- 每次提交间隔至少 3-5 天
- 保持与审核团队的专业沟通
- 不要滥用加急审核（Expedited Review）——过度使用会标记账号

### 6.6 2025-2026 特别注意事项

| 时间 | 政策变化 |
|------|---------|
| 2025 年 2 月起 | 涉及隐私的第三方 SDK 必须包含自己的隐私清单文件 |
| 2025 年 7 月起 | 新增年龄分级（13+、16+、18+），需在 2026 年 1 月 31 日前完成更新 |
| 2025 年 11 月起 | AI 透明度要求——使用外部 AI 服务必须提供用户同意弹窗 |
| 2026 年 4 月起 | 所有提交必须使用 iOS 26 SDK（Xcode 26）编译 |

**重要提醒：**
- Apple 现在使用 **AI 辅助审核 + 人工审核** 结合，模板化 App 的自动检测能力显著增强
- 如果 App 不使用任何外部 AI 服务（纯本地功能），则无需处理 AI 透明度要求
- 第三方 SDK 隐私清单是指 SDK 提供者的责任，但集成方需确认所用 SDK 已包含

---

## 7. 提交审核

### 7.1 提交前验证

```bash
asc validate \
  --app "APP_ID" \
  --version "1.0.0" \
  --platform IOS \
  --strict \
  --output table
```

此命令检查：元数据长度限制、必填字段、审核详情完整性、主分类、构建已附加、定价计划、截图存在性、年龄分级。

**`--strict` 会将警告视为错误。在提交前务必通过 strict 检查。**

### 7.2 提交

```bash
asc submit create \
  --app "APP_ID" \
  --version "1.0.0" \
  --build "BUILD_ID" \
  --confirm
```

或使用一键发布（上传 + 附加构建 + 提交）：

```bash
asc publish appstore \
  --app "APP_ID" \
  --ipa ./build/AppName.ipa \
  --version "1.0.0" \
  --submit \
  --confirm \
  --wait
```

### 7.3 监控审核状态

```bash
# 查看完整发布仪表盘
asc status --app "APP_ID" --output table

# 仅查看提交和审核状态
asc status --app "APP_ID" --include submission,review --output table
```

**注意事项：**
- 首次版本不支持 `--whats-new` 参数（只有更新版本才需要）
- 如果提交卡在 READY_FOR_REVIEW，asc 会自动取消旧提交再创建新的
- 审核通常 24-48 小时，节假日可能更久

---

## 8. 提交前终极检查清单

Agent 在执行 `asc submit` 前，必须逐项确认：

### 构建质量
- [ ] 真机或模拟器测试无崩溃
- [ ] 无占位符文本（搜索 TODO / FIXME / placeholder / Lorem）
- [ ] 所有链接可访问
- [ ] 所有按钮有响应
- [ ] 无网络环境下不崩溃
- [ ] 引导页正常显示
- [ ] `ITSAppUsesNonExemptEncryption = NO` 已加入 Info.plist

### 隐私合规
- [ ] `PrivacyInfo.xcprivacy` 存在且准确
- [ ] Privacy Policy URL 已设置
- [ ] App 内可访问隐私政策
- [ ] 所有权限有 UsageDescription

### 元数据
- [ ] 中文和英文描述已设置
- [ ] 关键词已填写（100 字符以内）
- [ ] 截图已上传且与实际 UI 一致
- [ ] App 名称无关键词堆砌
- [ ] 分类选择正确
- [ ] 年龄分级已填写
- [ ] Review Notes 清晰说明用途
- [ ] 定价已设置

### 功能深度
- [ ] Widget 已实现
- [ ] App Intent / Siri Shortcut 已注册
- [ ] 至少 3 个不同页面
- [ ] SwiftData 数据持久化
- [ ] 设置页面存在
- [ ] 引导流程存在

### 反同质化（如果是系列 App）
- [ ] 与已提交的其他 App 在 UI 上有明显区别
- [ ] 描述文案与其他 App 无雷同
- [ ] 使用了不同的系统框架组合
- [ ] 属于不同的 App Store 分类
- [ ] 距离上次提交至少间隔 3 天

### 最终验证
- [ ] `asc validate --strict` 全部通过
- [ ] `asc status` 确认构建已处理完成

---
