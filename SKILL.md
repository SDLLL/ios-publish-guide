---
name: appstore-publisher
description: End-to-end iOS App Store publishing workflow using asc CLI. Covers build & upload, metadata, ASO optimization, screenshot automation, pricing, review guidelines, and submission. This skill should be used when publishing, submitting, or uploading an iOS app to the App Store, or when configuring App Store metadata, screenshots, and pricing via CLI.
---

# ASC CLI App Store Publishing Complete Guide

> Agent Operations Handbook — Execute each phase in order to complete the full workflow from build to publication.
> Based on asc v0.35.2, designed for paid iOS apps (no IAP, no subscriptions, no backend).

---

## 0. Prerequisites

### 0.1 Install asc

```bash
# Option 1: Homebrew (recommended)
brew install asc

# Option 2: Build from source
git clone https://github.com/rudrankriyam/App-Store-Connect-CLI.git
cd App-Store-Connect-CLI
go build -o /usr/local/bin/asc .

# Verify installation
asc --version
```

### 0.2 Configure API Key

Direct the user to [App Store Connect → Keys](https://appstoreconnect.apple.com/access/integrations/api) to create an API Key and download the `.p8` file.

```bash
# Login (interactive, stores credentials in the system keychain)
asc auth login

# Or configure via environment variables (suitable for CI / Agent)
export ASC_KEY_ID="YOUR_KEY_ID"
export ASC_ISSUER_ID="YOUR_ISSUER_ID"
export ASC_PRIVATE_KEY_PATH="/path/to/AuthKey_XXXXXX.p8"

# Verify authentication status
asc auth status
asc doctor --output table
```

**Notes:**
- The API Key requires Admin or App Manager permissions
- The `.p8` file can only be downloaded once — store it securely
- If authentication issues arise, run `asc doctor` to diagnose

### 0.3 Register Bundle ID

```bash
asc bundle-ids create \
  --identifier "com.yourcompany.appname" \
  --name "AppName" \
  --platform IOS
```

**Verify:** `asc bundle-ids list` to confirm creation.

### 0.4 Fetch Signing Certificates and Provisioning Profiles

```bash
asc signing fetch \
  --bundle-id com.yourcompany.appname \
  --profile-type IOS_APP_STORE \
  --output ./signing
```

Output: `.cer` and `.mobileprovision` files in the `./signing/` directory.

**Notes:**
- If using xcodebuild cloud signing (`-allowProvisioningUpdates -authenticationKeyPath`), this step can be skipped
- The `teamID` in `ExportOptions.plist` must match your developer team ID

---

## 1. Build and Upload

### 1.1 Create the App on the App Store Connect Web Portal

**This step cannot be performed via CLI — it must be done manually.**

Go to [App Store Connect → Apps → +](https://appstoreconnect.apple.com/apps) and create a new App:
- Select the registered Bundle ID
- Enter the App name (metadata can be modified via CLI later)
- Select the primary language
- Enter the SKU (recommended: use the bundle ID suffix)

After creation, note the **App ID** (numeric) — it will be needed for subsequent commands:

```bash
# Look up the App ID
asc apps --output table
```

### 1.2 Build and Archive

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

Minimal `ExportOptions.plist` configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>app-store-connect</string>
    <key>teamID</key>
    <string>YOUR_TEAM_ID</string>
    <key>uploadSymbols</key>
    <true/>
    <key>destination</key>
    <string>upload</string>
</dict>
</plist>
```

### 1.3 Upload IPA

```bash
asc publish appstore \
  --app "APP_ID" \
  --ipa ./build/AppName.ipa \
  --version "1.0.0" \
  --wait
```

`--wait` waits for Apple to process the build (typically 5–15 minutes).

**Verify:** `asc builds list --app "APP_ID" --limit 1 --output table`

**Notes:**
- Upload timeout can be configured via `ASC_UPLOAD_TIMEOUT=30m`
- If the IPA is large, be patient — do not upload repeatedly
- `Info.plist` must include `ITSAppUsesNonExemptEncryption = NO`, otherwise an encryption compliance questionnaire will appear after each upload

---

## 2. Metadata Configuration

### 2.1 App Information (Bilingual)

```bash
# Chinese metadata
asc app-info set \
  --app "APP_ID" \
  --locale "zh-Hans" \
  --description "Your App's Chinese description..." \
  --keywords "keyword1,keyword2,keyword3" \
  --support-url "https://yoursite.com/support" \
  --marketing-url "https://yoursite.com"

# English metadata
asc app-info set \
  --app "APP_ID" \
  --locale "en-US" \
  --description "Your App English description..." \
  --keywords "keyword1,keyword2,keyword3" \
  --support-url "https://yoursite.com/support" \
  --marketing-url "https://yoursite.com"
```

**Verify:** `asc app-info get --app "APP_ID" --output table`

### 2.2 Age Rating

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

Adjust each value based on the actual App content (`NONE` / `INFREQUENT_OR_MILD` / `FREQUENT_OR_INTENSE`).

### 2.3 Review Information

```bash
# Get the version ID
asc versions list --app "APP_ID" --output table

# Create review details
asc review details-create \
  --version-id "VERSION_ID" \
  --contact-email "your@email.com" \
  --contact-first-name "YourName" \
  --contact-last-name "YourLastName" \
  --contact-phone "+86xxxxxxxxxx" \
  --notes "Review notes: Describe the App's purpose, testing steps, and special feature instructions"
```

**Tips for Writing Review Notes:**
- Open with a clear statement of what the App does
- If functionality is simple, proactively explain its unique value
- If special actions are required to experience the full functionality, describe the steps
- Never leave it blank — empty Review Notes increase the chance of rejection

---

## 3. ASO Optimization

ASO (App Store Optimization) should be completed before writing metadata.

### 3.0 Install the ASO Analysis Tool

Before performing ASO optimization, the Agent must first check and install the professional ASO analysis skill:

```bash
# Check if already installed
ls ~/.claude/skills/app-store-optimization/SKILL.md 2>/dev/null || \
  npx skills add sickn33/antigravity-awesome-skills --skill app-store-optimization --all -y
```

Once installed, you gain full ASO analysis capabilities: keyword scoring models, competitor analysis frameworks, metadata optimizers, ASO health score assessments, and more. **When conducting keyword research and metadata optimization, invoke the `app-store-optimization` skill to obtain data-driven analysis results.**

> Credit: The ASO analysis skill is from [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills) (MIT License)

### 3.1 Keyword Strategy

**100-character limit — every character counts:**

- Do not repeat words already in the App name or subtitle (Apple indexes them automatically)
- Separate with commas, **no spaces after commas** (wastes characters)
- Prioritize long-tail keywords with high search volume but low competition
- Chinese keywords are split by individual characters, yielding more combinations (e.g., "记账本" is split into "记", "账", "本")
- Use singular forms for English keywords (Apple automatically matches plurals)
- Do not use competitor brand names (violates guidelines and is ineffective)

**Keyword Research Methods:**
1. App Store search suggestions (type a core term and review autocomplete suggestions)
2. Competitor App keyword analysis (examine their names, subtitles, and descriptions)
3. Apple Search Ads popularity data for reference

### 3.2 Description Writing Guidelines

**The first three lines are critical** — users can only see the first three lines in the store; they must tap "More" to expand.

```
Template structure:
Line 1: One sentence stating the App's core value (what problem it solves)
Lines 2-3: The 2-3 most important selling points
---
Lines 4-8: Feature list (emoji + short sentences)
---
Final lines: Privacy statement / support information
```

**Prohibited practices:**
- Do not start with "Welcome to..."
- Do not stuff keywords
- Each App's description must be written independently from scratch — no copy-pasting
- Do not use unverified claims ("the best", "number one")

### 3.3 App Name and Subtitle

- **Name** (30-character limit): Concise and distinctive, include one core keyword
- **Subtitle** (30-character limit): Supplement with functionality or value, include secondary keywords
- Do not use Apple trademarks (e.g., "for iPhone")
- Name + Subtitle + Keywords should complement each other without repetition

### 3.4 Localization Priority

Key markets for paid Apps (sorted by revenue):
1. en-US (required)
2. zh-Hans (required)
3. ja (recommended)
4. en-GB (can reuse en-US)
5. de-DE / fr-FR (optional)

Keywords and descriptions for each language should be **natively written**, not machine-translated. User search behaviors differ significantly across language markets.

---

## 4. Screenshot Automation

asc v0.29+ introduced an experimental screenshot automation pipeline that relies on [Koubou](https://github.com/nicklama/koubou) for device frame compositing.

### 4.1 Install Dependencies

```bash
pip install koubou==0.14.0
```

### 4.2 Write a Screenshot Plan

Create `.asc/screenshots.json`:

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

Supported actions: `launch`, `tap`, `type`, `wait`, `wait_for` (polling), `screenshot`.

### 4.3 Automated Screenshots

```bash
# Ensure the simulator is running and the App is installed
# Execute the screenshot sequence
asc screenshots run --plan .asc/screenshots.json
```

Or capture individual screenshots:

```bash
asc screenshots capture \
  --bundle-id "com.yourcompany.appname" \
  --name "home" \
  --output-dir ./screenshots/raw
```

### 4.4 Add Device Frames

```bash
# Frame a single screenshot
asc screenshots frame \
  --input ./screenshots/raw/01-home.png \
  --device iphone-air \
  --output-dir ./screenshots/framed

# Supported devices:
# iphone-air (default), iphone-17-pro, iphone-17-pro-max
# iphone-16e, iphone-17, mac
```

Advanced usage — use a Koubou YAML config for custom titles and backgrounds:

```bash
asc screenshots frame \
  --config ./koubou.yaml \
  --watch  # Watch for file changes and regenerate automatically
```

### 4.5 Review Preview

```bash
# Generate an HTML comparison report (raw vs. framed screenshots)
asc screenshots review-generate \
  --framed-dir ./screenshots/framed \
  --raw-dir ./screenshots/raw \
  --output-dir ./screenshots/review

# Open the review in a browser
asc screenshots review-open --output-dir ./screenshots/review

# Approve all screenshots that are ready
asc screenshots review-approve --all-ready --output-dir ./screenshots/review
```

### 4.6 Upload Screenshots

```bash
# Get the version localization ID
asc localizations list --version-id "VERSION_ID" --output table

# Upload iPhone screenshots (6.7-inch)
asc screenshots upload \
  --version-localization "LOC_ID" \
  --path "./screenshots/framed" \
  --device-type "IPHONE_65"
```

**Screenshot Rules:**
- Must match the actual App UI
- Text overlays are allowed, but the main content must be real screenshots
- Each App's screenshots must be independently produced
- For most iOS submissions, one set of iPhone (`IPHONE_65`) + one set of iPad (`IPAD_PRO_3GEN_129`) screenshots is sufficient
- Use `asc screenshots sizes` to view supported dimensions, `--all` for the full matrix

---

## 5. Pricing Configuration

### 5.1 View Price Points

```bash
# View available price points
asc pricing price-points --app "APP_ID" --territory "USA" --output table

# Filter by price
asc pricing price-points --app "APP_ID" --territory "USA" --price 0.99
```

### 5.2 Create a Pricing Schedule

```bash
# Set the price (requires obtaining the price-point ID first)
asc pricing schedule create \
  --app "APP_ID" \
  --price-point "PRICE_POINT_ID" \
  --base-territory "USA"
```

### 5.3 Manage Territory Availability

```bash
# View currently available territories
asc pricing availability get --app "APP_ID" --output table

# Set specific territories as available
asc pricing availability set \
  --app "APP_ID" \
  --territory "USA,CHN,JPN,GBR,DEU" \
  --available true
```

**Verify:** `asc pricing schedule get --app "APP_ID" --output table`

---

## 6. Review Guidelines and Pitfalls

### 6.1 Critical Risk: Guideline 4.3 — Spam / App Similarity

**Worst-case consequence: Permanent developer account ban.**

Behaviors that trigger 4.3:
- Multiple Apps sharing the same source code or assets
- Using templates to mass-produce Apps
- Similar binaries, metadata, or designs
- Splitting a single feature into multiple Apps

**Defensive measures:**
- Each App solves a **different problem** and belongs to a **different category**
- Each App has independent UI, independent metadata, and independent screenshots
- Different Apps use **different combinations of system frameworks** (e.g., one uses MapKit, another uses Charts)
- Never submit multiple Apps on the same day — space submissions at least 3–5 days apart
- Never copy-paste App descriptions, keywords, or screenshot copy

### 6.2 High Risk: Guideline 4.2 — Minimum Functionality

**Reviewers spend approximately 5 minutes evaluating.** They will not deeply explore your App — they need to determine its value within a very short timeframe.

**Required "native depth" signals (by importance):**

| Signal | Description |
|--------|-------------|
| Widget (WidgetKit) | Strongest signal for passing 4.2 review |
| App Intents / Siri | Demonstrates deep system integration |
| Multi-page interaction (3+ pages) | Proves the App is not trivial |
| SwiftData persistence | Stateful experience |
| Settings page | User customization |
| Onboarding flow | Helps reviewers quickly understand value |

**Rejection response strategy:**
1. Do not modify immediately — first understand the rejection reason
2. If the reason is vague (template response), **resubmit directly** — you may get a different reviewer
3. If the reason is specific, add targeted features and resubmit
4. Respond to rejections **professionally** — do not argue

**4.2 enforcement is highly inconsistent** — the same App may be rejected by Reviewer A but approved by Reviewer B. Do not abuse Expedited Review — excessive use will flag your account.

### 6.3 Required: Guideline 2.1 — App Completeness

Accounts for **40%** of all rejections, but is entirely preventable:
- App crashes on launch
- Leftover placeholder text ("Lorem ipsum", "TODO", "Coming Soon")
- Broken links
- Incomplete features (unresponsive buttons, empty pages)
- Login required but no test account provided
- Crashes under no-network / weak-network conditions

**Prevention measures:**
- Test on **real devices**, not just the simulator
- Test under no-network / weak-network conditions
- Search the project for all "TODO", "FIXME", "placeholder" strings
- Verify all URLs are accessible
- Provide clear testing instructions in Review Notes

### 6.4 Required: Guideline 5.1.1 — Privacy Compliance

Every App must have the following (even if no data is collected):
- `PrivacyInfo.xcprivacy` — Declare API usage reasons
- Privacy Policy URL — Set in App Store Connect
- In-app privacy policy entry point — Provide a link in the settings page
- App Privacy nutrition labels — Fill out accurately

If using `UserDefaults`, file timestamp APIs, disk space APIs, etc., you must declare Required Reasons in `PrivacyInfo.xcprivacy`.

### 6.5 Submission Cadence

| Phase | Strategy |
|-------|----------|
| Reputation building (first 4 weeks) | 1 per week, submit the most unique and polished Apps |
| Steady acceleration (weeks 5–12) | 2 per week, provided there are no rejections |
| Normal pace (week 13 onward) | 2–3 per week, adjust based on review feedback |
| After a rejection | Resolve the issue before submitting the next App — do not gamble |

**Key principles:**
- **Never submit multiple Apps on the same day**
- Space submissions at least 3–5 days apart
- Maintain professional communication with the review team
- Do not abuse Expedited Review — excessive use will flag your account

### 6.6 2025–2026 Special Considerations

| Timeline | Policy Change |
|----------|---------------|
| From February 2025 | Privacy-related third-party SDKs must include their own privacy manifest files |
| From July 2025 | New age ratings added (13+, 16+, 18+), must be updated by January 31, 2026 |
| From November 2025 | AI transparency requirement — Apps using external AI services must provide a user consent prompt |
| From April 2026 | All submissions must be compiled with the iOS 26 SDK (Xcode 26) |

**Important reminders:**
- Apple now uses **AI-assisted review + human review** in combination, significantly enhancing automated detection of template-based Apps
- If the App does not use any external AI services (purely local functionality), the AI transparency requirement does not apply
- Third-party SDK privacy manifests are the SDK provider's responsibility, but integrators must verify that the SDKs they use include them

---

## 7. Submission

### 7.1 Pre-Submission Validation

```bash
asc validate \
  --app "APP_ID" \
  --version "1.0.0" \
  --platform IOS \
  --strict \
  --output table
```

This command checks: metadata length limits, required fields, review details completeness, primary category, build attachment, pricing schedule, screenshot presence, and age rating.

**`--strict` treats warnings as errors. Always pass the strict check before submitting.**

### 7.2 Submit

```bash
asc submit create \
  --app "APP_ID" \
  --version "1.0.0" \
  --build "BUILD_ID" \
  --confirm
```

Or use one-click publish (upload + attach build + submit):

```bash
asc publish appstore \
  --app "APP_ID" \
  --ipa ./build/AppName.ipa \
  --version "1.0.0" \
  --submit \
  --confirm \
  --wait
```

### 7.3 Monitor Review Status

```bash
# View the full release dashboard
asc status --app "APP_ID" --output table

# View only submission and review status
asc status --app "APP_ID" --include submission,review --output table
```

**Notes:**
- The first version does not support the `--whats-new` parameter (only required for updates)
- If a submission is stuck at READY_FOR_REVIEW, asc will automatically cancel the old submission and create a new one
- Review typically takes 24–48 hours; holidays may extend this

---

## 8. Pre-Submission Ultimate Checklist

The Agent must verify each item before executing `asc submit`:

### Build Quality
- [ ] Tested on a real device or simulator with no crashes
- [ ] No placeholder text (search for TODO / FIXME / placeholder / Lorem)
- [ ] All links are accessible
- [ ] All buttons are responsive
- [ ] No crashes under no-network conditions
- [ ] Onboarding screens display correctly
- [ ] `ITSAppUsesNonExemptEncryption = NO` is added to Info.plist

### Privacy Compliance
- [ ] `PrivacyInfo.xcprivacy` exists and is accurate
- [ ] Privacy Policy URL is configured
- [ ] Privacy policy is accessible within the App
- [ ] All permissions have UsageDescription entries

### Metadata
- [ ] Chinese and English descriptions are configured
- [ ] Keywords are filled in (within 100 characters)
- [ ] Screenshots are uploaded and match the actual UI
- [ ] App name is free of keyword stuffing
- [ ] Category is correctly selected
- [ ] Age rating is completed
- [ ] Review Notes clearly describe the App's purpose
- [ ] Pricing is configured

### Feature Depth
- [ ] Widget is implemented
- [ ] App Intent / Siri Shortcut is registered
- [ ] At least 3 distinct pages
- [ ] SwiftData persistence
- [ ] Settings page exists
- [ ] Onboarding flow exists

### Anti-Similarity (for App series)
- [ ] Visually distinct UI compared to other submitted Apps
- [ ] Description copy has no overlap with other Apps
- [ ] Uses a different combination of system frameworks
- [ ] Belongs to a different App Store category
- [ ] At least 3 days since the last submission

### Final Validation
- [ ] `asc validate --strict` passes entirely
- [ ] `asc status` confirms the build has finished processing

---
