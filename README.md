# copilot-studio-topic-to-skill

A reusable Cowork skill that converts a **Microsoft Copilot Studio classic topic**
(AdaptiveDialog YAML) into a **new Copilot Studio agent skill** (generative orchestration),
in the platform's native skill format.

New Copilot Studio's generative orchestration no longer relies on hand-built topic
flows (`OnRecognizedIntent` + `triggerQueries` + chained `Question`/`ConditionGroup`
nodes). This skill turns that classic YAML into the three fields of the **Add skill** dialog:
a **Name**, a **Description** (what it does and when to use it), and **Instructions** in the
platform's native format — "When this skill is activated:" steps, plus Guidelines, Examples, and Notes.

## What's in this folder

| File | Purpose |
|------|---------|
| `SKILL.md` | The skill itself — this is the shareable artifact. |
| `README.md` | This file. |
| `examples/travel-brochure.topic.yaml` | Sample input: a simple slot-filling topic. |
| `examples/travel-brochure-request.SKILL.md` | The output (Name + Description + Instructions, with a conditional branch). |
| `examples/workday-goals.topic.yaml` | Sample input: a topic that calls a flow and renders an **adaptive card**. |
| `examples/workday-goals.SKILL.md` | The output — flow call + data presented as a generated response, with a notice that the card wasn't carried over. |

## How to install (for each person you share it with)

1. Copy the `copilot-studio-topic-to-skill/` folder (containing `SKILL.md`) into your
   personal Cowork skills folder:
   `…/.claude/skills/copilot-studio-topic-to-skill/SKILL.md`
   (In Cowork this lives in your OneDrive under `Documents/Cowork`.)
2. The folder name **must match** the `name:` in the SKILL.md (`copilot-studio-topic-to-skill`).
3. Start a new Cowork session — the skill is now available.

## How to use

Paste a classic topic's YAML (or attach a `.yml` / `.topic.mcs.yml` file) and say:

> "Convert this Copilot Studio topic to a SKILL.md"

The skill writes the converted `SKILL.md` to `output/` and tells you what converted
cleanly and what (if anything) was flagged for human review.

## What it converts (mapping)

| Classic topic construct | → Becomes in the Copilot Studio skill |
|---|---|
| `modelDescription` | **Description** field (primary) |
| `triggerQueries` phrase list | **Description** example phrases + `## Examples` "User request" lines |
| Topic name | **Name** field |
| `Question` nodes | numbered "When this skill is activated" steps ("ask for…") |
| Prebuilt entities (PersonName, Email, Phone, Address…) | type / validation note in the step or a Guideline |
| `ClosedListEntity` / option sets | a step offering the accepted values |
| `ConditionGroup` | conditional activation steps + a Guideline |
| `Message` / `SendActivity` (no card) | an activation step (respond / present) |
| Flow / connector / action calls | "call `<ExactName>`" activation step |
| Power Fx, `SetVariable`, loops | `REVIEW:` items in `## Notes` |
| `AdaptiveCardTemplate` | not reproduced — data kept, card-not-supported notice in Notes (see below) |

Anything without a clean equivalent is **flagged, never silently dropped**.

## Adaptive cards

New Copilot Studio (generative orchestration) doesn't use authored adaptive cards — the agent
generates responses from tool outputs. So when the source topic renders an `AdaptiveCardTemplate`,
the converter **works around it**: it keeps the flow/data behind the card, turns the card's fields
into a "present these results" instruction (and any button into a text link), and adds a visible
notice to the output that the card was **not carried over** (card-only styling — logos, backgrounds,
layout — is dropped). It never reproduces the card or bundles card JSON. It still flags placeholder
action URLs and persona/prompt text baked into flow inputs. See the `workday-goals` example.

## Notes

- The output gives you all three required Copilot Studio skill fields — **Name**, **Description**,
  and **Instructions** — so you can paste them into the **Add skill** dialog or save the file and
  use the dialog's **"Upload a skill"** option.
- This skill does **one** direction: topic → skill. It does not author new topics or convert a
  skill back into a topic.
