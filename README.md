# gha-slash-commands

ChatOps in an issue or pull request: a collaborator comments `/label bug`, `/assign-me` or `/help` and a workflow acts on it. The interesting part is doing it **safely**, because anyone can comment on a public repository.

## What ran (2026-10-16)

On a test issue (#1):

| Comment | Result |
|---|---|
| `/label bug` | label `bug` added, 👍 reaction |
| `/label hacker-label` | rejected: reply listing allowed labels, 😕 reaction (**not** added) |
| `/assign-me` | commenter assigned, 👍 reaction |
| `/nope` | reply "Unknown command. Try `/help`", 😕 reaction |
| `/help` | reply listing the commands |
| a normal comment with no `/` | workflow run **skipped** |

## The safety rules

- **Only OWNER, MEMBER and COLLABORATOR** comments run (`author_association`), checked in the job's `if`. Anyone else's comment is skipped.
- **The comment text goes through `env:`**, never into the script: a comment is attacker-controlled input.
- **An allow-list of labels**, so a command cannot create or apply arbitrary labels.
- **Least privilege:** `permissions: {}` at the top; only `issues: write` and `pull-requests: write` on the job.
- The workflow does **not** check out any code, which is what makes `issue_comment` triggers much safer than running pull-request code.

## Not tested

- A comment from someone who is **not** a collaborator (I cannot impersonate another account), so that "skipped" path is by design and by reading the condition, not observed.
- zizmor flags `issue_comment` as a "dangerous trigger"; it is used deliberately here, with the guards above.
- Rate limits, spam, and commands on pull requests (it uses the same event).

## A mistake I made while testing

My first test comments were mangled by my own shell (Git Bash rewrote `/label bug` into a Windows path), so they did not start with `/` and the workflow correctly skipped them. I deleted them and re-posted correctly. A reminder that "skipped" can mean the filter worked.

MIT licensed.
