---
name: copilot-studio-topic-to-skill
description: |
  Converts a Microsoft Copilot Studio classic topic (AdaptiveDialog YAML — the
  kind exported from the topic code editor, with beginDialog / OnRecognizedIntent /
  triggerQueries / Question / ConditionGroup nodes) into a new Copilot Studio
  agent **skill** in the platform's native skill format ("When this skill is
  activated:" steps + Guidelines + Examples + Notes). Use when the user says
  "convert this topic to a skill", "turn this Copilot Studio topic into a skill",
  "migrate this AdaptiveDialog YAML", "convert my topic YAML", "topic to skill",
  or pastes Copilot Studio topic YAML and asks for a skill.
  Do NOT use to author NEW topics from scratch, to convert the other direction
  (skill → topic), or to build/validate generic Cowork skills — use the `skills`
  skill for that.
cowork:
  category: automation
  icon: DocumentArrowRight
---

## Overview

Takes a Copilot Studio **classic topic** (AdaptiveDialog YAML) and emits a **new
Copilot Studio agent skill** in the platform's native format. The classic pattern
(`OnRecognizedIntent` + `triggerQueries` + chained `Question`/`ConditionGroup`/flow
nodes) becomes: a **name** and **description** (the skill's UI fields), a **"When this
skill is activated:"** numbered step list, **Guidelines**, **Examples**, and **Notes**.
The conversion is deterministic for routing/slot-filling; constructs with no skill
equivalent are surfaced in **Notes** as `REVIEW:` items rather than dropped.

## When to Use

- User pastes topic YAML (`kind: AdaptiveDialog …`) and wants a Copilot Studio skill out of it.
- User drops a `.yml` / `.yaml` / `.topic.mcs.yml` file in their materials and asks to convert it.
- User is migrating a classic-orchestration agent's topics to a generative-orchestration skill.

## When NOT to Use

- **Authoring new topics or generic Cowork skills** — use the `skills` skill.
- **skill → topic** (reverse direction) — not supported; say so.
- **Power Automate flow / connector definitions** — only AdaptiveDialog topics convert here.

## Output format (Copilot Studio skill)

Copilot Studio's **Add skill** dialog ("Create from blank") has three **required** fields —
**Name**, **Description**, and **Instructions**. Produce all three, in this shape:

```
Name: <skill name — required, max 64 chars>

Description: <required, max 1024 chars — describe what the skill does AND when to use it.
Fold in 3-5 example phrases so the orchestrator routes to it.>

Instructions:

When this skill is activated:

1. [First step or action the agent should take]
2. [Second step or action]

## Guidelines

- [Key guideline or constraint]
- [Another important consideration]

## Examples

**Example 1: [Scenario name]**
- User request: "[Example user input]"
- Expected behavior: [How the agent should respond]

## Notes

[Edge cases, data shapes, dropped-construct notices, and REVIEW: items.]
```

All three fields are required. The `When this skill is activated:` block (down through Notes) is
the **Instructions** body and matches Copilot Studio's placeholder verbatim. The output works two
ways: paste each field into the dialog, or save the whole thing as a skill file and use the
dialog's **"Upload a skill"** option.

## Input Handling

1. If the user **pasted YAML** in the message, use it directly.
2. Otherwise look for a topic file: `Glob` `input/**/*.{yml,yaml}` (topics are commonly `*.topic.mcs.yml`). If multiple, ask which one.
3. If neither is found, ask the user to paste the topic YAML or attach the file. Never invent topic content.

## Conversion Mapping (the core logic — apply every row)

| Classic topic construct | → Copilot Studio skill target | Rule |
|---|---|---|
| `modelDescription` (root) | **Description** field (primary) | Port as the lead sentence of the description. |
| `beginDialog.intent.triggerQueries` (phrase list) | **Description** field + **Examples** | Fold 3-5 phrases into the description as `Example phrases: …`; turn representative phrases into `## Examples` "User request" lines. |
| Topic display name / file name | **Name** field | kebab-case or short display name. |
| `Question` node (`variable` + `prompt` + `entity`) | A numbered **activation step** ("Ask the user for …") | One step per Question, in order. Human-friendly field name so the agent can auto-prompt. |
| `entity:` prebuilt (`PersonName`, `Email`, `PhoneNumber`, `StreetAddress`, `City/State/ZipCode`, `String`, etc.) | Type/validation note in the step or a **Guideline** | "valid email", "two-letter state", etc. — validate before use. |
| `ClosedListEntity` / `optionSetName` + `items` | Step that offers the **accepted values** | List the option `displayName`s (e.g. "Email or Mail"). |
| `ConditionGroup` → `conditions[].condition` (Power Fx) | **Conditional activation steps** ("If …, …") + a Guideline | Translate each branch to plain language. Keep non-trivial Power Fx as a `REVIEW:` note. |
| Tool / flow / action calls (`InvokeFlowAction`, `InvokeConnectorAction`, `SearchAndSummarizeContent`, redirect to another topic) | "Call **`<ExactName>`** …" **activation step** | Preserve the exact flow/action name + its inputs and output variable. Put `latencyMessage` in a step ("While it runs, say …"). |
| `Message` / `SendActivity` text (no card) | An **activation step** (respond/present) | Fixed copy → a step; dynamic content → "present `<var>`". |
| `SendActivity` attachment `AdaptiveCardTemplate` | Data → a "present these fields" **step** + a **Notes** disclosure | New Copilot Studio has no authored cards. Do NOT reproduce the card. Keep the flow's data, present it as a generated response, capture any `Action.OpenUrl` as a text link, and add a Note that the card wasn't carried over. |
| `SetVariable`, `Global.`/`Topic.`/`Dialog.` plumbing, loops, `ParseValue` | **Notes** → `REVIEW:` item | No clean skill equivalent — surface for human review; never silently drop. |

**Two hard rules** (generative-orchestration guidance):
- Step/field names must be **human-friendly** ("email address", "start date") so the agent's auto-generated questions read naturally.
- The **Description** must be **specific**, seeded with example phrases — not vague ("handles requests"). It drives routing.

## Procedure

1. Acquire the topic YAML (see Input Handling).
2. Parse the `AdaptiveDialog`: read `beginDialog`, its `intent`/`triggerQueries`, `modelDescription`, and the ordered `actions` (recursing into `ConditionGroup.conditions[].actions`).
3. Build the **Name** and **Description** fields (description = `modelDescription`/intent summary + 3-5 example phrases).
4. Build the **When this skill is activated:** numbered steps in the topic's action order — Question nodes become "ask for …" steps, flow/action calls become "call …" steps, ConditionGroups become conditional steps.
5. For any `AdaptiveCardTemplate`, follow the card row: keep the data step, present results as a generated response (any OpenUrl as a text link), and add the "card not carried over" Note.
6. Build **## Guidelines** from branch constraints, entity validation, and any fixed rules.
7. Build **## Examples** from representative trigger phrases — each as `**Example N: …**` with a `User request` and `Expected behavior`.
8. Build **## Notes** with data shapes, the dropped-card notice, and `REVIEW:` items for anything in the catch-all mapping row.
9. Write the result to `output/<name>/SKILL.md` and tell the user what converted cleanly, what was flagged, and what was dropped.

## Guardrails

- **Match the Copilot Studio skill format exactly** — "When this skill is activated:" steps, then `## Guidelines`, `## Examples`, `## Notes`. Keep Name/Description as separate fields.
- **Never fabricate topic behavior** — convert only what the YAML contains. Omit empty sections.
- **Preserve exact flow/action names verbatim** — they are load-bearing for routing.
- **Don't reproduce adaptive cards** — keep the data as a "present these results" step, capture any OpenUrl as a text link, and add a Note that the card wasn't carried over.
- **Flag, don't drop** — anything lossy (Power Fx, SetVariable, loops) becomes a `REVIEW:` item in Notes.
- If the YAML is malformed or isn't an `AdaptiveDialog`, say so and show the parse point — don't guess.
