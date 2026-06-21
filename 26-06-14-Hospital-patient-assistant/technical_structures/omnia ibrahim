# Comprehensive Technical & AI Implementation Plan: Agentic Patient Assistant

> [!NOTE]  
> This document represents the unified master plan for the Patient Assistant. It seamlessly integrates the AI/Agentic logic with the foundational software engineering, infrastructure, and deployment architecture to provide a holistic view of the entire project.

---

## 1. System Architecture & Core Layers

The system relies on a highly decoupled architecture, ensuring that the AI reasoning layers are safely integrated with traditional software infrastructure.

### 1.1 Communication Layer (Interfaces)
- **WhatsApp Integration**: Using WhatsApp Business API (e.g., via Meta or Twilio) for bilingual (Arabic/English) text conversations.
- **Voice Gateway**: Interactive Voice Response (IVR) integrated with STT/TTS for elderly patients.
- **Website Widget**: A custom React/Vanilla JS chat interface via WebSockets.
- **Receptionist Dashboard**: A secure internal web dashboard (React/Next.js) receiving real-time clinical alerts via SSE.

### 1.2 Orchestration & State Management
- **Framework**: **LangGraph** orchestrates the multi-agent workflow, graph state, memory, and transitions.
- **State Store**: **Redis** for managing active conversation state and short-term memory, and **PostgreSQL** for session persistence.

### 1.3 Integration Layer (Data Exchange)
- **EMR/EHR System**: Deep integration via **HL7 FHIR API** to read patient profiles and write appointments.
- **Loyalty & Analytics DB**: An internal database tracking points, referrals, and NPS survey results.

---

## 2. Multi-Agent System Design

The system is powered by 8 specialized LLM-driven agents working in an assembly-line fashion.

1. **Supervisor Agent**: The orchestrator. Analyzes intent (NLU), language (AR/EN), and routes the task.
2. **Patient Onboarding Agent**: Builds structured profiles for new patients conversationally.
3. **Identity Verification Agent**: Performs fuzzy matching (Name/DOB/Phone) against EMR records to prevent duplicates.
4. **Missing Data Handler**: Uses dynamic schema validation to generate minimal targeted prompts for missing required fields.
5. **Triage Agent (4 Sub-agent Pipeline)**: 
   - *Symptom Validator*: Checks if the patient's input is clinically sufficient.
   - *Severity Scorer*: Maps symptoms to the Emergency Severity Index (ESI).
   - *Department Matcher*: Routes to the appropriate clinical specialty.
   - *Escalation Decider*: Triggers instant human handoff for high-risk inputs.
6. **Scheduling Agent**: Interprets temporal intents and matches them against EMR slot availability logic.
7. **Care Coordinator Agent**: Generates contextual follow-up messages and manages NPS survey processing.
8. **Human Handoff Agent**: Summarizes the entire context window into a structured, high-density clinical alert for the receptionist.

### 2.1 Agent Toolsets (Custom AI Tools)
To execute their tasks, these agents will be equipped with the following custom tools (Functions/APIs):
- `SearchEMRTool`: Queries the HL7 FHIR API to find existing patient records (used by Identity Agent).
- `CreatePatientProfileTool`: Sends a POST request to create a new FHIR Patient resource (used by Onboarding Agent).
- `ValidateSymptomsTool`: Checks if the reported symptoms contain enough context for triage.
- `CheckAvailabilityTool`: Queries the EMR calendar for open slots in a specific department (used by Scheduling Agent).
- `BookAppointmentTool`: Writes a confirmed appointment to the FHIR EMR (used by Scheduling Agent).
- `SendWhatsAppTool`: Triggers the WhatsApp Business API to send confirmations or follow-ups.
- `TriggerDashboardAlertTool`: Emits a WebSocket/SSE alert to the receptionist dashboard (used by Human Handoff Agent).

---

## 3. AI Engineering Layer

This layer governs how the agents "think," remember, and behave safely.

