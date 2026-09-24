# Security Policy

## Reporting a Vulnerability

Please do not open a public issue for a security problem. Report it privately through GitHub's private vulnerability reporting:
<https://github.com/allandominguez/expo-app-template/security/advisories/new>

## Known Accepted Findings

Findings from `npm audit` (or other dependency/security scanners) that are confirmed to have no runtime reachability into the
device bundle are documented here with the reasoning, rather than silently ignored or forced through with a breaking fix.

### `uuid` < 11.1.1 (moderate) — via `xcode` → `@expo/config-plugins` → `expo`

**Advisory:** [GHSA-w5hq-g745-h8pq](https://github.com/advisories/GHSA-w5hq-g745-h8pq) — missing buffer bounds check in `uuid`
v3/v5/v6 when a buffer is supplied by the caller.

**Risk:** `npm audit` reports this finding against several packages in the dependency tree, but they are not independent
vulnerabilities — all of them are `expo`'s toolchain re-reporting the same advisory once per ancestor package. The actual
vulnerable code lives in `uuid@7.0.3`, pulled in by `xcode@3.0.1` (a `.pbxproj`/Xcode-project-file manipulation library), which is
only ever invoked by `@expo/config-plugins` during native project generation (`expo prebuild`, or the equivalent step inside
`expo run:android`/EAS Build). That step runs on the developer's machine or the build server, producing native project scaffolding
files — it is never part of the JavaScript bundle Metro ships to a user's device. The finding also has no exploitation path in this
project's threat model: the buffer the vulnerable code operates on is build-tooling-controlled, not attacker-supplied input.

**Why no fix is applied:** `xcode@3.0.1` pins its `uuid` dependency to `^7.0.3`. The only way to land a patched `uuid` (`>=11.1.1`)
is to force a major-version override into a range the package explicitly declared as `^7.x` — untested by `xcode`'s own author,
and risking breakage of `expo prebuild`/native builds. `npm audit`'s own suggested fix is a major downgrade of `expo`, which is not
a viable option for a project targeting a current SDK version.

**Re-check triggers:**

- The next time `expo`/`@expo/config-plugins` bumps its `xcode` dependency (or `xcode` bumps its `uuid` dependency) to a
  non-vulnerable version, re-run `npm audit` — this finding should resolve on its own without any action here.
- Independent of the above, re-triage this and any other accepted finding if: the total `npm audit` count climbs past ~20, any
  finding is `high`/`critical` severity, or a finding is confirmed reachable from a runtime (non-dev) dependency.

This finding is a property of Expo's own toolchain, not this project — it will reappear identically in any project generated from
this template until Expo resolves it upstream. Re-verify it's still the same finding (same advisory, same dependency chain) before
assuming this write-up still applies.
