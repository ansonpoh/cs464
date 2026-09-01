# AI-Assisted Emergency Detection & Coordination Platform

> **Project Status: Planning / Pre-Development**
>
> This project has **not yet been implemented**. The architecture, technologies, workflows, and features described in this README represent the current design direction and proposed MVP. They are subject to change as development begins.

## Overview

This project proposes an **AI-assisted emergency detection and coordination platform** that uses camera feeds and computer vision to identify possible emergency situations, present them to a human operator for verification, and support coordination between emergency services and community organisations.

The system is intentionally designed as a **decision-support platform**, not an autonomous emergency dispatch system.

The intended workflow is:

```text
Camera / Video
      ↓
AI Detection
      ↓
Incident Candidate
      ↓
Human Verification
      ↓
Incident Management
      ↓
Response Recommendation
      ↓
Emergency / Community Coordination
```

AI is responsible for detecting and assessing possible incidents.

A human operator remains responsible for deciding whether an incident is genuine before any response workflow proceeds.

---

## Core Idea

The platform separates emergency intelligence into four layers:

```text
PERCEPTION
"What does the camera see?"
        ↓
UNDERSTANDING
"What appears to be happening?"
        ↓
DECISION SUPPORT
"Is this likely an emergency and how severe is it?"
        ↓
COORDINATION
"Who should respond and what resources are needed?"
```

For example:

```text
Camera
  ↓
Person detected
  ↓
Person suddenly falls
  ↓
Person remains motionless
  ↓
Possible medical emergency
Confidence: 91%
  ↓
Human operator verifies
  ↓
Response recommendation generated
  ↓
Incident tracked until resolution
```

This separation is intended to prevent the project from becoming simply a collection of computer-vision detectors.

Computer vision acts as the platform's **sensory layer**, while incident reasoning, human verification, and response coordination form the broader system.

---

## Goals

The proposed platform aims to:

* Detect possible emergencies from camera or video sources.
* Combine multiple AI observations into meaningful incident candidates.
* Estimate incident confidence and severity.
* Reduce false alarms through temporal and multi-camera validation.
* Keep humans responsible for confirming AI-generated alerts.
* Maintain a common incident record throughout the response lifecycle.
* Recommend appropriate emergency and community responders.
* Coordinate information between responding organisations.
* Maintain an auditable history of incident decisions and updates.

---

## Non-Goals

At least initially, this project is **not intended to**:

* Automatically dispatch emergency services without human confirmation.
* Replace emergency operators or first responders.
* Treat AI classifications as confirmed facts.
* Build detectors for every possible emergency.
* Integrate directly with real emergency-service infrastructure during the prototype stage.
* Provide production-grade emergency response capabilities in the first version.

The initial goal is to demonstrate the complete emergency-management workflow with a small number of well-supported incident types.

---

# Proposed MVP

The first version should remain intentionally small.

Rather than building the complete architecture immediately, the proposed MVP is:

```text
Camera / Recorded Video
        ↓
Python AI Detection
        ↓
FastAPI Backend
        ↓
PostgreSQL
        ↓
Web Dashboard
        ↓
Operator Verification
        ↓
Notification / Coordination Service
        ↓
Simulated Responders
```

The responder integrations may initially be simulated for organisations such as:

* Police
* Ambulance
* Fire Department
* Red Cross / community organisations

### Initial Incident Types

The MVP is expected to focus on approximately three incident categories:

```text
🔥 Fire / Smoke

🤕 Person Falling / Possible Medical Emergency

🚗 Vehicle Collision
```

Supporting a small number of incident types allows development to focus on the complete workflow instead of producing many low-quality detectors.

---

# Proposed System Architecture

The long-term architecture may evolve toward the following model:

