# Task 1 — Requirement Extraction

**Student:** Zeeshan Ashraf | **Roll No.:** 065

Requirements are phrased as observable behaviours. Verification methods define evidence for a pass/fail decision.

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| FR-01 | The robot shall accept a delivery request only when it is IDLE and the request contains a valid destination and batch reference. | FR | High | Submit valid and invalid requests; inspect response and state. |
| FR-02 | Before executing each remote robot command, the system shall validate operator authentication and role authorisation for that command. | FR | Critical | Attempt commands as unauthenticated, authorised and wrong-role operators. |
| FR-03 | On a valid delivery request, the robot shall transition from IDLE to NAVIGATING and record the request identifier. | FR | High | Verify transition telemetry and request event log. |
| FR-04 | When an obstacle is detected during navigation, the robot shall suspend route progress, enter obstacle handling, and resume navigation only after the obstacle is confirmed cleared. | FR | Critical | Simulate obstacle detection and clearance; inspect states and telemetry. |
| FR-05 | At the destination, the robot shall deliver the assigned batch and then transition to RETURNING toward the warehouse. | FR | High | End-to-end route simulation and delivery/return event inspection. |
| FR-06 | When critical battery is detected during navigation or obstacle handling, the robot shall abandon delivery and return toward the warehouse unless a higher-priority Safe Mode response is required. | FR | Critical | Inject critical battery during navigation and obstacle handling. |
| FR-07 | At power-on, the chamber shall run a self-check and permit monitoring only if every essential sensor passes. | FR | High | Inject sensor failures; verify monitoring remains disabled on failure. |
| FR-08 | The chamber shall begin active conservation only when a batch profile is loaded and the door is confirmed closed. | FR | Critical | Test profile/door combinations and conservation status. |
| FR-09 | For out-of-range temperature or humidity, the chamber shall issue a correction action and confirm recovery using subsequent sensor readings before reporting the value in range. | FR | Critical | Test successful and failed correction with fresh sensor readings. |
| FR-10 | If the door opens during conservation, the chamber shall suspend normal conservation and require sensor and condition verification before resuming; batch removal shall be blocked while unsafe or protection is active. | FR | Critical | Door-open, resumption and removal-interlock tests. |
| NFR-01 | The robot shall enter Safe Mode within 3 seconds of detecting the CR-01 critical chamber/battery temperature or emergency battery-capacity condition. | NFR | Critical | Timestamp detection and Safe Mode entry during fault injection. |
| NFR-02 | Mission Control shall support at least 20 simultaneously connected robots in the pilot configuration. | NFR | High | Load test with 20 concurrent robot sessions. |
| NFR-03 | Every accepted or rejected command shall produce an auditable record containing command ID, operator ID, decision, timestamp and outcome. | NFR | High | Compare command transactions against audit records. |
| NFR-04 | The remote interface shall tolerate several-second link delay without executing the same command ID more than once. | NFR | High | Network-delay and duplicate-message test. |
| NFR-05 | The static project portal shall present robot, operator and project-artefact documentation in clearly labelled sections accessible through ordinary web navigation. | NFR | Medium | Inspect labels and run navigation/usability checks. |
| SAF-01 | A robot shall not enter a railway crossing unless the barrier is confirmed open and no train is approaching. | Safety | Critical | Crossing simulator with train and barrier-state combinations. |
| SAF-02 | On a CR-01 critical battery or temperature condition, the robot shall enter Safe Mode within 3 seconds and stop normal delivery activity. | Safety | Critical | Timed fault-injection and activity observation. |
| SAF-03 | ARLCCS shall keep the barrier closed while a train is present and permit opening only after the crossing is confirmed clear. | Safety | Critical | Simulate train presence, clearance and barrier feedback. |
| SAF-04 | If a critical sensor or control-link failure makes safe crossing status untrustworthy, the robot shall not enter the crossing and the system shall alert and log the fault. | Safety | Critical | Inject sensor/link failures; verify stop, alert and log. |
