# System Integration & Architecture (SIA) Documentation
## PetStay: Enterprise Veterinary Clinic Management & Security Platform

**Academic Course**: IT/CS-SIA312 – System Integration & Architecture  
**Evaluator**: CICTE Faculty / Architecture Board  
**Target System**: PetStay Enterprise Veterinary Clinic Management & Security Platform  
**System Classification**: Distributed Multi-Tier Mobile & Enterprise Web Architecture  

---

### Executive Architectural Overview

PetStay is an integrated multi-tier veterinary clinic management and security system designed for seamless collaboration among Pet Owners, Clinic Staff (Veterinarians and Veterinary Technicians), and Clinic Administrators. The architecture couples a high-performance **Single-Page Application (SPA)** client tier (responsive dynamic mobile and desktop shell) with a cloud-native **Database-as-a-Service (DBaaS)** PostgreSQL backend via RESTful APIs, enforced by Information Assurance and Security (IAS 1) protocols including SHA-256 cryptographic hashing, progressive lockout protection, single-use OTP tokens, session watchdog timers, and immutable audit trails.

---

### 1. Use Case Diagram (System Users & Functionalities)

The diagram below identifies the three primary system actors (**Pet Owner**, **Clinic Staff / Veterinarian**, and **Clinic Administrator**) along with the external **Payment Gateway & SMS Carrier Services**, defining their relationships with core functional and security use cases according to standard UML conventions.

```mermaid
flowchart LR
    %% Actors
    subgraph Actors["System Actors"]
        Owner(("Pet Owner\n(Client)"))
        Staff(("Clinic Staff\n(Vet / Tech)"))
        Admin(("Clinic Admin\n(Chief Administrator)"))
        Gateway["External Payment Gateway\n(GCash / Maya / Card)"]
        Carrier["SMS / Email Carrier\n(Simulated OTP Engine)"]
    end

    %% Use Cases
    subgraph UC_Auth["Authentication & Security Module"]
        UC1(["Register / Login"])
        UC2(["OTP Verification & Cooldown"])
        UC3(["Manage Password & Strength Check"])
        UC4(["Session Management & Inactivity Watchdog"])
    end

    subgraph UC_Owner["Owner Self-Service Module"]
        UC5(["Manage Pet Directory"])
        UC6(["Book 8-Hour Shift Appointment"])
        UC7(["View Clinical History & Prescriptions"])
        UC8(["Pay Invoices & Settle Bills"])
        UC9(["Print Official Receipt"])
    end

    subgraph UC_Staff["Clinical Operations Module"]
        UC10(["Triage & Active Patient Queue"])
        UC11(["Record Clinical SOAP Notes & Vitals"])
        UC12(["Manage Pharmacy & Inventory"])
        UC13(["Discharge Patient & Generate Bill"])
        UC14(["Submit Shift Swap & Fee Waiver"])
    end

    subgraph UC_Admin["System Governance & Auditing Module"]
        UC15(["User CRUD & Role Assignment"])
        UC16(["Review & Approve Staff Requests"])
        UC17(["Monitor Clinic Analytics & Charts"])
        UC18(["Inspect Security Audit Trail"])
        UC19(["Backup & Restore Database"])
    end

    %% Actor Relationships
    Owner --> UC1
    Owner --> UC2
    Owner --> UC3
    Owner --> UC4
    Owner --> UC5
    Owner --> UC6
    Owner --> UC7
    Owner --> UC8
    Owner --> UC9

    Staff --> UC1
    Staff --> UC3
    Staff --> UC4
    Staff --> UC10
    Staff --> UC11
    Staff --> UC12
    Staff --> UC13
    Staff --> UC14

    Admin --> UC1
    Admin --> UC3
    Admin --> UC4
    Admin --> UC12
    Admin --> UC15
    Admin --> UC16
    Admin --> UC17
    Admin --> UC18
    Admin --> UC19

    %% External Systems
    UC8 -.-> Gateway
    UC2 -.-> Carrier
    UC13 --> UC8
```

---

### 2. System Flow Diagram (Swimlanes & Pools)

The Swimlane Process Flow Diagram models the end-to-end clinical lifecycle—from appointment booking through veterinary triage, SOAP documentation, discharge invoicing, customer payment, and administrative audit logging.

