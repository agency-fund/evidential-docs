# Experiment Design Types

Evidential offers two families of experiment design:

- **Frequentist (A/B testing)** keeps the split between arms fixed for the whole experiment. Use it to
    measure whether a change makes a difference, and by how much.
- **Bayesian (multi-armed bandits)** learns as results come in and sends more participants to the
    arms that are doing best. Use it to find and serve the best option while the experiment runs.

Within each family, the design you pick decides when participants are assigned, what Evidential needs
from your data warehouse, and how results reach Evidential.

## At a glance

| Design                             | Participants are assigned          | Data warehouse | Results come from            |
| ---------------------------------- | ---------------------------------- | -------------- | ---------------------------- |
| Preassigned A/B                    | All at once, before the start date | Required       | Your data warehouse          |
| Online A/B                         | One at a time, as they arrive      | Required       | Your data warehouse          |
| Multi-Armed Bandit                 | One at a time, as they arrive      | Not needed     | Your app reports each result |
| Multi-Armed Bandit (DWH-connected) | One at a time, as they arrive      | Required       | Your data warehouse          |
| Contextual Multi-Armed Bandit      | One at a time, as they arrive      | Not needed     | Your app reports each result |

Evidential connects to **BigQuery**, **Redshift** and **Postgres**. Bandits that don't use a data
warehouse run on your organization's **API Only** datasource.

______________________________________________________________________

## Frequentist designs

A frequentist experiment is a classic A/B test. You set the arms and how participants are split
between them, check that you'll have enough participants to detect the effect you care about, and
compare the arms once the experiment ends.

Every frequentist design:

- **Uses a participant table.** You point Evidential at a table or view in your data warehouse that
    lists your participants, with one column that uniquely identifies each one. Metrics, filters and
    strata are all columns in this table.
- **Includes a power check.** Before you launch, Evidential uses your data to estimate how many
    participants you need to detect your target effect, and shows how that changes with sample size.
- **Reads results from your warehouse.** Evidential collects metric values for assigned participants
    directly from your data warehouse and compares each arm against the baseline arm. Your app doesn't
    report results.

### Preassigned A/B

Evidential selects participants and assigns every one of them to an arm when you create the
experiment.

- **Sample size.** Choose how many participants to include. Evidential draws them at random from the
    participants who match your filters.
- **Balanced arms.** You can pick strata, such as region or age group, so each arm gets a similar mix
    of participants. Evidential also balances the arms on your metrics, and checks the result before
    saving the assignments.
- **Group assignment.** If your intervention reaches whole groups at once, such as a school or a
    village, you can assign groups instead of individuals. Choose the column that identifies the group
    and how many groups to include. Everyone who matches your filters in a selected group joins the same
    arm. Participants with no group are left out. Group assignment can't be combined with strata.
- **Getting assignments.** Download the full list as a CSV, fetch it through the API, or look up one
    participant at a time.

Choose Preassigned A/B when you know your participants ahead of time, or need the full list of who's
in which arm before the start date.

### Online A/B

Evidential assigns each participant the first time your app asks for their assignment, using the split
you set. There's no list of participants up front.

- **Your participant table is still required.** Evidential uses it for the power check and to read
    results, but doesn't select participants from it.
- **No strata or group assignment.** Each participant is assigned independently as they arrive.
- **Every request assigns.** Asking about a new participant always enrolls them. If you only want part
    of your traffic in the experiment, decide which participants to include in your own app first, then
    request assignments only for them.
- **Assignments don't change.** Once a participant has an arm, every later request returns the same
    arm.

Choose Online A/B when you don't know who will take part in advance, or participants arrive
continuously.

### How filters work: Preassigned vs Online

Filters limit an experiment to the participants you want, for example *country is Kenya* or *signed up
after 1 January*. Each filter compares one column to one or more values using **includes**,
**excludes** or **between**. An experiment can have up to 20 filters, and a participant must match all
of them.

The two frequentist designs apply the same filters in different ways.

**Preassigned A/B** applies filters to your participant table:

- in the **power check**, so sample size estimates reflect only participants who match, and
- when you **create the experiment**, to decide who can be selected.

Who's eligible is decided once, from the data in your warehouse at that moment.

**Online A/B** can't look up a new participant in your warehouse at the moment they arrive. Instead,
your app sends the participant's details along with the assignment request, and Evidential checks those
details against your filters:

- If the details match, the participant is assigned an arm.
- If they don't, no arm is returned and the participant isn't enrolled.