### 3.1 Prompting Strategy
- **Meta/System Prompts**: Global guardrails enforcing tone (empathetic, professional), language compliance, and the absolute rule of *never diagnosing*.
- **Task-Specific Prompts**: Injected at the node level in LangGraph (e.g., specific FHIR extraction rules for the Onboarding Agent).

### 3.2 Context Window Management
- **Data Minimization**: Only the specific FHIR resources (e.g., current medications, recent visits) relevant to the current task are loaded into the prompt context.
- **Rolling Memory**: Conversation history is summarized periodically by a background agent, ensuring the context window remains focused on the active intent without bloat.

### 3.3 AI Safety & Guardrails
- **Clinical Boundary Enforcement**: The LLM must reject requests for medical advice, prescriptions, or diagnosis.
- **Output Validation**: Agent responses pass through structured output schema validation to ensure they conform perfectly to JSON structures before EMR API calls.
- **Hallucination Mitigation**: Grounding the LLM strictly in the retrieved FHIR data (RAG). If an answer isn't in the provided context, the model defaults to "I don't know."

### 3.4 Evaluation & Testing (AI Metrics)
- **LLM-as-a-Judge**: Using a superior model to grade the Triage Agent's outputs against established clinical guidelines.
- **Safety Benchmarks (Red-teaming)**: Adversarial testing to ensure the Human Handoff Agent is triggered 100% of the time for critical scenarios.

---

## 4. Comprehensive Technology Stack

| Category | Recommended Technologies |
| :--- | :--- |
| **AI & LLM Provider** | OpenAI (GPT-4o) or Anthropic (Claude 3.5 Sonnet) |
| **Speech Models (STT/TTS)** | OpenAI Whisper, Google Cloud Speech-to-Text |
| **AI Observability** | LangSmith or Phoenix (Tracing & debugging) |
| **Agent Orchestration** | LangChain / LangGraph (Python or TypeScript) |
| **Backend API Framework** | FastAPI (Python) or NestJS (Node.js) |
| **Frontend Framework** | React.js / Next.js with TailwindCSS (Dashboard) |
| **Databases** | PostgreSQL (Relational) + Redis (Cache/State) |
| **Integration Standards** | HL7 FHIR (Smile CDR, GCP Healthcare API, Direct client) |
| **Cloud Infrastructure** | AWS, Azure, or GCP (HIPAA-compliant instances) |
| **DevOps & CI/CD** | GitHub Actions / GitLab CI, Datadog / ELK Stack |

---

## 5. Security & Compliance (HIPAA-Aligned)

> [!IMPORTANT]  
> Patient Health Information (PHI) is handled with the highest security standards.

1. **Encryption**: In-transit (**TLS 1.3**) and At-rest (**AES-256**) encryption for any temporarily stored state data.
2. **Data Minimization**: Strictly isolating and stripping irrelevant PHI before it reaches the LLM context window.
3. **Audit Trails**: Every LLM decision, EMR API call, and human escalation is logged securely with a timestamp.
4. **Access Control (RBAC)**: Multi-Factor Authentication (MFA) and strict role bounds for the dashboard.
5. **Cloud BAA**: Business Associate Agreements (BAA) signed with cloud and LLM providers.

---

## 6. Implementation Phasing

### Phase 1A: Pilot (Months 1–2)
- **Focus**: Single department (e.g., General Medicine). NLU accuracy for Supervisor and Triage logic.
- **Features**: WhatsApp text channel, Identity Check, Triage, Human Handoff, and prompt testing.

### Phase 1B: Booking & Live EMR (Months 3–4)
- **Focus**: End-to-end self-scheduling.
- **Features**: Scheduling Agent, live HL7 FHIR read/write integration, temporal logic validation.

### Phase 1C: Retention (Months 5–6)
- **Focus**: Patient follow-up and feedback.
- **Features**: Care Coordinator Agent, Loyalty system DB, NPS tracking.

### Phase 2: Scale (Month 7+)
- **Focus**: Accessibility and expansion.
- **Features**: Voice/Phone call channel integration, multi-department support, advanced analytics dashboard.