```text
┌───────────────────────────────────────┐
│            CAMERA SOURCES             │
│ CCTV | Traffic | Buildings | Drones  │
└───────────────────┬───────────────────┘
                    ↓
┌───────────────────────────────────────┐
│         VIDEO INGESTION LAYER         │
│ RTSP | ONVIF | WebRTC                 │
│ Stream Gateway | Frame Sampling       │
└───────────────────┬───────────────────┘
                    ↓
┌───────────────────────────────────────┐
│          AI DETECTION LAYER           │
│                                       │
│ Object Detection                      │
│ Action Recognition                    │
│ Scene Analysis                        │
│                                       │
│             ↓                         │
│        Event Fusion                   │
│             ↓                         │
│ Confidence / Risk Score               │
└───────────────────┬───────────────────┘
                    ↓
┌───────────────────────────────────────┐
│       EVENT PROCESSING / TRIAGE       │
│                                       │
│ Deduplication                         │
│ Severity Classification               │
│ Location Resolution                   │
│ Temporal Validation                   │
│ Nearby-Camera Correlation             │
└───────────────────┬───────────────────┘
                    ↓
┌───────────────────────────────────────┐
│       HUMAN VERIFICATION LAYER        │
│                                       │
│ Live Feed                             │
│ Incident Clip                         │
│ Location                              │
│ AI Classification                     │
│ Confidence                            │
│ Response Recommendation               │
│                                       │
│ [ CONFIRM ] [ FALSE ALARM ]           │
│ [ MONITOR ] [ ESCALATE ]              │
└───────────────────┬───────────────────┘
                    ↓
┌───────────────────────────────────────┐
│       INCIDENT MANAGEMENT LAYER       │
│                                       │
│ Incident Record                       │
│ Priority                              │
│ Required Resources                    │
│ Incident State                        │
│ Audit Trail                           │
└───────────────────┬───────────────────┘
                    ↓
┌───────────────────────────────────────┐
│        RESPONSE COORDINATION          │
│                                       │
│ Emergency Services                    │
│ NGOs / Community Organisations        │
│ Resource Requests                     │
│ Status Updates                        │
│ Communication                         │
└───────────────────────────────────────┘
```

This represents a possible long-term architecture rather than the initial implementation target.

---

# AI Detection Strategy

The current design does not assume that one large model will identify every emergency.

Instead, the system may use specialised models or services.

```text
                     Video
                       │
        ┌──────────────┼───────────────┐
        ↓              ↓               ↓
 Fire Detection   Human Activity   Vehicle Analysis
        │              │               │
 smoke/flame       fall/fight       crash/stopped
        │              │               │
        └──────────────┼───────────────┘
                       ↓
                 Event Fusion
```

Possible detector categories include:

| Detector             | Possible Outputs                    |
| -------------------- | ----------------------------------- |
| Fire model           | Smoke, flames, spreading fire       |
| Human activity model | Fall, fight, collapse               |
| Vehicle model        | Crash, rollover, stopped vehicle    |
| Flood model          | Rising water, submerged road        |
| Crowd model          | Unusual or dangerous crowd movement |
| Object model         | Person, vehicle, motorcycle, debris |

Not all of these detectors are expected to exist in the MVP.

---

## Event Fusion

Individual detections do not necessarily represent emergencies.

For example:

```text
Vehicle A moving quickly
        +
Vehicle B moving quickly
        ↓
Trajectories intersect
        ↓
Both vehicles suddenly stop
        ↓
Debris appears
        ↓
Occupants leave vehicles
        ↓
Possible Vehicle Collision
Confidence: High
```

An **event fusion layer** would combine observations from one or more models and determine whether they collectively indicate a meaningful incident.

This is expected to become one of the project's most important components.

---

# Incident Candidates

AI detections should eventually be normalised into a common internal representation.

A conceptual incident candidate could look like:

```json
{
  "event_id": "evt_98213",
  "type": "vehicle_collision",
  "confidence": 0.94,
  "severity": "high",

  "location": {
    "latitude": 1.3521,
    "longitude": 103.8198,
    "camera_id": "CAM_281"
  },

  "detections": [
    "vehicle_collision",
    "person_on_ground",
    "road_obstruction"
  ],

  "evidence": {
    "clip": "...",
    "snapshot": "...",
    "camera_ids": [
      "CAM_281",
      "CAM_282"
    ]
  },

  "timestamp": "..."
}
```

The exact schema has **not yet been finalised**.

A standard incident format would allow downstream services to process fires, collisions, falls, floods, and other emergencies using the same interface.

