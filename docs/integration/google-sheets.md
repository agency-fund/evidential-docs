# Google Sheets

Google Sheets is a read-only datasource for preassigned A/B experiments. Evidential reads a tab's CSV export;
assignments are downloaded as CSV and imported into Google Sheets manually. OAuth, private-sheet access,
and automatic assignment writeback are not supported.

## Availability

Google Sheets is restricted to privileged users and disabled by default.

- Set `XNGIN_ENABLE_GOOGLE_SHEETS=true` in the API and snapshot-worker environments, then restart those processes.
- Enable the existing frontend flag mechanism with `ffon('google_sheets')` in the browser console.
  Use `ffoff('google_sheets')` to hide it again.
- Both switches and privileged-user status are required. The browser flag does not grant backend access.
  Datasource API keys do not grant access to Sheets; use a privileged user session.

## Connect a datasource

1. Share the spreadsheet as **Anyone with the link → Viewer** and allow downloads. Your Workspace must permit public access.
   Do not use this connector for confidential or personal data: access restrictions in Evidential do not make the Google export private.
1. Copy the source tab's URL, including `gid`, and add a **Google Sheets** datasource in Evidential.
   Without `gid`, the first tab is used. The table picker displays the source name supplied by Google.
1. Create a preassigned A/B experiment. Select the participant ID column and primary and optional secondary metrics,
   using the standard experiment setup controls.

The datasource tab remains the source for setup, enrollment, and assignment exports.

## Connect experiment outcomes

1. Save the experiment and download its assignments using the standard CSV export in **Arms & Allocations**.
   The export includes participant IDs, selected experiment fields, and `evidential_<experiment_id>_arm`.
1. Import the CSV with **File → Import → Upload → Insert new sheet(s)**. Keep the source tab unchanged.
   Disable **Convert text to numbers, dates, and formulas** to preserve IDs, including leading zeroes.
1. Open the experiment's Google Sheets connection settings and enter the outcomes tab's URL, including `gid`.
   The source URL is locked, and the experiment name is displayed as context rather than an editable field.

Each experiment owns its outcomes connection. Changing it does not repoint the datasource or other experiments.
Assignments remain authoritative in Evidential; editing arm labels in the spreadsheet does not reassign participants.

Enter outcome values in the connected tab and use the normal analysis refresh control. Scheduled snapshots
retain analysis history. Google may briefly cache CSV exports. If a tab is replaced or deleted, reconnect its current URL.

## Data requirements

- Each populated row needs a unique, stable participant ID. Format IDs as plain text to preserve leading zeroes.
  Rows may be reordered; newly added participants are not automatically enrolled in a preassigned experiment.
- Row 1 must have unique headers starting with a letter or underscore, using only letters, numbers, and underscores.
- Use plain numeric outcome cells without currency symbols or thousands separators. `TRUE` and `FALSE` are booleans.
  Blank cells represent missing outcomes; zero and `FALSE` are observed outcomes.
- Capacity limits are 5,000 data rows, 100 columns including assignment columns, and an 8 MB CSV export.

## API notes

The experiment config exposes `google_sheets_experiment_url`. Omitting it from an experiment PATCH preserves
the connection; explicit `null` disconnects it. Analysis requires a connection, and unconnected experiments
are skipped by scheduled snapshots.

Table inspection exposes Google's source name as `display_name`; `linked_sheet` remains an internal identifier.
The UI shows a loading state while the source name is being retrieved.
