# Medica — Design System (Foundations)

A standalone companion and the bridge from the specs to the visual design. It defines the **foundations** a designer (in Figma or Claude Design) and a `shadcn/ui` codebase share: design tokens, the component inventory mapped to shadcn, the consolidated badge and error-pattern catalogs, RTL/i18n rules, responsive targets, an accessibility floor, and how to organize the Figma file.

What this doc fixes is the **structure and the contract**. The final visual choices — exact palette, type scale, motion language — are the designer's to lock; §2 gives a deliberate starting direction and the open list is in §10.

---

## 1. The system has two halves sharing one set of tokens

This is the most important thing to understand before starting:

- **Figma side** — visual tokens (as Figma *variables*/*styles*), component specs, screens, and a clickable prototype.
- **Code side** — `shadcn/ui`: React components you copy into your repo, built on **Radix UI** + **Tailwind CSS**, themed with **CSS variables**. You own the code; there's no runtime dependency.
- **The tokens are the contract between them.** Define each token once (color, type, spacing, radius), express it as a CSS variable for Tailwind/shadcn and as a Figma variable for design, and keep the two in step.

Practical notes for a newbie:
- `shadcn/ui` is **not** a Figma UI kit. To mirror it visually in Figma, use a community "shadcn/ui" Figma kit as a starting library, then retheme it with the tokens below.
- Theming shadcn = editing CSS variables in `globals.css` + the Tailwind theme; components read those variables, so changing a token restyles everything.

---

## 2. Design tokens (proposed starting direction — finalize in Figma)

**Direction (a proposal, not a default).** Medica is in-home GLP-1 care in Iran: the feeling to earn is *calm clinical trust* — a nurse you'd let into your home — not a flashy fitness app and not a cold hospital portal. A Persian-first identity: a warm, legible neutral base, a single confident medical-teal primary for trust, a restrained warm accent for human warmth, and the five status tones doing real work (they are not decoration — they carry state). This is a starting point; refine it in Figma rather than treating the hex values as final law.

### Color (semantic tokens — name by role, not by hue)

| token | role | starting value (refine) |
|---|---|---|
| `--background` / `--foreground` | app bg / text | `#FBFAF8` / `#1B2B2A` |
| `--primary` / `--primary-foreground` | primary actions, trust | `#0E7C7B` (teal) / `#FFFFFF` |
| `--secondary` | secondary surfaces | `#EAF3F2` |
| `--accent` | warm human accent (sparingly) | `#E8A06A` |
| `--muted` / `--muted-foreground` | quiet text/fills | `#F1EFEA` / `#5C6B6A` |
| `--border` / `--input` / `--ring` | lines, fields, focus | `#E2E0DA` / … / `#0E7C7B` |
| `--success` | success tone | `#1F9D6B` |
| `--info` | info tone | `#2D7FB8` |
| `--warning` | warning tone | `#C9881F` |
| `--danger` / `--destructive` | danger/destructive | `#C0492F` |
| `--neutral` | neutral tone | `#6B7775` |

The last five **must** be the badge tones in §4 — the design system has exactly one source for `neutral · info · success · warning · danger`. Decide light/dark theme support in §10.

### Typography

- **Persian (primary, RTL):** a high-quality Persian UI face — e.g. **Vazirmatn** or **Estedad** — for both display and body; Persian digits.
- **Latin (`en`):** a complementary pairing — a characterful but restrained display face + a clean body face; Latin digits.
- Set one **type scale** (e.g. 12/14/16/18/20/24/30/36) with intentional weights; body 16px min on mobile. Numerals follow locale (§5).
- Pin the type treatment as part of the identity — not a neutral default.

### Other tokens

- **Spacing** — a 4px base scale (4/8/12/16/24/32/48/64).
- **Radius** — one family (e.g. `--radius: 10px`); shadcn derives sm/md/lg from it.
- **Elevation** — 2–3 shadows, used sparingly.
- **Motion** — short durations (≈150–250ms), one easing; **respect `prefers-reduced-motion`** (accessibility floor).
- **Z-index** — a named scale (base, dropdown, sticky, overlay, modal, toast).

---

## 3. `shadcn/ui` component mapping

Map the UI patterns used across the `CMP-` inventory to shadcn components, so design and code share a vocabulary:

| pattern in Medica | shadcn/ui component | notes |
|---|---|---|
| Buttons / CTAs | `Button` | variants: primary, secondary, ghost, destructive |
| Forms & fields | `Form`, `Input`, `Textarea`, `Label`, `Select`, `Checkbox`, `RadioGroup`, `Switch` | most patient/onboarding screens |
| Status badges | `Badge` | the **5 tones** of §4 as variants |
| Inline / banner errors | `Alert` (+ `Alert` variant) | unhappy-paths `inline` / `banner` |
| Transient errors / confirmations | `Sonner` (toast) | unhappy-paths `toast` |
| Blocking / confirm dialogs | `Dialog`, `AlertDialog` | `blocking modal`; cold-chain block, destructive confirms |
| Drawers / side panels | `Sheet` | notification center (`SCR-SHL-02`), filters |
| Lists & admin grids | `Table`, `DataTable` (TanStack) | admin/doctor/pharmacy tables |
| Tabs / segmented views | `Tabs` | dashboards, CS panels |
| Loading | `Skeleton` | every async screen |
| Empty states | `Card` + illustration | unhappy-paths empty states |
| Dates | `Calendar` / `Popover` | **Gregorian only** — see §5; Jalali needs a Persian-calendar picker |
| Misc | `Tooltip`, `Avatar`, `Progress`, `Accordion`, `Command`, `DropdownMenu` | as needed |

Two cautions: shadcn's `Calendar` (react-day-picker) is **Gregorian** — for `fa-IR` you'll need a Persian-calendar date picker (e.g. a `react-multi-date-picker`-style component) themed to match. And RTL: Radix primitives accept `dir="rtl"`; pair with Tailwind logical utilities (§5).

---

## 4. The two catalogs design must build (finite sets — don't invent per screen)

### 4.A Badge inventory (from `Medica_Lifecycle_State_Machines.md`)

Every **user-visible** status badge, with its tone. This is the complete set the `Badge` component renders:

| domain | badges (tone) |
|---|---|
| subscription | Trial (info) · Active (success) · Payment due (warning) · Cancelled (neutral) · Expired (neutral) |
| visit | Scheduled (info) · Completed (success) · Cancelled (neutral) |
| prescription | Issued (success) · Sent (info) · Rejected (danger) |
| nurse_visit | Scheduled / Nurse on the way (info) · Completed (success) · Could not complete (danger) · Rescheduled (neutral) |
| provider | Active (success) · Inactive (warning) |
| ticket | Open (info) · In progress (info) · Resolved (success) · Closed (neutral) |

(Internal-only states show a simplified label or none — see the lifecycle spec. Badges must not rely on color alone — pair tone with a label/icon for accessibility, §7.)

### 4.B Error / empty-state pattern inventory (from `Medica_Unhappy_Paths.md`)

The finite set of feedback patterns, each a design-system component/state:

| pattern | shadcn | when |
|---|---|---|
| `toast` | Sonner | transient, auto-retry/minor |
| `inline` | Alert / field error | form/section validation |
| `banner` | Alert (persistent) | whole-screen state (past_due, offline) |
| `full-screen state` | Card/empty layout | screen's purpose failed/unavailable |
| `blocking modal` | AlertDialog | safety/money-critical acknowledgement |
| `badge` | Badge | status change in a list/detail |
| empty / loading | Card + Skeleton | first-run and async states |

---

## 5. RTL & internationalization in design

Per `Medica_Internationalization.md`:

- **Design `fa-IR` (RTL) first**, then verify `en` (LTR). Mirror layout by document direction; use **logical** properties (start/end), never hard left/right.
- **Numerals** follow locale: Persian digits for `fa`, Latin for `en`.
- **Dates:** Jalali calendar for `fa`, Gregorian for `en` — drives the date-picker choice (§3).
- **Icons:** mirror directional icons (arrows, chevrons, progress) in RTL; leave non-directional ones.
- **No text baked into images** — all copy is a localized message key; leave room for text expansion.

---

## 6. Responsive & device targets

| surface | target |
|---|---|
| Patient (`mfe-patient`), Nurse (`mfe-nurse`) | **mobile-first** (nurse uses it in the field — large touch targets, one-hand reach, offline-tolerant where specced) |
| Doctor, Pharmacy, Admin/ops, CS, Affiliate | **desktop-first** panels (dense tables, multi-column) |
| Marketing (`mfe-marketing`) | **responsive** + SEO (SSG/SSR) |

Define a breakpoint scale (e.g. sm 640 / md 768 / lg 1024 / xl 1280) and minimum touch target (≥44px) for the mobile surfaces.

---

## 7. Accessibility floor

Build to this without announcing it: WCAG **AA** contrast on text and on the status tones; **visible keyboard focus** (the `--ring` token); full keyboard navigation; **`prefers-reduced-motion`** respected; touch targets ≥44px on mobile; every form field labeled; correct `dir` for RTL; and — because tone/color alone is insufficient — **status badges carry a text label and/or icon, not just color**, with an accessible name for screen readers.

---

## 8. Figma organization (recommended for this project)

One Figma **file** for the product, structured as **pages**:

| page | contents |
|---|---|
| `00 · Cover` | project name, version, links to the specs |
| `01 · Foundations` | tokens as Figma **variables/styles**: color, type, spacing, radius, grid |
| `02 · Components` | the shadcn-mirrored **component library** — each reusable element as a Figma **component** with **variants** (Button variants, Badge ×5 tones, Input states, Alert patterns, …) |
| `03 · Marketing` … `0N · Affiliate` | one page **per surface**; each screen is a **frame** named by its `SCR-` id |
| `Flows · Prototype` | screens wired into clickable **prototypes** for the key journeys |

Vocabulary (so the structure makes sense): a **file** is the document; **pages** are its tabs; a **frame** is one screen; a **component** is a reusable element you instance everywhere; **variants** are versions of one component; **sections** group related frames within a page.

Conventions:
- **Name every screen frame by its `SCR-` id** (e.g. `SCR-PAT-14 · Checkout`) so Figma ↔ specs map 1:1.
- Build the `02 · Components` library from the tokens; if it grows, split it into its own file and **publish it as a team library**.
- Use **sections** on each surface page to group a flow (e.g. "Onboarding", "Home visit").
- Keep one set of variables in `01 · Foundations`; everything else references them — change a token once, the file updates.

---

## 9. User stories & prototype — what you actually need

- **User stories:** *optional.* Your `F-` feature list + the Screen Inventory **journey map** (§3) already function as the backlog and the flows. Add stories only if you want explicit "as a … I want … so that …" acceptance criteria; I can generate a concise set on request.
- **Prototype:** a **Figma deliverable**, not a markdown file — built from the journey map. Prototype, at minimum: the **discover → signup → eligibility → subscribe → visit → home injection** happy path, plus the high-stakes unhappy paths (**payment failure** UP-PAT-07, **ineligible** UP-PAT-05, **cold-chain block** UP-NUR-01). Those validate the riskiest moments before code.

---

## 10. Open design decisions

Deliberately left to the designer:

1. **Final palette & type scale** — §2 is a starting direction; lock the values and confirm AA contrast.
2. **Light/dark mode** — support dark, or light-only for the pilot?
3. **Information architecture** — the surface groupings proposed in `Medica_Screen_Inventory.md` §8 (patient bottom-nav, onboarding wizard, nurse visit step-flow, admin sections).
4. **Motion language** — where motion serves the product vs. where it's noise.
5. **Jalali date-picker component** — which Persian-calendar picker, themed to the tokens.
6. **Iconography & illustration** — a coherent set for empty states and the marketing surface.

---

## 11. Fold-back

No entities, APIs, components, or screens change here. This doc adds the design-token layer over the existing inventory and consolidates the badge catalog (`Lifecycle`) and error-pattern catalog (`Unhappy Paths`) for the design system; both remain owned by their source specs. `[clinical]` copy (ineligible, side-effect, recall, public eligibility) comes from the clinical-safety spec — use placeholders in design until then.
