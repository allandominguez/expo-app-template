# expo-app-template

[![CI](https://github.com/allandominguez/expo-app-template/actions/workflows/ci.yml/badge.svg)](https://github.com/allandominguez/expo-app-template/actions/workflows/ci.yml)
[![CodeQL](https://github.com/allandominguez/expo-app-template/actions/workflows/codeql.yml/badge.svg)](https://github.com/allandominguez/expo-app-template/actions/workflows/codeql.yml)

A batteries-included Expo + TypeScript starting point: strict TypeScript, linting/formatting, pre-commit hooks, tests, and CI wired up from the first commit — so a new project starts at "ready to build features," not "ready to configure tooling."

Extracted from the base setup redone from scratch across a couple of real Expo projects — this collects that tooling into one reusable starting point instead of re-deriving it each time.

---

## Using this template

Click **Use this template** on GitHub, or:

```bash
gh repo create <new-repo-name> --template allandominguez/expo-app-template --public --clone
```

Then update `app.json`/`package.json`'s `name`/`slug` and `README.md`'s title for the new project.

Defaulting to `--public` here is deliberate, not just a style choice — see [`MAINTENANCE.md`](MAINTENANCE.md) for why a private repo breaks the CodeQL workflow this template ships with.

---

## What's included

- TypeScript strict mode
- ESLint flat config (`eslint-config-expo` + Prettier as a lint rule)
- Husky pre-commit hooks: `gitleaks` secret scanning, `lint-staged` (`eslint --fix` + `prettier --write`)
- Jest via `jest-expo` + `@testing-library/react-native`
- GitHub Actions CI (lint, typecheck, test) on every PR and push to `main`
- CodeQL static analysis (PR, push to `main`, and weekly schedule)
- Dependabot, with `ignore` rules for the packages Expo SDK-curates (`react`, `react-native`, `jest`, `typescript`, `@types/react`, `@types/jest`) — see [`MAINTENANCE.md`](MAINTENANCE.md) for what to do when one of these PRs fails CI anyway
- `.nvmrc` Node pinning
- `features/`/`lib`/`navigation/` folder convention (see [Project Structure](#project-structure))

Not included: a database layer, navigation library, or any other feature-shaping dependency. Those are decisions specific to what you're building — add them per project rather than inheriting one project's choices.

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) 22.13+ (Node 24 LTS recommended — see [`.nvmrc`](.nvmrc); this is what CI runs)
- [Expo Go](https://expo.dev/go) on a device, or a connected device/emulator

### Node version

The repo pins Node via [`.nvmrc`](.nvmrc), and CI reads that same file. A version manager keeps your local Node in step automatically — with [fnm](https://github.com/Schniz/fnm):

```bash
brew install fnm
eval "$(fnm env --use-on-cd)"   # add to your shell profile — see fnm's shell setup docs
fnm install                     # reads .nvmrc, installs the pinned Node
```

`cd`-ing into the project then selects the pinned version automatically. [`nvm`](https://github.com/nvm-sh/nvm) works too — it reads the same `.nvmrc` (without the auto-switch-on-`cd`).

### Installation

```bash
git clone <repo-url>
cd <repo-dir>
npm ci
```

### Running the app

```bash
npm start        # prompts to choose platform
```

---

## Project Structure

```
<repo>/
├── features/
│   └── <name>/             # One folder per feature
│       ├── components/     # Screens and UI components
│       ├── hooks/          # Feature-specific hooks
│       └── types.ts        # Feature-specific types
├── lib/                    # Shared infrastructure (storage helpers, utilities)
├── navigation/              # React Navigation root and stack definitions
├── assets/
├── App.tsx                 # Root component
├── index.ts                # Expo entry point
└── app.json
```

Feature folders are self-contained — components, hooks, types, and any storage calls for a feature live together under `features/<name>/`, rather than being split across top-level `components/`, `hooks/`, or `services/` directories.

---

## Branching

| Branch                       | Purpose                                        |
| ----------------------------- | ---------------------------------------------- |
| `main`                        | Production-ready code                          |
| `add/desc`, `update/desc`     | New features                                   |
| `fix/desc`                    | Bug fixes                                      |
| `chore/desc`, `improve/desc`  | Maintenance tasks (dependencies, config, etc.) |

---

## Development

```bash
npm run lint       # expo lint (ESLint) on .ts/.tsx
npm run typecheck  # tsc --noEmit
npm test           # Jest
```

All three run on every pull request and on pushes to `main` via [GitHub Actions](.github/workflows/ci.yml), and a passing run is required to merge. Pre-commit hooks (gitleaks secret scanning, lint-staged) run the same lint/format checks locally on staged files.

---

## Architecture Principles

This template is built on a small, deliberately curated set of engineering principles — package-by-feature structure, responsibility separation between hooks and components, YAGNI/simple design, and Kent Beck's Test Desiderata for testing — each one grounded in where it actually comes from and where it's genuinely practiced in this codebase, not cited for the sake of it.

See [`ARCHITECTURE.md`](ARCHITECTURE.md) for the full reasoning and sources.

---

## Contact

**Allan Dominguez**
[Portfolio](https://allandominguez.dev/) | [GitHub](https://github.com/allandominguez) | [LinkedIn](https://www.linkedin.com/in/allan-dominguez-113625146/) | [Email](mailto:allan.c.dominguez@gmail.com)

---

## License

MIT License — see [LICENSE](LICENSE) file for details.
