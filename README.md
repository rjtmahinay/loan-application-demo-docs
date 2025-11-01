# SOFTCON PH 2025: Loan Application Demo Documentation

This repository contains comprehensive documentation for the Loan Application Demo for Software Conference Philippines 2025, including workflow diagrams, API specifications, and deployment procedures for RedHat OpenShift Developer Sandbox.

**Presentation Title:** Beyond Chatbots: Unlocking the Power of Agentic AI with watsonx Orchestrate

## 📋 Table of Contents

- [Overview](#overview)
- [Demo Architecture](#demo-architecture)
- [watsonx Orchestrate Free Trial](#-watsonx-orchestrate-setup)
- [API Projects](#api-projects)
- [OpenShift Developer Sandbox Setup](#openshift-developer-sandbox-setup)
- [Deployment Procedures](#deployment-procedures)
- [Additional Resources](#additional-resources)

## 🔍 Overview

The Loan Application Demo showcases an automated loan processing workflow using IBM watsonx Orchestrate. The system processes loan applications through multiple verification steps including credit checks, collateral validation, and risk assessment.

## 🏗️ Demo Architecture

The loan application process consists of 6 main steps orchestrated by watsonx Orchestrate:

1. **Step 0**: Initial Submission (LOS Creation)
2. **Step 1**: Verification & Credit Check
3. **Step 2**: Collateral & Lien Check
4. **Step 3**: Risk Decision Gate
5. **Step 4**: Final Registration
6. **Step 5**: Final Outcome

For detailed workflow diagrams and step-by-step processes, see [DEMO-FLOW.md](./DEMO-FLOW.md).

### Agents and Responsibilities

| Agent | Responsibility | API Endpoint |
|-------|----------------|--------------|
| **Orchestrator** | Manages overall workflow | N/A |
| **Registration Agent** | Loan application registration and approval | `POST /api/v1/loan-applications`, `PUT /api/v1/loan-applications/ID/approve` |
| **Customer Verification Agent** | Credit checks and document verification | `POST /api/v1/credit/check` |
| **Encumbrance Agent** | Collateral valuation and lien checks | `POST /api/v1/auto-loan/valuation/vehicle/appraise`, `GET /api/v1/encumbrances/collateral/ID/active` |
| **Underwriting Agent** | Risk assessment evaluation | `POST /api/v1/risk-assessment/evaluate` |

## 🤖 AI Agents for watsonx Orchestrate

This section documents the AI agents created for the loan application workflow using watsonx Orchestrate. Each agent is designed as a specialized component that handles specific aspects of the loan processing pipeline.

Learn on how to define AI agents programatically in [Authoring Agents in watsonx Orchestrate](https://developer.watson-orchestrate.ibm.com/agents/build_agent)

### Agent Configuration Files

The agents are configured using YAML specifications that define their behavior, tools, and integration patterns:

```
agents/
├── markdown/                    # Documentation files
│   ├── CUSTOMER_VERIFICATION_AGENT.md
│   ├── ENCUMBRANCE_AGENT.md
│   ├── LOAN_ORCHESTRATOR.md
│   ├── REGISTRATION_AGENT.md
│   └── UNDERWRITING_AGENT.md
└── yaml/                        # watsonx Orchestrate configurations
    ├── customer_verification_agent.yaml
    ├── encumbrance_agent.yaml
    ├── loan_orchestrator.yaml
    ├── registration_agent.yaml
    └── underwriting_agent.yaml
```

### Agent Specifications

#### 1. Loan Orchestrator (`loan_orchestrator.yaml`)

**Model**: `meta-llama/llama-3-405b-instruct`  
**Style**: ReAct (Think-Act-Observe loop)  
**Role**: Primary supervisor and coordinator

**Key Features**:
- **Sequential Workflow Management**: Orchestrates the 5-step loan process
- **Dynamic Task Delegation**: Routes tasks to specialized agents based on context
- **Error Handling & Escalation**: Manages failures and routes to manual review
- **Data Aggregation**: Combines results from all agents for final decision

**Tools**:
- `get_customer_by_id`
- `search_customers_by_name` 
- `get_loan_applications_by_customer_id`
- `start_loan_application_review`

**Collaborators**: All other agents (customer_verification_agent, encumbrance_agent, registration_agent, underwriting_agent)

#### 2. Customer Verification Agent (`customer_verification_agent.yaml`)

**Model**: `meta-llama/llama-3-405b-instruct`  
**Style**: Default  
**Role**: Credit Eligibility Validator

**Key Features**:
- **Identity Validation**: Confirms applicant's SSN, firstName, lastName
- **Credit Assessment**: Interfaces with external Credit Bureau Service
- **Financial Metrics Extraction**: Retrieves creditScore and verified annualIncome
- **Security Compliance**: Handles PII data according to security protocols

**Tools**:
- `perform_credit_check` (POST /api/v1/credit/check)

**Primary Functions**:
1. Receive customer data from Orchestrator
2. Map data to CreditCheckRequest payload
3. Execute credit check via external API
4. Extract and return financial metrics
5. Signal success/failure to Orchestrator

#### 3. Encumbrance Agent (`encumbrance_agent.yaml`)

**Model**: `meta-llama/llama-3-405b-instruct`  
**Style**: Default  
**Role**: Collateral Quality & Legal Status Validator

**Key Features**:
- **Sequential Processing**: Performs valuation first, then lien check
- **Vehicle Valuation**: Determines current market value of collateral
- **Legal Verification**: Confirms no active liens or encumbrances
- **Risk Mitigation**: Prevents lending against compromised collateral

**Tools**:
- `appraise_vehicle_value` (POST /api/v1/auto-loan/valuation/vehicle/appraise)
- `get_active_encumbrances_by_collateral` (GET /api/v1/encumbrances/collateral/{id}/active)

**Primary Functions**:
1. Receive vehicle details (VIN, make, model, year, zipCode)
2. Execute vehicle appraisal to get collateralId and market value
3. Perform lien check using collateralId
4. Return success only if both checks pass
5. Signal specific failure reasons for escalation

#### 4. Registration Agent (`registration_agent.yaml`)

**Model**: `meta-llama/llama-3-405b-instruct`  
**Style**: Default  
**Role**: Loan Origination System (LOS) Gateway

**Key Features**:
- **Data Assembly**: Combines customer data with loan application details
- **Dual Function**: Handles both initial submission and final approval
- **Comprehensive Data Collection**: Manages 12 required fields for loan applications
- **Transaction Management**: Ensures proper status tracking throughout process

**Tools**:
- `submit_loan_application` (POST /api/v1/loan-applications)
- `approve_loan_application` (PUT /api/v1/loan-applications/{id}/approve)

**Required Data Fields**:
- `loanType`, `loanAmount`, `loanTermMonths`
- `purpose`, `downpayment`, `monthlyDebtPayments`
- `employmentYears`, `vin`, `make`, `year`, `model`, `zipCode`

**Primary Functions**:
1. **Initial Registration**: Collect application data and create LOS entry
2. **Data Validation**: Ensure all required fields are present and valid
3. **Final Approval**: Update application status to APPROVED after underwriting
4. **Status Management**: Track application through SUBMITTED → APPROVED states

#### 5. Underwriting Agent (`underwriting_agent.yaml`)

**Model**: `meta-llama/llama-3-405b-instruct`  
**Style**: Default  
**Role**: Final Automated Decision-Maker

**Key Features**:
- **Risk Model Execution**: Interfaces with Internal Risk Engine Service
- **Comprehensive Data Aggregation**: Combines all validated data into single payload
- **Binary Decision Making**: Returns clear approve/deny recommendation
- **Interest Rate Calculation**: Provides recommended rate for approved applications

**Tools**:
- `evaluate_loan_application_risk` (POST /api/v1/risk-assessment/evaluate)

**Primary Functions**:
1. Aggregate data from all previous agents (credit, collateral, loan terms)
2. Construct comprehensive LoanApplication payload
3. Execute risk assessment via Internal Risk Engine
4. Extract approvalRecommendation and recommendedInterestRate
5. Signal final decision to Orchestrator for completion or escalation

### Configuration Standards

All agents follow consistent YAML configuration standards:

```yaml
spec_version: v1
kind: native
name: <agent_name>
llm: meta-llama/llama-3-405b-instruct
style: default | react
hide_reasoning: False
description: |
  <Multi-line agent description>
instructions: |
  <Detailed behavior instructions and guidelines>
collaborators: []  # List of other agents (for Orchestrator only)
tools: []          # List of available tools/APIs
knowledge_base: []
restrictions: editable
```

## 🌐 watsonx Orchestrate Setup

- Follow the steps in [Free Trial Documentation](https://www.ibm.com/docs/en/watsonx/watson-orchestrate/base?topic=orchestrate-accessing-trial-version)
- The free trial version has 2 default available models:
   - `meta-llama/llama-3-405b-instruct`
   - `meta-llama/llama-3-2-90b-vision-instruct`

## 🚀 API Projects

The demo consists of four microservice API projects:

| Service | Repository | Description |
|---------|------------|-------------|
| **Loan Service** | [loan-service](https://github.com/rjtmahinay/loan-service) | Core loan application management and approval |
| **Credit Bureau Service** | [credit-bureau-service](https://github.com/rjtmahinay/credit-bureau-service) | Credit score verification and financial validation |
| **Collateral Service** | [collateral-service](https://github.com/rjtmahinay/collateral-service) | Vehicle appraisal and collateral valuation |
| **Internal Risk Engine Service** | [internal-risk-engine-service](https://github.com/rjtmahinay/internal-risk-engine-service) | Risk assessment and underwriting decisions |

## 🌐 OpenShift Developer Sandbox Setup

### Prerequisites

Before deploying the API projects, ensure you have:

1. **RedHat Developer Account**: Sign up at [developers.redhat.com](https://developers.redhat.com/)
2. **OpenShift Developer Sandbox Access**: Request access at [developers.redhat.com/developer-sandbox](https://developers.redhat.com/developer-sandbox)
3. **Git Account**: GitHub, GitLab, or Bitbucket account with repository access
4. **OpenShift CLI (oc)**: Download from [OpenShift CLI Tools](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html)

### Initial Setup

1. **Access OpenShift Developer Sandbox**
   ```bash
   # Navigate to your OpenShift console
   https://console-openshift-console.apps.sandbox-m2.ll9k.p1.openshiftapps.com/
   ```

2. **Login via CLI**
   ```bash
   # Get login token from OpenShift console (Copy Login Command)
   oc login --token=sha256~XXXX --server=https://api.sandbox-m2.ll9k.p1.openshiftapps.com:6443
   ```

3. **Verify Connection**
   ```bash
   oc whoami
   oc projects
   ```

## 📦 Deployment Procedures

### Method 1: Using OpenShift Web Console (Recommended for Beginners)

#### Step 1: Deploy Loan Service

1. **Access Developer Console**
   - Navigate to OpenShift Developer Sandbox
   - Switch to **Developer** perspective
   - Select your project namespace

2. **Import from Git**
   - Click **+Add** → **Import from Git**
   - **Git Repo URL**: `https://github.com/rjtmahinay/loan-service`
   - **Git Reference**: `main` (or your default branch)
   - **Context Dir**: `/` (leave empty if repository root)

3. **General Settings**
   - **Application**: `loan-application-demo`
   - **Name**: `loan-service`
   - **Resources**: `Deployment`
   - **Create a route to the Application**: ✅ Checked

4. **Build Configuration**
   - **Builder Image**: Auto-detect (Node.js, Java, Python, etc.)
   - **Builder Image Version**: Latest
   - **Environment Variables**: Add any required configuration
   - **Build Triggers**: ✅ Configure triggers for Git webhook

5. **Deployment Configuration**
   - **Environment Variables**: Add runtime configuration
   - **Labels**: `app=loan-service, component=backend`
   - **Resource Limits**:
     - CPU: `500m` (or as needed)
     - Memory: `512Mi` (or as needed)

6. **Networking**
   - **Create a route to the Application**: ✅ Checked
   - **Target Port**: `8080` (adjust based on service configuration)
   - **Secure Route**: Enable if HTTPS is required
   - **TLS Termination**: Edge (if secure route enabled)

7. **Advanced Options**
   - **Scaling**: Set initial replica count (default: 1)
   - **Health Checks**: Configure readiness and liveness probes
   - **Storage**: Add persistent volumes if needed

8. **Deploy**
   - Review all configurations
   - Click **Create**
   - Monitor build progress in **Builds** section
   - Verify route creation in **Routes** section
   - Check pod deployment in **Topology** view

#### Step 2: Deploy Credit Bureau Service

1. **Import from Git**
   - Click **+Add** → **Import from Git**
   - **Git Repo URL**: `https://github.com/rjtmahinay/credit-bureau-service`
   - **Git Reference**: `main` (or your default branch)
   - **Context Dir**: `/` (leave empty if repository root)

2. **General Settings**
   - **Application**: Select existing `loan-application-demo`
   - **Name**: `credit-bureau-service`
   - **Resources**: `Deployment`
   - **Create a route to the Application**: ✅ Checked

3. **Build Configuration**
   - **Builder Image**: Auto-detect (Node.js, Java, Python, etc.)
   - **Builder Image Version**: Latest
   - **Environment Variables**: Add any required configuration
   - **Build Triggers**: ✅ Configure triggers for Git webhook

4. **Deployment Configuration**
   - **Environment Variables**: Add runtime configuration
   - **Labels**: `app=credit-bureau-service, component=backend`
   - **Resource Limits**:
     - CPU: `500m` (or as needed)
     - Memory: `512Mi` (or as needed)

5. **Networking**
   - **Create a route to the Application**: ✅ Checked
   - **Target Port**: Adjust based on service requirements
   - **Secure Route**: Enable if HTTPS is required
   - **TLS Termination**: Edge (if secure route enabled)

6. **Deploy and Verify**
   - Review all configurations
   - Click **Create**
   - Monitor build progress in **Builds** section
   - Verify route creation in **Routes** section
   - Ensure each service has its own unique route

#### Step 3: Deploy Collateral Service

1. **Import from Git**
   - Click **+Add** → **Import from Git**
   - **Git Repo URL**: `https://github.com/rjtmahinay/collateral-service`
   - **Git Reference**: `main` (or your default branch)
   - **Context Dir**: `/` (leave empty if repository root)

2. **General Settings**
   - **Application**: Select existing `loan-application-demo`
   - **Name**: `collateral-service`
   - **Resources**: `Deployment`
   - **Create a route to the Application**: ✅ Checked

3. **Build Configuration**
   - **Builder Image**: Auto-detect (Node.js, Java, Python, etc.)
   - **Builder Image Version**: Latest
   - **Environment Variables**: Add any required configuration
   - **Build Triggers**: ✅ Configure triggers for Git webhook

4. **Deployment Configuration**
   - **Environment Variables**: Add runtime configuration
   - **Labels**: `app=collateral-service, component=backend`
   - **Resource Limits**:
     - CPU: `500m` (or as needed)
     - Memory: `512Mi` (or as needed)

5. **Networking**
   - **Create a route to the Application**: ✅ Checked
   - **Target Port**: Adjust based on service requirements
   - **Secure Route**: Enable if HTTPS is required
   - **TLS Termination**: Edge (if secure route enabled)

6. **Deploy and Verify**
   - Review all configurations
   - Click **Create**
   - Monitor build progress in **Builds** section
   - Verify route creation in **Routes** section
   - Ensure each service has its own unique route

#### Step 4: Deploy Internal Risk Engine Service

1. **Import from Git**
   - Click **+Add** → **Import from Git**
   - **Git Repo URL**: `https://github.com/rjtmahinay/internal-risk-engine-service`
   - **Git Reference**: `main` (or your default branch)
   - **Context Dir**: `/` (leave empty if repository root)

2. **General Settings**
   - **Application**: Select existing `loan-application-demo`
   - **Name**: `internal-risk-engine-service`
   - **Resources**: `Deployment`
   - **Create a route to the Application**: ✅ Checked

3. **Build Configuration**
   - **Builder Image**: Auto-detect (Node.js, Java, Python, etc.)
   - **Builder Image Version**: Latest
   - **Environment Variables**: Add any required configuration
   - **Build Triggers**: ✅ Configure triggers for Git webhook

4. **Deployment Configuration**
   - **Environment Variables**: Add runtime configuration
   - **Labels**: `app=internal-risk-engine-service, component=backend`
   - **Resource Limits**:
     - CPU: `500m` (or as needed)
     - Memory: `512Mi` (or as needed)

5. **Networking**
   - **Create a route to the Application**: ✅ Checked
   - **Target Port**: Adjust based on service requirements
   - **Secure Route**: Enable if HTTPS is required
   - **TLS Termination**: Edge (if secure route enabled)

6. **Deploy and Verify**
   - Review all configurations
   - Click **Create**
   - Monitor build progress in **Builds** section
   - Verify route creation in **Routes** section
   - Ensure each service has its own unique route

### Method 2: Using OpenShift CLI

#### Step 1: Create Application and Deploy Services

```bash
# Create new application from Git repository - Loan Service
oc new-app https://github.com/rjtmahinay/loan-service \
  --name=loan-service \
  --labels=app=loan-application-demo

# Create route for Loan Service (using default name)
oc expose svc/loan-service

# Deploy Credit Bureau Service
oc new-app https://github.com/rjtmahinay/credit-bureau-service \
  --name=credit-bureau-service \
  --labels=app=loan-application-demo

# Create route for Credit Bureau Service (using default name)
oc expose svc/credit-bureau-service

# Deploy Collateral Service
oc new-app https://github.com/rjtmahinay/collateral-service \
  --name=collateral-service \
  --labels=app=loan-application-demo

# Create route for Collateral Service (using default name)
oc expose svc/collateral-service

# Deploy Internal Risk Engine Service
oc new-app https://github.com/rjtmahinay/internal-risk-engine-service \
  --name=internal-risk-engine-service \
  --labels=app=loan-application-demo

# Create route for Internal Risk Engine Service (using default name)
oc expose svc/internal-risk-engine-service
```

#### Step 2: Monitor Deployments

```bash
# Check build status
oc get builds

# Monitor deployment status
oc get deployments

# Check running pods
oc get pods

# View application routes
oc get routes
```

#### Step 3: Verify Individual Routes

```bash
# Check all routes are created
oc get routes

# Verify each service has its own route (using default names)
oc get route loan-service
oc get route credit-bureau-service
oc get route collateral-service
oc get route internal-risk-engine-service

# Get individual route URLs
LOAN_SERVICE_URL=$(oc get route loan-service -o jsonpath='{.spec.host}')
CREDIT_BUREAU_URL=$(oc get route credit-bureau-service -o jsonpath='{.spec.host}')
COLLATERAL_URL=$(oc get route collateral-service -o jsonpath='{.spec.host}')
RISK_ENGINE_URL=$(oc get route internal-risk-engine-service -o jsonpath='{.spec.host}')

echo "Individual Service URLs:"
echo "Loan Service: https://$LOAN_SERVICE_URL"
echo "Credit Bureau Service: https://$CREDIT_BUREAU_URL"
echo "Collateral Service: https://$COLLATERAL_URL"
echo "Risk Engine Service: https://$RISK_ENGINE_URL"
```

#### Step 4: Configure Service Communication

```bash
# Configure environment variables using external route URLs for service discovery
oc set env deployment/loan-service \
  CREDIT_BUREAU_SERVICE_URL=https://$CREDIT_BUREAU_URL \
  COLLATERAL_SERVICE_URL=https://$COLLATERAL_URL \
  INTERNAL_RISK_ENGINE_URL=https://$RISK_ENGINE_URL

# Alternative: Use internal service names for cluster communication
oc set env deployment/loan-service \
  CREDIT_BUREAU_URL=http://credit-bureau-service:8080 \
  COLLATERAL_SERVICE_URL=http://collateral-service:8080 \
  RISK_ENGINE_URL=http://internal-risk-engine-service:8080
```


## 📚 Additional Resources

### Documentation Links

- [OpenShift Developer Sandbox Getting Started](https://developers.redhat.com/developer-sandbox/get-started)
- [OpenShift CLI Reference](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/developer-cli-commands.html)
- [IBM watsonx Orchestrate Documentation](https://www.ibm.com/docs/en/watsonx/watson-orchestrate/base)
- [watsonx Orchestrate Development Guidelines](https://developer.watson-orchestrate.ibm.com/getting_started/guidelines)

### API Documentation

After deployment, API documentation will be available at:

- **Loan Service**: `https://<loan-service-route>/swagger-ui.html`
- **Credit Bureau Service**: `https://<credit-bureau-service-route>/swagger-ui.html`
- **Collateral Service**: `https://<collateral-service-route>/swagger-ui.html`
- **Internal Risk Engine**: `https://<risk-engine-service-route>/swagger-ui.html`

### Support and Community

- **Red Hat Developer Community**: [developers.redhat.com/community](https://developers.redhat.com/community)
- **OpenShift Documentation**: [docs.openshift.com](https://docs.openshift.com/)

### Quick Reference Commands

```bash
# Deploy all services at once with individual routes (using default names)
for repo in loan-service credit-bureau-service collateral-service internal-risk-engine-service; do
  oc new-app https://github.com/rjtmahinay/$repo --name=$repo --labels=app=loan-application-demo
  oc expose svc/$repo
done

# Check all service status
oc get all -l app=loan-application-demo

# Get all individual service URLs with route names
oc get routes -l app=loan-application-demo -o custom-columns=NAME:.metadata.name,URL:.spec.host --no-headers

# Verify individual routes are created (using default names)
echo "Individual Routes:"
for service in loan-service credit-bureau-service collateral-service internal-risk-engine-service; do
  route_url=$(oc get route ${service} -o jsonpath='{.spec.host}' 2>/dev/null)
  if [ ! -z "$route_url" ]; then
    echo "$service: https://$route_url"
  else
    echo "$service: Route not found"
  fi
done

# Scale all services
oc scale deployment --replicas=2 -l app=loan-application-demo

# View logs for all services
for service in loan-service credit-bureau-service collateral-service internal-risk-engine-service; do
  echo "=== $service logs ==="
  oc logs deployment/$service --tail=10
done

# Test all individual service endpoints (using default route names)
echo "Testing Individual Service Health Endpoints:"
for service in loan-service credit-bureau-service collateral-service internal-risk-engine-service; do
  route_url=$(oc get route ${service} -o jsonpath='{.spec.host}' 2>/dev/null)
  if [ ! -z "$route_url" ]; then
    echo "Testing $service at https://$route_url/actuator/health"
    curl -s -o /dev/null -w "%{http_code}" https://$route_url/actuator/health || echo " - Failed to connect"
    echo ""
  fi
done
```

## 📄 License

This project is licensed under the Apache-2.0 License - see the [LICENSE](./LICENSE) file for details.

---

**Note**: This documentation assumes familiarity with OpenShift and containerized applications. For beginners, it's recommended to start with the web console method before attempting CLI.
