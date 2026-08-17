# Experiment Design Types

Evidential supports two families of experiment design:

- **Frequentist** designs hold the traffic split fixed and answer *"is the treatment different from
    the control, and by how much?"* with a power analysis up front and significance testing at the end.
- **Bayesian** designs (multi-armed bandits) update a posterior belief about each arm as outcomes
    arrive and shift traffic toward the arms that are performing best.

Pick the family first, then the variation within it. The variation determines *when* participants are
assigned, *what* Evidential needs from your data warehouse, and *which* API calls your integration
makes.

## At a glance

| Design                             | Type ID            | Assignment happens         | Data warehouse                          |
| ---------------------------------- | ------------------ | -------------------------- | --------------------------------------- |
| Preassigned A/B                    | `freq_preassigned` | At commit time, in bulk    | Required (participants sampled from it) |
| Online A/B                         | `freq_online`      | Per API call, in real time | Required (design + analysis only)       |
| Multi-Armed Bandit                 | `mab_online`       | Per API call, in real time | Not required                            |
| Multi-Armed Bandit (DWH-connected) | `mab_online_dwh`   | Per API call, in real time | Required at design time only            |
| Contextual Multi-Armed Bandit      | `cmab_online`      | Per API call, in real time | Not required                            |

Where a data warehouse is required, Evidential currently connects to **BigQuery**, **Redshift** and
**Postgres**. Organizations also get an **API Only** datasource, which is what bandits use when you
choose *No data warehouse*.

______________________________________________________________________

## Frequentist designs

Frequentist experiments are the classic A/B test. You fix the arms and their weights before the
experiment starts, size the experiment against a minimum detectable effect (MDE), and read the result
once at the end.

**What they have in common:**

- **A participant table.** Every frequentist design points at a table (or view) in your warehouse and
    a `primary_key` column that uniquely identifies a participant. Metrics, filters and strata are all
    columns on that table.
- **A power check.** Before you commit, Evidential queries the warehouse for the mean and variance of
    each metric across the eligible population and reports the sample size needed to detect your target
    effect, at your chosen `power` (default 80%) and `alpha` (default 5%).
- **Primary and secondary metrics.** One primary metric drives sizing; secondary metrics are analysed
    alongside it.
- **Analysis from the warehouse.** At analysis time Evidential re-reads metric values for the
    assigned participants and compares each arm against a baseline arm. You never push outcome values
    for frequentist designs.

!!! note

    Frequentist designs measure a *fixed* comparison. If your goal is to keep serving whichever
    variant is currently winning rather than to measure the difference between them, use a bandit.

### Preassigned A/B (`freq_preassigned`)

Assignment happens once, when you commit the experiment. Evidential queries your participant table,
draws a sample of the size you chose, and writes the arm assignments for every participant up front.

- **Sampling.** `desired_n` participants are drawn from the eligible population. The experiment
    cannot be created if the filters leave no eligible participants, or if the sampled rows contain
    duplicate primary keys.
- **Stratification.** Optional `strata` columns balance the arms on participant attributes. By
    default Evidential also stratifies on your metrics, so the arms start from comparable baselines.
- **Cluster randomization.** Set a `cluster_key` (for example a school or clinic ID) and
    `desired_n_clusters` to randomize whole clusters instead of individuals. All eligible participants
    in a sampled cluster are included, rows with a null cluster key are dropped, and per-metric ICC,
    average cluster size and coefficient of variation are either supplied by you or derived from the
    warehouse during the power check. Cluster-randomized designs cannot also use strata.
- **Balance check.** The assignment is checked for balance and stored with the experiment. A p-value
    above `fstat_thresh` (default 0.6) is treated as balanced.
- **Integration.** Export the full assignment list as CSV from the UI, fetch it in bulk from
    `GET /v1/experiments/{experiment_id}/assignments`, or look up one participant at a time. Lookups
    never create new assignments — `create_if_none` is ignored for this type.

Use it when you know your population in advance and want the whole roster of who is in which arm
before the experiment starts — for example to load arm labels into a messaging tool.

### Online A/B (`freq_online`)

Assignment happens in real time, in response to your application's API calls. There is no roster: a
participant enters the experiment the first time you ask for their assignment, and the arm is drawn
according to the design's fixed weights.