```mermaid
flowchart TD
    subgraph Pool_PetStay["PetStay Clinic End-to-End Operational Lifecycle"]
        
        subgraph Lane_Owner["Pet Owner Lane"]
            O_Start(["Start: Owner Logs In"])
            O_Book["Select Pet & Schedule 8-Hour Shift Slot\n(08:00 AM - 05:00 PM)"]
            O_Wait["Wait for Clinic Day Arrival"]
            O_Pay["Receive Discharge Bill & Settle via GCash/Maya/Card"]
            O_Rcpt["Download / Print Official Receipt"]
            O_End(["End: Patient Returned Home"])
        end

        subgraph Lane_Staff["Clinic Staff / Veterinarian Lane"]
            S_Queue["Review Clinic Queue & Admit Patient"]
            S_Exam["Perform Clinical Physical Examination"]
            S_SOAP["Record SOAP Note:\n(Subjective, Objective, Assessment, Plan)"]
            S_Inv["Dispense Medications & Deduct Inventory"]
            S_Discharge["Initiate Patient Discharge & Issue Invoice"]
        end

        subgraph Lane_System["PetStay System / Gateway Lane"]
            Sys_Validate["Validate 8-Hour Shift & Enforce Operating Rules"]
            Sys_CreateApt["Create Reservation (Status: Pending -> Approved)"]
            Sys_UpdateInv["Update Pharmacy Stock & Low Reorder Thresholds"]
            Sys_GenInvc["Calculate Total Bill & Generate #INV-XXXXX"]
            Sys_ProcessPay["Process Transaction & Update Invoice to PAID"]
            Sys_Log["Append Immutable Security Event to Log Trail"]
        end

        subgraph Lane_Admin["Clinic Administrator Lane"]
            A_Monitor["Monitor Real-time Clinic Analytics & Capacity"]
            A_Review["Review Audit Trails & Approve Staff HR Requests"]
        end
    end

    %% Flow transitions across swimlanes
    O_Start --> O_Book
    O_Book --> Sys_Validate
    Sys_Validate --> Sys_CreateApt
    Sys_CreateApt --> O_Wait
    O_Wait --> S_Queue
    S_Queue --> S_Exam
    S_Exam --> S_SOAP
    S_SOAP --> S_Inv
    S_Inv --> Sys_UpdateInv
    Sys_UpdateInv --> S_Discharge
    S_Discharge --> Sys_GenInvc
    Sys_GenInvc --> O_Pay
    O_Pay --> Sys_ProcessPay
    Sys_ProcessPay --> Sys_Log
    Sys_ProcessPay --> O_Rcpt
    O_Rcpt --> O_End

    Sys_Log -.-> A_Review
    Sys_UpdateInv -.-> A_Monitor
```

---

### 3. Entity Relationship Diagram (ERD)

The Entity Relationship Diagram specifies the relational database schema implemented in `petstay_schema.sql` and deployed on the cloud PostgreSQL database. It captures all primary keys (`PK`), foreign keys (`FK`), attributes, and entity relationships.

