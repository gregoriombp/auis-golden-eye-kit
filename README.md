# Auis — Golden Eye, UX Flow & Styleguide kit

A standalone pack, pulled out of [AuisDesign](https://github.com/gregoriombp/AuisDesign), of the agent skills and reference code behind three parts of Auis:

1. **UX Flow · Golden Eye**: compiled, multi-scenario flow pages (several journeys on one ReactFlow board, shared screens deduplicated, a focus lens per scenario, state deep links, Review Bridge comments).
2. **UX Flow hub**: the `/auis/ux-flow` hub, the single-journey flow page, and the skills that create and update flows.
3. **Styleguide pages**: the canonical page-structure prompt and the skills that add a component and build its page under `/auis/styleguide`.

Everything here was copied as-is from `AuisDesign@main` (`b8de2fd`). `skills/` and `docs/` keep their original repo paths, and `reference/` mirrors the repo layout. That means the skills' internal references (`app/auis/ux-flow/...`, `docs/component-map.md`, `AGENTS.md`) still resolve when you drop the pack into an Auis project.

> The pack assumes an Auis project: Next.js App Router, shadcn/ui, the `Au*` components in `components/ui/`, the `@auis/*` packages and the Auis tokens. The `reference/` code is the source the skills tell the agent to read and copy. It is not a runnable app by itself.

---

## What's inside

```
.
├── skills/
│   ├── ux-flows/
│   │   ├── auis-create-ux-flow-golden-eye/SKILL.md   ← Golden Eye (compiled multi-scenario view)
│   │   ├── auis-create-ux-flow/SKILL.md              ← single-journey UX flow page
│   │   └── auis-update-ux-flow/SKILL.md (+ .codex)   ← structural updates + changelog
│   └── design-system/
│       ├── auis-new-component/SKILL.md               ← Au* component + styleguide showcase + nav
│       └── auis-component/SKILL.md                   ← generic (Cowork) version: shadcn → extend → custom
├── docs/
│   ├── styleguide-page-structure.md                  ← THE prompt for building a styleguide page
│   ├── component-layers.md                           ← which sidebar layer a page lands in
│   └── component-map.md                              ← inventory the component skills check first
├── reference/                                        ← code the skills read / copy / import
│   ├── AGENTS.md
│   ├── app/auis/ux-flow/                             ← the whole UX Flow hub
│   │   ├── page.tsx · layout.tsx · navigation.ts
│   │   ├── _data/flow-meta.ts                        ← flow registry (hub sidebar + gallery)
│   │   ├── _components/
│   │   │   ├── golden-eye-overlays.tsx               ← GoldenEyeScreenPreview + GoldenEyeCommentComposer
│   │   │   ├── use-golden-eye-comments.ts            ← Review Bridge comments (origin: "ux-flow")
│   │   │   ├── flow-updates.tsx                      ← changelog badge + history
│   │   │   ├── flow-editor.tsx · flow-comments.tsx · flow-subflow.tsx
│   │   ├── example-golden-eye/page.tsx               ← canonical Golden Eye page (copy this)
│   │   └── example/page.tsx                          ← canonical single-journey flow
│   ├── app/api/flow-suggestions/                     ← API the flow editor posts suggestions to
│   ├── app/auis/styleguide/
│   │   ├── _primitives.tsx                           ← PageHero, Section, Stage, ApiTable, DoDont…
│   │   └── components/_ComponentShowcase.tsx + au-button/page.tsx
│   ├── components/auis/FlowStateDriver.tsx           ← replays ?ge= recipes
│   └── lib/auis-states/                              ← State Mode registry behind ?state=
└── LICENSE (MIT)
```

---

## Golden Eye: when to use it

Use `auis-create-ux-flow-golden-eye` when the value is in **seeing several journeys at once and toggling between them**. Examples: "create a workspace" and "adjust an existing workspace" on one map, or a "has a template?" gate that routes into a sub-journey and converges back. For one linear journey, use `auis-create-ux-flow`.

Prompt it with:

```txt
Compiled view name: Workspace — compiled view
Slug:               workspace-golden-eye
Scenarios:          create a workspace · adjust a workspace · create a template
Per scenario:       [ordered screens/states + decision points]
Shared screens:     [screens more than one scenario touches]
Convergences:       [where scenarios rejoin]
Cross-scenario:     [A's end = B's start / decision → sub-journey]
Prototype links:    [real internal routes, optional]
States per screen:  [?state= / ?ge= variants, optional]
Intro text:         [1–2 sentences]
Group:              Compiled views
```

It produces `app/auis/ux-flow/<slug>/page.tsx` and registers the page in `_data/flow-meta.ts`.

## Styleguide page: the prompt

`docs/styleguide-page-structure.md` is the source of truth for any page in `app/auis/styleguide/`. It defines:

- **Component page**: 15 sections in a fixed order (PageHero → Tldr → Toc → Anatomy → Variants → Sizes → States → Composition → Responsive → API → Tokens consumed → Accessibility → Code → Do/Don't → Related links)
- **Family hub page**: inline inventory first, the "when to use" table last
- **Foundation page**: 10 sections, for tokens and principles

`auis-new-component` / `auis-component` create the component and its showcase. The agent then orders the page to match this doc.

---

## Install into an Auis project

From the root of your Auis project:

```bash
KIT=/path/to/auis-golden-eye-kit

# 1. Skills → auto-discovery folders (Claude Code + Codex/Cursor)
for s in "$KIT"/skills/*/*/; do
  name=$(basename "$s")
  mkdir -p .claude/skills/"$name" .agents/skills/"$name"
  cp "$s"SKILL.md .claude/skills/"$name"/SKILL.md
  if [ -f "$s"SKILL.codex.md ]; then cp "$s"SKILL.codex.md .agents/skills/"$name"/SKILL.md; else cp "$s"SKILL.md .agents/skills/"$name"/SKILL.md; fi
done

# 2. Docs the skills read
cp -n "$KIT"/docs/*.md docs/

# 3. Reference code. Only if the project doesn't have the UX Flow hub yet (-n never overwrites)
cp -Rn "$KIT"/reference/app/auis/ux-flow app/auis/
```

Inside AuisDesign itself, the source of truth for skills is `skills/<capability>/<name>/`, and `npm run skills:sync` regenerates `.claude/skills/` and `.agents/skills/`.

## Dependencies the reference code expects

`@xyflow/react` ^12 · `next` ^16 · `react` ^19 · `@radix-ui/react-collapsible` · `zustand` · `@auis/core` · `@auis/ui` · and the Auis `components/ui/Au*` set (`AuButton`, `AuSheet`, `AuModal`, `AuSegmented`, `AuTextarea`, `AuCard`, `AuPill`, `AuEmpty`, `AuRadioGroup`, `AuLogo`, `Icon`).

## House rules (from the skills)

- Tokens only (`var(--au-*)`, `var(--fg-*)`, `var(--bg-*)`…): no hex and no arbitrary values.
- `Au*` components for overlays. Never hand-roll a modal, sheet or menu.
- Icons through `components/ui/Icon.tsx` (Material Symbols).
- No emoji, and English UI text.
- `npm run typecheck` and `npm run lint` must pass.

---

Synced from `gregoriombp/AuisDesign@main` (`b8de2fd`). MIT © Gregório Pinheiro
