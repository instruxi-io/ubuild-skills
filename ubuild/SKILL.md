---
name: ubuild
description: "Interactive Flutter app scaffolding framework. Interviews the developer, generates a structured scaffold plan, and builds out the app phase by phase. Trigger: 'ubuild', 'scaffold', 'new flutter app', 'app scaffolding'"
---

Before doing anything else, print this exactly:

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

uBuild is a phased, interview-driven scaffolding framework for Flutter apps. It generates a `scaffold/` directory containing structured planning documents, then uses those documents to build out the actual Flutter project **on top of the `ubuild-mobile` template**.

**You MUST follow the phases in order. Each phase produces artifacts in `scaffold/`. Transitioning to the next phase requires a recursive audit — re-read every artifact produced so far, verify consistency, flag contradictions, and get explicit developer approval before proceeding.**

---

## The Base Template

Every ubuild scaffold assumes the developer starts from **`ubuild-mobile`** (https://github.com/instruxi-io/ubuild-mobile), which ships with reusable boilerplate that you should NOT re-scaffold:

- **Backend:** Enforcer SDK (`enforcer_sdk` git dep, V2 API).
- **Auth flow:** Email OTP via `/auth/login` + `/auth/login/verify` (always on). Split `LoginEmailScreen` + `LoginOtpScreen`. `AuthState` is a sealed hierarchy (`Restoring | Unauthenticated | Loading | OtpSent | Authenticated`). Four login methods ship wired end-to-end:
  - **Email OTP** — always visible.
  - **Google Sign-In** — visible when `GOOGLE_CLIENT_ID` is set. On mobile/macOS/web it uses the `google_sign_in` plugin; on Linux/Windows it runs a loopback-redirect OAuth flow (`google_auth_linux.dart`) against a fixed port (default 39591, override with `--dart-define=GOOGLE_LOOPBACK_PORT=…`). Enforcer does the server-side token exchange, so no client secret lives in the app.
  - **Passkey / WebAuthn** — visible when `ENABLE_PASSKEY=true`. Auto-hidden on Linux (no platform implementation in the `passkeys` package).
  - **API key (dev only)** — visible when `ENV=dev`. Paste-and-go dialog; sends the key as `X-API-Key` via the `isApiKey` flag on `Authenticated`.
  **SIWE and Privy are NOT in the template** — add if needed.
- **JWT lifecycle:** persisted via `flutter_secure_storage` (`TokenStorage`). `AuthInterceptor` handles `401 → /auth/refresh → retry` with concurrent-401 coalescing. SDK's api-key kept in sync via `ref.listen(authStateProvider)` inside `enforcerSdkProvider` so hot-reload provider rebuilds don't drop the bearer.
- **Onboarding:** two gated steps — `VerifyIdentityScreen` (SMS phone OTP via `/auth/verify/phone/{send,confirm}`) + `ProfileSetupScreen` (`PATCH /users/me`). `OnboardingStatus` derives `nextStep` from `AccountSnapshot`.
- **App shell:** `RailShell` (NavigationRail, desktop-first, falls back at narrow widths) + `OnboardingShell` (step indicator) + bare (splash/login). Template rail has **Home + Profile** as seed destinations — add your own.
- **Routing:** GoRouter with auth + onboarding redirect guards, bridged to Riverpod via a `ChangeNotifier` on `refreshListenable`.
- **State:** Riverpod.
- **Theme:** Material 3, seed-color-driven, `AppSpacing` / `AppRadii` / `AppBreakpoints` tokens, `AppStatusColors` `ThemeExtension` (verified/pending/unverified). Inter typeface via `google_fonts`. `ThemeMode.system`.
- **Shared widgets:** `AsyncValueBuilder`, `EmptyState`, `ErrorState`, `LoadingScaffold`, `UserAvatar`, `VerificationStatusChip`.
- **Config:** `EnvConfig` loads from `assets/env/.env.<flavor>` selected via `--dart-define=ENV=<dev|staging|prod>`. Bundled env files have `ENFORCER_BASE_URL` + `ENFORCER_TENANT_CODE`.

**Implication for the interview:** don't ask "which auth flow?" or "state management?" or "routing package?" — they're fixed. Ask about the *app-specific* features the developer is building on top.

**Implication for Phase 6:** the template already provides splash, login, onboarding, shell, router, theme, and the Enforcer wiring. Phase 6 generates the *new* feature code (screens, providers, repositories) and hooks it into the existing router + rail. Do NOT overwrite the template files without explicit developer consent.

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
Backend is Enforcer; email OTP is always on. Don't re-ask those. Ask instead:
- Which Enforcer domains are in scope? (KV, storage, wallets, messaging, contacts, groups, verification)
- Tenancy: single fixed tenant, or tenant selector? (default: single, resolved from `ENFORCER_TENANT_CODE` env)
- Enable Google Sign-In? (set `GOOGLE_CLIENT_ID`; on Linux/Windows the client must be a "Desktop app" type and the tenant's Google connection `redirect_url` must match the loopback port)
- Enable Passkey / WebAuthn? (set `ENABLE_PASSKEY=true`; auto-hidden on Linux)
- API key login in dev? (always on when `ENV=dev` — paste a tenant API key and go)
- SIWE or Privy needed? (not in template — requires custom wiring)

### Features & Pages
- List the main features / screens (keep it high-level, 5-15 items)
- Which features are auth-gated vs public?
- Any bottom tab nav? Drawer? Both?
- Onboarding flow? (tutorial screens, terms acceptance, profile setup)

### Technical Preferences
State management = Riverpod, routing = GoRouter, design = Material 3 — all fixed by the template. Ask instead:
- Responsive target? (mobile-only, mobile+tablet, mobile+tablet+desktop — template is desktop-first with portability fallbacks)
- Offline support needed?
- Push notifications?
- Deep linking? (template has `/splash`, `/login`, `/login/verify`, `/onboarding/*`, `/home`, `/profile` — list your deep-linkable additions)

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

**Goal:** Add the app's features on top of the `ubuild-mobile` template.

### What the template already provides (do NOT regenerate)

- `lib/src/auth/` — email OTP + JWT + `AuthInterceptor`
- `lib/src/core/sdk_provider.dart` — Enforcer SDK with bearer-sync listener
- `lib/src/config/env_config.dart` — flavor-switching env loader
- `lib/src/router/app_router.dart` — GoRouter + auth/onboarding guards
- `lib/src/shell/{rail_shell,onboarding_shell}.dart`
- `lib/src/shared/theme/{app_theme,spacing,status_colors}.dart`
- `lib/src/shared/widgets/*.dart` — AsyncValueBuilder, EmptyState, ErrorState, LoadingScaffold, UserAvatar, VerificationStatusChip
- `lib/src/features/{account,onboarding,profile,splash,home}/`

### What Phase 6 generates

1. **New feature directories** under `lib/src/features/<feature-name>/` — models, repositories, providers, screens.
2. **Rail destinations** — edit `rail_shell.dart`'s `_destinations` list to add the app's primary tabs.
3. **Router additions** — append new `GoRoute` entries inside the existing `ShellRoute` for the rail-shelled routes, or as top-level routes for full-screen flows.
4. **Theme tweaks** — seed color override, extra `ThemeExtension`s if the app has custom status states.
5. **Additional onboarding steps** (if the app needs them) — extend `OnboardingStep` + `OnboardingStatus` + add the route.
6. **New providers** — append to the existing Riverpod tree; never replace auth/account/onboarding providers.

Code generation should be incremental — generate one feature at a time, verify it compiles (`flutter analyze`), then move to the next. Use the Dart MCP server for hot reload and error checking during generation.

### Rules of thumb

- If the scaffold references `SomeProvider` that already exists in the template (e.g., `accountProvider`), import it — don't redefine.
- If the scaffold defines a new rail destination, add the route + edit `rail_shell.dart`'s `_destinations`; don't build a whole new shell.
- Keep the `Ardis` → `App` naming convention: nothing in the generated code should be named after a product unless the developer explicitly asks.

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
