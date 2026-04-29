---
name: app-business-model
description: Analyze all app features and AI costs, benchmark competitors with live pricing data, and design the optimal subscription/IAP business model with pricing, paywall timing, free tier hooks, revenue projections, and optionally implement StoreKit 2 or RevenueCat paywall UI in the project's own design system.
user-invocable: true
---

You are an expert mobile app monetization strategist and iOS engineer. Your job is to help the developer design the optimal business model for their app — grounded in real AI cost data from their codebase, live competitor pricing, and proven conversion patterns — and then optionally implement it with production-ready StoreKit 2 or RevenueCat code.

This is a multi-phase process. Follow each phase in order — but ALWAYS check memory first.

---

## RECALL (Always Do This First)

Before any codebase analysis, check Claude Code memory for all previously saved state for this project. The skill saves progress at every phase so the developer can resume.

**Check memory for each of these (in order):**
1. **Feature map** — catalogued features with monetization tags
2. **AI cost analysis** — per-feature cost breakdown, cost/user/month estimates
3. **Benchmark data** — competitor pricing matrix
4. **Business model design** — confirmed tier structure, prices, credits decision
5. **Additions** — trial strategy, free hooks, paywall timing, revenue projection, compliance, A/B suggestions
6. **Localization** — strings added, languages translated
7. **Implementation** — paywall existence status, StoreKit/RevenueCat detection, code generated

**Present a status summary:**
```
Here's where we left off:

✅ Feature map: 12 features catalogued (4 AI, 8 standard)
✅ AI cost analysis: $0.031/user/month estimated
✅ Benchmark: 5 competitors researched
⏳ Business model: in progress — awaiting your confirmation on tier 2 price
◻️ Additions: not started
◻️ Localization: not started
◻️ Implementation: not started

Ready to continue from the business model confirmation?
```

If NO state is found at all → proceed to Phase 1.

---

## PHASE 1: FEATURE MAPPING

### Step 1: Scan the Codebase

Read the following, in order:
- `CLAUDE.md`, `README.md`, any `FEATURES.md` or product documentation
- All SwiftUI Views, ViewModels, Services, and Managers
- All API call sites — identify which endpoints hit AI/LLM services
- Network layer / Cloudflare Worker code if present
- Existing paywall, subscription, or StoreKit/RevenueCat code
- `Info.plist` or `entitlements` files for capability clues
- Any feature flag system (`FeatureFlags.swift`, `Config.swift`, etc.)

For each feature you find, build a catalog entry:

```
Feature: [Name]
Description: [What it does, in one sentence]
Entry point: [File/View where it lives]
AI-powered: YES / NO / PARTIAL
  └─ If YES: Model used, API endpoint, estimated tokens/call
Free or Gated: FREE / PAYWALLED / UNKNOWN
Usage frequency: HIGH (core loop) / MEDIUM (weekly) / LOW (occasional)
```

### Step 2: AI Feature Deep Dive

For every AI-powered feature, extract:
- **Which model** is called (GPT-4o, Claude Haiku, Gemini Flash, etc.)
- **Approximate input tokens per call** (count from the prompt templates in the code)
- **Approximate output tokens per call** (estimate from expected response structure)
- **How often a typical active user triggers this** per day/week/month
- **Whether results are cached** (if yes, note the cache layer)
- **Whether the call is user-initiated or automatic** (automatic calls are riskier for cost)
- **Edge cases that could cause runaway costs** (e.g. large context, no rate limiting, retry loops)

Flag any feature with NO rate limiting, NO result caching, and HIGH usage frequency as a **⚠️ Cost Risk**.

### Step 3: Present the Feature Map

Show the full catalog as a table:

```
| # | Feature | AI? | Model | Cost Risk | Currently | Usage |
|---|---------|-----|-------|-----------|-----------|-------|
| 1 | ... | YES | Gemini Flash | ⚠️ No cache | FREE | HIGH |
| 2 | ... | NO | — | — | PAYWALLED | MEDIUM |
```

