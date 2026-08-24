# AGENTS.md

Instructions for AI coding agents working in this repository. Prefer
mechanical enforcement (linters, CI, tests) over relying on the agent to
remember rules — treat this file as a fallback for judgment calls linters
can't make.

If the repository already has an established style or pattern that
conflicts with something below, follow the repository. This file is a
default for cases the codebase doesn't already decide.

Don't apply every rule below mechanically regardless of context. Think
about what the task actually needs, then apply the rules that are
relevant — blindly pattern-matching every rule against every change ends
up misapplying rules to situations they don't fit.

## Scope of changes

- Don't touch code blocks unrelated to the current task, even if you notice
  something you'd like to improve. Flag unrelated issues for a human to
  decide on instead of fixing them inline.
- If you hit something that blocks clean progress — a flaky test, an
  ambiguous requirement, missing information, a decision only a human can
  make, a check that fails for a reason unrelated to your change — report
  it and ask, rather than silently working around it, suppressing it, or
  guessing.
- Don't extract single-use logic into a helper function purely to make the
  diff look smaller — prioritize good design over a small diff.

## User-facing impact

- If a change alters user-visible behavior or a public/external API
  contract — even as a side effect of a pure-looking cleanup — flag it
  explicitly rather than shipping it silently. That's a decision someone
  outside engineering may need to make, not just a code-quality call.
- Don't remove or alter analytics, telemetry, or logging calls as a side
  effect of a refactor without calling it out. They may be relied on for
  product metrics in ways that aren't visible from the code alone.

## Design judgment

- Before writing new logic, look for an existing function, utility, or
  dependency in the codebase that already does it (or most of it). Don't
  duplicate what already exists, and don't create a new near-duplicate
  function just to avoid touching an existing one — adjust the shared
  function (with new parameters if needed) when that's the right design.
- Don't add configurability, parameters, or abstraction layers for
  hypothetical future needs the task didn't ask for. Build for the
  requirement in front of you.
- Be aware of obviously poor complexity or performance for the problem
  size at hand (e.g. an avoidable O(n²) loop, a query issued inside a
  loop) — but don't optimize beyond what the task actually needs.

## Naming

- Names should be clear and unambiguous, not clever or abbreviated. Prefer
  a slightly longer, obvious name over a short, cryptic one.
- No hard character limit — clarity wins over brevity when the two conflict.

## Code structure

If the project has a linter/formatter configured, defer to it for anything
syntactic (bracing, indentation, line length, single-statement block
style) — don't restate those as prose rules here. The points below are
worth stating on their own merits, even where a linter also happens to
flag them:

- Reduce nesting: use early returns / guard clauses instead of deep
  conditional chains.
- Avoid nested ternaries (or equivalent inline conditional expressions);
  flatten complex conditionals into named, intermediate variables or
  statements.
- Extract recurring magic numbers/strings into named constants; leave
  genuine one-off values inline.
- Prefer an enum (or equivalent named-variant type) over a bare boolean
  parameter when the meaning of `true`/`false` isn't obvious at the call
  site.
- Keep fields and functions as private/internal as possible by default;
  only widen visibility when there's a concrete reason to.
- Encapsulate low-level mechanics (I/O, parsing, protocol details) behind a
  higher-level API; don't let those details leak across module boundaries.
- Respect existing architectural layers: a module should talk to its
  immediate neighbors, not reach across or skip a layer to call something
  two levels away, even if that would be a shorter path.

## Defensive programming

- Don't swallow or ignore errors reflexively. Only catch an error where you
  can do something meaningful with it — retry, add context, or degrade
  gracefully (which can include logging and continuing, e.g. skipping one
  bad item in a batch and processing the rest, when that's a deliberate
  choice). Otherwise let it propagate.
- Close or release what you open — files, connections, locks, handles —
  including on error paths, using whatever the language's standard
  mechanism is (`finally`, `defer`, `using`, RAII, context managers, etc.).
- Consider null/nil/empty/zero/negative and other boundary values
  explicitly, not just the happy path, before treating something as done.
- Treat external input (user input, network responses, file contents,
  environment variables) as untrusted at the boundary where it enters the
  system; validate or sanitize it there. Never hardcode secrets or
  credentials in code.

## Comments

- Comments explain *why*, not *what*. The code already says what it does;
  a comment repeating that in prose is noise.
- Add a comment when there's a non-obvious constraint, a subtle invariant, a
  workaround for a specific bug, or a decision that would otherwise look
  arbitrary.
- If code needs a comment to explain *what* it does, consider whether the
  code itself should be clearer instead.
- For system-level architecture that's genuinely hard to convey in prose,
  link to an external diagram or doc rather than drawing ASCII art in
  comments — agents are unreliable at producing and maintaining ASCII
  diagrams.

## Testing

- New functionality needs test coverage, sized to the risk and complexity
  of the change — not just bug fixes.
- For bug fixes specifically: write a test that reproduces the bug and
  fails first, then write the fix, then confirm the test passes.
- Remove scaffolding/throwaway tests that only existed to drive
  implementation; keep tests that verify real behavior going forward.
- Don't mark work done based on your own read of the code — run the tests
  (and the app, where applicable) and report actual results.

## Commits / diffs

- Keep commits focused on one logical change.
- Subject line: imperative mood, concise (roughly 50 characters), no
  trailing period.
- Explain *what* changed and *why* in the body when it's not obvious from
  the subject and diff alone — not a line-by-line narration of *how*.
- Don't add AI co-author trailers (`Co-Authored-By`, session links, etc.).
  The human who reviews and ships the change is the author of record —
  they're the one accountable for it, and authorship should signal that.

## Stopping points

A change is done when one of these is true:

- **Success** — the requested capability works and is verified (tests pass,
  or you've run/exercised the change).
- **Meaningful progression** — a real blocker was removed and the next step
  is clearly scoped, even if the whole task isn't finished yet.
- **Honest stop** — you've hit a limit covered by the "report it and ask"
  rule above, or the scope has turned out to be much larger than expected,
  and you say so explicitly instead of pushing on.

Don't confuse activity with progress. Stop and say why instead of
continuing when either of these happen:

- The same approach has failed twice in a row (e.g. two different fixes
  for the same test still fail it).
- A dependency, API, or capability the fix needs doesn't exist and would
  itself be a separate, unscoped piece of work.

## Context hygiene

If you notice your own output drifting from these rules or losing track of
earlier context in a long session, say so explicitly and suggest starting
a fresh session rather than silently continuing at reduced quality.

(Maintainers: keep this file itself short — a bloated instructions file
dilutes the rules that matter.)
