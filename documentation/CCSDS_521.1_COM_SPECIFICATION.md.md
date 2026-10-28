# CCSDS 521.1-B-1 & ISO 18202 Mission Operations Common Object Model (COM) Specification

## Executive Summary

This document provides the technical architecture and specification for **MOCOMObjectForge Pro**, an industrial workstation implementation of the **Consultative Committee for Space Data Systems (CCSDS) 521.1-B-1** Recommended Standard (*Mission Operations Common Object Model*, Blue Book, Issue 1, February 2014\) and **ISO 18202**.

While the Message Abstraction Layer (MAL, CCSDS 521.0-B-2) specifies service interaction patterns, the **Common Object Model (COM)** establishes the standardized data model and support services for all Mission Operations services. COM defines how spacecraft telemetry, commanding activities, events, and ground operations are uniquely identified, linked by causal provenance, persisted, and queried.

---

## 1\. Architectural Model & Core Services

\+-------------------------------------------------------------------------+

\|                  Mission Application & Control Layer                    \|

\|             (Telemetry Displays, Procedure Engines, Planners)           \|

\+-------------------------------------------------------------------------+

                                    \|

\+-----------------------------------+-------------------------------------+

\|         CCSDS 521.1-B-1 Common Object Model (COM) Support Services      \|

\|  \+--------------------+  \+--------------------+  \+--------------------+ \|

\|  \|    Event Service   \|  \|   Archive Service  \|  \|Activity Tracking Svc\|

\|  \|   (Severity/PubSub)\|  \|     (CRUD Store)   \|  \| (5-Stage Lifecycle)\| \|

\|  \+--------------------+  \+--------------------+  \+--------------------+ \|

\+-------------------------------------------------------------------------+

                                    \|

\+-------------------------------------------------------------------------+

\|                    Generic Object Model Core (§2)                       \|

\|       ObjectId: (Domain, Area, Service, Version, Number, InstanceId)    \|

\|       ObjectKey: (Domain, InstanceId) • Source Provenance Links         \|

\+-------------------------------------------------------------------------+

                                    \|

\+-------------------------------------------------------------------------+

\|                Message Abstraction Layer (MAL) CCSDS 521.0              \|

\+-------------------------------------------------------------------------+

---

## 2\. Generic Object Model & Identity (§2)

Every mission artifact (telemetry value, event, command activity) is modeled as a COM Object:

- **`ObjectId`**: Complete globally unique identifier composed of:  
  - `domain`: Hierarchical namespace (e.g. `Mission.Lunar.Gateway.Power`)  
  - `type`: `(area, service, version, number)`  
  - `instanceId`: Unique 64-bit integer  
- **`ObjectKey`**: Compact tuple of `(domain, instanceId)`  
- **`sourceObjectId`**: Explicit causal link connecting a telemetry update or event to the specific command activity that generated it.

---

## 3\. Event Service Specification (§3)

Provides standardized event generation, distribution, and publication over MAL PUB/SUB:

- **Severity Classification**:  
  1. `INFORMATIONAL`: Routine state transitions and nominal status milestones  
  2. `WARNING`: Minor threshold excursions and pre-limit notifications  
  3. `ALARM`: Actionable operational alarms requiring flight controller attention  
  4. `CRITICAL`: Immediate spacecraft safety threats or critical hardware trip triggers  
- **Multi-Domain Subscriber Filtering**: Subscribers filter events based on domain hierarchy prefixes and minimum severity thresholds.

---

## 4\. Archive Service Specification (§4)

Provides standardized persistence for all COM Objects:

- **`store`**: Persists objects with auto-incrementing or explicit instance IDs.  
- **`retrieve`**: Fetches objects by `ObjectKey`.  
- **`query`**: Evaluates composite filters: domain matching, object type, temporal windows (`startTime` to `endTime`), and source activity links.  
- **`count`**: Returns the count of matching records.  
- **`delete`**: Atomic deletion of records enforcing data retention policies.

---

## 5\. Activity Tracking Service Specification (§5)

Tracks the execution lifecycle of telecommands and automated procedures across 5 normative stages:

1. `RELEASED`: Command released from ground control queue.  
2. `UPLINKED`: Command radiated and acknowledged by ground antenna.  
3. `EXECUTING`: Flight software initiates on-board subsystem actuation.  
4. `COMPLETED`: Successful operation confirmed by telemetry feedback.  
5. `FAILED`: Execution aborted, timed out, or rejected with error code.