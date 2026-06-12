# RCM Call Automation Platform — Tech Stack

## Stack Philosophy

The product is **not** a fully autonomous voice bot.

Instead, it is a workflow platform that:

1. Checks electronic payer channels first
2. Automates IVR navigation
3. Waits on hold
4. Warm-transfers a live payer representative to a biller
5. Generates structured notes after the call

The stack therefore prioritizes:

- Telephony control
- Workflow orchestration
- PHI-safe data handling
- Configurable payer IVRs
- Auditability
- Human-in-the-loop review

---

# Recommended MVP Stack

| Layer | Tool | Why |
|--------|------|-----|
| Frontend | Next.js + React | Fast dashboard development and easy deployment |
| Backend | Next.js API Routes (FastAPI later) | Lightweight orchestration with easy upgrade path |
| Database | PostgreSQL (Supabase) | Relational claim data and audit logs |
| Telephony | Twilio | Best support for IVR navigation, DTMF, and warm transfers |
| Workflow | Inngest | Event-driven orchestration |
| AI | GPT-4o / GPT-4o Mini | Structured extraction and summarization |
| Storage | Supabase Storage (S3 later) | Store recordings and transcripts |
| Deployment | Vercel | Fast deployment for MVP |
| Monitoring | Sentry | Error tracking and logging |

---

# 1. Frontend

## Chosen: Next.js + React

### Used For

- A/R Workqueue
- Claim dashboard
- Call monitoring
- Human review queue
- Audit logs

### Why This Over React + Express

Next.js keeps the frontend and backend together in a single project.

Benefits:

- Built-in API Routes
- Faster development
- Simpler deployment
- Shared TypeScript types

### Tradeoff

As the platform grows, backend services should eventually move into FastAPI for better scalability and cleaner separation.

---

# 2. Backend

## Chosen: Next.js API Routes (MVP)

### Used For

- Creating call sessions
- Starting IVR workflows
- Receiving Twilio webhooks
- Saving call metadata
- Triggering GPT extraction
- Updating A/R tasks

### Why

The backend mainly orchestrates workflows rather than performs heavy computation.

### Tradeoff

API Routes aren't ideal for long-running jobs or complex integrations.

---

## Future: FastAPI

### Used For

- EHR integrations
- Clearinghouse integrations
- Portal integrations
- 276/277 APIs
- Internal business logic

### Why

FastAPI provides:

- Better async performance
- Background workers
- Automatic API documentation
- Easier microservice architecture

---

# 3. Database

## Chosen: PostgreSQL (Supabase)

### Used For

- Patients
- Claims
- Payers
- IVR sessions
- Call history
- Transcripts
- Audit logs
- Human review tasks

### Why This Over MongoDB

Healthcare billing data is highly relational.

```text
Patient
   ↓
Claim
   ↓
Payer
   ↓
Call
   ↓
Transcript
   ↓
Review Task
```

Postgres provides:

- Transactions
- Foreign keys
- Reliable querying
- Auditability

### Tradeoff

MongoDB is more flexible for unstructured documents, but claims naturally fit relational databases.

---

# 4. Telephony

## Chosen: Twilio

### Used For

- Dialing payers
- DTMF keypad entry
- IVR navigation
- Hold detection
- Call bridging
- Warm transfer
- Recording (when permitted)

### Why This Over Vapi / Retell

The difficult engineering problem is **telephony**, not AI conversation.

We need:

- Precise DTMF control
- Call state events
- Hold detection
- Conference calls
- Warm transfer

Twilio exposes these low-level primitives.

### Tradeoff

Twilio requires more engineering effort than higher-level AI voice platforms.

---

# 5. AI Layer

## Chosen: GPT-4o

### Used For

- Call transcription analysis
- Structured information extraction
- Call summaries
- Denial reason extraction
- Suggested next actions

### Not Used For

- Speaking to payer representatives
- Negotiating claims
- Making reimbursement decisions

### Why

