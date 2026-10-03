# skill-creator-skill

A Cerase skill that lets a person manage their own skills through the
assistant: write and save a new `SKILL.md`, import one from a public repository,
list the ones they own, and delete them. The assistant uses it when someone
asks it to "always remember" a process, policy or template, to add a
capability, or to extend its behaviour with a lasting instruction. A preference
or a fact about the person ("call me Marco") goes to memory instead, and a
request that is only an import from a URL goes to `skill-installer`.

## What the assistant does

1. **Brief.** Works from what the person said, with at most one clarifying
   question.
2. **Compile.** Writes a `SKILL.md` in the agentskills.io format: frontmatter
   with `name` (a kebab-case slug of 3 to 64 characters), `description` (40 to
   1,024 characters, saying both what the skill does and when it applies) and
   `license` (default `Apache-2.0`), then a body of at least 80 characters with
   *When to activate*, *What to do* and *Don't* sections. The display name is
   the name the person used, verbatim.
3. **Confirm and save.** Shows the slug and a one-line description, and after a
   yes calls `skills.create` with `slug`, `display_name`, `description` and
   `body`. The platform binds the skill to the person; the assistant passes no
   identity.

To import instead of writing, the assistant first runs `skills.security_scan`
on the public GitHub or GitLab URL, which installs nothing, tells the person in
one or two lines what the scan reported and what the skill would make it do,
and calls `skills.import` only after the person answers. A scan with no result
is reported as such, never as safe. `skills.list` and `skills.delete` cover
the person's existing skills.

On a 409 (slug taken) it proposes a variant or a replacement; on a 422 it fixes
the field and retries once; on a server error it saves the draft as
`pending-skill-<slug>.md` in the workspace. It does not paste the whole
`SKILL.md` into the chat, and does not write a skill that relies on a tool the
assistant does not have.

## Requirements

The Cerase recipes `skills.create`, `skills.import`, `skills.list`,
`skills.delete` and `skills.security_scan`.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The instructions the assistant loads: `name` and `description` frontmatter, then the method. |
| `cerase.json` | Marketplace manifest: namespace `studio.guidance`, name `skill-creator`, display name, description, licence. |
| `LICENSE` | MIT licence text. |

## Installation

Published in the Cerase Marketplace as `studio.guidance/skill-creator`
([marketplace page](https://marketplace.cerase.ai/en/p/studio.guidance/skill-creator)).
A Cerase appliance also ships it in its image and attaches it to every
assistant; an administrator cannot detach it.

## License

MIT. See [LICENSE](LICENSE).