```mermaid
erDiagram
    USERS ||--o{ PETS : owns
    USERS ||--o{ RESERVATIONS : books
    USERS ||--o{ MEDICAL_RECORDS : prescribes
    USERS ||--o{ INVOICES : billed_to
    USERS ||--o{ STAFF_REQUESTS : submits
    USERS ||--o{ LOG_TRAILS : triggers
    USERS ||--o{ SYSTEM_BACKUPS : generates

    PETS ||--o{ RESERVATIONS : admitted_in
    PETS ||--o{ MEDICAL_RECORDS : receives

    UNITS ||--o{ RESERVATIONS : allocated_to
    BOARDING_UNITS ||--o{ RESERVATIONS : legacy_references

    RESERVATIONS ||--o| MEDICAL_RECORDS : documents
    RESERVATIONS ||--o| INVOICES : billed_for

    USERS {
        int user_id PK
        varchar username UK
        varchar email UK
        varchar password
        varchar password_hash
        varchar full_name
        varchar role
        varchar status
        boolean is_locked
        int failed_attempts
        timestamp lockout_until
        int lockout_tier
        varchar recovery_otp
        timestamp otp_expires_at
        timestamp created_at
    }

    PETS {
        int pet_id PK
        int owner_id FK
        varchar pet_name
        varchar species
        varchar breed
        int age
        varchar gender
        decimal weight_kg
        text allergies
        text special_notes
        timestamp created_at
    }

    UNITS {
        int unit_id PK
        varchar name
        varchar type
        decimal base_rate
        int capacity
        varchar status
        text description
    }

    BOARDING_UNITS {
        int unit_id PK
        varchar unit_name
        varchar unit_type
        varchar status
    }

    RESERVATIONS {
        int reservation_id PK
        int user_id FK
        int pet_id FK
        int unit_id FK
        timestamp check_in_date
        timestamp check_out_date
        varchar status
        text triage_notes
        timestamp created_at
    }

    MEDICAL_RECORDS {
        int record_id PK
        int pet_id FK
        int vet_id FK
        int reservation_id FK
        decimal weight_kg
        decimal temperature
        text subjective
        text objective
        text assessment
        text plan
        timestamp created_at
    }

    INVENTORY {
        int item_id PK
        varchar item_name
        varchar category
        int quantity
        int reorder_level
        decimal unit_price
        varchar supplier
        timestamp last_restocked
        timestamp last_updated
    }

    INVOICES {
        int invoice_id PK
        int reservation_id FK
        int user_id FK
        decimal service_fee
        decimal supplies_fee
        decimal discount_amount
        decimal total_amount
        varchar status
        varchar payment_method
        timestamp issued_at
        timestamp paid_at
    }

    STAFF_REQUESTS {
        int request_id PK
        int requester_id FK
        varchar request_type
        varchar target_date
        decimal amount
        text reason
        varchar status
        int reviewed_by FK
        timestamp created_at
    }

    LOG_TRAILS {
        int log_id PK
        int user_id FK
        varchar username
        varchar action
        varchar module
        text details
        varchar ip_address
        varchar status
        timestamp created_at
    }

    SYSTEM_BACKUPS {
        int backup_id PK
        varchar filename
        varchar backup_type
        int record_count
        int created_by FK
        jsonb snapshot_data
        timestamp created_at
    }
```

---

### 4. System Flowchart (Operational Logic & Decisions)

The flowchart maps the algorithmic decisions within the system—including authentication validation, progressive lockout tiers, CAPTCHA challenges, 8-hour operational shift enforcement, clinical SOAP recording, and online payment settlement.

```mermaid
flowchart TD
    Start([User Initiates Action]) --> CheckAuth{Is User Authenticated?}

    %% Authentication Sub-Branch
    CheckAuth -- No --> InputCreds[/Input Username & Password/]
    InputCreds --> CheckLockout{Is Account Locked?}
    CheckLockout -- Yes --> ShowLockout[Display Countdown Timer Banner] --> Terminate([Wait for Lockout Expiry])
    CheckLockout -- No --> NeedCaptcha{Failed Attempts >= 3?}
    NeedCaptcha -- Yes --> VerifyCaptcha{Is CAPTCHA Valid?}
    VerifyCaptcha -- No --> IncFail[Increment Failed Counter & Lockout Tier] --> Terminate
    VerifyCaptcha -- Yes --> VerifyHash
    NeedCaptcha -- No --> VerifyHash{Does SHA-256 Hash Match?}
    VerifyHash -- No --> IncFail
    VerifyHash -- Yes --> ResetFail[Reset Failed Attempts & Lockout Tiers]
    ResetFail --> StartWatchdog[Start 15-Minute Session Watchdog]
    StartWatchdog --> CheckRole{Inspect User Role}

    %% Role-Based Routing
    CheckAuth -- Yes --> CheckRole
    CheckRole -- "Role = Owner" --> OwnerPortal[/Load Owner Portal Shell/]
    CheckRole -- "Role = Staff" --> StaffPortal[/Load Staff Clinic Workspace/]
    CheckRole -- "Role = Admin" --> AdminPortal[/Load Desktop Enterprise Suite/]

    %% Owner Flow
    OwnerPortal --> OwnerAction{Select Action}
    OwnerAction -- "Book Appointment" --> PickSlot[/Select Pet, Date, and Shift Slot/]
    PickSlot --> ValidateShift{Is Slot within 8:00 AM - 5:00 PM\nand Mon - Sat?}
    ValidateShift -- No --> ShiftError[Show Operating Hours Warning] --> PickSlot
    ValidateShift -- Yes --> SaveBooking[(Insert into Reservations)] --> BookingDone[Show Confirmation Alert]

    OwnerAction -- "Pay Invoices" --> ViewInvoices[(Fetch Unpaid Invoices)]
    ViewInvoices --> SelectChannel[/Choose GCash, Maya, or Card/]
    SelectChannel --> ProcessPay[(Update Invoice Status to Paid)]
    ProcessPay --> AppendLog1[(Append to Log Trails)]
    AppendLog1 --> TriggerReceipt[Render Official Printable Receipt]

    %% Staff Flow
    StaffPortal --> StaffAction{Select Action}
    StaffAction -- "Patient Triage" --> SelectPatient[(Fetch Admitted Patients)]
    SelectPatient --> WriteSOAP[/Record Vitals & SOAP Notes/]
    WriteSOAP --> SaveSOAP[(Insert into Medical Records)]
    SaveSOAP --> DischargePatient{Discharge Patient?}
    DischargePatient -- Yes --> IssueInvoice[(Insert Invoice & Mark Reservation Completed)]

    StaffAction -- "Pharmacy" --> AdjustInv[(Adjust Stock Quantities)]
    AdjustInv --> CheckLowStock{Quantity <= Reorder Level?}
    CheckLowStock -- Yes --> ShowStockAlert[Trigger Low Stock Alert Badge]

    %% Admin Flow
    AdminPortal --> AdminAction{Select Action}
    AdminAction -- "User CRUD" --> ManageUsers[(Search, Toggle Lock, or Update Roles)]
    AdminAction -- "Audit Logs" --> ViewAudit[(Query and Filter Immutable Log Trails)]
    AdminAction -- "Backup & Recovery" --> CreateBackup[(Generate JSON Snapshot & Download)]

    %% Universal Termination
    BookingDone --> End([Session Active / Idle])
    IssueInvoice --> End
    TriggerReceipt --> End
    ShowStockAlert --> End
    CreateBackup --> End
```

