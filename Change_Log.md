# Task 2 — Change Log

**Student:** Zeeshan Ashraf | **Roll No.:** 065

| Change ID / Requirement | Old text | New text | Reason | Status |
|---|---|---|---|---|
| CR-01 / SAF-02, NFR-01 | The robot shall enter Safe Mode when a critical battery or thermal condition is detected. | The robot shall enter Safe Mode within 3 seconds when chamber or battery temperature exceeds the critical threshold or battery capacity falls below the emergency level. | Adds a measurable response time and observable trigger conditions. | Modified |
| CR-02 / NFR-02 | The system shall support communication with multiple robots simultaneously. | The system shall support at least 20 simultaneously connected robots. | Replaces vague “multiple” with a measurable minimum. | Modified |
| CR-03 / FR-02 | Only authenticated operators shall be permitted to issue robot commands. | The system shall require authenticated and role-authorised operators before accepting any robot command. | Adds role-based permission and makes command acceptance the enforcement point. | Modified |

## Why the new wording is more testable

- **CR-01:** Detection and Safe Mode timestamps can be compared; the result must be no more than 3 seconds. Trigger thresholds are configuration values.
- **CR-02:** A load test can connect 20 robots concurrently and verify each session.
- **CR-03:** Tests can separately reject an unauthenticated operator and an authenticated operator whose role lacks permission.

## Regression tests
- **CR-01:** Re-run `TC-05`, `TC-10` and Safe Mode transition checks.
- **CR-02:** Re-run `TC-11` and connection/session regression.
- **CR-03:** Re-run `TC-01`, `TC-02`, `TC-14` and command execution checks to ensure role checks cannot be bypassed.
