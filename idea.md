# Emergency Room Workflow & Triage Management System
## Technical Design Document (Node.js + MongoDB)

### 1. Project Overview
This project is an enterprise-grade backend system designed to manage and simulate the complex operations of a hospital Emergency Room (ER). Unlike standard CRUD applications, this system enforces strict workflow transitions, priority-based queuing, concurrent resource management (beds), and comprehensive auditability. The architecture prioritizes correctness, scalability, and clean code principles, leveraging **Node.js, TypeScript, and MongoDB**.

### 2. Core Objectives
-   **Workflow Enforcement:** Strict state machine managing patient lifecycle (e.g., `REGISTERED` -> `TRIAGED` -> `ADMITTED`).
-   **Triage Intelligence:** Dynamic priority calculation based on vitals and injury severity using the Strategy Pattern.
-   **Resource Management:** Concurrency-safe bed allocation system using **MongoDB Sessions/Transactions** to prevent double-booking.
-   **Audit & Compliance:** Immutable audit logs for every state change and critical action.
-   **Role-Based Access:** Granular permission control via Middleware.

### 3. Architecture: Clean Architecture (Strict)
The system follows the **Clean Architecture** strict separation of concerns:

1.  **Presentation Layer (API):**
    *   RESTful Controllers (Express/NestJS).
    *   DTO validation (using `class-validator`/`zod`).
    *   **Rule:** No business logic here. Dispatches commands to Application Layer.

2.  **Application Layer (Use Cases):**
    *   Orchestrates the flow of data.
    *   Implements specific business Use Cases (e.g., `PerformTriageUseCase`, `AdmitPatientUseCase`, `DischargePatientUseCase`).
    *   Defines abstract `Repository` interfaces.
    *   **Rule:** Pure TypeScript. No dependencies on database drivers (mongoose) or HTTP frameworks.

3.  **Domain Layer (Core):**
    *   **Entities:** `Patient`, `Bed`, `User`, `Admission`, `Treatment`.
    *   **Value Objects:** `VitalSigns`, `SeverityLevel`, `PriorityScore`.
    *   **Domain Services:** `BedAllocationService` (manages logic spanning multiple entities), `TriagePriorityService`.
    *   **State Machine:** Logic to validate transitions (e.g., prevent `REGISTERED` -> `DISCHARGED` directly).

4.  **Infrastructure Layer:**
    *   **Persistence:** Mongoose implementations of Repository interfaces.
    *   **Concurrency:** MongoDB Transaction Managers.
    *   **External:** Event Bus (Node `EventEmitter` or Redis), JWT Service.
    *   **Middleware:** Audit Logging Interceptors.

### 4. Authentication & Security
-   **JWT (JSON Web Tokens):** Access + Refresh Token rotation.
-   **RBAC (Role-Based Access Control):**
    -   `ADMIN`: Configuration, User Management.
    -   `DOCTOR`: Treatments, Admissions, Discharges.
    -   `NURSE`: Triage, Vitals, Monitoring.
    -   `RECEPTIONIST`: Registration, Contact Updates.

### 5. Patient Lifecycle (State Machine)
The Domain Layer enforces these strict transitions:
*   `REGISTERED`: Initial state.
*   `TRIAGED`: Vitals recorded, severity assigned.
*   `WAITING_FOR_DOCTOR`: Queued by priority.
*   `UNDER_TREATMENT`: Doctor assigned.
*   `ADMITTED`: Moved to a bed (requires Transaction).
*   `DISCHARGED`: Released.
*   `CLOSED`: Case finalized.

### 6. Subsystems

#### A. Triage Priority Engine
*   **Strategy Pattern:** Priority = f(Severity, WaitTime, VitalAbnormality).
*   **Implementation:** 
    *   Queue stored in MongoDB (indexed by `status` + `priorityScore`) or Redis.
    *   Re-calculation triggers on Vital updates.

#### B. Bed Allocation (Concurrency Control)
*   **Challenge:** Prevention of race conditions (Double Booking).
*   **Solution:** **MongoDB Transactions** (Replica Set required).
    1.  Start Client Session.
    2.  `StartTransaction()`.
    3.  Find available Bed (`lock` logic or atomic update).
    4.  Create `Admission` record.
    5.  Update Patient State -> `ADMITTED`.
    6.  `CommitTransaction()`.
    7.  On Error: `AbortTransaction()`.

#### C. Notification System
*   **Event-Driven:** Internal Event Bus.
*   **Events:** `PatientEscalated`, `BedAllocated`, `HighSeverityArrival`.
*   **Handlers:** Log to DB, Send Alert (Console/Socket).

#### D. Audit Logging
*   **Interceptor:** Captures every specialized Command/Mutation.
*   **Schema:** `{ actorId, action, targetId, oldState, newState, timestamp }`.
*   **Storage:** Append-only MongoDB Collection (`audit_logs`).

### 7. Database Design (MongoDB)
Designed for Document-Orientation but structured for integrity.
*   `users`: Auth and Profile.
*   `patients`: Core Profile + Status.
*   `triage_records`: **Referenced** (High volume, usually immutable).
*   `beds`: Resource tracking.
*   `treatments`: **Embedded** in Patient OR Referenced (depending on size).
*   `audit_logs`: Separate collection (High write volume).

### 8. Technology Stack
*   **Runtime:** Node.js
*   **Language:** TypeScript
*   **Framework:** NestJS (Recommended for Clean Arch) or Express.
*   **Database:** MongoDB v6+ (Mongoose ODM).
*   **Validation:** `zod` or `class-validator`.
*   **Testing:** Jest.

### 9. Future Roadmap
*   **WebSocket:** Real-time dashboard for "Waiting Room".
*   **Microservices:** Split `Auth` and `Billing` services.
*   **AI:** Predictive triage scoring.