---

### 5. Network Diagram (Network Flow & Tier Architecture)

The Network Diagram visualizes the distributed network topology, communication boundaries, protocols, and data flow between client devices, web servers, cloud database clusters, and external third-party services.

```mermaid
flowchart TB
    subgraph Tier_Clients["Presentation Tier (Client Devices & Workstations)"]
        Client_Mobile["Pet Owner Smartphone\n(iOS / Android Web Browser)\nPort 443 HTTPS"]
        Client_Clinic["Staff Tablet & Clinic Frontdesk\n(Veterinary Triage Station)\nPort 443 HTTPS"]
        Client_Admin["Administrator Desktop PC\n(Executive Management Suite)\nPort 443 HTTPS"]
    end

    subgraph Tier_Boundary["Network Perimeter & Edge Security"]
        DNS["Authoritative DNS Resolution\n(Cloudflare / Domain Registry)"]
        WAF["Web Application Firewall & SSL Termination\n(TLS 1.3 / Anti-DDoS / Rate Limiter)"]
    end

    subgraph Tier_Web["Application & Static Web Hosting Tier"]
        WebServer["Apache Web Server / XAMPP\n(Document Root: c:\\xampp\\htdocs\\petstay)\nServes index.html, JS Engines, Assets\nPort 80 HTTP / 443 HTTPS"]
    end

    subgraph Tier_Backend["Cloud Data & Integration Tier (Supabase BaaS Cluster)"]
        API_Gateway["PostgREST API Gateway\n(Auto-generated REST Endpoints)\nJWT Authentication & Header Validation\nPort 443 HTTPS"]
        
        subgraph DB_Cluster["PostgreSQL 15 Relational Cluster"]
            DB_Master[("PostgreSQL Master Database\nTables: users, pets, reservations,\nmedical_records, inventory, invoices,\nstaff_requests, log_trails")]
            DB_Storage["Encrypted Blob Storage\n(System Snapshots & Backups)"]
        end
    end

    subgraph Tier_External["External Services Tier (Third-Party Integrations)"]
        Payment_GCash["GCash / Maya Payment Gateway\n(Simulated Merchant Settlement)\nPort 443 HTTPS API"]
        Carrier_SMS["SMS / SMTP Telecom Gateway\n(Carrier OTP Token Dispatch)\nPort 443 HTTPS API"]
        CDN_Assets["Global Edge CDN\n(Tailwind CSS, FontAwesome, Chart.js)\nPort 443 HTTPS"]
    end

    %% Network Connections
    Client_Mobile -->|HTTPS / TLS 1.3| DNS
    Client_Clinic -->|HTTPS / TLS 1.3| DNS
    Client_Admin -->|HTTPS / TLS 1.3| DNS

    DNS --> WAF
    WAF --> WebServer
    WebServer --> CDN_Assets

    Client_Mobile -.->|Direct REST API Calls / Bearer Token| API_Gateway
    Client_Clinic -.->|Direct REST API Calls / Bearer Token| API_Gateway
    Client_Admin -.->|Direct REST API Calls / Bearer Token| API_Gateway

    API_Gateway -->|Encrypted Internal Connection| DB_Master
    DB_Master --> DB_Storage

    Client_Mobile -.->|Payment Reference Verification| Payment_GCash
    API_Gateway -.->|Dispatch OTP Token| Carrier_SMS
```

