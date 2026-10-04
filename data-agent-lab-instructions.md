# Fabric Data Agent Workshop Labs

**Participant guide | October 2026**

Build a manufacturing operations data agent, improve its business understanding,
evaluate its answers, and extend it with a Lakehouse source.

This guide follows the Microsoft Fabric Data Agent Workshop's three labs.
Questions and actions are provided in execution order. Use the **Expected
behavior** and **If different** notes to assess your own results: wording,
generated queries, visual formats, and latency can vary between runs.

**Starting point:** `SetupDataAgentJumpstart` has already run successfully.
Do not reinstall the Jumpstart or rerun setup just to begin this guide.

**Formats:** [Markdown](data-agent-lab-instructions.md) |
[Download the PDF](data-agent-lab-instructions.pdf)

## Contents

- [Before you start](#before-you-start)
- [Lab 1: Creating and optimizing data agents](#lab-1-creating-and-optimizing-data-agents)
- [Lab 2: Programmatic evaluation of data agents](#lab-2-programmatic-evaluation-of-data-agents)
- [Lab 3: Adding multiple data sources](#lab-3-adding-multiple-data-sources)
- [Workshop completion checklist](#workshop-completion-checklist)
- [Troubleshooting](#troubleshooting)
- [Sources and license](#sources-and-license)

## Before you start

### Confirm the prepared environment

1. Sign in to [Microsoft Fabric](https://app.fabric.microsoft.com/) with the
   account provided for the workshop, then open your assigned workspace.
2. Open the `getting-started-data-agents` folder.
3. Confirm that both a semantic model and a report exist for
   `ManufacturingOps` and `ManufacturingOpsAIReady`.
4. Confirm that the following six notebooks are present.

| Notebook | Role in the workshop |
| --- | --- |
| `SetupDataAgentJumpstart` | Already completed: imports and configures the populated models and reports. |
| `RefreshSemanticModel` | Optional maintenance; not required simply because the workshop is starting. |
| `JudgeCalibration` | Calibrates and registers the evaluation judge in Lab 2. |
| `EvaluateDataAgent` | Evaluates your AI-ready agent in Lab 2. |
| `BuildOpsRefData` | Builds the Lakehouse and its SQL-facing objects in Lab 3. |
| `CreateMultiSourceDataAgent` | Configures and publishes your multi-source agent in Lab 3. |

**Expected behavior:** two semantic models, two reports, and six notebooks are
available. The models contain cached data. Notebook presence alone does not
prove that setup completed: open the two reports and confirm that their visuals
load data rather than displaying connection errors.

Both reports open on a blank **Page 1**. Use the page tabs at the bottom:

- `ManufacturingOps` report: open the **info** page. It shows the data-as-of
  card.
- `ManufacturingOpsAIReady` report: open the **Verified Answer** page. It shows
  scrap rate by manufacturer.

**If different:** ask the facilitator to finish or repair setup. Do not create
duplicate models, change capacity, or rerun setup over other participants'
work. A successful refresh is not a prerequisite when the populated imported
models already work.

**Facilitator note — known setup failure:** in the tested notebook version, the
first `SetupDataAgentJumpstart` run can stop at the DataFolder parameter update
with **HTTP 403**. The cause is one request whose `Authorization` header contains
a literal placeholder instead of the token. In that cell, set the header to
`"Bearer " + power_bi_token` (the notebook's existing Power BI token variable),
or ask notebook Copilot to fix the error, then rerun. The second run completes
both parameter updates and the final data checks. No permission change is
needed.

### Access and optional features

The facilitator normally prepares the paid Fabric capacity, Power BI Pro
license, workspace Contributor access, and required Data Agent/Copilot tenant
settings. If **New item > Data agent** is unavailable, stop and ask the
facilitator to check these prerequisites.

Model editing and notebook execution require the appropriate permissions.
Modeling Copilot, Preview runtime, Code Interpreter, and Microsoft 365 Copilot
integration can have additional tenant, licensing, or regional requirements.
Their absence is a feature/access blocker, not evidence of an incorrect
analytical answer. Record any section you cannot execute.

### Use your own agent names

Choose a short, unique identifier, such as your initials followed by a number.
This guide uses `AB01` as an example. Replace it consistently with your own
identifier; do not use angle brackets in the item names.

| Item | Example name |
| --- | --- |
| Baseline agent | `MfgOps_DA_AB01` |
| AI-ready agent | `MfgOps_DA_AIReady_AB01` |
| Multi-source agent | `MfgOps_DA_AIReady_AB01_MultiSource` |

In a shared workspace, model changes affect everyone using those models.
Coordinate the model-editing exercises with the facilitator. Do not rename or
overwrite somebody else's agent. For a resumed lab, reuse only your own items
and inspect their configuration before continuing.

### How to read a result

For each question, review the paraphrased request, generated DAX or SQL, query
execution result, and final answer. Expand the run-step control beneath the
answer; its label and number of steps can vary.

An acceptable numerical answer uses the intended measure, entity, grouping, and
period, and agrees with the query output. Fluent prose alone is not sufficient.
A query error is not an empty dataset; an empty dataset is not numeric zero.

Use **Clear chat** between independent tests. Keep the same conversation only
for exercises explicitly identified as follow-ups.

Each exercise below shows the result observed when this guide was rehearsed
end to end on a fresh deployment:

- **Reference result** gives the values from the workshop sample data. Your
  wording, layout and timing will vary; the measure, period and numbers should
  match.
- **Known behavior** marks an answer that is deliberately imperfect on the
  baseline agent. It is the lesson of that exercise, not a broken lab. Run the
  **Recovery prompt** that follows it to see the correct result.

### Establish data-as-of dates before relative questions

**Why this matters:** "last month" can mean the previous calendar month relative
to today, or the previous month relative to the latest loaded fact date. Those
are different questions when the imported data is historical. A refresh
timestamp tells you when a model was processed; it does not tell you the date
of the latest business event.

After connecting each agent to its model in Lab 1, ask:

```text
What are the earliest and latest dates with production records in this model?
Use ProductionLog dates, not the last date in the calendar table. Return one
row with both dates. Do not apply a default time window.
```

1. Expand the run step and verify that the query reads `ProductionLog`
   dates, rather than only the calendar table.
2. Write down the minimum and maximum dates for each model separately.
3. Do not assume that the baseline and AI-ready models have identical data.
4. Do not assume that a maximum date proves every earlier day is complete.
   Inspect monthly counts and consult the facilitator if completeness matters.

**Reference result (workshop sample data):** the models are imported snapshots,
so every deployment returns the same coverage:

| Source | First production date | Latest production date |
| --- | --- | --- |
| `ManufacturingOps` (baseline) | 1 June 2024 | **6 July 2026** |
| `ManufacturingOpsAIReady` | 1 June 2024 | **8 August 2026** |
| `OpsRefData` Lakehouse (Lab 3) | built relative to the day you run `BuildOpsRefData` | runs up to that day |

Neither model contains data up to today. This is why "last month" in the
baseline returns **June 2026**, and why Lab 3 needs explicit dates when it
combines the models with the Lakehouse.

If you also ask for "the number of production records", the agent may return
the count on each boundary date rather than the all-history total. That is
an ambiguity in the question, not a data problem. Use the DAX check below for
an exact count.

The check is **optional**. You do not need it for the later questions to work.
It tells you whether an unexpected period comes from the agent's
interpretation or from data that is not loaded. If the answer is unavailable
or ambiguous, open the semantic model's **DAX query view**, create a query, and
run this read-only check. If that view is not available to you, ask the
facilitator to run it.

```dax
EVALUATE
ROW(
    "First production date", MIN(ProductionLog[Date]),
    "Latest production date", MAX(ProductionLog[Date]),
    "Production rows", COUNTROWS(ProductionLog)
)
```

For numerical comparisons between agents, choose the same explicit date range
within both models' coverage. Use the relative questions as a separate test of
time interpretation, rather than silently changing the question to fit the
data. A model ending partway through a month should not be presented as a
complete month.

**Workshop time convention:** when no period is supplied, the AI-ready agent
will use the latest 30 calendar days **including** the latest production date:
from that date minus 29 days through that date. Explicit user dates take
precedence. Ambiguous relative phrases should be clarified or accompanied by
the exact resolved dates. In production, the business owner must decide whether
such phrases are anchored to today or to a data snapshot.

**Facilitator cue:** "Before trusting the number, we check which dates it
describes. The agent cannot report a month that has not been loaded. We want
to distinguish data freshness from query correctness."

## Lab 1: Creating and optimizing data agents

**Goal:** build a baseline agent, diagnose ambiguous questions, configure an
AI-ready model and agent, and explore runtime and analysis tools.

### Step 1: Creating your first data agent

1. In the prepared folder, select **New item > Data agent**. Use the item search
   if needed.
2. Name the agent `MfgOps_DA_AB01`, using your own identifier.
3. In **Explorer**, select **Add Data > Data source**.
4. In the OneLake catalog, select the `ManufacturingOps` **semantic model** in
   your workspace. Select **Add**.
5. Expand the model and **select all its tables** for this baseline exercise.
   Selecting the model root shows a generic warning about selecting many
   tables (more than 25); this model has 13, so continue. Confirm that each
   checkbox is selected; adding a source is not the same as selecting its
   tables.
6. Run the data-as-of check above. Record this model's production coverage
   (reference: 1 June 2024 to 6 July 2026).
7. Clear chat and ask the introduction question.

```text
I am new to this agent and the data. Tell me more about it and how to use it.
```

**Expected behavior:** the agent describes the connected data and suggests
questions. Treat these as suggestions, not a guarantee that every suggested
question is in scope or will execute successfully.

Ask the following one at a time. Clear chat between these independent tests.

```text
Show me the production quantity for the last month.
```

**Expected behavior:** a production total with an identifiable period. Inspect
whether DAX anchors "last month" to the current date or the latest loaded date.
A historically anchored result can be internally consistent without answering
the calendar-relative question you intended.

**Reference result:** **45,078 units for June 2026**. The DAX takes
`MAX('Date'[Date])` in the model, not today's date, as the reference, so "last
month" becomes June. The final sentence may not name the month: expand the run
step to see it.

**If different:** ask the agent to state the exact dates. Repeat with an
explicit month inside the recorded coverage. Do not describe "no records" as
zero production, and do not silently relabel an older month as last month.

```text
List the top 5 products by total sales.
```

**Expected behavior:** up to five products ranked by a sales measure, with the
period and currency stated. The baseline exercise includes sales; the
operations-only agent created later will deliberately decline sales questions.

**Reference result:** five products ranked by the governed sales measure over
**all available history**. The all-history scope is visible in the paraphrase
and DAX but is often missing from the final sentence.

**If different:** inspect the ranking, measure, and period. Product list prices
are not total sales. If the query returns fewer than five products, distinguish
insufficient data from an omitted result.

```text
Give me a pivot table with product, total sales, quantity, and inventory.
```

**Expected behavior:** a table at a declared grain. Check whether "quantity"
means units sold or produced, and whether inventory represents one snapshot or
an aggregation over multiple snapshots.

**Known behavior:** a 20-product table with sales, sales quantity and
inventory, but **no date filter**. Inventory is summed across every daily
snapshot, so it is not a stock level. A text table rather than an interactive
pivot is normal.

**Recovery prompt** (clear chat first):

```text
For June 2026, give me a table with product, total sales and sales quantity.
Add inventory on hand from the single latest inventory snapshot in June 2026
only; do not sum inventory across dates. State the snapshot date.
```

**Reference result:** June sales with inventory from **one** snapshot (29 June
2026), labelled as such.

#### Multi-part question and conversational context

Clear chat, then ask:

```text
Which products have inventory below the reorder quantity? How often does that happen?
```

**Expected behavior:** identification of low-stock products plus an explanation
of "how often": for example, product-plant-date snapshots below reorder point.
Frequency must not be confused with shortfall units.

**Reference result:** the governed `Inventory Risk SKU Count` measure returns,
per product, the number of **product-plant-date snapshots** below reorder
quantity, plus a total. The wording may loosely call these "dates" or
"records"; they are snapshot occurrences.

Without clearing chat, ask:

```text
For the top three products by frequency, show me the monthly trend as a bar chart.
```

**Expected behavior:** the follow-up refers to the same three products and the
same definition of frequency. The agent may use one query or several; a fixed
number of run steps is not required.

**Known behavior:** the follow-up keeps three products but **changes the
metric** to distinct days instead of snapshot occurrences. The query returns
many months, but the chart can show only one month. This is the lesson:
conversational context does not guarantee the same definition.

**Recovery prompt** (same conversation):

```text
Use the same frequency measure as the previous answer: the number of
product-plant-date inventory snapshots below reorder quantity. Take the top
three products by that total, breaking ties by product name. Return a table
with product, month and that count for every month with data.
```

**Reference result:** a product-by-month table for FlowGuard 10 Control Valve
(41), TorqueMax 30 Induction Motor (39) and AquaFlow 100 Centrifugal Pump (38,
first alphabetically among four products tied at 38). The monthly counts add
up to each product's total from the first answer. Prefer the table: chart
rendering of multi-month series was unreliable in testing.

**If different:** confirm the three product names, date range, and frequency
definition explicitly, then retry. If the previous answer did not establish a
valid ranking, there is no trustworthy "top three" to carry forward.

#### Domain-specific KPI question

Clear chat and ask:

```text
For the bottom 20% of products by revenue, do we run into inventory issues? Why?
```

**Expected behavior:** inspect how the agent defines bottom 20%, resolves ties,
and connects the selected products to inventory measures. It can describe
observed low stock or shortfalls.

**Known behavior:** the DAX comparison is valid, but the answer **invents
causes** (for example "lower demand" or "conservative stocking policies") that
no data supports, and treats inventory summed across snapshots as a stock
position.

**Recovery prompt** (clear chat first):

```text
List the products in the bottom 20% by total revenue across all history, with
their revenue and their count of inventory snapshots below reorder quantity.
Report only what the data shows. If the data cannot explain why, say so instead
of suggesting causes.
```

**Reference result:** **four** low-revenue products (Pump Seal Kit, Pump
Impeller, SenseLine Temperature Sensor, Turbine Bearing Set) with revenue and
risk counts, and an explicit statement that the data does not explain the
cause.

**If different:** request the revenue period, selected products, and inventory
criterion. "Why" does not authorize a causal explanation absent supporting data.
Distinguish measured associations from hypotheses.

**Facilitator cue:** "A plausible answer is the start of the review, not the
end. We inspect the measure, time window, and business definition."

### Step 2: Improving and optimizing data agent responses

Stay in the baseline agent. Clear chat before each question, expand its run
steps, and compare the paraphrased question, generated DAX, and final answer.
These are diagnostic exercises: the baseline is not required to fail in a
particular way.

#### Q1: Scrap rate and implicit dates

```text
What is our scrap %?
```

**Expected behavior:** look for `[Scrap Rate %]` and identify the selected time
window. The baseline may use all available history, a recent window, or ask for
clarification. A correct measure with an unstated period is still ambiguous.

**Reference result:** the governed `[Scrap Rate %]` over **all available
history**, with that scope stated. This one is correct on the baseline.

**Next action:** retain the measure and make the default period explicit in
the AI-ready model's instructions.

#### Q2: A named KPI without a governed definition

```text
What is our OEE?
```

**Expected behavior:** the baseline may infer Overall Equipment Effectiveness,
construct an ad hoc calculation, or ask for clarification. On a resumed model,
an OEE measure may already exist. Inspect rather than assume.

**Known behavior:** the baseline builds an **ad hoc OEE formula** (about
**92.6%** in testing) that leaves out the Performance factor and does not state
its date. It looks plausible but is not the business OEE.

**Next action:** compare any generated formula with the business definition.
The AI-ready model contains `[OEE %]`. Reuse that governed measure instead of
assuming that an improvised formula is authoritative.

#### Q3: Day-shift yield versus daily yield

```text
What is our day production yield for the last six months?
```

**Expected behavior:** check whether "day" means **excluding the Night shift**
or simply grouping all-shift yield by calendar day. The baseline's
`[prd_yld_day]` implements the former, but its name is not self-explanatory.

**Known behavior:** the baseline returns **all-shift yield by calendar day**,
not the day-shift measure. That is the ambiguity this exercise demonstrates.

**Next action:** the AI-ready model uses `[Day Yield Pct]` with a description.
Put business meaning in model metadata and Prep data for AI, not solely in
DAX comments. If the six-month period extends outside coverage, disclose that
rather than silently shortening it.

#### Q4: Internal acronyms

```text
What were the TP sales last week?
```

**Expected behavior:** "TP" may be unresolved, misinterpreted, or correctly
inferred. In this workshop it means the **Pumps and Turbines** product
categories. The cryptic baseline measure `[sls_amt_x]` expresses that sales
filter.

**Known behavior:** the baseline searches for product names containing "TP",
finds nothing, gets **BLANK** and reports it as **0 sales**. Two lessons: an
undefined acronym is guessed, and "no rows" is not zero.

**Next action:** inspect the actual category filter and dates. A clarifying
question is preferable to a confident answer for the wrong meaning. Later,
the AI-ready operations-only agent must decline this **sales** request even
though TP itself is a recognized term.

#### Q5: Business meaning of "machine"

```text
Show me the distribution of scrap rate by machine.
```

**Expected behavior:** the agent can interpret "machine" as equipment,
production line, or manufacturer. Inspect the grouping column, not just the
chart label.

**Known behavior:** the baseline groups by **line asset**. The text and DAX list
eight assets, but the chart may show only one category. Check the table in
the run step rather than the chart.

**Next action:** for this workshop's AI-ready exercise, "machine" is a
business alias for **equipment manufacturer**, mapped to
`Lines[Manufacturer]`. This is a workshop-specific convention, not a universal
meaning of machine. Use `Lines[Manufacturer]` because production rows reach
it through an **active** relationship; the `Assets` relationships to
`ProductionLog` and `Lines` are **inactive**, so grouping a production KPI by
an `Assets` column does not filter it. The bundled verified answer also uses
`Lines[Manufacturer]`.

**Facilitator cue:** "We are not teaching the agent a universal definition of
machine. We are documenting the meaning our business expects."

### Step 3: Configuring and optimizing the semantic model and data agent

#### Define scope

The next agent answers manufacturing operations questions: production,
inventory, assets, plants, lines, scrap, yield, and OEE. It does not answer
sales, customer, vendor, or purchasing questions. Lab 3 will explicitly add
product sales through a different source.

Do not remove sales tables from the underlying shared model to enforce this
exercise. Focus the AI schema and agent table selection, and add scope
instructions. These controls are not a replacement for permissions or
row-/object-level security.

#### Inspect the AI-ready model

1. Return to the workspace and open the `ManufacturingOpsAIReady` **semantic
   model**, not its report.
2. Switch to **Editing** mode when authorized.
3. Inspect names and descriptions. Examples include `Customers[Customer Name]`,
   `Assets[Asset Name]`, `[Day Yield Pct]`, `[Scrap Rate %]`, and `[OEE %]`.
4. Confirm that descriptions explain business meaning and usage, rather than
   merely repeating the field name.
5. Inspect `[sls_amt_x]` as a metadata example: the description explains TP,
   although sales will remain outside this agent's scope.

**Expected behavior:** clearer metadata and governed measures are available.
Not every object is necessarily renamed; existing downstream dependencies can
make descriptions preferable to renaming.

#### Configure Prep data for AI

1. In the semantic model, select **Prep data for AI**.
2. Open **Simplify the data schema** / **AI data schema**.
3. Include the operations tables `Assets`, `Business Measures`, `Date`,
   `Inventory`, `Lines`, `Plants`, `ProductionLog`, and `Products`. Within those
   tables, include relevant operational measures and their dependencies.
   Exclude unrelated sales and purchasing measures from the focused AI schema.
4. Review **Verified answers**. Look for the scrap-rate-by-machine example.
   Inspect its underlying grouping, measure, and filters.
5. Confirm that the verified answer groups by `Lines[Manufacturer]`. This
   matches the rule below. Do not switch it to `Assets[Manufacturer]`: the
   Assets relationships are inactive, so that column does not filter
   production measures.
6. Open **Add AI instructions**. The bundled text conflicts with this
   workshop in two places: `RQX = [Quality %] measure` (RQX is scrap rate)
   and `For all questions related to "machines" use Assets[Manufacturer]
   column` (that column does not filter production measures). **Select all
   the existing text and replace it** with the block below. The block keeps
   the bundled rules that remain valid, including the `CONTAINSSTRING` rule
   for names. It is the instruction set used in the rehearsal that produced
   the reference results in Step 4 (lightly reformatted for reading).

```text
Use governed measures rather than inventing alternative KPI formulas.

For production questions with no period, use the 30 calendar days ending on
the latest ProductionLog[Date], inclusive: latest date minus 29 days through
latest date. Do not use the maximum calendar-table date as the data-as-of date.

Explicit user dates take precedence. For ambiguous relative phrases such as
"last month", "last week", or "this year", establish the reference date and
calendar convention. Return the exact start and end dates used. Disclose
incomplete coverage; do not substitute an older period without saying so.

User-specified durations are explicit periods and ALWAYS override the default:
six weeks is 42 calendar days, ending on the latest production date unless the
user specifies another reference. Use end minus 41 days through end, inclusive,
never 30 days for a six-week question.

For every production KPI query, include the actual latest ProductionLog date as
a Data As Of result alongside the requested start and end dates. A requested
end date later than Data As Of does not mean those later dates contain data.

Day production yield / DPY means yield excluding the Night shift.
Use [Day Yield Pct], not all-shift yield grouped by day. Day production yield
is a day-shift-only KPI, not a request for a daily time series. When asked to
break a KPI down by line for a period, evaluate the governed measure over the
entire period and return one row per line. Add daily grouping only when
explicitly requested. Do not average daily percentages.

RQX means [Scrap Rate %], not [Quality %].
OEE means [OEE %]. In this workshop, "reliability" is an alias for OEE.
TP refers to Products[Category] values Pumps and Turbines, not Motors.

In this workshop, machine means equipment manufacturer. For a breakdown by
machine, group ONLY by Lines[Manufacturer]; do not add asset name, line or
location unless the user explicitly asks for that detail. For individual
equipment, use Assets[Asset Name]. Use Lines[Manufacturer] through the active
Lines-to-ProductionLog relationship: the Assets-to-ProductionLog and
Assets-to-Lines relationships are inactive, so grouping a production KPI by
Assets attributes does not filter that KPI.

Aggregate at the requested grain before returning results. Do not derive a
full-period total or average from a truncated detail result.

Unless the user requests missing groups, exclude rows with BLANK production
quantity and BLANK KPI. Preserve actual numeric zeros. Never replace BLANK
with zero, including for visualization.

Never calculate sales revenue from Products[Price].
If a query fails, report an execution problem. Do not claim that data is
missing unless a successful coverage query supports that statement.

For named-entity columns (e.g., names, places, organizations), use
CONTAINSSTRING for partial-text matching by default. Use exact-match filters
only when the user explicitly requests a specific entity.
```

7. Search the saved instructions for `Quality %` and `Assets[Manufacturer]`.
   Neither should appear as a rule (only "not [Quality %]" remains).
8. Save/apply the model changes and close the Prep data for AI pane.

**Expected behavior:** the AI schema, instructions, and verified-answer
metadata express a consistent business interpretation.

**Important:** semantic-model query-generation rules belong in **Prep data for
AI**. Agent-level instructions control orchestration, scope, and presentation;
they are not a substitute for model-level DAX guidance.

Verified answers guide DAX generation using their prompts and visual metadata.
A data agent does not necessarily return the original Power BI visual. Do not
promise an identical chart or guaranteed latency improvement.

**Facilitator cue:** "We place each rule where it is consumed: business
calculations in the model, routing and response behavior in the agent."

### Step 4: Testing the optimized semantic model

#### Create and configure the AI-ready agent

1. Create a new data agent named `MfgOps_DA_AIReady_AB01`.
2. Add the `ManufacturingOpsAIReady` semantic model from the same workspace.
3. Explicitly select these eight tables in the agent's Explorer:
   `Assets`, `Business Measures`, `Date`, `Inventory`, `Lines`, `Plants`,
   `ProductionLog`, and `Products`. Click **one checkbox at a time** and wait
   a second for each to save: rapid clicks can be lost when you leave the
   page.
4. Switch to another Explorer tab and back, expand the source, and confirm the
   eight checkboxes are still selected. An attached source with no selected
   tables cannot answer data questions (in an earlier test, this alone
   dropped the Lab 2 score from 3/3 to 1/3).
5. Open the agent's **Instructions** area and enter the following.

```markdown
# Scope
Answer manufacturing operations questions using the configured AI-ready
semantic model: production, inventory, assets, plants, lines, operational
performance, trends, and KPIs.

# Audience and tone
Help plant managers, operations leaders, production supervisors, inventory
planners, analysts, and executives. Be concise, professional, and clear.
Explain manufacturing terminology when necessary.

# Guidelines
- Decline sales, customer, vendor, purchasing, and purchase-order questions
  with: "This question is out of scope for this agent. Please ask a
  manufacturing operations-related question."
- Respect the model's governed KPI definitions and time-period rules.
- When no period is supplied, use the latest 30 days of production data.
- State the requested start/end dates and the actual latest production date
  separately. If the requested end is later than the data-as-of date,
  explicitly disclose that records are only available through the data-as-of
  date. Do not imply full coverage through today.
- For YoY comparisons, always show both current and prior-year start/end dates.
- Clarify ambiguous plant, line, product, or date requests.
- State units, filters, and assumptions. Give the answer first, then
  supporting evidence.
- Return the requested aggregation grain. For a period broken down by line,
  request one full-period row per line, not a daily time series. Add daily
  granularity only when explicitly requested.
- Do not derive complete totals or averages from truncated detail. Requery at
  the requested aggregate grain.
- Do not invent explanations, missing values, or unsupported causal claims.
  Distinguish query failure from no matching records and from numeric zero.
- Location means plant location when a location grouping or filter is
  requested; do not add a location grouping otherwise.

# Common terminology
- Machine means equipment manufacturer in this workshop. For by-machine
  questions request manufacturer-only grouping, not individual assets or
  lines. Individual equipment means asset name.
- Day production yield / DPY means the governed yield excluding Night shift,
  evaluated over the requested period. It does not mean daily yield.
- RQX: scrap rate.
- OEE: Overall Equipment Effectiveness; workshop alias "reliability".
- DPMP: defects per million parts.
- YoY / YOY: year over year. MTD: month to date. MOM: month over month.
- TP: Pumps and Turbines product categories.

# Period precedence and missing data
Explicit user dates or durations ALWAYS override the default. Apply the
latest-30-day default ONLY when no period is specified. Last six weeks means
42 calendar days ending on the latest production date, inclusive; never
replace six weeks with 30 days.
Never convert a blank or unavailable KPI into zero, including for a chart.
```

6. Close/save the instructions pane, reopen it, and confirm the text persisted.
7. Run the production data-as-of check for this model. Record its dates
   (reference: 1 June 2024 to **8 August 2026**).
8. Clear chat before each independent question below.

**Why two layers of instructions?** In the rehearsal, model instructions alone
did not fix the day-yield, machine and "this year" questions: the agent's
orchestrator rephrased the question (for example into a daily series or an
asset grouping) before the model rules applied. Agent instructions control
that rephrasing; model instructions control the DAX.

#### Question 1: Default period

```text
What is our scrap rate?
```

**Expected behavior:** `[Scrap Rate %]` over 30 days including the latest
production date, with the resolved dates in the answer.

**Reference result:** **2.36%**, 10 July to 8 August 2026, with scrap units and
production quantity that reconcile.

**If different:** check the model's saved instructions, selected tables, and
DAX filter. Retry with explicit start/end dates. Do not report success merely
because the response repeats "30 days."

#### Question 2: Explicit period overrides the default

```text
What is the OEE this year?
```

**Expected behavior:** `[OEE %]`, not an ad hoc replacement; the default
30-day period is overridden. The answer should clarify the year/reference date
or state it explicitly, and disclose whether it is year-to-date rather than a
complete year.

**Reference result:** **87.18%**, 1 January to 8 August 2026, with "data
available through 8 August 2026" stated. Without the data-as-of guidance, the
first attempt implied coverage through today.

**If different:** ask for OEE for an explicit year and end date within coverage.
Historical sample data does not become current-year data just because the
question says "this year."

#### Question 3: Day production yield

```text
What is our day production yield for the last six weeks? Break it down by lines.
```

**Expected behavior:** `[Day Yield Pct]`, excluding Night shift, grouped by
line. The answer identifies the six-week boundaries; "six weeks" can otherwise
mean a rolling 42-day interval or six calendar weeks.

**Reference result:** **eight lines, one value each** (about 97.4% to 97.7%)
for **28 June to 8 August 2026** (42 days). Earlier instruction versions
produced 30 days, or a 336-row daily series that hit the 200-row limit; the
period-precedence and grain rules fix both.

**If different:** request 42 days ending on a stated production date. Confirm
the measure and line grouping in DAX rather than trusting the answer's title.

#### Question 4: Out-of-scope request

```text
What were the TP sales last week?
```

**Expected behavior:** a refusal, not a sales total. Recognizing TP does not
make sales part of this agent's scope.

**Reference result:** the configured out-of-scope message, without running a
query.

**If different:** check that you are in the AI-ready operations-only agent,
not the baseline or multi-source agent. Check scope instructions and the
focused schema. Never use scope prompts as a substitute for data security.

#### Question 5: Multiple abbreviations and year-over-year logic

```text
What's the YOY TP reliability?
```

**Expected behavior:** TP means Pumps and Turbines; reliability means the
governed OEE measure. YoY requires two comparable periods. Asking for the
intended period is acceptable.

**Reference result:** governed OEE for Pumps and Turbines over the latest 30
days and the same window a year earlier: **86.83% vs 86.76% (+0.07 pp)**, with
both date ranges stated.

**If different:** expand the run steps. A DAX execution error is not evidence
that prior-year data is missing. First establish coverage for both periods.
Then use this diagnostic follow-up:

```text
First establish the available production dates for Pumps and Turbines.
Ask me to choose comparable current and prior-year periods within coverage.
Then return [OEE %] for each period separately and the difference in
percentage points. Do not interpret a failed query as missing data.
```

If necessary, ask the two period questions separately. Compare results only
after both queries succeed; do not calculate a YoY percentage from a missing
or zero denominator.

#### Question 6: Machine/manufacturer interpretation

```text
Show me the distribution of scrap rate by machine.
```

**Expected behavior:** manufacturer-level scrap rates consistent with the
saved business rule. Inspect whether the query uses `Lines[Manufacturer]`,
and whether verified-answer metadata changes that grouping or its filters.

**Reference result:** six manufacturers for 10 July to 8 August 2026: Fluke
2.60%, GE 2.41%, Mazak 2.38%, NI 2.36%, Marsilli 2.28%, Siemens 2.22%, with
matching chart categories. If you see the same rate for every manufacturer,
the query grouped by an `Assets` column, which does not filter production.

**If different:** inspect conflicting metadata and ask explicitly:

```text
Show [Scrap Rate %] grouped by Lines[Manufacturer] for the latest 30 days of
production data. State the dates. I mean equipment manufacturers, not
individual assets or production lines.
```

Different equivalent-looking columns are not automatically interchangeable:
check relationships, grain, and totals before accepting the result.

**Facilitator cue:** "The improvement is clearer, inspectable behavior. We
still verify the actual query; instructions are guidance, not a guarantee."

### Data agent runtime

The runtime controls the agent's orchestration and query-generation
components. Standard and Preview can produce different queries and timings.
Preview includes advanced DAX generation; it is not a promise that every query
will be faster or error-free.

1. In the AI-ready agent's runtime setting, select **Standard**.
2. Clear chat and ask the following original comparison question.

```text
For each plant, return the smallest ordered set of line-and-shift combinations
that reaches at least 80% of that plant's downtime. Include the first
combination that crosses 80%. Show plant, line, shift, downtime minutes, rank
within plant, share of plant downtime, cumulative share, and cumulative share
before the row. Show the result as a table.
```

3. Record success/failure, total elapsed time, period, returned rows, and DAX.
4. Switch to **Preview**, clear chat, and repeat the identical question.
5. Check correctness before comparing speed. Within each plant, the rows must
   be ordered by descending downtime, use that plant's full downtime total,
   and stop at the first row whose cumulative share reaches or exceeds 80%.
6. Inspect tie handling. If downtime values tie, repeat both runtime tests with
   the same added instruction: `Break ties by line name, then shift name, in
   ascending order.` Keep the time range identical.
7. For a fuller comparison, perform ten independent runs per runtime with
   unchanged configuration and data. Alternate runtimes where practical.

**Expected behavior:** two sets of measured outcomes, not a prescribed
performance percentage. Record unsuccessful runs separately; report both
failure rate and the median latency of successful runs. Do not silently drop
failures or call a single faster response a benchmark.

**Reference result (10 runs per runtime, F16 capacity):**

| Runtime | Queries executed | Fully correct answers | Median time of executed runs |
| --- | --- | --- | --- |
| Standard | 5 of 10 (4 backend timeouts/errors, 1 not submitted) | 0 | 83 s |
| Preview | 10 of 10 | 5 | 29.5 s |

Preview was faster and more reliable on this question, but **neither runtime
was error-free**. Present it that way: a measured improvement, not a
guarantee. For reference, in the correct result Riverside's 80% set has 9 line-shift
rows covering 4,679 downtime minutes, and Rheinland's has 9 rows covering
4,118 minutes.
Ten runs per runtime take about 30 minutes; in a timed workshop, run one or
two each and compare with this table.

**If different:** if a runtime is unavailable, record that limitation. If a
query fails, inspect its error and retry a simpler ranking before returning to
the cumulative-threshold question.

**Facilitator cue:** "We compare correctness and speed together. A faster
incorrect answer is not an improvement."

### AI-assisted modeling changes

This exercise modifies the **baseline** model. Perform it after the baseline
question tests. In a shared workshop, the facilitator should coordinate who
applies changes. A resumed, already-edited model will not reproduce the same
starting behavior as a clean deployment.

1. Open the `ManufacturingOps` semantic model, not
   `ManufacturingOpsAIReady`.
2. Switch to **Editing** mode and open **Copilot**.
3. Ask for proposed names without applying changes.

```text
Identify all the name columns in this model. Using the table context and
sample values, propose more descriptive names that clearly reflect the
business meaning. Return a table with: Current Name, Proposed Name.
```

4. If a tool approval appears, review its scope and select **Allow** only for
   the intended operation.
5. Review the proposed names with the facilitator. Check dependencies in
   measures, relationships, reports, and verified answers.
6. Once approved, ask:

```text
Approved, make the changes.
```

7. Reopen the affected tables to confirm the names actually changed. Open the
   baseline report and check affected visuals. An assistant's completion
   message is not sufficient evidence that all changes were applied.

**Expected behavior:** proposed changes followed by persisted, reviewed
changes. Copilot can produce partial results or require follow-up.

**Reference result:** Copilot's first proposal renamed **43 columns**,
including descriptive fields that are not names. Narrow it before approving:

```text
Limit the proposal to columns that hold the name of an entity (customer,
product, vendor, plant, line, asset and similar). Do not rename IDs, codes,
dates, numeric fields or descriptive attributes.
```

The narrowed list had **16** entity-name columns (for example `custName` to
`Customer Name`, `Products[Name]` to `Product Name`). After approval all 16
persisted, source mappings and relationships were preserved, and dependent
measures updated automatically. Reopen the tables to confirm.

#### Optional: Add descriptions

Ask:

```text
This semantic model will be used by a Fabric data agent to answer
manufacturing operations questions. Add descriptions based on the following
guidelines.

DO: Add a description to every visible table, column, and measure. Keep
descriptions concise and front-load the key meaning within the first 200
characters. Front-load preferred usage, disambiguation, and units. Make
implicit knowledge explicit. Do not restate the field name or add DAX logic.
Sample values if necessary to learn the domain and context. Include expected
grain where useful. For calculation groups, describe the items and their use.

DON'T: Generate descriptions purely from AI without business context.
AI-only descriptions tend to restate what the model structure already shows.
Always validate descriptions with the user or a domain expert. Do not
contradict descriptions across related fields.
```

Review the proposals before saying:

```text
Update the descriptions.
```

**Expected behavior:** useful, persisted descriptions, reviewed against the
data and business meaning. The 200-character target is a concise-writing
convention here, not a claim that every product surface has the same hard
retrieval limit.

**Reference result:** Copilot proposed descriptions and flagged **27** objects
as uncertain ("for review"). Apply only the confident ones (109 in testing):

```text
Apply only the descriptions you are confident about. Skip every item you
flagged for review.
```

Then check that no saved description contains "for review" text.

Finish by reviewing business-friendly names, descriptions, synonyms, sensible
hierarchies, model relationships, security, the AI schema, verified answers,
and AI instructions. Do not introduce new relationships or security changes
solely to complete this checklist.

### Code Interpreter

1. Return to your **AI-ready agent**.
2. Select **Tools > Add tools > Code interpreter**.
3. Clear chat and ask:

```text
Show me a pivot table of products vs. the last six months by reliability. Put
products on rows and months in columns.
```

4. Check the OEE measure, six-month boundaries, and coverage. Keep this
   conversation open and ask:

```text
Show me a heatmap.
```

5. Expand the analysis steps. Inspect the Python code, input data, and output.

**Expected behavior:** a heatmap based on the preceding product/month result,
not a new and unrelated query. Missing cells should remain distinguishable
from zero. Inspect how percentages and color scales are represented.

**Reference result:**

- Pivot question: governed OEE by product for **March to August 2026**, with
  data available through 8 August. **August is partial** (1–8 August) even if
  its label says "August"; say so when presenting. The result may be a list
  rather than a grid.
- Heatmap: the run steps show a `code_interpreter.execute` step and a
  products-by-months heatmap image.

**If different:** if no Python tool ran, do not present the result as a Code
Interpreter demonstration. Confirm the tool is enabled and ask:
`Use Code Interpreter to render the preceding table as a heatmap. Preserve
missing cells and label the OEE percentage scale.`

#### Optional: Detrending and FFT

Clear chat and ask:

```text
Detrend the daily scrap rate for Pump Impeller for the last two months, run
FFT on it, and identify any dominant modes.
```

**Expected behavior:** data retrieval followed by Python analysis. Check the
product, daily measure, time range, observation count, missing dates, detrending
method, and FFT frequency units.

**Reference result:** two steps (query, then Python). The agent used the last
two **complete** months, June and July 2026, stated the data-as-of date,
**interpolated** days without production, removed a linear trend and found a
dominant cycle of about **15 days**. Point out the interpolation: it is an
assumption, and the suggested operational causes are hypotheses.

**If different:** an empty or irregular series may not support the requested
analysis. Request coverage and sampling checks before accepting any result.
Do not silently fill gaps with zero. A spectral peak is a candidate pattern,
not proof of a manufacturing root cause.

**Facilitator cue:** "Python extends what we can calculate, but it does not
remove the need to check the data and the statistical assumptions."

## Lab 2: Programmatic evaluation of data agents

**Goal:** calibrate an LLM judge against human labels, evaluate your
operations-only AI-ready agent, and inspect the results in MLflow.

Run both notebooks **inside Fabric**. The workshop's Responses API evaluation
workflow is not an instruction to run the notebooks locally. Allow extra time
for environment startup, package installation, and service calls.

### Step 1: LLM-as-Judge calibration

1. Open the public [judge calibration workbook](https://github.com/microsoft/fabric-data-agent-workshop/blob/v1.0.4/eval/judge_calibration_labeling.xlsx)
   for review.
2. Inspect `calibration_development` and `calibration_holdout`. Development
   examples are for rubric refinement; holdout examples are for an independent
   acceptance decision. The lab provides labels, so you do not need to invent
   them.
3. Open `JudgeCalibration` in your Fabric workspace.
4. Inspect its **Configuration** cell. Keep the supplied model, rubric, data
   reference, and acceptance thresholds unless the facilitator instructs
   otherwise. Note the judge model and `JUDGE_REGISTRY_EXPERIMENT`.
5. Select **Run all**, and wait for execution to finish. If package installation
   requests a session restart, follow the notebook instruction before
   continuing.
6. Review the development/holdout results and the notebook's registration
   decision. Do not register a failed judge by lowering thresholds to make
   the exercise pass.
7. Open the MLflow run linked by the notebook. Inspect the rubric, model,
   human-label agreement metrics, data reference, and registration tags.

**Expected behavior:** the notebook creates or reuses the evaluation Lakehouse,
loads the labeled workbook, evaluates the candidate judge, logs evidence, and
registers a champion **only if its acceptance rules are met**.

**Reference result:** champion judge registered (model `gpt-5.1`):
development agreement **97.1%**; holdout agreement **94.4%**, Cohen's kappa
**0.886**. If you run the notebook as a scheduled job, its status may show
**Cancelled**: the last cell stops the session on purpose. Check the MLflow
run and registration instead.

**If different:** distinguish authentication, model availability, capacity,
package, or storage errors from poor agreement with human labels. Stop before
agent evaluation if no approved judge is available.

**Facilitator cue:** "Before asking an AI to grade another AI, we check that
its grading agrees with the human standard."

### Step 2: Evaluate your AI-ready agent

1. Open `EvaluateDataAgent`.
2. Find **Select the Data Agent**. Set the following to the **exact display
   name of your own agent**, not the illustrative `..._SAP` default.

```python
DATA_AGENT_NAME = "MfgOps_DA_AIReady_AB01"
```

3. In **Configuration**, confirm `GT_MODEL = "ManufacturingOpsAIReady"`.
   Leave `REFRESH = "No"` unless you intentionally need a refresh. Refreshing
   during a comparison can change the answer key.
4. Confirm that the agent is the operations-only agent from Lab 1. The
   workshop evaluation includes a sales refusal check; it is not the same
   policy as the multi-source agent in Lab 3.
5. Review the evaluation workbook path used by your notebook. The source
   version accompanying this guide uses `eval_set_L400.xlsx` and selects three
   cases when `TEST_MODE = True`. Inspect `TEST_CASE_IDS`; do not assume the
   workbook itself contains only three rows.
6. Confirm that the eight source tables remain selected and the model and
   agent instructions are saved.
7. Select **Run all**. The workflow generates numerical ground truth from the
   semantic model and evaluates policy questions against expected behavior.
8. Review the overall score and **Per-question review** under the results
   section. Open the MLflow run.

**Expected behavior:** inspectable per-question answers, DAX/run steps,
ground truth or policy expectations, judge reasoning, and metrics. Completion
of the notebook is not equivalent to 100% answer accuracy.

**Reference result:** overall, factual and behavioral accuracy **1.0 (3/3)**,
0 infrastructure errors, with the judge loaded from the registry. In an
earlier run where the agent's tables were not selected, the score was 1/3:
only the refusal passed.

**If different:** when a refusal succeeds but data questions fail, first inspect
source selection, data access, query errors, and the period used. A policy
refusal can succeed without successful data retrieval.

Correct one issue at a time, label the next run using the notebook's `STAGE`
field, and rerun. Preserve the first run for comparison. The supplied source
uses the agent's `sandbox` stage for evaluation: do not assume this proves the
published endpoint is identical.

**Checkpoint:** do not move on with an unexplained failed factual answer.
Record the failed case and its cause, or ask the facilitator for help. A
three-case evaluation is a learning exercise, not production certification.

## Lab 3: Adding multiple data sources

**Goal:** add Lakehouse data for downtime reasons and product sales, inspect
the resulting routing and queries, then publish and consume the agent.

This explicitly broadens scope: **product sales become allowed through
OpsRefData**. Customer-level data, vendors, and purchasing remain outside scope.

### Step 1: Build the Lakehouse source

1. Open `BuildOpsRefData`.
2. Use the supported Python 3.11 or 3.12 notebook runtime for the supplied
   notebook. Do not convert it to an unrelated Spark runtime solely for this
   exercise.
3. In a shared workshop, ask the facilitator whether `OpsRefData` is already
   prepared. The notebook rebuilds shared data and SQL objects; participants
   should not all rebuild it concurrently.
4. If it needs building, select **Run all** and wait for the final result.
5. Return to the workspace and open the `OpsRefData` Lakehouse.
6. Switch to its **SQL analytics endpoint**.
7. Expand the `fda` schema and confirm the `Downtime_Reasons` and `Sales_Orders`
   views, plus the `Downtime_By_Line_Type` table-valued function.
8. Open **New SQL query** in that endpoint and run:

```sql
SELECT 'Downtime_Reasons' AS [Object], COUNT(*) AS [Rows]
FROM [fda].[Downtime_Reasons]
UNION ALL
SELECT 'Sales_Orders', COUNT(*)
FROM [fda].[Sales_Orders];

SELECT *
FROM [fda].[Downtime_By_Line_Type]('Assembly')
ORDER BY [Downtime Minutes] DESC;
```

9. Record source coverage:

```sql
SELECT MIN([Date]) AS [First Date], MAX([Date]) AS [Latest Date]
FROM [fda].[Downtime_Reasons];

SELECT MIN([Order Date]) AS [First Sales Month],
       MAX([Order Date]) AS [Latest Sales Month]
FROM [fda].[Sales_Orders];
```

**Expected behavior:** both views contain queryable data, and the function
returns the Assembly downtime-reason breakdown. The function is all-history
and has no date parameter; use the view for date-filtered analysis.

**Important — dates differ from the models:** `BuildOpsRefData` generates
Lakehouse data **up to the day you run it** (in the rehearsal, downtime
reasons ran to the build day and sales to the first day of that month). The
semantic models stop on 6 July / 8 August 2026. So "latest 30 days" means a
different period in each source. Keep this in mind for Step 3.

**If different:** SQL endpoint metadata can lag table creation. Inspect the
notebook's first failed cell, wait for endpoint metadata refresh, and rerun
the affected step under the facilitator's guidance. Do not treat missing SQL
objects as successfully configured sources.

**Important grain distinction:** `Sales_Orders` contains monthly product/plant
facts; `Order Date` is the first day of the sales month. It is not a daily
transaction feed. A latest month key does not prove that the month is complete.

### Step 2: Create the multi-source agent with the SDK

1. Open `CreateMultiSourceDataAgent` in the **same workspace** as your
   AI-ready agent and `OpsRefData`.
2. Inspect **Step 1 - Parameters** and update:

```python
BASE_DATA_AGENT_NAME = "MfgOps_DA_AIReady_AB01"
MULTI_SOURCE_AGENT_NAME = f"{BASE_DATA_AGENT_NAME}_MultiSource"
SEMANTIC_MODEL_NAME = "ManufacturingOpsAIReady"
LAKEHOUSE_NAME = "OpsRefData"
PUBLISH_CHANGES = True
```

3. Before running, review the notebook's publication behavior and the target
   name. `PUBLISH_CHANGES = True` publishes in Step 9. If you are not ready to
   publish, set it to `False` and stop before the published MCP test in Step 10.
4. Run cells from **Step 0** through **Step 8** in order. Allow package
   installation and source discovery to finish.
5. Read the output: the base agent must exist; the eight model tables and
   three Lakehouse objects must be selected.
6. Open the newly created `..._MultiSource` agent in Fabric and inspect its
   sources, agent instructions, and Lakehouse **Setup** guidance/examples.

**Expected behavior:** your base agent remains available, and the separate
multi-source agent has both sources configured.

**Important:** inspect your notebook version. The supplied source creates or
reuses the derived agent and applies its own source selection, instructions,
and examples. It is not a guarantee of a complete clone of every custom base
setting, runtime, or tool. Recheck any customizations you need.

The notebook broadens sales scope and replaces agent instructions. Do not keep
the operations-only "all sales are out of scope" rule on this multi-source
agent. Conversely, do not broaden the Lab 1 agent.

#### Inspect and clarify time handling

The supplied notebook contains snapshot-relative time defaults, including
"this year" based on the selected source's latest date. Inspect those rules:
they can differ from the calendar-relative meaning a user expects.

For the cross-source exercises, use **explicit matching periods**. When
comparing daily production with monthly sales, use calendar months whose
coverage is confirmed in both sources. Do not claim that simply taking the
earlier maximum date proves complete coverage, or prorate a monthly sales row
into daily sales.

Never join daily downtime detail directly to monthly sales detail. Aggregate
each side independently to the same plant/month grain first. Keep missing
groups visible when the question requires them, and do not infer causality
from a correlation.

### Step 3: Publish and test the MCP endpoint

1. Return to **Step 9 - Review and publish**. Review the staging source names
   and saved settings.
2. If publication is intended and authorized, run Step 9 with
   `PUBLISH_CHANGES = True`. Otherwise, publish your agent manually after
   reviewing it and only then proceed.
3. Run **Step 10 - Query the published agent through MCP**. Keep the notebook
   session open until you finish the later post-publication checks.
4. Inspect each test result, not just `is_error`. A successful MCP transport
   response can still contain a refusal, incomplete answer, or query failure.

The supplied tests cover the following:

| Test | Question / intended route |
| --- | --- |
| Semantic model | "What was the scrap rate in the latest 30 days?" - governed model measure. |
| Lakehouse | "Which product had the highest sales?" - actual revenue from `Sales_Orders`. |
| Both sources | "Which line had the most downtime minutes, and what were its leading downtime reasons, using the same latest 30-day period?" - align periods and distinguish totals from reasons. |
| Function | "Across all available history, what is the downtime-reason breakdown for Assembly lines?" - the all-history function is appropriate. |

**Expected behavior:** a published endpoint can be initialized and queried
using the signed-in identity, and each answer meets its analytical criteria.
Keep authentication tokens out of screenshots, notebook exports, and public
repositories.

**Reference result:** the cell completes and its `is_error` assertion passes.
Reading the answers:

| Test | What you will see |
| --- | --- |
| Semantic model | **2.36%**, 10 July to 8 August 2026 — same as the Lab 1 agent. Correct. |
| Lakehouse | The top product by revenue over the Lakehouse's **own** latest 30 days (HelioGen 2000 in the rehearsal). Correct, but note the period differs from the model's. |
| Both sources | **Known behavior — wrong.** The downtime total comes from the model (ending 8 August) while the reasons come from the Lakehouse's recent weeks (September–October in the rehearsal). The answer still claims a single period. `is_error` is `False`. |
| Function | All-history Assembly breakdown. Correct. |

This is the key Lab 3 lesson: **a transport success is not an answer
check.** Rerun the combined question with explicit dates, in the agent's
Test chat or by replacing that entry in `MCP_TEST_QUESTIONS`:

```text
For 2026-07-10 through 2026-08-08 inclusive, which line had the most downtime
minutes, and what were its leading downtime reasons in that same period?
Apply exactly these dates to both sources and state them.
```

**Reference result:** Line A1 - Pump Assembly, **1,588** downtime minutes;
reasons Equipment Failure 713, Changeover 322, Planned Maintenance 207,
Material Shortage 190, Operator Error 156 (sum 1,588).

**If different:** check whether the agent was published, whether your identity
has source access, and whether the endpoint has become available. A saved
draft does not update the published endpoint. Do not change tenant permissions
just to hide a failed test.

### Step 4: Creator Assistant - Build agent with AI

Use this exercise on the **multi-source agent** and its Lakehouse source.
The assistant can propose configuration changes, but you must verify each
requested change was persisted.

1. Select **Build agent with AI**.
2. Enter the original multi-change request:

```text
Can you update the agent instructions and the OpsRefData data source
instructions, and add a few-shot example so that when a user asks about the
turbomachinery category, the query filters for Pumps and Turbines, but not
Motors? Please sample the values first to confirm.
```

3. Review the sampled category values and all proposed changes.
4. When satisfied, say:

```text
Apply these changes.
```

5. Inspect **three separate locations** before continuing:

| Location | Required persisted content |
| --- | --- |
| Agent instructions | Turbomachinery means Pumps and Turbines, excluding Motors; sales routes to OpsRefData. |
| OpsRefData source instructions | SQL filters `[Category] IN ('Pumps', 'Turbines')` for turbomachinery. |
| OpsRefData example queries | A saved question/query pair demonstrates that category filter. |

**Expected behavior:** all three requested changes exist in their correct
locations. A confirmation message is not evidence of all three writes.

**Known behavior:** the assistant sampled the category values (Motors, Pumps,
Turbines) correctly, but the first "Apply" saved **only the agent
instructions**, then offered to do the rest. The source instructions and the
example query each needed **a separate request**:

```text
Now update the OpsRefData data source instructions with the turbomachinery
rule.
```

```text
Now add the turbomachinery few-shot example query to the OpsRefData source.
```

After the third request, all three locations contained the change.

**If different:** ask for the missing change individually, or use the source's
**Setup** editor to add it. Preserve existing routing, grain, and date rules.
For a simple category-only example, add the following question and SQL pair.

```text
Across all available sales history, what are revenue and units sold for
turbomachinery, excluding Motors?
```

```sql
SELECT SUM([Revenue]) AS [Revenue],
       SUM([Units Sold]) AS [Units Sold]
FROM [fda].[Sales_Orders]
WHERE [Category] IN ('Pumps', 'Turbines');
```

This example explicitly requests all available history. Do not reuse its
unfiltered dates for questions asking for a particular month or year.

6. Ask the assistant:

```text
Test this change and verify that it works as expected.
```

7. Independently ask the same all-history question in the agent's **Test**
   conversation. Clear chat first.
8. Run the SQL above directly in the `OpsRefData` SQL analytics endpoint.
   Compare revenue and units, source, categories, and period.
9. Inspect a category breakdown if totals disagree:

```sql
SELECT [Category], SUM([Revenue]) AS [Revenue],
       SUM([Units Sold]) AS [Units Sold]
FROM [fda].[Sales_Orders]
GROUP BY [Category]
ORDER BY [Category];
```

**Expected behavior:** the turbomachinery result equals the Pumps plus
Turbines totals for the same period and excludes Motors. Do not copy a
numerical total from a different environment as the answer key.

**Reference result:** revenue **419,704,600** and **7,118** units, filter
`[Category] IN ('Pumps','Turbines')`, identical in the assistant's own test,
the Test chat, the published agent, the MCP endpoint and Microsoft 365
Copilot (the assistant's category check showed Motors at 33,728,200 / 9,026,
correctly excluded). Your totals can differ if your Lakehouse was built on
another date; what must match is the agent's answer and your direct SQL
query.

To return from **Build agent with AI** to the normal chat, select **Test
data agent** in the toolbar (it may be under **More** (…) on a narrow window).

**Facilitator cue:** "We asked for three changes, so we inspect three saved
settings. Then we compare the agent's answer with a direct source query."

### Step 5: Description and final publication

1. Ask Creator Assistant:

```text
Generate a concise description of what this data agent does. Focus on the
specific business domain and the questions it answers. Avoid generic,
reusable descriptions. Make it specific to this data agent's unique purpose
and capabilities. Do not use bullet points and do not mention table names,
column names, datasets, schemas, or other technical implementation details.
Use clear language that accurately describes its value.
```

2. Review and copy the description. It should describe both manufacturing
   operations and product sales without claiming unsupported functionality.
3. Select **Publish**. The description field may be **prefilled with a
   summary of your recent changes** rather than a capabilities description:
   replace it with the description from step 1.
4. If you will use Microsoft 365 Copilot, append the original output-handling
   guidance:

```text
The output from the data agent should be delivered as-is, without summarizing,
rephrasing, or adding extra interpretation or insight.
```

5. Publish the latest changes. Keep **Also publish to the Agent Store in
   Microsoft 365 Copilot** off unless you are completing the next step and
   have the required authorization.
6. Return to the still-open notebook session and rerun the MCP query cell
   with the same turbomachinery question used in Step 4, replacing or adding
   an entry in `MCP_TEST_QUESTIONS`. Do not rerun the configuration-writing
   cells: they can overwrite the Creator Assistant changes.
7. Compare the published answer with the direct SQL and draft answer, using
   identical scope and dates. When finished, run the notebook's final session
   stop cell.

For step 6, replace only the `MCP_TEST_QUESTIONS` list in notebook Step 10
with the following, then rerun that Step 10 cell:

```python
MCP_TEST_QUESTIONS = [
    (
        "turbomachinery",
        "Across all available sales history, what are revenue and units sold "
        "for turbomachinery, excluding Motors?",
    ),
]
```

**Expected behavior:** the published agent reflects the latest configuration.
The description guidance can reduce Microsoft 365 Copilot rephrasing; it
does not guarantee verbatim output.

**Reference result:** the published Test view and the MCP endpoint both
returned 419,704,600 / 7,118 with Motors excluded, `is_error = False`.

### Step 6: Consume the agent in Microsoft 365 Copilot

Complete this step when the workshop account has the required Microsoft 365
entitlement and Copilot extensibility enabled. Use the **same account and
tenant** as Fabric. Publishing to the Agent Store is an additional action,
not a prerequisite for the preceding Fabric and MCP exercises.

1. In your agent's **Publish** dialog, review the description and enable
   **Also publish to the Agent Store in Microsoft 365 Copilot**.
2. Publish and open Microsoft 365 Copilot with the same account.
3. Open **Agents** / **Agent Store**, locate your agent by its exact name,
   and open it. Navigation labels can vary.
4. If it has not appeared, refresh the agent list or expand navigation and
   allow propagation time. If still unavailable, ask the facilitator to
   check entitlement and extensibility settings.
5. Start a new conversation directly with the agent, or select it using `@`.
6. Ask the **same** turbomachinery question used for SQL, draft, and MCP.
7. Compare revenue, units, categories, and period. Do not require identical
   prose; Microsoft 365 Copilot has its own orchestration layer.

**Expected behavior:** the agent is discoverable for the authorized user and
returns a source-grounded answer consistent with the equivalent direct query.

**Reference result:** the agent appeared in the Agent Store search about
**10 minutes** after publishing. The store shows the name **cut to 30
characters** (`MfgOps_DA_AIReady_AB01_MultiSo`) and a generic description.
Select **Open**. The answer took about 40 seconds: 419,704,600 revenue and
7,118 units, Pumps and Turbines only.

**If different:** confirm agent identity, account, tenant, publication state,
underlying source access, and time range. A missing Agent Store entry is not
a failed DAX or SQL calculation.

Do not share the agent externally as part of this exercise. Agent Store
publication does not grant recipients access to underlying sources.
Microsoft 365 consumption can process results under Microsoft 365's data
handling terms and outside Fabric's compliance boundary; follow your
organization's policy before enabling it.

## Workshop completion checklist

| Area | Evidence to retain in your own approved environment |
| --- | --- |
| Baseline agent | Selected source tables, data-as-of dates, and inspected responses. |
| AI-ready agent | Eight selected tables, consistent model/agent guidance, governed KPI queries, and correct sales refusal. |
| Relative questions | Declared reference dates and exact resolved periods; no silent substitution of missing months. |
| Runtime | Recorded correctness and timing; unsuccessful runs remain visible. |
| Modeling | Approved names/descriptions persisted, with dependent content checked. |
| Code Interpreter | Python execution and chart inputs inspected, or feature blocker recorded. |
| Evaluation | Approved judge plus per-question agent results and MLflow evidence. |
| Multi-source | Both sources selected, routing inspected, source grain and periods understood. |
| Creator Assistant | All three requested configuration changes persisted and directly checked. |
| Publication | Matching question/scope checked in draft and published MCP. |
| Microsoft 365 Copilot | Matching factual result, or a clearly recorded access/feature blocker. |

You have completed the learning path when you can explain both successful
answers and unresolved failures. Do not mark an unexecuted optional feature as
passed.

## Troubleshooting

| Symptom | First action |
| --- | --- |
| Setup notebook fails with HTTP 403 at the DataFolder parameter step | Fix that cell's `Authorization` header to `"Bearer " + power_bi_token`, then rerun (see Before you start). |
| Agent loses table selections | Click one checkbox at a time, wait for each to save, then reopen the Explorer to confirm. |
| Every manufacturer shows the same scrap rate | The query grouped by an `Assets` column (inactive relationship). Use `Lines[Manufacturer]`. |
| Six-week question uses 30 days, or returns a daily series | Check the period-precedence and grain rules in both model and agent instructions. |
| Combined downtime question mixes periods | Expected with relative periods: the Lakehouse runs to today, the models do not. Ask with explicit dates. |
| Stuck in **Build agent with AI** mode | Select **Test data agent** in the toolbar (or under **More** (…)). |
| Agent not found in Microsoft 365 Copilot | Wait about 10 minutes after publishing and search part of the name; the store shows only 30 characters. |
| "Last month" returns an unexpected month | Inspect the DAX reference date and the fact table's coverage; ask with explicit dates. |
| Baseline and AI-ready totals differ | Align measure, filters, date range, and underlying data coverage before comparing quality. |
| AI-ready agent cannot answer data questions | Confirm that tables are selected, the source is accessible, and the query executed. |
| RQX gives quality rather than scrap | Remove the conflicting model instruction; confirm the query uses `[Scrap Rate %]`. |
| Day yield includes Night shift | Inspect `[Day Yield Pct]` selection and model-level guidance. |
| YoY answer says "no data" after a failed query | Treat it as a query failure; establish coverage with a successful independent query. |
| Notebook cannot find an agent | Replace the sample name with the exact agent display name in that workspace. |
| Notebook evaluation succeeds but score is low | Read each failed case and its ground truth; notebook execution is not answer correctness. |
| Creator Assistant says done, but behavior is unchanged | Inspect agent instructions, source instructions, and few-shot examples separately. |
| Sales behavior changes after Lab 3 | Expected only on the multi-source agent, which routes product sales to OpsRefData. |
| SQL objects are absent after building the Lakehouse | Inspect the failed build/metadata step; wait for endpoint synchronization before retrying. |
| Published result differs from draft | Republish the reviewed configuration and use the same question, source, and period. |
| Copilot wording differs from Fabric | Compare facts and filters; the extra orchestrator can rephrase results. |

## Sources and license

Based on the MIT-licensed
[Microsoft Fabric Data Agent Workshop](https://github.com/microsoft/fabric-data-agent-workshop).
Lab and notebook reference:
[commit 136bcf196a7f935fb030bd9fe81ad0e133d2fbf3](https://github.com/microsoft/fabric-data-agent-workshop/tree/136bcf196a7f935fb030bd9fe81ad0e133d2fbf3).
Check your deployed notebook cells if their version differs.

Product guidance:

- [Semantic model best practices for data agents](https://learn.microsoft.com/en-us/fabric/data-science/semantic-model-best-practices)
- [Data agent configurations](https://learn.microsoft.com/en-us/fabric/data-science/data-agent-configurations)
- [Data agent query-generation best practices](https://learn.microsoft.com/en-us/fabric/data-science/data-agent-configuration-best-practices)
- [Consume a data agent from Microsoft 365 Copilot](https://learn.microsoft.com/en-us/fabric/data-science/data-agent-microsoft-365-copilot)

This is a community workshop guide, not a product warranty or a guarantee of
deterministic AI responses. Microsoft product names remain their owners'
trademarks. This guide does not imply Microsoft sponsorship.

### MIT license

Copyright (c) Microsoft Corporation.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
