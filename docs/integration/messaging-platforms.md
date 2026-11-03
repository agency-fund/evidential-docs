# Messaging Platforms

Evidential decides which arm each participant is in. Your messaging platform delivers the content for
that arm.

Evidential integrates directly with **Turn.io**. For other platforms, your app can request assignments
through the [API](https://api.evidential.dev/docs). See [API Integration](integration.md).

______________________________________________________________________

## Turn.io

[Turn.io](https://whatsapp.turn.io/) organizes WhatsApp conversations into **Journeys**. With the Evidential
Turn.io App, each arm of your experiment is linked to a Journey. When a contact enters your
experiment, the app gets their arm from Evidential and starts the matching Journey.

### Before you start

You'll need:

- A **base Journey** where contacts enter the experiment.
- One **arm Journey** for each arm, with the content for that arm. If you're running a bandit where
    contacts' results are collected in Turn.io, each arm Journey should also collect the result.
- An experiment in Evidential. See [Experiment Design Types](../welcome/experiment-design-types.md).

### Set up the connection

You only need to do this once for your organization.

1. **Create a Turn.io API token.** In Turn.io, go to **Settings > API & Webhooks > Create a Token**.
1. **Add the token to Evidential.** In Evidential, go to **Integrations** and click **Add API Key** in
    the **Turn.io API Key** section. Evidential loads your Journeys and creates a **Webhook ID** and
    **Webhook Auth Token** for Turn.io.
1. **Install the app in Turn.io.** On the Evidential **Integrations** page, download the Evidential
    Turn.io App. In Turn.io, go to **Apps > Installed Apps > Upload app**, upload the .zip file, and
    click **Install app**.

### Set up an experiment

Do this for each experiment you run in Turn.io.

1. **Link arms to Journeys.** In Evidential, open your experiment and click **Integration Guide**.
    Under **Configure integration for third-party tools**, choose a Journey for each arm and click
    **Save**.
1. **Get an API key.** In the same Integration Guide, copy the **Experiment ID**. If the app doesn't
    have an Evidential API key yet, click **Add API Key** and copy the key. One key works for every
    experiment on the same datasource.
1. **Configure the app.** In Turn.io, open the Evidential app's **Settings** page. Paste in the
    Evidential API key, Webhook ID and Webhook Auth Token.
1. **Route contacts to the experiment.** In your base Journey, add a step that calls the app's
    `route_to_experiment` function with the Experiment ID. The app's **About** page in Turn.io has
    full instructions.
1. **Record results, if needed.** For a **Multi-Armed Bandit** or **Contextual Multi-Armed Bandit**,
    add a step to each arm Journey that calls the app's `post_outcome_for_contact` function with the
    contact's result. Skip this step for A/B tests and DWH-connected bandits, which read results from
    your data warehouse.
1. **Publish** the app and your Journeys.

### What happens when a contact enters

1. The contact reaches the step in your base Journey that calls the app.
1. The app asks Evidential for the contact's arm, using the contact's WhatsApp ID as the participant
    ID.
1. The app saves the experiment ID and arm ID to the contact's profile, then starts the Journey linked
    to that arm.

If a contact enters again, Evidential returns the same arm.

### Things to know

- **Participant IDs are WhatsApp IDs.** For a **Preassigned A/B** experiment, the column that
    identifies participants in your participant table must contain your contacts' WhatsApp IDs.
    Otherwise, Evidential won't find their assignments.
- **Online A/B filters aren't applied.** The app doesn't send contact details when it asks for an
    assignment, so every contact routed to an Online A/B experiment is enrolled. To limit who takes
    part, add conditions to your base Journey before the step that calls the app.
- **Journey changes sync automatically.** When you add, rename or delete Journeys in Turn.io, the app
    tells Evidential the next time it runs or its settings are saved. Updating your Turn.io API key in
    Evidential also refreshes the list.
- **Fix out-of-date links.** If a linked Journey is deleted, the **Integration Guide** button turns
    red and the affected arms show a warning. Choose a new Journey for each flagged arm and click
    **Save**.
- **One Turn.io workspace per organization.** Each organization can connect one Turn.io API token.
- **Deleting the connection removes all links.** If you delete the Turn.io API key in Evidential, the
    links between arms and Journeys are deleted for every experiment in your organization.

### Security

- Evidential stores your Turn.io API token encrypted and only ever shows its last 4 characters.
- Turn.io must include the Webhook Auth Token when it contacts Evidential. If you think the token has
    been exposed, click the rotate button next to it on the **Integrations** page, then paste the new
    token into the app's settings in Turn.io.
- The Evidential API key only works for experiments on its datasource. Manage your keys on the
    datasource's page in Evidential.

______________________________________________________________________

## Other platforms

To run experiments on other messaging or survey platforms:

- For **Preassigned A/B**, download the assignments as a CSV or fetch them through the API. Then tag
    each contact in your platform with their arm, so your flows can branch on it.
- For **Online A/B** and **bandits**, have your app request each participant's assignment from
    Evidential when they arrive, then show them the content for their arm.

See [API Integration](integration.md) for details, or [contact us](../contact.md) to ask about support
for your platform.