- **Design-time warehouse use only.** The participant table is still required — it is how Evidential
    resolves field types and runs the power check — but nothing is sampled at commit time.
- **No stratification or clustering.** Strata and `cluster_key` are preassigned-only features;
    arriving participants are randomized independently.
- **Every request assigns.** Each call for an unseen participant creates an assignment. Sample the
    share of traffic you want in the experiment *in your own system first*, then call Evidential for
    the participants you selected — rather than routing 100% of traffic into an experiment with a 90%
    control arm.
- **Repeat lookups are stable.** Once a participant has an arm, the same experiment/participant pair
    always returns that arm. Pass `create_if_none=false` to check for an existing assignment without
    creating one.

Use it when you don't know who will show up, or when participants arrive continuously.

### How filters work: Preassigned vs Online

Filters narrow the eligible audience to the participants you actually want in the experiment. You
define them once in the design — a field name, a relation (`includes`, `excludes` or `between`) and a
list of values — and up to 20 filters can be attached to one experiment. Filters always combine with
**AND**.

The same filter definition is enforced very differently by the two frequentist types.

#### Preassigned: filters run as SQL, once

For a preassigned experiment, filters are compiled into a `WHERE` clause and executed against your
participant table:

- during the **power check**, so the reported sample sizes and MDEs describe the filtered population,
    and
- at **commit time**, to build the eligible pool that `desired_n` is sampled from.

The result is a snapshot of eligibility, taken at commit time and stored with the experiment.
Evidential trusts your warehouse: the values it filters on are the values in your table.

#### Online: filters are evaluated per request, from properties you send

For an online experiment, Evidential does not read your warehouse when assigning. The filters are
stored as eligibility metadata, and evaluated in-process against **participant properties supplied in
the request**:

```
POST /v1/experiments/{experiment_id}/assignments/{participant_id}/assign_with_filters
{
  "properties": [
    {"field_name": "country", "value": "KE"},
    {"field_name": "signup_date", "value": "2026-01-15"}
  ]
}
```

If the properties pass every filter, an assignment is created and returned. If they don't, the
response comes back with a null assignment and no assignment is recorded.

The plain `GET .../assignments/{participant_id}` endpoint has no properties to evaluate, so on a
filtered online experiment it would assign **everyone** who is asked about. For that reason, the
Integration Guide for a filtered online experiment shows the `GET` only as a read-only call
(`create_if_none=false`) and lists `assign_with_filters` as the call to assign with.

#### Limitations

**Preassigned**

- Eligibility is frozen at commit time. Warehouse rows that become eligible later never enter the
    experiment, and participants whose attributes stop matching stay assigned.
- Changing filters means designing a new experiment; a committed design's audience cannot be
    re-filtered.

**Online**

- Filters are only enforced when you send properties. A call to the plain `GET` endpoint, or an
    `assign_with_filters` call with an empty `properties` list, assigns the participant regardless of
    the filters.
- Property values are **self-reported by the caller** and trusted as-is. Evidential does not verify
    them against the warehouse, so the integration is responsible for supplying honest values.
- Send a property for **every** filtered field. A field you omit is read as `NULL`, which normally
    fails an `includes` filter and silently drops an otherwise eligible participant.
- Eligibility is checked only at first assignment. Once a participant has an arm, later calls return
    it and the properties in the request body are ignored — participants are never re-screened or
    removed.
- Filter fields must still exist on the participant table with the type you filter on; property
    values are validated against that type and a mismatch is an error, not a silent skip.

**Both**

- Filters can only reference columns on the experiment's participant table, and there is no support
    for compound or nested boolean logic — every filter must hold (`AND`), so "A or B" across two
    different fields can't be expressed.
- String comparisons are exact and case-sensitive; there is no wildcard, prefix or regex matching.
- `between` applies to numeric, date and timestamp fields only, and takes 2 values (optionally a
    third value of `null` to also admit null values).
- Nulls are explicit. `includes [null]` matches nulls; `excludes` treats nulls as passing unless you
    exclude `null` yourself.
- Date and timestamp values must be ISO 8601 and, if a timezone is given, UTC.
- Bandit designs ignore filters entirely — they have no eligibility criteria of their own.

______________________________________________________________________

## Bayesian designs

Bayesian designs are multi-armed bandits. Each arm carries a prior belief about its performance;
Evidential samples an arm from those beliefs (Thompson sampling), and every outcome you report updates
the posterior. Traffic drifts toward the better arms while the experiment runs.

