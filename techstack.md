# Technology Decisions

The goal is to automate payer calls, not overbuild infrastructure. The stack should prioritize fast iteration, reliable voice workflows, structured extraction, and human review.

---

## 1. Next.js vs React + Express

### Option A

**React + Express**

### Option B

**Next.js — Chosen**

### Why

- One codebase for frontend and backend
- Built-in API routes
- Fast dashboard development
- Easy Vercel deployment

### Tradeoff

Next.js is great for an MVP, but a larger RCM platform may need a dedicated backend for integrations, background jobs, and complex business logic.

---

## 2. Next.js API Routes vs FastAPI

### Option A

**FastAPI**

### Option B

**Next.js API Routes — Chosen for MVP**

### Why

The MVP only needs simple endpoints:

- Start call
- Save transcript
- Run extraction
- Update claim
- Load workqueue

### Tradeoff

FastAPI is better long-term for async services, API documentation, and backend separation.

---

## 3. Vapi / Retell vs Twilio

### Option A

**Twilio**

### Option B

**Vapi / Retell — Chosen for MVP**

### Why

Vapi and Retell already handle:

- Speech recognition
- Voice synthesis
- Turn-taking
- Interruptions
- LLM conversation flow

This lets the team focus on RCM logic instead of low-level telephony.

### Tradeoff

Twilio gives more production control over IVRs, routing, call handling, and enterprise telephony, but takes longer to build with.

---

## 4. GPT-4o vs Rule-Based Parsing

### Option A

**Regex / Rules**

### Option B

**GPT-4o — Chosen**

### Why

Payer reps say the same thing in many different ways.

GPT-4o can normalize messy conversations into structured fields like:

```json
{
  "correction_possible": true,
  "appeal_applicable": false,
  "investigation_required": false
}
```

### Tradeoff

LLMs can be wrong, so outputs need confidence scores, evidence, and human review flags.

---

## 5. PostgreSQL / Supabase vs MongoDB

### Option A

**MongoDB**

### Option B

**PostgreSQL / Supabase — Chosen**

### Why

RCM data is relational:

```text
Patient
  ↓
Claim
  ↓
Denial
  ↓
Call
  ↓
Transcript
  ↓
Human Review Task
```

Postgres supports structured querying, transactions, and audit logs.

### Tradeoff

MongoDB is more flexible for unstructured data, but claim workflows need relational consistency.

---

## 6. Inngest vs Celery

### Option A

**Celery + Redis**

### Option B

**Inngest — Chosen for MVP**

### Why

The workflow is event-driven:

```text
Start Call
    ↓
Transcript Ready
    ↓
Extract JSON
    ↓
Update Claim
    ↓
Notify Human Reviewer
```

Inngest is easier to set up in a JavaScript/Next.js stack.

### Tradeoff

Celery is more powerful for high-volume distributed jobs, but requires more infrastructure.

---

## 7. Vercel vs AWS

### Option A

**AWS**

### Option B

**Vercel — Chosen for MVP**

### Why

- Easy Next.js deployment
- Fast previews
- Low setup
- Good for demos

### Tradeoff

Production healthcare infrastructure may eventually need AWS for stronger control, monitoring, networking, and compliance architecture.

---

# Proposed MVP Architecture

```text
                Next.js Frontend
                       │
                       ▼
             Next.js API Routes
                       │
       ┌───────────────┴────────────────┐
       │                                │
       ▼                                ▼
  Vapi / Retell                  Supabase Postgres
       │                                │
       ▼                                ▼
Insurance Company            Claims / A/R Tasks
       │
       ▼
Call Transcript
       │
       ▼
GPT-4o Structured Extraction
       │
       ▼
Correction / Appeal / Investigation Decision
       │
       ▼
Human Review Queue
```

---

# Build Order

## Phase 1: Mock RCM Data

- Create sample denied claims
- Add patient/provider/payer fields
- Add denial reasons and claim status
- Build A/R workqueue

## Phase 2: Call Flow

- Create pre-call packet
- Define denial call script
- Use simulated payer call or voice provider
- Capture transcript

## Phase 3: Decision Logic

Implement decision tree:

Verify identity
    ↓
Correction possible?
    ↓
Appeal applicable?
    ↓
Investigation required?
    ↓
Human review?


## Phase 4: Structured Output

- Extract denial reason
- Extract correction needed
- Extract appeal path
- Extract reference number
- Assign confidence
- Flag human review

## Phase 5: Review UI

- Show transcript
- Show structured extraction
- Show recommended next action
- Let human approve or investigate

---

# Success Metrics

- Calls completed
- Reference numbers captured
- Identity verification success rate
- Correction opportunities found
- Appeal paths identified
- Investigation escalations
- Human review rate
- Time saved per claim
- Claims updated with structured notes