---

### Architectural Consistency & Verification Matrix

| Architecture Component | Diagram 1: Use Case | Diagram 2: Swimlane | Diagram 3: ERD | Diagram 4: Flowchart | Diagram 5: Network |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **User Roles** | Owner, Staff, Admin | Owner, Staff, Admin | `users.role` | CheckRole Split | Client Devices |
| **8-Hour Shift Appointments** | UC6 | Owner & System Lanes | `reservations`, `units` | ValidateShift | Mobile/Web to API |
| **Clinical SOAP Notes** | UC11 | Staff Lane (Exam) | `medical_records` | WriteSOAP Decision | Staff Tablet to API |
| **Pharmacy / Stock Alerts** | UC12 | Staff Lane (Dispense) | `inventory` | CheckLowStock | Admin/Staff Console |
| **Billing & Payments** | UC8, UC9 | Owner & Gateway Lanes | `invoices` | ProcessPay & Receipt | Payment Gateway |
| **Security & Audit Logs** | UC18 | System Lane | `log_trails` | AppendLog1 | PostgreSQL Cluster |
| **Backup & Recovery** | UC19 | Admin Lane | `system_backups` | CreateBackup | Blob Storage |

---

### 7. Official SIA Defense Presentation Blueprint (PowerPoint .pptx)

The complete presentation artifact has been compiled into **`PetStay_SIA_Architectural_Blueprints.pptx`**, matching the official **Western Leyte College of Ormoc (CICTE)** capstone defense format in 16:9 widescreen layout (`13.33" x 7.5"`), featuring high-contrast dark-emerald styling, institutional branding, and full technical specifications across 7 slides:

1. **Slide 1: Cover Slide** — Western Leyte College of Ormoc, CICTE Header, PetStay Capstone Title, Evaluators (Mr. Roel Daniot & Mr. Ian Celmar Avila), and 5-Blueprint Specification Box.
2. **Slide 2: SIA DIAGRAM 1 OF 5** — UML Use Case Diagram (System Actors, UC-01 to UC-13, Cloud Boundaries).
3. **Slide 3: SIA DIAGRAM 2 OF 5** — BPMN Swimlane Process Flow Diagram (Cross-functional interaction across Pet Owner, Clinic Staff, Core Engine, Supabase, and Admin).
4. **Slide 4: SIA DIAGRAM 3 OF 5** — Entity-Relationship Diagram (ERD) (PostgreSQL 15 relational schema: 10 production tables, foreign keys, cardinality).
5. **Slide 5: SIA DIAGRAM 4 OF 5** — System Logic Flowchart (End-to-End Decision Tree: SHA-256 Auth, 3-Tier Lockout, CAPTCHA, and Sunday Shift Exclusion Filter).
6. **Slide 6: SIA DIAGRAM 5 OF 5** — Network Architecture Diagram (5 distributed multi-tier layers: Presentation, Perimeter, Web Hosting, Supabase BaaS, and Cloud Gateways).
7. **Slide 7: Summary Matrix** — Architectural Diagram Specifications Summary Matrix mapping all 5 diagrams to academic notations, target subsystems, and security controls.

---

### 8. Operational Policy: Sunday Booking & Schedule Exclusion Rule

To ensure strict compliance with clinic labor regulations and operational shift policies:
- **Operating Days**: Monday through Saturday only (8:00 AM – 5:00 PM).
- **Sunday Prohibition**:
  - **Database Level**: All legacy Sunday appointments have been migrated to Monday (`2026-10-12`). All active reservations adhere to valid operating days.
  - **API Layer**: `PetStayDB.Reservations.create()` actively asserts `new Date(check_in_date).getDay() !== 0`.
  - **Client UI**: The appointment modal date picker restricts initial selection to valid weekdays, auto-advances any picked Sunday to Monday, displays real-time warning toasts (`"Clinic is closed on Sundays"`), and displays explicit badges (`"Mon–Sat Only (No Sundays)"`).

---
*Documentation compiled and formatted in accordance with standard UML 2.5, BPMN 2.0, Crow's Foot ERD, and ISO/IEC 25010 System Architecture specifications.*
