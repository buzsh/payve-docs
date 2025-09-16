# Agentic Payments - Technical Architecture Document

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [System Architecture Overview](#system-architecture-overview)
3. [Core Workflows](#core-workflows)
4. [Data Models & Structures](#data-models--structures)
5. [Technology Stack](#technology-stack)
6. [Agentic Workflow Architecture](#agentic-workflow-architecture)
7. [API Architecture](#api-architecture)
8. [Integration Architecture](#integration-architecture)
9. [Security & Compliance](#security--compliance)
10. [Implementation Roadmap](#implementation-roadmap)

---

## Executive Summary

Agentic Payments is an AI-native Accounts Payable (AP) operating system designed specifically for supply chain businesses. The platform leverages intelligent automation through browser-use agents (particularly for API-less systems like Famous), ML-powered document processing, and sophisticated payment orchestration across multiple rails.

### Key Technical Differentiators
- **Browser Automation Agents**: Handle non-API ERPs (Famous) through intelligent UI automation
- **Multi-Rail Payment Engine**: ACH, RTP, Check, International Stablecoin (USDC)
- **Intelligent Document Processing**: OCR with line-level matching and anomaly detection
- **Real-time Reconciliation**: Automated 3-way matching with 95% auto-reconciliation target
- **Immutable Audit Trail**: Event-sourced architecture for complete compliance

---

## System Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           Frontend Layer                            │
│  ┌────────────────┐  ┌──────────────┐  ┌───────────────┐            │
│  │ Command Center │  │ Scheduled    │  │ Reconciliation│            │
│  │ Dashboard      │  │ Payments     │  │ Workbench     │            │
│  └────────────────┘  └──────────────┘  └───────────────┘            │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         API Gateway Layer                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ REST API     │  │ GraphQL API  │  │ SSE (AI/LLM) │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        Core Services Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ Payment      │  │ Document     │  │ Policy       │               │
│  │ Orchestrator │  │ Processor    │  │ Engine       │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
│  ┌───────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │ Reconciliation│  │ Vendor       │  │ Audit        │              │
│  │ Engine        │  │ Management   │  │ Service      │              │
│  └───────────────┘  └──────────────┘  └──────────────┘              │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       Agentic Layer                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐              │
│  │ Browser      │  │ Document     │  │ Communication │              │
│  │ Automation   │  │ Extraction   │  │ Agent         │              │
│  │ Agent        │  │ Agent        │  │               │              │
│  └──────────────┘  └──────────────┘  └───────────────┘              │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    Integration & Data Layer                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ QuickBooks   │  │ Famous ERP   │  │ Bank/Payment │               │
│  │ API          │  │ (Browser)    │  │ Providers    │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ MongoDB      │  │ Redis        │  │ S3 Storage   │               │
│  │ (Primary DB) │  │ (Cache/Queue)│  │ (Documents)  │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Core Workflows

### 1. Document Ingestion & Processing Workflow

```mermaid
graph TD
    A[Email/Upload Receipt] --> B[Document Processor Agent]
    B --> C{Document Type?}
    C -->|Invoice| D[Invoice Parser]
    C -->|PO| E[PO Parser]
    C -->|BOL| F[BOL Parser]
    
    D --> G[Extract Line Items]
    E --> G
    F --> G
    
    G --> H[Duplicate Check]
    H --> I{Duplicate?}
    I -->|Yes| J[Flag & Queue]
    I -->|No| K[Create Bill Record]
    
    K --> L[Match to PO]
    L --> M{Match Success?}
    M -->|Yes| N[Apply Policy Rules]
    M -->|No| O[Manual Review Queue]
    
    N --> P[Schedule Payment]
```

### 2. Payment Execution Workflow

```mermaid
graph TD
    A[Scheduled Payment] --> B[Payment Orchestrator]
    B --> C{Payment Method}
    
    C -->|ACH| D[Astra ACH API]
    C -->|RTP| E[Astra RTP API]
    C -->|Check| F[PostGrid API]
    C -->|Stablecoin| G[Bridge/Astra USDC]
    
    D --> H[Execute Payment]
    E --> H
    F --> H
    G --> H
    
    H --> I[Update Payment Status]
    I --> J[Send Remittance]
    J --> K[Trigger Reconciliation]
    K --> L[Update Audit Log]
```

### 3. Browser Automation Workflow (Famous ERP)

```mermaid
graph TD
    A[Reconciliation Trigger] --> B[Browser Agent Init]
    B --> C[Login to Famous]
    C --> D[Navigate to AP Module]
    
    D --> E[Search for Bill/PO]
    E --> F{Found?}
    F -->|Yes| G[Extract Details]
    F -->|No| H[Create New Entry]
    
    G --> I[Compare with Internal]
    I --> J{Match?}
    J -->|Yes| K[Mark as Reconciled]
    J -->|No| L[Flag Discrepancy]
    
    H --> M[Input Bill Details]
    M --> N[Submit & Capture ID]
    N --> K
    
    K --> O[Update Internal Records]
    O --> P[Generate Audit Entry]
```

---

## Data Models & Structures

### Core Entity Models

#### 1. Organization & Tenant
```typescript
interface Organization {
  id: string; // UUID
  name: string;
  subdomain: string;
  settings: {
    emailIngestionAddress: string;
    defaultCurrency: string;
    timezone: string;
    fiscalYearStart: Date;
  };
  subscription: {
    plan: 'starter' | 'professional' | 'enterprise';
    status: 'active' | 'suspended' | 'cancelled';
    billingCycle: 'monthly' | 'annual';
  };
  createdAt: Date;
  updatedAt: Date;
}
```

#### 2. Vendor Model
```typescript
interface Vendor {
  id: string; // UUID
  organizationId: string;
  erpVendorId?: string; // External ERP ID
  
  // Basic Information
  legalName: string;
  dba?: string; // Doing Business As
  taxId?: string; // EIN or equivalent
  
  // Status & Standing
  status: 'active' | 'inactive' | 'onboarding' | 'suspended';
  standing: 'good' | 'warning' | 'blocked' | 'pending';
  riskScore?: number; // 0-100
  
  // Payment Configuration
  paymentPreferences: {
    defaultMethod: 'ACH' | 'RTP' | 'Check' | 'Wire' | 'Stablecoin';
    paymentTerms: 'Net15' | 'Net30' | 'Net45' | 'Net60' | 'Custom';
    customTermsDays?: number;
    earlyPaymentDiscount?: {
      percentage: number;
      withinDays: number;
    };
  };
  
  // Banking Information
  bankAccounts: {
    domestic: Array<{
      id: string;
      type: 'ACH' | 'RTP';
      routingNumber: string; // Encrypted
      accountNumber: string; // Encrypted
      accountType: 'checking' | 'savings';
      verificationStatus: 'pending' | 'verified' | 'failed';
      verifiedAt?: Date;
      isDefault: boolean;
    }>;
    international: Array<{
      id: string;
      type: 'SWIFT' | 'IBAN' | 'Stablecoin';
      swiftCode?: string;
      iban?: string;
      walletAddress?: string; // For crypto
      bankName: string;
      country: string;
      currency: string;
      isDefault: boolean;
    }>;
  };
  
  // Contact Information
  contacts: Array<{
    id: string;
    type: 'primary' | 'ap' | 'remittance' | 'escalation';
    name: string;
    email: string;
    phone?: string;
    isActive: boolean;
  }>;
  
  // Addresses
  addresses: {
    remittance?: Address;
    physical?: Address;
    billing?: Address;
  };
  
  // Compliance & Verification
  kyb: {
    status: 'not_started' | 'pending' | 'approved' | 'rejected';
    completedAt?: Date;
    documents: Array<{
      type: string;
      url: string;
      uploadedAt: Date;
    }>;
  };
  
  // Metadata
  tags: string[];
  customFields: Record<string, any>;
  createdAt: Date;
  updatedAt: Date;
  createdBy: string; // User ID
  updatedBy: string; // User ID
}

interface Address {
  line1: string;
  line2?: string;
  city: string;
  state: string;
  postalCode: string;
  country: string;
}
```

#### 3. Purchase Order Model
```typescript
interface PurchaseOrder {
  id: string; // UUID
  organizationId: string;
  poNumber: string; // Human-readable PO number
  erpPoId?: string; // External ERP ID
  
  // Vendor Association
  vendorId: string;
  vendorName: string; // Denormalized for performance
  
  // Order Details
  orderDate: Date;
  expectedDeliveryDate?: Date;
  actualDeliveryDate?: Date;
  
  // Financial Details
  currency: string;
  amounts: {
    subtotal: number;
    tax: number;
    shipping: number;
    otherCharges: number;
    total: number;
  };
  
  // Line Items
  lineItems: Array<{
    id: string;
    lineNumber: number;
    sku: string;
    description: string;
    category?: string;
    quantity: number;
    unitOfMeasure: string;
    unitPrice: number;
    totalPrice: number;
    taxRate?: number;
    taxAmount?: number;
    
    // Matching & Reconciliation
    matchingTolerance?: {
      quantityPercent: number;
      pricePercent: number;
    };
    
    // Fulfillment Tracking
    receivedQuantity?: number;
    receivedDate?: Date;
    discrepancies?: Array<{
      type: 'quantity' | 'price' | 'quality';
      description: string;
      reportedAt: Date;
    }>;
  }>;
  
  // Status Tracking
  status: {
    current: 'draft' | 'submitted' | 'approved' | 'partial' | 'fulfilled' | 'cancelled';
    history: Array<{
      status: string;
      changedAt: Date;
      changedBy: string;
      reason?: string;
    }>;
  };
  
  // Payment Terms
  paymentTerms: {
    dueDate: Date;
    terms: string; // "Net 30", etc.
    discountTerms?: string; // "2/10 Net 30"
  };
  
  // Associated Documents
  documents: Array<{
    id: string;
    type: 'po' | 'quote' | 'contract' | 'other';
    name: string;
    url: string; // S3 URL
    mimeType: string;
    size: number;
    uploadedAt: Date;
    uploadedBy: string;
  }>;
  
  // Workflow & Approvals
  approvals: {
    required: boolean;
    threshold?: number;
    chain: Array<{
      userId: string;
      role: string;
      approvedAt?: Date;
      status: 'pending' | 'approved' | 'rejected';
      comments?: string;
    }>;
  };
  
  // Links to Other Entities
  linkedBillIds: string[];
  linkedPaymentIds: string[];
  
  // Metadata
  department?: string;
  costCenter?: string;
  project?: string;
  tags: string[];
  customFields: Record<string, any>;
  
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
  updatedBy: string;
}
```

#### 4. Bill/Invoice Model
```typescript
interface Bill {
  id: string; // UUID
  organizationId: string;
  billNumber: string; // Vendor's invoice number
  erpBillId?: string; // External ERP ID
  
  // Vendor Association
  vendorId: string;
  vendorName: string;
  
  // PO Association
  purchaseOrderIds: string[]; // Can be multiple POs
  
  // Invoice Details
  invoiceDate: Date;
  receivedDate: Date;
  dueDate: Date;
  
  // Financial Details
  currency: string;
  amounts: {
    subtotal: number;
    tax: number;
    shipping: number;
    otherCharges: number;
    discount?: number;
    total: number;
  };
  
  // Line Items (from invoice)
  lineItems: Array<{
    id: string;
    lineNumber: number;
    description: string;
    quantity: number;
    unitPrice: number;
    totalPrice: number;
    
    // PO Matching
    matchedPoLineItemId?: string;
    matchingStatus: 'matched' | 'partial' | 'unmatched' | 'disputed';
    matchingDiscrepancies?: {
      quantityVariance?: number;
      priceVariance?: number;
      description?: string;
    };
  }>;
  
  // Processing Status
  status: {
    current: 'received' | 'processing' | 'matched' | 'approved' | 
             'scheduled' | 'paid' | 'reconciled' | 'disputed';
    history: Array<{
      status: string;
      timestamp: Date;
      userId?: string;
      agentId?: string;
      notes?: string;
    }>;
  };
  
  // Matching Results
  matching: {
    status: 'pending' | 'auto_matched' | 'manual_matched' | 'failed';
    method: 'auto' | 'manual' | 'agent';
    confidence?: number; // 0-100
    performedAt?: Date;
    performedBy?: string; // User or Agent ID
    
    // Three-way match components
    poMatch?: {
      matched: boolean;
      poId: string;
      variances: Record<string, any>;
    };
    receiptMatch?: {
      matched: boolean;
      receiptId?: string;
      variances: Record<string, any>;
    };
  };
  
  // Policy Evaluation
  policyEvaluation: {
    evaluatedAt?: Date;
    violations: Array<{
      ruleId: string;
      ruleName: string;
      severity: 'info' | 'warning' | 'blocking';
      description: string;
      suggestedAction?: string;
    }>;
    approvalRequired: boolean;
    approvalThreshold?: number;
  };
  
  // Documents
  documents: Array<{
    id: string;
    type: 'invoice' | 'bol' | 'receipt' | 'other';
    name: string;
    url: string;
    mimeType: string;
    hash: string; // For duplicate detection
    ocrProcessed: boolean;
    ocrConfidence?: number;
    uploadedAt: Date;
  }>;
  
  // Duplicate Detection
  duplicateCheck: {
    performedAt: Date;
    isDuplicate: boolean;
    duplicateOf?: string; // Bill ID if duplicate
    checksum: string;
  };
  
  // Payment Information
  payment?: {
    scheduledDate?: Date;
    method?: 'ACH' | 'RTP' | 'Check' | 'Wire' | 'Stablecoin';
    paymentId?: string;
    paidDate?: Date;
    paidAmount?: number;
  };
  
  // Metadata
  source: 'email' | 'api' | 'manual' | 'erp_sync';
  emailMessageId?: string;
  tags: string[];
  notes?: string;
  customFields: Record<string, any>;
  
  createdAt: Date;
  updatedAt: Date;
  createdBy?: string;
  processedBy?: string; // User or Agent ID
}
```

#### 5. Payment Model
```typescript
interface Payment {
  id: string; // UUID
  organizationId: string;
  paymentNumber: string; // Human-readable
  
  // Vendor & Financial Info
  vendorId: string;
  vendorName: string;
  
  // Linked Entities
  billIds: string[];
  purchaseOrderIds: string[];
  
  // Payment Details
  amount: number;
  currency: string;
  
  // Payment Method & Rail
  method: 'ACH' | 'RTP' | 'Check' | 'Wire' | 'Stablecoin';
  rail: {
    provider: 'Astra' | 'Bridge' | 'PostGrid';
    providerPaymentId?: string;
    providerStatus?: string;
    
    // Method-specific details
    achDetails?: {
      type: 'standard' | 'same_day';
      routingNumber: string; // Encrypted
      accountNumber: string; // Encrypted
      traceNumber?: string;
    };
    
    rtpDetails?: {
      clearingSystemReference?: string;
      endToEndId?: string;
    };
    
    checkDetails?: {
      checkNumber?: string;
      mailedDate?: Date;
      expectedDeliveryDate?: Date;
      voidedAt?: Date;
      clearedAt?: Date;
    };
    
    wireDetails?: {
      swiftCode?: string;
      iban?: string;
      intermediaryBank?: string;
      reference?: string;
    };
    
    stablecoinDetails?: {
      network: 'ethereum' | 'polygon' | 'solana';
      tokenAddress: string;
      walletAddress: string;
      transactionHash?: string;
      gasUsed?: number;
      gasCost?: number;
    };
  };
  
  // Scheduling & Execution
  scheduling: {
    scheduledDate: Date;
    scheduledBy: string; // User ID
    scheduledAt: Date;
    
    // Optimization flags
    optimizationApplied?: {
      quickPayDiscount?: number;
      floatDays?: number;
      creditUtilization?: number;
    };
  };
  
  // Status & Timeline
  status: {
    current: 'scheduled' | 'pending' | 'processing' | 'sent' | 
             'settled' | 'failed' | 'cancelled' | 'returned';
    
    timeline: Array<{
      status: string;
      timestamp: Date;
      metadata?: Record<string, any>;
    }>;
  };
  
  // Execution Details
  execution: {
    initiatedAt?: Date;
    sentAt?: Date;
    settledAt?: Date;
    failedAt?: Date;
    
    attempts: Array<{
      attemptNumber: number;
      timestamp: Date;
      status: 'success' | 'failed';
      errorCode?: string;
      errorMessage?: string;
    }>;
    
    fees?: {
      bankFee?: number;
      providerFee?: number;
      totalFee?: number;
    };
  };
  
  // Bank Account
  bankAccount: {
    id: string;
    institution: string;
    last4: string;
    type: 'checking' | 'savings';
  };
  
  // Remittance
  remittance: {
    sent: boolean;
    sentAt?: Date;
    recipients: string[]; // Email addresses
    documents: Array<{
      type: 'pdf' | 'csv';
      url: string;
    }>;
  };
  
  // Reconciliation
  reconciliation: {
    status: 'pending' | 'matched' | 'exception' | 'manual';
    reconciledAt?: Date;
    bankTransactionId?: string;
    erpPaymentId?: string;
    
    exceptions?: Array<{
      type: 'amount_mismatch' | 'missing_bank' | 'missing_erp' | 'duplicate';
      description: string;
      resolvedAt?: Date;
      resolvedBy?: string;
    }>;
  };
  
  // Approvals
  approvals: Array<{
    userId: string;
    role: string;
    approvedAt: Date;
    comments?: string;
  }>;
  
  // Policy Overrides
  policyOverrides?: Array<{
    ruleId: string;
    originalValue: any;
    overrideValue: any;
    reason: string;
    approvedBy: string;
    approvedAt: Date;
  }>;
  
  // Metadata
  notes?: string;
  tags: string[];
  customFields: Record<string, any>;
  
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
  lastModifiedBy: string;
}
```

#### 6. Reconciliation Record Model
```typescript
interface ReconciliationRecord {
  id: string;
  organizationId: string;
  
  // Core Associations
  paymentId: string;
  vendorId: string;
  
  // Three-way match components
  matches: {
    internal: {
      paymentId: string;
      amount: number;
      date: Date;
      status: 'found' | 'missing';
    };
    
    bank: {
      transactionId?: string;
      amount?: number;
      date?: Date;
      description?: string;
      status: 'found' | 'missing' | 'pending';
      source: 'plaid' | 'manual' | 'statement';
    };
    
    erp: {
      documentId?: string; // Bill or Payment ID in ERP
      amount?: number;
      date?: Date;
      status: 'found' | 'missing' | 'error';
      source: 'api' | 'agent' | 'manual';
      
      // Agent-specific for Famous
      agentExecution?: {
        agentId: string;
        executedAt: Date;
        screenshots: string[]; // S3 URLs
        actions: Array<{
          action: string;
          timestamp: Date;
          result: string;
        }>;
      };
    };
  };
  
  // Reconciliation Results
  result: {
    status: 'fully_matched' | 'partial_match' | 'exception' | 'manual_review';
    confidence: number; // 0-100
    
    variances?: {
      amountVariance?: number;
      dateVariance?: number; // Days
      description?: string;
    };
    
    autoReconciled: boolean;
    reconciledAt?: Date;
    reconciledBy?: string; // User or System
    
    manualAdjustments?: Array<{
      field: string;
      originalValue: any;
      adjustedValue: any;
      reason: string;
      adjustedBy: string;
      adjustedAt: Date;
    }>;
  };
  
  // Rules Applied
  rulesApplied: Array<{
    ruleId: string;
    ruleName: string;
    matched: boolean;
    confidence: number;
  }>;
  
  // Processing Metadata
  processing: {
    attemptCount: number;
    lastAttemptAt: Date;
    nextRetryAt?: Date;
    
    errors?: Array<{
      timestamp: Date;
      source: 'bank' | 'erp' | 'internal';
      error: string;
      stackTrace?: string;
    }>;
  };
  
  createdAt: Date;
  updatedAt: Date;
  completedAt?: Date;
}
```

#### 7. Policy & Exception Models
```typescript
interface PolicyRule {
  id: string;
  organizationId: string;
  
  // Rule Definition
  name: string;
  description: string;
  category: 'approval' | 'matching' | 'payment' | 'vendor' | 'anomaly';
  
  // Scope
  scope: {
    level: 'global' | 'vendor' | 'category' | 'po';
    vendorIds?: string[];
    categories?: string[];
    poIds?: string[];
  };
  
  // Conditions
  conditions: Array<{
    field: string; // e.g., "amount", "shipping", "unitPrice"
    operator: 'equals' | 'greater' | 'less' | 'between' | 'in' | 'not_in' | 'contains';
    value: any;
    dataType: 'number' | 'string' | 'date' | 'boolean';
  }>;
  
  // Logic
  conditionLogic: 'all' | 'any' | 'custom';
  customLogic?: string; // For complex boolean expressions
  
  // Actions
  actions: Array<{
    type: 'require_approval' | 'flag' | 'block' | 'notify' | 'auto_approve';
    
    approvalConfig?: {
      approverRoles: string[];
      approverIds?: string[];
      threshold?: number;
      escalationAfterHours?: number;
    };
    
    notificationConfig?: {
      recipients: string[];
      template: string;
      channels: ('email' | 'slack' | 'webhook')[];
    };
  }>;
  
  // Priority & Status
  priority: number; // Lower number = higher priority
  enabled: boolean;
  
  // Inheritance
  inheritance: {
    canBeOverridden: boolean;
    inheritedFrom?: string; // Parent rule ID
    overrides?: string[]; // Child rule IDs
  };
  
  // Analytics
  statistics?: {
    timesTriggered: number;
    lastTriggered?: Date;
    avgResolutionTime?: number; // Hours
  };
  
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
  updatedBy: string;
}

interface PolicyException {
  id: string;
  organizationId: string;
  
  // Rule Association
  ruleId: string;
  ruleName: string;
  
  // Exception Context
  context: {
    entityType: 'payment' | 'bill' | 'po' | 'vendor';
    entityId: string;
    entityDetails: Record<string, any>;
  };
  
  // Exception Details
  exception: {
    type: 'override' | 'waiver' | 'adjustment';
    
    originalValue: any;
    exceptionValue: any;
    
    reason: string;
    justification?: string;
    
    expiresAt?: Date;
    
    // Scope of exception
    scope: 'one_time' | 'entity' | 'vendor' | 'permanent';
    
    // If promoted to higher scope
    promotedTo?: {
      scope: 'vendor' | 'global';
      promotedAt: Date;
      promotedBy: string;
    };
  };
  
  // Approval Chain
  approvals: Array<{
    userId: string;
    role: string;
    action: 'approved' | 'rejected';
    timestamp: Date;
    comments?: string;
  }>;
  
  // Impact Tracking
  impact: {
    affectedPayments?: string[];
    affectedBills?: string[];
    financialImpact?: number;
  };
  
  createdAt: Date;
  createdBy: string;
  resolvedAt?: Date;
  resolvedBy?: string;
}
```

#### 8. Audit Event Model
```typescript
interface AuditEvent {
  id: string;
  organizationId: string;
  
  // Event Classification
  category: 'payment' | 'reconciliation' | 'policy' | 'vendor' | 
            'document' | 'system' | 'user' | 'agent';
  
  action: string; // e.g., "payment.executed", "bill.matched", "policy.overridden"
  
  // Actor Information
  actor: {
    type: 'user' | 'agent' | 'system' | 'vendor' | 'integration';
    id: string;
    name: string;
    
    // For agents
    agentDetails?: {
      agentType: 'browser' | 'document' | 'communication' | 'reconciliation';
      version: string;
      executionId: string;
    };
    
    // For users
    userDetails?: {
      email: string;
      role: string;
      ipAddress?: string;
      userAgent?: string;
    };
  };
  
  // Event Details
  event: {
    timestamp: Date;
    
    // What was affected
    entities: Array<{
      type: string;
      id: string;
      name?: string;
    }>;
    
    // State changes
    changes?: {
      before: Record<string, any>;
      after: Record<string, any>;
      delta: Record<string, any>;
    };
    
    // Additional context
    metadata: Record<string, any>;
    
    // For failures
    error?: {
      code: string;
      message: string;
      stackTrace?: string;
    };
  };
  
  // Attachments & Evidence
  attachments: Array<{
    id: string;
    type: 'screenshot' | 'document' | 'log' | 'trace';
    name: string;
    url: string;
    mimeType: string;
    size: number;
  }>;
  
  // Compliance & Security
  compliance: {
    requiresReview: boolean;
    reviewedAt?: Date;
    reviewedBy?: string;
    
    tags: string[]; // e.g., ["pii", "financial", "critical"]
    
    retentionPolicy: {
      retainUntil: Date;
      legalHold: boolean;
    };
  };
  
  // Tracing
  tracing: {
    traceId: string; // For distributed tracing
    spanId: string;
    parentSpanId?: string;
    
    // Related events
    relatedEventIds: string[];
    
    // Session tracking
    sessionId?: string;
    requestId?: string;
  };
  
  // Indexing
  searchableText?: string; // Denormalized for full-text search
  tags: string[];
  
  createdAt: Date;
  ttl?: number; // Time to live in seconds
}
```

#### 9. Agent Execution Model
```typescript
interface AgentExecution {
  id: string;
  organizationId: string;
  
  // Agent Configuration
  agent: {
    type: 'browser' | 'document' | 'communication' | 'reconciliation';
    name: string;
    version: string;
    
    // For browser agents
    browserConfig?: {
      targetSystem: 'famous' | 'netsuite' | 'custom';
      headless: boolean;
      viewport: {
        width: number;
        height: number;
      };
    };
  };
  
  // Execution Context
  context: {
    triggeredBy: 'scheduled' | 'manual' | 'event' | 'api';
    triggerSource?: string; // User ID, Event ID, etc.
    
    purpose: string;
    
    inputs: Record<string, any>;
    
    relatedEntities: Array<{
      type: string;
      id: string;
    }>;
  };
  
  // Execution Status
  status: {
    current: 'queued' | 'running' | 'completed' | 'failed' | 'cancelled';
    
    startedAt?: Date;
    completedAt?: Date;
    
    progress?: {
      current: number;
      total: number;
      message: string;
    };
  };
  
  // Execution Steps (for browser agents)
  steps?: Array<{
    id: string;
    name: string;
    type: 'navigate' | 'click' | 'type' | 'wait' | 'extract' | 'screenshot';
    
    selector?: string;
    value?: any;
    
    status: 'pending' | 'running' | 'completed' | 'failed';
    
    startedAt?: Date;
    completedAt?: Date;
    
    result?: any;
    error?: string;
    
    screenshot?: string; // S3 URL
  }>;
  
  // Results
  results?: {
    success: boolean;
    
    outputs: Record<string, any>;
    
    // For document agents
    extractedData?: {
      vendor?: string;
      amount?: number;
      invoiceNumber?: string;
      lineItems?: any[];
      confidence: number;
    };
    
    // For browser agents
    browserResults?: {
      pagesVisited: number;
      elementsInteracted: number;
      dataExtracted: Record<string, any>;
    };
    
    // For communication agents
    communicationResults?: {
      messagesProcessed: number;
      responsesSent: number;
      escalationsCreated: number;
    };
  };
  
  // Error Handling
  errors?: Array<{
    timestamp: Date;
    step?: string;
    error: string;
    stackTrace?: string;
    screenshot?: string;
    recoverable: boolean;
  }>;
  
  // Performance Metrics
  metrics: {
    executionTimeMs: number;
    memoryUsedMb?: number;
    apiCallsMade?: number;
    
    // For browser agents
    browserMetrics?: {
      pageLoadTimeMs: number[];
      networkRequests: number;
      consoleErrors: number;
    };
  };
  
  // Audit Trail
  auditLog: Array<{
    timestamp: Date;
    level: 'debug' | 'info' | 'warning' | 'error';
    message: string;
    data?: any;
  }>;
  
  createdAt: Date;
  updatedAt: Date;
}
```

### Data Relationships

```mermaid
erDiagram
    Organization ||--o{ Vendor : has
    Organization ||--o{ PurchaseOrder : has
    Organization ||--o{ Bill : has
    Organization ||--o{ Payment : has
    
    Vendor ||--o{ PurchaseOrder : receives
    Vendor ||--o{ Bill : sends
    Vendor ||--o{ Payment : receives
    
    PurchaseOrder ||--o{ Bill : generates
    PurchaseOrder ||--o{ Payment : triggers
    
    Bill ||--o{ Payment : requires
    Bill ||--|| ReconciliationRecord : has
    
    Payment ||--|| ReconciliationRecord : has
    Payment ||--o{ AuditEvent : generates
    
    PolicyRule ||--o{ PolicyException : creates
    PolicyException ||--o{ Payment : affects
    
    AgentExecution ||--o{ AuditEvent : logs
    AgentExecution ||--o{ ReconciliationRecord : creates
```

---

## Technology Stack

### Core Infrastructure

#### Backend Services
- **Primary Language**: TypeScript/Node.js
- **Framework**: Fastify (high-performance, schema-based)
- **API Layer**: 
  - REST API (Fastify-based)
  - GraphQL (Apollo Server) for complex queries
  - WebSocket (Socket.io) for real-time updates

#### Database & Storage
- **Primary Database**: MongoDB 7.0+
  - Native document storage with flexible schemas
  - Full-text search and Atlas Search capabilities
  - Field-level encryption and client-side field level encryption (CSFLE)
  - Role-based access control (RBAC) for multi-tenancy
  - SCRAM-SHA-256 authentication
  - TLS/SSL encryption for all connections
  
- **Caching Layer**: Redis
  - Session management
  - Job queues (BullMQ)
  - Real-time event pub/sub
  
- **Document Storage**: AWS S3 / MinIO
  - Invoice/PO/BOL storage
  - Audit trail artifacts
  - Agent screenshots

#### Message Queue & Events
- **Queue System**: BullMQ (Redis-based)
  - Payment processing jobs
  - Document processing pipeline
  - Reconciliation tasks
  
- **Event Bus**: EventEmitter2 + Redis Pub/Sub
  - Internal service communication
  - Real-time notifications
  - Audit event streaming

### AI & Automation Stack

#### Document Processing
- **OCR Engine**: 
  - Primary: AWS Textract / Azure Form Recognizer
  - Fallback: Tesseract OCR
  
- **LLM Integration**:
  - OpenAI GPT-4 for document understanding
  - Anthropic Claude for complex extraction
  - Local LLM (Llama 3) for sensitive data

#### Browser Automation (Famous ERP)
- **Framework**: Playwright
  - Headless Chrome for production
  - Full browser for debugging
  
- **Agent Framework**: 
  - LangChain for orchestration
  - Custom state machines for workflows
  - Screenshot capture at each step

#### Machine Learning
- **Anomaly Detection**: 
  - Python microservice (FastAPI)
  - Scikit-learn for statistical models
  - TensorFlow for deep learning models
  
- **Matching Engine**:
  - Fuzzy matching (Levenshtein distance)
  - ML-based confidence scoring
  - Custom rule engine

### Payment Infrastructure

#### Payment Providers
- **ACH/RTP**: Astra API
- **International/Stablecoin**: 
  - Primary: Bridge
  - Backup: Circle USDC APIs
- **Check Printing**: PostGrid API

#### Banking Integration
- **Account Aggregation**: Plaid
- **Bank Feeds**: Direct bank APIs where available
- **Reconciliation**: Custom matching algorithms

### Security & Compliance

#### Authentication & Authorization
- **Auth Provider**: Auth0 / AWS Cognito
- **MFA**: TOTP, SMS, Email
- **RBAC**: Casbin for policy enforcement

#### Encryption
- **Data at Rest**: 
  - MongoDB native encryption at rest (AES-256)
  - Client-Side Field Level Encryption (CSFLE) for PII
  - Automatic encryption key rotation
- **Data in Transit**: 
  - TLS 1.3 for all MongoDB connections
  - Certificate validation and mutual TLS support
- **Secrets Management**: 
  - AWS KMS for encryption key management
  - AWS Secrets Manager / HashiCorp Vault for credentials
  - MongoDB Atlas KMS integration for CSFLE

#### Compliance
- **Audit Logging**: Immutable event store
- **Data Retention**: Configurable policies
- **PII Handling**: Field-level encryption

### MongoDB Security Configuration

#### Connection Security
```typescript
// Secure MongoDB connection with SCRAM-SHA-256 and TLS
const mongoConfig = {
  url: 'mongodb://username:password@host:port/database',
  options: {
    authSource: 'admin',
    authMechanism: 'SCRAM-SHA-256',
    tls: true,
    tlsCertificateKeyFile: '/path/to/client.pem',
    tlsCAFile: '/path/to/ca.pem',
    tlsAllowInvalidHostnames: false,
    tlsAllowInvalidCertificates: false,
    replicaSet: 'rs0',
    readPreference: 'primaryPreferred',
    w: 'majority',
    journal: true,
    retryWrites: true,
    maxPoolSize: 100,
    minPoolSize: 10,
    serverSelectionTimeoutMS: 5000,
    socketTimeoutMS: 45000
  }
};
```

#### NoSQL Injection Prevention
```typescript
// Input sanitization middleware for MongoDB queries
export const sanitizeMongoQuery = (query: any): any => {
  if (typeof query !== 'object' || query === null) {
    return query;
  }
  
  // Remove dangerous operators
  const dangerousOperators = ['$where', '$regex', '$options', '$expr'];
  
  Object.keys(query).forEach(key => {
    if (dangerousOperators.includes(key)) {
      delete query[key];
    } else if (typeof query[key] === 'object') {
      query[key] = sanitizeMongoQuery(query[key]);
    }
  });
  
  return query;
};

// Parameterized queries using MongoDB driver
export const findVendorSecure = async (vendorId: string) => {
  // Always use parameterized queries, never string concatenation
  const sanitizedId = new ObjectId(vendorId); // Validates ObjectId format
  return await vendorsCollection.findOne({ _id: sanitizedId });
};

// Schema validation at database level
const vendorSchema = {
  $jsonSchema: {
    bsonType: "object",
    required: ["legalName", "ein", "status"],
    properties: {
      legalName: {
        bsonType: "string",
        maxLength: 255,
        pattern: "^[a-zA-Z0-9\\s\\-\\.,'&]+$"
      },
      ein: {
        bsonType: "string",
        pattern: "^[0-9]{2}-[0-9]{7}$"
      },
      status: {
        enum: ["active", "inactive", "pending", "blocked"]
      }
    }
  }
};
```

#### Field-Level Encryption (CSFLE)
```typescript
// Client-Side Field Level Encryption configuration
const encryptionConfig = {
  keyVaultNamespace: 'encryption.__keyVault',
  kmsProviders: {
    aws: {
      accessKeyId: process.env.AWS_ACCESS_KEY_ID,
      secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
      sessionToken: process.env.AWS_SESSION_TOKEN
    }
  },
  schemaMap: {
    'payve.vendors': {
      bsonType: 'object',
      encryptMetadata: {
        keyId: [new Binary(Buffer.from(dataKeyId, 'base64'), 4)]
      },
      properties: {
        // Encrypt sensitive fields
        ein: {
          encrypt: {
            bsonType: 'string',
            algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic'
          }
        },
        bankAccounts: {
          encrypt: {
            bsonType: 'object',
            algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Random'
          }
        },
        taxId: {
          encrypt: {
            bsonType: 'string',
            algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic'
          }
        }
      }
    }
  }
};
```

#### Role-Based Access Control (RBAC)
```javascript
// MongoDB RBAC configuration
db.createRole({
  role: "apReadOnly",
  privileges: [
    {
      resource: { db: "payve", collection: "vendors" },
      actions: ["find", "count"]
    },
    {
      resource: { db: "payve", collection: "bills" },
      actions: ["find", "count"]
    }
  ],
  roles: []
});

db.createRole({
  role: "apProcessor",
  privileges: [
    {
      resource: { db: "payve", collection: "" },
      actions: ["find", "insert", "update", "remove"]
    }
  ],
  roles: ["apReadOnly"]
});

db.createRole({
  role: "apAdmin",
  privileges: [
    {
      resource: { db: "payve", collection: "" },
      actions: ["find", "insert", "update", "remove", "createIndex", "dropIndex"]
    }
  ],
  roles: ["apProcessor", "dbAdmin"]
});

// User creation with specific roles
db.createUser({
  user: "ap_service",
  pwd: passwordPrompt(),
  roles: [
    { role: "apProcessor", db: "payve" }
  ],
  mechanisms: ["SCRAM-SHA-256"]
});
```

#### Audit Configuration
```javascript
// MongoDB audit configuration (Enterprise)
{
  "auditLog": {
    "destination": "file",
    "format": "JSON",
    "path": "/var/log/mongodb/audit.json",
    "filter": {
      "$or": [
        { "atype": "authenticate" },
        { "atype": "createCollection" },
        { "atype": "dropCollection" },
        { "atype": "createDatabase" },
        { "atype": "dropDatabase" },
        { "atype": "createUser" },
        { "atype": "dropUser" },
        { "atype": "updateUser" },
        { "atype": "grantRolesToUser" },
        { "atype": "revokeRolesFromUser" }
      ]
    }
  }
}
```

#### Security Best Practices
1. **Network Security**
   - Bind MongoDB to specific IP addresses only
   - Use VPC peering or PrivateLink for cloud deployments
   - Implement IP whitelisting
   - Never expose MongoDB directly to the internet

2. **Authentication**
   - Always enable authentication
   - Use SCRAM-SHA-256 as minimum
   - Implement x.509 certificates for service accounts
   - Rotate credentials regularly

3. **Data Protection**
   - Enable encryption at rest
   - Use CSFLE for PII and sensitive data
   - Implement backup encryption
   - Regular security audits

4. **Monitoring**
   - Enable audit logging
   - Monitor failed authentication attempts
   - Track privilege escalations
   - Alert on unusual query patterns

### Frontend Stack

#### Web Application
- **Build Tool**: Vite 5
- **UI Library**: React 18
- **Component Library**: Shadcn/ui + Tailwind CSS
- **State Management**: Zustand + React Query
- **Real-time**: Socket.io client
- **Routing**: React Router v6

### Backend Stack

#### API Server
- **Framework**: Fastify 4
- **Runtime**: Node.js 20 LTS
- **Validation**: Ajv + Fastify Type Provider
- **WebSockets**: Socket.io with Fastify adapter
- **Process Manager**: PM2 for production

#### Monitoring & Observability

- **APM**: DataDog / New Relic
- **Logging**: Winston + ELK Stack
- **Metrics**: Prometheus + Grafana
- **Error Tracking**: Sentry
- **Distributed Tracing**: OpenTelemetry

---

## Agentic Workflow Architecture

### Browser Automation Agent (Famous ERP)

```typescript
// Example Browser Agent Implementation
class FamousERPAgent {
  private browser: Browser;
  private page: Page;
  private context: BrowserContext;
  
  async initialize(credentials: Credentials): Promise<void> {
    this.browser = await playwright.chromium.launch({
      headless: process.env.NODE_ENV === 'production',
      args: ['--disable-dev-shm-usage'],
    });
    
    this.context = await this.browser.newContext({
      viewport: { width: 1920, height: 1080 },
      userAgent: 'Mozilla/5.0...',
    });
    
    this.page = await this.context.newPage();
    
    // Setup request interception for API monitoring
    await this.page.route('**/*', (route) => {
      this.logNetworkActivity(route.request());
      route.continue();
    });
  }
  
  async login(credentials: Credentials): Promise<void> {
    await this.page.goto('https://famous-erp.com/login');
    
    // Take screenshot for audit
    await this.screenshot('login-page');
    
    await this.page.fill('#username', credentials.username);
    await this.page.fill('#password', credentials.password);
    
    await Promise.all([
      this.page.waitForNavigation(),
      this.page.click('#login-button'),
    ]);
    
    await this.screenshot('after-login');
  }
  
  async navigateToAPModule(): Promise<void> {
    await this.page.click('nav >> text="Accounts Payable"');
    await this.page.waitForSelector('.ap-dashboard');
    await this.screenshot('ap-module');
  }
  
  async searchBill(billNumber: string): Promise<BillDetails | null> {
    await this.page.fill('#bill-search', billNumber);
    await this.page.click('#search-button');
    
    await this.page.waitForTimeout(2000); // Wait for results
    
    const results = await this.page.$$('.bill-result-row');
    
    if (results.length === 0) {
      return null;
    }
    
    // Extract bill details
    const billDetails = await this.page.evaluate(() => {
      const row = document.querySelector('.bill-result-row');
      return {
        billNumber: row?.querySelector('.bill-number')?.textContent,
        vendor: row?.querySelector('.vendor-name')?.textContent,
        amount: parseFloat(row?.querySelector('.amount')?.textContent?.replace(/[^0-9.-]+/g, '') || '0'),
        status: row?.querySelector('.status')?.textContent,
        dueDate: row?.querySelector('.due-date')?.textContent,
      };
    });
    
    await this.screenshot('bill-search-results');
    
    return billDetails as BillDetails;
  }
  
  async createBill(billData: CreateBillData): Promise<string> {
    await this.page.click('#create-bill-button');
    await this.page.waitForSelector('.bill-form');
    
    // Fill form fields
    await this.page.fill('#vendor-select', billData.vendorName);
    await this.page.fill('#bill-number', billData.billNumber);
    await this.page.fill('#amount', billData.amount.toString());
    await this.page.fill('#due-date', billData.dueDate);
    
    // Add line items
    for (const item of billData.lineItems) {
      await this.page.click('#add-line-item');
      const lastRow = await this.page.$('.line-item-row:last-child');
      
      await lastRow?.$eval('.description', (el, value) => {
        (el as HTMLInputElement).value = value;
      }, item.description);
      
      await lastRow?.$eval('.quantity', (el, value) => {
        (el as HTMLInputElement).value = value;
      }, item.quantity.toString());
      
      await lastRow?.$eval('.unit-price', (el, value) => {
        (el as HTMLInputElement).value = value;
      }, item.unitPrice.toString());
    }
    
    await this.screenshot('bill-form-filled');
    
    // Submit form
    await Promise.all([
      this.page.waitForNavigation(),
      this.page.click('#save-bill-button'),
    ]);
    
    // Extract created bill ID
    const billId = await this.page.evaluate(() => {
      return document.querySelector('.bill-id')?.textContent || '';
    });
    
    await this.screenshot('bill-created');
    
    return billId;
  }
  
  async applyPayment(billId: string, paymentData: PaymentData): Promise<void> {
    await this.navigateToBill(billId);
    
    await this.page.click('#apply-payment-button');
    await this.page.waitForSelector('.payment-modal');
    
    await this.page.fill('#payment-amount', paymentData.amount.toString());
    await this.page.fill('#payment-date', paymentData.date);
    await this.page.fill('#payment-reference', paymentData.reference);
    
    await this.screenshot('payment-form');
    
    await Promise.all([
      this.page.waitForSelector('.payment-success'),
      this.page.click('#confirm-payment'),
    ]);
    
    await this.screenshot('payment-applied');
  }
  
  private async screenshot(name: string): Promise<string> {
    const timestamp = Date.now();
    const filename = `${name}-${timestamp}.png`;
    const buffer = await this.page.screenshot();
    
    // Upload to S3
    const url = await this.uploadToS3(filename, buffer);
    
    // Log for audit
    await this.logAuditEvent({
      action: 'screenshot',
      filename,
      url,
      timestamp: new Date(),
    });
    
    return url;
  }
  
  async cleanup(): Promise<void> {
    await this.context.close();
    await this.browser.close();
  }
}
```

### Document Processing Agent

```typescript
class DocumentProcessingAgent {
  private ocrService: OCRService;
  private llmService: LLMService;
  private mlService: MLService;
  
  async processInvoice(documentUrl: string): Promise<ProcessedInvoice> {
    // Step 1: Download document
    const document = await this.downloadDocument(documentUrl);
    
    // Step 2: OCR Processing
    const ocrResult = await this.ocrService.extract(document);
    
    // Step 3: LLM Enhancement for complex extraction
    const llmEnhanced = await this.llmService.enhanceExtraction({
      ocrText: ocrResult.text,
      documentType: 'invoice',
      extractFields: [
        'vendor_name',
        'invoice_number',
        'invoice_date',
        'due_date',
        'total_amount',
        'line_items',
        'po_number',
      ],
    });
    
    // Step 4: Line Item Extraction
    const lineItems = await this.extractLineItems(ocrResult, llmEnhanced);
    
    // Step 5: Validate and Clean Data
    const cleanedData = this.validateAndClean({
      ...llmEnhanced,
      lineItems,
    });
    
    // Step 6: Duplicate Detection
    const isDuplicate = await this.checkDuplicate(cleanedData);
    
    // Step 7: Confidence Scoring
    const confidence = await this.mlService.calculateConfidence(cleanedData);
    
    return {
      ...cleanedData,
      isDuplicate,
      confidence,
      ocrConfidence: ocrResult.confidence,
      processingMetadata: {
        processedAt: new Date(),
        processingTimeMs: Date.now() - startTime,
        ocrProvider: 'textract',
        llmModel: 'gpt-4',
      },
    };
  }
  
  private async extractLineItems(
    ocrResult: OCRResult,
    llmEnhanced: any
  ): Promise<LineItem[]> {
    // Use table detection from OCR
    const tables = ocrResult.tables || [];
    
    if (tables.length > 0) {
      return this.parseTableToLineItems(tables[0]);
    }
    
    // Fallback to LLM extraction
    return llmEnhanced.line_items || [];
  }
  
  private async checkDuplicate(invoice: any): Promise<boolean> {
    // Generate hash
    const hash = crypto
      .createHash('sha256')
      .update(`${invoice.vendor_name}-${invoice.invoice_number}-${invoice.total_amount}`)
      .digest('hex');
    
    // Check database
    const existing = await this.db.bills.findOne({
      where: {
        duplicateChecksum: hash,
      },
    });
    
    return !!existing;
  }
}
```

### Communication Agent

```typescript
class VendorCommunicationAgent {
  private llmService: LLMService;
  private emailService: EmailService;
  
  async processVendorEmail(email: IncomingEmail): Promise<void> {
    // Step 1: Classify email intent
    const intent = await this.classifyIntent(email);
    
    // Step 2: Extract relevant context
    const context = await this.extractContext(email, intent);
    
    // Step 3: Generate appropriate response
    const response = await this.generateResponse(intent, context);
    
    // Step 4: Execute actions if needed
    await this.executeActions(intent, context);
    
    // Step 5: Send response
    await this.sendResponse(email.from, response);
    
    // Step 6: Log interaction
    await this.logInteraction(email, intent, response);
  }
  
  private async classifyIntent(email: IncomingEmail): Promise<EmailIntent> {
    const classification = await this.llmService.classify({
      text: email.body,
      categories: [
        'payment_inquiry',
        'invoice_submission',
        'dispute',
        'remittance_update',
        'general_inquiry',
      ],
    });
    
    return {
      category: classification.category,
      confidence: classification.confidence,
      urgency: classification.urgency,
    };
  }
  
  private async generateResponse(
    intent: EmailIntent,
    context: any
  ): Promise<EmailResponse> {
    const template = this.getTemplate(intent.category);
    
    const response = await this.llmService.generateResponse({
      template,
      context,
      tone: 'professional',
      maxLength: 500,
    });
    
    // Add relevant attachments
    const attachments = await this.gatherAttachments(intent, context);
    
    return {
      subject: `Re: ${context.originalSubject}`,
      body: response.text,
      attachments,
    };
  }
  
  private async executeActions(
    intent: EmailIntent,
    context: any
  ): Promise<void> {
    switch (intent.category) {
      case 'payment_inquiry':
        await this.updatePaymentTracking(context.paymentId, {
          vendorInquired: true,
          inquiryDate: new Date(),
        });
        break;
        
      case 'dispute':
        await this.createDispute({
          vendorId: context.vendorId,
          billId: context.billId,
          reason: context.disputeReason,
        });
        break;
        
      case 'remittance_update':
        await this.updateVendorRemittance(context.vendorId, context.newRemittance);
        break;
    }
  }
}
```

---

## API Architecture

### RESTful API Structure

```typescript
// API Routes Structure
const apiRoutes = {
  // Authentication
  'POST /auth/login': 'AuthController.login',
  'POST /auth/logout': 'AuthController.logout',
  'POST /auth/refresh': 'AuthController.refresh',
  
  // Vendors
  'GET /vendors': 'VendorController.list',
  'GET /vendors/:id': 'VendorController.get',
  'POST /vendors': 'VendorController.create',
  'PUT /vendors/:id': 'VendorController.update',
  'DELETE /vendors/:id': 'VendorController.delete',
  'POST /vendors/:id/verify-banking': 'VendorController.verifyBanking',
  
  // Purchase Orders
  'GET /purchase-orders': 'PurchaseOrderController.list',
  'GET /purchase-orders/:id': 'PurchaseOrderController.get',
  'POST /purchase-orders': 'PurchaseOrderController.create',
  'PUT /purchase-orders/:id': 'PurchaseOrderController.update',
  'POST /purchase-orders/:id/approve': 'PurchaseOrderController.approve',
  'POST /purchase-orders/:id/match-bill': 'PurchaseOrderController.matchBill',
  
  // Bills/Invoices
  'GET /bills': 'BillController.list',
  'GET /bills/:id': 'BillController.get',
  'POST /bills': 'BillController.create',
  'POST /bills/ingest-email': 'BillController.ingestEmail',
  'POST /bills/:id/match': 'BillController.match',
  'POST /bills/:id/approve': 'BillController.approve',
  
  // Payments
  'GET /payments': 'PaymentController.list',
  'GET /payments/:id': 'PaymentController.get',
  'POST /payments': 'PaymentController.create',
  'POST /payments/:id/schedule': 'PaymentController.schedule',
  'POST /payments/:id/execute': 'PaymentController.execute',
  'POST /payments/:id/cancel': 'PaymentController.cancel',
  'GET /payments/scheduled': 'PaymentController.getScheduled',
  
  // Reconciliation
  'GET /reconciliations': 'ReconciliationController.list',
  'GET /reconciliations/:id': 'ReconciliationController.get',
  'POST /reconciliations/run': 'ReconciliationController.runBatch',
  'POST /reconciliations/:id/resolve': 'ReconciliationController.resolve',
  
  // Policies
  'GET /policies': 'PolicyController.list',
  'GET /policies/:id': 'PolicyController.get',
  'POST /policies': 'PolicyController.create',
  'PUT /policies/:id': 'PolicyController.update',
  'POST /policies/:id/evaluate': 'PolicyController.evaluate',
  
  // Audit
  'GET /audit-events': 'AuditController.list',
  'GET /audit-events/:id': 'AuditController.get',
  'GET /audit-events/export': 'AuditController.export',
  
  // Agent Executions
  'GET /agent-executions': 'AgentController.listExecutions',
  'GET /agent-executions/:id': 'AgentController.getExecution',
  'POST /agent-executions': 'AgentController.startExecution',
  'POST /agent-executions/:id/cancel': 'AgentController.cancelExecution',
  
  // Dashboard/Analytics
  'GET /dashboard/summary': 'DashboardController.getSummary',
  'GET /dashboard/actions': 'DashboardController.getRequiredActions',
  'GET /analytics/payment-trends': 'AnalyticsController.getPaymentTrends',
  'GET /analytics/reconciliation-stats': 'AnalyticsController.getReconciliationStats',
};
```

### GraphQL Schema

```graphql
type Query {
  # Vendors
  vendor(id: ID!): Vendor
  vendors(
    filter: VendorFilter
    pagination: PaginationInput
    sort: VendorSort
  ): VendorConnection!
  
  # Purchase Orders
  purchaseOrder(id: ID!): PurchaseOrder
  purchaseOrders(
    filter: PurchaseOrderFilter
    pagination: PaginationInput
  ): PurchaseOrderConnection!
  
  # Bills
  bill(id: ID!): Bill
  bills(
    filter: BillFilter
    pagination: PaginationInput
  ): BillConnection!
  
  # Payments
  payment(id: ID!): Payment
  payments(
    filter: PaymentFilter
    pagination: PaginationInput
  ): PaymentConnection!
  
  scheduledPayments(
    startDate: Date!
    endDate: Date!
    groupBy: GroupByOption
  ): ScheduledPaymentSummary!
  
  # Reconciliation
  reconciliationStatus(paymentId: ID!): ReconciliationRecord
  reconciliationQueue: [ReconciliationRecord!]!
  
  # Dashboard
  dashboardSummary: DashboardSummary!
  requiredActions: [RequiredAction!]!
}

type Mutation {
  # Vendor Operations
  createVendor(input: CreateVendorInput!): Vendor!
  updateVendor(id: ID!, input: UpdateVendorInput!): Vendor!
  verifyVendorBanking(id: ID!): VendorVerificationResult!
  
  # Bill Operations
  ingestBill(input: IngestBillInput!): Bill!
  matchBillToPO(billId: ID!, poId: ID!): MatchResult!
  approveBill(id: ID!, comment: String): Bill!
  
  # Payment Operations
  schedulePayment(input: SchedulePaymentInput!): Payment!
  executePayment(id: ID!): PaymentExecutionResult!
  cancelPayment(id: ID!, reason: String!): Payment!
  
  # Reconciliation
  runReconciliation(paymentId: ID!): ReconciliationRecord!
  resolveReconciliationException(
    id: ID!
    resolution: ResolutionInput!
  ): ReconciliationRecord!
  
  # Policy Operations
  createPolicy(input: CreatePolicyInput!): PolicyRule!
  overridePolicy(
    ruleId: ID!
    entityId: ID!
    reason: String!
  ): PolicyException!
}

type Subscription {
  # Real-time payment updates
  paymentStatusChanged(paymentId: ID!): Payment!
  
  # Reconciliation updates
  reconciliationCompleted: ReconciliationRecord!
  
  # Audit events
  auditEventCreated(category: String): AuditEvent!
  
  # Agent executions
  agentExecutionUpdated(executionId: ID!): AgentExecution!
}
```

---

## Integration Architecture

### QuickBooks Integration

```typescript
class QuickBooksIntegration {
  private client: QuickBooksClient;
  
  async syncVendor(vendor: Vendor): Promise<void> {
    const qbVendor = await this.client.vendor.create({
      DisplayName: vendor.legalName,
      CompanyName: vendor.dba || vendor.legalName,
      BillAddr: this.mapAddress(vendor.addresses.billing),
      PrimaryEmailAddr: {
        Address: vendor.contacts[0]?.email,
      },
      TaxIdentifier: vendor.taxId,
    });
    
    // Update vendor with QB ID
    await this.vendorService.update(vendor.id, {
      erpVendorId: qbVendor.Id,
    });
  }
  
  async syncBill(bill: Bill): Promise<void> {
    const qbBill = await this.client.bill.create({
      VendorRef: {
        value: bill.vendor.erpVendorId,
      },
      Line: bill.lineItems.map(item => ({
        Amount: item.totalPrice,
        DetailType: 'AccountBasedExpenseLineDetail',
        AccountBasedExpenseLineDetail: {
          AccountRef: {
            value: this.getExpenseAccount(item),
          },
        },
        Description: item.description,
      })),
      TxnDate: bill.invoiceDate,
      DueDate: bill.dueDate,
      DocNumber: bill.billNumber,
      TotalAmt: bill.amounts.total,
    });
    
    // Update bill with QB ID
    await this.billService.update(bill.id, {
      erpBillId: qbBill.Id,
    });
  }
  
  async syncPayment(payment: Payment): Promise<void> {
    const qbPayment = await this.client.billPayment.create({
      VendorRef: {
        value: payment.vendor.erpVendorId,
      },
      PayType: this.mapPaymentType(payment.method),
      Line: payment.billIds.map(billId => ({
        Amount: payment.amount / payment.billIds.length, // Pro-rate
        LinkedTxn: [{
          TxnId: this.getBillErpId(billId),
          TxnType: 'Bill',
        }],
      })),
      TotalAmt: payment.amount,
      TxnDate: payment.execution.sentAt,
    });
    
    // Update payment with QB ID
    await this.paymentService.update(payment.id, {
      erpPaymentId: qbPayment.Id,
    });
  }
}
```

### Famous ERP Integration (Browser-based)

```typescript
class FamousIntegration {
  private agentPool: FamousERPAgent[];
  
  async reconcilePayment(payment: Payment): Promise<void> {
    const agent = await this.getAvailableAgent();
    
    try {
      // Start agent execution
      const execution = await this.startExecution(payment);
      
      // Login to Famous
      await agent.login(this.credentials);
      
      // Navigate to AP module
      await agent.navigateToAPModule();
      
      // Search for bill
      const billDetails = await agent.searchBill(payment.bills[0].billNumber);
      
      if (!billDetails) {
        // Create bill if not found
        const billId = await agent.createBill({
          vendorName: payment.vendor.legalName,
          billNumber: payment.bills[0].billNumber,
          amount: payment.amount,
          dueDate: payment.bills[0].dueDate,
          lineItems: payment.bills[0].lineItems,
        });
        
        // Update our records
        await this.billService.update(payment.bills[0].id, {
          erpBillId: billId,
        });
      }
      
      // Apply payment
      await agent.applyPayment(billDetails.id, {
        amount: payment.amount,
        date: payment.execution.sentAt,
        reference: payment.paymentNumber,
      });
      
      // Mark reconciliation complete
      await this.reconciliationService.markComplete(payment.id, {
        erpPaymentId: billDetails.id,
        reconciledAt: new Date(),
        method: 'agent',
      });
      
      // Complete execution
      await this.completeExecution(execution, 'success');
      
    } catch (error) {
      await this.handleAgentError(agent, error);
      throw error;
    } finally {
      await this.releaseAgent(agent);
    }
  }
}
```

---

## Security & Compliance

### Security Architecture

```typescript
// Security Middleware Stack
class SecurityMiddleware {
  // Rate Limiting
  rateLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP',
  });
  
  // CORS Configuration
  corsOptions = {
    origin: process.env.ALLOWED_ORIGINS?.split(','),
    credentials: true,
    optionsSuccessStatus: 200,
  };
  
  // Security Headers
  securityHeaders = helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        scriptSrc: ["'self'"],
        imgSrc: ["'self'", 'data:', 'https:'],
      },
    },
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
      preload: true,
    },
  });
  
  // Input Validation for Fastify
  validateInput = (schema: any) => {
    return async (request: FastifyRequest, reply: FastifyReply) => {
      const { error } = schema.validate(request.body);
      if (error) {
        return reply.status(400).send({
          error: 'Validation failed',
          details: error.details,
        });
      }
    };
  };
  
  // NoSQL Injection Prevention for Fastify
  sanitizeQuery = async (request: FastifyRequest, reply: FastifyReply) => {
    request.query = this.sanitizeObject(request.query);
    request.params = this.sanitizeObject(request.params);
  };
  
  // XSS Prevention
  xssProtection = async (request: FastifyRequest, reply: FastifyReply) => {
    req.body = this.sanitizeObject(req.body);
    next();
  };
}
```

### Encryption Service

```typescript
class EncryptionService {
  private algorithm = 'aes-256-gcm';
  private keyDerivation = 'pbkdf2';
  
  // Field-level encryption for sensitive data
  async encryptField(data: string, context: string): Promise<EncryptedData> {
    const salt = crypto.randomBytes(32);
    const iv = crypto.randomBytes(16);
    
    const key = await this.deriveKey(
      process.env.MASTER_KEY!,
      salt,
      context
    );
    
    const cipher = crypto.createCipheriv(this.algorithm, key, iv);
    
    let encrypted = cipher.update(data, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    return {
      encrypted,
      salt: salt.toString('hex'),
      iv: iv.toString('hex'),
      authTag: authTag.toString('hex'),
      algorithm: this.algorithm,
      context,
    };
  }
  
  async decryptField(encryptedData: EncryptedData): Promise<string> {
    const key = await this.deriveKey(
      process.env.MASTER_KEY!,
      Buffer.from(encryptedData.salt, 'hex'),
      encryptedData.context
    );
    
    const decipher = crypto.createDecipheriv(
      encryptedData.algorithm,
      key,
      Buffer.from(encryptedData.iv, 'hex')
    );
    
    decipher.setAuthTag(Buffer.from(encryptedData.authTag, 'hex'));
    
    let decrypted = decipher.update(encryptedData.encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
  
  // Encrypt sensitive vendor banking data
  async encryptBankAccount(account: BankAccount): Promise<EncryptedBankAccount> {
    return {
      ...account,
      routingNumber: await this.encryptField(
        account.routingNumber,
        'bank_routing'
      ),
      accountNumber: await this.encryptField(
        account.accountNumber,
        'bank_account'
      ),
    };
  }
}
```

### Audit Service

```typescript
class AuditService {
  async logEvent(event: AuditEventInput): Promise<void> {
    const auditEvent: AuditEvent = {
      id: uuidv4(),
      organizationId: event.organizationId,
      category: event.category,
      action: event.action,
      actor: event.actor,
      event: {
        timestamp: new Date(),
        entities: event.entities,
        changes: event.changes,
        metadata: event.metadata,
      },
      attachments: event.attachments || [],
      compliance: {
        requiresReview: this.requiresReview(event),
        tags: this.generateComplianceTags(event),
        retentionPolicy: {
          retainUntil: this.calculateRetentionDate(event),
          legalHold: false,
        },
      },
      tracing: {
        traceId: event.traceId || uuidv4(),
        spanId: uuidv4(),
        parentSpanId: event.parentSpanId,
        relatedEventIds: [],
        sessionId: event.sessionId,
        requestId: event.requestId,
      },
      searchableText: this.generateSearchableText(event),
      tags: event.tags || [],
      createdAt: new Date(),
    };
    
    // Write to immutable store
    await this.writeToImmutableStore(auditEvent);
    
    // Index for search
    await this.indexForSearch(auditEvent);
    
    // Check compliance rules
    await this.checkComplianceRules(auditEvent);
    
    // Emit event for real-time monitoring
    this.eventEmitter.emit('audit.event.created', auditEvent);
  }
  
  private async writeToImmutableStore(event: AuditEvent): Promise<void> {
    // Write to append-only log
    await this.appendOnlyLog.append(event);
    
    // Create cryptographic proof
    const hash = this.calculateHash(event);
    await this.proofStore.store(event.id, hash);
    
    // Replicate to backup
    await this.replicateToBackup(event);
  }
  
  private calculateHash(event: AuditEvent): string {
    const content = JSON.stringify(event);
    return crypto
      .createHash('sha256')
      .update(content)
      .digest('hex');
  }
}
```

---

## Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)

#### Week 1-2: Infrastructure Setup
- [ ] Set up development environment
- [ ] Configure MongoDB with secure multi-tenancy
- [ ] Set up SCRAM-SHA-256 authentication
- [ ] Configure TLS/SSL certificates
- [ ] Implement role-based access control
- [ ] Set up field-level encryption keys
- [ ] Set up Redis for caching and queues
- [ ] Configure S3/MinIO for document storage
- [ ] Implement authentication with Auth0/Cognito
- [ ] Set up CI/CD pipeline

#### Week 3-4: Core Data Models
- [ ] Implement Organization/Tenant model
- [ ] Implement Vendor management
- [ ] Implement Purchase Order model
- [ ] Implement Bill/Invoice model
- [ ] Set up database migrations
- [ ] Create seed data scripts

### Phase 2: Document Processing (Weeks 5-8)

#### Week 5-6: OCR Integration
- [ ] Integrate AWS Textract/Azure Form Recognizer
- [ ] Implement document upload pipeline
- [ ] Build extraction templates
- [ ] Implement duplicate detection

#### Week 7-8: Matching Engine
- [ ] Build PO-to-Bill matching logic
- [ ] Implement tolerance rules
- [ ] Create exception handling
- [ ] Build manual review interface

### Phase 3: Payment Engine (Weeks 9-12)

#### Week 9-10: Payment Provider Integration
- [ ] Integrate Astra for ACH/RTP
- [ ] Integrate Bridge for Stablecoin
- [ ] Implement payment scheduling
- [ ] Build payment execution engine

#### Week 11-12: Reconciliation
- [ ] Build bank feed integration (Plaid)
- [ ] Implement reconciliation matching
- [ ] Create exception queue
- [ ] Build reconciliation UI

### Phase 4: Browser Automation (Weeks 13-16)

#### Week 13-14: Famous ERP Agent
- [ ] Set up Playwright infrastructure
- [ ] Build login and navigation flows
- [ ] Implement bill search/creation
- [ ] Build payment application flow

#### Week 15-16: Agent Orchestration
- [ ] Implement agent pool management
- [ ] Build execution monitoring
- [ ] Create screenshot capture
- [ ] Implement error recovery

### Phase 5: Policy & Compliance (Weeks 17-20)

#### Week 17-18: Policy Engine
- [ ] Implement rule definition system
- [ ] Build rule evaluation engine
- [ ] Create approval workflows
- [ ] Implement exception handling

#### Week 19-20: Audit & Compliance
- [ ] Build immutable audit log
- [ ] Implement compliance checks
- [ ] Create audit export functionality
- [ ] Build compliance reports

### Phase 6: UI & Integration (Weeks 21-24)

#### Week 21-22: Frontend Development
- [ ] Build command center dashboard
- [ ] Create payment scheduling UI
- [ ] Implement reconciliation workbench
- [ ] Build vendor management interface

#### Week 23-24: Testing & Deployment
- [ ] Comprehensive integration testing
- [ ] Performance optimization
- [ ] Security audit
- [ ] Production deployment

---

## Sample Test Data Implementation

### Test Vendor with Full Banking Details

```typescript
const testVendor: Vendor = {
  id: "v_01J7XQWERTY123456",
  organizationId: "org_01J7XQABCDEF",
  erpVendorId: "V-001",
  
  legalName: "Global Suppliers Inc.",
  dba: "GSI Supply Chain",
  taxId: "XX-XXXXXXX", // Encrypted
  
  status: "active",
  standing: "good",
  riskScore: 85,
  
  paymentPreferences: {
    defaultMethod: "ACH",
    paymentTerms: "Net30",
    earlyPaymentDiscount: {
      percentage: 2,
      withinDays: 10,
    },
  },
  
  bankAccounts: {
    domestic: [
      {
        id: "ba_01J7XQBANK001",
        type: "ACH",
        routingNumber: "ENCRYPTED_1234567890",
        accountNumber: "ENCRYPTED_9876543210",
        accountType: "checking",
        verificationStatus: "verified",
        verifiedAt: new Date("2024-01-15T09:30:00Z"),
        isDefault: true,
      },
    ],
    international: [],
  },
  
  contacts: [
    {
      id: "c_01J7XQCONTACT1",
      type: "primary",
      name: "John Smith",
      email: "accounting@globalsuppliers.com",
      phone: "+1-555-0100",
      isActive: true,
    },
    {
      id: "c_01J7XQCONTACT2",
      type: "remittance",
      name: "Jane Doe",
      email: "payments@globalsuppliers.com",
      phone: "+1-555-0101",
      isActive: true,
    },
  ],
  
  addresses: {
    remittance: {
      line1: "123 Main St",
      line2: "Suite 100",
      city: "Anytown",
      state: "CA",
      postalCode: "90210",
      country: "USA",
    },
    physical: {
      line1: "123 Main St",
      city: "Anytown",
      state: "CA",
      postalCode: "90210",
      country: "USA",
    },
  },
  
  kyb: {
    status: "approved",
    completedAt: new Date("2024-01-10T14:00:00Z"),
    documents: [
      {
        type: "w9",
        url: "s3://documents/vendors/v_01J7XQWERTY123456/w9.pdf",
        uploadedAt: new Date("2024-01-10T13:45:00Z"),
      },
    ],
  },
  
  tags: ["preferred", "high-volume", "produce"],
  customFields: {
    accountManager: "Sarah Johnson",
    annualVolume: 2500000,
  },
  
  createdAt: new Date("2024-01-15T09:30:00Z"),
  updatedAt: new Date("2025-08-20T11:45:00Z"),
  createdBy: "user_01J7XQUSER001",
  updatedBy: "user_01J7XQUSER002",
};
```

### Test Purchase Order with Line Items

```typescript
const testPurchaseOrder: PurchaseOrder = {
  id: "po_01J7XQPO123456",
  organizationId: "org_01J7XQABCDEF",
  poNumber: "PO-2025-001234",
  erpPoId: "ERP-PO-001234",
  
  vendorId: "v_01J7XQWERTY123456",
  vendorName: "Global Suppliers Inc.",
  
  orderDate: new Date("2025-09-01T10:00:00Z"),
  expectedDeliveryDate: new Date("2025-09-15T10:00:00Z"),
  
  currency: "USD",
  amounts: {
    subtotal: 10000.00,
    tax: 875.00,
    shipping: 250.00,
    otherCharges: 0,
    total: 11125.00,
  },
  
  lineItems: [
    {
      id: "li_01J7XQLINE001",
      lineNumber: 1,
      sku: "PROD-001",
      description: "Organic Apples - Gala",
      category: "Produce",
      quantity: 500,
      unitOfMeasure: "LB",
      unitPrice: 2.50,
      totalPrice: 1250.00,
      taxRate: 0.0875,
      taxAmount: 109.38,
      matchingTolerance: {
        quantityPercent: 5,
        pricePercent: 2,
      },
    },
    {
      id: "li_01J7XQLINE002",
      lineNumber: 2,
      sku: "PROD-002",
      description: "Organic Bananas",
      category: "Produce",
      quantity: 1000,
      unitOfMeasure: "LB",
      unitPrice: 1.75,
      totalPrice: 1750.00,
      taxRate: 0.0875,
      taxAmount: 153.13,
    },
  ],
  
  status: {
    current: "approved",
    history: [
      {
        status: "draft",
        changedAt: new Date("2025-09-01T10:00:00Z"),
        changedBy: "user_01J7XQUSER001",
      },
      {
        status: "submitted",
        changedAt: new Date("2025-09-01T10:30:00Z"),
        changedBy: "user_01J7XQUSER001",
      },
      {
        status: "approved",
        changedAt: new Date("2025-09-01T11:00:00Z"),
        changedBy: "user_01J7XQUSER002",
      },
    ],
  },
  
  paymentTerms: {
    dueDate: new Date("2025-10-15T00:00:00Z"),
    terms: "Net 30",
    discountTerms: "2/10 Net 30",
  },
  
  documents: [
    {
      id: "doc_01J7XQDOC001",
      type: "po",
      name: "PO-2025-001234.pdf",
      url: "s3://documents/pos/po_01J7XQPO123456/original.pdf",
      mimeType: "application/pdf",
      size: 125000,
      uploadedAt: new Date("2025-09-01T10:00:00Z"),
      uploadedBy: "user_01J7XQUSER001",
    },
  ],
  
  approvals: {
    required: true,
    threshold: 10000,
    chain: [
      {
        userId: "user_01J7XQUSER002",
        role: "Manager",
        approvedAt: new Date("2025-09-01T11:00:00Z"),
        status: "approved",
        comments: "Approved for standard processing",
      },
    ],
  },
  
  linkedBillIds: [],
  linkedPaymentIds: [],
  
  department: "Procurement",
  costCenter: "CC-100",
  project: "Q3-Supply-2025",
  tags: ["urgent", "recurring"],
  customFields: {},
  
  createdAt: new Date("2025-09-01T10:00:00Z"),
  updatedAt: new Date("2025-09-01T11:00:00Z"),
  createdBy: "user_01J7XQUSER001",
  updatedBy: "user_01J7XQUSER002",
};
```

### Test Bill with Matching Results

```typescript
const testBill: Bill = {
  id: "bill_01J7XQBILL123",
  organizationId: "org_01J7XQABCDEF",
  billNumber: "INV-2025-5678",
  erpBillId: "QB-BILL-5678",
  
  vendorId: "v_01J7XQWERTY123456",
  vendorName: "Global Suppliers Inc.",
  
  purchaseOrderIds: ["po_01J7XQPO123456"],
  
  invoiceDate: new Date("2025-09-10T00:00:00Z"),
  receivedDate: new Date("2025-09-11T14:30:00Z"),
  dueDate: new Date("2025-10-10T00:00:00Z"),
  
  currency: "USD",
  amounts: {
    subtotal: 10050.00,
    tax: 879.38,
    shipping: 250.00,
    otherCharges: 25.00,
    discount: 0,
    total: 11204.38,
  },
  
  lineItems: [
    {
      id: "bli_01J7XQBLINE001",
      lineNumber: 1,
      description: "Organic Apples - Gala",
      quantity: 495,
      unitPrice: 2.55,
      totalPrice: 1262.25,
      matchedPoLineItemId: "li_01J7XQLINE001",
      matchingStatus: "matched",
      matchingDiscrepancies: {
        quantityVariance: -5,
        priceVariance: 0.05,
        description: "Within tolerance",
      },
    },
  ],
  
  status: {
    current: "matched",
    history: [
      {
        status: "received",
        timestamp: new Date("2025-09-11T14:30:00Z"),
        agentId: "agent_email_processor",
      },
      {
        status: "processing",
        timestamp: new Date("2025-09-11T14:31:00Z"),
        agentId: "agent_ocr",
      },
      {
        status: "matched",
        timestamp: new Date("2025-09-11T14:35:00Z"),
        agentId: "agent_matcher",
      },
    ],
  },
  
  matching: {
    status: "auto_matched",
    method: "auto",
    confidence: 95,
    performedAt: new Date("2025-09-11T14:35:00Z"),
    performedBy: "agent_matcher",
    poMatch: {
      matched: true,
      poId: "po_01J7XQPO123456",
      variances: {
        totalAmount: 79.38,
        lineItemCount: 0,
      },
    },
  },
  
  policyEvaluation: {
    evaluatedAt: new Date("2025-09-11T14:36:00Z"),
    violations: [],
    approvalRequired: false,
  },
  
  documents: [
    {
      id: "bdoc_01J7XQBDOC001",
      type: "invoice",
      name: "INV-2025-5678.pdf",
      url: "s3://documents/bills/bill_01J7XQBILL123/invoice.pdf",
      mimeType: "application/pdf",
      hash: "sha256:abcdef1234567890",
      ocrProcessed: true,
      ocrConfidence: 98.5,
      uploadedAt: new Date("2025-09-11T14:30:00Z"),
    },
  ],
  
  duplicateCheck: {
    performedAt: new Date("2025-09-11T14:31:00Z"),
    isDuplicate: false,
    checksum: "sha256:1234567890abcdef",
  },
  
  payment: {
    scheduledDate: new Date("2025-10-08T00:00:00Z"),
    method: "ACH",
  },
  
  source: "email",
  emailMessageId: "msg_01J7XQEMAIL123",
  tags: ["auto-processed"],
  customFields: {},
  
  createdAt: new Date("2025-09-11T14:30:00Z"),
  updatedAt: new Date("2025-09-11T14:36:00Z"),
  processedBy: "agent_processor",
};
```

---

## Conclusion

This technical architecture document provides a comprehensive foundation for building the Agentic Payments platform. The system is designed with:

1. **Scalability**: Microservices architecture with clear separation of concerns
2. **Reliability**: Robust error handling, retry mechanisms, and fallback strategies
3. **Security**: End-to-end encryption, comprehensive audit trail, and compliance focus
4. **Intelligence**: AI-powered document processing and browser automation for non-API systems
5. **Flexibility**: Support for multiple payment rails and integration methods

The implementation roadmap provides a clear path from MVP to full production system, with each phase building upon the previous one. The detailed data models ensure proper relationships and tracking throughout the payment lifecycle.

Key success factors:
- Achieving 95% auto-reconciliation rate
- Sub-2 hour data freshness
- Seamless browser automation for Famous ERP
- Comprehensive audit trail for compliance
- Intuitive user interface for AP teams

This architecture supports both the immediate MVP requirements and future expansion into advanced features like QuickPay optimization, international payments, and complex supply chain workflows.