---

# Human-in-the-Loop Verification

Human verification is a fundamental design requirement.

The AI should generate statements such as:

```text
Possible Fire

Confidence: 89%
Location: Building 17

Evidence:
- Smoke detected
- Flames detected

[ LIVE VIDEO ]

[ CONFIRM ]
[ FALSE ALARM ]
[ MONITOR ]
```

The platform should **not** treat this as a confirmed fire until an authorised operator verifies it.

This protects the system from situations such as:

* Steam being classified as smoke.
* Play fighting being classified as assault.
* Someone resting being classified as a medical emergency.
* Heavy rain being classified as flooding.

The AI should effectively communicate:

> A possible emergency has been detected. Please verify.

rather than autonomously ordering a response.

---

# Incident Orchestration

After an incident is confirmed, a separate orchestration layer may determine which organisations and resources should be notified.

For example:

```text
INCIDENT
Major Vehicle Collision
Possible Multiple Casualties
        ↓
Response Rules
        ↓
Police              YES
Ambulance           YES
Fire / Rescue       MAYBE
Traffic Management  YES
Hospital Alert      YES
Community NGO       MAYBE
```

Initial response policies should preferably remain **deterministic, transparent, and auditable**.

Conceptually:

```text
IF incident == fire
    recommend Fire Department

IF fire.severity >= HIGH
    recommend Ambulance

IF evacuation_required == TRUE
    recommend Civil Defence
    recommend appropriate NGO support

IF incident == vehicle_collision
    recommend Police
    recommend Ambulance

IF trapped_person == TRUE
    recommend Fire / Rescue
```

AI may eventually assist with resource recommendations, but final response policy should remain understandable and reviewable.

---

# Community & NGO Coordination

A longer-term goal is to support more than traditional emergency services.

Community organisations may have useful capabilities during large incidents such as floods, fires, evacuations, or disasters.

Example capability profiles could include:

```text
Humanitarian Organisation
- First aid
- Shelter management
- Food distribution
- Volunteers

Community Rescue Team
- Boats
- Flood rescue
- Search and rescue

Food Relief Organisation
- Meal preparation
- Drinking water distribution
```

The system could compare incident needs against available capabilities.

For example:

```text
Flood affecting approximately 1,200 residents

Identified Needs
- Evacuation assistance
- Temporary shelter
- Food
- Drinking water

Suggested Resources

Civil Defence
→ Evacuation

Humanitarian Organisation
→ Shelter

Community Rescue Team
→ Boats

Food Relief Organisation
→ Meals and water
```

This creates a potential **resource-matching problem** that may become a major part of the platform.

---

# Operator Dashboard

The eventual interface is expected to be map-centric and provide a common operating picture.

A conceptual view might include:

```text
┌───────────────────────────────────────────────┐
│ INCIDENT COMMAND                              │
├───────────────────────────────────────────────┤
│                                               │
│                    MAP                        │
│                                               │
│        Ambulance ─────→                       │
│                                               │
│                INCIDENT                       │
│                                               │
│                 ←──── Fire Unit               │
│                                               │
├───────────────────────────────────────────────┤
│ INCIDENT                                      │
│ Warehouse Fire                                │
│ Severity: CRITICAL                            │
│                                               │
│ Detected: 13:21                               │
│ Confirmed: 13:22                              │
│                                               │
│ Responding                                    │
│ Fire Unit             ETA 4 min               │
│ Ambulance             ETA 6 min               │
│ Police                ETA 5 min               │
│ Community Team        Preparing               │
└───────────────────────────────────────────────┘
```

Potential dashboard functionality includes:

* Incident location.
* Live or recorded camera evidence.
* AI-generated classification.
* Confidence score.
* Severity.
* Operator verification controls.
* Responding organisations.
* Estimated arrival times.
* Resource requirements.
* Incident status.
* Communication and updates.

None of these interfaces have been implemented yet.

---

# Edge Processing

For larger deployments, continuously sending every full-resolution camera stream to central infrastructure may be unnecessarily expensive.

A future architecture may therefore use edge inference:

