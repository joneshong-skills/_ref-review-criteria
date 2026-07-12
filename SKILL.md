---
name: _ref-review-criteria
description: "Code review criteria, adversarial review mode, and structured output schema reference"
user-invocable: false
---

# Review Criteria

## Two-Axis Review — Standards + Spec (never merged)

Cannibalized from mattpocock/skills `code-review` (2026-07-12). A change is judged on two independent axes:

- **Standards axis** — does the code follow documented standards? (the checklist + smell baseline below)
- **Spec axis** — does the implementation actually do what the original issue / PRD / acceptance criteria asked for?

Rules:

- When a review covers both axes, run them as **separate passes** (parallel sub-agents when dispatched — they must not pollute each other's context); the aggregator only collects.
- **Do NOT merge or rerank findings across axes.** Report a verdict **per axis** — never a single winner across both. Code that follows every standard but implements the wrong thing is `Standards: pass, Spec: fail`, and that split state is exactly the information the separation exists to preserve.
- `schemas/review-output.schema.json` supports two-axis runs natively (2026-07-12): populate `axes.standards` / `axes.spec` with per-axis verdicts and tag every finding with its `axis`. The top-level `verdict` is then the **mechanical worst-of gate** across axes (ship only if both pass) — a computation, not a judgment merge; the per-axis verdicts carry the real information. Single-axis runs omit `axes`/`axis` and behave exactly as before. (Verified consumers of this schema: this file + `agents/reviewer.md` only — an earlier note claiming `code-review-interceptor` consumes it was wrong; it never references the schema.)

## Standard Review Checklist

### Security (Critical)
- No hardcoded secrets or API keys in source code
- Input validation at system boundaries (user input, external APIs)
- SQL injection prevention (parameterized queries only)
- Auth checks on every mutation endpoint (`@require_permission`)

### Architecture
- Module boundaries respected: no cross-module model imports
- Cross-module writes via EventBus, not direct DB
- Services.py is the public API — routes.py is thin HTTP layer
- Error handling uses WorkshopError hierarchy
- Wrapper/proxy/cache correctness: each method forwards to the wrapped instance (`delegate.get`), not back through a registry/session/global (else re-entry or recursion); every method the caller uses is actually forwarded

### Code Quality
- Functions < 50 lines, files < 500 lines
- No dead code or unused imports
- Naming: snake_case for Python, camelCase for TypeScript
- Type hints on public function signatures

### Code Smells — Fowler baseline (Standards axis)

Cannibalized from mattpocock/skills `code-review` (2026-07-12). Documented Workshop standards above take precedence; these twelve apply even where no documented standard exists. One line each: smell → fix.

- **Mysterious Name** — a name that doesn't reveal what the thing does → rename to reveal intent
- **Duplicated Code** — same logic in more than one place → extract a shared function
- **Feature Envy** — a method more interested in another module's data than its own → move it there
- **Data Clumps** — the same group of fields/params always traveling together → extract a type/object
- **Primitive Obsession** — domain concepts passed around as bare strings/ints → introduce a domain type
- **Repeated Switches** — the same switch/if-chain reappearing across the codebase → polymorphism or a dispatch table
- **Shotgun Surgery** — one logical change forcing edits across many files → consolidate the responsibility
- **Divergent Change** — one module changing for many unrelated reasons → split it by reason for change
- **Speculative Generality** — hooks/params for futures that never arrived → delete (YAGNI; cf. coding-discipline §1)
- **Message Chains** — `a.b().c().d()` reaching through objects → hide the delegate
- **Middle Man** — a class that only delegates → inline it, call the target directly
- **Refused Bequest** — a subclass ignoring/overriding most of its parent → prefer composition over inheritance

### Performance
- N+1 query prevention (use joinedload/selectinload)
- Pagination on list endpoints (PaginatedResponse)
- Redis cache for hot-path reads
- Embedding calls rate-limited (semaphore, max 4 concurrent)

### Testing
- Test data must be purged (hard delete) after tests
- Idempotent event handlers (same event twice = no side effects)

---

## Adversarial Review Mode

When invoked with "adversarial" or "challenge" intent, switch from checklist mode to adversarial stance. Cannibalized from openai/codex-plugin-cc adversarial-review prompt.

### Operating Stance
Default to skepticism. Assume the change can fail in subtle, high-cost, or user-visible ways until the evidence says otherwise. Do not give credit for good intent, partial fixes, or likely follow-up work. If something only works on the happy path, treat that as a real weakness.

### Attack Surface Priority
Prioritize failures that are expensive, dangerous, or hard to detect:
1. **Auth/trust boundaries**: permissions, tenant isolation, session handling
2. **Data integrity**: loss, corruption, duplication, irreversible state changes
3. **Rollback safety**: retries, partial failure, idempotency gaps
4. **Concurrency**: race conditions, ordering assumptions, stale state, re-entrancy
5. **Edge cases**: empty-state, null, timeout, degraded dependency behavior
6. **Schema drift**: migration hazards, compatibility regressions, version skew
7. **Observability gaps**: hidden failures, missing metrics, unactionable alerts

### Review Method
- Actively try to disprove the change
- Look for violated invariants, missing guards, unhandled failure paths
- Trace how bad inputs, retries, concurrent actions, or partial operations flow through code
- If user supplied focus area, weight it heavily, but still report other material issues
- Prefer one strong finding over several weak ones — do not dilute with filler

### Finding Bar
Report only material findings. Each must answer:
1. What can go wrong?
2. Why is this code path vulnerable?
3. What is the likely impact?
4. What concrete change would reduce the risk?

Do NOT include: style feedback, naming feedback, low-value cleanup, speculative concerns without evidence.

### Grounding Rules
- Every finding must be defensible from repository context or tool outputs
- Do not invent files, lines, code paths, or runtime behavior
- If a conclusion depends on inference, state that explicitly and keep confidence honest

---

## Structured Output Schema

When structured review output is requested, use the JSON schema at `schemas/review-output.schema.json`.

Key fields:
- **verdict**: `approve` | `needs-attention` | `critical-block`
- **summary**: Terse ship/no-ship assessment
- **findings[]**: severity, title, body, file, line_start, line_end, confidence, recommendation, category
- **next_steps[]**: Actionable follow-ups

Severity levels:
- **critical**: Security vulnerability, data loss risk, auth bypass
- **high**: Logic error, race condition, missing validation at trust boundary
- **medium**: Edge case unhandled, degraded path not tested, observability gap
- **low**: Minor improvement, defensive coding suggestion