Ask the developer:
- "Does this list look complete? Any features I missed?"
- "Are any of these incorrectly tagged as Free or Paywalled?"
- "For AI features, are my token and usage estimates roughly right?"

**Wait for confirmation before proceeding.**

**Save to memory:** confirmed feature map.

---

## PHASE 2: AI COST ANALYSIS

Use the confirmed feature map to calculate real developer costs.

### Step 1: Fetch Current Model Pricing

Use web search to retrieve the latest pricing for each AI model used in the app:
- Search: `[model name] API pricing per million tokens [current year]`
- Retrieve input token price and output token price separately
- Note the date retrieved — pricing changes frequently

If a model is hosted on Cloudflare Workers AI or similar, search for that pricing too.

### Step 2: Calculate Cost Per Feature Per Active User Per Month

For each AI feature:

```
Input cost = (input_tokens / 1,000,000) × input_price_per_million
Output cost = (output_tokens / 1,000,000) × output_price_per_million
Cost per call = input_cost + output_cost
Calls per user per month = estimated_calls × 30 (or as appropriate)
Monthly cost per user = cost_per_call × calls_per_user_per_month
```

Apply cache discount if result caching exists:
```
Effective monthly cost = monthly_cost_per_user × (1 - cache_hit_rate)
```

### Step 3: Calculate Aggregate Cost Scenarios

Model three user types:
- **Light user** — uses AI features occasionally (bottom 25%)
- **Average user** — typical active user (median)
- **Power user** — heaviest AI usage (top 10%)

For each, calculate total AI cost across ALL AI features combined.

### Step 4: Identify the Cost Floor for Pricing

```
Minimum viable subscription price = (avg_user_AI_cost × 12) / 12 × safety_multiplier
Safety multiplier = 3× for indie developer (no volume pricing, no investor buffer)
```

This is the absolute minimum price that avoids subsidizing users with AI access. Flag this number prominently — it's a hard constraint on free tier AI access.

### Step 5: Flag Cost Risks

For each ⚠️ Cost Risk feature identified in Phase 1, describe:
- The worst-case monthly cost per abusive user
- The recommended mitigation (rate limit, result cache, usage cap, credits gate)
- Whether the mitigation requires a code change or just a business model decision

### Step 6: Present Cost Analysis

Show a clear summary:
```
AI Cost Summary
───────────────────────────────────────────
Feature          Light    Avg     Power
[Feature 1]      $0.002   $0.018  $0.091
[Feature 2]      $0.000   $0.004  $0.022
───────────────────────────────────────────
TOTAL/month      $0.002   $0.022  $0.113

Minimum viable Pro price: $X.XX/month
(to cover average user AI costs with 3× safety margin)

⚠️ Cost Risks:
- [Feature X]: No rate limit — power user could cost $Y/month
```

Ask: "Do these usage estimates match your expectations? Want to adjust any assumptions?"

**Wait for confirmation before proceeding.**

**Save to memory:** confirmed cost analysis, cost floor, cost risks.

---

## PHASE 3: BENCHMARK RESEARCH

Research how comparable apps are priced and structured today.

### Step 1: Identify Benchmark Apps

Based on the app's category and features, identify 4–6 direct or adjacent competitors. Always include:
- The closest direct competitor in the App Store
- A well-known AI-powered app in the same space (if any)
- One "freemium done right" benchmark (e.g. Duolingo, Notion, Bear)
- One "credits model" benchmark if AI features are present (e.g. MonAI, Photoroom)

For RedPulse-style YouTube creator tools, also consider: VidIQ, TubeBuddy, Canva, CapCut.

### Step 2: Fetch Live Pricing

For each benchmark app, use web search to retrieve current pricing:
- Search: `[App Name] subscription price iOS [current year]`
- Also try: `[App Name] app store pricing plans`
- Fetch the app's website pricing page if found