```text
CCTV
  ↓
Edge AI Device
  ↓
Lightweight Detection
  ↓
Possible Incident
  ↓
Send Event + Short Video Clip
  ↓
Central Verification / Analysis
```

This would create a two-stage process:

```text
Stage 1 — Detection

"Something unusual may have happened."

                ↓

Stage 2 — Verification

"What happened?"
"How serious is it?"
```

Possible edge hardware could include embedded AI devices, camera NPUs, or local compute servers.

Edge processing is **not required for the initial MVP**.

---

# Proposed Technical Stack

The technology stack has **not been finalised**.

Technologies currently under consideration include:

| Area                   | Possible Technology     |
| ---------------------- | ----------------------- |
| Backend API            | FastAPI, Go, or Node.js |
| AI / ML                | Python, PyTorch         |
| Object Detection       | YOLO-family models      |
| Video Processing       | FFmpeg, GStreamer       |
| Camera Protocols       | RTSP, ONVIF             |
| Browser Video          | WebRTC                  |
| Primary Database       | PostgreSQL              |
| Geospatial Data        | PostGIS                 |
| Cache                  | Redis                   |
| Messaging              | Kafka or RabbitMQ       |
| Real-Time Updates      | WebSockets              |
| Object / Video Storage | S3-compatible storage   |
| Containers             | Docker                  |
| Orchestration          | Kubernetes              |
| Monitoring             | Prometheus, Grafana     |

For the MVP, a significantly smaller stack is expected to be sufficient.

A likely starting point is:

```text
Python
FastAPI
PostgreSQL
Web Frontend
Computer Vision Model
Recorded / Simulated Camera Input
```

Technology choices will be confirmed during implementation.

---

# Event-Driven Architecture

A future distributed version of the platform may use an event-streaming system.

Conceptually:

```text
camera.events
      ↓
ai.detections
      ↓
incident.candidates
      ↓
incident.verified
      ↓
dispatch.requests
      ↓
response.status
```

Individual services could subscribe only to the events relevant to them.

Kafka, RabbitMQ, or another messaging platform may eventually support this architecture.

This is not expected to be necessary for the earliest prototype.

---

# Possible Service Architecture

If the platform eventually grows beyond the MVP, it may be separated into services such as:

```text
                        API Gateway
                             │
        ┌────────────────────┼───────────────────┐
        │                    │                   │
        ↓                    ↓                   ↓
 Camera Service       Incident Service        Auth
        │                    │
        ↓                    ↓
 Video Gateway         Event Service
        │                    │
        ↓                    ↓
 AI Services ───────→ Event Fusion
                             │
                             ↓
                   Verification Service
                             │
                             ↓
                     Response Service
                             │
             ┌───────────────┼───────────────┐
             ↓               ↓               ↓
           Police        Ambulance          Fire
                             │
                             ↓
                       NGO Service
                             │
                             ↓
                    Notification Service
```

This diagram represents a possible long-term design.

The project should **not begin as a large microservice system** unless development requirements justify it.

---

# Development Roadmap

## Phase 0 — Planning

Current phase.

* Define functional requirements.
* Define MVP scope.
* Select initial incident categories.
* Finalise technology choices.
* Define data models.
* Design operator workflow.
* Identify suitable datasets or test videos.
* Define evaluation criteria.

## Phase 1 — Detection Prototype

Planned:

* Process recorded video input.
* Implement the first incident detector.
* Generate structured detection events.
* Store basic event metadata.
* Capture relevant evidence frames or clips.

## Phase 2 — Incident Backend

Planned:

* Implement backend API.
* Define incident candidate schema.
* Store incident records.
* Support incident status changes.
* Add confidence and severity fields.

## Phase 3 — Verification Dashboard

Planned:

* Display incident candidates.
* Show supporting video evidence.
* Allow operators to confirm incidents.
* Allow operators to reject false alarms.
* Track operator actions.

## Phase 4 — Response Recommendation

Planned:

* Add deterministic response rules.
* Associate incident categories with responder types.
* Display recommended responders.
* Simulate notifications and response acknowledgements.

## Phase 5 — Coordination

Planned:

* Add incident map.
* Track responding resources.
* Add incident timeline.
* Add status updates.
* Explore NGO capability matching.

