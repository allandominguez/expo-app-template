# Maintenance

A runbook for situations that recur throughout a project's life — not one-time setup steps, but things that will come back around on their own schedule (a Dependabot run, an upstream release) as long as the project is alive. Kept as its own file, separate from `AGENTS.md`, so it survives even on a project where `AGENTS.md` itself gets deleted — this information is worth keeping regardless of whether AI tooling is in the loop.

## Dependabot's `expo-sdk`-group PR fails CI

**Symptom:** A Dependabot PR for the `expo-sdk` group (or an individual `react`/`react-native`/`jest`/`typescript`/`@types/react`/`@types/jest` bump) fails CI — usually `expo-doctor` flagging a version-set misalignment, sometimes a broken `npm test` (e.g. `Cannot find module '@react-native/assets-registry/registry'`).

**Why it happens:** Dependabot resolves each package's latest semver-compatible version independently. It has no concept of Expo's SDK-curated version set — the specific combination of `react`/`react-native`/`jest`/etc. versions that a given Expo SDK release actually expects and tests against. A Dependabot bump that's individually valid semver can still break that curated set. That's why `dependabot.yml` permanently `ignore`s these six packages — Dependabot proposing them at all just reproduces this failure; only `expo install` (which reads Expo's own curation data) should move them.

**Fix:** First check the PR's diff. If it moves `expo` to a new major (e.g. 57 → 58), stop: this fix doesn't apply. `expo install --fix` would try to undo the jump and abort with `ERESOLVE`. Follow "Upgrading the Expo SDK" below instead, and close the PR as superseded afterwards. If the bumps stay within the current SDK:

```bash
git checkout <dependabot-branch>       # or main, if the PR was already closed
npx expo install --check               # or --fix, to apply directly
git add package.json package-lock.json
git commit -m "Align deps to Expo SDK's expected versions"
git push
```

Then let CI re-run and merge once green.

**This is the expected, recurring way these packages get updated going forward** — not a one-off fix to eventually stop needing. Each Expo SDK release re-curates this version set, so this will keep happening on whatever cadence you upgrade the SDK.

## Upgrading the Expo SDK

An SDK bump touches native modules, React, React Native, and the test toolchain at once, so it's worth doing the same way each time:

1. **Check that Expo Go supports the target SDK first.** If you develop in Expo Go on a device, the store build lags new SDKs, sometimes by months, and a project bumped ahead of it can't be tested on the device. The build installed directly from Expo has supported new SDKs sooner.
2. **One major at a time, one commit per hop** (e.g. 56 → 57, not 54 → 57 in one go), so a breakage is attributable to a single SDK. Run `npx expo install expo@~<n>.0.0`, then `npx expo install --fix`, then `npx expo-doctor`. If `--fix` aborts with an `ERESOLVE` peer conflict (moving to SDK 57 did, over `@react-native/jest-preset`), install the packages named in the error explicitly, then re-run it.
3. **Run the checks, then the app.** Lint, typecheck, tests and a real Android bundle (`npx expo export --platform android`) catch most breakage. Two kinds are easy to miss: an API that was removed but doesn't crash, so it quietly stops doing anything at runtime, and deprecation warnings that only log on a device. Each SDK's removals differ, so read its release notes, treat typecheck errors as real, then open the app and read the console.
4. **Re-run `npm audit` afterwards.** SDK bumps change the audit outcome.

## `eslint` major-version bump blocked

**Symptom:** `dependabot.yml` permanently ignores `eslint`'s major-version bumps (`update-types: ["version-update:semver-major"]`) — minor/patch bumps still flow through normally.

**Why:** `eslint-config-expo`'s bundled `eslint-plugin-react` doesn't support ESLint 10 yet. Letting the major bump through breaks `npm run lint` outright (`contextOrFilename.getFilename is not a function`).

**Fix/check:** Periodically check [`eslint-config-expo`'s changelog or npm page](https://www.npmjs.com/package/eslint-config-expo) for ESLint 10 support. Once it lands, remove the `eslint` entry from `dependabot.yml`'s `ignore` list and let Dependabot retry the major bump normally.

## CodeQL fails on a private repo

**Symptom:** The `CodeQL` workflow fails on every push with `Code scanning is not enabled for this repository. Please enable code scanning in the repository settings.` — not a workflow bug, and not fixable by changing `codeql.yml`'s `permissions:` block.

**Why:** CodeQL code scanning is part of GitHub Advanced Security. It's free and automatic on public repos, but on a personal (non-Enterprise) GitHub account, private repos don't get it at all unless Advanced Security has been purchased separately — confirmed via `gh api -X PATCH ... security_and_analysis` returning `"Advanced security has not been purchased."` This applies to _every_ project generated from this template, not just the template repo itself — it's an account-level constraint, not something specific to any one repo.

**Fix:** Make the repo public (`gh repo create ... --public`, or flip visibility later in Settings → General) — this is why the template's own usage instructions in `README.md` default to `--public` rather than `--private`. If a project genuinely needs to stay private, either purchase GitHub Advanced Security for the account, or remove `.github/workflows/codeql.yml` from that project — a red, permanently-failing CodeQL check isn't better than no CodeQL check at all.
