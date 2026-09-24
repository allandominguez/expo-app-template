# expo-app-template

Reference documentation for this codebase's layout and commands. For the engineering principles this template is built on — and the reasoning behind them — see [`ARCHITECTURE.md`](ARCHITECTURE.md).

## Expo has changed

Expo v57 changed enough from earlier versions that old habits or cached assumptions will lead you wrong — check the exact versioned docs at https://docs.expo.dev/versions/v57.0.0/ before assuming a pattern still applies.

## Commands

```bash
# Start dev server (prompts to choose platform)
npm start

# Platform-specific
npm run android
npm run ios
npm run web

# Quality checks (also run in CI on every PR + push to main)
npm run lint        # expo lint (ESLint; Prettier runs as an ESLint rule)
npm run typecheck   # tsc --noEmit
npm test            # jest (jest-expo preset)
npm run format      # prettier --write .
```

Node is pinned via `.nvmrc` and managed locally with fnm — `cd` into the repo auto-selects it (see README). Non-interactive shells don't load the fnm hook, so use `fnm exec --using <version> -- <cmd>` when the Node version matters for a scripted run.

Pre-commit hooks (husky): `gitleaks` secret-scans staged files, then `lint-staged` runs `eslint --fix` + `prettier --write` on staged `.ts`/`.tsx`.

## Architecture

React Native + Expo app with TypeScript strict mode. Entry point is `index.ts` → `App.tsx`.

### Folder structure

Code is organized by feature under `features/`. Shared infrastructure lives in `lib/` and navigation in `navigation/`:

```
features/
  <name>/       # One folder per feature
    components/ # Screens and UI components for this feature
    hooks/      # Feature-specific hooks
    types.ts    # Feature-specific types
    (+ api.ts, context.tsx, etc. as needed)
lib/            # Shared infrastructure (storage helpers, utilities)
navigation/     # React Navigation root and stack definitions
App.tsx         # Root component, mounts navigation
index.ts        # Expo entry point
```

The reasoning behind this layout — and the rest of this project's engineering standards — is in [`ARCHITECTURE.md`](ARCHITECTURE.md).

### Testing

Jest via the `jest-expo` preset with `@testing-library/react-native`. Run `npm test` or `npm run test:watch`.
