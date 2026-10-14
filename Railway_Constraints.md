# Task 5 — Railway Crossing Constraints, Formalisation & Violations

**Student:** Zeeshan Ashraf | **Roll No.:** 065

## 5.1 Constraints

| Constraint ID | Constraint in simple English | Why necessary |
|---|---|---|
| C-01 | Robot crossing entry is permitted only when the barrier is confirmed open and no train is approaching. | Prevents robot/train collision. |
| C-02 | If a train is present, the barrier must not be open. | Prevents opening while a train occupies the crossing. |
| C-03 | If a train is approaching, the barrier must be closed or closing and warning lights/audible warning must be active. | Provides warning and blocks entry. |
| C-04 | The barrier may open only after the crossing is confirmed clear by valid train-detection evidence. | Prevents premature barrier opening. |
| C-05 | A failed or invalid train sensor makes crossing status untrusted; robot entry is blocked. | Fail-safe response to sensor uncertainty. |
| C-06 | If ARLCCS communication is lost before crossing permission is confirmed, the robot must not enter. | Missing permission is not permission. |
| C-07 | The chamber may report environmental recovery only when fresh sensor readings are within active batch limits. | Avoids false safe status for medicine. |
| C-08 | The robot must not be DELIVERING while the chamber door is open or protection response is active. | Protects medicine and prevents unsafe delivery. |
| C-09 | Every accepted/rejected remote command and critical crossing/chamber fault must be logged; command execution requires authentication and role authorisation. | Security, audit and incident reconstruction. |
| C-10 | During SAFE_MODE, normal delivery and crossing-entry actions are inhibited. | Prevents hazardous activity after a critical fault. |

## 5.2 Formalisation

| Constraint | Logical expression | Meaning |
|---|---|---|
| C-01 | `Robot_Enter_Crossing → (Barrier_Open ∧ ¬Train_Approaching)` | Entry guard. |
| C-02 | `Train_Present → ¬Barrier_Open` | Barrier remains closed while train is present. |
| C-03 | `Train_Approaching → ((Barrier_Closed ∨ Barrier_Closing) ∧ Warning_Active)` | Approach triggers barrier protection and warning. |
| C-04 | `Barrier_Open → (Crossing_Clear ∧ Train_Sensors_Valid)` | Opening requires confirmed clearance and valid sensors. |
| C-05 | `¬Train_Sensors_Valid → ¬Robot_Enter_Crossing` | Sensor uncertainty blocks entry. |
| C-06 | `(Communication_Lost ∧ ¬Crossing_Permission_Confirmed) → ¬Robot_Enter_Crossing` | No confirmed permission means no entry. |
| C-07 | `Environment_Recovery_Reported → (Temperature_In_Range ∧ Humidity_In_Range ∧ Fresh_Readings)` | Recovery requires observed readings. |
| C-08 | `(Robot_State = DELIVERING) → (Door_Closed ∧ ¬Protection_Active)` | Delivery requires safe chamber condition. |
| C-09 | `Command_Executed → (Authenticated ∧ Role_Authorised ∧ Event_Logged)` | Command execution is controlled and auditable. |
| C-10 | `(Robot_State = SAFE_MODE) → (¬Normal_Delivery_Active ∧ ¬Robot_Enter_Crossing)` | Safe Mode inhibits normal hazardous actions. |

Boolean variables are evaluated from the current monitored system snapshot. `Robot_State = DELIVERING` is a predicate. A monitor evaluates an implication whenever its antecedent is true.

## 5.3 Violation scenarios, detection and response

| Constraint | Variable values showing violation | Detection | Expected response |
|---|---|---|---|
| C-01 | `Robot_Enter_Crossing=TRUE; Barrier_Open=FALSE; Train_Approaching=FALSE` | Entry guard detects barrier is not open. | Stop before crossing; deny entry; alert and log. |
| C-02 | `Train_Present=TRUE; Barrier_Open=TRUE` | Train and barrier sensors conflict with rule. | Critical alarm; command closure if mechanism is healthy; inhibit entry; alert/log. |
| C-03 | `Train_Approaching=TRUE; Barrier_Closed=FALSE; Barrier_Closing=FALSE; Warning_Active=FALSE` | Approach detector sees train but barrier/warning response is absent. | Raise alarm; issue close/warning commands; block entry; log fault. |
| C-04 | `Barrier_Open=TRUE; Crossing_Clear=FALSE; Train_Sensors_Valid=TRUE` | Barrier-open feedback conflicts with occupancy monitor. | Treat crossing as unsafe; inhibit entry; alarm/log; request safe barrier response if available. |
| C-05 | `Train_Sensors_Valid=FALSE; Robot_Enter_Crossing=TRUE` | Sensor-health monitor marks data invalid during entry request. | Reject entry and stop robot; sensor-failure alert and log. |
| C-06 | `Communication_Lost=TRUE; Crossing_Permission_Confirmed=FALSE; Robot_Enter_Crossing=TRUE` | Link monitor sees no confirmed permission while entry is requested. | Deny entry; stop before crossing; alert and record link loss. |
| C-07 | `Environment_Recovery_Reported=TRUE; Temperature_In_Range=FALSE; Humidity_In_Range=TRUE; Fresh_Readings=TRUE` | Post-correction monitor compares fresh readings to profile and detects temperature out of range. | Withdraw recovery status; continue correction or enter protection; reduce light; alert/log. |
| C-08 | `Robot_State=DELIVERING; Door_Closed=FALSE; Protection_Active=FALSE` | Door sensor and robot activity monitor detect delivery while door is open. | Suspend delivery; inhibit unsafe handling; alert/log; resume only after checks. |
| C-09 | `Command_Executed=TRUE; Authenticated=TRUE; Role_Authorised=FALSE; Event_Logged=TRUE` | Authorisation audit finds role mismatch despite execution record. | Stop/reject action where possible; security alert; log incident and review command path. |
| C-10 | `Robot_State=SAFE_MODE; Normal_Delivery_Active=TRUE; Robot_Enter_Crossing=FALSE` | Activity monitor sees normal delivery during Safe Mode. | Inhibit delivery; preserve Safe Mode; alert/log violation. |

## Monitoring and verification notes
- Monitors must use time-stamped, health-checked sensor values. Invalid or stale readings cannot prove that a crossing is clear or chamber conditions recovered.
- If a barrier actuator fails, the system must not assume it moved. Keep the robot outside the crossing until safe permission is confirmed, and escalate the fault.
- Log the constraint ID, input snapshot, timestamp, action taken and alert outcome so each violation can be reproduced.
- C-07 and C-08 link chamber safety to robot activity; C-05 and C-06 cover sensor and communication failures.