## Future Work

Potential future development may include:

* Multiple-camera correlation.
* Edge AI inference.
* Real-time camera streaming.
* Additional incident detectors.
* Event fusion.
* Geospatial resource matching.
* Multi-organisation coordination.
* Production authentication and access control.
* Integration with external systems.
* Distributed event processing.
* Advanced monitoring and observability.

---

# Key Engineering Challenges

Several problems need to be treated as first-class design concerns from the beginning.

### False Positives

Emergency systems cannot assume that every AI prediction is correct.

Confidence thresholds, temporal validation, multi-model reasoning, and human verification will be important.

### Privacy

Camera footage may contain highly sensitive information.

The project will need clear policies around:

* Data access.
* Video retention.
* Authentication.
* Authorisation.
* Logging.
* Storage.
* Sharing of incident evidence.

### Latency

Emergency detection is time-sensitive.

The system should eventually minimise the time between:

```text
Incident
   ↓
Detection
   ↓
Verification
   ↓
Response
```

### Camera Reliability

The platform must eventually account for:

* Offline cameras.
* Corrupted streams.
* Obstructed cameras.
* Poor visibility.
* Network failures.

### Duplicate Incidents

The same emergency may appear on several nearby cameras.

The platform should eventually correlate these observations instead of creating multiple unrelated incidents.

### Auditability

Important actions should be traceable.

Examples include:

* What the AI detected.
* Which evidence was available.
* What confidence was assigned.
* Who verified the incident.
* What response was recommended.
* Which organisations were notified.
* How the incident status changed.

### AI vs Confirmed Facts

The interface must make a clear distinction between:

```text
AI Prediction:
"Possible vehicle collision"
```

and:

```text
Operator Verified:
"Vehicle collision confirmed"
```

An AI-generated classification must never be presented as though it has already been independently confirmed.

---

# Repository Structure

The repository structure has not yet been established.

A possible structure may eventually look similar to:

```text
.
├── backend/
├── frontend/
├── ai/
├── database/
├── tests/
├── docs/
├── infrastructure/
└── README.md
```

This is only a placeholder and should evolve based on the implementation.

---

# Installation

There is currently nothing to install.

The project is in the **planning stage** and no application code has been implemented yet.

Installation and development instructions will be added once the initial project structure exists.

---

# Running the Project

The project cannot currently be run because implementation has not started.

Future instructions will cover topics such as:

```text
Environment setup
Database configuration
Model setup
Backend startup
Frontend startup
Video input configuration
Testing
```

---

# Testing

No automated test suite currently exists.

Testing strategy will be defined alongside implementation and is expected to eventually cover:

* Backend unit tests.
* API integration tests.
* Incident workflow tests.
* AI model evaluation.
* False-positive testing.
* Failure scenarios.
* Permission and access-control tests.
* End-to-end verification and coordination flows.

---

# Current Status

```text
Requirements / Architecture     ██████████  In Progress
Repository Structure            ░░░░░░░░░░  Not Started
Backend                         ░░░░░░░░░░  Not Started
Frontend                        ░░░░░░░░░░  Not Started
AI Detection                    ░░░░░░░░░░  Not Started
Database                        ░░░░░░░░░░  Not Started
Verification Workflow           ░░░░░░░░░░  Not Started
Response Coordination           ░░░░░░░░░░  Not Started
External Integrations           ░░░░░░░░░░  Not Started
```

---

# Disclaimer

This project is currently an experimental / prototype concept.

It must **not** be used as a substitute for existing emergency communication, dispatch, or response systems.

AI-generated incident classifications may be incorrect. Any real-world implementation involving emergency response would require appropriate human oversight, security controls, regulatory review, operational testing, reliability engineering, and formal integration agreements with participating organisations.

---

## Summary

The project is intended to become more than:

```text
CCTV + Object Detection + Notifications
```

The proposed system is:

```text
Computer Vision
      +
Event Understanding
      +
Human Verification
      +
Incident Management
      +
Response Coordination
```

The core principle is simple:

> **AI detects and assists. Humans verify. The platform coordinates.**
