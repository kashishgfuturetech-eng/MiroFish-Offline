# MiroFish Offline — Obsidian Architect UI Redesign

> "Interstellar meets Leica" — A cinematic, high-end futuristic command center interface

## Design System: The Obsidian Architect

This redesign applies the **Obsidian Architect** design system from the Stitch design spec to every UI screen in the MiroFish Offline application.

### Core Principles
- **Deep Sea Foundation** — `#101419` base through `#31353b` tonal nesting
- **Kinetic Orange Primary** — `#FFB59E → #FF5A1F` gradient accents
- **AI Blue Secondary** — `#B6C4FF` for AI processing indicators
- **No-Line Rule** — Sections defined by tonal background shifts, not borders
- **Glassmorphism** — Floating nav/modals use `backdrop-filter: blur(20px)`
- **Typography** — Space Grotesk (headlines) + Manrope (body) + JetBrains Mono (code)

---

## File Structure

```
src/
├── assets/
│   └── design-tokens.css     ← Complete design system CSS variables + utilities
├── components/
│   ├── TopNav.vue             ← Glassmorphic top navigation bar
│   ├── SideNav.vue            ← Tonal sidebar with step states
│   ├── Step1GraphBuild.vue    ← Ontology generation + entity extraction + graph storage
│   ├── Step2EnvSetup.vue      ← Simulation instance + agent personas + config
│   ├── Step3Simulation.vue    ← Live dual-platform simulation runner
│   ├── Step4Report.vue        ← AI report generation with collapsible sections
│   └── Step5Interaction.vue   ← Chat interface with Simulation Oracle
├── views/
│   ├── Home.vue               ← Landing page with hero + file upload console
│   ├── MainView.vue           ← Process wrapper (Step 1 & 2) with sidebar
│   ├── SimulationView.vue     ← Simulation wrapper (Step 3)
│   ├── ReportView.vue         ← Report wrapper (Step 4)
│   └── InteractionView.vue    ← Interaction wrapper (Step 5)
├── router/
│   └── index.js               ← Vue Router with all 5 routes
├── App.vue                    ← Root with page transitions
└── main.js                    ← App entry point
```

---

## All UI Screens (from video analysis)

| # | Screen | Route | Component |
|---|--------|-------|-----------|
| 1 | **Landing / Home** | `/` | `Home.vue` |
| 2 | **Graph Build** | `/process/:id` | `Step1GraphBuild.vue` |
| 3 | **Env Setup** | `/process/:id` | `Step2EnvSetup.vue` |
| 4 | **Simulation Run** | `/simulation/:id` | `Step3Simulation.vue` |
| 5 | **Prediction Report** | `/report/:id` | `Step4Report.vue` |
| 6 | **Interaction / Chat** | `/interaction/:id` | `Step5Interaction.vue` |

---

## Quick Start

```bash
# Install dependencies
npm install

# Start dev server (proxies /api to localhost:8000)
npm run dev

# Build for production
npm run build
```

---

## Design Token Reference

```css
/* Surfaces — use these for tonal nesting, NO borders */
--surface:                  #101419   /* Infinite void base */
--surface-low:              #181c21   /* Sidebar, section dividers */
--surface-container:        #1c2025   /* Cards, panels */
--surface-container-high:   #262a30   /* Hover states, nested cards */
--surface-container-highest: #31353b  /* Active states */

/* Accents */
--primary:            #FFB59E   /* Orange text, icons */
--primary-container:  #FF5A1F   /* Orange fills, gradient end */
--secondary:          #B6C4FF   /* AI Blue, secondary text */
--secondary-container: #0E3FAE  /* AI chip backgrounds */

/* Typography */
--font-headline: 'Space Grotesk'
--font-body:     'Manrope'
--font-mono:     'JetBrains Mono'

/* Key utilities available */
.btn-primary       /* Orange gradient button */
.btn-secondary     /* Ghost button */
.card              /* Surface container card with ghost border */
.card-nested       /* Inner card on surface-low */
.chip-orange       /* Orange badge */
.chip-blue         /* AI Blue badge */
.chip-green        /* Success badge */
.ai-chip           /* Pulsing AI processing indicator */
.glass             /* Glassmorphic surface */
.glass-elevated    /* Elevated glass with shadow */
.input-field       /* Styled input with focus glow */
.step-tag          /* Step indicator pill */
.label-sm          /* Uppercase label style */
.display-lg        /* Hero headline */
.section-divider   /* Gradient line (no solid borders!) */
.orb-orange        /* Decorative background glow */
.orb-blue          /* Decorative AI glow */
.animate-fade-up   /* Entrance animation */
```

---

## Component API

### `<TopNav>`
| Prop | Type | Description |
|------|------|-------------|
| `show-links` | Boolean | Show nav links in center |
| `active-step` | String | Highlight the active nav link |
| `project-id` | String | Used for graph build link |
| `simulation-id` | String | Used for simulation link |

### `<SideNav>`
| Prop | Type | Description |
|------|------|-------------|
| `current-step` | String | `landing\|graph\|env\|simulation\|report` |
| `completed-steps` | Array | Steps to show as completed |
| `@navigate` | Event | Emits step key when nav item clicked |

### `<Step1GraphBuild>`
| Prop | Type | Description |
|------|------|-------------|
| `project-data` | Object | `{ project_id, graph_id, ontology }` |
| `@completed` | Event | Emits graph data when all 3 phases done |

### `<Step2EnvSetup>`
| Prop | Type | Description |
|------|------|-------------|
| `project-data` | Object | From Step1 output |
| `@completed` | Event | Emits `{ simulation_id }` |

### `<Step3Simulation>`
| Prop | Type | Description |
|------|------|-------------|
| `simulation-id` | String | Active simulation ID |
| `@completed` | Event | Fired when both platforms finish |

### `<Step4Report>`
| Prop | Type | Description |
|------|------|-------------|
| `report-id` | String | Report identifier |
| `@completed` | Event | Fired when all sections generated |

### `<Step5Interaction>`
| Prop | Type | Description |
|------|------|-------------|
| `report-id` | String | Report to reference in chat |

---

## Key Design Decisions

1. **Phase-based progressive disclosure** — Each step component uses a `phase` counter; locked phases are dimmed and non-interactive
2. **AI Chip** — Pulsing glass pill (`animation: ai-pulse`) replaces generic spinners for AI operations
3. **Ghost Border** — `rgba(171,137,127,0.15)` used instead of solid `1px` borders everywhere
4. **Active Glow** — Primary orange or AI blue glow on active/selected elements
5. **No lines** — All section separation uses background color steps, not borders
6. **Activity Graph** — Custom CSS bar chart in Simulation view (no external chart library needed)
7. **Tonal Nesting** — Sidebar is `surface-low`, main is `surface`, cards are `surface-container`
