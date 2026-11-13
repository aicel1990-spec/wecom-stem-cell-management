---
name: wecom-stem-cell-management
description: Design, build, verify, and migrate an Enterprise WeChat smart-table system for hematology stem-cell mobilization and collection scheduling. Use when the task involves patient scheduling fields, collection-date formulas, capacity warnings, dashboards, reminders, or moving this workflow to another Enterprise WeChat tenant.
---

# Enterprise WeChat stem-cell management

Use this skill to turn a clinical scheduling requirement into a maintainable Enterprise WeChat smart table, dashboard, and automation workflow. Keep clinical decisions with the treating team; the table is a scheduling and reminder aid.

## Operating rules

- Confirm the target tenant, document owner, intended users, and whether the work is a new build, an update, or a migration.
- Use synthetic records for testing. Never copy real patient identifiers into a test table.
- Preserve the existing table before migration or broad edits. Prefer a template or duplicate plus export backup; do not delete the source.
- Treat date logic, capacity limits, cancellation state, and reminder timing as business rules that must be visible and testable.
- Verify platform behavior in the live tenant. Do not claim that a field, automation, popup, calendar sync, or lock-screen notification works merely because it was configured.
- When native Enterprise WeChat cannot enforce a rule, implement a visible warning and document the missing enforcement layer. A true hard block requires a suitable app, API, or administrator-supported entry point.
- After every UI mutation, re-read the live state and capture a screenshot of the final user-facing result.

## Preferred workflow

1. Convert the request into a field-and-rule specification. Resolve naming and date conventions before editing.
2. Build or inspect the base table using the schema in [references/workflow.md](references/workflow.md).
3. Implement date formulas, manual-date precedence, cancellation handling, and capacity warnings.
4. Create clinical and management views. Keep formula/helper fields out of the clinical view when they distract from care work.
5. Build the dashboard: today collection patients, tomorrow collection patients, pre-collection-day-3 testing patients, and today's booked collection count.
6. Configure reminders for the responsible doctor or approved recipient. Test document-dynamic delivery first; test group messages or app notifications only when the tenant supports them.
7. Run the synthetic pressure test and score the result with the checklist in the reference.
8. For migration, create the target copy, rebuild anything not carried over, test the target, and only then switch users to it.
9. Report verified behavior, unverified behavior, and remaining dependencies separately.

## Fixed domain rules for this workflow

- Count the mobilization start date as day 1.
- `5天方案` computes collection on start date + 4 calendar days.
- `14天方案` computes collection on start date + 13 calendar days.
- An explicitly entered collection date takes precedence over the computed date.
- Mobilization collections plus same-day therapeutic apheresis must total no more than 3 patients per day. In a native table this may be a warning rather than a hard block; never describe a warning as an enforced prohibition.
- Keep cancelled cases distinguishable and retain the cancellation reason. A cancelled case should not silently release a slot until the workflow explicitly defines that behavior.
- Default reminders are collection-day minus 3 days and minus 1 day at 08:00, subject to local operational confirmation.

## Naming and dashboard convention

Use clinical names that are immediately readable on a phone: `患者姓名`, `诊断`, `负责医生`, `动员开始日期`, `采集规则`, `计划采干时间`, `采干日期（计算）`, `采集状态`, `备注`.

If the native dashboard exposes an aggregate as `记录数` and does not allow an alias or a formula count field, preserve the underlying row-count calculation and rename the cards to `今日采集患者数`, `明日采集患者数`, and `采前3天检测患者数`. Do not replace `占用名额` with a patient-count field.

For detailed field schema, formulas, migration checks, and pressure-test cases, read [references/workflow.md](references/workflow.md).
