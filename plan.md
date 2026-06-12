# RCM Call Automation Platform

## One-Liner

An AI-assisted platform that eliminates payer hold time by automating claim-status checks, IVR navigation, and warm transfers so billers spend time solving claims instead of waiting on hold.

---

# Why This Matters

Revenue Cycle Management teams spend 20–40 minutes following up on a single claim.

Most of that time is spent:

* Calling insurance companies
* Navigating IVR phone trees
* Waiting on hold
* Repeating authentication information

The bottleneck is **biller time**, not claim complexity.

The goal is to maximize **dollars recovered per biller minute**.

---

# Product Philosophy

The platform **does not replace billers.**

Instead, it removes everything that happens **before** a biller speaks to a payer representative.

The workflow becomes:

```text
Bot waits.
Human solves.
```

---

# End-to-End Workflow

```text
Claim enters A/R Workqueue
        │
        ▼
Determine why follow-up is needed
        │
        ▼
Can the answer be obtained electronically?
        │
   ┌────┴────┐
   │         │
 Yes        No
   │         │
   ▼         ▼
Portal /    Phone Call Required
EDI 276
   │         │
   └────┬────┘
        ▼
Retrieve Claim Context
        │
        ▼
Build Pre-Call Packet
        │
        ▼
Navigate IVR
        │
        ▼
Authenticate Automatically
        │
        ▼
Wait on Hold
        │
        ▼
Representative Answers
        │
        ▼
Warm Transfer to Biller
        │
        ▼
Conversation Ends
        │
        ▼
Generate Structured Notes
        │
        ▼
Update A/R Workqueue
```

---

# MVP Build Plan

## Phase 1 — Electronic Status Engine

### Goal

Avoid unnecessary phone calls.

### Build

* EDI 276/277 status lookup
* Payer portal integrations
* Decision engine:

  * Can this claim be answered electronically?
  * If yes, stop.
  * If no, create a phone task.

Deliverable:

```text
Claim
      ↓
Electronic Lookup
      ↓
Portal / EDI Result
OR
Phone Call Required
```

---

## Phase 2 — Payer Directory

### Goal

Centralize payer information.

Build a payer database containing:

* Phone number
* IVR supported
* Portal URL
* 276/277 support
* Required authentication fields

Deliverable:

```text
Payer
 ├── Phone
 ├── Portal
 ├── EDI
 ├── Auth Requirements
 └── IVR Map
```

---

## Phase 3 — IVR Navigation Engine

### Goal

Navigate phone trees automatically.

Represent each payer IVR as a configurable state machine.

Example:

```text
Dial
 ↓
Press 1
 ↓
Enter Member ID
 ↓
Press 2
 ↓
Claims
 ↓
Hold
```

Store IVRs as configuration instead of code.

Deliverable:

Bot successfully reaches the correct department without human input.

---

## Phase 4 — Phonetic & DTMF Library

### Goal

Reliably enter claim identifiers.

Support:

* Member IDs
* NPIs
* Claim numbers
* Dates of service
* Tax IDs

Output:

* DTMF
* NATO phonetics
* Digit-by-digit speech

Prefer DTMF whenever possible.

Deliverable:

Reliable identifier entry across different payer IVRs.

---

## Phase 5 — Hold Detection

### Goal

Remove billers from waiting on hold.

Build:

* Hold music detection
* Human speech detection
* Representative detection

Deliverable:

Bot waits on hold without human involvement.

---

## Phase 6 — Warm Transfer

### Goal

Connect billers only when needed.

Workflow:

```text
Representative Answers
        │
        ▼
Ring Assigned Biller
        │
        ▼
Bridge Call
```

Provide a screen-pop showing:

* Patient
* Claim
* Denial reason
* Previous notes
* Call objective

Deliverable:

Biller joins only when the representative answers.

---

## Phase 7 — AI Call Assistant

The AI does **not** speak for the biller.

Instead it:

* Records the call
* Transcribes the conversation
* Extracts structured information
* Generates claim notes
* Suggests next actions

Example output:

```json
{
  "payer_status": "...",
  "reference_number": "...",
  "denial_reason": "...",
  "appeal_deadline": "...",
  "next_action": "..."
}
```

Deliverable:

The biller never manually writes call notes again.

---

# Future Roadmap

After the workflow is reliable:

* Automated IVR readout parsing
* Portal automation
* Appeal packet generation
* Denial classification
* Underpayment detection
* Workqueue prioritization
* Analytics dashboard
* Payer performance insights

---

# Success Metrics

Primary KPIs

* Hold time eliminated
* Calls avoided through portal/EDI
* Warm transfer success rate
* IVR navigation success rate
* Minutes saved per claim
* Claims resolved per biller hour

Secondary KPIs

* Structured note accuracy
* Time to first payer contact
* Average investigation time
* Appeal turnaround time
* Human satisfaction