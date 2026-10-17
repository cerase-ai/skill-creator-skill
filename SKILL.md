---
name: skill-creator
description: "Writes, changes, imports and deletes the person's own skills: procedures the assistant follows whenever a given kind of request comes in. Load it when the person asks to save or create a skill, when the person has walked the assistant through a task in several steps, when the person asks again for a task already done, or when the person describes a routine; in the last three cases it offers once to save the task. Not for a fact about the person, a rule about how the assistant works, or a request that is only to import a skill from a URL."
---
# Skill creator

A skill is a procedure the assistant follows whenever a given kind of request comes in. This skill writes, changes, imports and deletes the person's own skills, and says when to offer to save a task as one.

In Italian, call a skill «competenza». When you offer to save one, explain it with the console's own sentence: «Una competenza è qualcosa che l'assistente ha imparato a fare, o la procedura che segue per farlo.»

## Offering to save a task

When the person walks you through a task in several steps, asks again for a task you already did, or describes a routine, do the task first. Then offer once, in one line, to save it as a skill. If the person declines or does not answer, do not offer it again in that conversation. Nothing is created until the person agrees. Do not offer for a one-off question, a rule about how you work, or a fact about the person.

## Before writing: list first

Before creating a skill, call `call_recipe("skills.list", {})` and read the person's own skills. If one of them already does the same task, change it with `skills.update` instead of creating a second one.

## Writing the skill

- Write the skill in English unless the person asks for another language. Talk to the person in their language.
- `display_name` is the person's own name for the skill, verbatim: their capitals, their spacing, their language.
- The `description` is short and in the third person. It says what the skill does and when to load it, because the body is read only after the description loaded the skill.
- The body holds the steps and nothing else. A personal skill is one file, so there is no reference material.
- The skill names no person, account, mailbox or address. It says "the person", or names the connection or connector. What changes on each run is asked for or looked up when the skill runs, never stored in it.
- The test for every sentence: would this text still be right if another colleague installed it?
- Name only recipes that exist, by their exact `call_recipe(...)` call. If something the skill needs does not exist, tell the person instead of inventing a call.
- The skill carries no time or day. Those stay in the scheduled message, which names the skill instead of repeating its steps.
- Write plain sentences, without slogans or motivational framing.
- Keep the field limits: slug is kebab-case, 3-64 characters, `[a-z0-9-]`; description is 40-1024 characters; body is at least 80 characters.

```
---
name: open-orders-summary
description: Summarises a supplier's open orders and lists the late ones. Use when the person asks where the orders of a supplier stand.
---

# Open orders summary

1. Ask the person which supplier, or read it from the request.
2. Read that supplier's open orders with the exact call_recipe(...) call its connector offers.
3. Group them by status and list the ones that are late.
4. Report the result in a short list.
```

An example `display_name` in Italian, with the person's capitals: `Riepilogo Ordini aperti`.

## Saving

Show the person a short summary in their language: the name and one line on what it does. Then make the call. The platform asks the person to approve it, and that approval is their confirmation, so do not ask for a second confirmation first.

```
call_recipe("skills.create", {
  slug: "<slug>",
  display_name: "<the person's own name, verbatim>",
  description: "<short third-person description>",
  body: "<the instructions without the frontmatter>"
})
```

To change a skill, call `call_recipe("skills.update", { slug: "<slug>", ... })` and pass only what changes. The body is sent back complete, because it replaces the whole instructions: load the skill with the `skill` tool and send its body back with the change. Never delete a skill and create it again to change it. To remove one, call `call_recipe("skills.delete", { slug: "<slug>" })`.

## Importing instead of writing

The person can also ask for a skill that already exists in a public repository. An imported skill is a set of instructions somebody else wrote that you will then follow, with the connectors this assistant already holds, so inspect it before you install it:

1. Run the security check; it is read-only and installs nothing: `call_recipe("skills.security_scan", {"git_url": "<public github/gitlab url>"})`
2. Say in one or two lines, in the person's language, what the check reported and what the skill asks you to do: which recipes it calls, and what it would send where.
3. Import only after the person answers: `call_recipe("skills.import", { git_url: "<public github/gitlab url>" })`

An `esito` of `Non disponibile` or `Non analizzata` means the check has no opinion, not that the skill is safe: say so and let the person decide with that stated.

## Failures

- 409: the slug is taken. A slug is unique across the whole platform, so the name may belong to a system skill, an organisation skill or another person's skill. If it is one of the person's own skills doing the same task, change it with `skills.update`; otherwise pick a distinct name, never a version suffix.
- 422: fix the field the platform names and retry once.
- 5xx: do not retry blindly. Tell the person and save the draft as the workspace file `pending-skill-<slug>.md`.

## Don't

- Do not paste the whole skill into the chat; the summary is enough.
- A fact about the person or a rule about how you work is not a skill: do not save either as one.