To have filters applied, your app must use the assignment request that includes participant details.
The **Integration Guide** for each experiment shows this request under **Example API Calls**. A
request without participant details assigns the participant regardless of your filters.

#### Limitations

**Preassigned A/B**

- Eligibility is fixed when you create the experiment. People who match your filters later aren't
    added, and participants who stop matching stay in the experiment.
- To change filters, create a new experiment.

**Online A/B**

- Filters only apply when your app sends participant details. Without them, every participant is
    assigned.
- Evidential uses the details your app sends as they are. It doesn't check them against your data
    warehouse.
- Send a value for every filtered column. A missing value counts as empty, which usually means the
    participant doesn't match and isn't enrolled.
- Eligibility is checked only the first time a participant is assigned. They aren't checked again or
    removed later.

**Both designs**

- Filters can only use columns in the experiment's participant table.
- Filters always combine with "and". You can't create "this or that" conditions across different
    columns.
- Text matches are exact and case-sensitive. There's no partial or wildcard matching.
- **Between** works only with numbers and dates, and includes both end values. You can leave one end
    open, such as "on or after 1 January".
- Empty values are handled explicitly. **Includes** matches empty values only if you add "empty" to
    the list. **Excludes** keeps empty values unless you exclude "empty". **Between** leaves out empty
    values unless you choose to include them.
- Dates and times are in UTC.

______________________________________________________________________

## Bayesian designs

Bayesian designs are multi-armed bandits. Each arm starts with an expected performance, called a
prior. As results come in, Evidential updates its estimate for each arm and sends more participants to
the arms that look best.

Every bandit:

- **Has an outcome type**: **Binary** for yes/no results such as a click or a sign-up, or
    **Real-valued** for numbers such as a rating or time spent.
- **Starts from priors**, which you can set for each arm or have Evidential derive from starting
    weights.
- **Shows results as estimates** for each arm, which update as results arrive. There's no power check.

!!! note

    Bandits are built to find and serve the best option, not to measure the size of a difference. If
    you need a reliable estimate of an effect, use an A/B test.

### Multi-Armed Bandit

The standard bandit, and the only design that needs no data warehouse. Your app requests an
assignment for each participant and reports each participant's result back to Evidential. To set it
up, choose **No data warehouse** in the experiment wizard.

**Autofail** is available for this design. If a participant has no result after a time window you set,
Evidential records a default result for them, such as a "no" for a binary outcome.

### Multi-Armed Bandit (DWH-connected)

A Multi-Armed Bandit whose results come from your data warehouse instead of from your app. In the
wizard's datasource step, pick a datasource, table, the column that identifies participants, and a
**Target column** that holds each participant's result.

- **Your app requests assignments, but doesn't report results.** Evidential reads the target column
    for assigned participants on a regular schedule. The participant IDs your app uses must match the
    values in the column that identifies participants.
- **Results can arrive late.** Empty values are checked again on later reads. Evidential keeps reading
    for 8 days after the end date to catch results that land late.
- **The target column sets the outcome type.** A true/false column is a binary outcome. A numeric
    column is real-valued. Values that don't fit the column's type are ignored.
- **Autofail isn't available**, since results come only from your warehouse.

### Contextual Multi-Armed Bandit

A contextual bandit learns which arm works best for different kinds of participants. Along with each
assignment request, your app sends context: information about the participant or their situation, such
as their age or the time of day.

- **At least one context is required.** Each context is either binary or real-valued.
- **Your app reports each result**, as with the standard Multi-Armed Bandit. Autofail is available.
- **Results depend on context.** To see an arm's estimated performance, you choose context values.
    Saved results over time are calculated at the average of the context values seen so far.
- **No example calls.** The Integration Guide doesn't include example API calls for this design. See
    the [API documentation](https://api.evidential.dev/docs) instead.

______________________________________________________________________

## Choosing a design

| If you want to…                                                        | Use                                   |
| ---------------------------------------------------------------------- | ------------------------------------- |
| Assign a known list of participants before you start                   | Preassigned A/B                       |
| Assign whole groups, such as schools or villages                       | Preassigned A/B with group assignment |
| Measure the effect of a change on participants who arrive over time    | Online A/B                            |
| Find and serve the best option, with your app reporting results        | Multi-Armed Bandit                    |
| Find and serve the best option, with results already in your warehouse | Multi-Armed Bandit (DWH-connected)    |
| Find the best option for each kind of participant                      | Contextual Multi-Armed Bandit         |

To connect your app to an experiment, see [API Integration](../integration/integration.md). To run
experiments in WhatsApp with Turn.io, see [Messaging Platforms](../integration/messaging-platforms.md).
