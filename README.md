<h1 align="center">AppStore Publisher Guide</h1>

<p align="center">
  <strong>Give your AI Agent the power to publish apps to the App Store</strong>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge" alt="MIT License"></a>
  <a href="https://github.com/rudrankriyam/App-Store-Connect-CLI"><img src="https://img.shields.io/badge/asc_CLI-v0.35-green.svg?style=for-the-badge" alt="asc CLI"></a>
  <a href="https://github.com/SDLLL/appstore-publisher/stargazers"><img src="https://img.shields.io/github/stars/SDLLL/appstore-publisher?style=for-the-badge" alt="GitHub Stars"></a>
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> · <a href="#what-it-does">Features</a> · <a href="#workflow-overview">Workflow</a> · <a href="#faq">FAQ</a>
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README_CN.md">中文</a>
</p>

---

## Why This Exists

You've built an iOS app and want to ship it to the App Store — then reality hits:

- 📦 "Upload my app to the App Store" → **No idea how to use xcodebuild CLI**, stuck clicking Archive in Xcode
- 📝 "Write my App Store description and keywords" → **No ASO knowledge**, keywords filled in randomly
- 📸 "Generate store screenshots" → **Manual screenshots + Figma framing**, half a day per set
- 💰 "Set up pricing" → **Clicking around App Store Connect**, can't find where price points are
- ❌ "My app got rejected" → **4.2 minimum functionality, 4.3 spam** — don't know how to prevent or respond
- 🔒 "How to fill in PrivacyInfo.xcprivacy" → **Checking docs every time**, afraid of missing something

**You repeat all of this for every single app.**

**AppStore Publisher Guide turns it into one command:**

```
npx skills add SDLLL/appstore-publisher
```

After installation, tell your AI Agent "publish my app" and it will handle the entire process from build to submission.

---

## Quick Start

Run this in your terminal:

```bash
npx skills add SDLLL/appstore-publisher
```

Then launch [Claude Code](https://claude.ai/download) in your project directory:

```bash
claude
```

Say "publish my app to the App Store". The Agent will automatically guide you through asc installation, authentication setup, and all prerequisites.

> Compatible with any AI Agent that supports Skills: Claude Code, Cursor, Windsurf, etc.

---

## What It Does

One Skill covers the entire publishing workflow:

| Stage | What the Agent Does |
|-------|-------------------|
| **Prerequisites** | Install asc CLI, configure API Key, register Bundle ID, fetch signing certificates |
| **Build & Upload** | xcodebuild archive → export IPA → `asc publish appstore` upload |
| **Metadata** | Bilingual descriptions, keywords, age rating, Review Notes |
| **ASO Optimization** | Keyword strategy, description writing rules, localization priorities (auto-installs [ASO analysis skill](https://github.com/sickn33/antigravity-awesome-skills)) |
| **Screenshot Automation** | `asc screenshots run` capture → `frame` device bezels → `upload` to ASC |
| **Pricing** | Query price points, create price schedules, manage territory availability |
| **Review Guidelines** | 4.2/4.3/5.1.1 checks, submission pacing advice, Review Notes guidance |
| **Submission** | `asc validate --strict` preflight → `asc submit` → `asc status` monitoring |
| **Final Checklist** | Build quality, privacy compliance, metadata, feature depth, anti-spam verification |

---

## Workflow Overview

```
Configure Auth → Register Bundle ID → Archive Build → Upload IPA
    → Configure Metadata (bilingual) → ASO Optimization → Screenshot Automation
    → Pricing Setup → Review Compliance Check → Submit for Review → Monitor Status
```

The Agent executes each stage in order. You only need to confirm at key checkpoints.

---

## FAQ

<details>
<summary><strong>Do I need to install anything beforehand?</strong></summary>

No. The Skill includes complete prerequisite steps. The Agent will guide you through installing asc CLI, configuring your API Key, and fetching signing certificates. You just need an Apple Developer account and Xcode.
</details>

<details>
<summary><strong>What types of apps does this support?</strong></summary>

Primarily designed for paid iOS apps (no IAP, no subscriptions, no ads, no backend). If your app has in-app purchases or subscriptions, the pricing and metadata sections may need adjustment.
</details>

<details>
<summary><strong>How does screenshot automation work?</strong></summary>

asc CLI v0.29+ includes an experimental screenshot pipeline. The Agent automatically writes a screenshot plan (JSON) → captures from the simulator → frames with [Koubou](https://github.com/nicklama/koubou) device bezels → uploads to App Store Connect. Requires `pip install koubou==0.14.0`.
</details>

<details>
<summary><strong>Is the ASO optimization reliable?</strong></summary>

The Skill includes battle-tested ASO strategies (keyword optimization rules, description templates, localization priorities), and also auto-installs the professional [ASO analysis skill](https://github.com/sickn33/antigravity-awesome-skills) for data-driven keyword scoring and competitor analysis.
</details>

<details>
<summary><strong>How do I avoid app review rejection?</strong></summary>

The Skill includes a comprehensive review survival guide covering the most common rejection reasons (4.2 minimum functionality, 4.3 spam/copycat, 2.1 app completeness, 5.1.1 privacy compliance), plus 2025–2026 policy changes. Before submission, the Agent automatically runs `asc validate --strict` for preflight checks.
</details>

<details>
<summary><strong>Which AI Agents are compatible?</strong></summary>

Any AI Agent that supports Claude Code Skills, including Claude Code, Cursor, Windsurf, and more.
</details>

---

## Acknowledgments

[App Store Connect CLI (asc)](https://github.com/rudrankriyam/App-Store-Connect-CLI) · [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills) · [claude-code-apple-skills](https://github.com/rshankras/claude-code-apple-skills) · [Koubou](https://github.com/nicklama/koubou)

## License

[MIT](LICENSE)
