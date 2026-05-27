# Design System Stack Decision

**Date:** 2026-05-27
**Status:** Decided
**Spike:** [oscar-ospina/saas-planner#6](https://github.com/oscar-ospina/saas-planner/issues/6)
**Parent epic:** [oscar-ospina/saas-planner#5](https://github.com/oscar-ospina/saas-planner/issues/5)

## Decision

**Tailwind CSS v4 + Radix UI primitives, with shadcn/ui component sources adopted as the starting point, bundled into a versioned `@saas/ui` package.**

## Context

This spike picks the styling + primitives stack for the foundational design system (epic #5). Constraints locked before this ADR:

- **Scope:** web frontend only. Mobile is on the roadmap but deferred; tokens are framework-agnostic and shared cross-platform via the token pipeline (spike #7).
- **Distribution:** `@saas/ui` as a versioned npm-style package consumed by sibling repos. Copy-paste registry distribution (shadcn-style) is explicitly **out**.
- **Single developer** initially; design system has to be maintainable by one person.
- **No web framework chosen yet** — could be Next.js, Remix, Vite + React, Astro, or React Router 7. Stack must not lock that decision in.
- **Figma source of truth:** `UI-Exercise` (key `i4WmV5Gfk9uivVQXC5NY8j`), accessed via the Framelink Figma MCP. **No paid Figma Dev seat** — Code Connect is therefore not available.
- **Product hypothesis still open** — the system favours flexibility over commitment to a product-specific shape.

## Candidates evaluated

1. **Tailwind CSS v4 + Radix UI** — utility-first CSS with `@theme`-based design tokens; accessibility primitives from Radix; shadcn/ui source patterns as a starting point.
2. **Panda CSS + Radix UI** — build-time CSS-in-TS with type-safe recipes and tokens.
3. **vanilla-extract + Radix UI** — zero-runtime CSS-in-TS using `.css.ts` files, themes as TypeScript values.

Radix is common to all three because primitive accessibility is a non-negotiable epic success criterion and re-implementing focus traps, combobox semantics, dialog scroll locks, etc. is not a reasonable solo-dev investment.

## Comparison matrix

| Criterion | Tailwind v4 + Radix | Panda + Radix | vanilla-extract + Radix |
|---|---|---|---|
| **Token-pipeline fit** | Strong — `@theme` block consumes generated CSS vars directly; one-file output target | Strong — typed token config, but requires mirroring tokens into `panda.config.ts` | Moderate — tokens live in TS, recompile needed; theme contract via `createTheme()` |
| **Runtime cost** | Zero | Zero (build-time codegen) | Zero |
| **Type safety** | Moderate — class strings + IntelliSense; no compile-time check that class exists | Strong — typed `css()`, recipes, variants | Strong — fully typed style objects |
| **Distribution as package** | Friction — needs Tailwind v4 in consumer + `@source` directive; documented below | Friction — consumers need Panda runtime; codegen at consumer side | Cleanest — `.css.ts` compiled per-consumer via their bundler plugin |
| **Framework lock-in** | None — works with Next, Remix, Vite, Astro, RR7; NativeWind path to React Native | Low — bundler-integration plugins for all major frameworks | Moderate — needs bundler plugin per framework; rough edges in some RSC patterns |
| **Code Connect compatibility** | N/A — no paid Figma Dev seat (see "Component-Figma parity" below) | N/A | N/A |
| **Learning curve / ecosystem** | Lowest — ~12M weekly DLs, every example/blog/SO answer | Higher — ~130K-500K weekly DLs, pre-v1.0 with occasional breaking changes between minor versions | Moderate — ~940K weekly DLs, mature, narrower community/examples |
| **Maturity** | v4 stable since 2025-01-22 | Pre-v1.0 as of 2026; team close to but not at 1.0 | Mature, used by Twitter/Shopify; stable API |
| **Solo-dev velocity** | Highest — shadcn source pool ready to copy; massive snippets ecosystem | Lower — must hand-write recipes per primitive | Lower — must hand-write `.css.ts` per primitive |

## Rationale

Tailwind v4 + Radix wins on three things that matter most for this context:

1. **Framework-agnostic** — the consumer web framework is not picked yet. Tailwind is the only candidate that is inert across Next/Remix/Vite/Astro/RR7 with no per-framework integration story.
2. **Velocity via shadcn source pool** — the shadcn/ui codebase is essentially a curated implementation of Radix + Tailwind primitives. Adopting it as a starting point converts roughly 4-8 weeks of solo primitive-writing into days of copy + adapt + own. Panda and vanilla-extract have no comparable source pool.
3. **Token pipeline integration** — `@theme` in Tailwind v4 maps directly to a generated CSS file (`@theme { --color-...: ...; }`), and the same tokens are auto-exposed as native CSS custom properties. The token export pipeline (spike #7) can emit one artifact that serves both.

The cost of choosing Tailwind for a packaged distribution is real but resolvable (see "Distribution model" below). The cost of choosing Panda is pre-v1.0 instability and a much smaller ecosystem. The cost of choosing vanilla-extract is solo-dev velocity, since every primitive becomes a `.css.ts` file written from scratch.

## Distribution model

This section answers the three load-bearing questions the spike must not punt on.

### 1. How `@saas/ui` ships

- **TSX source only** (no per-component compiled CSS). Tailwind purging requires the consumer's compiler to see the class strings; shipping compiled CSS defeats the purge and bloats consumers.
- **Generated `@saas/ui/theme.css`** — the single token artifact from spike #7, containing the `@theme` block. Imported by consumer global CSS.
- **Peer dependencies:** `tailwindcss@^4`, `react@^18 || ^19`. Pinned in `package.json`.
- **Output formats:** ESM only (no CJS — all candidate web frameworks are ESM-native in 2026).
- Package consumed via private npm registry or GitHub Packages — picked in the `Bootstrap packages/ui` story.

### 2. How consumers wire it up

In the consumer app's global CSS:

```css
@import "tailwindcss";
@import "@saas/ui/theme.css";
@source "../node_modules/@saas/ui";
```

The `@source` directive tells Tailwind v4 to scan `@saas/ui`'s class strings for purging — necessary because Tailwind's auto-content-detection does not include `node_modules` by default.

Documented in `@saas/ui/README.md` and validated by the "example consumer app" story in epic #5.

### 3. Theme composition and overrides

- `@saas/ui/theme.css` declares the **base layer** of tokens via `@theme { ... }`.
- Consumer apps can **override individual tokens** by redeclaring the same CSS variable in their own `@theme` block — later declarations win in CSS cascade.
- **Per-app brand variations** (e.g. marketing site vs product app) are supported by overriding a defined `--brand-*` token namespace. The pattern is documented in `@saas/ui/README.md`.
- Token names use a stable contract; non-`brand-*` token renames are breaking changes and trigger a major version bump.

### Failure mode: Tailwind version skew

Tailwind minor versions can introduce changes to the `@theme` namespace or utility set. Mitigation:

- Pin `tailwindcss` as a peer dep with a tight range (`^4.x`) and update in lockstep across `@saas/ui` and the example consumer.
- Maintain a one-line compatibility matrix in `@saas/ui/README.md` (e.g. `@saas/ui@1.x requires tailwindcss@^4.0 <4.2`).
- CI in `packages/ui` runs the example consumer build on every PR to catch skew early.

## Implementation pattern: copy shadcn sources, then own them

**Concretely:** the `Bootstrap packages/ui` story will copy the relevant shadcn/ui component source files into `packages/ui/src/` as the v0 starting point, then adapt them to:

- Reference `@saas/ui` tokens (CSS variables from `@theme`) rather than shadcn's default token names.
- Match the primitive surface listed in epic #5 (Button, Input, Select, Card, Modal, Toast, Avatar, Badge).
- Strip shadcn assumptions that don't fit (e.g. its dark-mode token model — dark mode is deferred per epic #5 "out of scope").

**Upstream tracking:** manual changelog review quarterly. Shadcn does not push frequent breaking changes; the cost of being a fork is low.

**This is not "shadcn-inspired" hand-waving** — the ADR commits to *copy the sources*, then own them. Anything else slips into 4-8 weeks of solo work and undercuts the velocity argument that justifies this stack choice.

## Component-Figma parity: visual regression (not Code Connect)

Epic #5's success criterion *"Component ↔ Figma parity validated for every primitive via Code Connect or visual regression baseline"* contains an OR. The Code Connect half is unavailable — it requires a paid Figma Dev seat, and the project intentionally chose the Framelink MCP path to avoid that cost.

**Chosen tool: Playwright snapshots, story-per-primitive in Storybook.**

- In-repo, no external SaaS dependency.
- One snapshot per primitive × variant × state, baseline committed to `packages/ui`.
- CI run on every PR; visual diffs surface as PR check failures.
- Figma-source comparison is **manual at PR review time** — the snapshot tells you the component is stable, and a side-by-side with the Figma frame (via the MCP) tells you the snapshot still matches the design.

**Alternatives considered:**

- **Chromatic** — Storybook-native, free tier exists, but adds an external SaaS dependency and per-seat cost as the team grows.
- **Loki** — less actively maintained as of 2026; Playwright covers the same ground with better tooling.

**Action item for epic #5:** the success criterion wording should be tightened to remove the Code Connect branch, since this ADR commits to visual regression as the parity mechanism. Either edit the issue body, or accept that the OR is now effectively "visual regression only."

## Rejected alternatives

### Panda CSS + Radix

- **Why considered:** type-safe recipes, zero-runtime, framework-agnostic, Chakra DNA.
- **Why rejected:**
  - Pre-v1.0 as of 2026, with documented breaking changes between minor versions — high maintenance cost for a solo developer.
  - ~5-25x smaller ecosystem than Tailwind (snippets, examples, community help, AI-assistant familiarity).
  - No equivalent of the shadcn source pool — every primitive is hand-written, eroding the time advantage.
  - Recipe API is a learning curve on top of the design-system work itself.

### vanilla-extract + Radix

- **Why considered:** mature (940K weekly DLs, established 2021+), zero-runtime, type-safe, the cleanest "ship a package" story of the three candidates.
- **Why rejected:**
  - `.css.ts` per component slows solo-dev velocity; no shadcn-equivalent source pool.
  - Needs a bundler plugin per consumer framework — adds friction now that the web framework is not yet chosen.
  - Some rough edges with React Server Components patterns.
  - Smaller community / fewer examples than Tailwind (and most of those examples are not design-system focused).

If the distribution model later flips to "ship precompiled, framework-agnostic CSS at all costs," vanilla-extract is the right second choice. Tailwind's distribution friction is the gating risk; if it bites in practice (see "Failure mode" above), revisit.

### shadcn registry (copy-paste distribution)

Not a stack choice — a distribution choice. Already ruled out by the locked decision to ship `@saas/ui` as a versioned package. The shadcn *implementation pattern* (Radix + Tailwind, source-owned) is adopted here; the *distribution model* (per-consumer code copying via CLI) is not.

## Risks and open questions

- **Tailwind version skew** between `@saas/ui` and consumers. Mitigation documented above.
- **Tailwind v4 plugin ecosystem maturity** — some third-party plugins still target v3. Acceptable risk; we own the primitives, so v3-only plugins are not load-bearing.
- **shadcn upstream divergence** — over time our copies will drift. Quarterly changelog review is the mitigation; not auto-pulled.
- **Mobile** — if mobile eventually shares `@saas/ui`, NativeWind extends Tailwind v4 to React Native. Doesn't constrain us today.

## Out of scope (deferred)

- Dark mode / theming strategy (separate decision, after primitives stabilise)
- Animation system (Motion / Tailwind utilities) — pick when first animated primitive lands
- Iconography / icon system — pick during the example consumer story
- Data-display primitives (Table, Charts) — explicitly deferred per epic #5

## Next steps

1. **Update epic #5 success criteria** to remove the Code Connect branch (or leave the OR but acknowledge the second half is the actual path).
2. **Open story:** `Bootstrap packages/ui` — references this ADR in its technical notes, scaffolds the repo with Tailwind v4 + Radix + Storybook + Playwright snapshots + the shadcn-source copy step.
3. **Token export pipeline (spike #7)** — output shape is `theme.css` containing the `@theme` block, plus a sibling `tokens.json` for any non-CSS consumer (mobile later).
4. **Close spike #6** with a comment linking this ADR.

## References

- Tailwind CSS v4.0 release — <https://tailwindcss.com/blog/tailwindcss-v4>
- Radix UI primitives — <https://www.radix-ui.com/primitives>
- shadcn/ui — <https://ui.shadcn.com>
- Panda CSS — <https://panda-css.com>
- vanilla-extract — <https://vanilla-extract.style>
- Framelink Figma MCP — `figma-developer-mcp` (configured user-scope)
