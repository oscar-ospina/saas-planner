# Token Export Pipeline Decision

**Date:** 2026-05-27
**Status:** Decided
**Spike:** [oscar-ospina/saas-planner#7](https://github.com/oscar-ospina/saas-planner/issues/7)
**Parent epic:** [oscar-ospina/saas-planner#5](https://github.com/oscar-ospina/saas-planner/issues/5)
**Related:** [stack decision ADR](2026-05-27-ds-stack-decision.md)

## Decision

**Custom transformer that walks the Figma file tree (via `/v1/files/:key`, used by the Framelink MCP under the hood) and emits a W3C DTCG `tokens.json` plus a Tailwind v4 `theme.css`.**

- **Contract:** DTCG (W3C Design Tokens Community Group) JSON is the canonical interchange format. Lives in `packages/ui/tokens/tokens.json` in production.
- **Source-of-truth feeding `tokens.json`:** local Figma styles (TEXT / FILL / EFFECT), discovered via `/v1/files/:key` and matched against `file.styles` for naming. The transformer is decoupled from the source-of-truth tooling — if the user later adopts the Tokens Studio Figma plugin or upgrades to a plan that exposes native Variables via REST, only the *fetcher* changes.
- **Style Dictionary:** evaluated, not adopted today. Rationale below.
- **PoC artifacts** (real, generated from `UI-Exercise` on 2026-05-27): [`tokens.json`](../plans/2026-05-27-ds-tokens-poc/tokens.json) and [`theme.css`](../plans/2026-05-27-ds-tokens-poc/theme.css).

## Context

This spike picks the pipeline that converts Figma design tokens into code-shipped artifacts (`theme.css` + `tokens.json`) for the design system in epic #5. Constraints carried into this ADR:

- **Stack already decided** ([previous ADR](2026-05-27-ds-stack-decision.md)): Tailwind v4 + Radix, bundled in `@saas/ui`. Consumers wire via `@import "@saas/ui/theme.css"`. Pipeline output must therefore land cleanly inside a Tailwind v4 `@theme { ... }` block.
- **Figma source file:** `UI-Exercise` (key `i4WmV5Gfk9uivVQXC5NY8j`). The product hypothesis is still TBD, so this file is the placeholder design surface today; the pipeline must not assume a specific aesthetic.
- **Mobile deferred** but on the roadmap — the DTCG `tokens.json` doubles as the cross-platform contract.
- **Solo developer**, no funded Figma plan beyond Pro. **No paid Dev seat**, **no Enterprise plan**.

## What Figma actually exposes (and what it doesn't)

This shaped the decision more than any framework preference:

| Surface | Endpoint | Access today | Notes |
|---|---|---|---|
| File node tree (incl. styles dict) | `/v1/files/:key` | ✅ 200 with `file_content:read` | Returns the full document tree + `styles` map (`{styleId: {name, styleType, description}}`) for local styles. **This is what we use.** |
| Published library styles | `/v1/files/:key/styles` | ✅ 200 with `library_content:read` | Returns empty for files with no published styles — `UI-Exercise` has 93 local styles but 0 published. Not the right endpoint for our case. |
| Local file styles (named, values) | (none) | ❌ | Figma does not expose a dedicated "list all local styles with values" endpoint. Values must be walked off nodes that apply the style. |
| Native Figma Variables | `/v1/files/:key/variables/local` | ❌ 403 | Requires `file_variables:read` scope **and** an Enterprise plan. Double-blocked. |

**Implication:** the pipeline must walk the document tree, collect `node.styles.fill / text / effect / stroke` references, and read the actual values off those nodes. The `file.styles` dictionary supplies the human-readable names.

## Comparison: custom transformer vs. Tokens Studio

| Criterion | Custom transformer (over MCP / REST) | Tokens Studio Figma plugin |
|---|---|---|
| **Round-trip fidelity** | High — direct read of the live file; whatever Figma returns is what you get | Indirect — plugin stores tokens in its own JSON store, not natively in Figma |
| **Semantic vs raw token support** | Reads whatever the designer named. Quality of token semantics is bounded by Figma style naming discipline | Designed for semantic tokens; explicit category model (color, spacing, typography, shadow, ...) |
| **Mode / theme handling** | None today — we read one document; modes/themes would need extension | First-class — themes are a core concept in the plugin |
| **CI ergonomics** | Single Node script, runs from `packages/ui` CI with a Figma token in secrets | Plugin pushes to a sync target (GitHub/GitLab); CI reacts to the resulting PR/commit |
| **Ongoing maintenance cost** | We own ~200 lines of TS. Bumps when Figma response shape changes (rare) | Zero TS to maintain; depends on third-party plugin lifecycle + their GitHub sync tier (Pro paywall risk) |
| **Adoption cost on the design side** | None — designers keep working in Figma as they already do | Significant — designers model tokens through the plugin UI, not native Figma |
| **Lock-in** | None — DTCG output is portable; fetcher is replaceable | Couples our token model to Tokens Studio's data model |
| **Native Figma Variables path** | Can swap the fetcher to `/variables/local` when plan/scope allows | Plugin has Variables integration in v2 |
| **Cost** | Free (uses scopes we already have) | Free tier exists; GitHub sync historically Pro-tier — hidden recurring cost risk |

**Custom transformer wins on five things that matter today:** zero new install, zero new scopes beyond what we already have, zero workflow change imposed on the design side, no third-party data-model lock-in, and no hidden subscription cost. The decoupling means we can adopt Tokens Studio later as a feeder if it ever becomes worth the price of admission.

## Worked examples: DTCG → Tailwind v4 namespace mapping

This is the technical risk the spike has to retire. Each DTCG category maps differently into Tailwind v4's `@theme` namespaces. Below is the actual transform performed by the PoC, one example per category.

### Color (simple value)

```jsonc
// DTCG
"color": {
  "m3": { "sys": { "light": {
    "primary": { "$type": "color", "$value": "#65558f" }
  } } }
}
```

```css
/* theme.css */
--color-m3-sys-light-primary: #65558f;
```

Maps DTCG path `color.m3.sys.light.primary` → CSS var `--color-m3-sys-light-primary`. Tailwind v4 picks this up under the `--color-*` namespace and exposes `bg-m3-sys-light-primary`, `text-m3-sys-light-primary`, etc.

### Typography (composite, with references)

```jsonc
// DTCG
"text": {
  "header": {
    "h1-bold": {
      "$type": "typography",
      "$value": {
        "fontFamily": "{font.family.archivo}",
        "fontSize": "56px",
        "fontWeight": "{font.weight.bold}",
        "lineHeight": "60.48px",
        "letterSpacing": "-0.25px"
      }
    }
  }
}
```

```css
/* theme.css */
--text-header-h1-bold: 56px;
--text-header-h1-bold--font-family: var(--font-archivo);
--text-header-h1-bold--font-weight: var(--font-weight-bold);
--text-header-h1-bold--line-height: 60.48px;
--text-header-h1-bold--letter-spacing: -0.25px;
```

Tailwind v4's `--text-*` namespace accepts a *family of related variables* via the `--text-NAME--PROPERTY` suffix convention. `text-header-h1-bold` will then set font-size, font-family, font-weight, line-height, and letter-spacing together. References (`{font.family.archivo}`) resolve via a namespace-aware mapper (the DTCG path `font.family.archivo` becomes `--font-archivo`, not `--font-family-archivo`, matching Tailwind v4's family namespace).

### Font family

```jsonc
// DTCG
"font": { "family": { "archivo": { "$type": "fontFamily", "$value": ["Archivo", "sans-serif"] } } }
```

```css
/* theme.css */
--font-archivo: "Archivo", "sans-serif";
```

Tailwind v4 uses `--font-*` (not `--font-family-*`) for the family namespace. The transformer's resolver hard-codes this exception.

### Font weight (with normalization)

```jsonc
// DTCG (values are normalized, not raw Figma reports)
"font": { "weight": {
  "medium":   { "$type": "fontWeight", "$value": 500 },
  "semibold": { "$type": "fontWeight", "$value": 600 }
} }
```

```css
/* theme.css */
--font-weight-medium: 500;
--font-weight-semibold: 600;
```

**Normalization is load-bearing.** Variable fonts in Figma report weights like 508, 510, 590 — the transformer rounds to the nearest CSS standard (100..900). Without this, the output would have `--font-weight-508`, `--font-weight-510`, `--font-weight-590` as three separate tokens that browsers treat identically. The rounding also dedupes: 508 and 510 both collapse into a single `medium` token.

### Shadow

```jsonc
// DTCG
"shadow": {
  "default": {
    "$type": "shadow",
    "$value": {
      "color": "#00000014",
      "offsetX": "0px", "offsetY": "4px",
      "blur": "8px", "spread": "0px",
      "inset": false
    }
  }
}
```

```css
/* theme.css */
--shadow-default: 0px 4px 8px 0px #00000014;
```

The transformer flattens the shadow object into CSS box-shadow syntax. Multi-shadow composites (DTCG `$value` as array) are emitted as comma-separated.

## Why not Style Dictionary

[Style Dictionary v4](https://amzn.github.io/style-dictionary/) supports DTCG natively and has a [community Tailwind v4 transformer plugin](https://github.com/nado1001/style-dictionary-tailwindcss-transformer). It is battle-tested and a reasonable default for any team distributing tokens to multiple platforms.

**Honest reason for going custom here (not "YAGNI"):**

The Tailwind v4 namespace mapping above has at least three rules that aren't pure path-to-name transforms — `font.family.*` collapses to `--font-*` (not `--font-family-*`), typography composites explode into a `--text-NAME--PROPERTY` family, and font weights need value-level normalization. Implementing any of these inside Style Dictionary means writing a custom formatter and a custom transform, which is ~80 lines. The full custom transformer (the one in this ADR's PoC) is ~200 lines. We trade ~120 lines of bespoke logic for zero dependencies, zero version pinning, and zero "magic" — every output rule is grep-able in our own source.

For the current scope (Tailwind v4 + future JSON consumer for mobile), this is a fair trade. **The decision point that flips this:** if the output target list grows beyond two (e.g. Android XML + iOS Swift + Tailwind + raw CSS), Style Dictionary's transform chain pays for itself. Revisit then.

## PoC: end-to-end run against UI-Exercise

The transformer was executed against the real `UI-Exercise` file on 2026-05-27. Results:

```json
{
  "source": "UI Exercise",
  "styles_defined": 93,
  "styles_applied_in_tree": 88,
  "emitted": {
    "colors": 20,
    "typography": 58,
    "shadows": 1,
    "font_families": 5,
    "font_weights": 4
  }
}
```

Five locally-defined styles are unused in the document tree — the transformer skips them (you can't recover a style's value without finding a node that applies it).

Artifacts committed for review:

- [`docs/superpowers/plans/2026-05-27-ds-tokens-poc/tokens.json`](../plans/2026-05-27-ds-tokens-poc/tokens.json) — DTCG output (24 KB)
- [`docs/superpowers/plans/2026-05-27-ds-tokens-poc/theme.css`](../plans/2026-05-27-ds-tokens-poc/theme.css) — Tailwind v4 `@theme` block (15 KB)

### Known limitations of this PoC (not the decision)

These are real and should be fixed by the production pipeline in `packages/ui`, not the spike:

- **No spacing scale extracted.** UI-Exercise uses raw inline pixel values for `itemSpacing`, `paddingLeft/Top/...`, not named tokens. Production transformer should statistically infer a 4-base scale from the layout properties, or wait for the designer to introduce named spacing variables.
- **No border radius extracted.** Same reason — inline values, not named.
- **One style literally named after its hex (`0f172a`)** survives into the output as `--color-0f172a`. The production pipeline should detect and warn on styles whose names match their values (low-information naming).
- **No mode/theme support** (light vs. dark, brand variants). DTCG supports `$extensions` for this; the production transformer adds it when dark mode lands (currently out of scope per epic #5).
- **No CI integration yet.** Production pipeline runs `npm run build:tokens` in `packages/ui` CI; output diff visible in PR.

## Transformer source (~200 lines)

This is documentation of the approach. The actual file ships with the `Bootstrap packages/ui` story — *application code does not live in `saas-planner` per the repo's CLAUDE.md*. Recreate from this snippet:

```js
// packages/ui/scripts/build-tokens.mjs
// Figma file tree -> DTCG tokens.json -> Tailwind v4 theme.css

import fs from "node:fs";
import path from "node:path";

const [, , inPath, outDir] = process.argv;
if (!inPath || !outDir) {
  console.error("usage: node build-tokens.mjs <figma-dump.json> <output-dir>");
  process.exit(1);
}

const file = JSON.parse(fs.readFileSync(inPath, "utf8"));
const styleDefs = file.styles || {};
const observed = new Map();

function walk(node) {
  if (!node) return;
  const ref = node.styles || {};
  if (ref.fill && node.fills?.length) {
    const f = node.fills.find((p) => p.type === "SOLID") || node.fills[0];
    if (!observed.has(ref.fill)) observed.set(ref.fill, { kind: "fill", paint: f });
  }
  if (ref.text && node.style) {
    if (!observed.has(ref.text)) observed.set(ref.text, { kind: "text", style: node.style });
  }
  if (ref.effect && node.effects?.length) {
    if (!observed.has(ref.effect)) observed.set(ref.effect, { kind: "effect", effects: node.effects });
  }
  if (Array.isArray(node.children)) for (const child of node.children) walk(child);
}
walk(file.document);

const rgbToHex = ({ r, g, b, a = 1 }) => {
  const ch = (v) => Math.round(v * 255).toString(16).padStart(2, "0");
  const hex = `#${ch(r)}${ch(g)}${ch(b)}`;
  return a < 1 ? `${hex}${ch(a)}` : hex;
};

const normalizeWeight = (w) => Math.round(w / 100) * 100;
const weightName = (w) => ({100:"thin",200:"extralight",300:"light",400:"regular",500:"medium",600:"semibold",700:"bold",800:"extrabold",900:"black"})[normalizeWeight(w)] || String(normalizeWeight(w));

const slug = (s) => s.toLowerCase().trim().replace(/[^a-z0-9/]+/g, "-").replace(/^-+|-+$/g, "");
const splitName = (name) => name.split("/").map(slug).filter(Boolean);
const setDeep = (obj, parts, value) => {
  let cur = obj;
  for (let i = 0; i < parts.length - 1; i++) (cur[parts[i]] ||= {}), (cur = cur[parts[i]]);
  cur[parts.at(-1)] = value;
};

// Build DTCG tree
const dtcg = { color: {}, font: { family: {}, weight: {} }, text: {}, shadow: {} };
const fams = new Set(), wts = new Set();
const round2 = (n) => Math.round(n * 100) / 100;

for (const [id, def] of Object.entries(styleDefs)) {
  const seen = observed.get(id);
  if (!seen) continue;
  const parts = splitName(def.name);
  if (def.styleType === "FILL" && seen.paint?.type === "SOLID") {
    setDeep(dtcg.color, parts, { $type: "color", $value: rgbToHex(seen.paint.color) });
  } else if (def.styleType === "TEXT") {
    const st = seen.style;
    fams.add(st.fontFamily);
    wts.add(st.fontWeight);
    setDeep(dtcg.text, parts, { $type: "typography", $value: {
      fontFamily: `{font.family.${slug(st.fontFamily)}}`,
      fontSize: `${st.fontSize}px`,
      fontWeight: `{font.weight.${weightName(st.fontWeight)}}`,
      lineHeight: st.lineHeightUnit === "PERCENT" ? `${round2(st.lineHeightPercent/100)}` : st.lineHeightPx ? `${round2(st.lineHeightPx)}px` : "1.2",
      letterSpacing: st.letterSpacing ? `${round2(st.letterSpacing)}px` : "normal",
    }});
  } else if (def.styleType === "EFFECT") {
    const shadows = seen.effects.filter((e) => e.type === "DROP_SHADOW" || e.type === "INNER_SHADOW").map((e) => ({
      color: rgbToHex(e.color),
      offsetX: `${e.offset.x}px`, offsetY: `${e.offset.y}px`,
      blur: `${e.radius}px`, spread: e.spread ? `${e.spread}px` : "0px",
      inset: e.type === "INNER_SHADOW",
    }));
    if (shadows.length) setDeep(dtcg.shadow, parts, { $type: "shadow", $value: shadows.length === 1 ? shadows[0] : shadows });
  }
}
for (const f of fams) dtcg.font.family[slug(f)] = { $type: "fontFamily", $value: [f, "sans-serif"] };
for (const w of new Set([...wts].map(normalizeWeight))) dtcg.font.weight[weightName(w)] = { $type: "fontWeight", $value: w };

// Emit DTCG
fs.mkdirSync(outDir, { recursive: true });
fs.writeFileSync(path.join(outDir, "tokens.json"), JSON.stringify(dtcg, null, 2) + "\n");

// Emit Tailwind v4 theme.css
const dtcgPathToCssVar = (parts) => {
  if (parts[0] === "font" && parts[1] === "family") return "--font-" + parts.slice(2).join("-");
  if (parts[0] === "font" && parts[1] === "weight") return "--font-weight-" + parts.slice(2).join("-");
  return "--" + parts.join("-");
};
const resolveRef = (str) => {
  const m = String(str).match(/^\{([\w.-]+)\}$/);
  return m ? `var(${dtcgPathToCssVar(m[1].split("."))})` : str;
};

const css = ["@theme {"];
const emitColor = (obj, prefix) => {
  for (const [k, v] of Object.entries(obj)) {
    if (v?.$type === "color") css.push(`  --color-${[...prefix, k].join("-")}: ${v.$value};`);
    else if (v && typeof v === "object") emitColor(v, [...prefix, k]);
  }
};
emitColor(dtcg.color, []);
css.push("");
for (const [k, v] of Object.entries(dtcg.font.family)) css.push(`  --font-${k}: ${v.$value.map((s) => `"${s}"`).join(", ")};`);
css.push("");
for (const [k, v] of Object.entries(dtcg.font.weight)) css.push(`  --font-weight-${k}: ${v.$value};`);
css.push("");
const emitText = (obj, prefix) => {
  for (const [k, v] of Object.entries(obj)) {
    if (v?.$type === "typography") {
      const n = [...prefix, k].join("-"), val = v.$value;
      css.push(`  --text-${n}: ${val.fontSize};`);
      css.push(`  --text-${n}--font-family: ${resolveRef(val.fontFamily)};`);
      css.push(`  --text-${n}--font-weight: ${resolveRef(val.fontWeight)};`);
      css.push(`  --text-${n}--line-height: ${val.lineHeight};`);
      if (val.letterSpacing && val.letterSpacing !== "normal") css.push(`  --text-${n}--letter-spacing: ${val.letterSpacing};`);
      css.push("");
    } else if (v && typeof v === "object") emitText(v, [...prefix, k]);
  }
};
emitText(dtcg.text, []);
const emitShadow = (obj, prefix) => {
  for (const [k, v] of Object.entries(obj)) {
    if (v?.$type === "shadow") {
      const sh = Array.isArray(v.$value) ? v.$value : [v.$value];
      css.push(`  --shadow-${[...prefix, k].join("-")}: ${sh.map((s) => `${s.inset ? "inset " : ""}${s.offsetX} ${s.offsetY} ${s.blur} ${s.spread} ${s.color}`).join(", ")};`);
    } else if (v && typeof v === "object") emitShadow(v, [...prefix, k]);
  }
};
emitShadow(dtcg.shadow, []);
css.push("}");
fs.writeFileSync(path.join(outDir, "theme.css"), css.join("\n") + "\n");
```

**To run:**

```bash
# Fetch the Figma file (uses the FIGMA_API_KEY env var)
curl -H "X-Figma-Token: $FIGMA_API_KEY" \
  "https://api.figma.com/v1/files/$FIGMA_FILE_KEY?geometry=paths" \
  -o ui-exercise.json

# Transform
node build-tokens.mjs ui-exercise.json ./output/
```

## Risks and open questions

- **Style naming discipline.** The output quality is bounded by how well the designer names Figma styles. `UI-Exercise` has good text-style naming (`Header/H1 Bold`, `Body/B1 Regular`) but inconsistent color naming (`Light/Background/Blue` all map to white; one style is named after its hex). Production needs a style-naming contract documented in `packages/ui/CONTRIBUTING.md`.
- **Tailwind v4 namespace conventions can drift.** The hard-coded exceptions (`font.family.X → --font-X`) live in the transformer. A Tailwind v4 minor that adds new namespaces or renames existing ones requires updating the resolver. CI test (consumer app build) catches this.
- **Single source file.** Today the transformer reads one Figma file. Multi-file consolidation (e.g. brand tokens in one file, semantic tokens in another) is not addressed; defer until needed.
- **No write-back to Figma.** This is a one-way Figma → code pipeline. If we want round-trip (code changes propagate to Figma), revisit; would require either Variables Write API (Enterprise) or Tokens Studio.

## Out of scope (deferred)

- Spacing scale extraction from inline layout values (statistical inference)
- Border radius extraction (same problem)
- Dark mode / theming `$extensions`
- Composite breakdown into Tailwind utility classes via `@utility` directives — current approach uses `--text-NAME--PROPERTY` family which is sufficient

## Next steps

1. **Close spike #7** with a comment linking this ADR + the PoC artifacts.
2. **The `Bootstrap packages/ui` story** consumes this ADR — scaffolds `packages/ui/scripts/build-tokens.mjs` (the transformer), `packages/ui/tokens/tokens.json` (snapshot), and `packages/ui/src/theme.css` (generated). CI re-runs the script on every PR touching `tokens.json`.
3. **Token-feeding workflow** for the designer: name Figma styles using a `Category/Variant` slash-separated convention (already established in `UI-Exercise`). Re-run the script after any style addition; commit the diff.
4. **Document the style-naming contract** (Category/Variant slug convention, no hex-as-name) in `packages/ui/CONTRIBUTING.md` when that repo lands.

## References

- W3C Design Tokens Community Group format — <https://tr.designtokens.org/format/>
- Style Dictionary v4 — <https://amzn.github.io/style-dictionary/>
- Tailwind v4 theme variables — <https://tailwindcss.com/docs/theme>
- Figma REST API — <https://www.figma.com/developers/api>
- Tokens Studio for Figma — <https://docs.tokens.studio/>
