# Architecture & Engineering Principles

This is a deliberately curated statement of the principles this template is built on — not an AI-context file, and not an attempt to cite every well-known idea in software design. Each entry below is included because it maps to something concretely practiced here; where a principle from established literature doesn't have a real instance in this codebase yet, it's left out rather than padded in for the citation, and added later if a real instance shows up.

## Structure: package-by-feature

**Source:** Robert C. Martin, *Clean Architecture* — "Screaming Architecture."

Code is organized by feature (`features/<name>/`), not by technical layer (`components/`, `hooks/`, `services/`). Martin's argument is that a codebase's top-level structure should announce what the system *does*, not which framework it uses — opening the top level of this repo should tell you it's an app with features, not just "a React Native project." A layer-first structure inverts that: opening `hooks/` or `components/` tells you nothing about the product, and every change to one feature touches directories shared by every other feature, inviting accidental coupling.

**In this repo right now:** `features/`, `lib/`, and `navigation/` exist as the convention, each holding only a `.gitkeep` — there's no feature code yet to organize, but the shape is already committed to.

## Responsibility: hooks own logic, components own rendering

**Source:** SOLID's Single Responsibility Principle, applied to React's component model; the boundary-mocking approach below follows from SOLID's Dependency Inversion Principle.

Business logic (data fetching, validation, derived state) lives in custom hooks; components stay thin rendering layers. This is SRP applied at the hook/component boundary rather than the class boundary SOLID was originally written for: a component has one reason to change (the UI changed), a hook has a different one (the logic changed). Testing follows from this — mock at system boundaries (device APIs, external services), not between a hook and the component that calls it. That's only a coherent testing strategy because the component depends on the hook's *interface*, not its internals — Dependency Inversion, not just a testing convenience.

**In this repo right now:** no illustration — `App.tsx` is still the unmodified Expo starter screen with no hooks, and `features/` is empty. This is a convention for code that doesn't exist yet, not a demonstrated pattern. It'll hold once the first feature lands.

## Simplicity: YAGNI, simple design, and earned abstraction

**Sources:** Kent Beck's four rules of simple design (passes its tests, reveals intent, no duplication, fewest elements — in that priority order); "You Aren't Gonna Need It" (Extreme Programming); Martin Fowler / Sandi Metz's observation that duplication is cheaper to live with than the wrong abstraction.

Don't build for a requirement you don't have yet. Three similar lines of code are better than a premature shared abstraction — the abstraction can always be extracted once a real third use case proves what it should actually look like; guessing at that shape upfront usually guesses wrong, and the wrong abstraction costs more to unwind than the duplication would have cost to tolerate. The same reasoning applies one level up, to whole dependencies: don't wire in a library for a need the project doesn't have yet, even if you're fairly sure you'll want it eventually — and one level down, to individual optimizations: don't reach for `useMemo`/`useCallback` speculatively. React's reconciler handles most re-renders cheaply on its own; memoizing before a concrete performance problem is observed adds complexity with no measurable benefit, and obscures the actual data flow.

**In this repo right now:** `package.json` has no `expo-sqlite`, `expo-asset`, or `better-sqlite3` — an earlier draft of this template wired up local SQLite storage by default, reasoning that most projects built from it would want it. That reasoning didn't hold up: local relational storage is a specific architectural choice (see [leaf](https://github.com/allandominguez/leaf-app), a local-first app that does need it), not a default every Expo app shares — `lib/.gitkeep` is empty rather than holding a placeholder data layer as a result. Re-adding SQLite when a project actually needs it is a small, additive change; the template stays lean until then.

## Testing: Kent Beck's Test Desiderata

**Source:** Kent Beck's Test Desiderata — the properties a good test should have (isolated, deterministic, behavioral, fast, readable, and others).

- **Test behaviour, not implementation** — assert what a user or consumer observes (rendered output, state changes, navigation), not which internal functions were called. A behavioral test survives a refactor that doesn't change behaviour; an implementation test breaks on it anyway.
- **Mock at system boundaries** — device APIs, external services — not between layers within a feature. A boundary mock verifies the feature's actual contract; mocking a sibling hook or component couples the test to internal wiring instead.
- **A mock-call assertion is still behavioural when the call itself is the outcome being verified** — a callback prop firing, a navigation action, a boundary mock receiving the data it should persist. These have no other observable signal in a test environment.
- **Test names read as user-facing scenarios** — "shows the item in search results after capture," not "calls insertItem with the correct argument." A failing test should tell you what broke for the user, not just which function was involved.

**In this repo right now:** `App.test.tsx` asserts on rendered text (`screen.getByText(...)`), not on any internal call — a small but real instance of the behavioural-testing property. It doesn't yet illustrate boundary-mocking; there's no boundary to mock until a feature introduces one.

## Professional practices (not tied to a named principle)

Some things here are deliberate discipline rather than an application of citable literature — worth stating plainly rather than force-fitting a source that doesn't really apply:

- **Accessibility from day one** — add `accessibilityLabel`/`accessibilityRole` at authoring time, not as a retrofit. Cheap while you're already reasoning about what an element does; expensive to add back later across every component.
- **Vulnerability triage is documented, not silent** — `npm audit` (or similar) findings that are safe to accept get a written rationale in `SECURITY.md` (what the finding is, why it can't reach the shipped bundle, what would trigger re-triage), rather than being silently ignored or force-fixed with a breaking change.
