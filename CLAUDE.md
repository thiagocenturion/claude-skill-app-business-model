# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## What This Is

A Claude Code skill (`app-business-model`) that analyzes an app's full feature set and AI cost profile, benchmarks competitor pricing via live web search, and designs the optimal monetization strategy — subscription tiers, pricing, free tier hooks, paywall timing, trial strategy, credits model (if warranted), revenue projections, App Store compliance, and A/B test recommendations. Optionally implements the paywall UI and entitlement logic in the project's own design system using StoreKit 2 or RevenueCat.

Invoked via `/app-business-model` from within the app project directory.

## Skill Structure

* **SKILL.md** — The skill prompt. Defines a 7-phase workflow with mandatory confirmation gates between phases. Uses memory to persist state across conversations.
* **CLAUDE.md** — This file. Development guidance for working on the skill itself.

## The 7-Phase Workflow

1. **Feature Mapping** — Catalogs every feature in the codebase. Tags each as Free/Paywalled/AI-powered. Deep-dives on AI call sites to extract model, token counts, usage frequency, and cost risks.
2. **AI Cost Analysis** — Fetches live model pricing via web search. Calculates cost per call, cost per user per month (light/average/power scenarios). Establishes the pricing cost floor.
3. **Benchmark Research** — Identifies 4–6 comparable apps. Fetches live App Store pricing via web search. Builds a competitor pricing matrix with tier structures, trial lengths, and credits presence.
4. **Business Model Design** — Recommends one of three archetypes (single tier, tiered, subscription + credits). Defines every tier in detail — feature lists, limits, prices. Uses cost floor and benchmarks to justify pricing.
5. **Strategic Additions** — Six targeted analyses: trial strategy, free tier hooks, paywall timing logic, revenue projection model, App Store compliance check, A/B test recommendations.
6. **Localization** — Extracts all paywall/subscription strings. Adds to the project's existing localization system (`.xcstrings` or `Localizable.strings`). Translates to all supported languages.
7. **Implementation** — Audits existing paywall infrastructure, guides RevenueCat dashboard setup if needed, generates production-ready SwiftUI paywall code matched to the project's design system and coding patterns, wires up entitlement gates across the codebase.

## Key Design Decisions

* **Cost-first pricing** — The skill establishes a hard cost floor before recommending any price. This prevents a common indie developer mistake: pricing below the cost of serving AI features.
* **Live competitor data** — Uses web search for real-time pricing. Competitor prices change; built-in knowledge goes stale within months.
* **Respects existing infrastructure** — Before generating code, the skill detects existing StoreKit 2 or RevenueCat setup and works within it rather than replacing it. If a paywall exists, the developer chooses: replace, variant, or update in place.
* **RevenueCat dashboard guidance** — RevenueCat requires dashboard configuration before code works. The skill surfaces this as a blocking checklist rather than letting the developer discover missing config at runtime.
* **Credits as optional, not default** — Credits are recommended only when specific AI features have significant per-call costs that would be unfair to subsidize through a flat subscription. The skill evaluates this explicitly rather than defaulting to credits for all AI apps.
* **Memory-driven resume** — All phases save to Claude Code memory so the developer can resume across conversations without restarting analysis.

## Phase Gate Protocol

After every phase, the skill **stops and asks for confirmation** before proceeding. This is intentional:
- The developer may have information not visible in the codebase (internal usage data, planned features, pricing experiments already tried)
- Business model decisions have downstream consequences — a mistake in Phase 4 propagates to Phases 5, 6, and 7
- The confirmation gates make the developer an active participant, not a passive recipient

Never skip a phase gate even if the developer seems eager to move fast.

## Conventions

* The skill runs from the user's app project directory, not from this skill directory.
* SKILL.md frontmatter must include `name`, `description`, and `user-invocable: true`.
* Always check memory for prior state before starting any phase.
* When generating code: read existing patterns first, match naming conventions, use the design system, use the localization system.
* Prices are never hardcoded in generated UI — always pulled from StoreKit/RevenueCat product objects.
* Keychain for credits balance (monetary value), not UserDefaults.
