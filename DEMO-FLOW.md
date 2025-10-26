# Loan Application Demo Flow

This document contains a Mermaid diagram showing the complete flow of the loan application demo process, including all agents, API endpoints, success criteria, and failure paths.

## High-Level Workflow Overview

```mermaid
flowchart LR
    Start([User Submits<br/>Loan Application]) --> Step0[Step 0: Initial<br/>Submission]
    Step0 --> Step1[Step 1: Verification &<br/>Credit Check]
    Step1 --> Step2[Step 2: Collateral &<br/>Lien Check]
    Step2 --> Step3[Step 3: Risk<br/>Decision Gate]
    Step3 --> Step4[Step 4: Final<br/>Registration]
    Step4 --> Step5[Step 5: Final<br/>Outcome]
    Step5 --> End([APPROVED])
    
    %% Styling
    classDef process fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000000
    classDef endpoint fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000000
    
    class Step0,Step1,Step2,Step3,Step4,Step5 process
    class Start,End endpoint
```

## Step 0: Initial Submission

```mermaid
flowchart LR
    Start([User Submits<br/>Loan Application]) --> Orchestrator[Orchestrator<br/>Begins Workflow]
    Orchestrator --> LOS_API[Registration Agent<br/>POST /api/v1/loan-applications]
    
    LOS_API --> LOS_Success{LOS Creation<br/>Successful?}
    LOS_Success -->|✅ Yes| Success[Receives Application ID<br/>Status: SUBMITTED]
    LOS_Success -->|❌ No| HITL_LOS[HITL Tool:<br/>LOS Creation Failed]
    
    Success --> NextStep[Continue to<br/>Step 1]
    
    %% Styling
    classDef agent fill:#e8eaf6,stroke:#3f51b5,stroke-width:2px,color:#000000
    classDef api fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px,color:#000000
    classDef decision fill:#fff8e1,stroke:#ff8f00,stroke-width:2px,color:#000000
    classDef success fill:#e8f5e8,stroke:#4caf50,stroke-width:2px,color:#000000
    classDef failure fill:#ffebee,stroke:#f44336,stroke-width:2px,color:#000000
    classDef process fill:#e3f2fd,stroke:#2196f3,stroke-width:2px,color:#000000
    
    class Orchestrator agent
    class LOS_API api
    class LOS_Success decision
    class Success,NextStep success
    class HITL_LOS failure
```

## Step 1: Verification & Credit Check

```mermaid
flowchart LR
    PrevStep[From Step 0] --> CVA[Customer<br/>Verification Agent]
    CVA --> Credit_API[Credit Bureau Service<br/>POST /api/v1/credit/check]
    
    Credit_API --> Credit_Success{Credit Check<br/>Successful?}
    Credit_Success -->|✅ Yes| Success[Returns creditScore<br/>and annualIncome<br/>Documents Verified]
    Credit_Success -->|❌ No| HITL_Credit[HITL Tool:<br/>Low/Invalid/Missing<br/>Credit Score]
    
    Success --> NextStep[Continue to<br/>Step 2]
    
    %% Styling
    classDef agent fill:#e8eaf6,stroke:#3f51b5,stroke-width:2px,color:#000000
    classDef api fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px,color:#000000
    classDef decision fill:#fff8e1,stroke:#ff8f00,stroke-width:2px,color:#000000
    classDef success fill:#e8f5e8,stroke:#4caf50,stroke-width:2px,color:#000000
    classDef failure fill:#ffebee,stroke:#f44336,stroke-width:2px,color:#000000
    classDef process fill:#e3f2fd,stroke:#2196f3,stroke-width:2px,color:#000000
    
    class CVA agent
    class Credit_API api
    class Credit_Success decision
    class Success,NextStep,PrevStep success
    class HITL_Credit failure
```

## Step 2: Collateral & Lien Check

```mermaid
flowchart LR
    PrevStep[From Step 1] --> EA[Encumbrance Agent]
    EA --> Collateral_API[Collateral Service<br/>POST /api/v1/auto-loan/<br/>valuation/vehicle/appraise]
    
    Collateral_API --> Lien_API[GET /api/v1/encumbrances/<br/>collateral/ID/active]
    
    Lien_API --> Collateral_Success{High Market Value<br/>AND No Active Liens?}
    Collateral_Success -->|✅ Yes| Success[High marketValue<br/>Empty encumbrances list]
    Collateral_Success -->|❌ No| HITL_Collateral[HITL Tool:<br/>Low Market Value<br/>OR Active Lien Found]
    
    Success --> NextStep[Continue to<br/>Step 3]
    
    %% Styling
    classDef agent fill:#e8eaf6,stroke:#3f51b5,stroke-width:2px,color:#000000
    classDef api fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px,color:#000000
    classDef decision fill:#fff8e1,stroke:#ff8f00,stroke-width:2px,color:#000000
    classDef success fill:#e8f5e8,stroke:#4caf50,stroke-width:2px,color:#000000
    classDef failure fill:#ffebee,stroke:#f44336,stroke-width:2px,color:#000000
    classDef process fill:#e3f2fd,stroke:#2196f3,stroke-width:2px,color:#000000
    
    class EA agent
    class Collateral_API,Lien_API api
    class Collateral_Success decision
    class Success,NextStep,PrevStep success
    class HITL_Collateral failure
```