The AI assists the biller after the conversation rather than replacing them.

### Tradeoff

LLMs are probabilistic.

To reduce risk:

- Generate structured JSON
- Include confidence scores
- Require human approval before actions

---

# 6. IVR Navigation Engine

## Chosen: Configurable State Machines

### Used For

Representing each payer's phone tree.

Example:

```json
{
  "payer": "Aetna",
  "steps": [
    {
      "prompt": "claims",
      "input": "1",
      "method": "dtmf"
    },
    {
      "prompt": "member_id",
      "field": "member_id",
      "method": "dtmf"
    }
  ]
}
```

### Why

IVRs change frequently.

Keeping IVRs as configuration allows:

- Faster updates
- No redeployment
- Non-engineer maintenance

### Tradeoff

Configuration must be maintained as payer phone trees change.

---

# 7. Phonetic / DTMF Library

## Chosen: Custom Internal Module

### Used For

Converting identifiers into IVR-safe inputs.

Supports:

- Member IDs
- NPIs
- Claim numbers
- Tax IDs
- Dates of service

Outputs:

- DTMF
- NATO phonetics
- Digit-by-digit speech

### Why

This is core infrastructure for IVR reliability.

Whenever possible, DTMF is preferred because it is:

- More reliable
- Faster
- Better for PHI protection

### Tradeoff

Requires payer-specific testing and maintenance.

---

# 8. Workflow Orchestration

## Chosen: Inngest

### Used For

Managing long-running workflows.

```text
Create Call
      ↓
Dial Payer
      ↓
Navigate IVR
      ↓
Wait on Hold
      ↓
Representative Detected
      ↓
Warm Transfer
      ↓
Call Ends
      ↓
Generate Notes
      ↓
Update Claim
```

### Why This Over Celery

Inngest integrates well with a Next.js stack while requiring much less infrastructure.

### Tradeoff

Production systems with very high throughput may eventually migrate to Temporal, Celery, or AWS Step Functions.

---

# 9. Storage

## MVP: Supabase Storage

Stores:

- Call recordings
- Transcripts
- Generated notes

## Future: Amazon S3

Provides:

- Better scalability
- Lifecycle policies
- Enterprise storage

### Tradeoff

Supabase is faster for an MVP, while S3 is more production-ready.

---

# 10. Deployment

## MVP: Vercel

### Why

- Native Next.js support
- Preview deployments
- Fast iteration

### Future

Separate deployment:

- Frontend → Vercel
- Backend → AWS
- Database → Managed PostgreSQL

---

# 11. Security & Compliance

## Core Principles

- Prefer DTMF over spoken identifiers
- Encrypt data at rest and in transit
- Mask PHI in logs
- Role-based access control
- Full audit trails
- HIPAA-eligible vendors with BAAs
- Recording consent controls

PHI protection is treated as a core architectural requirement, not an afterthought.

---

# Final Architecture

```text
                    Next.js Frontend
                           │
                           ▼
                 Next.js API Routes
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
     PostgreSQL        Inngest         Twilio
     (Supabase)      Workflow Engine   Telephony
          │                │                │
          │                ▼                ▼
          │         IVR Navigation     Payer Phone Tree
          │                │                │
          │                ▼                ▼
          │         Hold Detection    Warm Transfer
          │                │                │
          └────────────────┴────────────────┘
                           │
                           ▼
                     Human Biller
                           │
                           ▼
                  Transcript + Metadata
                           │
                           ▼
                        GPT-4o
                           │
                           ▼
          Structured Notes + Updated A/R Task
```

---

# Summary

### MVP

- Next.js
- Supabase
- Twilio
- Inngest
- GPT-4o
- Vercel

### Production

- Next.js
- FastAPI
- PostgreSQL
- Twilio
- Temporal/Celery
- GPT-4o
- AWS
- S3

The startup's competitive advantage is **automating IVR navigation and hold time**, not replacing human billers with conversational AI.