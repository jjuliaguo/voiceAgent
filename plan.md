# RCM Voice Agent Plan + Tech Stack Brainstorm

## One-Liner

An AI voice agent that calls insurance companies for healthcare providers, verifies claim information, checks for correctable denial issues first, determines whether appeals are applicable, and escalates complex cases to human review.

---

# Product Goal

Turn payer phone calls into structured RCM workflows.

```text
Denied Claim
    ↓
AI Payer Call
    ↓
Identity Verification
    ↓
Correction Check
    ↓
Appeal Check
    ↓
Investigation / Human Review
    ↓
Updated A/R Task
```

---

# Denial Call Workflow

For denial-related calls, the agent should not jump straight to appeals.

The priority order is:

```text
1. Verify identity and claim information
2. Check for simple corrections first
3. Check whether an appeal is applicable
4. If unclear, escalate to investigation
5. Loop in a human whenever the call goes off path
```

---

## Step 1: Identity Verification

The agent first verifies:

- Patient name
- Date of birth
- Member ID
- Provider NPI/TIN
- Claim number
- Date of service
- Authorization number, if applicable

If authentication fails, the agent escalates to a human.

---

## Step 2: Correction Check First

The agent checks whether the denial was caused by a correctable administrative issue.

Examples:

- Wrong member ID
- Incorrect DOB
- Missing authorization number
- Incorrect claim number
- Missing documentation
- Incorrect units
- Incorrect modifier
- Incorrect provider information
- Claim data mismatch

If correction is possible, the agent records what needs to be fixed and routes the claim for human approval before any corrected claim is submitted.

---

## Step 3: Appeal Check

If all identification and claim information is correct, the agent checks whether the denial is appealable.

Appeal-related examples:

- Medical necessity denial
- Incorrect payer determination
- Missing clinical evidence
- Authorization exists but was not recognized
- Underpayment or contract discrepancy

The agent gathers:

- Exact denial reason
- Required evidence
- Appeal deadline
- Payer instructions
- Fax/address/portal submission path
- Call reference number

Then it prepares an appeal packet for human review.

---

## Step 4: Investigation State

If all identification info is correct and there is no obvious correction, the claim should be escalated as:

```text
Requires Investigation
```

This means the denial may require deeper review by an RCM specialist.

Examples:

- Payer explanation is unclear
- Denial reason conflicts with claim data
- Multiple possible root causes
- Payer refuses to provide enough information
- Contract/payment issue is suspected
- The rep gives information that does not match the ERA/EOB

---

## Step 5: Human Loop-In

A human should be looped in whenever:

- Identity verification fails
- The payer response is ambiguous
- The call goes off script
- The payer asks for judgment
- The denial reaches investigation
- The agent cannot confidently classify correction vs appeal
- Any external submission is required

The agent gathers information and prepares documentation. It does not make final reimbursement decisions.

---

# Core Features

## 1. A/R Workqueue

Displays denied or unresolved claims with:

- Claim ID
- Payer
- Patient
- Balance
- Aging bucket
- Denial reason
- Current status
- Next action
- Human review flag

---

## 2. Pre-Call Packet Builder

Builds the call packet before contacting the payer.

Includes:

- Patient identity fields
- Provider identity fields
- Claim details
- Denial codes
- Authorization details
- Date of service
- Billed amount
- Call objective

---

## 3. AI Voice Agent

Calls the payer and follows a narrow script:

```text
Authenticate
    ↓
Ask denial reason
    ↓
Check for corrections
    ↓
Check appeal path
    ↓
Capture reference number
    ↓
Escalate if unclear
```

---

## 4. Structured Extraction

Turns the call transcript into JSON.

```json
{
  "claim_id": "...",
  "payer_claim_number": "...",
  "identity_verified": true,
  "denial_reason": "...",
  "correction_possible": true,
  "correction_needed": "missing authorization number",
  "appeal_applicable": false,
  "investigation_required": false,
  "next_action": "corrected_claim_review",
  "call_reference_number": "...",
  "confidence": "high",
  "human_review_required": true
}
```

---

## 5. Human Review Queue

Shows claims that need human judgment.

Reasons include:

- Investigation required
- Low confidence extraction
- Ambiguous payer response
- Correction needs approval
- Appeal packet needs approval