## Step 3: Risk Decision Gate (CRITICAL)

```mermaid
flowchart LR
    PrevStep[From Step 2] --> UA[Underwriting Agent]
    UA --> Risk_API[Internal Risk Engine<br/>POST /api/v1/risk-assessment/<br/>evaluate]
    
    Risk_API --> Risk_Success{Approval<br/>Recommendation<br/>= True?}
    Risk_Success -->|✅ Yes| Success[Returns approvalRecommendation: true<br/>and recommendedInterestRate]
    Risk_Success -->|❌ No| HITL_Risk[HITL Tool:<br/>DTI/LTV Ratio Failed<br/>approvalRecommendation: false]
    
    Success --> NextStep[Continue to<br/>Step 4]
    
    %% Styling
    classDef agent fill:#e8eaf6,stroke:#3f51b5,stroke-width:2px,color:#000000
    classDef api fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px,color:#000000
    classDef decision fill:#fff8e1,stroke:#ff8f00,stroke-width:2px,color:#000000
    classDef success fill:#e8f5e8,stroke:#4caf50,stroke-width:2px,color:#000000
    classDef failure fill:#ffebee,stroke:#f44336,stroke-width:2px,color:#000000
    classDef process fill:#e3f2fd,stroke:#2196f3,stroke-width:2px,color:#000000
    classDef critical fill:#fff3e0,stroke:#ff9800,stroke-width:3px,color:#000000
    
    class UA agent
    class Risk_API api
    class Risk_Success decision
    class Success,NextStep,PrevStep success
    class HITL_Risk failure
    class Risk_API critical
```

## Step 4: Final Registration

```mermaid
flowchart LR
    PrevStep[From Step 3] --> RA[Registration Agent]
    RA --> Approval_API[Loan Service<br/>PUT /api/v1/loan-applications/<br/>ID/approve]
    
    Approval_API --> Final_Success{Final LOS<br/>Update Successful?}
    Final_Success -->|✅ Yes| Success[Application Status:<br/>APPROVED]
    Final_Success -->|❌ No| HITL_Final[HITL Tool:<br/>Final LOS Update/<br/>Approval Failed]
    
    Success --> NextStep[Continue to<br/>Step 5]
    
    %% Styling
    classDef agent fill:#e8eaf6,stroke:#3f51b5,stroke-width:2px,color:#000000
    classDef api fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px,color:#000000
    classDef decision fill:#fff8e1,stroke:#ff8f00,stroke-width:2px,color:#000000
    classDef success fill:#e8f5e8,stroke:#4caf50,stroke-width:2px,color:#000000
    classDef failure fill:#ffebee,stroke:#f44336,stroke-width:2px,color:#000000
    classDef process fill:#e3f2fd,stroke:#2196f3,stroke-width:2px,color:#000000
    
    class RA agent
    class Approval_API api
    class Final_Success decision
    class Success,NextStep,PrevStep success
    class HITL_Final failure
```

## Step 5: Final Outcome

```mermaid
flowchart LR
    PrevStep[From Step 4] --> Orchestrator_End[Orchestrator<br/>Concludes Workflow]
    Orchestrator_End --> End([✅ APPROVED with<br/>Application ID<br/>and Interest Rate])
    
    %% Styling
    classDef agent fill:#e8eaf6,stroke:#3f51b5,stroke-width:2px,color:#000000
    classDef success fill:#e8f5e8,stroke:#4caf50,stroke-width:2px,color:#000000
    classDef endpoint fill:#e1f5fe,stroke:#0288d1,stroke-width:3px,color:#000000
    
    class Orchestrator_End agent
    class PrevStep success
    class End endpoint
```

## Workflow Summary

### Agents and Their Roles:
- **Orchestrator**: Manages the overall workflow
- **Registration Agent (LOS)**: Handles loan application registration and final approval
- **Customer Verification Agent (CVA)**: Performs credit checks and document verification
- **Encumbrance Agent (EA)**: Handles collateral valuation and lien checks
- **Underwriting Agent (UA)**: Performs risk assessment
- **Registration Agent (RA)**: Finalizes loan registration

### API Endpoints:
1. `POST /api/v1/loan-applications` - Initial loan application creation
2. `POST /api/v1/credit/check` - Credit bureau verification
3. `POST /api/v1/auto-loan/valuation/vehicle/appraise` - Vehicle appraisal
4. `GET /api/v1/encumbrances/collateral/{collateralId}/active` - Active lien check
5. `POST /api/v1/risk-assessment/evaluate` - Risk assessment evaluation
6. `PUT /api/v1/loan-applications/{id}/approve` - Final loan approval

### Success Flow:
Application → Credit Check → Collateral & Lien Verification → Risk Assessment → Final Approval → APPROVED

### Failure Points & HITL Tools:
Each step has defined failure conditions that trigger Human-in-the-Loop intervention for manual review and decision-making.

## API Projects for Demo Simulation

- Loan Service: https://github.com/rjtmahinay/loan-service
- Credit Bureau Service: https://github.com/rjtmahinay/credit-bureau-service
- Collateral Service: https://github.com/rjtmahinay/collateral-service 
- Internal Risk Engine Service: https://github.com/rjtmahinay/internal-risk-engine-service
