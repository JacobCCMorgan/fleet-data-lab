# Fleet Data Lab

Applied data work on a working ELD / fleet-telematics business: real operational data (de-identified), concrete business questions, measured results, honest limits. Each project is its own repository with code, data lineage, tests and a write-up; this page is the map.

The data behind every project comes from one US ELD provider's device-management system, driver-app back office and CRM — ~5,000 ELD units on heavy-duty trucks, ~5,700 customer organisations, seven years of contracts, 30-second telemetry and device firmware logs. It is released only as keyed pseudonyms with coordinates coarsened and all personal / company identifiers removed (policy in the foundation repository). Numbers describe that fleet at the stated snapshot dates, not the industry and not the provider's current performance.

## Projects

| | Repository | Question | Status |
|---|---|---|---|
| F0 | [telematics-data-foundation](https://github.com/JacobCCMorgan/telematics-data-foundation) | How do you turn PII-laden operational exports into a shareable, join-safe, tested data model — and what does the fleet look like once you can query it? | **Released** — 12 tables, DuckDB views, 43 quality checks, 16 tests, snapshot findings |
| P1 | eld-ecm-disconnect | Which ELD units have lost their engine (ECM) connection persistently rather than transiently, can tomorrow's chronic cases be predicted today, and who should the field team visit first? | In progress |
| P2 | eld-device-log-parser | What do 85,000 lines of device firmware log say about upload latency, offline buffering, state transitions and error codes — and how do you make that queryable? | Planned |
| P3 | eld-subscription-lifecycle | Where in a one-year-contract business does churn happen, which organisations are at risk before their renewal date, and how much of next quarter's device demand is migration versus new business? | Planned |
| P5 | fmcsa-eld-logbook-toolkit | Can an FMCSA ELD output file be parsed, checksum-verified, reconciled against the eRODS event list and run through the hours-of-service rules — on a real driver's month? | Planned |
| P4 | eld-telemetry-replay | When 2,000+ trucks stream 30-second reports, how do you segment trips and stops, detect ECM silence while moving, and stay correct when buffered data arrives late? | Planned (awaiting raw telemetry file) |

## Findings so far (from F0, snapshot 2026-09-28)

- 40 % of reporting ELD units have never read a VIN from the engine computer; on the anomaly day, 47 % of flagged reports show VIN, speed and RPM all missing while GPS says the truck is doing 96 km/h. 1,176 units were silent on RPM all day.
- Only 1 of the 29 chronically disconnected units ever appeared in the field-issue tracker — the support queue does not see this failure mode.
- The newest "Volvo 2026" batch is moving-while-silent on 98 % of flagged reports versus 33–61 % for regular batches: a protocol / wiring issue on new model-year trucks, not random hardware failure.
- 12.6 % of reporting units last reported in low-battery mode and 500 have not updated in 30+ days — a ready installer work-list.
- 74 % of contract terms are exactly 365 days, tenure clusters at 365 / 730 / 1,095 days and 90 % of returned units came back within the first year: churn is a first-renewal event. 82 % of 2026 activations replaced an earlier device.

Each statement links to a reproducible script in the foundation repository.

## How to read the projects

- **Real / public / synthetic** is stated for every dataset. Nothing synthetic is presented as fleet performance.
- **Validation** is designed per question (time-based splits, independent time windows, round-trip checks, hand-computed fixtures), and every README has a "what this cannot tell you" section.
- Code is Python (pandas, DuckDB, scikit-learn, lifelines) and SQL; tests run in GitHub Actions.

## About

Built by Jacob Chiang, data scientist at an ELD / fleet-telematics company (ELD compliance, device telemetry, driver risk scoring). This lab is one of two portfolio tracks; the other is [data-portfolio](https://github.com/JacobCCMorgan/data-portfolio). [LinkedIn](https://www.linkedin.com/in/jacob-chiang)
