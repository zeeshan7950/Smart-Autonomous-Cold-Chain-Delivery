# Task 4 — Conservation Chamber Operations & Schemas

**Student:** Zeeshan Ashraf | **Roll No.:** 065

## Identified operations

| Operation ID | Operation name | Description |
|---|---|---|
| OP-01 | AuthenticateAndAuthoriseCommand | Validate operator identity, role and command permission. |
| OP-02 | AcceptDeliveryRequest | Validate destination/batch references and submit request to robot controller. |
| OP-03 | HandleObstacle | Suspend route progress, perform avoidance and verify path clear. |
| OP-04 | EvaluateCriticalBattery | Evaluate battery capacity and raise emergency condition when threshold is crossed. |
| OP-05 | RunSensorSelfCheck | Check essential chamber sensors at power-on and record pass/fail. |
| OP-06 | LoadBatchProfile | Read batch identification and load required temperature/humidity limits. |
| OP-07 | StartConservation | Enable conservation only when profile is loaded and door confirmed closed. |
| OP-08 | CorrectAndVerifyEnvironment | Apply temperature/humidity correction and verify recovery using fresh readings. |
| OP-09 | HandleDoorOpening | Suspend conservation and inhibit removal when door opens. |
| OP-10 | HandleVibration | Suspend risky activity until vibration stays below threshold for the stabilisation period. |
| OP-11 | HandlePowerLoss | Switch to emergency power if available; otherwise record incident and shut down safely. |
| OP-12 | EnterProtectionAndAlert | Enter protection response, reduce light exposure and alert operator after failed recovery. |
| OP-13 | VerifyBatchRemoval | Permit removal only when chamber is safe and no protection response is active. |
| OP-14 | RecordEvent | Record command, chamber, robot, fault and recovery events for audit. |

## Operation schemas

### OP-05 — RunSensorSelfCheck
- **Description:** Run at chamber power-on to establish whether essential sensors are usable.
- **Inputs:** PowerOn; essential sensor values/status; sensor inventory.
- **Outputs:** SelfCheckResult; failed sensor list; timestamped event.
- **Reads:** Sensor health/status; chamber power status.
- **Changes:** Self-check result; monitoring enable flag; event log.
- **Sends/Alerts:** Alert operator if any essential sensor fails.
- **Assumes (precondition):** Chamber is powered; sensor inventory is configured.
- **Result (postcondition):** If all essential sensors pass, monitoring may be enabled; otherwise monitoring/conservation remains disabled and the fault is recorded.

### OP-06 — LoadBatchProfile
- **Description:** Identify the placed medicine batch and load its required environmental limits.
- **Inputs:** Batch identifier; profile catalogue entry.
- **Outputs:** Loaded batch ID; temperature range; humidity range; profile status.
- **Reads:** Batch identifier and approved profile data.
- **Changes:** Active batch reference and target limits.
- **Sends/Alerts:** Record profile loaded or rejected.
- **Assumes (precondition):** Batch is placed in chamber and identification data is available.
- **Result (postcondition):** Profile is marked loaded only if the batch is identified and profile is valid.

### OP-07 — StartConservation
- **Description:** Start active conservation only after start conditions are verified.
- **Inputs:** Door status; profile status; sensor health; current readings.
- **Outputs:** Start accepted/rejected; conservation-active status.
- **Reads:** Door sensor, profile status, sensor self-check result.
- **Changes:** Conservation enable flag; chamber operating state; event log.
- **Sends/Alerts:** Log/alert rejected start if a prerequisite fails.
- **Assumes (precondition):** Monitoring is available and a batch is present.
- **Result (postcondition):** Conservation starts only if door is confirmed closed, profile loaded and essential sensors healthy; otherwise remains inactive.

