---
name: prompt-engineering
description: Designs, optimizes, and validates high-performance prompts for LLMs and AI agents. Use when improving vague prompts, building reusable prompt templates, enforcing structured outputs, or designing agent workflows with verification loops.
metadata:
  author: nick-brennan-system
  version: "2.0"
  category: ai-core-skill
---

# 🧠 Prompt Engineering Agent Skill

## Purpose
Transform user intent into structured, high-performance prompts that produce reliable, repeatable, and verifiable outputs.

This skill follows modern prompt engineering principles:
- Clear instructions
- Structured formatting
- Iterative refinement
- Verification loops

---

## Core Prompt Framework (MANDATORY)

All prompts MUST follow this structure:

[ROLE]  
Define expertise and perspective

[TASK]  
Explicit objective using action verbs

[CONTEXT]  
Relevant background, constraints, or data

[INSTRUCTIONS]  
Step-by-step execution rules

[OUTPUT FORMAT]  
Exact structure required (JSON, table, bullets)

[CONSTRAINTS]  
Restrictions and rules

[EXAMPLES] (optional)  
Input → Output samples

[SELF-CHECK] (REQUIRED)
- Validate output format
- Validate constraints
- Fix errors automatically if needed

---

## Execution Workflow

### Step 1: Diagnose
- Identify user intent
- Detect ambiguity
- Identify missing constraints

### Step 2: Classify Prompt Type
- Generation
- Transformation
- Extraction
- Reasoning
- Agent Workflow

### Step 3: Construct Prompt
- Add structured framework
- Add explicit instructions
- Define output format
- Add constraints

### Step 4: Apply Reasoning Strategy
- SIMPLE → direct execution
- STANDARD → structured steps
- COMPLEX → plan → execute → verify

### Step 5: Add Verification Loop (CRITICAL)

All prompts must include:

[SELF-CHECK]
- Did output follow format? (YES/NO)
- Did output meet constraints? (YES/NO)
- If NO → regenerate corrected output

---

## Output Modes

### Mode A: Optimized Prompt Only
Returns production-ready prompt

### Mode B: Full Breakdown (DEFAULT)
- Issues detected
- Optimized prompt
- Explanation
- Variants

### Mode C: System Template
Reusable prompt template for workflows

---

## Prompt Design Rules

- Use explicit verbs (Write, Build, Analyze, Extract)
- Never assume context — define it
- Avoid conflicting instructions
- Always define output structure
- Prefer constraints over suggestions
- Break complex tasks into steps
- Force deterministic outputs when possible

---

## Advanced Capabilities

### 🔹 Multi-Step Prompt Systems
Split tasks into:
1. Planning Prompt
2. Execution Prompt
3. Verification Prompt

---

### 🔹 Meta Prompting Loop

1. Diagnose failure
2. Rewrite prompt
3. Validate output

---

### 🔹 Structured Output Enforcement

Always enforce:
- Tables
- JSON
- Bullet systems

---

### 🔹 Agent Integration

When used in agents:
- Define tool usage conditions
- Require explanation before actions
- Enforce step-by-step execution

---

### Verification Block

[SELF-CHECK]
- Did output follow format? (YES/NO)
- Did it meet constraints? (YES/NO)
- If NO → fix automatically

---

### Execution Block

[MODE]
- FAST → minimal reasoning
- BALANCED → default
- DEEP → multi-step + validation

---

### Stop Conditions

[STOP CONDITIONS]
- Task is complete
- All constraints satisfied
- Output validated

---

### Tool Awareness 

[TOOLS]
- Use tools when needed
- Explain action before using tool

---


## Example

### Input
"Make this better"

### Output

[ROLE]
You are a senior content editor specializing in clarity and engagement.

[TASK]
Rewrite the provided content to improve clarity, readability, and engagement.

[CONTEXT]
The content may contain vague language and weak structure.

[INSTRUCTIONS]
- Improve sentence clarity
- Remove redundancy
- Strengthen wording
- Maintain original meaning

[OUTPUT FORMAT]
Rewritten version followed by bullet list of improvements

[CONSTRAINTS]
- Do not change intent
- Do not add new ideas

[SELF-CHECK]
- Is the content clearer? (YES/NO)
- Are constraints followed? (YES/NO)

---

## Edge Case Handling

- If prompt is vague → infer intent and state assumptions
- If no format provided → enforce structured output
- If task is complex → split into steps
- If already optimized → refine minimally

---

## Integration Notes

Use this skill in:
- Copilot agents
- MCP systems
- Prompt libraries
- Workflow automation

Store reusable prompts in:
assets/templates.md

---

## End of Skill
