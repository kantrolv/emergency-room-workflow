# Emergency Room Workflow & Triage Management System
## Technical Design Document

### 1. Project Overview
This project is an enterprise-grade backend system designed to manage and simulate the complex operations of a hospital Emergency Room (ER). Unlike standard CRUD applications, this system enforces strict workflow transitions, priority-based queuing, concurrent resource management (beds), and comprehensive auditability. The architecture prioritizes correctness, scalability, and clean code principles.

### 2. Core Objectives
- **Workflow Enforcement:** Strict state machine managing patient lifecycle from registration to discharge.
- **Triage Intelligence:** Dynamic priority calculation based on vitals and injury severity using the Strategy Pattern.
- **Resource Management:** Concurrency-safe bed allocation system to prevent double-booking.
- **Audit & Compliance:** Immutable audit logs for every state change and critical action.
- **Role-Based Access:** Granular permission control for Doctors, Nurses, Receptionists, and Admins.

### 3. Architecture: Clean Architecture
The system follows the **Clean Architecture** strict separation of concerns:

1.  **Presentation Layer (API):**
    *   RESTful Controllers (or GraphQL Resolvers).
    *   Handles HTTP requests/responses, input validation (DTOs).
    *   *No business logic here.*

2.  **Application Layer (Use Cases):**
    *   Orchestrates the flow of data.
    *   Implements specific business Use Cases (e.g., `AdmitPatient`, `PerformTriage`, `DischargePatient`).
    *   Dependent only on the Domain Layer.

3.  **Domain Layer (Core):**
    *   Start of the dependency graph.
    *   **Entities:** `Patient`, `Triage`, `Bed`, `User`.
    *   **Value Objects:** `VitalSigns`, `SeverityLevel`.
    *   **Domain Services:** Logic that spans multiple entities (e.g., `BedAllocationService`).
    *   **Repository Interfaces:** Defitions of data access contracts.

4.  **Infrastructure Layer:**
    *   Implementations of Repository Interfaces (e.g., `PostgresPatientRepository`).
    *   External services (e.g., Email Service, SMS).
    *   Database configuration and migrations.

### 4. Authentication & Security
- **JWT (JSON Web Tokens):** Stateless authentication.
- **RBAC (Role-Based Access Control):** Middleware to enforce permissions.
    - `ADMIN`: System configuration, user management, full audit access.
    - `DOCTOR`: View patient history, assign treatments, admit/discharge, request beds.
    - `NURSE`: Triage patients, record vitals, view queue.
    - `RECEPTIONIST`: Register patients, update basic info.

### 5. Patient Lifecycle (State Machine)
The core of the system is the Patient State Machine. Transitions are strictly validated.
*   `REGISTERED`: Initial state upon entry.
*   `TRIAGED`: Vitals taken, severity assigned.
*   `WAITING_FOR_DOCTOR`: Queued based on priority.
*   `UNDER_TREATMENT`: Doctor assigned and assessment begun.
*   `ADMITTED`: Moved to a bed for longer stay.
*   `DISCHARGED`: Treatment complete, released.
*   `CLOSED`: Case file closed.

### 6. Subsystems

#### A. Triage Priority Engine
*   **Pattern:** Strategy Pattern.
*   **Logic:** Calculates a `PriorityScore` based on:
    1.  **Severity Level:** (CRITICAL > HIGH > MEDIUM > LOW).
    2.  **Wait Time:** Time elapsed since arrival.
    3.  **Vital Signs:** Abnormal vitals escalates weight.
*   **Dynamic Re-ordering:** The queue is a Priority Queue that re-sorts automatically when a patient's status changes.

#### B. Bed Allocation (Concurrency Control)
*   **Challenge:** Preventing two doctors from booking the same bed simultaneously.
*   **Solution:** Optimistic Locking or Database constraints (SELECT FOR UPDATE).
*   **Flow:**
    1.  Check availability.
    2.  Lock bed row.
    3.  Assign patient.
    4.  Update Bed status to `OCCUPIED`.
    5.  Commit transaction.

#### C. Notification System
*   **Event-Driven:** Uses an internal Event Bus.
*   **Triggers:**
    - High-severity patient arrival -> Notify Dept Head.
    - Lab results ready -> Notify Doctor.
    - Wait time threshold exceeded -> Notify Triage Nurse.

#### D. Audit Logging
*   **middleware/inteceptor:** Intercepts all modifying requests.
*   **Data:** `ActorID`, `Action`, `Timestamp`, `ResourceId`, `OldValue`, `NewValue`.
*   **Storage:** Write-only append log in database.

### 7. Database Design (Normalized)
*   **Users/Roles:** Standard identity management.
*   **Patients:** Core entity.
*   **TriageRecords:** Historical vitals and assessments (One-to-Many with Patient).
*   **Beds:** Manages physical resources.
*   **Admissions:** Links Patient <-> Bed <-> Doctor.
*   **Treatments:** Clinical notes and procedures.
*   **AuditLogs:** Compliance tracking.

### 8. Technology Stack (Recommended)
*   **Languge:** TypeScript (Node.js).
*   **Framework:** NestJS (Node) or Spring Boot (Java) - ideal for Clean Arch.
*   **Database:** PostgreSQL (Relational, ACID compliance).
*   **ORM:** TypeORM or Prisma / Hibernate.
*   **Caching:** Redis (for Queue/Bed status).
*   **Containerization:** Docker.

### 9. Future Roadmap
*   **Microservices:** Split Auth, Patient, and Billing into separate services.
*   **Real-time Dashboard:** WebSockets for live ER view.
*   **AI Triage:** Machine learning model to suggest severity based on symptoms.