### OP-08 — CorrectAndVerifyEnvironment
- **Description:** Correct out-of-range temperature or humidity and verify the result from new readings.
- **Inputs:** Measured temperature/humidity; target ranges; recovery period.
- **Outputs:** Correction status; fresh readings; recovery success/failure.
- **Reads:** Current sensor readings, target profile, actuator status, recovery timer.
- **Changes:** Correction actuator setting; recovery status; protection trigger if needed.
- **Sends/Alerts:** Alert operator if recovery fails within recovery period.
- **Assumes (precondition):** Profile loaded; sensors usable; correction actuator available.
- **Result (postcondition):** Recovery is successful only if fresh readings are within configured limits; otherwise protection mode is entered and alert is sent.

### OP-09 — HandleDoorOpening
- **Description:** Respond to door opening during conservation by suspending normal operation.
- **Inputs:** Door-open event; current chamber state; sensor status.
- **Outputs:** Conservation-suspended status; door event record; resume eligibility.
- **Reads:** Door sensor; current temperature/humidity; sensor health; protection status.
- **Changes:** Conservation enable flag; chamber state; removal interlock.
- **Sends/Alerts:** Alert/log door-open event.
- **Assumes (precondition):** Door sensor is monitored while a batch is present.
- **Result (postcondition):** Normal conservation is suspended and removal blocked; resumption requires door closed plus verified conditions and healthy sensors.

### OP-11 — HandlePowerLoss
- **Description:** Handle loss of main power using emergency power where available.
- **Inputs:** Main-power status; emergency-power availability; chamber operating state.
- **Outputs:** Power transition result; shutdown status; incident record.
- **Reads:** Power sensors, emergency supply status, active batch and current chamber state.
- **Changes:** Power source selection; operating state; incident log.
- **Sends/Alerts:** Alert operator if emergency power is unavailable or shutdown occurs.
- **Assumes (precondition):** Power monitoring is active.
- **Result (postcondition):** If emergency power is available, switch to it and record the change; otherwise record the incident and shut down safely.

### OP-12 — EnterProtectionAndAlert
- **Description:** Protect the batch after failed environmental recovery.
- **Inputs:** Recovery failure; environmental readings; batch ID.
- **Outputs:** Protection-active status; light-reduction status; alert result.
- **Reads:** Recovery status, temperature/humidity, active batch profile.
- **Changes:** Protection response flag; light exposure setting; event log.
- **Sends/Alerts:** Operator alert containing fault and batch reference.
- **Assumes (precondition):** Recovery period expired without verified recovery.
- **Result (postcondition):** Protection is active, light exposure is reduced, and failure/alert attempt is logged.

### OP-13 — VerifyBatchRemoval
- **Description:** Check that removal is safe before allowing a batch to be removed.
- **Inputs:** Removal request; door status; sensor status; protection status; readings.
- **Outputs:** Removal allowed/denied; reason code.
- **Reads:** Current readings, sensor health, door status, protection response flag.
- **Changes:** Removal interlock result; event log.
- **Sends/Alerts:** Notify operator if removal is denied.
- **Assumes (precondition):** Batch is present and a removal request is received.
- **Result (postcondition):** Removal is allowed only if chamber safety conditions are verified and no protection response is active.

## Links to Task 3 state model
- When OP-09 detects an open door during active conservation, the chamber leaves active conservation and enters a suspended/protection-required condition. Batch removal is blocked. The robot must not continue DELIVERING while the door is open (C-08).
- If OP-08 cannot verify recovery within the recovery period, OP-12 activates protection and alerts the operator. If the fault is critical, the robot enters ST-06 SAFE_MODE and inhibits normal delivery.
- If OP-05 fails, monitoring and active conservation are not enabled, and the batch is not treated as ready for delivery.

## Operations requiring sensor-confirmed postconditions
1. **OP-08 CorrectAndVerifyEnvironment:** A cooling/heating/humidity command does not prove recovery. Actuator failure, delayed response or system dynamics can leave values out of range; fresh readings inside profile limits establish success.
2. **OP-09 HandleDoorOpening / resume checks:** A door-close command alone does not prove safe conditions. Door status, environmental readings and sensor health must be rechecked before conservation resumes.

Thresholds and recovery/stabilisation periods are configuration parameters because the scenario does not provide numeric values.
