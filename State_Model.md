# Task 3 — State Modelling & Transition Verification

**Student:** Zeeshan Ashraf | **Roll No.:** 065

## States

| State ID / Name | Meaning | Entry condition | Exit condition / allowed next states |
|---|---|---|---|
| ST-01 / IDLE | Powered on, waiting for a valid delivery request. | Power-on completes; no delivery active. | Valid request → NAVIGATING; critical fault → SAFE_MODE; link loss → LINK_LOST. |
| ST-02 / NAVIGATING | Following the approved route to destination. | Valid request accepted or obstacle cleared and route resumes. | ObstacleDetected → AVOIDING_OBSTACLE; DestinationReached → DELIVERING; CriticalBattery → RETURNING; critical fault → SAFE_MODE; link loss → LINK_LOST. |
| ST-03 / AVOIDING_OBSTACLE | Route progress suspended while obstacle is handled. | Obstacle detected during navigation. | ObstacleCleared and path verified → NAVIGATING; CriticalBattery → RETURNING if safe, otherwise SAFE_MODE; critical fault → SAFE_MODE; link loss → LINK_LOST. |
| ST-04 / DELIVERING | Performs delivery action at destination. | Destination reached from NAVIGATING and delivery prerequisites pass. | DeliveryComplete → RETURNING; critical fault → SAFE_MODE; link loss → LINK_LOST. |
| ST-05 / RETURNING | Travels back to warehouse; delivery may have been abandoned. | Delivery complete or critical battery requires abort/return. | WarehouseReached → IDLE; critical fault → SAFE_MODE; link loss → LINK_LOST. |
| ST-06 / SAFE_MODE | Normal delivery actions are inhibited during critical fault handling. | Critical battery, temperature or critical fault. | Exit only after fault clearance, health checks and authorised recovery; then IDLE or RETURNING under recovery policy. |
| ST-07 / LINK_LOST | Remote command/telemetry link is unavailable or stale. | Link monitor declares communication unavailable. | LinkRestored plus integrity checks → prior safe state or SAFE_MODE; critical fault → SAFE_MODE. |

## Events and conditions

| Event ID | Event | Meaning |
|---|---|---|
| EV-01 | DeliveryRequest | Valid request received and input checks pass. |
| EV-02 | ObstacleDetected | Obstacle detector reports an obstacle in the route. |
| EV-03 | ObstacleCleared | Obstacle no longer present and path verified clear. |
| EV-04 | DestinationReached | Arrival at destination geofence confirmed. |
| EV-05 | DeliveryComplete | Delivery action and required confirmation complete. |
| EV-06 | CriticalBattery | Battery capacity below emergency level. |
| EV-07 | CriticalTemperature | Chamber or battery temperature exceeds critical threshold. |
| EV-08 | WarehouseReached | Arrival at warehouse confirmed. |
| EV-09 | LinkLost / LinkRestored | Communication monitor declares link unavailable/restored. |
| EV-10 | FaultCleared | Required checks pass and recovery is authorised. |

A **state** describes the current mode over an interval; an **event** is an occurrence or detected condition that may trigger a transition.

## Transition table

| Transition | Current State + Event | Next State | Requirement | Valid? | Verification evidence |
|---|---|---|---|---|---|
| TR-01 | IDLE + valid DeliveryRequest | NAVIGATING | FR-01, FR-03 | Valid | Request validation and state telemetry. |
| TR-02 | NAVIGATING + ObstacleDetected | AVOIDING_OBSTACLE | FR-04 | Valid | Obstacle event logged; route progress suspended. |
| TR-03 | AVOIDING_OBSTACLE + ObstacleCleared and path verified | NAVIGATING | FR-04 | Valid | Clear-path evidence and navigation-resumed event. |
| TR-04 | NAVIGATING + DestinationReached | DELIVERING | FR-05 | Valid | Destination confirmation and delivery-entry event. |
| TR-05 | DELIVERING + DeliveryComplete | RETURNING | FR-05 | Valid | Delivery completion record and return route. |
| TR-06 | NAVIGATING + CriticalBattery | RETURNING | FR-06 | Valid if no higher-priority critical fault | Battery reading and delivery-abandoned/return event. |
| TR-07 | AVOIDING_OBSTACLE + CriticalBattery | RETURNING | FR-06 | Conditional: only if safe to manoeuvre; otherwise SAFE_MODE | Battery reading and safe return/abort evidence. |
| TR-08 | RETURNING + WarehouseReached | IDLE | FR-05 | Valid | Warehouse geofence and idle telemetry. |
| TR-09 | IDLE + direct Deliver command | DELIVERING | FR-01, FR-05 | Invalid | Must reject and log invalid transition. |
| TR-10 | AVOIDING_OBSTACLE + Deliver command | DELIVERING | FR-04 | Invalid | Delivery remains inhibited during obstacle handling. |
| TR-11 | Any normal state + CriticalTemperature/critical fault | SAFE_MODE | SAF-02 | Valid | Safe Mode entered within 3 seconds for CR-01 triggers. |
| TR-12 | Any remote-dependent state + LinkLost | LINK_LOST | SAF-04 | Valid | Link-loss event; stale remote commands not executed. |
| TR-13 | LINK_LOST + LinkRestored and checks pass | Prior safe state or SAFE_MODE | SAF-04 | Conditional | Session integrity checked; duplicates handled safely. |
| TR-14 | SAFE_MODE + FaultCleared and authorised recovery | IDLE or RETURNING | SAF-02 | Conditional | Fault-clear evidence and authorised recovery record. |
| TR-15 | DELIVERING + CriticalBattery | SAFE_MODE or controlled RETURNING | FR-06, SAF-02 | Policy required; default SAFE_MODE for emergency-level condition | Delivery stops and high-priority safety response is logged. |

## Extended state policy
- SAFE_MODE may be entered from IDLE, NAVIGATING, AVOIDING_OBSTACLE, DELIVERING, RETURNING or LINK_LOST when a critical temperature, emergency battery condition or critical fault is detected.
- LINK_LOST may be entered from any state depending on remote command/telemetry. Local movement must follow an approved safe policy; missing permission is not permission to continue a hazardous action.
- Critical safety conditions take priority over routine transitions.

## Verifier findings
1. **Can IDLE → DELIVERING happen?** No. It violates FR-01/FR-05 and the explicit rule that a waiting robot must never begin delivering directly. Normal delivery requires NAVIGATING + DestinationReached.
2. **If AVOIDING_OBSTACLE has no transition back, can the robot still deliver?** Not through the intended safe workflow. It risks a liveness failure and cannot safely reach the destination. ObstacleCleared must return it to NAVIGATING; timeout/fault paths also need definition.
3. **Can AVOIDING_OBSTACLE → DELIVERING occur?** No. It violates FR-04 and the prohibition on delivery while handling an obstacle. Reject and log it.
4. **CriticalBattery in AVOIDING_OBSTACLE or DELIVERING?** Missing transitions make the model incomplete. In obstacle handling, define abort/return if safe or Safe Mode otherwise. During delivery, emergency-level battery/thermal conditions take priority and should stop delivery, with Safe Mode as the conservative default.
5. **Additional behavioural risks:** duplicate delayed messages may execute twice unless command IDs are deduplicated; LinkLost must not leave stale commands valid; Safe Mode exit needs explicit clearance and authorised recovery; transitions should be timestamped and logged.
