# TODO

Validation depth
- Decide whether to upgrade syntax-only linting to typecheck/dbt compile (and keep parity across sandboxes).

Scoring
- Calibrate rubric weights and validate scoring on a small labeled set.

Robustness
- Add realistic drift scenarios (late Stripe invoices, missing stripe_customer_id mappings).

Harder queries / external benchmarks
- Adapt a subset of @matsonj's BIRD platinum set (https://github.com/matsonj/bird-bench) for each sandbox. 42 human-curated queries across 11 databases, complex joins, CTEs, NULL handling. Tests whether the architecture advantage holds as query difficulty scales.
- Review KramaBench (https://github.com/mitdbg/Kramabench) — MIT benchmark for end-to-end data science agents. 104 tasks across 6 domains, 1700+ tables, ~1.7GB raw data. Tests full pipeline: load, clean, transform, compute. Evaluates intermediate steps not just final answers. Interesting for multi-step analytical workflows that go beyond single-query metrics. Check if any tasks can be adapted to the sandbox format.
- Review Sphinx benchmarks (https://www.sphinx.ai/blog/sphinx-1-0-re-inventing-ai-for-data-science/) — references DABStep (Adyen/HuggingFace, real payments data) and KramaBench. Sphinx claims SOTA on both. Their SphinxBench (proprietary, 200+ scenarios) tests data intelligence, statistical reasoning, and workflow alignment. Worth tracking for methodology ideas even if the benchmark itself isn't open.

Richer data + schema architecture variants
- Integrate work from PR #55 (https://github.com/mattarderne/ai-research/pull/55): schema architecture evaluation framework with synthetic data generator across three business domains (Stripe billing, CRM sales pipeline, Support tickets). Generates identical data in three architectures:
  - Normalized (13 tables): canonical relational model
  - Star schema (8 tables): dimension + fact tables, classic BI pattern
  - One Big Table / OBT (3 tables): fully denormalized, pre-joined wide tables
  Hypothesis from PR: OBT should perform better for smaller LLMs due to reduced join reasoning. This adds a new axis to the current benchmark (architecture shape, not just typed vs SQL). Types already exist for CRM and Support domains. Data generator produces 500 customers, 3000 invoices, 300 accounts, 600 opportunities, 1500 tickets. Could run the existing benchmark harness against normalized vs star vs OBT variants to test whether denormalization helps agents the way types do.
- Plan was for 60 benchmark questions (20 per domain) at 4 complexity levels. These would significantly expand the current 3-task set.

New sandbox variants
- Create `warehouse-dbt-documented` sandbox: identical to `warehouse-dbt` but with comprehensive schema.yml documenting all columns, join keys, and relationships. Tests whether the gap is architecture or information density. See legacy/docs/INFORMATION_PARITY_ANALYSIS.md for the enhanced YAML spec (already written, just needs wiring).
- Add `app-drizzle-cube` sandbox using drizzle-cube (https://github.com/cliftonc/drizzle-cube), a Drizzle-native semantic layer. Since app-drizzle was the best-performing sandbox, this tests whether adding semantic abstractions (pre-defined measures, dimensions, joins) on top of the winning architecture helps or hurts agents. Also has MCP server support which is relevant if we move to tool-use agents.
- Add Cube sandbox to the architecture benchmark (currently only in the baseline benchmark). Compare Cube's declarative measure definitions against Drizzle's imperative queries and dbt's SQL. Tests whether pre-defined abstractions constrain or guide models.

Does more scaffolding help? (Dash review)
- Review Dash (https://github.com/agno-agi/dash), a self-learning data agent inspired by OpenAI's internal data agent. Uses 6 layers of context: table usage, human annotations, query patterns, institutional knowledge, learned error fixes, and runtime schema detection. The interesting question: Dash's approach is to pile on more context and scaffolding around SQL to make agents succeed. Our results suggest the opposite, that reducing indirection (typed codebase) works better than adding layers. Test whether Dash-style layered context on top of the warehouse-dbt sandbox closes the gap, or whether it's just more complexity for the same result. Could be a sandbox variant that adds annotations, query patterns, and error learnings to the dbt environment.

Context layer / context graph review and POC
- The blog promises to test whether "context layers" and "context graphs" are meaningfully distinct from what we already have. Need to actually do this:
  1. Literature review: collect the best articulations of what a context layer/graph claims to be. Sources: OpenAI data stack post, Databricks Unity Catalog, dbt semantic layer docs, any "context engineering" posts that define it concretely.
  2. Define what "meaningfully distinct" means: identify 2-3 concrete capabilities that a context graph claims and that a typed codebase or dbt+semantic layer doesn't already provide.
  3. Build POC sandboxes: implement the best-case version of a context graph for the benchmark tasks. If it can't be made concrete for ARPU/churn/LTV, that itself is a finding.
  4. Run through the same benchmark and compare. The bar is: does it measurably outperform app-drizzle?
- This is the existence proof test. If we can't build a simple instance that performs differently, the concept doesn't meaningfully exist yet.

Open research questions (from legacy plans, still unanswered)
- Declarative vs imperative: SQL/Cube (declarative) vs TypeScript/Drizzle (imperative). Baseline benchmark has some signal but architecture benchmark doesn't include Cube.
- Does the semantic layer help or hurt? Cube in the baseline scored mixed results (some models 3/3, some 0/3). drizzle-cube could clarify this by testing a semantic layer on top of the architecture that already works best.
- Edit/extend tasks: current benchmark is write-from-scratch. What about tasks that require modifying existing analytics code? The legacy scripts (edit-benchmark.ts, simple-edit-benchmark.ts) had this direction but were never run in the architecture benchmark.
- Multi-step workflows: current tasks are single-query. KramaBench-style pipelines (load → clean → transform → compute) would test whether the architecture advantage compounds or diminishes over multi-step reasoning.
