# skill-creator-skill

A Cerase skill that lets a person manage their own skills through the
assistant: write and save a new one, change one in place, import one from a
public repository, and delete one. In Italian a skill is called «competenza».
The assistant uses it when the person asks to save or create a skill, and also
when the person has walked it through a task in several steps, asks again for a
task it already did, or describes a routine: in those three cases it offers,
once, to save the task, and creates nothing until the person agrees. A rule or
preference for how the assistant works and a fact about the person are not
skills, and a request that is only an import from a URL goes to
`skill-installer`.

## What the assistant does

1. **List first.** Calls `skills.list` and, if one of the person's skills
   already does the same task, changes it with `skills.update` instead of
   creating a second one.
2. **Write.** Writes the skill in English unless the person asks for another
   language, with a short third-person `description` that says what the skill
   does and when to load it, and a body that holds only the steps. The skill
   names no person, account, mailbox or address, names only recipes that exist
   by their exact call, and carries no time or day: those stay in the scheduled
   message that names the skill. The `display_name` is the person's own name
   for it, verbatim.
3. **Save.** Shows the person a short summary and calls `skills.create` with
   `slug`, `display_name`, `description` and `body`. The platform asks the
   person to approve the call, and that approval is the confirmation. The
   platform binds the skill to the person; the assistant passes no identity.

To change a skill it calls `skills.update` with only what changes; it never
deletes a skill and creates it again. To import instead of writing, the
assistant first runs `skills.security_scan` on the public GitHub or GitLab URL,
which installs nothing, tells the person in one or two lines what the scan
reported and what the skill would make it do, and calls `skills.import` only
after the person answers. A scan with no result is reported as such, never as
safe.

On a 409 (slug taken) it changes the person's own skill if that one does the
same task, or picks a distinct name; on a 422 it fixes the field and retries
once; on a server error it saves the draft as `pending-skill-<slug>.md` in the
workspace. It does not paste the whole skill into the chat.

## Requirements

The Cerase recipes `skills.create`, `skills.update`, `skills.import`,
`skills.list`, `skills.delete` and `skills.security_scan`.

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
