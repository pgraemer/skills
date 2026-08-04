---
name: frontend-review
description: Review Vue component CSS/SCSS for self-containment and the helper-class hierarchy (iai- → kds- → tailwind → own class). Use when reviewing or writing frontend styles, .vue components, or SCSS — e.g. "review this component's styles", "is this CSS okay", or before committing frontend changes that touch templates or style blocks.
---

# Frontend style review

Review styling in `frontend/` (`auditing/`, `labeling/`, `managing/`, `shared/`) against two rules: **components are style-self-contained** and **helpers are used in a fixed lookup order**. Flag violations with file:line and a concrete fix.

## Rule 1 — Components are self-contained

A component owns its internal layout and ships every style it needs. A parent must be able to drop the component anywhere without knowing its CSS internals.

**Ownership split (the boundary is the component's root element):**

- **Child owns everything inside its root:** its own `display: flex`/`grid` for laying out its children, internal spacing, appearance.
- **Parent owns the child's external geometry:** where it sits, gaps between siblings (prefer `gap` on the parent container over margins), and the size of the slot it occupies — `flex: 1`, `width`, `grid-area` *on the child's host element* is parent business.

**Violations to flag:**

- A component whose **inner** layout only works because the parent sets `display: flex`/`grid` from outside — "parent sets flex so the child's internals work" is broken; "parent sets `flex: 1` so the child fills the row" is correct.
- A component setting outer `margin` on its own root — that decides its placement in every parent context; placement is the parent's job.
- A parent styling a child's internals: selectors targeting the child's root class, or passing classes that reach into the child's DOM.
- `:deep()` into **our own components: hard ban, no exceptions.** If a parent needs to vary a child's appearance, the child exposes a prop, a slot, or a CSS custom property — that's the child's public style API, and custom properties are the sanctioned way to pass style decisions across the boundary.
- `:deep()` into **KDS/third-party internals: last resort only** — first check the component's props/design-token API. Every such `:deep()` must carry a one-line comment naming what KDS lacks that forced it (the audit trail for KDS upgrades). Flag any uncommented `:deep()`.
- Styles for a component defined in a *different* component's `<style>` block, or in a shared stylesheet when only one component uses them.
- A component that renders broken/unstyled unless a sibling stylesheet is imported somewhere up the tree.

**Correct shape:** `<style scoped lang="scss">` is the only sanctioned block form in components.

- Plain `<style scoped>` without SCSS → nit: migrate when touched, not a hard failure, no blanket rewrites.
- Unscoped `<style>` in a component → violation, unless it overrides KDS/third-party globals — then it needs a one-line comment stating what forced it (same rule as `:deep()`).
- Global class names must be prefixed (`iai-`, and that set is frozen — see Rule 2). The "short names" rule below applies *inside scoped blocks only*; nothing short and unprefixed may ever go global.

## Rule 2 — Helper hierarchy (strict order)

When a template needs a style, resolve it in this order. Only fall through when the current level genuinely has no fit — "I didn't check" is not a fall-through.

1. **`iai-` helpers** — our own utilities, defined in `frontend/shared/src/styles/shared.scss` (e.g. `iai-no-wrap`, `iai-text-ellipsis`, `iai-cursor-pointer`, `iai-border`, `iai-no-flex-shrink`). Check this file first; it's short. **This set is frozen for reads-first, not a growth layer:** never add a new `iai-` helper as part of ordinary work — additions require an explicit team decision. `kds-` is the default home for token-based styling; flag any PR that introduces a new `iai-` class.
2. **`kds-` utility classes** — the design-system utilities (spacing `kds-p-4`/`kds-mb-2`/`kds-px-3`, typography `kds-font-bold`/`kds-text-caption`, grid `kds-row`/`kds-col-6`/`kds-no-gutter`, alignment `kds-start`/`kds-end`, display `kds-display-flex`). Prefer these for spacing, typography, and grid — they carry the design tokens.
3. **Tailwind** — available (v4 via `@tailwindcss/vite`), but **structural/behavioral utilities only**: things that carry no design-token value — `flex`, `grid`, `items-center`, `justify-between`, `relative`/`absolute`, `overflow-hidden`, `truncate`, `shrink-0`, `w-full`. **Hard ban on Tailwind classes that encode a themable value**: spacing (`p-*`, `m-*`, `gap-*`), colors (`text-*-500`, `bg-*`), font sizes/weights, radii, shadows — those come from Tailwind's scale, not KDS tokens, and drift. Mechanical check: *Tailwind class encodes a design value → violation.* If kds- lacks a token-carrying utility you need, fall through to level 4 with a KDS token/variable — never to Tailwind's scale.
4. **Own class + SCSS** — last resort, when the styling can't be expressed with the above (component-specific layout, states, transitions, computed sizes).

**Review checks:**

- Any custom class or inline style that replicates an existing helper → flag it, name the helper that should replace it. Typical offenders: hand-rolled `white-space: nowrap`, `text-overflow: ellipsis`, `cursor: pointer`, margins/padding that map to `kds-*` spacing steps, `font-weight` that maps to `kds-font-*`.
- Tailwind used where a `kds-` token utility exists (e.g. `p-4` instead of `kds-p-4`) → flag; kds carries the design system, tailwind values may drift from it.
- Inline `style=""` attributes → almost always a violation; only dynamic values computed at runtime (e.g. `width` from data) are legitimate.
- Mixing levels for the same property on the same element (e.g. `kds-mb-2` plus a custom margin-bottom) → flag as conflict.

## Rule 3 — When writing own classes

- **Short names, no BEM.** Flag anything shaped like `block__element--modifier`. Scoped styles already provide isolation, so names only need to be meaningful inside the component: `.header`, `.thumb`, `.empty-state` — not `.image-import-edit__header-container--collapsed`.
- **Use SCSS nesting instead of name-spacing.** A modifier is a nested `&.active` or a state class, not a `--modifier` suffix. Parent–child relationships are expressed by nesting, not by compound class names.
- Keep selectors flat-ish: nesting deeper than ~3 levels usually means the DOM chunk should be its own component.
- No magic numbers duplicating a token that exists as an `iai-` variable/helper or KDS token — reference the token.
- **SCSS features — sanctioned:** nesting (≤3 deep), `&` for states/modifiers (`&.active`, `&:hover`), local variables for repeated values within a component, shared variables/mixins imported from `shared/` for genuinely cross-component patterns.
- **SCSS features — banned:** `@extend`, always — it rewrites and hoists selectors invisibly, bloats output, and misbehaves across scoped boundaries. A shared visual pattern is a mixin, or more often a sign it should be a shared component or existing helper. Loops/maps/functions belong only in the shared stylesheet layer (where helpers are generated), never inside a component block.

## Scope & severity

**Diff-scoped:** review only changed lines/components. Pre-existing violations in a *touched* file get one collapsed mention ("pre-existing, not from this change — optional cleanup"), never mixed into the main findings. Untouched files are off-limits. Never ask a PR to fix violations it didn't introduce.

**Two tiers:**

- **Must-fix (blocking):** self-containment breaks — parent styling child internals, `:deep()` into own components, inner layout depending on parent CSS. These export coupling across components and get harder to remove every day.
- **Should-fix (advisory):** hierarchy violations (custom CSS duplicating an existing helper, Tailwind token-values, uncommented `:deep()` into KDS), BEM names, self-margins on roots, plain `<style scoped>` without SCSS.

## Output format

For each finding: `file:line`, the tier, the rule broken, and the fix (which helper to use, or how to restructure). Must-fix first, then should-fix, then the collapsed pre-existing note. If the styles are clean, say so — don't invent findings.