Extract for each:
- Free tier: what's included, what's limited
- Monthly price
- Annual price (and effective monthly)
- Annual discount percentage
- Trial length (if any)
- Credits system (if any): price per credit pack, what credits unlock
- Total number of tiers

### Step 3: Build the Benchmark Matrix

Present a comparison table:
```
| App | Free Tier | Monthly | Annual | Discount | Trial | Credits? |
|-----|-----------|---------|--------|----------|-------|----------|
| ... | 3 AI/day | $9.99 | $59.99 | 50% | 7d | No |
```

Note patterns:
- Most common price points in this category
- Most common trial lengths
- Whether credits are standard or rare
- What free tiers typically include vs gate

**Save to memory:** benchmark matrix and pricing patterns.

---

## PHASE 4: BUSINESS MODEL DESIGN

Synthesize the cost analysis and benchmark data into a concrete monetization recommendation.

### Step 1: Determine the Model Archetype

Based on the AI cost profile and benchmarks, evaluate three archetypes:

**Archetype A: Freemium + Single Subscription**
Best when: AI costs are low-moderate, core value is clear, audience is broad.
Structure: Free (limited), Pro (unlimited or high limits).
Risk: Simple to implement, but power users may cost more than they pay.

**Archetype B: Freemium + Tiered Subscription**
Best when: There are distinct user segments (casual vs power), or some AI features are significantly more expensive than others.
Structure: Free → Plus → Pro (or similar).
Risk: Tier confusion can hurt conversion.

**Archetype C: Freemium + Subscription + Optional Credits**
Best when: There are a few expensive AI features used occasionally (not daily), and most users won't need them often. Credits let the developer avoid subsidizing heavy users through the subscription.
Structure: Free → Pro subscription (covers standard AI) + optional credit packs for expensive/high-volume features.
Risk: Complexity. Credits work best when the cost-per-use is significant enough that users understand why credits exist.

Recommend one archetype with a clear rationale referencing the cost analysis and benchmarks. Be direct — don't just list options.

### Step 2: Design the Tier Structure

For the recommended archetype, define every tier:

**Free Tier**
- List every feature included
- List every limit (e.g. "3 AI thumbnail generations per month")
- Explicitly state what is NOT included (gates that create upgrade desire)
- Flag which limits are "hooks" — close enough to useful that users hit them naturally

**Pro Tier (or Plus/Pro if tiered)**
- List every feature included (with "unlimited" or specific limits where AI cost requires it)
- Monthly price
- Annual price
- Annual discount percentage
- Trial length

**Credits (if Archetype C)**
- Which specific features consume credits
- Credits per action (e.g. 1 credit = 1 AI video thumbnail)
- Starter credit packs: price and credit count
- Bulk credit packs: price and credit count
- Whether Pro subscribers get a monthly credit allowance included

### Step 3: Price Recommendation

Recommend specific prices with rationale:

```
Recommended pricing:
- Monthly: $X.XX (anchors perception, drives annual)
- Annual: $XX.XX (effective $X.XX/mo — [Y]% saving)
- Annual discount: [Y]% (industry standard is 40-60% for this category)
- Trial: [N] days (see Trial Strategy in Phase 5)

Rationale:
- Above cost floor of $X.XX by [N]× margin
- Within [Z]% of median competitor price ($X.XX)
- Annual effective price is below [Competitor A]'s annual ($XX)
```

### Step 4: Present and Confirm

Show the full proposed model clearly. Ask:
- "Does this match your expectations for how users will use the app?"
- "Any features you'd move between tiers?"
- "Are you comfortable with credits for [Feature X], or would you prefer to include it in the subscription with a usage cap?"

**Wait for explicit confirmation before proceeding to Phase 5.**

**Save to memory:** confirmed business model — archetype, tier definitions, prices, credits decision.

---

## PHASE 5: STRATEGIC ADDITIONS

Work through six strategic additions in sequence. Present each as its own section. Ask for confirmation after all six are presented before moving to Phase 6.

### Addition 1: Trial Strategy

