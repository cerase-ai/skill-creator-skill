---
slug: skill-creator
description: "Compila e salva una nuova skill SKILL.md custom quando l'utente / admin chiede di \"ricordare sempre\", aggiungere una nuova capability o estendere il comportamento dell'assistente con istruzioni durevoli."
is_core: true
---
# Skill creator — bottom-up skill genesis

The user (or admin) wants you to **remember a durable instruction** that should change how you behave from now on — not just for this conversation but persistently. This skill turns that intent into a proper SKILL.md you save via the internal API.

## When to activate

Trigger patterns (Italian/English):
- "ricordati sempre di..." / "d'ora in poi quando..." / "voglio che tu sappia che..."
- "crea una skill che..." / "add a skill for..."
- "insegna ti a fare X" / "learn to handle X"
- The user describes a **process / policy / preference / template** that should apply across future turns

Don't activate for:
- one-shot questions ("how do I X?") — answer directly
- ephemeral preferences that fit `memory-curator` better ("chiamami Marco" → memory, not skill)
- requests to **install an existing skill** from elsewhere — that's `skill-installer`

## Stage 1 — extract the brief (1 question max)

Most of the time the user gives you enough context inline. If genuinely unclear:
- ask ONE clarification ("vuoi che applichi questa regola sempre, o solo per il template X?")
- propose a **draft frontmatter + body**, show it to the user, get green light

## Stage 2 — compile the SKILL.md

Produce a SKILL.md following agentskills.io standard:

```
---
name: <kebab-case-slug>
description: <40-1024 chars: what the skill does AND when it should activate>
license: Apache-2.0
---

# <Title>

## When to activate

<concrete triggers — be explicit about user-facing patterns>

## What to do

<step-by-step process or rules>

## Don't

<failure modes / what NOT to do>
```

Rules:
- **slug**: kebab-case, 3-64 chars, [a-z0-9-]. Pick something specific (`forest-fattura-pec` over `policy-1`).
- **description**: must be ≥ 40 chars and ≤ 1024. Mention BOTH the behavior AND the trigger.
- **body**: at least 80 chars. Aim for 200-2000 — enough to be actionable, not so much that it's unreadable.
- **license**: default `Apache-2.0` for user-created skills (they own the IP via tenant agreement).

## Stage 3 — confirm + save

Show the user a 2-line summary:

> Sto creando la skill `<slug>`: "<one-line description>". Procedo?

When the user says yes, POST to the internal endpoint:

```
call_recipe("skills.create", {
  slug: "<slug>",
  description: "<full description>",
  body: "<full SKILL.md body without the YAML frontmatter — the controller stores them in separate columns>"
})
```

The skill is saved as the user's OWN skill (you don't pass any identity — the platform binds it to your user). It returns `{ok, skill_id, slug}` on success or `409` if the slug already exists. To import a ready-made skill from a public repo instead, use `call_recipe("skills.import", { git_url: "<public github/gitlab url>" })`; to see or remove the user's own skills, use `skills.list` / `skills.delete`.

## Failures

- 409 conflict → slug already used → suggest a variant or ask the user ("esiste già una skill `<slug>` — la vuoi sostituire o creo `<slug>-v2`?")
- 422 validation → typically description too short or metadata too big → fix + retry once
- 5xx → don't retry blindly. Tell the user, save the draft as workspace file `pending-skill-<slug>.md`, suggest re-running later.

## Don't

- Don't create skills for things that should be `memory-curator` write calls (preferences, facts about the user) — wrong tool.
- Don't echo the full SKILL.md body in chat — that's noise. Show the slug + 1-line description, that's enough.
- Don't invent capabilities the agent can't actually deliver ("chiama l'API X") — if the recipe doesn't exist, the skill won't work. Check what `## Knowledge bases` and tool list expose before composing.
