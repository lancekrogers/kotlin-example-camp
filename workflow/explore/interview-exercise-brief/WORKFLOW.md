---
fest_type: workflow
fest_id: wf-interview-exercise-brief-WF
fest_parent: wf-interview-exercise-brief
fest_workflow_position: after
---

# Explore the FDE interview exercise brief


---

## Step 1: Land the source material — The brief exists in the camp as readable, diffable markdown alongside its original PDF.

**Goal:** The brief exists in the camp as readable, diffable markdown alongside its original PDF.

**Actions:**
1. Convert docs/interview-exercise.pdf to markdown with pdftotext -layout and restore headings/code fences by hand.
2. Keep both docs/interview-exercise.md and docs/interview-exercise.pdf under docs/.
3. Mark the markdown as a transcription of an external document, not as agent instructions.

**Checkpoint:** None — proceed to Step 2

---

## Step 2: Extract the hard requirements — Every mandatory deliverable in the brief is enumerated with no interpretation mixed in.

**Goal:** Every mandatory deliverable in the brief is enumerated with no interpretation mixed in.

**Actions:**
1. Walk sections 1-6 plus Submission and list each concrete deliverable verbatim.
2. Separate the six numbered obligations from the 'What We Care About' evaluation criteria.
3. Record the stated constraints: ~90 minutes, JVM 16 target, JDK 17/21 CI matrix, in-memory H2.

**Checkpoint:** None — proceed to Step 3

---

## Step 3: Identify the decision points — The choices the brief deliberately leaves open are named, with the trade-off for each.

**Goal:** The choices the brief deliberately leaves open are named, with the trade-off for each.

**Actions:**
1. Compare the three offered features (popular articles, user activity stats, article search) on implementation cost against demonstrated depth.
2. Note where the brief invites judgment: feature choice, test strategy, agent usage, CI scope beyond the minimum.
3. Flag the JVM 16 vs JDK 17/21 tension as an explicit decision that must be documented in the submission.

**Checkpoint:** None — proceed to Step 4

---

## Step 4: Write the brief analysis — A single note that a later design or festival can be planned from without re-reading the PDF.

**Goal:** A single note that a later design or festival can be planned from without re-reading the PDF.

**Actions:**
1. Summarize what is being evaluated (engineering, verification, agentic engineering, judgment, communication, FDE mindset) and what each maps to in practice.
2. State the recommended feature choice and the evidence the submission needs to produce.
3. Link forward to the codebase review workitem so the plan and the repo facts stay connected.

**Checkpoint:** None — workflow complete
