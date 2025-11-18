# Data Storage

Evidential is built with **privacy by design** — it stores **only what’s essential** to make experiments work, mostly aggregated, while raw data always stays in your own environment.

What we store:

______________________________________________________________________

## ✅ Aggregared Data

We mostly only **anonymized and aggregated** information in a **PostgreSQL application database**, such as:

- **Experiment specifications** — name, hypothesis, start/end dates, configuration details
- **Experiment results** — regular snapshots of aggregated experiment metrics

______________________________________________________________________

## ⚠️ Granular Data

There are two types of user-led actions that can expose granular data containing PII:

- **Participant IDs**
    Used to track assignment and outcomes.
    👉 *Users must ensure IDs used in experiments containt no raw personal information, or hash them before use*

- **Unique values of columns used as filters**
    Stored to enable autocompled. Avoid using columns with PII (e.g., names, emails) as filters.

______________________________________________________________________

## 🚫 What We Don’t Store

Evidential **doesn't stores or copies raw-level data**. All analysis queries run **directly in your database**.

______________________________________________________________________

## 📄 Learn More

Read our full [Privacy Policy](#) *(coming soon)* for details on data handling and protection.
