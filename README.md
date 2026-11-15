# Redo Tripwire

**When a one-line redeploy quietly reprocesses every document, should deploy stop and ask?**
A rejection-sampling Monte Carlo prototype that picks the default cost threshold for a "confirm before full reprocess" gate.

- **Live demo:** https://shriyashekatkar25-byte.github.io/redo-tripwire/
- **Gallery:** https://shriyashekatkar25-byte.github.io/apm-lab/ (APM Lab)
- By Shriya Shekatkar · UCLA · Stats 102C Monte Carlo Methods

## The launch that inspired it

On November 13, 2026, Databricks added Beta support for **AI Function tasks in Declarative Automation Bundles**
([release notes, Nov 2026](https://docs.databricks.com/aws/en/release-notes/product/2026/november); [bundle task type docs](https://docs.databricks.com/aws/en/dev-tools/bundles/job-task-types#ai-function-task)).
AI Function tasks (Beta since Nov 4) run `ai_parse_document`, `ai_extract` or `ai_classify` over volume files or table rows and only process new or changed inputs on each run.
The [AI Function task docs](https://docs.databricks.com/aws/en/jobs/tasks/ai-functions/) note that changing the input or output table causes all inputs to be processed again, which adds cost.

Now that these tasks live in YAML, a tidy-up rename in a pull request can trigger a full reprocess. That's the product decision this prototype takes on.

## The PM decision

When a deploy will trigger a full reprocess, sample about 60 files, estimate the cost, and **ask for confirmation only above a threshold T** (in CI this means requiring an explicit approve flag). What should the default T be?

- **Success metric:** missed bill shocks (redeploys over the customer's shock line with no prompt) ≤ 1%
- **Guardrails:** prompt rate ≤ 15%; needless prompts ≤ 35%; P90 estimate error ≤ ±40%; added deploy time ≤ 3 s
- **Stakeholders:** Bundles/CLI, AI Functions, Jobs, FinOps admins, Support

## What the prototype shows about product judgment

| Databricks APM skill | Where it shows up |
|---|---|
| Customer problem framing | A one-line YAML diff that can produce a month-sized bill, written from the data engineer's side |
| Prototyping | Seeded, interactive simulation of 1,200 redeploys with sliders for threshold, shock line, volume size, doc mix and function |
| SQL / Python | Illustrative query to back-test on real redeploy history, plus a NumPy rejection sampler |
| Data-driven decisions | "Find the best T" picks the highest threshold that keeps missed shocks ≤ 1%, then checks the guardrails |
| Clear writing | A director memo and a one-sentence line for the VP, both rewritten live from the simulation |

## Methods

| Method (Stats 102C) | Used for |
|---|---|
| Ch4 Rejection sampling (envelope, accept if U ≤ f/(M·g)) | Pages per file from a two-humped shape (short invoices + long contracts) |
| Ch4 acceptance rate = 1/M | Compares a flat box (~6% kept) with a tighter exponential + flat tent (~54% kept) |
| Ch3 inverse CDF + Ch5 mixture | Sampling from the exponential + flat proposal |
| Ch1 Monte Carlo estimation | Prompt rate, missed-shock rate and estimate-error percentiles across simulated redeploys |

## Design baseline

Four pins from Shriya's Pinterest home feed. Everything on the page is redrawn in SVG, CSS and canvas. No pin images are used.

- **Hero:** ["Today I am excited about everything" sticker collage by chlo](https://www.pinterest.com/pin/1151936410972917554/). I took the white die-cut sticker borders, glitter speckle, candy palette (sky, hot pink, orange, red, violet, green, yellow) and bold hand-lettered caps.
- **Decision cards and buttons:** [rainy umbrella painting (Pin on art)](https://www.pinterest.com/pin/1126111081822628732/). I took the periwinkle field with vertical rain-streak texture, the peach and cream blocks, and the navy scribble outlines.
- **KPI badges:** [red crayon "Hey Girl" drawing by Women of Prayer (@women_of_prayer91)](https://www.pinterest.com/pin/1083045410427952306/). I took the cream paper, red crayon hatching, a loose hand-drawn ring, the pearl dot and handwritten numerals.
- **Simulation room:** [Dragonfly/Damselfly pin by Taty](https://www.pinterest.com/pin/598767712988782831/). I took the blush paper, iridescent teal-to-lime gradient, wing-vein lattice and the classic serif.

---

Unofficial concept prototype by Shriya Shekatkar. All data is simulated. Not affiliated with any company.