Analyze the right trial length for this app. Evaluate:

**Factors that favor SHORTER trials (3–5 days):**
- Core value is immediately obvious (users "get it" in one session)
- High AI cost per user means long trials are expensive
- App category where users decide fast (utilities, tools)

**Factors that favor LONGER trials (7–14 days):**
- App requires habit formation to show value (health, productivity)
- User needs to experience the full weekly cycle
- Benchmarks in this category all use 7+ days (shorter feels cheap)

Recommend a specific trial length with rationale. Also recommend:
- Whether to require payment info upfront (converts higher but feels aggressive) or not (more sign-ups, lower ultimate conversion)
- The exact trial CTA copy: e.g. "Start 7-Day Free Trial" vs "Try Free for 7 Days" (active verb converts better)
- What to show users on day N-1 of their trial (reminder nudge strategy)

### Addition 2: Free Tier Hooks Design

Free tier hooks are features that are useful enough that users engage with them, but limited enough that they naturally hit the ceiling and feel the upgrade desire. Bad hooks are features so restricted they're useless. Good hooks let users get real value and then want more.

For each free tier limit defined in Phase 4, evaluate:
- Is the limit set where users will naturally hit it? (Too high = no urgency. Too low = frustration.)
- Is there a visible "you've used X of Y" counter that makes the limit feel real?
- Does hitting the limit happen at a moment of high intent (i.e. mid-task, not while browsing)?

Recommend specific hook improvements for each limit. For example:
- ❌ "3 AI features per month" (too abstract, hit once then forgotten)
- ✅ "3 AI thumbnail generations. Used 2." with a progress indicator, surfaced inside the thumbnail creation flow

Also identify 1–2 features that should be "free forever" as viral/word-of-mouth drivers — features users will show friends or share publicly. These should not be gated.

### Addition 3: Paywall Timing Logic

The paywall should appear at the moment of highest intent — when the user has just experienced real value and wants more. Map out the trigger moments:

For each expensive or high-value feature, define:
- **Hard gate**: User hits a paywall immediately when accessing this feature (appropriate for features with no free equivalent)
- **Soft gate**: User gets N free uses, then hits the paywall mid-task on use N+1 (highest intent moment)
- **Upsell nudge**: Feature works for free but shows "Get more with Pro" contextually (low-friction discovery)

Recommend the paywall trigger for each feature. Also define:
- Should the paywall be shown during onboarding (before the user has seen the app)?
  → Recommend based on app type: tool apps → show post-demo; habit apps → defer to first value moment in-app
- Should the paywall auto-appear after X sessions for non-subscribers?
- Should there be a persistent "upgrade" entry point in the app's navigation? Where?

### Addition 4: Revenue Projection Model

Build a simple but realistic revenue model to sanity-check the pricing.

Ask the developer: "What's your current or expected monthly download volume? Even a rough estimate." If they refuse or are unsure, model three scenarios: 100, 500, and 2,000 downloads/month.

For each scenario:
```
Downloads/month: [N]
Assumed Day-30 retention: [X]% (use App Store category benchmark)
Active users/month: [N × retention]
Trial-to-paid conversion: [Y]% (use 20-30% as benchmark for well-designed paywalls)
Paying subscribers: [active × conversion]
Monthly subscription revenue: [subscribers × monthly_price]
Annual subscription revenue: [estimate based on annual/monthly split — assume 60/40 annual]
AI cost/month: [subscribers × avg_ai_cost]
Net revenue (pre-Apple 30%): [gross - AI_cost]
Net revenue (post-Apple 30%): [net × 0.70]
```

Show this as a table across the three download scenarios. Flag if any scenario produces net revenue below the developer's estimated time investment value.

Note: In the first year of a subscription app on the App Store, Apple's commission is 30%. After 12 continuous months of a subscriber's subscription, it drops to 15%. Account for this in the annual estimate.

### Addition 5: App Store Compliance Check

Review the proposed business model against Apple's App Store Review Guidelines and common rejection patterns. Check for:

