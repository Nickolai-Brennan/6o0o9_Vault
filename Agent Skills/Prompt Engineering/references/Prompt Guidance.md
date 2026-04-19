# Prompt guidance for GPT-5.4

GPT-5.4, our newest mainline model, is designed to balance long-running task performance, stronger control over style and behavior, and more disciplined execution across complex workflows. Building on advances from GPT-5 through GPT-5.3-Codex, GPT-5.4 improves token efficiency, sustains multi-step workflows more reliably, and performs well on long-horizon tasks.

GPT-5.4 is designed for production-grade assistants and agents that need strong multi-step reasoning, evidence-rich synthesis, and reliable performance over long contexts. It is especially effective when prompts clearly specify the output contract, tool-use expectations, and completion criteria. In practice, the biggest gains come from choosing the right reasoning effort for the task, using explicit grounding and citation rules, and giving the model a precise definition of what "done" looks like. This guide focuses on prompt patterns and migration practices that preserve those efficiency wins. For model capabilities, API parameters, and broader migration guidance, see [our latest model guide](https://developers.openai.com/api/docs/guides/latest-model).

When troubleshooting cases where GPT-5.4 treats an intermediate update as the
  final answer, verify your integration preserves the assistant message `phase`
  field correctly. See [Phase parameter](#phase-parameter) for details.

## Understand GPT-5.4 behavior

### Where GPT-5.4 is strongest

GPT-5.4 tends to work especially well in these areas:

- Strong personality and tone adherence, with less drift over long answers
- Agentic workflow robustness, with a stronger tendency to stick with multi-step work, retry, and complete agent loops end to end
- Evidence-rich synthesis, especially in long-context or multi-tool workflows
- Instruction adherence in modular, skill-based, and block-structured prompts when the contract is explicit
- Long-context analysis across large, messy, or multi-document inputs
- Batched or parallel tool calling while maintaining tool-call accuracy
- Spreadsheet, finance, and Excel workflows that need instruction following, formatting fidelity, and stronger self-verification

### Where explicit prompting still helps

Even with those strengths, GPT-5.4 benefits from more explicit guidance in a few recurring patterns:

- Low-context tool routing early in a session, when tool selection can be less reliable
- Dependency-aware workflows that need explicit prerequisite and downstream-step checks
- Reasoning effort selection, where higher effort is not always better and the right choice depends on task shape, not intuition
- Research tasks that require disciplined source collection and consistent citations
- Irreversible or high-impact actions that require verification before execution
- Terminal or coding-agent environments where tool boundaries must stay clear

These patterns are observed defaults, not guarantees. Start with the smallest prompt that passes your evals, and add blocks only when they fix a measured failure mode.

## Use core prompt patterns

### Keep outputs compact and structured

To improve token efficiency with GPT-5.4, constrain verbosity and enforce structured output through clear output contracts. In practice, this acts as an additional control layer alongside the `verbosity` parameter in the Responses API, allowing you to guide both how much the model writes and how it structures the output.

```xml
<output_contract>
- Return exactly the sections requested, in the requested order.
- If the prompt defines a preamble, analysis block, or working section, do not treat it as extra output.
- Apply length limits only to the section they are intended for.
- If a format is required (JSON, Markdown, SQL, XML), output only that format.
</output_contract>

<verbosity_controls>
- Prefer concise, information-dense writing.
- Avoid repeating the user's request.
- Keep progress updates brief.
- Do not shorten the answer so aggressively that required evidence, reasoning, or completion checks are omitted.
</verbosity_controls>
```

### Set clear defaults for follow-through

Users often change the task, format, or tone mid-conversation. To keep the assistant aligned, define clear rules for when to proceed, when to ask, and how newer instructions override earlier defaults.

Use a default follow-through policy like this:

```xml
<default_follow_through_policy>
- If the user’s intent is clear and the next step is reversible and low-risk, proceed without asking.
- Ask permission only if the next step is:
  (a) irreversible,
  (b) has external side effects (for example sending, purchasing, deleting, or writing to production), or
  (c) requires missing sensitive information or a choice that would materially change the outcome.
- If proceeding, briefly state what you did and what remains optional.
</default_follow_through_policy>
```

Make instruction priority explicit:

```xml
<instruction_priority>
- User instructions override default style, tone, formatting, and initiative preferences.
- Safety, honesty, privacy, and permission constraints do not yield.
- If a newer user instruction conflicts with an earlier one, follow the newer instruction.
- Preserve earlier instructions that do not conflict.
</instruction_priority>
```

Higher-priority developer or system instructions remain binding.

**Guidance:** When instructions change mid-conversation, make the update explicit, scoped, and local. State what changed, what still applies, and whether the change affects the next turn or the rest of the conversation.

### Handle mid-conversation instruction updates

For mid-conversation updates, use explicit, scoped steering messages that state:

1. Scope
2. Override
3. Carry forward

```text
<task_update>
For the next response only:
- Do not complete the task.
- Only produce a plan.
- Keep it to 5 bullets.

All earlier instructions still apply unless they conflict with this update.
</task_update>
```

If the task itself changes, say so directly:

```text
<task_update>
The task has changed.
Previous task: complete the workflow.
Current task: review the workflow and identify risks only.

Rules for this turn:
- Do not execute actions.
- Do not call destructive tools.
- Return exactly:
  1. Main risks
  2. Missing information
  3. Recommended next step
</task_update>
```

### Make tool use persistent when correctness depends on it

Use explicit rules to keep tool use thorough, dependency-aware, and appropriately paced, especially in workflows where later actions rely on earlier retrieval or verification. A common failure mode is skipping prerequisites because the right end state seems obvious.

GPT-5.4 can be less reliable at tool routing early in a session, when context is still thin. Prompt for prerequisites, dependency checks, and exact tool intent.

```xml
<tool_persistence_rules>
- Use tools whenever they materially improve correctness, completeness, or grounding.
- Do not stop early when another tool call is likely to materially improve correctness or completeness.
- Keep calling tools until:
  (1) the task is complete, and
  (2) verification passes (see <verification_loop>).
- If a tool returns empty or partial results, retry with a different strategy.
</tool_persistence_rules>
```

This is especially important for workflows where the final action depends on earlier lookup or retrieval steps. One of the most common failure modes is skipping prerequisites because the intended end state seems obvious.

```xml
<dependency_checks>
- Before taking an action, check whether prerequisite discovery, lookup, or memory retrieval steps are required.
- Do not skip prerequisite steps just because the intended final action seems obvious.
- If the task depends on the output of a prior step, resolve that dependency first.
</dependency_checks>
```

Prompt for parallelism when the work is independent and wall-clock matters. Prompt for sequencing when dependencies, ambiguity, or irreversible actions matter more than speed.

```xml
<parallel_tool_calling>
- When multiple retrieval or lookup steps are independent, prefer parallel tool calls to reduce wall-clock time.
- Do not parallelize steps that have prerequisite dependencies or where one result determines the next action.
- After parallel retrieval, pause to synthesize the results before making more calls.
- Prefer selective parallelism: parallelize independent evidence gathering, not speculative or redundant tool use.
</parallel_tool_calling>
```

### Force completeness on long-horizon tasks

For multi-step workflows, a common failure mode is incomplete execution: the model finishes after partial coverage, misses items in a batch, or treats empty or narrow retrieval as final. GPT-5.4 becomes more reliable when the prompt defines explicit completion rules and recovery behavior.

Coverage can be achieved through sequential or parallel retrieval, but completion rules should remain explicit either way.

```xml
<completeness_contract>
- Treat the task as incomplete until all requested items are covered or explicitly marked [blocked].
- Keep an internal checklist of required deliverables.
- For lists, batches, or paginated results:
  - determine expected scope when possible,
  - track processed items or pages,
  - confirm coverage before finalizing.
- If any item is blocked by missing data, mark it [blocked] and state exactly what is missing.
</completeness_contract>
```

For workflows where empty, partial, or noisy retrieval is common:

```xml
<empty_result_recovery>
If a lookup returns empty, partial, or suspiciously narrow results:
- do not immediately conclude that no results exist,
- try at least one or two fallback strategies,
  such as:
  - alternate query wording,
  - broader filters,
  - a prerequisite lookup,
  - or an alternate source or tool,
- Only then report that no results were found, along with what you tried.
</empty_result_recovery>
```

### Add a verification loop before high-impact actions

Once the workflow appears complete, add a lightweight verification step before returning the answer or taking an irreversible action. This helps catch requirement misses, grounding issues, and format drift before commit.

```xml
<verification_loop>
Before finalizing:
- Check correctness: does the output satisfy every requirement?
- Check grounding: are factual claims backed by the provided context or tool outputs?
- Check formatting: does the output match the requested schema or style?
- Check safety and irreversibility: if the next step has external side effects, ask permission first.
</verification_loop>
```

```xml
<missing_context_gating>
- If required context is missing, do NOT guess.
- Prefer the appropriate lookup tool when the missing context is retrievable; ask a minimal clarifying question only when it is not.
- If you must proceed, label assumptions explicitly and choose a reversible action.
</missing_context_gating>
```

For agents that actively take actions, add a short execution frame:

```xml
<action_safety>
- Pre-flight: summarize the intended action and parameters in 1-2 lines.
- Execute via tool.
- Post-flight: confirm the outcome and any validation that was performed.
</action_safety>
```

## Handle specialized workflows

### Choose image detail explicitly for vision and computer use

If your workflow depends on visual precision, specify the image `detail` level in the prompt or integration instead of relying on `auto`. Use `high` for standard high-fidelity image understanding. Use `original` for large, dense, or spatially sensitive images, especially [computer use, localization, OCR, and click-accuracy tasks](https://developers.openai.com/api/docs/guides/tools-computer-use) on `gpt-5.4` and future models. Use `low` only when speed and cost matter more than fine detail. For more details on image detail levels, see the [Images and Vision guide](https://developers.openai.com/api/docs/guides/images-vision).

### Lock research and citations to retrieved evidence

When citation quality matters, make both the source boundary and the format requirement explicit. This helps reduce fabricated references, unsupported claims, and citation-format drift.

```xml
<citation_rules>
- Only cite sources retrieved in the current workflow.
- Never fabricate citations, URLs, IDs, or quote spans.
- Use exactly the citation format required by the host application.
- Attach citations to the specific claims they support, not only at the end.
</citation_rules>
```

```xml
<grounding_rules>
- Base claims only on provided context or tool outputs.
- If sources conflict, state the conflict explicitly and attribute each side.
- If the context is insufficient or irrelevant, narrow the answer or say you cannot support the claim.
- If a statement is an inference rather than a directly supported fact, label it as an inference.
</grounding_rules>
```

If your application requires inline citations, require inline citations. If it requires footnotes, require footnotes. The key is to lock the format and prevent the model from improvising unsupported references.

### Research mode

Push GPT-5.4 into a disciplined research mode. Use this pattern for research, review, and synthesis tasks. Do not force it onto short execution tasks or simple deterministic transforms.

```xml
<research_mode>
- Do research in 3 passes:
  1) Plan: list 3-6 sub-questions to answer.
  2) Retrieve: search each sub-question and follow 1-2 second-order leads.
  3) Synthesize: resolve contradictions and write the final answer with citations.
- Stop only when more searching is unlikely to change the conclusion.
</research_mode>
```

If your host environment uses a specific research tool or requires a submit step, combine this with the host's finalization contract.

### Clamp strict output formats

For SQL, JSON, or other parse-sensitive outputs, tell GPT-5.4 to emit only the target format and check it before finishing.

```text
<structured_output_contract>
- Output only the requested format.
- Do not add prose or markdown fences unless they were requested.
- Validate that parentheses and brackets are balanced.
- Do not invent tables or fields.
- If required schema information is missing, ask for it or return an explicit error object.
</structured_output_contract>
```

If you are extracting document regions or OCR boxes, define the coordinate system and add a drift check:

```text
<bbox_extraction_spec>
- Use the specified coordinate format exactly, such as [x1,y1,x2,y2] normalized to 0..1.
- For each box, include page, label, text snippet, and confidence.
- Add a vertical-drift sanity check so boxes stay aligned with the correct line of text.
- If the layout is dense, process page by page and do a second pass for missed items.
</bbox_extraction_spec>
```

### Keep tool boundaries explicit in coding and terminal agents

In coding agents, GPT-5.4 works better when the rules for shell access and file editing are unambiguous. This is especially important when you expose tools like [Shell](https://developers.openai.com/api/docs/guides/tools-shell) or [Apply patch](https://developers.openai.com/api/docs/guides/tools-apply-patch).

### User updates

GPT-5.4 does well with brief, outcome-based updates. Reuse the user-updates pattern from the 5.2 guide, but pair it with explicit completion and verification requirements.

Recommended update spec:

```xml
<user_updates_spec>
- Only update the user when starting a new major phase or when something changes the plan.
- Each update: 1 sentence on outcome + 1 sentence on next step.
- Do not narrate routine tool calls.
- Keep the user-facing status short; keep the work exhaustive.
</user_updates_spec>
```

For coding agents, see the Prompting patterns for coding tasks section below for more specific guidance.

### Prompting patterns for coding tasks

**Autonomy and persistence**

GPT-5.4 is generally more thorough end to end than earlier mainline models on coding and tool-use tasks, so you often need less explicit "verify everything" prompting. Still, for high-stakes changes such as production, migrations, or security work, keep a lightweight verification clause.

```xml
<autonomy_and_persistence>
Persist until the task is fully handled end-to-end within the current turn whenever feasible: do not stop at analysis or partial fixes; carry changes through implementation, verification, and a clear explanation of outcomes unless the user explicitly pauses or redirects you.

Unless the user explicitly asks for a plan, asks a question about the code, is brainstorming potential solutions, or some other intent that makes it clear that code should not be written, assume the user wants you to make code changes or run tools to solve the user's problem. In these cases, it's bad to output your proposed solution in a message, you should go ahead and actually implement the change. If you encounter challenges or blockers, you should attempt to resolve them yourself.
</autonomy_and_persistence>
```

**Intermediary updates**

Keep updates sparse and high-signal. In coding tasks, prefer updates at key points.

```xml
<user_updates_spec>
- Intermediary updates go to the `commentary` channel.
- User updates are short updates while you are working. They are not final answers.
- Use 1-2 sentence updates to communicate progress and new information while you work.
- Do not begin responses with conversational interjections or meta commentary. Avoid openers such as acknowledgements ("Done -", "Got it", or "Great question") or similar framing.
- Before exploring or doing substantial work, send a user update explaining your understanding of the request and your first step. Avoid commenting on the request or starting with phrases such as "Got it" or "Understood."
- Provide updates roughly every 30 seconds while working.
- When exploring, explain what context you are gathering and what you learned. Vary sentence structure so the updates do not become repetitive.
- When working for a while, keep updates informative and varied, but stay concise.
- When work is substantial, provide a longer plan after you have enough context. This is the only update that may be longer than 2 sentences and may contain formatting.
- Before file edits, explain what you are about to change.
- While thinking, keep the user informed of progress without narrating every tool call. Even if you are not taking actions, send frequent progress updates rather than going silent, especially if you are thinking for more than a short stretch.
- Keep the tone of progress updates consistent with the assistant's overall personality.
</user_updates_spec>
```

**Formatting**

GPT-5.4 often defaults to more structured formatting and may overuse bullet lists. If you want a clean final response, explicitly clamp list shape.

```xml
Never use nested bullets. Keep lists flat (single level). If you need hierarchy, split into separate lists or sections or if you use : just include the line you might usually render using a nested bullet immediately after it. For numbered lists, only use the `1. 2. 3.` style markers (with a period), never `1)`.
```

**Frontend tasks**

Use this only when additional frontend guidance is useful.

```xml
<frontend_tasks>
When doing frontend design tasks, avoid generic, overbuilt layouts.

Use these hard rules:
- One composition: The first viewport must read as one composition, not a dashboard, unless it is a dashboard.
- Brand first: On branded pages, the brand or product name must be a hero-level signal, not just nav text or an eyebrow. No headline should overpower the brand.
- Brand test: If the first viewport could belong to another brand after removing the nav, the branding is too weak.
- Full-bleed hero only: On landing pages and promotional surfaces, the hero image should usually be a dominant edge-to-edge visual plane or background. Do not default to inset hero images, side-panel hero images, rounded media cards, tiled collages, or floating image blocks unless the existing design system clearly r
