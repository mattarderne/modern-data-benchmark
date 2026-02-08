# Linting Experiment (Option B) — February 8, 2026

## Objective

Evaluate whether adding linting (TypeScript typecheck + SQLFluff + DuckDB EXPLAIN) improves model outcomes, and whether lint can function as a useful helper instead of just a gate.

## Setup

Model
- `claude-sonnet-4-20250514`

Tasks (same across all runs)
- Active User ARPU
- Organization Churn Rate
- Average Org LTV

Sandboxes
- `app-typed` (TypeScript functions over app + Stripe JSON)
- `app-drizzle` (Drizzle ORM over unified SQLite)
- `warehouse-dbt` (DBT-style SQL in DuckDB)

Run parameters
- `--max-turns=10`
- All sandboxes (`--sandbox=all`)

## Lint Variants

1) No lint (baseline)
- Lint disabled with `--no-lint`.

2) Lint gated (older syntax-only lint)
- TypeScript: `node --experimental-strip-types --check`
- DBT: DuckDB `EXPLAIN`
- If lint failed, task failed immediately.

3) Lint helper (Option B)
- TypeScript: `tsc --noEmit`
- DBT: SQLFluff lint + DuckDB `EXPLAIN`
- If lint fails, the model gets a short fix attempt (up to 3 turns), then validation proceeds regardless.

Schema-only lint mode
- Implemented via `--lint-mode=schema` (not used in these runs).
- DBT lint only reports missing table/column errors (ignores SQLFluff style and non-schema errors).
- TypeScript lint only reports missing property/module errors (ignores other type errors).

## Results (Sonnet only)

Plain text table (mean passes over runs):

Variant | n | Mean total (out of 9) | app-typed | app-drizzle | warehouse-dbt | Mean cost/run | Cost per pass
No lint | 2 | 2.0 | 1.0 | 0.5 | 0.5 | $1.2706 | $0.6353
Lint gated (syntax-only) | 2 | 2.5 | 1.5 | 0.5 | 0.5 | $1.0970 | $0.4388
Lint helper (Option B) | 2 | 3.5 | 1.0 | 2.0 | 0.5 | $1.4476 | $0.4136

Per-run totals:
- No lint: 2, 2
- Lint gated: 2, 3
- Lint helper: 3, 4

## What Lint Actually Caught

TypeScript
- Missing or invalid properties (e.g., `Property 'customer' does not exist on type ...`)
- Basic type errors (string vs number, date math), depending on configuration

DBT
- SQLFluff: style and some syntax errors (line length, aliasing, formatting)
- DuckDB EXPLAIN: missing columns/tables and invalid identifiers

## Interpretation

- The main lift in the lint-helper runs came from **app-drizzle**, not from DBT or app-typed.
- The helper loop **adds extra edit budget**, which likely drives much of the improvement.
- Lint errors were often unrelated to the task’s numeric correctness. Some tasks passed validation even when lint failed.
- SQLFluff produced **style noise**, which can distract smaller models and does not correlate with correctness.

## Notes on Extra Turns

Lint helper provides extra turns only when lint fails:
- Each lint failure triggers up to 3 extra turns.
- In the two lint-helper runs, lint failed on 4–5 tasks, which allowed up to 12–15 extra turns total.

## Limitations

- Small n (2 runs per condition) makes results noisy.
- Lint helper introduces extra turns, so comparisons are not purely about lint signal.
- SQLFluff style rules add noise for small models and do not always align with correctness.

## Recommended Next Tests

1) Control run with extra turns but no lint feedback
- Gives the same extra budget to isolate the effect of lint feedback.

2) Schema-only lint for DBT
- Use `--lint-mode=schema` to reduce noise and focus on missing-column errors.

3) Lint after each `write_file`
- Moves feedback earlier in the loop and may improve fixability.

## Reproduction Commands

No lint (baseline)
```bash
node --experimental-strip-types scripts/architecture-benchmark.ts \
  --sandbox=all \
  --model=claude-sonnet-4-20250514 \
  --max-turns=10 \
  --no-lint
```

Lint helper (Option B)
```bash
node --experimental-strip-types scripts/architecture-benchmark.ts \
  --sandbox=all \
  --model=claude-sonnet-4-20250514 \
  --max-turns=10
```

Schema-only lint mode
```bash
node --experimental-strip-types scripts/architecture-benchmark.ts \
  --sandbox=all \
  --model=claude-sonnet-4-20250514 \
  --max-turns=10 \
  --lint-mode=schema
```
