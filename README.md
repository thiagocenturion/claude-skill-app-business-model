# App Business Model Skill

A [Claude Code](https://claude.ai/code) skill that analyzes your app's features and AI costs, benchmarks competitor pricing with live App Store data, and designs the optimal monetization strategy — then optionally implements it with production-ready StoreKit 2 or RevenueCat code matched to your own design system.

## What It Does

Run `/app-business-model` from your app project and the skill will:

1. **Map every feature** in your codebase — tagging each as free, paywalled, or AI-powered, with deep analysis of AI call sites (model, tokens, usage frequency, cost risks)
2. **Calculate real AI costs** — cost per call, cost per active user per month (light/average/power user), and a hard pricing floor below which you'd subsidize AI usage
3. **Benchmark competitors** — fetches live App Store pricing for 4–6 comparable apps via web search, building a matrix of tier structures, trial lengths, and credits models
4. **Design the business model** — recommends one of three archetypes (freemium + single tier, tiered, or subscription + optional credits), defines every tier, and sets prices grounded in cost analysis and competitor benchmarks
5. **Add strategic depth** — trial strategy, free tier hooks, paywall timing logic, revenue projections, App Store compliance check, and A/B test recommendations
6. **Localize all strings** — extracts paywall/subscription copy, adds to your existing localization system (`.xcstrings` or `Localizable.strings`), translates to all supported languages
7. **Implement the paywall** *(optional)* — detects existing StoreKit 2 / RevenueCat setup, guides RevenueCat dashboard configuration, generates SwiftUI paywall code in your design system, and wires entitlement gates across the codebase

## What Makes This Different

Most monetization advice is generic. This skill is grounded in your actual codebase:

- **Cost floor is calculated, not guessed.** The skill reads your AI call sites, fetches current model pricing, and tells you the minimum price that doesn't lose money on AI features.
- **Live competitor data.** Pricing benchmarks are fetched in real-time, not pulled from stale training data.
- **Respects your infrastructure.** If you already have StoreKit 2 or RevenueCat, the skill works within it — and if you have an existing paywall, you choose: replace it, create a variant for A/B testing, or update it in place.
- **RevenueCat dashboard guidance included.** The skill surfaces the required dashboard setup as a blocking checklist before generating code, so you don't discover missing config at runtime.
- **Credits model evaluated, not assumed.** Credits are only recommended when specific AI features have per-call costs high enough that a flat subscription would unfairly subsidize heavy users. The skill explains the reasoning.

## Installation

### Option 1: Add to your Claude Code skills directory

```bash
cd ~/.claude/skills
git clone https://github.com/[your-username]/claude-skill-app-business-model.git app-business-model
```

### Option 2: Add as a dependency in your project's `.claude/settings.json`

```json
{
  "skills": [
    "github:[your-username]/claude-skill-app-business-model"
  ]
}
```

## Usage

From your app's project directory in Claude Code:

```
/app-business-model
```

The skill will analyze your codebase and walk you through each phase interactively. It saves progress at every phase — you can close the session and resume later.

## The 7 Phases

| # | Phase | Output |
|---|-------|--------|
| 1 | Feature Mapping | Full feature catalog with monetization tags and AI cost risk flags |
| 2 | AI Cost Analysis | Per-feature cost breakdown, cost/user/month, pricing cost floor |
| 3 | Benchmark Research | Live competitor pricing matrix (4–6 apps) |
| 4 | Business Model Design | Confirmed tier structure, prices, credits decision |
| 5 | Strategic Additions | Trial strategy · Free hooks · Paywall timing · Revenue projection · Compliance · A/B tests |
| 6 | Localization | All paywall/subscription strings added and translated |
| 7 | Implementation | StoreKit 2 / RevenueCat code, paywall UI, entitlement gates |

Every phase ends with a confirmation gate — the skill waits for your approval before proceeding.

## Requirements

- An iOS app project (SwiftUI preferred, UIKit supported)
- Claude Code with memory enabled (for cross-session resume)
- Internet access (for live competitor pricing in Phase 3 and AI model pricing in Phase 2)
- App Store Connect access (for setting up subscription products in Phase 7)
- RevenueCat account (optional, for Phase 7 — recommended over raw StoreKit 2 for indie developers)

## License

MIT
