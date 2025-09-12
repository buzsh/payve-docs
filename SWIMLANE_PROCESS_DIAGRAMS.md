# Agentic Payments - Swimlane Process Diagrams

## Table of Contents
1. [End-to-End Invoice Processing](#end-to-end-invoice-processing)
2. [Payment Execution Workflow](#payment-execution-workflow)
3. [Three-Way Reconciliation Process](#three-way-reconciliation-process)
4. [Vendor Onboarding & KYB](#vendor-onboarding--kyb)
5. [Famous ERP Browser Automation](#famous-erp-browser-automation)
6. [Policy Exception & Approval Flow](#policy-exception--approval-flow)
7. [Document Matching & Dispute Resolution](#document-matching--dispute-resolution)
8. [Multi-Channel Payment Scheduling](#multi-channel-payment-scheduling)

---

## End-to-End Invoice Processing

```mermaid
flowchart TB
    subgraph "Vendor"
        V1[Send Invoice Email]
        V2[Receive Acknowledgment]
        V3[Receive Payment Notice]
        V4[Query Payment Status]
    end
    
    subgraph "Email System"
        E1[Receive Email]
        E2[Forward to Ingestion]
        E3[Send Auto-Reply]
        E4[Send Remittance]
    end
    
    subgraph "Document Agent"
        DA1[Parse Email]
        DA2[Extract Attachments]
        DA3[OCR Processing]
        DA4[Extract Fields]
        DA5[Validate Data]
        DA6[Retry with LLM]
    end
    
    subgraph "Matching Engine"
        ME1[Load PO Data]
        ME2[Match Invoice to PO]
        ME3[Check Tolerances]
        ME4[Calculate Variances]
        ME5[Flag Exceptions]
    end
    
    subgraph "Policy Engine"
        PE1[Load Rules]
        PE2[Evaluate Conditions]
        PE3[Check Thresholds]
        PE4[Require Approval]
        PE5[Auto-Approve]
    end
    
    subgraph "AP Manager"
        AP1[Review Exception]
        AP2[Approve/Reject]
        AP3[Add Comments]
        AP4[Override Policy]
    end
    
    subgraph "Payment System"
        PS1[Create Bill]
        PS2[Schedule Payment]
        PS3[Execute Payment]
        PS4[Update Status]
    end
    
    subgraph "ERP System"
        ERP1[Create Bill Entry]
        ERP2[Update GL]
        ERP3[Mark Paid]
    end
    
    %% Main Flow
    V1 --> E1
    E1 --> E2
    E2 --> DA1
    E1 --> E3
    E3 --> V2
    
    DA1 --> DA2
    DA2 --> DA3
    DA3 --> DA4
    DA4 --> DA5
    DA5 -->|Valid| ME1
    DA5 -->|Invalid| DA6
    DA6 -->|Retry| DA4
    DA6 -->|Failed| AP1
    
    ME1 --> ME2
    ME2 --> ME3
    ME3 -->|Within| ME4
    ME3 -->|Outside| ME5
    ME4 --> PE1
    ME5 --> AP1
    
    PE1 --> PE2
    PE2 --> PE3
    PE3 -->|Below Threshold| PE5
    PE3 -->|Above Threshold| PE4
    PE4 --> AP1
    PE5 --> PS1
    
    AP1 --> AP2
    AP2 -->|Approve| AP3
    AP2 -->|Reject| V4
    AP3 --> AP4
    AP4 --> PS1
    
    PS1 --> PS2
    PS1 --> ERP1
    PS2 --> PS3
    PS3 --> PS4
    PS4 --> ERP2
    PS4 --> E4
    E4 --> V3
    
    ERP2 --> ERP3
    
    %% Query flow
    V4 -->|Status Query| E1
    
    classDef vendor fill:#e8f5e9,stroke:#4caf50
    classDef email fill:#e3f2fd,stroke:#2196f3
    classDef agent fill:#fce4ec,stroke:#e91e63
    classDef matching fill:#fff3e0,stroke:#ff9800
    classDef policy fill:#f3e5f5,stroke:#9c27b0
    classDef human fill:#ffebee,stroke:#f44336
    classDef payment fill:#e0f2f1,stroke:#009688
    classDef erp fill:#f5f5f5,stroke:#607d8b
    
    class V1,V2,V3,V4 vendor
    class E1,E2,E3,E4 email
    class DA1,DA2,DA3,DA4,DA5,DA6 agent
    class ME1,ME2,ME3,ME4,ME5 matching
    class PE1,PE2,PE3,PE4,PE5 policy
    class AP1,AP2,AP3,AP4 human
    class PS1,PS2,PS3,PS4 payment
    class ERP1,ERP2,ERP3 erp
```

---

## Payment Execution Workflow

```mermaid
flowchart TB
    subgraph "AP Manager"
        APM1[Select Payments]
        APM2[Choose Method]
        APM3[Set Schedule]
        APM4[Review Failures]
        APM5[Approve Retry]
    end
    
    subgraph "Payment Orchestrator"
        PO1[Validate Batch]
        PO2[Check Balances]
        PO3[Route by Method]
        PO4[Queue Execution]
        PO5[Monitor Status]
        PO6[Handle Failures]
    end
    
    subgraph "ACH Processor"
        ACH1[Format NACHA]
        ACH2[Submit to Astra]
        ACH3[Receive ACK]
        ACH4[Track Settlement]
    end
    
    subgraph "RTP Processor"
        RTP1[Format ISO20022]
        RTP2[Submit Real-time]
        RTP3[Instant Confirm]
        RTP4[Update Status]
    end
    
    subgraph "Stablecoin Processor"
        SC1[Get FX Quote]
        SC2[Convert to USDC]
        SC3[Submit Transaction]
        SC4[Confirm on Chain]
        SC5[Track Gas Fees]
    end
    
    subgraph "Check Processor"
        CHK1[Generate Check]
        CHK2[Send to PostGrid]
        CHK3[Track Mailing]
        CHK4[Monitor Clearing]
    end
    
    subgraph "Bank Account"
        BA1[Debit Funds]
        BA2[Record Transaction]
        BA3[Update Balance]
        BA4[Send Statement]
    end
    
    subgraph "Vendor System"
        VS1[Receive Payment]
        VS2[Send Confirmation]
        VS3[Apply to Invoice]
        VS4[Update AR]
    end
    
    subgraph "Reconciliation Engine"
        RE1[Capture Bank Data]
        RE2[Match Payments]
        RE3[Flag Discrepancies]
        RE4[Auto-Reconcile]
    end
    
    %% Main Flow
    APM1 --> APM2
    APM2 --> APM3
    APM3 --> PO1
    
    PO1 --> PO2
    PO2 --> PO3
    PO3 -->|ACH| ACH1
    PO3 -->|RTP| RTP1
    PO3 -->|Crypto| SC1
    PO3 -->|Check| CHK1
    
    %% ACH Flow
    ACH1 --> ACH2
    ACH2 --> ACH3
    ACH3 --> BA1
    ACH3 --> ACH4
    ACH4 --> VS1
    
    %% RTP Flow
    RTP1 --> RTP2
    RTP2 --> BA1
    RTP2 --> RTP3
    RTP3 --> RTP4
    RTP4 --> VS1
    
    %% Stablecoin Flow
    SC1 --> SC2
    SC2 --> SC3
    SC3 --> BA1
    SC3 --> SC4
    SC4 --> SC5
    SC5 --> VS1
    
    %% Check Flow
    CHK1 --> CHK2
    CHK2 --> CHK3
    CHK3 --> VS1
    CHK3 --> CHK4
    CHK4 --> BA1
    
    %% Bank & Vendor Processing
    BA1 --> BA2
    BA2 --> BA3
    BA3 --> BA4
    
    VS1 --> VS2
    VS2 --> VS3
    VS3 --> VS4
    
    %% Reconciliation
    BA4 --> RE1
    VS2 --> RE1
    RE1 --> RE2
    RE2 --> RE3
    RE2 --> RE4
    RE3 --> APM4
    
    %% Error Handling
    PO3 --> PO4
    PO4 --> PO5
    PO5 -->|Failed| PO6
    PO6 --> APM4
    APM4 --> APM5
    APM5 -->|Retry| PO1
    
    classDef manager fill:#e8f5e9,stroke:#4caf50
    classDef orchestrator fill:#e3f2fd,stroke:#2196f3
    classDef processor fill:#fff3e0,stroke:#ff9800
    classDef bank fill:#f3e5f5,stroke:#9c27b0
    classDef vendor fill:#fce4ec,stroke:#e91e63
    classDef recon fill:#e0f2f1,stroke:#009688
    
    class APM1,APM2,APM3,APM4,APM5 manager
    class PO1,PO2,PO3,PO4,PO5,PO6 orchestrator
    class ACH1,ACH2,ACH3,ACH4,RTP1,RTP2,RTP3,RTP4,SC1,SC2,SC3,SC4,SC5,CHK1,CHK2,CHK3,CHK4 processor
    class BA1,BA2,BA3,BA4 bank
    class VS1,VS2,VS3,VS4 vendor
    class RE1,RE2,RE3,RE4 recon
```

---

## Three-Way Reconciliation Process

```mermaid
flowchart TB
    subgraph "Payment System"
        PS1[Payment Executed]
        PS2[Get Payment Details]
        PS3[Queue for Recon]
        PS4[Update Status]
    end
    
    subgraph "Bank Integration"
        BI1[Fetch Transactions]
        BI2[Parse Statement]
        BI3[Extract Details]
        BI4[Match by Reference]
        BI5[Report Missing]
    end
    
    subgraph "QuickBooks API"
        QB1[Fetch Bills]
        QB2[Fetch Payments]
        QB3[Match Records]
        QB4[Create Entry]
        QB5[Update Status]
    end
    
    subgraph "Famous Browser Agent"
        FA1[Login to Famous]
        FA2[Navigate to AP]
        FA3[Search Payment]
        FA4[Extract Data]
        FA5[Apply Payment]
        FA6[Screenshot Proof]
    end
    
    subgraph "Reconciliation Engine"
        RE1[Load Payment Data]
        RE2[Load Bank Data]
        RE3[Load ERP Data]
        RE4[Three-Way Match]
        RE5[Calculate Confidence]
        RE6[Flag Exceptions]
    end
    
    subgraph "Exception Handler"
        EH1[Review Exception]
        EH2[Identify Cause]
        EH3[Manual Match]
        EH4[Create Adjustment]
        EH5[Approve Resolution]
    end
    
    subgraph "AP Manager"
        AM1[Review Queue]
        AM2[Investigate]
        AM3[Resolve Manually]
        AM4[Approve Adjustment]
    end
    
    subgraph "Audit System"
        AS1[Log Match Result]
        AS2[Log Exception]
        AS3[Log Resolution]
        AS4[Generate Report]
    end
    
    %% Main Flow
    PS1 --> PS2
    PS2 --> PS3
    PS3 --> RE1
    
    %% Bank Data Flow
    PS1 -.->|T+1| BI1
    BI1 --> BI2
    BI2 --> BI3
    BI3 --> BI4
    BI4 --> RE2
    BI4 -->|Not Found| BI5
    BI5 --> EH1
    
    %% QuickBooks Flow
    PS1 -.->|Immediate| QB1
    QB1 --> QB2
    QB2 --> QB3
    QB3 -->|Found| QB5
    QB3 -->|Not Found| QB4
    QB4 --> QB5
    QB5 --> RE3
    
    %% Famous Agent Flow
    PS1 -.->|If Famous| FA1
    FA1 --> FA2
    FA2 --> FA3
    FA3 -->|Found| FA4
    FA3 -->|Not Found| FA5
    FA4 --> FA6
    FA5 --> FA6
    FA6 --> RE3
    
    %% Reconciliation Logic
    RE1 --> RE4
    RE2 --> RE4
    RE3 --> RE4
    RE4 --> RE5
    RE5 -->|>95%| PS4
    RE5 -->|<95%| RE6
    RE6 --> EH1
    
    %% Exception Handling
    EH1 --> EH2
    EH2 -->|Amount Diff| EH4
    EH2 -->|Missing Record| EH3
    EH3 --> AM1
    EH4 --> AM1
    
    %% Manual Resolution
    AM1 --> AM2
    AM2 --> AM3
    AM3 --> AM4
    AM4 --> EH5
    EH5 --> PS4
    
    %% Audit Trail
    PS4 --> AS1
    RE6 --> AS2
    EH5 --> AS3
    AS1 --> AS4
    AS2 --> AS4
    AS3 --> AS4
    
    classDef payment fill:#e8f5e9,stroke:#4caf50
    classDef bank fill:#e3f2fd,stroke:#2196f3
    classDef qb fill:#fff3e0,stroke:#ff9800
    classDef famous fill:#fce4ec,stroke:#e91e63
    classDef recon fill:#f3e5f5,stroke:#9c27b0
    classDef exception fill:#ffebee,stroke:#f44336
    classDef human fill:#ffcdd2,stroke:#d32f2f
    classDef audit fill:#e0f2f1,stroke:#009688
    
    class PS1,PS2,PS3,PS4 payment
    class BI1,BI2,BI3,BI4,BI5 bank
    class QB1,QB2,QB3,QB4,QB5 qb
    class FA1,FA2,FA3,FA4,FA5,FA6 famous
    class RE1,RE2,RE3,RE4,RE5,RE6 recon
    class EH1,EH2,EH3,EH4,EH5 exception
    class AM1,AM2,AM3,AM4 human
    class AS1,AS2,AS3,AS4 audit
```

---

## Vendor Onboarding & KYB

```mermaid
flowchart TB
    subgraph "Vendor"
        V1[Submit Application]
        V2[Provide Documents]
        V3[Add Banking Info]
        V4[Sign Agreement]
        V5[Receive Approval]
    end
    
    subgraph "Onboarding Portal"
        OP1[Create Account]
        OP2[Collect Info]
        OP3[Upload Docs]
        OP4[Validate Input]
        OP5[Show Progress]
    end
    
    subgraph "KYB Service"
        KYB1[Verify Business]
        KYB2[Check Registry]
        KYB3[Validate TaxID]
        KYB4[Screen Lists]
        KYB5[Risk Score]
        KYB6[Request More Info]
    end
    
    subgraph "Document Verification"
        DV1[Verify W9/W8]
        DV2[Check Insurance]
        DV3[Validate License]
        DV4[OCR Extract]
        DV5[Cross-Reference]
    end
    
    subgraph "Banking Verification"
        BV1[Validate Routing]
        BV2[Micro-deposits]
        BV3[Account Verify]
        BV4[Plaid Auth]
        BV5[Mark Verified]
    end
    
    subgraph "Compliance Team"
        CT1[Review Application]
        CT2[Check Flags]
        CT3[Manual Review]
        CT4[Approve/Reject]
        CT5[Set Terms]
    end
    
    subgraph "System Setup"
        SS1[Create Vendor]
        SS2[Set Preferences]
        SS3[Configure Payment]
        SS4[Sync to ERP]
        SS5[Enable Portal]
    end
    
    subgraph "Communication Agent"
        CA1[Send Welcome]
        CA2[Request Info]
        CA3[Send Updates]
        CA4[Final Approval]
    end
    
    %% Main Flow
    V1 --> OP1
    OP1 --> OP2
    OP2 --> OP3
    V2 --> OP3
    OP3 --> OP4
    OP4 -->|Valid| KYB1
    OP4 -->|Invalid| OP5
    OP5 -->|Fix| V2
    
    %% KYB Flow
    KYB1 --> KYB2
    KYB2 --> KYB3
    KYB3 --> KYB4
    KYB4 -->|Clean| KYB5
    KYB4 -->|Flagged| CT2
    KYB5 -->|High Risk| CT1
    KYB5 -->|Low Risk| DV1
    KYB5 -->|Need Info| KYB6
    KYB6 --> CA2
    CA2 --> V2
    
    %% Document Flow
    DV1 --> DV2
    DV2 --> DV3
    DV3 --> DV4
    DV4 --> DV5
    DV5 -->|Match| BV1
    DV5 -->|Mismatch| CT3
    
    %% Banking Flow
    V3 --> BV1
    BV1 --> BV2
    BV2 --> BV3
    BV3 --> BV4
    BV4 --> BV5
    BV5 --> CT1
    
    %% Compliance Review
    CT1 --> CT2
    CT2 --> CT3
    CT3 --> CT4
    CT4 -->|Approve| CT5
    CT4 -->|Reject| CA3
    CT5 --> SS1
    
    %% System Setup
    SS1 --> SS2
    SS2 --> SS3
    SS3 --> SS4
    SS4 --> SS5
    SS5 --> CA4
    
    %% Final Steps
    CA4 --> V5
    V5 --> V4
    V4 --> CA1
    
    %% Communication flows
    CA3 -->|Rejection| V1
    OP5 --> CA3
    
    classDef vendor fill:#e8f5e9,stroke:#4caf50
    classDef portal fill:#e3f2fd,stroke:#2196f3
    classDef kyb fill:#fff3e0,stroke:#ff9800
    classDef doc fill:#f3e5f5,stroke:#9c27b0
    classDef bank fill:#fce4ec,stroke:#e91e63
    classDef compliance fill:#ffebee,stroke:#f44336
    classDef system fill:#e0f2f1,stroke:#009688
    classDef comm fill:#f5f5f5,stroke:#607d8b
    
    class V1,V2,V3,V4,V5 vendor
    class OP1,OP2,OP3,OP4,OP5 portal
    class KYB1,KYB2,KYB3,KYB4,KYB5,KYB6 kyb
    class DV1,DV2,DV3,DV4,DV5 doc
    class BV1,BV2,BV3,BV4,BV5 bank
    class CT1,CT2,CT3,CT4,CT5 compliance
    class SS1,SS2,SS3,SS4,SS5 system
    class CA1,CA2,CA3,CA4 comm
```

---

## Famous ERP Browser Automation

```mermaid
flowchart TB
    subgraph "Trigger System"
        TS1[Reconciliation Need]
        TS2[Queue Agent Task]
        TS3[Set Priority]
        TS4[Monitor Progress]
    end
    
    subgraph "Agent Pool"
        AP1[Select Agent]
        AP2[Check Available]
        AP3[Initialize Browser]
        AP4[Load Context]
        AP5[Release Agent]
    end
    
    subgraph "Authentication Manager"
        AM1[Load Credentials]
        AM2[Navigate Login]
        AM3[Enter Username]
        AM4[Enter Password]
        AM5[Handle MFA]
        AM6[Verify Success]
        AM7[Retry Login]
    end
    
    subgraph "Famous Navigation"
        FN1[Go to Home]
        FN2[Open AP Module]
        FN3[Select Function]
        FN4[Wait for Load]
        FN5[Verify Page]
    end
    
    subgraph "Data Operations"
        DO1[Search Bill]
        DO2[Extract Data]
        DO3[Fill Forms]
        DO4[Add Line Items]
        DO5[Submit Changes]
        DO6[Capture Result]
    end
    
    subgraph "Screenshot Service"
        SS1[Capture Screen]
        SS2[Add Timestamp]
        SS3[Upload to S3]
        SS4[Log Location]
    end
    
    subgraph "Error Handler"
        EH1[Detect Error]
        EH2[Classify Error]
        EH3[Retry Logic]
        EH4[Fallback Plan]
        EH5[Alert Human]
    end
    
    subgraph "Audit Logger"
        AL1[Log Action]
        AL2[Log Result]
        AL3[Log Error]
        AL4[Create Report]
    end
    
    %% Main Flow
    TS1 --> TS2
    TS2 --> TS3
    TS3 --> AP1
    
    %% Agent Initialization
    AP1 --> AP2
    AP2 -->|Available| AP3
    AP2 -->|Busy| AP1
    AP3 --> AP4
    AP4 --> AM1
    
    %% Authentication Flow
    AM1 --> AM2
    AM2 --> AM3
    AM3 --> AM4
    AM4 --> AM5
    AM5 --> AM6
    AM6 -->|Success| FN1
    AM6 -->|Failed| AM7
    AM7 -->|Retry| AM2
    AM7 -->|Max Tries| EH5
    
    %% Navigation Flow
    FN1 --> FN2
    FN2 --> FN3
    FN3 --> FN4
    FN4 --> FN5
    FN5 -->|Correct| DO1
    FN5 -->|Wrong| EH1
    
    %% Data Operations
    DO1 --> DO2
    DO2 --> SS1
    DO2 -->|Not Found| DO3
    DO2 -->|Found| DO6
    DO3 --> DO4
    DO4 --> DO5
    DO5 --> DO6
    DO6 --> SS1
    
    %% Screenshot Flow
    SS1 --> SS2
    SS2 --> SS3
    SS3 --> SS4
    SS4 --> AL1
    
    %% Error Handling
    EH1 --> EH2
    EH2 -->|Timeout| EH3
    EH2 -->|Element Missing| EH4
    EH2 -->|Critical| EH5
    EH3 -->|Retry| FN4
    EH4 -->|Alternative| DO3
    EH5 --> AL3
    
    %% Audit Logging
    AL1 --> AL2
    AL2 --> AL4
    AL3 --> AL4
    AL4 --> TS4
    
    %% Cleanup
    TS4 -->|Complete| AP5
    TS4 -->|Monitor| DO1
    EH5 --> AP5
    
    classDef trigger fill:#e8f5e9,stroke:#4caf50
    classDef pool fill:#e3f2fd,stroke:#2196f3
    classDef auth fill:#fff3e0,stroke:#ff9800
    classDef nav fill:#f3e5f5,stroke:#9c27b0
    classDef data fill:#fce4ec,stroke:#e91e63
    classDef screen fill:#e0f2f1,stroke:#009688
    classDef error fill:#ffebee,stroke:#f44336
    classDef audit fill:#f5f5f5,stroke:#607d8b
    
    class TS1,TS2,TS3,TS4 trigger
    class AP1,AP2,AP3,AP4,AP5 pool
    class AM1,AM2,AM3,AM4,AM5,AM6,AM7 auth
    class FN1,FN2,FN3,FN4,FN5 nav
    class DO1,DO2,DO3,DO4,DO5,DO6 data
    class SS1,SS2,SS3,SS4 screen
    class EH1,EH2,EH3,EH4,EH5 error
    class AL1,AL2,AL3,AL4 audit
```

---

## Policy Exception & Approval Flow

```mermaid
flowchart TB
    subgraph "Transaction Input"
        TI1[New Payment]
        TI2[New Bill]
        TI3[PO Change]
        TI4[Vendor Update]
    end
    
    subgraph "Policy Engine"
        PE1[Load Rules]
        PE2[Evaluate Amount]
        PE3[Check Vendor]
        PE4[Check Category]
        PE5[Calculate Score]
        PE6[Determine Action]
    end
    
    subgraph "Exception Detection"
        ED1[Amount Exceeds]
        ED2[Vendor Blocked]
        ED3[Unusual Pattern]
        ED4[Missing Approval]
        ED5[Create Exception]
    end
    
    subgraph "Approval Router"
        AR1[Identify Approvers]
        AR2[Check Hierarchy]
        AR3[Send Request]
        AR4[Set Deadline]
        AR5[Track Response]
    end
    
    subgraph "Manager Level 1"
        M1_1[Receive Alert]
        M1_2[Review Details]
        M1_3[Check History]
        M1_4[Approve/Escalate]
        M1_5[Add Comments]
    end
    
    subgraph "Manager Level 2"
        M2_1[Receive Escalation]
        M2_2[Deep Review]
        M2_3[Check Budget]
        M2_4[Final Decision]
        M2_5[Set Precedent]
    end
    
    subgraph "Exception Handler"
        EH1[Log Exception]
        EH2[Apply Override]
        EH3[Update Policy]
        EH4[Notify Parties]
        EH5[Close Exception]
    end
    
    subgraph "Audit Trail"
        AT1[Record Decision]
        AT2[Log Override]
        AT3[Track Changes]
        AT4[Generate Report]
    end
    
    %% Main Flow
    TI1 --> PE1
    TI2 --> PE1
    TI3 --> PE1
    TI4 --> PE1
    
    PE1 --> PE2
    PE2 --> PE3
    PE3 --> PE4
    PE4 --> PE5
    PE5 --> PE6
    
    %% Exception Detection
    PE6 -->|Block| ED1
    PE6 -->|Flag| ED2
    PE6 -->|Alert| ED3
    PE6 -->|Hold| ED4
    PE6 -->|Pass| AT1
    
    ED1 --> ED5
    ED2 --> ED5
    ED3 --> ED5
    ED4 --> ED5
    ED5 --> AR1
    
    %% Approval Routing
    AR1 --> AR2
    AR2 --> AR3
    AR3 --> AR4
    AR4 --> M1_1
    AR4 --> AR5
    
    %% Level 1 Manager
    M1_1 --> M1_2
    M1_2 --> M1_3
    M1_3 --> M1_4
    M1_4 -->|Approve| M1_5
    M1_4 -->|Escalate| M2_1
    M1_4 -->|Reject| EH4
    M1_5 --> EH1
    
    %% Level 2 Manager
    M2_1 --> M2_2
    M2_2 --> M2_3
    M2_3 --> M2_4
    M2_4 --> M2_5
    M2_5 --> EH2
    
    %% Exception Handling
    EH1 --> EH2
    EH2 --> EH3
    EH3 --> EH4
    EH4 --> EH5
    EH5 --> AT1
    
    %% Audit Trail
    AT1 --> AT2
    AT2 --> AT3
    AT3 --> AT4
    
    %% Response Tracking
    AR5 -->|Timeout| M2_1
    AR5 -->|Response| EH1
    
    %% Policy Learning
    EH3 -->|Update| PE1
    M2_5 -->|New Rule| PE1
    
    classDef input fill:#e8f5e9,stroke:#4caf50
    classDef policy fill:#e3f2fd,stroke:#2196f3
    classDef exception fill:#ffebee,stroke:#f44336
    classDef router fill:#fff3e0,stroke:#ff9800
    classDef manager1 fill:#f3e5f5,stroke:#9c27b0
    classDef manager2 fill:#fce4ec,stroke:#e91e63
    classDef handler fill:#e0f2f1,stroke:#009688
    classDef audit fill:#f5f5f5,stroke:#607d8b
    
    class TI1,TI2,TI3,TI4 input
    class PE1,PE2,PE3,PE4,PE5,PE6 policy
    class ED1,ED2,ED3,ED4,ED5 exception
    class AR1,AR2,AR3,AR4,AR5 router
    class M1_1,M1_2,M1_3,M1_4,M1_5 manager1
    class M2_1,M2_2,M2_3,M2_4,M2_5 manager2
    class EH1,EH2,EH3,EH4,EH5 handler
    class AT1,AT2,AT3,AT4 audit
```

---

## Document Matching & Dispute Resolution

```mermaid
flowchart TB
    subgraph "Document Intake"
        DI1[Receive Invoice]
        DI2[Receive PO]
        DI3[Receive BOL]
        DI4[Parse Documents]
    end
    
    subgraph "Matching Engine"
        ME1[Extract Line Items]
        ME2[Match SKUs]
        ME3[Compare Quantities]
        ME4[Check Prices]
        ME5[Calculate Variance]
        ME6[Apply Tolerances]
    end
    
    subgraph "Variance Detection"
        VD1[Quantity Mismatch]
        VD2[Price Difference]
        VD3[Missing Items]
        VD4[Extra Items]
        VD5[Create Dispute]
    end
    
    subgraph "AP Specialist"
        APS1[Review Variance]
        APS2[Check History]
        APS3[Contact Vendor]
        APS4[Document Issue]
        APS5[Propose Resolution]
    end
    
    subgraph "Vendor Communication"
        VC1[Send Dispute]
        VC2[Receive Response]
        VC3[Negotiate Terms]
        VC4[Agree Resolution]
        VC5[Update Records]
    end
    
    subgraph "Vendor Portal"
        VP1[View Dispute]
        VP2[Upload Evidence]
        VP3[Submit Response]
        VP4[Accept/Counter]
        VP5[Track Status]
    end
    
    subgraph "Resolution Engine"
        RE1[Create Credit Memo]
        RE2[Adjust Invoice]
        RE3[Update PO]
        RE4[Recalculate]
        RE5[Approve Changes]
    end
    
    subgraph "Finance Team"
        FT1[Review Resolution]
        FT2[Approve Adjustment]
        FT3[Update GL]
        FT4[Process Payment]
        FT5[Close Dispute]
    end
    
    %% Main Flow
    DI1 --> DI4
    DI2 --> DI4
    DI3 --> DI4
    DI4 --> ME1
    
    %% Matching Process
    ME1 --> ME2
    ME2 --> ME3
    ME3 --> ME4
    ME4 --> ME5
    ME5 --> ME6
    ME6 -->|Within| FT4
    ME6 -->|Outside| VD1
    
    %% Variance Detection
    ME5 -->|Qty Diff| VD1
    ME5 -->|Price Diff| VD2
    ME5 -->|Missing| VD3
    ME5 -->|Extra| VD4
    VD1 --> VD5
    VD2 --> VD5
    VD3 --> VD5
    VD4 --> VD5
    VD5 --> APS1
    
    %% AP Specialist Review
    APS1 --> APS2
    APS2 --> APS3
    APS3 --> APS4
    APS4 --> APS5
    APS5 --> VC1
    
    %% Vendor Communication
    VC1 --> VP1
    VP1 --> VP2
    VP2 --> VP3
    VP3 --> VC2
    VC2 --> VC3
    VC3 -->|Agree| VC4
    VC3 -->|Dispute| VP4
    VP4 -->|Counter| APS1
    VP4 -->|Accept| VC4
    VC4 --> VC5
    VC5 --> RE1
    
    %% Resolution Process
    RE1 --> RE2
    RE2 --> RE3
    RE3 --> RE4
    RE4 --> RE5
    RE5 --> FT1
    
    %% Finance Approval
    FT1 --> FT2
    FT2 --> FT3
    FT3 --> FT4
    FT4 --> FT5
    FT5 --> VP5
    
    %% Status Updates
    VP5 --> VC2
    APS4 --> VP5
    
    classDef intake fill:#e8f5e9,stroke:#4caf50
    classDef matching fill:#e3f2fd,stroke:#2196f3
    classDef variance fill:#ffebee,stroke:#f44336
    classDef specialist fill:#fff3e0,stroke:#ff9800
    classDef vendor fill:#f3e5f5,stroke:#9c27b0
    classDef portal fill:#fce4ec,stroke:#e91e63
    classDef resolution fill:#e0f2f1,stroke:#009688
    classDef finance fill:#f5f5f5,stroke:#607d8b
    
    class DI1,DI2,DI3,DI4 intake
    class ME1,ME2,ME3,ME4,ME5,ME6 matching
    class VD1,VD2,VD3,VD4,VD5 variance
    class APS1,APS2,APS3,APS4,APS5 specialist
    class VC1,VC2,VC3,VC4,VC5 vendor
    class VP1,VP2,VP3,VP4,VP5 portal
    class RE1,RE2,RE3,RE4,RE5 resolution
    class FT1,FT2,FT3,FT4,FT5 finance
```

---

## Multi-Channel Payment Scheduling

```mermaid
flowchart TB
    subgraph "Payment Queue"
        PQ1[Approved Bills]
        PQ2[Due Dates]
        PQ3[Priority Sort]
        PQ4[Batch Creation]
    end
    
    subgraph "Optimization Engine"
        OE1[Check Cash Flow]
        OE2[Calculate Discounts]
        OE3[Evaluate Float]
        OE4[Check Rebates]
        OE5[Optimize Schedule]
    end
    
    subgraph "Method Selection"
        MS1[Check Vendor Pref]
        MS2[Evaluate Urgency]
        MS3[Check Amount]
        MS4[Consider Fees]
        MS5[Select Method]
    end
    
    subgraph "Schedule Builder"
        SB1[Group by Method]
        SB2[Group by Date]
        SB3[Check Limits]
        SB4[Create Batches]
        SB5[Set Execution Time]
    end
    
    subgraph "Treasury Review"
        TR1[Review Schedule]
        TR2[Check Balances]
        TR3[Approve Batches]
        TR4[Adjust Timing]
        TR5[Lock Schedule]
    end
    
    subgraph "Execution Queue"
        EQ1[Load Batch]
        EQ2[Validate Accounts]
        EQ3[Process Payments]
        EQ4[Track Status]
        EQ5[Handle Errors]
    end
    
    subgraph "Multi-Rail Processor"
        MR1[Route ACH Batch]
        MR2[Process RTP]
        MR3[Print Checks]
        MR4[Send Wires]
        MR5[Execute Stablecoin]
    end
    
    subgraph "Notification Service"
        NS1[Vendor Emails]
        NS2[Internal Alerts]
        NS3[Status Updates]
        NS4[Error Notices]
        NS5[Daily Summary]
    end
    
    %% Main Flow
    PQ1 --> PQ2
    PQ2 --> PQ3
    PQ3 --> PQ4
    PQ4 --> OE1
    
    %% Optimization
    OE1 --> OE2
    OE2 --> OE3
    OE3 --> OE4
    OE4 --> OE5
    OE5 --> MS1
    
    %% Method Selection
    MS1 --> MS2
    MS2 --> MS3
    MS3 --> MS4
    MS4 --> MS5
    MS5 --> SB1
    
    %% Schedule Building
    SB1 --> SB2
    SB2 --> SB3
    SB3 --> SB4
    SB4 --> SB5
    SB5 --> TR1
    
    %% Treasury Review
    TR1 --> TR2
    TR2 --> TR3
    TR3 -->|Approved| TR5
    TR3 -->|Adjust| TR4
    TR4 --> SB5
    TR5 --> EQ1
    
    %% Execution
    EQ1 --> EQ2
    EQ2 --> EQ3
    EQ3 --> MR1
    EQ3 --> MR2
    EQ3 --> MR3
    EQ3 --> MR4
    EQ3 --> MR5
    
    %% Processing
    MR1 --> EQ4
    MR2 --> EQ4
    MR3 --> EQ4
    MR4 --> EQ4
    MR5 --> EQ4
    EQ4 -->|Success| NS1
    EQ4 -->|Error| EQ5
    EQ5 --> NS4
    
    %% Notifications
    NS1 --> NS3
    EQ5 --> NS2
    NS3 --> NS5
    NS4 --> TR1
    
    %% Feedback Loop
    NS5 --> PQ1
    EQ5 -->|Retry| EQ2
    
    classDef queue fill:#e8f5e9,stroke:#4caf50
    classDef optimize fill:#e3f2fd,stroke:#2196f3
    classDef method fill:#fff3e0,stroke:#ff9800
    classDef schedule fill:#f3e5f5,stroke:#9c27b0
    classDef treasury fill:#ffebee,stroke:#f44336
    classDef execution fill:#fce4ec,stroke:#e91e63
    classDef processor fill:#e0f2f1,stroke:#009688
    classDef notify fill:#f5f5f5,stroke:#607d8b
    
    class PQ1,PQ2,PQ3,PQ4 queue
    class OE1,OE2,OE3,OE4,OE5 optimize
    class MS1,MS2,MS3,MS4,MS5 method
    class SB1,SB2,SB3,SB4,SB5 schedule
    class TR1,TR2,TR3,TR4,TR5 treasury
    class EQ1,EQ2,EQ3,EQ4,EQ5 execution
    class MR1,MR2,MR3,MR4,MR5 processor
    class NS1,NS2,NS3,NS4,NS5 notify
```

---

## Summary

These swimlane diagrams illustrate the key cross-functional processes in the Agentic Payments platform:

1. **End-to-End Invoice Processing** - Shows the complete flow from vendor invoice submission through payment execution across 8 different actors/systems

2. **Payment Execution Workflow** - Details the multi-rail payment processing with different payment methods (ACH, RTP, Stablecoin, Check) and their respective flows

3. **Three-Way Reconciliation Process** - Illustrates the complex matching between Payment System, Bank, and ERP (both API-based and browser-automated)

4. **Vendor Onboarding & KYB** - Complete vendor onboarding with KYB verification, document checks, and banking validation

5. **Famous ERP Browser Automation** - Detailed browser agent workflow showing authentication, navigation, data operations, and error handling

6. **Policy Exception & Approval Flow** - Multi-level approval routing with exception handling and policy learning

7. **Document Matching & Dispute Resolution** - Variance detection, vendor communication, and dispute resolution process

8. **Multi-Channel Payment Scheduling** - Payment optimization, method selection, and batch execution across multiple payment rails

Each diagram clearly shows:
- **Actors/Systems** in separate swimlanes
- **Process flow** moving forward and backward between lanes
- **Decision points** and conditional logic
- **Error handling** and retry mechanisms
- **Feedback loops** where processes return to earlier stages

The color coding helps distinguish between different types of actors (vendors, agents, humans, systems) making the diagrams easy to follow and understand.