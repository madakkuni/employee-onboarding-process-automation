# 🚀 Employee Onboarding Process Automation

## 📌 Overview

This automation is a **fully governed, approval-driven onboarding solution** that provisions employee accounts in Active Directory through Adaxes, triggered by ServiceNow tickets.

The process begins when a ServiceNow ticket meets predefined conditions (state, description, assignment group). Once triggered, the automation performs **end-to-end validation, data enrichment, and identity preparation**.

Instead of directly creating accounts, the automation calls the **Adaxes API** to create a provisioning request. This request is placed in a controlled state: WAITING FOR APPROVAL


The request requires **manager approval**, ensuring proper governance and access control.

A **separate background job** continuously monitors these requests. Once approval is granted:

- The AD account is created via Adaxes
- Account details are shared with the manager/requestor
- The ServiceNow ticket is updated and automatically closed

The entire lifecycle is **tracked using a SharePoint list** and **detailed log files**, providing full visibility, auditability, and troubleshooting capability.

This design ensures:
- No unauthorized account creation
- Complete audit trail
- Strong governance and compliance alignment

---

## ⚙️ Core Functionalities

### 🔹 Ticket-Driven Automation

Triggers automatically from ServiceNow tickets based on defined conditions and processes onboarding requests end-to-end.

### 🔹 Data Validation & Enrichment

Validates employee details, resolves department/office, and prepares identity attributes required for account creation.

### 🔹 Identity Generation

Generates AD account, email ID, OU mapping, and determines access requirements.

### 🔹 Approval-Based Provisioning

Creates a request in Adaxes instead of directly creating accounts, enforcing manager approval before execution.

### 🔹 Asynchronous Processing

Background job monitors approvals and completes account creation once approved.

### 🔹 Tracking & Observability

Tracks each request using SharePoint and detailed logs for full visibility and audit.

---

## 🎯 Trigger Conditions

Automation starts only when ALL conditions are met:

- Ticket state = **Open**
- Short description = **Onboarding Automation**
- Assignment group = **Automation Queue**

If conditions are not satisfied → ticket is skipped

---

## 🔁 End-to-End Flow

<img width="392" height="413" alt="image" src="https://github.com/user-attachments/assets/0ec40615-57cd-4f2b-beae-78651d039087" />

### 1. Initialization

- Initialize global state
- Load configuration
- Validate configuration

❌ If invalid → Exit

---

### 2. Ticket Fetching

- Retrieve tickets from ServiceNow

❌ API failure → Exit\
⚠️ No tickets → End process

---

### 3. Ticket Processing

#### a. Data Collection

- Extract ticket details
- Fetch request variables
- Map to processing state

#### b. Ticket Update

- Mark ticket as **In Progress**
- Assign to processing group

#### c. Tracking Initialization

- Create/Update entry in **SharePoint tracking list**
- Status = *Initiated*

---

### 4. Data Enrichment & Validation

- Fetch manager details
- Resolve department and office
- Validate employee ID
- Normalize names

---

### 5. Identity Preparation

- Generate AD account
- Determine Organizational Unit (OU)
- Generate email ID
- Identify mailbox requirement

---

### 6. Validation Layer

- Validate company data (if applicable)
- Validate email format
- Validate uniqueness of employee ID

❌ Any validation failure:

- Update tracker → FAILED
- Reassign ticket
- Stop processing

---

### 7. Account Check

- Check if AD account exists

#### If Account Exists:

- If Enabled → Validate → Close ticket
- If Disabled → Reassign for action

#### If Account Does Not Exist:

- Continue provisioning flow

---

### 8. Pre-Provision Processing

- Duplicate email validation
- Generate secure password

❌ If any error:

- Update tracker → FAILED
- Reassign ticket
- Stop

---

## 🔄 Adaxes-Based Provisioning Flow

### 📌 Role of Adaxes

Adaxes acts as the **front-end and control layer for Active Directory**, enabling:

- API-driven provisioning
- Approval workflows
- Policy enforcement

---

### 🧩 Request Creation

Once all validations pass:

1. Automation calls **Adaxes API**
2. A provisioning request is created
3. Request status is set to:

```text
WAITING FOR APPROVAL
```

4. Manager approval is required before account creation

---

## ⏳ Asynchronous Approval Processing

A **separate background job** continuously monitors Adaxes requests.

### Responsibilities:

- Check request approval status
- Detect approved requests

```text
WAITING FOR APPROVAL → APPROVED
```

---

## ✅ Post-Approval Actions

Once approved:

1. AD account is created via Adaxes
2. Account details are retrieved
3. Notification sent to manager/requestor
4. ServiceNow ticket is:
   - Updated with account details
   - Closed as **Completed**

---

## 📊 Tracking & Observability

### SharePoint Tracking

- Each ticket is tracked in SharePoint
- Tracks:
  - Ticket ID
  - Status
  - Processing stage
  - Timestamps

---

### Logging

- Step-by-step execution logs
- Error and failure logs
- API response logs

Used for:

- Troubleshooting
- Auditing

---

## 🔐 Security, Governance & Compliance

### 🔑 Service Account

- A common service account is used for:
  - ServiceNow access
  - Adaxes integration
  - Tenant-level operations

---

### 🛡️ Security Implementation

- Least privilege access enforced
- Credentials managed securely (not hardcoded)
- API-based controlled access

---

### 📋 Governance Model

- Approval-based provisioning enforced
- No direct account creation without approval
- Separation of duties (request vs approval)
- Centralized tracking via SharePoint

---

### 📑 Compliance

Supports:

- Identity governance policies
- Access control standards
- Audit and traceability requirements

---

## 🔧 Supporting Actions

### Close Ticket

- Marks success
- Updates ServiceNow
- Sends notification
- Tracks automation

---

### Reassign Ticket

- Marks failure
- Updates tracker
- Sends notification

---

### Apply Delay

- Prevents API throttling

---

## 🔗 Dependencies

- `api_utils` → ServiceNow & tracker APIs
- `process_request` → AD & Adaxes logic
- `common` → configuration & validation
- `send_email` → notifications
- `log` → logging
- `automation_tracker` → execution tracking

---

## ⚠️ Known Limitations

- Heavy use of global state
- Multiple tracker updates may create inconsistencies
- Some validations allow continuation after error

---

## 🚀 Future Enhancements

- Replace global state with structured models
- Introduce orchestration layer
- Convert into microservices
- Enable event-driven processing

---

## 🧠 Summary

### Stage 1: Request Creation

- Validate ticket and employee data
- Generate identity attributes
- Create Adaxes request
- Move to **Waiting for Approval**

### Stage 2: Post-Approval Processing

- Monitor approvals
- Complete account provisioning
- Notify stakeholders
- Close ServiceNow ticket

---

## 🧠 Summary

### Stage 1: Request Creation

- Validate ticket and employee data
- Generate identity attributes
- Create Adaxes request
- Move to **Waiting for Approval**

### Stage 2: Post-Approval Processing

- Monitor approvals
- Complete account provisioning
- Notify stakeholders
- Close ServiceNow ticket

---

### 💡 Key Strengths

- Approval-driven workflow ensuring no unauthorized access
- Strong governance and compliance enforcement
- Secure, API-based provisioning through controlled layers
- Centralized tracking via SharePoint and logs
- Full audit trail for every action performed
- Asynchronous design enabling scalability and reliability

---

## 🔐 Governance, Compliance & Secure Deployment (Detailed)

### 🛡️ Governance Implementation

- Enforced **approval-based access provisioning** through Adaxes
- Ensured **separation of duties** (request creation ≠ approval ≠ execution)
- Implemented **controlled entry points** via ServiceNow ticket conditions
- Standardized onboarding workflow across all requests

---

### 📋 Compliance Controls

- Every action is **logged and traceable**
- SharePoint list acts as a **system of record for audit tracking**
- Supports compliance requirements like:
  - Identity lifecycle management
  - Access governance policies
  - Audit readiness

---

### 🔒 Secure Deployment Approach

- Central **service account** used with controlled permissions
- No hardcoded credentials (config-based secure handling)
- API interactions restricted to approved endpoints
- Role-based access enforced in ServiceNow and Adaxes

---

### 📊 Tracking & Monitoring

- SharePoint list tracks:
  - Ticket lifecycle
  - Status transitions
  - Approval stages

- Logs provide:
  - Step-by-step execution trace
  - Error diagnostics
  - API-level visibility

- Background job ensures:
  - Continuous monitoring of approval status
  - Reliable completion of pending requests

---

This ensures the automation is not just functional, but also **secure, compliant, auditable, and production-ready for enterprise environments**.
