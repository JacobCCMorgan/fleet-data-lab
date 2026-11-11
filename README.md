# Fleet Data Lab

Applied data work on a working ELD / fleet-telematics business: real operational data (de-identified), concrete business questions, measured results, honest limits. Each project is its own repository with code, data lineage, tests and a write-up; this page is the map.

The data behind every project comes from one US ELD provider's device-management system, driver-app back office and CRM — ~5,000 ELD units on heavy-duty trucks, ~5,700 customer organisations, seven years of contracts, 30-second telemetry and device firmware logs. It is released only as keyed pseudonyms with coordinates coarsened and all personal / company identifiers removed (policy in the foundation repository). Numbers describe that fleet at the stated snapshot dates, not the industry and not the provider's current performance.

## Projects

| | Repository | Question | Status |
|---|---|---|---|
| F0 | [telematics-data-foundation](https://github.com/JacobCCMorgan/telematics-data-foundation) | How do you turn PII-laden operational exports into a shareable, join-safe, tested data model — and what does the fleet look like once you can query it? | **Released** — 12 tables, DuckDB views, 43 quality checks, 16 tests, snapshot findings |
| P1 | [eld-ecm-disconnect](https://github.com/JacobCCMorgan/eld-ecm-disconnect) | Which ELD units have lost their engine (ECM) connection persistently rather than transiently, does it come back three days later, does predicting it take a model, and who should the field team visit first? | **Released** — device table, repeated-CV scorer comparison, decision curves, tiered priority list, dashboard, 21 tests |
| P2 | [eld-device-log-parser](https://github.com/JacobCCMorgan/eld-device-log-parser) | What do 85,000 lines of device firmware log say about upload latency, offline buffering, state transitions and error codes — and how do you make that queryable? | **Released** — 80-pattern registry (100 % of lines), nine event tables, Drain3 comparison, 81 tests |
| P3 | [eld-subscription-lifecycle](https://github.com/JacobCCMorgan/eld-subscription-lifecycle) | Where in a one-year-contract business does churn happen, which organisations are at risk before their renewal date, and how much of next year's renewal load is migration versus new business? | **Released** — reconstructed renewal histories validated on a second export, Kaplan-Meier / Cox / discrete-time hazard, time-ordered model holdouts, 60-day risk list, renewal calendar, call-campaign outcomes, 11 tests |
| P5 | [fmcsa-eld-logbook-toolkit](https://github.com/JacobCCMorgan/fmcsa-eld-logbook-toolkit) | Can an FMCSA ELD output file be parsed, checksum-verified, reconciled against the eRODS event list and run through the hours-of-service rules — on a real driver's month? | **Released** — Appendix A data checks verified on a production file, record-level reconciliation with the portal export, duty timeline, 395.3 and 395.5 rule engines, unidentified-driving analysis, 36 tests |
| P4 | eld-telemetry-replay | When 2,000+ trucks stream 30-second reports, how do you segment trips and stops, detect ECM silence while moving, and stay correct when buffered data arrives late? | Planned (awaiting raw telemetry file) |

## Findings so far (from F0, snapshot 2026-09-28)

- 40 % of reporting ELD units have never read a VIN from the engine computer; on the anomaly day, 47 % of flagged reports show VIN, speed and RPM all missing while GPS says the truck is doing 96 km/h. 1,176 units were silent on RPM all day.
- Only 1 of the 14 chronically disconnected customer units ever appeared in the field-issue tracker — the support queue does not see this failure mode. (A further 15 "chronic" units are the provider's internal bench devices, which have no engine attached and are excluded from every customer figure.)
- The newest "Volvo 2026" batch is moving-while-silent on 98 % of flagged reports versus 33–61 % for regular batches: a protocol / wiring issue on new model-year trucks, not random hardware failure.
- 12.6 % of reporting units last reported in low-battery mode and 500 have not updated in 30+ days — a ready installer work-list.
- 74 % of contract terms are exactly 365 days and tenure clusters at the anniversaries. (An earlier version of this line read churn as a first-year event from `activation_date`; P3 showed that field is the start of the current term, rolled forward at each renewal, and the foundation findings were corrected.)

Each statement links to a reproducible script in the foundation repository.

## Findings from P1 (ECM disconnect, 23-28 Sep 2026)

- 14 chronic units are 8 % of the 182 flagged customer devices and produce 54 % of the moving-while-silent telemetry; 69 % of them were severe again three days later versus 12-15 % of the rest.
- A model is not needed to rank them: the plain count of silent reports over three days predicts a severe day 4 as well as a random forest and better than the chronic / intermittent labels, which ignore how much the truck drives.
- The 27 devices above 1,000 silent reports in three days (15 % of the list) produced 82 % of the next observed day's silent driving; 14 of them are "intermittent" under the share-based rule and would be missed by a chronic-only campaign.
- Only 4.4 % of flagged devices have a support ticket (base rate 5.2 %); most August cases had resolved by September while 62 of 88 severe devices on 28 Sep were new. The ranking has to run daily.

## Findings from P2 (firmware logs, two devices)

- 73 % of a device log is serial-bus and LED chatter; everything support looks for (state changes, errors, OTA, offline buffering) is under 1 % of lines.
- Uploads go out every 30 s while driving and every 5 min while sleeping; 300 of 304 were acknowledged within a second, and the three that hit a broker connection reset were stored, re-sent and acknowledged 10–32 s later.
- The firmware's ECM-disconnect event (ERR005) fired 22 times in an hour on an OBD-II test vehicle whose only missing field was engine hours — a false positive a fleet-level rule on RPM would not raise.
- The truck rebooted five times around an over-the-air update; "VIN not yet detected" turned out to be parked trucks with the engine off, not broken installs.

## Findings from P3 (subscription lifecycle, 2019-2026)

- The CRM overwrites term dates at each renewal and purges rows of customers who leave; a second export from January 2024 proves both and lets the renewal histories be reconstructed and validated (95 % of the 2024 customers reconstructed as covered on that date).
- The first anniversary is not a cliff: 23 % of customers do not renew at the first, 28-30 % at the second to fourth. What moved is the calendar: non-renewal rose from 17 % of anniversaries in 2023 to 30 % in 2025, at every tenure.
- Fleet size at renewal time is the strongest attribute (one vehicle 25 % non-renewal, six or more 8 %); 43 % of churned customers returned the device mid-term, a third of first-year churn is done before the 60-day decision point, and a device return during the term multiplies the odds of non-renewal by 2.3.
- The CRM alone gives the renewal desk a modest ranking (AUC 0.59 on time-ordered holdouts; the top fifth of accounts holds 30 % of the churners), and the yearly level moves more than any attribute. 3,663 subscriptions come up in the next twelve months with 640-840 expected non-renewals; the 2.0 hardware migration is 90 % done and customers who agreed to swap on a 2025 call are 88 % migrated and 4 % gone.

## Findings from P5 (ELD output file, one driver, September 2026)

- The device's data-check values match the regulation's algorithms exactly: 800 of 800 line checks, the file check and 424 of 424 driver event checks verify, and the 83 unidentified-driver checks are consistent with one username. The output file and the portal's eRODS event list agree record for record (790 sequence ids, 0 differences).
- The driver ran 16 days of 9.6–10.0 driving hours. Under the property-carrying rules the month holds one violation (a 25-minute lunch where 30 were required); under the passenger-carrying rules none, but the longest day is 0.8 minutes under the 10-hour limit. Which rule set applies is a question about the operation, not the file, and it changes the answer.
- The same vehicle was driven on 9 of the driver's days off with nobody logged in — 52 hours, 1,424 miles in the unidentified driver profile — and three more unidentified driving records are the driver's own driving during involuntary logouts. The unidentified-driving diagnostic was raised on Sep 10 and never cleared.
- Thirty of the device maker's certification-test files (kept private) show what the checks catch: a VIN edited after the fact fails 24 line checks, duplicate sequence ids expose a hand-edited submission, and event checks computed before a vehicle got its unit number verify only with the empty element — a provenance change the validator reports separately from corruption.

## How to read the projects

- **Real / public / synthetic** is stated for every dataset. Nothing synthetic is presented as fleet performance.
- **Validation** is designed per question (time-based splits, independent time windows, round-trip checks, hand-computed fixtures), and every README has a "what this cannot tell you" section.
- Code is Python (pandas, DuckDB, scikit-learn, lifelines, matplotlib) and SQL; tests run in GitHub Actions.

## About

Built by Jacob Chiang, data scientist at an ELD / fleet-telematics company (ELD compliance, device telemetry, driver risk scoring). This lab is one of two portfolio tracks; the other is [data-portfolio](https://github.com/JacobCCMorgan/data-portfolio). [LinkedIn](https://www.linkedin.com/in/jacob-chiang)
