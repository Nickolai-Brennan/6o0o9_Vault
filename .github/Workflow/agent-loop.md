# 🔁 Agent Execution Loop

## Purpose
Automate multi-step development using chained agents.

---

## FLOW

### STEP 1 — Prompt Engineer
- Convert user input into structured prompt

### STEP 2 — System Architect
- Design:
  - File structure
  - Tech stack
  - Components

### STEP 3 — Code Engineer
- Generate:
  - Code
  - Config
  - Setup

### STEP 4 — Validator
- Check:
  - Errors
  - Logic issues
  - Missing dependencies

---

## LOOP RULE

If validation fails:
→ Send output back to Prompt Engineer
→ Re-run flow

Repeat until:
- No errors
- Output meets constraints

---

## STOP CONDITIONS

- Code compiles
- Output structured correctly
- All constraints satisfied
