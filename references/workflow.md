# Workflow reference

## Base table schema

Clinical fields:

- `患者姓名`: text, required.
- `诊断`: text or single select.
- `负责医生`: Enterprise WeChat person field when available.
- `负责医生（手动填写）`: text fallback for doctors outside the directory.
- `动员开始日期`: date, required for formula prediction.
- `采集规则`: single select with exactly `5天方案` and `14天方案`.
- `采集类型`: single select, including mobilization collection and therapeutic apheresis.
- `计划采干时间`: date/time or text, depending on local table capabilities.
- `指定采干日期`: optional manual date.
- `采干日期（计算）`: formula/helper date.
- `采集状态`: single select such as planned, confirmed, completed, cancelled, recheck.
- `取消采干`: checkbox.
- `取消原因`: text; keep it visible in the management view.
- `血常规检测已完成`: checkbox.
- `CD34计数检测已完成`: checkbox.
- `MNC/Kg`, `CD34+细胞比例`, `CD34+细胞/Kg`: numeric/result fields completed on collection day.
- `采前3天检测日期`, `采前1天提醒日期`: formula/helper dates.
- `占用名额`: numeric/helper value used for capacity accounting.
- `当日已占名额`: daily aggregate/helper value.
- `容量提示`: formula text.
- `周末提示`: formula text.
- `容量审核状态`: select for pending/approved/needs adjustment.
- `备注`: final field; use for recheck reasons and operational notes.
- `患者数`: optional formula `1` helper. Keep hidden from the clinical view if it adds clutter; do not assume the native dashboard can aggregate it.

## Date and capacity logic

Use the following pseudocode when implementing formulas or documenting behavior:

```text
if 指定采干日期 exists:
    collection_date = 指定采干日期
else if 动员开始日期 and 采集规则 == 5天方案:
    collection_date = 动员开始日期 + 4 days
else if 动员开始日期 and 采集规则 == 14天方案:
    collection_date = 动员开始日期 + 13 days
else:
    collection_date = blank

pre3_date = collection_date - 3 days
pre1_date = collection_date - 1 day
```

Capacity is evaluated by calendar date and includes both collection types:

```text
daily_total = mobilization_collection_count + therapeutic_apheresis_count
capacity_warning = daily_total > 3
```

When `daily_total == 3`, show a full-capacity warning before adding another booking. When the platform cannot prevent the fourth entry, retain the warning and require a scheduling owner to review `容量审核状态`.

Weekend and holiday handling:

- Weekend dates should produce a visible warning.
- Public holidays and internal non-collection days require a maintained holiday/non-service-day list or manual review; do not pretend a weekend-only formula covers holidays.

## Recommended views

- `临床工作台`: patient name, diagnosis, responsible doctor, mobilization start date, collection rule, planned collection time, calculated collection date, collection status, note.
- `管理视图（公式）`: all helper fields, capacity calculations, reminder dates, cancellation reason, and audit fields.
- `今日采集`, `未来3天`, `日期未定／需复查`: filtered working lists.
- `日历`: collection dates and planned times.
- `当天治疗性单采登记`: same-day therapeutic apheresis entries.

Use phone-first ordering. Keep `备注` last unless local workflow needs it earlier.

## Dashboard layout

Use four cards in a 2x2 layout:

1. `今日采集患者数`: responsible doctor, patient name, row count.
2. `明日采集患者数`: patient name, row count.
3. `采前3天检测患者数`: patient name, row count.
4. `今日已预约采干数量`: indicator card using the capacity/booking source.

Use a restrained blue theme, medium row height, consistent card sizes, and enough contrast for mobile viewing. If the native aggregate label remains `记录数`, the card title supplies the clinical meaning; keep the underlying row-count calculation unchanged.

## Automation plan

Default rules to configure and verify:

- At collection-day minus 3 days, send the responsible doctor the testing list and the planned test date.
- At collection-day minus 1 day, send the responsible doctor the next-day collection list.
- On collection day, send or expose the today's list when supported.
- On cancellation or restoration, update the collection status and preserve the reason.

Document-dynamic reminders may appear in the table's activity stream without creating an Android lock-screen notification. Treat lock-screen delivery, native popup behavior, true calendar synchronization, and hard capacity blocking as separate capabilities that require live testing or an Enterprise WeChat app/API.

## Migration procedure

1. Identify the source tenant, target tenant, source owner, target owner, and intended recipients.
2. Preserve the source. Export a data backup and record screenshots or a field inventory.
3. Prefer a template/duplicate for structure. If unavailable or incomplete, create the target table and import the exported data.
4. Recheck every field type, formula, select option, view filter, color rule, dashboard source, chart title, automation recipient, and permission.
5. Recreate tenant-bound items: person fields, group recipients, document permissions, automation ownership, and calendar/app integrations.
6. Populate only synthetic test records first. Do not move production users to the target until the test passes.
7. Validate on desktop and mobile, then give users the target link and retain the source as rollback.

## 20-record pressure test

Use synthetic records distributed across these cases:

- today, tomorrow, 2–3 days, and more than 3 days;
- 5-day and 14-day schemes;
- manual collection date overriding the formula;
- weekend date;
- blank date/recheck;
- completed and cancelled cases;
- therapeutic apheresis mixed with mobilization collection;
- daily totals of 0, 1, 2, 3, and 4.

Score each item 0/1: date correctness, manual override, status/color visibility, responsible doctor visibility, capacity warning, cancellation handling, reminder rule, dashboard counts, desktop readability, mobile readability, and migration parity. The result should include the raw cases, score, evidence screenshot, and unresolved limitations. Never call the test passed if a reminder was only configured but not delivered or if the mobile view was not inspected.
