---
name: ubuild
description: "Interactive Flutter app scaffolding framework. Interviews the developer, generates a structured scaffold plan, and builds out the app phase by phase. Trigger: 'ubuild', 'scaffold', 'new flutter app', 'app scaffolding'"
---

```
         ██╗   ██╗██████╗ ██╗   ██╗██╗██╗     ██████╗ 
         ██║   ██║██╔══██╗██║   ██║██║██║     ██╔══██╗
         ██║   ██║██████╔╝██║   ██║██║██║     ██║  ██║
         ██║   ██║██╔══██╗██║   ██║██║██║     ██║  ██║
         ╚██████╔╝██████╔╝╚██████╔╝██║███████╗██████╔╝
          ╚═════╝ ╚═════╝  ╚═════╝ ╚═╝╚══════╝╚═════╝ 
              F L U T T E R   S C A F F O L D E R
```

# uBuild — Interactive Flutter App Scaffolder

uBuild is a phased, interview-driven scaffolding framework for Flutter apps. It generates a `scaffold/` directory containing structured planning documents, then uses those documents to build out the actual Flutter project.

**You MUST follow the phases in order. Each phase produces artifacts in `scaffold/`. Transitioning to the next phase requires a recursive audit — re-read every artifact produced so far, verify consistency, flag contradictions, and get explicit developer approval before proceeding.**

---

## How It Works

### The scaffold/ Directory

All planning artifacts live in `scaffold/` at the project root:

```
scaffold/
├── master-plan.md              # Phase tracker, status, decisions log
├── 1-interview/
│   └── requirements.md         # Captured requirements from interview
├── 2-sitemap/
│   ├── sitemap.md              # Pages, routes, navigation tree
│   └── route-table.md          # Route paths, guards, params
├── 3-layout/
│   ├── shell.md                # App shell, nav pattern, shared scaffold
│   ├── components.md           # Reusable widget inventory
│   └── theme.md                # Colors, typography, spacing, design tokens
├── 4-data/
│   ├── models.md               # Data models / entities
│   ├── providers.md            # State management (Riverpod providers)
│   └── api-integration.md      # Backend client setup, endpoints used
├── 5-features/
│   └── {feature-name}.md       # One file per feature — screens, logic, deps
└── audit/
    └── audit-log.md            # Timestamped log of every phase transition
```

### master-plan.md

This is the source of truth. It tracks:

- **Current phase** (Interview / Sitemap / Layout / Data / Features / Build)
- **Status of each phase** (not started / in progress / audited / locked)
- **Key decisions** (with rationale)
- **Open questions** (unresolved items blocking the next phase)
- **Change log** (timestamped entries when phases transition)

Always read `scaffold/master-plan.md` first when resuming work. Always update it when a phase completes.

---

## Phase 1: Interview

**Goal:** Understand what the developer wants to build.

Ask these questions interactively. Don't dump them all at once — have a conversation. Group into natural clusters and adapt based on answers.

### Core Identity
- What is the app called? One-line description?
- Who is the target user? (consumer, enterprise, internal tool)
- Target platforms? (iOS, Android, Web, Linux, macOS, Windows)

### Authentication & Backend
- Which auth flows? (email OTP, Google, Apple, passkeys/WebAuthn, SIWE, Privy, none)
- Backend? (Enforcer API, Firebase, Supabase, custom REST, custom GraphQL, none)
- If Enforcer: which features? (KV, storage, wallets, messaging, contacts, groups)
- Multi-tenant? (single tenant, multi-tenant with tenant selector, auto-resolved)

### Features & Pages
- List the main features / screens (keep it high-level, 5-15 items)
- Which features are auth-gated vs public?
- Any bottom tab nav? Drawer? Both?
- Onboarding flow? (tutorial screens, terms acceptance, profile setup)

### Technical Preferences
- State management? (Riverpod recommended, Bloc, Provider, other)
- Navigation? (GoRouter recommended, auto_route, Navigator 2.0)
- Design system? (Material 3, Cupertino, custom design system)
- Responsive? (mobile-only, mobile+tablet, mobile+tablet+desktop)
- Offline support needed?
- Push notifications?
- Deep linking?

### Constraints
- Existing codebase to integrate with, or greenfield?
- Timeline / MVP scope? (what ships first vs later)
- Any design mockups, Figma links, or reference apps?

**Output:** Write `scaffold/1-interview/requirements.md` with all captured requirements, organized by category. Create `scaffold/master-plan.md` with Phase 1 marked complete.

**Transition gate:** Read requirements.md back to the developer. Ask: "Is this complete and accurate? Anything to add or change?" Do NOT proceed until they confirm.

---

## Phase 2: Sitemap

**Goal:** Define every page/route in the app and how they connect.

Using the requirements from Phase 1:

1. **Route tree** — hierarchical list of every screen with its route path
2. **Auth gates** — which routes require authentication, which are public
3. **Navigation pattern** — bottom tabs, drawer, or hybrid; which screens live in which nav context
4. **Deep links** — which routes support deep linking and their URI schemes
5. **Params & guards** — route parameters, redirect rules, role-based guards