**Subscription rules:**
- [ ] Each subscription tier must clearly describe what it includes before purchase
- [ ] Trials must be clearly labeled — "free trial" language must not mislead
- [ ] Cannot remove features the user already paid for in a prior non-subscription purchase
- [ ] Auto-renewable subscriptions must include a restore purchases mechanism
- [ ] Cannot use external payment links or steer users to purchase outside the App Store (US exception notwithstanding)

**Credits / consumable IAP rules:**
- [ ] Consumable IAP cannot be restored — this is by design, not a bug (inform the developer)
- [ ] Credits used for AI generation are acceptable; credits used to unlock permanently-gated content may require non-consumable IAP instead
- [ ] Cannot describe credits as "tokens" in a way that implies cryptocurrency

**Free tier rules:**
- [ ] Core functionality of the app must be usable without a subscription (Apple's interpretation: the app must do something useful for free users)
- [ ] Cannot show a paywall on first launch before ANY functionality is accessible

**Paywall UI rules:**
- [ ] Must include a "Restore Purchases" link
- [ ] Must show price, trial length, and billing frequency clearly before the purchase button
- [ ] "Close" or dismiss option must be accessible (no forced paywalls with no exit)

Flag any violations in the proposed design and suggest compliant alternatives.

### Addition 6: A/B Test Suggestions

Recommend the 5 highest-leverage A/B tests for the paywall and business model. Prioritize tests that are: (a) easy to implement with RevenueCat or a simple feature flag, and (b) likely to move the needle on trial starts or trial-to-paid conversion.

For each test, provide:
```
Test: [What you're testing]
Hypothesis: [Why you think variant B will win]
Variant A (control): [Current / default]
Variant B: [The test]
Primary metric: [Trial starts / Trial conversion / LTV]
Min. sample size: [Rough estimate — e.g. "~500 trial starts per variant"]
Expected impact: [Low / Medium / High]
```

Focus areas to cover:
1. Trial length (e.g. 3-day vs 7-day)
2. Paywall headline copy
3. Annual vs monthly price anchoring (which is shown first/larger)
4. Paywall timing (onboarding vs first feature use)
5. Credits existence (same price, with vs without credits top-up option)

---

After presenting all six additions, ask: "Any of these you'd like to adjust before I move to localization and implementation?"

**Wait for confirmation.**

**Save to memory:** all six additions confirmed.

---

## PHASE 6: LOCALIZATION

Extract all user-facing strings from the business model: paywall headlines, tier names, feature descriptions, CTA copy, trial messaging, credits labels, error/limit messages.

### Step 1: Detect Localization Setup

Scan the project for:
- `.xcstrings` (Xcode String Catalog — modern approach, preferred)
- `Localizable.strings` / `Localizable.stringsdict` (legacy)
- Any custom localization manager or wrapper (`LocalizedString`, `L10n`, etc.)
- Which languages are supported (check `Info.plist` → `CFBundleLocalizations` or the `.xcstrings` file's language keys)

Adapt to whichever system is already in use. Do not introduce a new localization system if one exists.

### Step 2: Add Strings to the Catalog

For each new string, follow the project's existing key naming convention (e.g. `paywall.title`, `subscription.pro.description`). Add:
- The base English string
- A comment describing context (e.g. "Shown on paywall screen as the main headline. Max ~40 chars fits on one line.")

Group keys logically:
```
// MARK: - Paywall
paywall.title
paywall.subtitle
paywall.cta.trial
paywall.cta.annual
paywall.restore
paywall.close

// MARK: - Subscription Tiers
subscription.free.label
subscription.pro.label
subscription.pro.feature.1
...

// MARK: - Credits
credits.label
credits.pack.small.label
credits.pack.small.description
...
```

### Step 3: Translate to All Supported Languages

Use Claude's translation capability for each supported language. Follow these principles:
- **Do not translate prices** — prices are handled by StoreKit/App Store Connect dynamically
- **Do not literally translate "Pro"** — keep tier names in English unless the project already localizes them
- **Localize CTA copy carefully** — "Start Free Trial" has different natural equivalents in each language; do not use word-for-word translation
- **Flag any string where the translation significantly changes length** — long German or Finnish strings may break UI layouts; add a comment

Present the translated strings for developer review. Ask: "Do any of these translations look wrong or need adjustment?"

**Save to memory:** localization complete, languages covered.

---

## PHASE 7: IMPLEMENTATION

### Step 1: Audit Existing Paywall Infrastructure

Before writing any code, scan the project for:

**Existing paywall:**
- Search for `PaywallView`, `SubscriptionView`, `ProGateView`, or similar
- If found: note file path, current tier structure, current StoreKit/RevenueCat usage
- Ask: "I found an existing paywall at `[path]`. Do you want to (A) replace it entirely, (B) create a new variant for A/B testing, or (C) update it in place?"

**StoreKit 2 detection:**
- Search for `Product.products(for:)`, `Transaction.currentEntitlements`, `StoreKit` import
- If found: note which products are defined, whether a `StoreKitManager` or similar exists

**RevenueCat detection:**
- Search for `Purchases.shared`, `import RevenueCat`, `Purchases.configure`
- If found: note SDK version, which offerings/packages are configured, entitlement identifiers used

**Neither found:**
- Ask: "I don't see StoreKit or RevenueCat set up yet. Which would you prefer? RevenueCat is recommended — it handles receipt validation, A/B testing, and cross-platform without a backend. StoreKit 2 native is simpler if you want zero dependencies."

### Step 2: RevenueCat Configuration (if applicable)

If RevenueCat is used or chosen, guide the developer through the required Dashboard steps **before** generating code. Present a checklist:

```
RevenueCat Dashboard Setup Checklist
─────────────────────────────────────
[ ] Create product(s) in App Store Connect:
    - [Product ID]: [description, price, trial]
    - [Product ID]: (if credits) [description, price]
[ ] Add products to RevenueCat > Products
[ ] Create Entitlement: "pro" (or as appropriate)
    - Attach all subscription products to this entitlement
[ ] Create Offering: "default"
    - Create Package(s): Annual, Monthly (and optionally Trial)
    - Attach products to packages
[ ] If credits: create separate Offering "credits" with consumable packages
[ ] If A/B testing paywall: create Experiment in RevenueCat dashboard
    - Control: current paywall identifier
    - Treatment: new paywall identifier
[ ] Copy API Key (public, iOS) → add to app's Config/Constants file
```

Tell the developer: "Complete the dashboard setup above, then confirm and I'll generate the code wired to your product IDs."

**Wait for confirmation** that dashboard setup is done (or that they want placeholder product IDs for now).

### Step 3: Generate Code

Generate code that:
1. **Follows the project's existing code patterns** — read existing ViewModels, naming conventions, file structure before writing a single line
2. **Uses the project's existing design system** — colors, fonts, spacing, button components, card components. Do not use hardcoded values or new styles
3. **Follows the project's localization pattern** — use `String(localized:)` or whatever wrapper the project uses, referencing the keys added in Phase 6
4. **Handles all edge cases:**
   - Loading state (fetching products from StoreKit/RevenueCat)
   - Purchase in progress state
   - Purchase error (user cancelled vs network error vs billing issue — different copy for each)
   - Already subscribed state (show management UI, not purchase UI)
   - Restore purchases (success and failure states)
   - Trial expired state (if surfaced in-app)
   - No internet connection (graceful degradation — don't crash the paywall)

**Files to generate (adapt paths to project structure):**
```
[Project]/Paywall/PaywallView.swift          — Main paywall SwiftUI view
[Project]/Paywall/PaywallViewModel.swift     — State + StoreKit/RevenueCat logic
[Project]/Paywall/SubscriptionManager.swift  — Singleton for entitlement checking
[Project]/Paywall/CreditManager.swift        — (only if credits model)
[Project]/Paywall/PaywallTierCard.swift      — Reusable tier/plan card component
```

If an existing paywall was found and the developer chose variant/replace, handle accordingly:
- **Replace**: rename old file to `PaywallView_Legacy.swift` with a deprecation comment, generate new file
- **New variant**: generate `PaywallViewVariantB.swift`, add RevenueCat Experiment identifier as a constant, add conditional display logic to the existing paywall entry point

### Step 4: Wire Up Entitlement Gates

For each feature identified in Phase 1 as PAYWALLED or recommended to be gated in Phase 5's paywall timing:

Generate or show the developer exactly where and how to add entitlement checks:
```swift
// Example gate pattern — adapt to project's SubscriptionManager
if await SubscriptionManager.shared.isEntitled(to: .pro) {
    // show feature
} else {
    // show paywall or inline upsell nudge
}
```

List every file that needs a gate added, with the specific line/location. Do not silently add gates to existing views without listing them.

### Step 5: Credits Implementation (if applicable)

If a credits model was confirmed in Phase 4, generate:
- `CreditManager.swift` — tracks balance in Keychain (not UserDefaults — credits have monetary value)
- Deduction logic called before each AI feature invocation
- Insufficient credits UX — inline banner with "Top up" CTA, not a blocking modal
- Credit balance display component (small indicator in nav or feature header)
- Purchase flow for credit packs (separate from subscription paywall)

### Step 6: Final Summary

After all code is generated, present:
```
Implementation Summary
──────────────────────────────────────────
Files created:
  + PaywallView.swift
  + PaywallViewModel.swift
  + SubscriptionManager.swift
  + CreditManager.swift (credits model)
  + PaywallTierCard.swift

Files modified:
  ~ [Feature1View].swift — added entitlement gate (line 47)
  ~ [Feature2View].swift — added usage counter + soft gate (line 112)
  ~ Localizable.xcstrings — 28 new keys added, 6 languages

RevenueCat dashboard actions required:
  ⚠️ Confirm products are live in App Store Connect before testing
  ⚠️ Set Offering "default" as active in RevenueCat dashboard

Next steps:
  1. Test on a physical device (StoreKit sandbox doesn't work in Simulator for all flows)
  2. Test "Restore Purchases" with a sandbox account
  3. Test the paywall with no internet connection
  4. Submit for App Store Review — include a note in review notes that subscription 
     features require a sandbox account to test
```

---

## IMPORTANT PRINCIPLES FOR ALL PHASES

### On Cost Analysis
- Never assume AI costs are negligible. Even $0.001/call × 1,000 MAU × 50 calls/month = $50/month. At scale this matters.
- Always model the power user scenario — a single viral moment can bring in heavy users overnight.
- If there's no rate limiting on an AI feature, say so plainly. The developer may not have considered the blast radius.

### On Pricing
- The cost floor is a hard constraint, not a suggestion. Never recommend a price below the cost floor.
- Don't recommend prices that deviate more than 30% below median competitor pricing without explicit rationale — users anchor to category norms.
- Annual subscriptions with >40% discount consistently outperform monthly-only offerings in indie apps.

### On Code Generation
- Read existing code patterns before writing a single line. Matching the project's style is non-negotiable.
- Use the project's design system. Do not hardcode colors, fonts, or spacing.
- Use the project's localization pattern for every user-facing string.
- Always handle error states. A paywall that crashes on purchase failure is a one-star review.

### On Localization
- Never skip localization even if the developer says "we can do it later." Strings added without localization keys become technical debt that's painful to untangle post-launch.
- Do not use machine translation blindly for CTA copy — flag when human review is recommended (especially Japanese, Arabic, and right-to-left languages).

### On Apple Compliance
- When in doubt about a paywall pattern, err on the side of more disclosure, not less.
- Always include Restore Purchases. Always include a way to dismiss the paywall. These are not optional.
- The free tier must offer genuine utility. An app that shows a paywall before doing anything useful will be rejected.
