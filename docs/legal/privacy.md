# Privacy Policy

_Last updated on 24 December 2025_

## 1 · Introduction & Scope

Evidential is an open-source A/B-testing platform operated by two U.S.–based maintainers The Agency Fund and IDInsight (“we,” “us,” “our”). The hosted instance is available only to pre-approved nonprofit organisations (located mainly in Africa and India). This Policy explains what personal information we handle when you use the service (the “Service”), why we do so, and the choices available to you.

## 2 · What Information We Collect

We collect and store the following:

- Email addresses (for login and account management)
- Credentials for each partner’s data warehouse connection (secrets are hashed)
- Experiment metadata (e.g., experiment name, start/end dates, metric names)
- Usage logs including timestamp, IP address, HTTP request
- Unique identifiers of the experiment participants (defined by user). It is the client's responsibility to ensure this identifier is anonymised before use in the Service.

We access, but do not store, raw-level data in partner databases. Queries are executed directly in the partner’s environment. We may store unique values for user-selected columns as part of Evidential’s summary views. These columns may contain PII, and it’s user responsibility to avoid using them as experiments specifications.

We also store anonymised snapshots of experiment results at regular intervals. These contain no PII.

## 3 · How We Use the Information

- To authenticate users and enforce access controls
- To execute and monitor experiments
- To troubleshoot, secure and improve the Service
- To communicate with users about platform updates or incidents

We never sell personal information or use it for advertising.

## 4 · Legal Bases

- Contractual necessity – to provide the Service to your organisation
- Legitimate interests – to maintain security, prevent misuse and improve product quality
- Consent – only where required (e.g., optional feedback surveys)

## 5 · Data Retention

- Account and credential data – retained until account deletion or termination of partnership. Snapshots of internal databse may be retain up to 3 months after account deletion, unless specifically requested otherwise by the user.
- Server and audit logs – retained for 30 days
- Experiment assignments and results snapshots - retained up up to 3 months after account deletion, unless specifically requested otherwise by the user.

## 6 · Sub-processors

We rely on the following sub-processors:

- Railway – hosting and database infrastructure (California, USA)
- GitHub – source-code hosting and CI/CD tools (USA) and hosting documentation
- Google – for end-user authentication
- CloudFlare – for DNS
- Sentry - for request observability and analytics

We will update this list before onboarding any additional sub-processor.

## 7 · Security Measures

- Transport Layer Security (TLS) encrypts all network communication
- Warehouse credentials are encrypted at rest and never logged or exposed
- Regular vulnerability scans of all software dependencies

We do not currently run periodic penetration tests.

## 8 · Your Rights

Depending on your jurisdiction, you may have rights to access, rectify, erase, or restrict the processing of your personal data. To exercise these rights, email us at support@evidential.dev We will respond within the time frame required by applicable law.

## 9 · International Transfers

The Service is hosted exclusively in the United States. By using it, your organisation authorises the transfer of all data detailed above to the U.S. We rely on standard contractual clauses or equivalent safeguards where required.

## 10 · Changes to This Policy

We may revise this Policy from time to time. Major changes will be notified to all participating organisations at least 30 days before they take effect.

## 11 · Contact

For any privacy question, reach us at support@evidential.dev