**What they have in common:**

- **Outcomes are pushed to Evidential.** After a participant's result is known, report it with
    `POST /v1/experiments/{experiment_id}/assignments/{participant_id}/outcome`. Bandits never read
    outcomes from your warehouse.
- **Priors.** `beta` priors take `alpha_init`/`beta_init` per arm and work only with binary outcomes;
    `normal` priors take `mu_init`/`sigma_init` and work with either outcome type. Alternatively, give
    the arms initial weights (summing to 100) and Evidential derives the priors from them.
- **Outcome type.** `binary` (0/1) or `real-valued`.
- **Optional autofail.** Turn it on to record a default outcome (`autofail_outcome_value`) for
    participants who report nothing within `autofail_window` hours — for binary outcomes that value
    must be 0 or 1. Without it, unobserved assignments simply don't move the posterior.
- **Analysis is posterior-based.** There is no power check and no p-value; the analysis reports each
    arm's posterior, refreshed on the same snapshot schedule as frequentist experiments.

!!! note

    Bandits are for optimizing, not for measuring. Because allocation changes as the data arrives,
    they are not a substitute for a fixed A/B test when your goal is a clean effect estimate.

### Multi-Armed Bandit (`mab_online`)

The standard bandit, and the only experiment type that needs no data warehouse at all. Assignments
come from `GET /v1/experiments/{experiment_id}/assignments/{participant_id}`, outcomes go back through
the outcome endpoint, and everything lives in Evidential. When you choose *No data warehouse* in the
wizard, the experiment is created under your organization's **API Only** datasource.

Note that power checks are not available on an API-only datasource — sizing an experiment requires
warehouse data.

### Multi-Armed Bandit, DWH-connected (`mab_online_dwh`)

A `mab_online` experiment gains this type when you bind its outcome to a warehouse column: pick a
datasource, table, primary key and **target column** in the optional MAB datasource step, and
Evidential stores the target column with the experiment.

Warehouse support here is deliberately narrow, and worth being precise about:

- The warehouse is consulted **at design time only**, to resolve the target column's data type.
- Outcomes still arrive through the push API. Evidential does not read outcome values out of the
    target column.
- The stored type is used to validate what you push: a boolean target column only accepts outcomes of
    0 or 1.

It behaves like a MAB everywhere else — same arms, priors, analysis and badge (shown as *MAB (+DWH)*
in the UI).

### Contextual Multi-Armed Bandit (`cmab_online`)

A CMAB conditions the choice of arm on context — participant or environmental features you pass in
with each assignment request.

- **Contexts are required** (at least one) and each is typed `binary` or `real-valued`.
- **Normal priors only.** Beta priors are rejected for this type.
- **Assignment needs the context vector**, so it uses its own endpoint:
    `POST /v1/experiments/{experiment_id}/assignments/{participant_id}/assign_cmab` with
    `context_inputs`. The plain `GET` can only return an assignment that already exists. For the same
    reason, the Integration Guide shows no example calls for CMAB experiments.
- **Analysis needs a context vector too**: use
    `POST /v1/datasources/{datasource_id}/experiments/{experiment_id}/analyze_cmab`. Automatic
    snapshots currently evaluate the posterior at the *mean* observed context values (rounded for
    binary contexts) rather than marginalizing over the context distribution.

______________________________________________________________________

## Choosing a design

| If you…                                                                | Use                                |
| ---------------------------------------------------------------------- | ---------------------------------- |
| Know your participant roster up front and need arm labels exported     | Preassigned A/B                    |
| Randomize groups (schools, clinics, villages) rather than individuals  | Preassigned A/B with a cluster key |
| Meet participants as they arrive and want a clean effect estimate      | Online A/B                         |
| Want to converge on the best variant while the experiment runs         | Multi-Armed Bandit                 |
| Want a bandit whose outcome column is validated against your warehouse | Multi-Armed Bandit (DWH-connected) |
| Want the best variant *per context*, not overall                       | Contextual Multi-Armed Bandit      |

Next: see [API Integration](../integration/integration.md) for how to fetch assignments and report
outcomes, and [Messaging Platforms](../integration/messaging-platforms.md) if you deliver your
treatments over WhatsApp with Turn.io.
