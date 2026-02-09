# Warehouse DBT Documented Sandbox Experiment — February 9, 2026

## Objective

Test whether providing **explicit column and join documentation** (schema.yml) in a warehouse+DBT sandbox narrows the performance gap compared to the baseline warehouse‑dbt sandbox.

This isolates the effect of `schema.yml` and documented dependencies, approximating how `ref()` + model documentation provide information density in real dbt projects.

## Hypotheses

Three possible outcomes:
1. **Gap narrows significantly** → the issue was information density, not architecture.
2. **Gap narrows slightly** → documentation helps but architecture still matters.
3. **Gap doesn’t move** → structural issues (split context, SQL indirection, casting) dominate.

## New Sandbox Variant

`warehouse-dbt-documented`
- Identical to `warehouse-dbt` in data, tasks, and validation.
- Adds a comprehensive `models/schema.yml` with column names and join hints.
- System prompt explicitly directs the model to read `models/schema.yml`.

### Documented Join Hints (examples)
- `stg_app_users.stripe_customer_id` ↔ `stg_stripe_invoices.customer_id`
- `stg_app_users.organization_id` ↔ `stg_app_organizations.organization_id`
- Invoice revenue uses `amount_paid` with `status = 'paid'`

## Method (Planned)

- Run the same harness and tasks as `warehouse-dbt`.
- Compare `warehouse-dbt` vs `warehouse-dbt-documented` across identical models.
- Keep max‑turns and model list constant.

Recommended command (example):
```bash
node --experimental-strip-types scripts/architecture-benchmark.ts \
  --sandbox=warehouse-dbt-documented \
  --model=claude-sonnet-4-20250514
```

## Files Added

- Sandbox: `sandboxes/warehouse-dbt-documented/`
- Schema docs: `sandboxes/warehouse-dbt-documented/models/schema.yml`

## Results

TBD — run pending.

## Notes

- The new sandbox is designed to be as close as possible to `warehouse-dbt`, differing only by documentation.
- This experiment is intended to strengthen the blog by isolating the effect of schema documentation.
