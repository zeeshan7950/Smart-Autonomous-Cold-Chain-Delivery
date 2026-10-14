# Task 2 — Requirements Traceability Matrix (RTM)

**Student:** Zeeshan Ashraf | **Roll No.:** 065

## Multi-layer traceability

| Requirement | State / Transition | Operation | Constraint | Test Case | Risk |
|---|---|---|---|---|---|
| FR-02 | ST-01/ST-02; TR-01 | OP-01 AuthenticateAndAuthoriseCommand | C-09 | TC-01, TC-02 | Critical |
| FR-03 | ST-01 → ST-02; TR-01 | OP-02 AcceptDeliveryRequest | C-10 | TC-03 | High |
| FR-04 | ST-02 → ST-03 → ST-02; TR-02/TR-03 | OP-03 HandleObstacle | C-10 | TC-04 | Critical |
| FR-06 / SAF-02 | ST-02 or ST-03 → ST-06; TR-06/TR-07 | OP-04 EvaluateCriticalBattery | C-10 | TC-05 | Critical |
| FR-07 / FR-08 | Chamber CH-01 → CH-02; CH-03 → CH-04 | OP-05 RunSensorSelfCheck; OP-07 StartConservation | C-07/C-08 | TC-06, TC-07 | Critical |
| FR-09 | Chamber CH-04 → CH-05 or CH-06 | OP-08 CorrectAndVerifyEnvironment | C-07 | TC-08 | Critical |
| FR-10 | Chamber CH-04 → CH-07; robot activity inhibited | OP-09 HandleDoorOpening | C-08/C-10 | TC-09 | Critical |
| NFR-01 / CR-01 | Any normal robot state → ST-06 within 3 s | OP-04 EvaluateCriticalBattery; OP-09 EnterSafeMode | C-10 | TC-05, TC-10 | Critical |
| NFR-02 / CR-02 | Mission Control capacity; state-independent | OP-10 MaintainRobotSession | C-09 | TC-11 | High |
| SAF-01 / SAF-03 | Crossing-entry guard; crossing states | OP-11 AuthoriseCrossingEntry | C-01/C-02/C-03 | TC-12 | Critical |
| SAF-04 | Link loss → ST-07; crossing entry blocked | OP-12 HandleLinkLoss | C-05/C-06 | TC-13 | Critical |
| NFR-03 | State-independent event logging | OP-14 RecordEvent | C-09 | TC-14 | High |

## Test cases

| Test ID | Scenario / input | Expected result | Traceability | Regression after |
|---|---|---|---|---|
| TC-01 | Command received from unauthenticated operator | Reject command; no robot action; rejection logged. | FR-02/C-09 | CR-03 |
| TC-02 | Authenticated operator without command role issues command | Reject command; no robot action; rejection logged. | FR-02/C-09 | CR-03 |
| TC-03 | Valid delivery request received while IDLE | Accept request; enter NAVIGATING; report status and log event. | FR-01/FR-03 | None directly |
| TC-04 | Obstacle detected during navigation, then cleared | Enter AVOIDING_OBSTACLE; do not deliver while avoiding; resume navigation only after clear confirmation. | FR-04/C-10 | None directly |
| TC-05 | Critical battery or chamber/battery temperature trigger injected | Enter Safe Mode within 3 seconds; stop normal delivery; report status and log event. | SAF-02/NFR-01/C-10 | CR-01 |
| TC-06 | Power-on with one essential chamber sensor failed | Self-check fails; monitoring/conservation remains disabled; fault logged and alerted. | FR-07/C-07 | None directly |
| TC-07 | Batch profile loaded while door open, then door closed | Conservation does not start with door open; starts only after profile and closed-door confirmation. | FR-08/C-08 | None directly |
| TC-08 | Temperature/humidity out of range; correction issued | Re-read sensors; report recovery only when values are in range; otherwise protection mode and alert. | FR-09/C-07 | None directly |
| TC-09 | Door opens during conservation; removal attempted | Suspend conservation; block removal until safety checks pass and protection is inactive. | FR-10/C-08 | None directly |
| TC-10 | Measure trigger detection-to-Safe-Mode interval | Elapsed time is ≤ 3 seconds for each CR-01 trigger; timestamps are logged. | NFR-01/SAF-02 | CR-01 |
| TC-11 | Connect 20 robots concurrently | All 20 sessions are connected and supported during the test. | NFR-02 | CR-02 |
| TC-12 | Train approaching or barrier not confirmed open | Robot does not enter; stop/alert/log. Entry allowed only when barrier is open and no train approaches. | SAF-01/SAF-03/C-01 | None directly |
| TC-13 | Remote link lost before crossing permission is confirmed | Robot stays outside crossing; alert and log link fault. | SAF-04/C-06 | None directly |
| TC-14 | Accepted and rejected commands sent with unique IDs | Every outcome has an audit record with operator, command, timestamp, decision and outcome. | NFR-03/C-09 | CR-03 |

## Scenario sequence coverage
The tests cover the intended command flow: **command received → authenticated → role-authorised → executed/rejected → status reported → event logged**. Duplicate command IDs and delayed links are also checked under NFR-04.