**Output:**
- `scaffold/2-sitemap/sitemap.md` — visual route tree with annotations
- `scaffold/2-sitemap/route-table.md` — table: route path, page name, auth required, params, parent

**Transition gate:** Recursive audit — re-read `requirements.md` + both sitemap files. Verify every feature from requirements has at least one route. Flag any orphaned routes or missing features. Present the audit to the developer. Do NOT proceed until they confirm.

---

## Phase 3: Layout

**Goal:** Define the visual structure, shared components, and design system.

1. **App shell** — the persistent scaffold (app bar, bottom nav, drawer, FAB placement). Define which sitemap routes share which shell configuration.
2. **Component inventory** — reusable widgets needed across multiple screens (list tiles, cards, form fields, dialogs, bottom sheets, empty states, error states, loading states)
3. **Theme** — Material 3 color scheme (seed color + surface variants), typography scale, spacing constants, border radii, elevation levels. Dark mode strategy (auto/manual/none).
4. **Responsive strategy** — breakpoints, layout shifts (e.g., bottom nav → rail → drawer at tablet), grid behavior

**Output:**
- `scaffold/3-layout/shell.md` — shell definition with nav mapping
- `scaffold/3-layout/components.md` — widget inventory with usage context
- `scaffold/3-layout/theme.md` — design tokens

**Transition gate:** Recursive audit — re-read ALL prior artifacts (requirements, sitemap, route table, shell, components, theme). Verify the shell covers all navigation contexts from the sitemap. Verify components cover the UI patterns implied by the features. Present findings. Do NOT proceed until developer confirms.

---

## Phase 4: Data

**Goal:** Define state management, data models, and backend integration.

1. **Models** — Dart classes/freezed models for every entity (User, Session, etc.). Include fields, types, JSON serialization notes.
2. **Providers** — Riverpod provider tree. For each provider: type (StateNotifier, FutureProvider, StreamProvider), what it depends on, what state it holds.
3. **API integration** — which backend endpoints each feature calls, how the SDK is initialized, auth token lifecycle (storage, refresh, logout), error handling strategy.

If using Enforcer, reference the enforcer-docs MCP server: `list_endpoints(tag: "...")` → `get_endpoint(operationId)` for exact request/response shapes.

**Output:**
- `scaffold/4-data/models.md` — entity definitions
- `scaffold/4-data/providers.md` — provider tree with dependencies
- `scaffold/4-data/api-integration.md` — endpoint mapping per feature

**Transition gate:** Recursive audit — re-read ALL artifacts. Verify every sitemap route has the providers it needs. Verify every feature's API calls are mapped. Verify models cover all API response types. Present findings. Do NOT proceed until developer confirms.

---

## Phase 5: Features

**Goal:** Define each feature in detail — screens, business logic, dependencies, edge cases.

For each feature identified in Phase 1, create `scaffold/5-features/{feature-name}.md` containing:

- **Screens** — which routes from the sitemap this feature owns
- **User flow** — step-by-step interaction (what the user sees and does)
- **Providers used** — which providers from Phase 4
- **API calls** — which endpoints, in what order, error handling
- **Edge cases** — empty state, error state, loading, offline, permissions
- **Dependencies** — other features this depends on or is depended on by
- **MVP vs later** — what ships first, what's deferred

**Output:** One `scaffold/5-features/{feature-name}.md` per feature.

**Transition gate:** Final recursive audit — re-read the ENTIRE `scaffold/` directory. Verify:
- Every requirement from Phase 1 is addressed by a feature
- Every route from Phase 2 is owned by a feature
- Every component from Phase 3 is used by at least one feature
- Every provider from Phase 4 is consumed by at least one feature
- No circular dependencies between features
- MVP scope is clear

Present the full audit. This is the last gate before code generation.

---

## Phase 6: Build

**Goal:** Generate the actual Flutter project structure from the scaffold.

This phase reads the entire `scaffold/` directory and generates:

1. **Directory structure** — `lib/src/` with feature-based organization
2. **Route configuration** — GoRouter setup from sitemap
3. **Theme** — `ThemeData` from theme.md tokens
4. **Models** — Dart classes from models.md
5. **Providers** — Riverpod providers from providers.md
6. **Screens** — stub widgets for every route, wired to providers
7. **App shell** — scaffold/nav from shell.md

Code generation should be incremental — generate one feature at a time, verify it compiles (`flutter analyze`), then move to the next. Use the Dart MCP server for hot reload and error checking during generation.

---

## Rules

1. **Never skip phases.** Phase order is 1 → 2 → 3 → 4 → 5 → 6.
2. **Never skip audits.** Every phase transition requires reading ALL prior artifacts and getting developer confirmation.
3. **master-plan.md is the source of truth.** Read it first, update it on every transition.
4. **One phase at a time.** Complete the current phase before starting the next.
5. **Interview is a conversation.** Don't ask all questions at once. Adapt based on answers.
6. **Scaffold is planning, not code.** Phases 1-5 produce markdown. Phase 6 produces code.
7. **Changes cascade.** If the developer changes a requirement, re-audit all downstream artifacts and update them.
8. **Use MCP tools.** For Enforcer-backed apps, use `enforcer-docs` MCP tools to look up endpoints. For Flutter, use the `dart` MCP server for analysis and hot reload.
