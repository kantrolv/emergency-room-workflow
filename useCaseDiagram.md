# Use Case Diagram

```mermaid
usecaseDiagram
    actor Receptionist
    actor Nurse
    actor Doctor
    actor Admin
    actor System

    package "Patient Management" {
        usecase "Register Patient" as UC1
        usecase "Update Patient Info" as UC2
        usecase "View Patient History" as UC3
    }

    package "Triage & Vitals" {
        usecase "Perform Triage" as UC4
        usecase "Record Vitals" as UC5
        usecase "Escalate Priority" as UC6
    }

    package "Treatment & Workflow" {
        usecase "Assign Doctor" as UC7
        usecase "Perform Treatment" as UC8
        usecase "Request Admission" as UC9
        usecase "Discharge Patient" as UC10
    }

    package "Resource Management" {
        usecase "Allocate Bed" as UC11
        usecase "Release Bed" as UC12
        usecase "Manage Beds" as UC13
    }

    package "Administration" {
        usecase "Manage Users" as UC14
        usecase "View Audit Logs" as UC15
        usecase "Generate Reports" as UC16
    }
    
    Receptionist --> UC1
    Receptionist --> UC2
    
    Nurse --> UC4
    Nurse --> UC5
    Nurse --> UC3
    
    Doctor --> UC3
    Doctor --> UC7
    Doctor --> UC8
    Doctor --> UC9
    Doctor --> UC10

    Admin --> UC13
    Admin --> UC14
    Admin --> UC15
    Admin --> UC16

    System --> UC6 : Auto-escalate based on time
    
    UC9 ..> UC11 : <<include>>
    UC10 ..> UC12 : <<include>>
```
