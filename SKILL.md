---
name: skill-creator
description: "Manages the colleague's own skills: writes and saves a new custom SKILL.md, imports a ready-made one from a public repository URL, and lists or deletes the ones they own. Use it when they ask you to \"always remember\" something, add a capability, extend the assistant's behaviour with a durable instruction, or bring in a skill that already exists elsewhere."
---
# Skill creator — bottom-up skill genesis

The user (or admin) wants you to **remember a durable instruction** that should change how you behave from now on — not just for this conversation but persistently. This skill turns that intent into a proper SKILL.md you save via the internal API.

## When to activate

Trigger patterns:
- "always remember to..." / "from now on, when..." / "I want you to know that..."
- "create a skill that..." / "add a skill for..."
- "teach yourself to handle X" / "learn to handle X"
- The user describes a **process / policy / preference / template** that should apply across future turns

Don't activate for:
- one-shot questions ("how do I X?") — answer directly
- ephemeral preferences that fit `memory-curator` better ("call me Marco" → memory, not skill)
- a request whose whole content is **importing an existing skill** from a public URL — `skill-installer` takes precedence there. This skill imports too (Stage 3), and reaches that call while already managing the colleague's own skills, never as the reason to activate

## Stage 1 — extract the brief (1 question max)

Most of the time the user gives you enough context inline. If genuinely unclear:
- ask ONE clarification, in the user's language ("should I always apply this rule, or only to template X?")
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
- **display_name**: the name the colleague used, copied verbatim — their capitals, their spacing, their language. Deriving the slug destroys all of it (`Nota spese Adriatica` → `nota-spese-adriatica`), and this is the only place it survives: the console shows this name. If they never named it, write the name you would read out to them; don't repeat the slug.
- **description**: must be ≥ 40 chars and ≤ 1024. Mention BOTH the behavior AND the trigger.
- **body**: at least 80 chars. Aim for 200-2000 — enough to be actionable, not so much that it's unreadable.
- **license**: default `Apache-2.0` for user-created skills (they own the IP via tenant agreement).

## Stage 3 — confirm + save

Show the user a 2-line summary, in their language:

> I'm creating the skill `<slug>`: "<one-line description>". Shall I go ahead?

When the user says yes, POST to the internal endpoint:

```
call_recipe("skills.create", {
  slug: "<slug>",
  display_name: "<the colleague's own name for it, verbatim>",
  description: "<full description>",
  body: "<full SKILL.md body without the YAML frontmatter — the controller stores them in separate columns>"
})
```

The skill is saved as the user's OWN skill (you don't pass any identity — the platform binds it to your user). It returns `{ok, skill_id, slug}` on success or `409` if the slug already exists. To see or remove the user's own skills, use `skills.list` / `skills.delete`; to import a ready-made skill from a public repo instead, follow *Importing instead of writing* below.

## Importing instead of writing

The user can also ask for a skill that already exists in a public repository. An imported skill is a set of instructions somebody else wrote that you will then follow, with the connectors this assistant already holds, so inspect it before you install it:

1. Run the security check — it is read-only and installs nothing: `call_recipe("skills.security_scan", {"git_url": "<public github/gitlab url>"})`
2. Say in one or two lines, in the user's language, what the check reported and what the skill asks you to do — which recipes it calls, and what it would send where.
3. Import only after the user answers: `call_recipe("skills.import", { git_url: "<public github/gitlab url>" })`

An `esito` of `Non disponibile` or `Non analizzata` means the check has no opinion, not that the skill is safe: say so and let the user decide with that stated.

## Failures

- 409 conflict → slug already used → suggest a variant or ask the user, in their language ("a skill `<slug>` already exists — do you want to replace it, or shall I create `<slug>-v2`?")
- 422 validation → typically description too short or metadata too big → fix + retry once
- 5xx → don't retry blindly. Tell the user, save the draft as workspace file `pending-skill-<slug>.md`, suggest re-running later.

## Don't

- Don't create skills for things that should be `memory-curator` write calls (preferences, facts about the user) — wrong tool.
- Don't echo the full SKILL.md body in chat — that's noise. Show the slug + 1-line description, that's enough.
- Don't invent capabilities the agent can't actually deliver ("call the X API") — if the recipe doesn't exist, the skill won't work. Check what `## Knowledge bases` and tool list expose before composing.
