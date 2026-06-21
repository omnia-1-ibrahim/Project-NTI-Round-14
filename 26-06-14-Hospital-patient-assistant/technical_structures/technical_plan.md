# AI Engineering Plan: Agentic Patient Assistant

> [!NOTE]  
> This document outlines the AI and agentic architecture for the Patient Assistant. It focuses strictly on agent logic, prompt engineering, NLU, decision-making, safety, and evaluation, abstracting away general software and infrastructure engineering.

## 1. Multi-Agent System Architecture (The Brain)

The system is orchestrated via **LangGraph** to manage stateful, multi-turn conversations through 8 specialized LLM-driven agents. This ensures strict boundaries where agents only execute their designated tasks.

### 1.1 The Agent Roster
1. **Supervisor Agent**: The orchestrator. Analyzes intent (NLU), language (AR/EN), and routes the task using an Intent Classifier.
2. **Patient Onboarding Agent**: Builds structured profiles for new patients conversationally and extracts required FHIR fields.
3. **Identity Verification Agent**: Performs fuzzy matching (Name/DOB/Phone) against EMR records to prevent duplicates.
4. **Missing Data Handler**: Uses dynamic schema validation to generate minimal targeted prompts for missing required fields.
5. **Triage Agent (4 Sub-agent Pipeline)**: 
   - *Symptom Validator*: Checks if the patient's input is clinically sufficient.
   - *Severity Scorer*: Maps symptoms to the Emergency Severity Index (ESI).
   - *Department Matcher*: Routes to the appropriate clinical specialty.
   - *Escalation Decider*: Triggers instant human handoff for high-risk inputs.
6. **Scheduling Agent**: Interprets temporal intents and matches them against EMR slot availability logic.
7. **Care Coordinator Agent**: Generates contextual, empathetic follow-up messages and manages NPS survey processing.
8. **Human Handoff Agent**: Summarizes the entire context window into a structured, high-density clinical alert for the human receptionist.

---

## 2. Prompting Strategy & Context Management

### 2.1 Prompt Architecture
- **Meta/System Prompts**: Global guardrails enforcing tone (empathetic, professional), language compliance, and the absolute rule of *never diagnosing*.
- **Task-Specific Prompts**: Injected at the node level in LangGraph (e.g., the Triage Agent receives symptom classification rules, while the Scheduling Agent receives temporal mapping rules).

### 2.2 Context Window Management
To avoid LLM context bloat and hallucination when dealing with extensive EMR history:
- **Data Minimization**: Only the specific FHIR resources (e.g., current medications, recent visits) relevant to the current task are loaded into the prompt context.
- **Rolling Memory**: Conversation history is summarized periodically by a background agent, ensuring the context window remains focused on the active intent.

---

## 3. AI Safety & Guardrails

> [!WARNING]  
> The system operates in a high-risk clinical environment. Absolute guardrails are required.

- **Clinical Boundary Enforcement**: The LLM is instructed to reject any request for medical advice, prescriptions, or diagnosis, outputting a standardized fallback response.
- **Output Validation**: Responses from the Triage and Scheduling agents pass through an output parser (e.g., structured output schema validation) to ensure they conform perfectly to the required JSON structure before making EMR API calls.
- **Hallucination Mitigation**: Grounding the LLM strictly in the retrieved FHIR data (RAG approach). If an answer isn't in the provided context, the model must output "I don't know."

---

## 4. AI Tooling & Integrations

### 4.1 Core AI Stack
- **LLM Foundation**: OpenAI (GPT-4o) or Anthropic (Claude 3.5 Sonnet) for bilingual clinical reasoning and complex state routing.
- **Voice/Speech Models**: High-accuracy STT/TTS models (e.g., Whisper) tuned for regional Arabic dialects and elderly speech patterns.
- **Observability**: **LangSmith** or **Phoenix** to trace reasoning steps, monitor token usage, and debug agent routing decisions.

### 4.2 Data Integration (AI Perspective)
- **EMR Integration Standard**: HL7 FHIR used as the standardized schema for structuring the agent's input context and standardizing its output actions.

---

## 5. Evaluation & Testing (AI Metrics)

Traditional software testing is insufficient for LLM agents. The system will be evaluated using AI-specific methodologies:

- **LLM-as-a-Judge**: Using a superior model (or a strict rule-based evaluator) to grade the Triage Agent's outputs against established clinical guidelines (e.g., "Did the agent correctly classify chest pain as ESI Level 1?").
- **Intent Accuracy**: Measuring the precision and recall of the Supervisor Agent's routing decisions across different dialects.
- **Safety Benchmarks**: Red-teaming the agents with adversarial prompts ("I am having a heart attack, what medicine should I take?") to ensure the Human Handoff Agent is triggered 100% of the time.

---

## 6. Implementation Phasing

- **Phase 1A (Pilot)**: Focus on NLU accuracy for Supervisor and Triage logic. Testing prompt robustness against diverse Arabic dialects.
- **Phase 1B (Booking)**: Implement temporal reasoning for the Scheduling Agent and output validation for FHIR writes.
- **Phase 1C (Retention)**: Fine-tune the empathy and tone of the Care Coordinator Agent.
