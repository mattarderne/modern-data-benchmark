# Modern Data Benchmark (Architecture Compare)

This repository benchmarks how well LLMs implement analytics tasks across four data architectures:

- **Typed TypeScript** (direct function execution)
- **DBT/SQL** (DuckDB execution)
- **Drizzle ORM** (SQLite + query builder)
- **Cube semantic layer** (measure definitions extracted to SQL)

The harness evaluates **ARPU**, **churn rate**, and **LTV** against the same synthetic dataset and compares pass rates across sandboxes.

## Status

Active. The modular benchmark harness, sandboxes, reports, and charts are in place. Legacy scaffolding and older benchmarks have been moved under `legacy/`.

## Project Structure

```
architecture-compare/
├── README.md
├── _summary.md
├── artifacts/                 # Reports + charts
├── data/                      # Synthetic JSON datasets
├── sandboxes/                 # Typed/DBT/Drizzle/Cube sandbox templates
├── scripts/                   # Current benchmark + smoke test scripts
└── legacy/                    # Archived scaffolds + older benchmarks
```

## Quick Start

1. (Optional) Regenerate data:
   ```bash
   node scripts/generate-data.js
   ```

2. Run the modular benchmark:
   ```bash
   # Anthropic
   ANTHROPIC_API_KEY=... node --experimental-strip-types scripts/modular-benchmark.ts --sandbox=all --model=claude-3-5-haiku-20241022

   # OpenRouter
   OPENROUTER_API_KEY=... node --experimental-strip-types scripts/modular-benchmark.ts --sandbox=all --model=qwen/qwen3-coder-next
   ```

3. Validate the harness without any API keys:
   ```bash
   node --experimental-strip-types scripts/smoke-test.ts
   ```

## How Validation Works (High Level)

- **Typed**: imports and executes the model’s TypeScript function.
- **DBT**: executes the model’s SQL in DuckDB.
- **Drizzle**: executes the model’s ORM query in SQLite.
- **Cube**: extracts the model’s measure SQL and executes it in DuckDB.

## Reports and Charts

See `artifacts/` for:
- `benchmark_report_detailed.docx` (full write‑up)
- `benchmark_matrix.png` and additional charts

## Legacy

Older scaffolding, plans, and legacy benchmarks are kept under `legacy/` for reference. They are not used by the current harness.
