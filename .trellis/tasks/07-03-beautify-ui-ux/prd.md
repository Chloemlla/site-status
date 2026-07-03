# Beautify UI UX

## Goal

Improve the existing Nuxt/Vue site-status UI into a cleaner, more polished monitoring dashboard while preserving current behavior, data flow, authentication, i18n, and deployment assumptions.

## What I already know

* User asked to beautify UI/UX as much as possible.
* The app is Nuxt 3 + Vue + Naive UI, not React/Tailwind despite some stale frontend spec text.
* Primary surfaces are `SiteNav`, `SiteHeader`, `SiteCards`, `SiteLogin`, `SiteFooter`, and global SCSS.
* Existing UI already has status color gradients, waves, cards, theme/language controls, login, and responsive styles.
* Current styling uses strong animation, viewport font scaling, decorative gradient text, and several broad global selectors that can hurt readability and stability.

## Assumptions

* Preserve all existing business behavior and APIs.
* Prefer a polished status dashboard over a marketing landing page.
* Keep Naive UI and existing icon assets.
* Make improvements directly in the current components instead of adding a new UI framework.

## Requirements

* Refresh global visual tokens: calmer surfaces, better shadows, readable typography, improved light/dark mode contrast.
* Improve the status header hierarchy and mobile layout without changing refresh logic.
* Upgrade monitor cards for scanability: stronger title/status grouping, clearer timeline bars, stable spacing, and better hover/focus behavior.
* Improve login presentation so password-protected mode feels intentional and centered.
* Refine nav and footer so controls feel integrated and do not dominate the UI.
* Reduce excessive animation and avoid layout shifts or text overlap on mobile.
* Rework related frontend component structure when it materially improves the modern dashboard experience.
* Keep all existing i18n keys and data handling intact unless a small copy addition is necessary.

## Acceptance Criteria

* [ ] Existing login, language switch, theme switch, refresh, and monitor list behavior still work.
* [ ] Desktop and mobile layouts avoid overlapping text, unstable card sizing, and oversized animations.
* [ ] Light and dark themes are both visually coherent.
* [ ] Lint and type-check/build validation pass, or any inability to run them is documented.
* [ ] No unrelated backend/API behavior changes are introduced.

## Definition of Done

* Code follows the existing Nuxt/Vue/Naive UI patterns.
* Styling is scoped where possible and global styles are intentional.
* Lint/build checks are run.
* New project knowledge is considered for spec updates.

## Technical Approach

Use conservative component-level SCSS refinements and Naive UI theme overrides. Preserve component structure and state logic, but improve layout density, spacing, surfaces, typography, status affordances, responsive behavior, and reduced-motion handling.

## Decision (ADR-lite)

**Context**: The request is broad and design-oriented, with no explicit brand direction. The repo already has a functional Nuxt/Vue monitoring UI.

**Decision**: Apply a dashboard-style visual refresh in-place instead of rebuilding the app or introducing dependencies.

**Consequences**: This keeps risk low and improves perceived quality quickly. It does not redesign the information architecture or add new analytics/history features.

## Out of Scope

* Backend/API changes.
* New monitoring features or charts.
* New icon libraries or design systems.
* Replacing Naive UI.

## Technical Notes

* Inspected `package.json`, `nuxt.config.ts`, `app/app.vue`, `SiteNav.vue`, `SiteHeader.vue`, `SiteCards.vue`, `SiteLogin.vue`, `SiteFooter.vue`, `GlobalProvider.vue`, `app/style/main.scss`, and `app/style/animate.scss`.
* Frontend spec index appears partially stale for this repo; implementation should prioritize actual Nuxt/Vue conventions.
