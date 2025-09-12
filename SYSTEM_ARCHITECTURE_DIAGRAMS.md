# Agentic Payments - Complete System Architecture Diagrams

## Table of Contents
1. [Complete System Architecture](#complete-system-architecture)
2. [Data Model Relationships](#data-model-relationships)
3. [Document Processing Flow](#document-processing-flow)
4. [Payment Orchestration Flow](#payment-orchestration-flow)
5. [Browser Automation Agent Flow](#browser-automation-agent-flow)
6. [Reconciliation Workflow](#reconciliation-workflow)
7. [Integration Architecture](#integration-architecture)
8. [Security & Audit Flow](#security--audit-flow)

---

## Complete System Architecture

```mermaid
graph TB
    %% External Users & Systems
    subgraph "External Layer"
        U[Users]
        V[Vendors]
        E[Email System]
        B[Banks]
    end

    %% Frontend Layer
    subgraph "Frontend Layer"
        subgraph "Web Application"
            CMD[Command Center Dashboard]
            SPV[Scheduled Payments View]
            POW[PO Workbench]
            RW[Reconciliation Workbench]
            VM[Vendor Management]
            AL[Audit Log Viewer]
            SE[System Setup]
        end
        
        subgraph "Real-time Updates"
            WS[WebSocket Client]
            SSE[Server-Sent Events]
        end
    end

    %% API Gateway Layer
    subgraph "API Gateway Layer"
        subgraph "API Endpoints"
            REST[REST API]
            GQL[GraphQL API]
            WSS[WebSocket Server]
        end
        
        subgraph "Gateway Services"
            AUTH[Auth Service]
            RL[Rate Limiter]
            VAL[Validation]
            CORS[CORS Handler]
        end
    end

    %% Core Services Layer
    subgraph "Core Services Layer"
        subgraph "Payment Services"
            PO_SVC[Payment Orchestrator]
            PS[Payment Scheduler]
            PE[Payment Executor]
            PR[Payment Router]
        end
        
        subgraph "Document Services"
            DI[Document Ingestion]
            DP[Document Processor]
            OCR[OCR Service]
            DM[Document Matcher]
        end
        
        subgraph "Business Logic"
            POL[Policy Engine]
            RECON[Reconciliation Engine]
            VEND[Vendor Service]
            BILL[Bill Service]
            PO_MGT[PO Management]
        end
        
        subgraph "Support Services"
            AUDIT[Audit Service]
            NOTIF[Notification Service]
            REPORT[Reporting Service]
        end
    end

    %% Agentic Layer
    subgraph "Agentic Layer"
        subgraph "Browser Agents"
            FA[Famous ERP Agent]
            NSA[NetSuite Agent]
            BA_POOL[Browser Agent Pool]
        end
        
        subgraph "Processing Agents"
            DOC_AG[Document Agent]
            MATCH_AG[Matching Agent]
            COMM_AG[Communication Agent]
        end
        
        subgraph "AI/ML Services"
            LLM[LLM Service]
            ML[ML Models]
            ANOM[Anomaly Detection]
        end
    end

    %% Integration Layer
    subgraph "Integration Layer"
        subgraph "Payment Providers"
            ASTRA[Astra - ACH/RTP]
            BRIDGE[Bridge - Stablecoin]
            POSTG[PostGrid - Checks]
        end
        
        subgraph "ERP/Accounting"
            QBO[QuickBooks Online]
            NS[NetSuite API]
            FAMOUS[Famous Browser]
        end
        
        subgraph "Banking"
            PLAID[Plaid API]
            BANK_API[Direct Bank APIs]
        end
    end

    %% Data Layer
    subgraph "Data Layer"
        subgraph "Primary Storage"
            PG[(PostgreSQL)]
            REDIS[(Redis Cache)]
            S3[(S3/Document Store)]
        end
        
        subgraph "Queue System"
            BULL[BullMQ Jobs]
            DLQ[Dead Letter Queue]
            PRIO[Priority Queue]
        end
        
        subgraph "Event System"
            EVT_BUS[Event Bus]
            EVT_STORE[(Event Store)]
        end
    end

    %% Monitoring & Security
    subgraph "Infrastructure Layer"
        subgraph "Monitoring"
            DD[DataDog APM]
            PROM[Prometheus]
            GRAF[Grafana]
            SENT[Sentry]
        end
        
        subgraph "Security"
            VAULT[HashiCorp Vault]
            KMS[AWS KMS]
            WAF[Web App Firewall]
        end
    end

    %% Connections - User Flows
    U --> CMD
    U --> SPV
    U --> POW
    U --> RW
    U --> VM
    
    V --> E
    E --> DI
    
    %% Frontend to API
    CMD --> REST
    SPV --> GQL
    POW --> REST
    RW --> GQL
    WS --> WSS
    
    %% API to Services
    REST --> AUTH
    GQL --> AUTH
    AUTH --> RL
    RL --> VAL
    
    %% Service Communications
    VAL --> PO_SVC
    VAL --> DI
    VAL --> POL
    VAL --> VEND
    
    %% Document Flow
    DI --> DP
    DP --> OCR
    DP --> DOC_AG
    OCR --> DM
    DM --> MATCH_AG
    
    %% Payment Flow
    PO_SVC --> PS
    PS --> PE
    PE --> PR
    PR --> ASTRA
    PR --> BRIDGE
    PR --> POSTG
    
    %% Reconciliation Flow
    PE --> RECON
    RECON --> FA
    RECON --> QBO
    FA --> FAMOUS
    
    %% Data Persistence
    PO_SVC --> PG
    VEND --> PG
    BILL --> PG
    AUDIT --> EVT_STORE
    
    %% Caching
    PO_SVC --> REDIS
    VEND --> REDIS
    
    %% Queue Processing
    DI --> BULL
    PE --> BULL
    RECON --> BULL
    
    %% Bank Integration
    RECON --> PLAID
    PLAID --> B
    
    %% Monitoring
    PO_SVC --> DD
    PE --> PROM
    AUDIT --> SENT

    classDef userLayer fill:#e1f5e1,stroke:#4caf50,stroke-width:2px
    classDef frontendLayer fill:#e3f2fd,stroke:#2196f3,stroke-width:2px
    classDef apiLayer fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px
    classDef serviceLayer fill:#fff3e0,stroke:#ff9800,stroke-width:2px
    classDef agentLayer fill:#fce4ec,stroke:#e91e63,stroke-width:2px
    classDef dataLayer fill:#f1f8e9,stroke:#8bc34a,stroke-width:2px
    classDef integrationLayer fill:#e0f2f1,stroke:#009688,stroke-width:2px
    
    class U,V,E,B userLayer
    class CMD,SPV,POW,RW,VM,AL,SE,WS,SSE frontendLayer
    class REST,GQL,WSS,AUTH,RL,VAL,CORS apiLayer
    class PO_SVC,PS,PE,PR,DI,DP,OCR,DM,POL,RECON,VEND,BILL,PO_MGT,AUDIT,NOTIF,REPORT serviceLayer
    class FA,NSA,BA_POOL,DOC_AG,MATCH_AG,COMM_AG,LLM,ML,ANOM agentLayer
    class PG,REDIS,S3,BULL,DLQ,PRIO,EVT_BUS,EVT_STORE dataLayer
    class ASTRA,BRIDGE,POSTG,QBO,NS,FAMOUS,PLAID,BANK_API integrationLayer
```

---

## Data Model Relationships

```mermaid
erDiagram
    ORGANIZATION ||--o{ TENANT : has
    TENANT ||--o{ USER : contains
    TENANT ||--o{ VENDOR : manages
    TENANT ||--o{ PURCHASE_ORDER : creates
    TENANT ||--o{ BILL : receives
    TENANT ||--o{ PAYMENT : processes
    TENANT ||--o{ POLICY_RULE : defines
    TENANT ||--o{ AUDIT_EVENT : logs
    
    USER ||--o{ SESSION : has
    USER ||--o{ AUDIT_EVENT : performs
    USER ||--o{ APPROVAL : makes
    
    VENDOR ||--o{ VENDOR_CONTACT : has
    VENDOR ||--o{ BANK_ACCOUNT : owns
    VENDOR ||--o{ PURCHASE_ORDER : receives
    VENDOR ||--o{ BILL : sends
    VENDOR ||--o{ PAYMENT : receives
    VENDOR ||--|| KYB_VERIFICATION : requires
    
    PURCHASE_ORDER ||--o{ PO_LINE_ITEM : contains
    PURCHASE_ORDER ||--o{ PO_DOCUMENT : attaches
    PURCHASE_ORDER ||--o{ APPROVAL : requires
    PURCHASE_ORDER ||--o{ BILL : generates
    PURCHASE_ORDER ||--o{ PAYMENT : triggers
    
    BILL ||--o{ BILL_LINE_ITEM : contains
    BILL ||--o{ BILL_DOCUMENT : attaches
    BILL ||--|| DUPLICATE_CHECK : has
    BILL ||--|| OCR_RESULT : processes
    BILL ||--o{ MATCHING_RESULT : produces
    BILL ||--o{ PAYMENT : requires
    
    PAYMENT ||--|| PAYMENT_METHOD : uses
    PAYMENT ||--o{ PAYMENT_BILL : pays
    PAYMENT ||--o{ PAYMENT_PO : covers
    PAYMENT ||--|| BANK_ACCOUNT : from
    PAYMENT ||--|| RECONCILIATION : has
    PAYMENT ||--o{ REMITTANCE : sends
    PAYMENT ||--o{ AUDIT_EVENT : generates
    
    RECONCILIATION ||--|| BANK_TRANSACTION : matches
    RECONCILIATION ||--|| ERP_RECORD : syncs
    RECONCILIATION ||--o{ RECON_EXCEPTION : may_have
    RECONCILIATION ||--|| AGENT_EXECUTION : may_use
    
    POLICY_RULE ||--o{ POLICY_CONDITION : defines
    POLICY_RULE ||--o{ POLICY_ACTION : triggers
    POLICY_RULE ||--o{ POLICY_EXCEPTION : creates
    
    POLICY_EXCEPTION ||--|| APPROVAL : requires
    POLICY_EXCEPTION ||--o{ AFFECTED_ENTITY : impacts
    
    AGENT_EXECUTION ||--o{ AGENT_STEP : performs
    AGENT_EXECUTION ||--o{ AGENT_SCREENSHOT : captures
    AGENT_EXECUTION ||--o{ AGENT_LOG : records
    AGENT_EXECUTION ||--|| AUDIT_EVENT : creates
    
    AUDIT_EVENT ||--o{ AUDIT_ATTACHMENT : includes
    AUDIT_EVENT ||--|| COMPLIANCE_TAG : has
    AUDIT_EVENT ||--|| RETENTION_POLICY : follows

    ORGANIZATION {
        uuid id PK
        string name
        string subdomain
        json settings
        json subscription
        timestamp created_at
        timestamp updated_at
    }
    
    VENDOR {
        uuid id PK
        uuid organization_id FK
        string erp_vendor_id
        string legal_name
        string dba
        string tax_id
        enum status
        enum standing
        float risk_score
        json payment_preferences
        json bank_accounts
        json contacts
        json addresses
        json kyb
        timestamp created_at
        timestamp updated_at
    }
    
    PURCHASE_ORDER {
        uuid id PK
        uuid organization_id FK
        string po_number
        uuid vendor_id FK
        date order_date
        date expected_delivery
        string currency
        json amounts
        json status
        json payment_terms
        timestamp created_at
        timestamp updated_at
    }
    
    BILL {
        uuid id PK
        uuid organization_id FK
        string bill_number
        uuid vendor_id FK
        uuid[] purchase_order_ids
        date invoice_date
        date due_date
        json amounts
        json status
        json matching
        timestamp created_at
        timestamp updated_at
    }
    
    PAYMENT {
        uuid id PK
        uuid organization_id FK
        string payment_number
        uuid vendor_id FK
        uuid[] bill_ids
        float amount
        enum method
        json rail
        json status
        json execution
        timestamp created_at
        timestamp updated_at
    }
    
    RECONCILIATION {
        uuid id PK
        uuid payment_id FK
        json matches
        json result
        json processing
        timestamp created_at
        timestamp completed_at
    }
    
    AGENT_EXECUTION {
        uuid id PK
        uuid organization_id FK
        enum agent_type
        json context
        json status
        json results
        json metrics
        timestamp created_at
        timestamp completed_at
    }
```

---

## Document Processing Flow

```mermaid
flowchart TB
    %% Document Sources
    subgraph "Document Sources"
        EMAIL[Email Attachment]
        UPLOAD[Manual Upload]
        API[API Upload]
        SCAN[Scanner Input]
    end
    
    %% Ingestion Pipeline
    subgraph "Ingestion Pipeline"
        INGEST[Document Ingestion Service]
        VIRUS[Virus Scanner]
        STORE[S3 Storage]
        QUEUE[Processing Queue]
    end
    
    %% Document Processing
    subgraph "Document Processing"
        CLASSIFY[Document Classifier]
        
        subgraph "OCR Processing"
            OCR_ENGINE[OCR Engine]
            TEXT_EXTRACT[Text Extraction]
            TABLE_EXTRACT[Table Extraction]
            CONFIDENCE[Confidence Scoring]
        end
        
        subgraph "AI Enhancement"
            LLM_PROCESS[LLM Processing]
            FIELD_EXTRACT[Field Extraction]
            LINE_PARSE[Line Item Parser]
        end
    end
    
    %% Document Types
    subgraph "Document Type Handlers"
        INV_HANDLER[Invoice Handler]
        PO_HANDLER[PO Handler]
        BOL_HANDLER[BOL Handler]
        RECEIPT_HANDLER[Receipt Handler]
    end
    
    %% Validation & Matching
    subgraph "Validation & Matching"
        DUP_CHECK[Duplicate Checker]
        VENDOR_MATCH[Vendor Matcher]
        PO_MATCH[PO Matcher]
        LINE_MATCH[Line Item Matcher]
        
        subgraph "Tolerance Rules"
            QTY_TOL[Quantity Tolerance]
            PRICE_TOL[Price Tolerance]
            DATE_TOL[Date Tolerance]
        end
    end
    
    %% Exception Handling
    subgraph "Exception Handling"
        MANUAL_REVIEW[Manual Review Queue]
        EXCEPTION_QUEUE[Exception Queue]
        HUMAN_VALIDATION[Human Validation]
    end
    
    %% Output & Integration
    subgraph "Output & Integration"
        CREATE_BILL[Create Bill Record]
        UPDATE_PO[Update PO Status]
        TRIGGER_POLICY[Trigger Policy Check]
        SEND_ACK[Send Acknowledgment]
        UPDATE_ERP[Update ERP]
    end
    
    %% Flow Connections
    EMAIL --> INGEST
    UPLOAD --> INGEST
    API --> INGEST
    SCAN --> INGEST
    
    INGEST --> VIRUS
    VIRUS -->|Clean| STORE
    VIRUS -->|Infected| EXCEPTION_QUEUE
    
    STORE --> QUEUE
    QUEUE --> CLASSIFY
    
    CLASSIFY -->|Invoice| INV_HANDLER
    CLASSIFY -->|PO| PO_HANDLER
    CLASSIFY -->|BOL| BOL_HANDLER
    CLASSIFY -->|Receipt| RECEIPT_HANDLER
    
    INV_HANDLER --> OCR_ENGINE
    PO_HANDLER --> OCR_ENGINE
    BOL_HANDLER --> OCR_ENGINE
    RECEIPT_HANDLER --> OCR_ENGINE
    
    OCR_ENGINE --> TEXT_EXTRACT
    OCR_ENGINE --> TABLE_EXTRACT
    TEXT_EXTRACT --> CONFIDENCE
    TABLE_EXTRACT --> CONFIDENCE
    
    CONFIDENCE -->|High| FIELD_EXTRACT
    CONFIDENCE -->|Low| LLM_PROCESS
    
    LLM_PROCESS --> FIELD_EXTRACT
    FIELD_EXTRACT --> LINE_PARSE
    
    LINE_PARSE --> DUP_CHECK
    DUP_CHECK -->|Unique| VENDOR_MATCH
    DUP_CHECK -->|Duplicate| EXCEPTION_QUEUE
    
    VENDOR_MATCH -->|Found| PO_MATCH
    VENDOR_MATCH -->|Not Found| MANUAL_REVIEW
    
    PO_MATCH --> LINE_MATCH
    LINE_MATCH --> QTY_TOL
    LINE_MATCH --> PRICE_TOL
    LINE_MATCH --> DATE_TOL
    
    QTY_TOL -->|Pass| CREATE_BILL
    PRICE_TOL -->|Pass| CREATE_BILL
    DATE_TOL -->|Pass| CREATE_BILL
    
    QTY_TOL -->|Fail| EXCEPTION_QUEUE
    PRICE_TOL -->|Fail| EXCEPTION_QUEUE
    DATE_TOL -->|Fail| EXCEPTION_QUEUE
    
    EXCEPTION_QUEUE --> HUMAN_VALIDATION
    MANUAL_REVIEW --> HUMAN_VALIDATION
    HUMAN_VALIDATION --> CREATE_BILL
    
    CREATE_BILL --> UPDATE_PO
    CREATE_BILL --> TRIGGER_POLICY
    CREATE_BILL --> SEND_ACK
    CREATE_BILL --> UPDATE_ERP
    
    classDef source fill:#e8f5e9,stroke:#4caf50
    classDef process fill:#e3f2fd,stroke:#2196f3
    classDef ai fill:#fce4ec,stroke:#e91e63
    classDef validation fill:#fff3e0,stroke:#ff9800
    classDef exception fill:#ffebee,stroke:#f44336
    classDef output fill:#f3e5f5,stroke:#9c27b0
    
    class EMAIL,UPLOAD,API,SCAN source
    class INGEST,VIRUS,STORE,QUEUE,CLASSIFY process
    class OCR_ENGINE,TEXT_EXTRACT,TABLE_EXTRACT,CONFIDENCE,LLM_PROCESS,FIELD_EXTRACT,LINE_PARSE ai
    class DUP_CHECK,VENDOR_MATCH,PO_MATCH,LINE_MATCH,QTY_TOL,PRICE_TOL,DATE_TOL validation
    class MANUAL_REVIEW,EXCEPTION_QUEUE,HUMAN_VALIDATION exception
    class CREATE_BILL,UPDATE_PO,TRIGGER_POLICY,SEND_ACK,UPDATE_ERP output
```

---

## Payment Orchestration Flow

```mermaid
flowchart TB
    %% Payment Initiation
    subgraph "Payment Initiation"
        SCHED_TRIGGER[Scheduled Trigger]
        MANUAL_TRIGGER[Manual Trigger]
        POLICY_TRIGGER[Policy Trigger]
        BULK_TRIGGER[Bulk Payment]
    end
    
    %% Payment Validation
    subgraph "Payment Validation"
        VAL_AMOUNT[Validate Amount]
        VAL_VENDOR[Validate Vendor]
        VAL_BANK[Validate Banking]
        VAL_POLICY[Policy Check]
        VAL_BUDGET[Budget Check]
    end
    
    %% Payment Routing
    subgraph "Payment Method Router"
        METHOD_SELECT[Method Selection]
        
        subgraph "Domestic Methods"
            ACH_STD[ACH Standard]
            ACH_SAME[ACH Same Day]
            RTP[Real-Time Payment]
            CHECK[Check]
        end
        
        subgraph "International Methods"
            WIRE[Wire Transfer]
            STABLE[Stablecoin USDC]
        end
    end
    
    %% Provider Integration
    subgraph "Provider Integration"
        subgraph "Astra Processing"
            ASTRA_ACH[Astra ACH API]
            ASTRA_RTP[Astra RTP API]
            ASTRA_STATUS[Status Webhook]
        end
        
        subgraph "Bridge Processing"
            BRIDGE_INIT[Bridge Init]
            BRIDGE_FX[FX Conversion]
            BRIDGE_SEND[Send USDC]
        end
        
        subgraph "PostGrid Processing"
            PG_FORMAT[Format Check]
            PG_PRINT[Print Service]
            PG_MAIL[Mail Service]
        end
    end
    
    %% Payment Execution
    subgraph "Payment Execution"
        EXEC_QUEUE[Execution Queue]
        BATCH_PROC[Batch Processor]
        SINGLE_PROC[Single Processor]
        
        subgraph "Execution Steps"
            DEBIT[Debit Account]
            SEND_PAY[Send Payment]
            CONFIRM[Await Confirmation]
            SETTLE[Settlement]
        end
    end
    
    %% Post-Payment
    subgraph "Post-Payment Processing"
        subgraph "Notifications"
            VENDOR_NOTIFY[Vendor Email]
            INTERNAL_NOTIFY[Internal Alert]
            REMIT_SEND[Remittance Doc]
        end
        
        subgraph "Record Updates"
            UPDATE_PAY[Update Payment]
            UPDATE_BILL[Update Bill]
            UPDATE_PO_PAY[Update PO]
            UPDATE_LEDGER[Update Ledger]
        end
        
        subgraph "Reconciliation Prep"
            RECON_QUEUE[Recon Queue]
            BANK_MATCH[Bank Match Prep]
            ERP_SYNC[ERP Sync Prep]
        end
    end
    
    %% Error Handling
    subgraph "Error Handling"
        RETRY_LOGIC[Retry Logic]
        FAIL_QUEUE[Failed Queue]
        MANUAL_INTERVENTION[Manual Fix]
        REVERSAL[Payment Reversal]
    end
    
    %% Flow Connections
    SCHED_TRIGGER --> VAL_AMOUNT
    MANUAL_TRIGGER --> VAL_AMOUNT
    POLICY_TRIGGER --> VAL_AMOUNT
    BULK_TRIGGER --> VAL_AMOUNT
    
    VAL_AMOUNT --> VAL_VENDOR
    VAL_VENDOR --> VAL_BANK
    VAL_BANK --> VAL_POLICY
    VAL_POLICY --> VAL_BUDGET
    
    VAL_BUDGET -->|Pass| METHOD_SELECT
    VAL_BUDGET -->|Fail| FAIL_QUEUE
    
    METHOD_SELECT -->|Domestic ACH| ACH_STD
    METHOD_SELECT -->|Urgent| ACH_SAME
    METHOD_SELECT -->|Instant| RTP
    METHOD_SELECT -->|Float| CHECK
    METHOD_SELECT -->|International| WIRE
    METHOD_SELECT -->|Crypto| STABLE
    
    ACH_STD --> ASTRA_ACH
    ACH_SAME --> ASTRA_ACH
    RTP --> ASTRA_RTP
    CHECK --> PG_FORMAT
    WIRE --> BRIDGE_INIT
    STABLE --> BRIDGE_INIT
    
    ASTRA_ACH --> EXEC_QUEUE
    ASTRA_RTP --> EXEC_QUEUE
    PG_FORMAT --> PG_PRINT
    PG_PRINT --> PG_MAIL
    PG_MAIL --> EXEC_QUEUE
    
    BRIDGE_INIT --> BRIDGE_FX
    BRIDGE_FX --> BRIDGE_SEND
    BRIDGE_SEND --> EXEC_QUEUE
    
    EXEC_QUEUE -->|Batch| BATCH_PROC
    EXEC_QUEUE -->|Single| SINGLE_PROC
    
    BATCH_PROC --> DEBIT
    SINGLE_PROC --> DEBIT
    
    DEBIT --> SEND_PAY
    SEND_PAY --> CONFIRM
    CONFIRM --> SETTLE
    
    SETTLE --> VENDOR_NOTIFY
    SETTLE --> INTERNAL_NOTIFY
    SETTLE --> REMIT_SEND
    
    VENDOR_NOTIFY --> UPDATE_PAY
    INTERNAL_NOTIFY --> UPDATE_PAY
    REMIT_SEND --> UPDATE_PAY
    
    UPDATE_PAY --> UPDATE_BILL
    UPDATE_BILL --> UPDATE_PO_PAY
    UPDATE_PO_PAY --> UPDATE_LEDGER
    
    UPDATE_LEDGER --> RECON_QUEUE
    RECON_QUEUE --> BANK_MATCH
    RECON_QUEUE --> ERP_SYNC
    
    SEND_PAY -->|Error| RETRY_LOGIC
    RETRY_LOGIC -->|Max Retries| FAIL_QUEUE
    FAIL_QUEUE --> MANUAL_INTERVENTION
    MANUAL_INTERVENTION --> REVERSAL
    
    classDef initiation fill:#e8f5e9,stroke:#4caf50
    classDef validation fill:#fff3e0,stroke:#ff9800
    classDef routing fill:#e3f2fd,stroke:#2196f3
    classDef provider fill:#f3e5f5,stroke:#9c27b0
    classDef execution fill:#fce4ec,stroke:#e91e63
    classDef post fill:#e0f2f1,stroke:#009688
    classDef error fill:#ffebee,stroke:#f44336
    
    class SCHED_TRIGGER,MANUAL_TRIGGER,POLICY_TRIGGER,BULK_TRIGGER initiation
    class VAL_AMOUNT,VAL_VENDOR,VAL_BANK,VAL_POLICY,VAL_BUDGET validation
    class METHOD_SELECT,ACH_STD,ACH_SAME,RTP,CHECK,WIRE,STABLE routing
    class ASTRA_ACH,ASTRA_RTP,ASTRA_STATUS,BRIDGE_INIT,BRIDGE_FX,BRIDGE_SEND,PG_FORMAT,PG_PRINT,PG_MAIL provider
    class EXEC_QUEUE,BATCH_PROC,SINGLE_PROC,DEBIT,SEND_PAY,CONFIRM,SETTLE execution
    class VENDOR_NOTIFY,INTERNAL_NOTIFY,REMIT_SEND,UPDATE_PAY,UPDATE_BILL,UPDATE_PO_PAY,UPDATE_LEDGER,RECON_QUEUE,BANK_MATCH,ERP_SYNC post
    class RETRY_LOGIC,FAIL_QUEUE,MANUAL_INTERVENTION,REVERSAL error
```

---

## Browser Automation Agent Flow

```mermaid
flowchart TB
    %% Agent Initialization
    subgraph "Agent Initialization"
        AGENT_POOL[Agent Pool Manager]
        AGENT_SELECT[Select Available Agent]
        BROWSER_INIT[Initialize Browser]
        CONTEXT_SETUP[Setup Context]
    end
    
    %% Authentication
    subgraph "Authentication Flow"
        LOAD_CREDS[Load Credentials]
        NAV_LOGIN[Navigate to Login]
        ENTER_CREDS[Enter Credentials]
        MFA[Handle MFA]
        VERIFY_LOGIN[Verify Login Success]
    end
    
    %% Famous ERP Navigation
    subgraph "Famous ERP Navigation"
        HOME_PAGE[ERP Home Page]
        AP_MODULE[AP Module]
        
        subgraph "AP Functions"
            BILL_SEARCH[Bill Search]
            BILL_CREATE[Bill Creation]
            PAYMENT_APPLY[Apply Payment]
            REPORT_GEN[Report Generation]
        end
    end
    
    %% Data Extraction
    subgraph "Data Extraction"
        PAGE_WAIT[Wait for Load]
        ELEM_SELECT[Element Selection]
        DATA_EXTRACT[Extract Data]
        SCREENSHOT[Capture Screenshot]
        
        subgraph "Extraction Methods"
            DOM_QUERY[DOM Query]
            TABLE_PARSE[Table Parser]
            TEXT_EXTRACT_AG[Text Extraction]
            FORM_READ[Form Reader]
        end
    end
    
    %% Data Entry
    subgraph "Data Entry"
        FORM_FILL[Fill Forms]
        DROPDOWN_SELECT[Select Dropdowns]
        DATE_PICK[Date Pickers]
        LINE_ITEMS_ADD[Add Line Items]
        SUBMIT_FORM[Submit Form]
    end
    
    %% Validation & Error Handling
    subgraph "Validation & Error Handling"
        VALIDATE_PAGE[Validate Page State]
        ERROR_DETECT[Detect Errors]
        RETRY_ACTION[Retry Action]
        FALLBACK[Fallback Strategy]
        
        subgraph "Error Types"
            TIMEOUT_ERR[Timeout Error]
            ELEM_NOT_FOUND[Element Not Found]
            VALIDATION_ERR[Validation Error]
            NETWORK_ERR[Network Error]
        end
    end
    
    %% Audit & Logging
    subgraph "Audit & Logging"
        ACTION_LOG[Log Action]
        SCREEN_CAPTURE[Screenshot Capture]
        STATE_SAVE[Save State]
        AUDIT_TRAIL[Create Audit Entry]
    end
    
    %% Cleanup
    subgraph "Cleanup"
        LOGOUT[Logout]
        CONTEXT_CLEAR[Clear Context]
        BROWSER_CLOSE[Close Browser]
        AGENT_RELEASE[Release Agent]
    end
    
    %% Flow Connections
    AGENT_POOL --> AGENT_SELECT
    AGENT_SELECT --> BROWSER_INIT
    BROWSER_INIT --> CONTEXT_SETUP
    
    CONTEXT_SETUP --> LOAD_CREDS
    LOAD_CREDS --> NAV_LOGIN
    NAV_LOGIN --> ENTER_CREDS
    ENTER_CREDS --> MFA
    MFA --> VERIFY_LOGIN
    
    VERIFY_LOGIN --> HOME_PAGE
    HOME_PAGE --> AP_MODULE
    
    AP_MODULE --> BILL_SEARCH
    AP_MODULE --> BILL_CREATE
    AP_MODULE --> PAYMENT_APPLY
    AP_MODULE --> REPORT_GEN
    
    BILL_SEARCH --> PAGE_WAIT
    BILL_CREATE --> PAGE_WAIT
    PAYMENT_APPLY --> PAGE_WAIT
    REPORT_GEN --> PAGE_WAIT
    
    PAGE_WAIT --> ELEM_SELECT
    ELEM_SELECT --> DATA_EXTRACT
    DATA_EXTRACT --> SCREENSHOT
    
    DATA_EXTRACT --> DOM_QUERY
    DATA_EXTRACT --> TABLE_PARSE
    DATA_EXTRACT --> TEXT_EXTRACT_AG
    DATA_EXTRACT --> FORM_READ
    
    BILL_CREATE --> FORM_FILL
    FORM_FILL --> DROPDOWN_SELECT
    DROPDOWN_SELECT --> DATE_PICK
    DATE_PICK --> LINE_ITEMS_ADD
    LINE_ITEMS_ADD --> SUBMIT_FORM
    
    SUBMIT_FORM --> VALIDATE_PAGE
    VALIDATE_PAGE --> ERROR_DETECT
    
    ERROR_DETECT -->|Error| RETRY_ACTION
    ERROR_DETECT -->|Success| ACTION_LOG
    
    RETRY_ACTION --> FALLBACK
    
    ERROR_DETECT --> TIMEOUT_ERR
    ERROR_DETECT --> ELEM_NOT_FOUND
    ERROR_DETECT --> VALIDATION_ERR
    ERROR_DETECT --> NETWORK_ERR
    
    ACTION_LOG --> SCREEN_CAPTURE
    SCREEN_CAPTURE --> STATE_SAVE
    STATE_SAVE --> AUDIT_TRAIL
    
    AUDIT_TRAIL -->|Complete| LOGOUT
    FALLBACK -->|Unrecoverable| LOGOUT
    
    LOGOUT --> CONTEXT_CLEAR
    CONTEXT_CLEAR --> BROWSER_CLOSE
    BROWSER_CLOSE --> AGENT_RELEASE
    
    classDef init fill:#e8f5e9,stroke:#4caf50
    classDef auth fill:#e3f2fd,stroke:#2196f3
    classDef nav fill:#f3e5f5,stroke:#9c27b0
    classDef extract fill:#fff3e0,stroke:#ff9800
    classDef entry fill:#fce4ec,stroke:#e91e63
    classDef validation fill:#ffebee,stroke:#f44336
    classDef audit fill:#e0f2f1,stroke:#009688
    classDef cleanup fill:#f5f5f5,stroke:#9e9e9e
    
    class AGENT_POOL,AGENT_SELECT,BROWSER_INIT,CONTEXT_SETUP init
    class LOAD_CREDS,NAV_LOGIN,ENTER_CREDS,MFA,VERIFY_LOGIN auth
    class HOME_PAGE,AP_MODULE,BILL_SEARCH,BILL_CREATE,PAYMENT_APPLY,REPORT_GEN nav
    class PAGE_WAIT,ELEM_SELECT,DATA_EXTRACT,SCREENSHOT,DOM_QUERY,TABLE_PARSE,TEXT_EXTRACT_AG,FORM_READ extract
    class FORM_FILL,DROPDOWN_SELECT,DATE_PICK,LINE_ITEMS_ADD,SUBMIT_FORM entry
    class VALIDATE_PAGE,ERROR_DETECT,RETRY_ACTION,FALLBACK,TIMEOUT_ERR,ELEM_NOT_FOUND,VALIDATION_ERR,NETWORK_ERR validation
    class ACTION_LOG,SCREEN_CAPTURE,STATE_SAVE,AUDIT_TRAIL audit
    class LOGOUT,CONTEXT_CLEAR,BROWSER_CLOSE,AGENT_RELEASE cleanup
```

---

## Reconciliation Workflow

```mermaid
flowchart TB
    %% Reconciliation Triggers
    subgraph "Reconciliation Triggers"
        DAILY_BATCH[Daily Batch Job]
        PAYMENT_COMPLETE[Payment Completion]
        MANUAL_RECON[Manual Trigger]
        BANK_WEBHOOK[Bank Webhook]
    end
    
    %% Data Collection
    subgraph "Data Collection"
        subgraph "Internal Data"
            PAY_RECORDS[Payment Records]
            BILL_RECORDS[Bill Records]
            PO_RECORDS[PO Records]
        end
        
        subgraph "Bank Data"
            PLAID_FETCH[Plaid API Fetch]
            BANK_FILE[Bank File Import]
            BANK_API_DIRECT[Direct Bank API]
        end
        
        subgraph "ERP Data"
            QBO_FETCH[QuickBooks Fetch]
            NS_FETCH[NetSuite Fetch]
            FAMOUS_AGENT[Famous Agent Fetch]
        end
    end
    
    %% Matching Engine
    subgraph "Matching Engine"
        subgraph "Match Criteria"
            AMOUNT_MATCH[Amount Match]
            DATE_MATCH[Date Match]
            REF_MATCH[Reference Match]
            VENDOR_MATCH_REC[Vendor Match]
        end
        
        subgraph "Fuzzy Matching"
            FUZZY_AMOUNT[Amount Tolerance]
            FUZZY_DATE[Date Window]
            FUZZY_TEXT[Text Similarity]
            ML_MATCH[ML Confidence]
        end
        
        MATCH_SCORE[Calculate Match Score]
    end
    
    %% Three-Way Match
    subgraph "Three-Way Reconciliation"
        INTERNAL_VS_BANK[Internal vs Bank]
        INTERNAL_VS_ERP[Internal vs ERP]
        BANK_VS_ERP[Bank vs ERP]
        
        THREE_WAY_RESULT[Three-Way Result]
    end
    
    %% Result Processing
    subgraph "Result Processing"
        subgraph "Match Results"
            FULL_MATCH[Fully Matched]
            PARTIAL_MATCH[Partial Match]
            NO_MATCH[No Match]
        end
        
        subgraph "Confidence Levels"
            HIGH_CONF[High Confidence >95%]
            MED_CONF[Medium 75-95%]
            LOW_CONF[Low <75%]
        end
    end
    
    %% Exception Handling
    subgraph "Exception Handling"
        EXCEPTION_QUEUE_REC[Exception Queue]
        
        subgraph "Exception Types"
            MISSING_BANK[Missing Bank Entry]
            MISSING_ERP[Missing ERP Entry]
            AMOUNT_VARIANCE[Amount Variance]
            DUPLICATE_ENTRY[Duplicate Entry]
        end
        
        MANUAL_REVIEW_REC[Manual Review]
        AUTO_RESOLVE[Auto Resolution]
    end
    
    %% Resolution Actions
    subgraph "Resolution Actions"
        subgraph "Automated Actions"
            CREATE_ERP_ENTRY[Create ERP Entry]
            UPDATE_ERP_ENTRY[Update ERP Entry]
            MARK_RECONCILED[Mark Reconciled]
            FLAG_DISPUTE[Flag for Dispute]
        end
        
        subgraph "Manual Actions"
            ADJUST_AMOUNT[Adjust Amount]
            LINK_RECORDS[Link Records]
            CREATE_ADJUSTMENT[Create Adjustment]
            VOID_PAYMENT[Void Payment]
        end
    end
    
    %% Update & Reporting
    subgraph "Update & Reporting"
        UPDATE_STATUS[Update Recon Status]
        UPDATE_AUDIT[Update Audit Log]
        NOTIFY_TEAM[Notify Team]
        GENERATE_REPORT[Generate Report]
    end
    
    %% Flow Connections
    DAILY_BATCH --> PAY_RECORDS
    PAYMENT_COMPLETE --> PAY_RECORDS
    MANUAL_RECON --> PAY_RECORDS
    BANK_WEBHOOK --> PLAID_FETCH
    
    PAY_RECORDS --> AMOUNT_MATCH
    BILL_RECORDS --> AMOUNT_MATCH
    PO_RECORDS --> AMOUNT_MATCH
    
    PLAID_FETCH --> DATE_MATCH
    BANK_FILE --> DATE_MATCH
    BANK_API_DIRECT --> DATE_MATCH
    
    QBO_FETCH --> REF_MATCH
    NS_FETCH --> REF_MATCH
    FAMOUS_AGENT --> REF_MATCH
    
    AMOUNT_MATCH --> MATCH_SCORE
    DATE_MATCH --> MATCH_SCORE
    REF_MATCH --> MATCH_SCORE
    VENDOR_MATCH_REC --> MATCH_SCORE
    
    MATCH_SCORE --> FUZZY_AMOUNT
    FUZZY_AMOUNT --> FUZZY_DATE
    FUZZY_DATE --> FUZZY_TEXT
    FUZZY_TEXT --> ML_MATCH
    
    ML_MATCH --> INTERNAL_VS_BANK
    INTERNAL_VS_BANK --> INTERNAL_VS_ERP
    INTERNAL_VS_ERP --> BANK_VS_ERP
    BANK_VS_ERP --> THREE_WAY_RESULT
    
    THREE_WAY_RESULT --> FULL_MATCH
    THREE_WAY_RESULT --> PARTIAL_MATCH
    THREE_WAY_RESULT --> NO_MATCH
    
    FULL_MATCH --> HIGH_CONF
    PARTIAL_MATCH --> MED_CONF
    NO_MATCH --> LOW_CONF
    
    HIGH_CONF --> MARK_RECONCILED
    MED_CONF --> AUTO_RESOLVE
    LOW_CONF --> EXCEPTION_QUEUE_REC
    
    EXCEPTION_QUEUE_REC --> MISSING_BANK
    EXCEPTION_QUEUE_REC --> MISSING_ERP
    EXCEPTION_QUEUE_REC --> AMOUNT_VARIANCE
    EXCEPTION_QUEUE_REC --> DUPLICATE_ENTRY
    
    MISSING_BANK --> MANUAL_REVIEW_REC
    MISSING_ERP --> CREATE_ERP_ENTRY
    AMOUNT_VARIANCE --> MANUAL_REVIEW_REC
    DUPLICATE_ENTRY --> AUTO_RESOLVE
    
    AUTO_RESOLVE --> UPDATE_ERP_ENTRY
    AUTO_RESOLVE --> MARK_RECONCILED
    
    MANUAL_REVIEW_REC --> ADJUST_AMOUNT
    MANUAL_REVIEW_REC --> LINK_RECORDS
    MANUAL_REVIEW_REC --> CREATE_ADJUSTMENT
    MANUAL_REVIEW_REC --> VOID_PAYMENT
    
    MARK_RECONCILED --> UPDATE_STATUS
    CREATE_ERP_ENTRY --> UPDATE_STATUS
    UPDATE_ERP_ENTRY --> UPDATE_STATUS
    CREATE_ADJUSTMENT --> UPDATE_STATUS
    
    UPDATE_STATUS --> UPDATE_AUDIT
    UPDATE_AUDIT --> NOTIFY_TEAM
    NOTIFY_TEAM --> GENERATE_REPORT
    
    classDef trigger fill:#e8f5e9,stroke:#4caf50
    classDef collect fill:#e3f2fd,stroke:#2196f3
    classDef match fill:#fff3e0,stroke:#ff9800
    classDef process fill:#f3e5f5,stroke:#9c27b0
    classDef exception fill:#ffebee,stroke:#f44336
    classDef resolve fill:#fce4ec,stroke:#e91e63
    classDef update fill:#e0f2f1,stroke:#009688
    
    class DAILY_BATCH,PAYMENT_COMPLETE,MANUAL_RECON,BANK_WEBHOOK trigger
    class PAY_RECORDS,BILL_RECORDS,PO_RECORDS,PLAID_FETCH,BANK_FILE,BANK_API_DIRECT,QBO_FETCH,NS_FETCH,FAMOUS_AGENT collect
    class AMOUNT_MATCH,DATE_MATCH,REF_MATCH,VENDOR_MATCH_REC,FUZZY_AMOUNT,FUZZY_DATE,FUZZY_TEXT,ML_MATCH,MATCH_SCORE match
    class INTERNAL_VS_BANK,INTERNAL_VS_ERP,BANK_VS_ERP,THREE_WAY_RESULT,FULL_MATCH,PARTIAL_MATCH,NO_MATCH,HIGH_CONF,MED_CONF,LOW_CONF process
    class EXCEPTION_QUEUE_REC,MISSING_BANK,MISSING_ERP,AMOUNT_VARIANCE,DUPLICATE_ENTRY,MANUAL_REVIEW_REC exception
    class AUTO_RESOLVE,CREATE_ERP_ENTRY,UPDATE_ERP_ENTRY,MARK_RECONCILED,FLAG_DISPUTE,ADJUST_AMOUNT,LINK_RECORDS,CREATE_ADJUSTMENT,VOID_PAYMENT resolve
    class UPDATE_STATUS,UPDATE_AUDIT,NOTIFY_TEAM,GENERATE_REPORT update
```

---

## Integration Architecture

```mermaid
flowchart TB
    %% Core System
    subgraph "Agentic Payments Core"
        CORE_API[Core API Gateway]
        INTEGRATION_ENGINE[Integration Engine]
        ADAPTER_LAYER[Adapter Layer]
        WEBHOOK_HANDLER[Webhook Handler]
    end
    
    %% ERP Integrations
    subgraph "ERP/Accounting Systems"
        subgraph "QuickBooks"
            QBO_AUTH[OAuth 2.0]
            QBO_API[QBO REST API]
            QBO_WEBHOOK[QBO Webhooks]
            QBO_SYNC[Sync Service]
        end
        
        subgraph "NetSuite"
            NS_AUTH[Token Auth]
            NS_REST[REST API]
            NS_SOAP[SOAP API]
            NS_SUITE[SuiteScript]
        end
        
        subgraph "Famous ERP"
            FAMOUS_BROWSER[Browser Agent]
            FAMOUS_LOGIN[Login Handler]
            FAMOUS_NAV[Navigation]
            FAMOUS_SCRAPE[Data Scraper]
        end
    end
    
    %% Payment Provider Integrations
    subgraph "Payment Providers"
        subgraph "Astra"
            ASTRA_AUTH[API Key Auth]
            ASTRA_PAY_API[Payment API]
            ASTRA_WEBHOOK[Webhooks]
            ASTRA_REPORT[Reports API]
        end
        
        subgraph "Bridge"
            BRIDGE_AUTH[OAuth]
            BRIDGE_QUOTE[Quote API]
            BRIDGE_TRANSFER[Transfer API]
            BRIDGE_STATUS[Status API]
        end
        
        subgraph "PostGrid"
            PG_AUTH[API Key]
            PG_CHECK[Check API]
            PG_LETTER[Letter API]
            PG_TRACK[Tracking API]
        end
    end
    
    %% Banking Integrations
    subgraph "Banking Services"
        subgraph "Plaid"
            PLAID_LINK[Plaid Link]
            PLAID_AUTH_API[Auth API]
            PLAID_TRANS[Transactions]
            PLAID_BALANCE[Balance API]
        end
        
        subgraph "Direct Bank APIs"
            BANK_AUTH[Bank Auth]
            BANK_STMT[Statements]
            BANK_TRANS[Transactions]
            BANK_NOTIFY[Notifications]
        end
    end
    
    %% Document Services
    subgraph "Document Processing"
        subgraph "OCR Services"
            AWS_TEXTRACT[AWS Textract]
            AZURE_FORM[Azure Forms]
            TESSERACT[Tesseract]
        end
        
        subgraph "Storage"
            S3[AWS S3]
            S3_EVENTS[S3 Events]
            CDN[CloudFront CDN]
        end
    end
    
    %% Communication Services
    subgraph "Communication"
        subgraph "Email"
            SENDGRID[SendGrid API]
            EMAIL_PARSE[Email Parser]
            EMAIL_TEMPLATE[Templates]
        end
        
        subgraph "Notifications"
            SLACK_API[Slack API]
            TEAMS_API[Teams API]
            SMS_API[Twilio SMS]
        end
    end
    
    %% Security & Compliance
    subgraph "Security Services"
        subgraph "Authentication"
            AUTH0[Auth0]
            COGNITO[AWS Cognito]
            OKTA[Okta]
        end
        
        subgraph "Secrets"
            KMS[AWS KMS]
            SECRETS_MGR[Secrets Manager]
            VAULT_INT[HashiCorp Vault]
        end
    end
    
    %% Data Flow Connections
    CORE_API --> INTEGRATION_ENGINE
    INTEGRATION_ENGINE --> ADAPTER_LAYER
    
    %% ERP Connections
    ADAPTER_LAYER --> QBO_AUTH
    QBO_AUTH --> QBO_API
    QBO_API --> QBO_SYNC
    QBO_WEBHOOK --> WEBHOOK_HANDLER
    
    ADAPTER_LAYER --> NS_AUTH
    NS_AUTH --> NS_REST
    NS_AUTH --> NS_SOAP
    
    ADAPTER_LAYER --> FAMOUS_BROWSER
    FAMOUS_BROWSER --> FAMOUS_LOGIN
    FAMOUS_LOGIN --> FAMOUS_NAV
    FAMOUS_NAV --> FAMOUS_SCRAPE
    
    %% Payment Provider Connections
    ADAPTER_LAYER --> ASTRA_AUTH
    ASTRA_AUTH --> ASTRA_PAY_API
    ASTRA_WEBHOOK --> WEBHOOK_HANDLER
    
    ADAPTER_LAYER --> BRIDGE_AUTH
    BRIDGE_AUTH --> BRIDGE_QUOTE
    BRIDGE_QUOTE --> BRIDGE_TRANSFER
    
    ADAPTER_LAYER --> PG_AUTH
    PG_AUTH --> PG_CHECK
    
    %% Banking Connections
    ADAPTER_LAYER --> PLAID_LINK
    PLAID_LINK --> PLAID_AUTH_API
    PLAID_AUTH_API --> PLAID_TRANS
    
    ADAPTER_LAYER --> BANK_AUTH
    BANK_AUTH --> BANK_TRANS
    BANK_NOTIFY --> WEBHOOK_HANDLER
    
    %% Document Connections
    ADAPTER_LAYER --> AWS_TEXTRACT
    ADAPTER_LAYER --> AZURE_FORM
    S3_EVENTS --> WEBHOOK_HANDLER
    
    %% Communication Connections
    INTEGRATION_ENGINE --> SENDGRID
    INTEGRATION_ENGINE --> SLACK_API
    
    %% Security Connections
    CORE_API --> AUTH0
    INTEGRATION_ENGINE --> KMS
    INTEGRATION_ENGINE --> SECRETS_MGR
    
    classDef core fill:#263238,color:#fff
    classDef erp fill:#e3f2fd,stroke:#2196f3
    classDef payment fill:#f3e5f5,stroke:#9c27b0
    classDef banking fill:#e0f2f1,stroke:#009688
    classDef document fill:#fff3e0,stroke:#ff9800
    classDef comm fill:#fce4ec,stroke:#e91e63
    classDef security fill:#ffebee,stroke:#f44336
    
    class CORE_API,INTEGRATION_ENGINE,ADAPTER_LAYER,WEBHOOK_HANDLER core
    class QBO_AUTH,QBO_API,QBO_WEBHOOK,QBO_SYNC,NS_AUTH,NS_REST,NS_SOAP,NS_SUITE,FAMOUS_BROWSER,FAMOUS_LOGIN,FAMOUS_NAV,FAMOUS_SCRAPE erp
    class ASTRA_AUTH,ASTRA_PAY_API,ASTRA_WEBHOOK,ASTRA_REPORT,BRIDGE_AUTH,BRIDGE_QUOTE,BRIDGE_TRANSFER,BRIDGE_STATUS,PG_AUTH,PG_CHECK,PG_LETTER,PG_TRACK payment
    class PLAID_LINK,PLAID_AUTH_API,PLAID_TRANS,PLAID_BALANCE,BANK_AUTH,BANK_STMT,BANK_TRANS,BANK_NOTIFY banking
    class AWS_TEXTRACT,AZURE_FORM,TESSERACT,S3,S3_EVENTS,CDN document
    class SENDGRID,EMAIL_PARSE,EMAIL_TEMPLATE,SLACK_API,TEAMS_API,SMS_API comm
    class AUTH0,COGNITO,OKTA,KMS,SECRETS_MGR,VAULT_INT security
```

---

## Security & Audit Flow

```mermaid
flowchart TB
    %% Entry Points
    subgraph "Entry Points"
        USER_ACTION[User Action]
        API_CALL[API Call]
        AGENT_ACTION[Agent Action]
        SYSTEM_EVENT[System Event]
    end
    
    %% Authentication Layer
    subgraph "Authentication & Authorization"
        AUTH_CHECK[Auth Check]
        MFA_VERIFY[MFA Verification]
        TOKEN_VALIDATE[Token Validation]
        RBAC[Role-Based Access]
        PERMISSION_CHECK[Permission Check]
    end
    
    %% Security Middleware
    subgraph "Security Middleware"
        RATE_LIMIT[Rate Limiting]
        IP_CHECK[IP Whitelist]
        INPUT_SANITIZE[Input Sanitization]
        XSS_PROTECT[XSS Protection]
        CSRF_PROTECT[CSRF Protection]
        SQL_INJECT[SQL Injection Prevention]
    end
    
    %% Encryption Layer
    subgraph "Data Encryption"
        FIELD_ENCRYPT[Field-Level Encryption]
        
        subgraph "Sensitive Data"
            BANK_ENCRYPT[Bank Account Encryption]
            SSN_ENCRYPT[SSN/TaxID Encryption]
            API_KEY_ENCRYPT[API Key Encryption]
        end
        
        TLS[TLS in Transit]
        AT_REST[Encryption at Rest]
    end
    
    %% Audit Processing
    subgraph "Audit Event Processing"
        EVENT_CREATE[Create Audit Event]
        
        subgraph "Event Enrichment"
            ADD_CONTEXT[Add Context]
            ADD_TRACE[Add Trace ID]
            ADD_SESSION[Add Session Info]
            ADD_METADATA[Add Metadata]
        end
        
        EVENT_VALIDATE[Validate Event]
        EVENT_SIGN[Sign Event]
    end
    
    %% Audit Storage
    subgraph "Immutable Audit Storage"
        APPEND_LOG[Append-Only Log]
        HASH_CHAIN[Hash Chain]
        TIMESTAMP[Timestamp Server]
        
        subgraph "Storage Backends"
            PRIMARY_STORE[Primary Store]
            BACKUP_STORE[Backup Store]
            ARCHIVE_STORE[Archive Store]
        end
    end
    
    %% Compliance Processing
    subgraph "Compliance & Monitoring"
        subgraph "Compliance Checks"
            PII_CHECK[PII Detection]
            POLICY_COMPLY[Policy Compliance]
            REG_COMPLY[Regulatory Compliance]
        end
        
        subgraph "Real-time Monitoring"
            ANOMALY_DETECT[Anomaly Detection]
            THREAT_DETECT[Threat Detection]
            ALERT_ENGINE[Alert Engine]
        end
        
        subgraph "Reporting"
            COMPLIANCE_REPORT[Compliance Reports]
            AUDIT_REPORT[Audit Reports]
            SECURITY_REPORT[Security Reports]
        end
    end
    
    %% Incident Response
    subgraph "Incident Response"
        INCIDENT_DETECT[Incident Detection]
        
        subgraph "Response Actions"
            BLOCK_USER[Block User/IP]
            REVOKE_TOKEN[Revoke Tokens]
            LOCKDOWN[System Lockdown]
            NOTIFY_ADMIN[Notify Admins]
        end
        
        INCIDENT_LOG[Incident Logging]
        FORENSICS[Forensic Analysis]
    end
    
    %% Flow Connections
    USER_ACTION --> AUTH_CHECK
    API_CALL --> AUTH_CHECK
    AGENT_ACTION --> AUTH_CHECK
    SYSTEM_EVENT --> EVENT_CREATE
    
    AUTH_CHECK --> MFA_VERIFY
    MFA_VERIFY --> TOKEN_VALIDATE
    TOKEN_VALIDATE --> RBAC
    RBAC --> PERMISSION_CHECK
    
    PERMISSION_CHECK --> RATE_LIMIT
    RATE_LIMIT --> IP_CHECK
    IP_CHECK --> INPUT_SANITIZE
    INPUT_SANITIZE --> XSS_PROTECT
    XSS_PROTECT --> CSRF_PROTECT
    CSRF_PROTECT --> SQL_INJECT
    
    SQL_INJECT --> FIELD_ENCRYPT
    
    FIELD_ENCRYPT --> BANK_ENCRYPT
    FIELD_ENCRYPT --> SSN_ENCRYPT
    FIELD_ENCRYPT --> API_KEY_ENCRYPT
    
    BANK_ENCRYPT --> TLS
    SSN_ENCRYPT --> TLS
    API_KEY_ENCRYPT --> TLS
    TLS --> AT_REST
    
    AT_REST --> EVENT_CREATE
    
    EVENT_CREATE --> ADD_CONTEXT
    ADD_CONTEXT --> ADD_TRACE
    ADD_TRACE --> ADD_SESSION
    ADD_SESSION --> ADD_METADATA
    
    ADD_METADATA --> EVENT_VALIDATE
    EVENT_VALIDATE --> EVENT_SIGN
    
    EVENT_SIGN --> APPEND_LOG
    APPEND_LOG --> HASH_CHAIN
    HASH_CHAIN --> TIMESTAMP
    
    TIMESTAMP --> PRIMARY_STORE
    PRIMARY_STORE --> BACKUP_STORE
    BACKUP_STORE --> ARCHIVE_STORE
    
    EVENT_SIGN --> PII_CHECK
    PII_CHECK --> POLICY_COMPLY
    POLICY_COMPLY --> REG_COMPLY
    
    REG_COMPLY --> ANOMALY_DETECT
    ANOMALY_DETECT --> THREAT_DETECT
    THREAT_DETECT --> ALERT_ENGINE
    
    ALERT_ENGINE --> INCIDENT_DETECT
    
    INCIDENT_DETECT --> BLOCK_USER
    INCIDENT_DETECT --> REVOKE_TOKEN
    INCIDENT_DETECT --> LOCKDOWN
    INCIDENT_DETECT --> NOTIFY_ADMIN
    
    BLOCK_USER --> INCIDENT_LOG
    REVOKE_TOKEN --> INCIDENT_LOG
    LOCKDOWN --> INCIDENT_LOG
    NOTIFY_ADMIN --> INCIDENT_LOG
    
    INCIDENT_LOG --> FORENSICS
    
    ARCHIVE_STORE --> COMPLIANCE_REPORT
    ARCHIVE_STORE --> AUDIT_REPORT
    ARCHIVE_STORE --> SECURITY_REPORT
    
    classDef entry fill:#e8f5e9,stroke:#4caf50
    classDef auth fill:#e3f2fd,stroke:#2196f3
    classDef security fill:#fff3e0,stroke:#ff9800
    classDef encrypt fill:#f3e5f5,stroke:#9c27b0
    classDef audit fill:#fce4ec,stroke:#e91e63
    classDef storage fill:#e0f2f1,stroke:#009688
    classDef compliance fill:#f5f5f5,stroke:#607d8b
    classDef incident fill:#ffebee,stroke:#f44336
    
    class USER_ACTION,API_CALL,AGENT_ACTION,SYSTEM_EVENT entry
    class AUTH_CHECK,MFA_VERIFY,TOKEN_VALIDATE,RBAC,PERMISSION_CHECK auth
    class RATE_LIMIT,IP_CHECK,INPUT_SANITIZE,XSS_PROTECT,CSRF_PROTECT,SQL_INJECT security
    class FIELD_ENCRYPT,BANK_ENCRYPT,SSN_ENCRYPT,API_KEY_ENCRYPT,TLS,AT_REST encrypt
    class EVENT_CREATE,ADD_CONTEXT,ADD_TRACE,ADD_SESSION,ADD_METADATA,EVENT_VALIDATE,EVENT_SIGN audit
    class APPEND_LOG,HASH_CHAIN,TIMESTAMP,PRIMARY_STORE,BACKUP_STORE,ARCHIVE_STORE storage
    class PII_CHECK,POLICY_COMPLY,REG_COMPLY,ANOMALY_DETECT,THREAT_DETECT,ALERT_ENGINE,COMPLIANCE_REPORT,AUDIT_REPORT,SECURITY_REPORT compliance
    class INCIDENT_DETECT,BLOCK_USER,REVOKE_TOKEN,LOCKDOWN,NOTIFY_ADMIN,INCIDENT_LOG,FORENSICS incident
```

---

## Summary

These comprehensive diagrams provide a complete visual representation of the Agentic Payments platform architecture:

1. **Complete System Architecture** - Shows all layers and components with their interactions
2. **Data Model Relationships** - Entity relationships with complete field definitions
3. **Document Processing Flow** - End-to-end document ingestion, OCR, and matching
4. **Payment Orchestration Flow** - Multi-rail payment execution and settlement
5. **Browser Automation Agent Flow** - Famous ERP integration via browser automation
6. **Reconciliation Workflow** - Three-way matching and exception handling
7. **Integration Architecture** - All external system connections and adapters
8. **Security & Audit Flow** - Complete security, compliance, and audit trail

Each diagram maintains clear separation of concerns with color-coded sections for easy navigation and understanding. The flows demonstrate both happy paths and error handling scenarios, ensuring comprehensive coverage of the system's functionality.