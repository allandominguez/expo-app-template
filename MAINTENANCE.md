# Maintenance

A runbook for situations that recur throughout a project's life — not one-time setup steps, but things that will come back around on their own schedule (a Dependabot run, an upstream release) as long as the project is alive. Kept as its own file, separate from `AGENTS.md`, so it survives even on a project where `AGENTS.md` itself gets deleted — this information is worth keeping regardless of whether AI tooling is in the loop.

## Dependabot's `expo-sdk`-group PR fails CI

**Symptom:** A Dependabot PR for the `expo-sdk` group (or an individual `react`/`react-native`/`jest`/`typescript`/`@types/react`/`@types/jest` bump) fails CI — usually `expo-doctor` flagging a version-set misalignment, sometimes a broken `npm test` (e.g. `Cannot find module '@react-native/assets-registry/registry'`).

**Why it happens:** Dependabot resolves each package's latest semver-compatible version independently. It has no concept of Expo's SDK-curated version set — the specific combination of `react`/`react-native`/`jest`/etc. versions that a given Expo SDK release actually expects and tests against. A Dependabot bump that's individually valid semver can still break that curated set. That's why `dependabot.yml` permanently `ignore`s these six packages — Dependabot proposing them at all just reproduces this failure; only `expo install` (which reads Expo's own curation data) should move them.

**Fix:**

```bash
git checkout <dependabot-branch>       # or main, if the PR was already closed
npx expo install --check               # or --fix, to apply directly
git add package.json package-lock.json
git commit -m "Align deps to Expo SDK's expected versions"
git push
```

Then let CI re-run and merge once green.

**This is the expected, recurring way these packages get updated going forward** — not a one-off fix to eventually stop needing. Each Expo SDK release re-curates this version set, so this will keep happening on whatever cadence you upgrade the SDK.

## `eslint` major-version bump blocked

**Symptom:** `dependabot.yml` permanently ignores `eslint`'s major-version bumps (`update-types: ["version-update:semver-major"]`) — minor/patch bumps still flow through normally.

**Why:** `eslint-config-expo`'s bundled `eslint-plugin-react` doesn't support ESLint 10 yet. Letting the major bump through breaks `npm run lint` outright (`contextOrFilename.getFilename is not a function`).

**Fix/check:** Periodically check [`eslint-config-expo`'s changelog or npm page](https://www.npmjs.com/package/eslint-config-expo) for ESLint 10 support. Once it lands, remove the `eslint` entry from `dependabot.yml`'s `ignore` list and let Dependabot retry the major bump normally.
