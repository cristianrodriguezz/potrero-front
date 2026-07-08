# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Product context

This app matches two amateur football (soccer) teams against each other, then lets them agree on a day/time/place to meet up and actually play the match in real life. Core product requirements:

- **Frontend-only scope**: this repo is the mobile client only. It consumes a separate backend API built with NestJS — do not design, mock, or take ownership of backend logic; assume the API contract exists (or is being built) elsewhere.
- **Multi-language**: the app must support multiple languages from the start (see i18n stack below). Don't hardcode user-facing strings.
- **Cross-platform**: ships to both Android and iOS (Expo managed workflow).
- **UX/UI bar**: this is a consumer-facing app — prioritize a fluid, polished, low-friction experience, not just functional correctness.
- **Global/themeable styling**: all visual styles (colors, spacing, typography) must be defined centrally so the color palette or theme can be swapped later without touching component code. Avoid one-off inline colors/styles in feature code.

## Target tech stack

The stack below is the intended direction for this project (some pieces may not be installed/wired up yet — check `package.json` for what's actually present before assuming a library is available):

| Layer | Choice |
|---|---|
| Framework | React Native + Expo (managed workflow) |
| Routing | Expo Router — file-based, same mental model as TanStack Router |
| Data fetching | TanStack Query — same library, query keys, and invalidation patterns as the web app |
| Global state | Zustand (token, user, activeTeamId) + `expo-secure-store` for the refresh token |
| UI | NativeWind (Tailwind for RN) + React Native Reusables or Tamagui |
| Forms | React Hook Form + Zod — schemas are portable across web/mobile |
| Maps | `react-native-maps` + `expo-location` — native maps, better perf than Leaflet-in-WebView |
| Auth | Supabase JS SDK, native OAuth via deep link; JWT validated by the existing NestJS backend — no backend changes needed |
| Push notifications | Expo Notifications (abstracts FCM + APNs); backend only needs an endpoint to register push tokens |
| i18n | i18next + react-i18next + `expo-localization` — same libraries as the web app |
| Release | EAS Build + EAS Submit (CI-driven builds for both stores, no Mac needed for iOS) + EAS Update for OTA hotfixes |

When adding a dependency for one of these concerns, prefer the library named above over an alternative, to stay consistent with the parallel web codebase.

## Expo version note

This project runs Expo SDK ~54. APIs and conventions have changed significantly from earlier SDKs (see AGENTS.md) — check the versioned docs at https://docs.expo.dev/versions/v54.0.0/ before relying on prior knowledge of Expo.

## Commands

- `npm start` — start the Expo dev server (Metro), then choose a platform from the CLI output
- `npm run android` / `npm run ios` / `npm run web` — start the dev server targeting a specific platform directly
- `npm run lint` — run ESLint via `expo lint` (flat config in `eslint.config.js`, extends `eslint-config-expo`)
- `npm run reset-project` — one-way move of the starter `app/` code to `app-example/` and creates a blank `app/`; only for discarding the starter template, not routine dev

There is no test runner configured in this project.

## Architecture

This is an Expo Router (file-based routing) app. Routes live under `app/`, and the file/folder structure there **is** the navigation structure:

- `app/_layout.tsx` — root layout; wraps everything in a `ThemeProvider` (light/dark via `useColorScheme`) and defines the root `Stack`. `unstable_settings.anchor` is set to `(tabs)` so that group is the initial route.
- `app/(tabs)/` — a route group rendering the bottom tab navigator (`index.tsx` home tab, `explore.tsx` second tab); the group's own layout/tab bar config is expected alongside it.
- `app/modal.tsx` — presented as a modal `Stack.Screen` from the root layout.

Path alias `@/*` maps to the project root (configured in `tsconfig.json`), so imports use `@/components/...`, `@/hooks/...`, `@/constants/...` rather than relative paths — follow this convention for new files.

Key shared modules:
- `constants/theme.ts` — exports `Colors` (light/dark palettes) and `Fonts` (per-platform font stacks via `Platform.select`), the single source of truth for theming values.
- `hooks/use-color-scheme.ts` (+ `.web.ts` variant) and `hooks/use-theme-color.ts` — resolve the active color scheme and look up a themed color from `constants/theme.ts`.
- `components/themed-text.tsx` / `components/themed-view.tsx` — `Text`/`View` wrappers that apply theme colors automatically; prefer these over raw RN `Text`/`View` for new UI.
- `components/parallax-scroll-view.tsx` — scrollable screen wrapper with a parallaxing header image, used by both tab screens.
- `components/ui/icon-symbol.tsx` (+ `.ios.tsx` variant) — cross-platform icon abstraction (SF Symbols on iOS, fallback elsewhere).

Platform-specific files use the standard Expo/RN suffix convention (`.web.ts`, `.ios.tsx`) and are resolved automatically by the bundler based on target platform — don't import them conditionally in code.

App config (bundle identifiers, icons, splash screen, plugins) lives in `app.json`, not in code.
