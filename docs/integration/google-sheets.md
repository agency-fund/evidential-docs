# Google Sheets demos

Google Sheets is a **read-only datasource for preassigned A/B demos**. Evidential reads a public tab's CSV export;
no API keys, OAuth, or Google credentials are needed. You import assignments manually.
Use synthetic data only: the linked tabs must be publicly accessible.

## Connect the raw tab

1. Import [the example CSV](https://github.com/agency-fund/evidential-be/blob/ac0707337712c31968b97afc4e069a6436955d0b/tools/google_sheets_demo.csv)
    into a Google spreadsheet. It contains 1,000 synthetic participants, regions, and seven metric options.
    Six metrics have sample observations; `onboarded_within_1_week` starts blank for live entry.
1. Set sharing to **Anyone with the link → Viewer** and allow downloads. Public access must be allowed by your Workspace.
    Copy the raw tab's URL including `gid`; without it, Evidential uses the first tab.
1. In Evidential, add a **Google Sheets (demo)** datasource with the **Raw tab URL**. The table picker shows
    the spreadsheet/tab name supplied by Google.
    Use `participant_id` as the ID and choose a primary metric and optional secondary metrics.
    Leave **Cluster key** empty for individual assignment. To balance regions across arms, add `region` to **Strata**;
    using it as the cluster key instead assigns whole regions together (four clusters in the example).

The raw tab remains the source for setup, power analysis, enrollment, and CSV exports.

## Connect an experiment's outcomes tab

1. Create and save a preassigned A/B experiment, then click **Download Experiment CSV** immediately before importing it.
    The CSV retains raw-tab values for participant IDs, selected metrics, and filter, strata, or cluster fields.
    It adds `evidential_<experiment_id>_arm`, matched by participant ID; unassigned participants have a blank arm.
1. In Google Sheets, use **File → Import → Upload → Insert new sheet(s)** and rename the new tab **Experiment**.
    Keep the raw tab. Disable **Convert text to numbers, dates, and formulas** to preserve IDs, including leading zeroes.
    See [Google's CSV import instructions](https://support.google.com/docs/answer/40608?hl=en).
1. Copy the new tab's URL including `gid`. On the experiment page, click **Connect Experiment tab**, paste it into
    **Experiment tab URL (outcomes)**, and click **Save experiment tab**. The experiment name is shown as text;
    the raw datasource URL is locked and greyed out.

Each experiment has its own outcomes connection. Connecting or reconnecting it leaves the datasource and other
experiments unchanged. Refresh and live polling stay disabled until connected.
Assignment exports continue to use raw-tab values, not later outcome edits.

## Run the demo

Edit the selected metric cells in the **Experiment** tab, then click **Refresh** to save and display an analysis snapshot.
The arm comparison and time-series chart update, with exact timestamps and history preserved after reloading.

**Start live demo** refreshes about every 10 seconds for up to 15 minutes. Polling stops when the page closes,
on an error, on reconnection, or when you click **Stop live demo**. Google may briefly cache CSV exports.
If you replace or delete a tab, its `gid` can change; reconnect using the current URL.

## Choose metrics

Each row represents one participant. Calculate measures in the sheet or upstream; Evidential reads the values you enter.
All seven example metrics are in the same sheet, so changing metrics does not require a new datasource.

| Sheet column                   | What it measures                                                                       | Example use               |
| ------------------------------ | -------------------------------------------------------------------------------------- | ------------------------- |
| `minutes_on_site_last_7_days`  | Total minutes per participant in the last 7 days                                       | Websites and apps         |
| `customer_satisfaction_1_to_5` | Satisfaction rating from 1 to 5                                                        | Customer experience       |
| `revenue_last_7_days`          | Revenue per participant in the last 7 days, in one consistent currency                 | Commerce and fundraising  |
| `purchases_last_7_days`        | Number of purchases per participant in the last 7 days                                 | Commerce                  |
| `support_resolution_hours`     | Hours to resolve a support request; lower is better                                    | Support teams             |
| `assessment_score_0_to_100`    | Assessment score from 0 to 100                                                         | Education and training    |
| `onboarded_within_1_week`      | Blank while unknown; 1 if onboarded within 7 days, otherwise 0 after the window closes | Onboarding and activation |

For onboarding, the mean is the rate among participants with observed results: 0.75 means 75%.
Keep results blank while a participant's seven-day window is still open.
The filled example values are synthetic, not measurements of a real intervention.

During setup, the selected column's mean and spread supply baseline statistics for sample-size planning.
**Minimum Effect** defaults to 10%, the relative change to detect. During analysis, the baseline arm is the control group.

Clustered confidence intervals reflect variation between clusters. Equal cluster means can produce a zero-width
mean interval even when individual values vary.

### Start with blank outcomes

Keep participant IDs and metric headers populated, with outcome cells empty. Select a metric, click **Estimate Sample Size**,
then choose **Use the maximum available sample size** (1,000 in the example) or a custom size.
Blank outcomes cannot provide meaningful power or minimum-detectable-effect estimates, but you can continue the demo.
Comparisons appear once both baseline and treatment arms have observations. Zero is a result, not a pending outcome.
Use the filled example if you also want to demonstrate power analysis.

For clustered assignment, blank outcomes still allow participant counts and cluster sizes, but not ICC or power estimates.
Choose a maximum or custom cluster count to continue.

## Sheet requirements and limits

- Row 1 must contain unique column names starting with a letter or underscore, using only letters, numbers, and underscores.
- Format participant IDs as plain text. Keep them unique and unchanged after assignment; sorting rows is supported.
    Newly added participants are not automatically enrolled in a preassigned experiment.
- Use plain numeric outcome cells without currency symbols or thousands separators. `TRUE`/`FALSE` are inferred as booleans.
    Blank cells are missing; zero and `FALSE` are observed outcomes. Entirely blank metric columns are inferred as numeric.
- Maximum size: 5,000 data rows, 100 columns including assignment columns, and an 8 MB CSV download.

## API notes

Each analysis reads a fresh, temporary in-memory snapshot through the warehouse query path.
The experiment config exposes `google_sheets_experiment_url`, updated through the experiment PATCH endpoint.
Omitting the field preserves the connection; explicit `null` disconnects it.
Analysis requires a connection, and scheduled snapshots skip unconnected Sheets experiments.
Table inspection returns Google's source name as `display_name`; `linked_sheet` remains the internal table ID.
