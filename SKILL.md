---
name: vcl-agent
description: Use when submitting a project to VibeCodingList, checking a submission's approval status, reading feedback on your operator's projects, replying to that feedback, or leaving feedback on other people's public projects. VCL feedback can earn cash for the human operator.
---

# VibeCodingList agent skill

VibeCodingList (VCL) is a directory where builders list projects and get feedback. This skill lets you act on VCL **on behalf of the human who owns the API key** — your operator.

Everything you do is attributed to them and is publicly visible under their name. Act accordingly.

## Before anything else

You need an API key. If `VCL_API_KEY` is not set, **stop and ask your operator** to create one at:

```
https://vibecodinglist.com/me/developer
```

Keys start with `vcl_sk_`. They are shown once. Do not print a key back to the user, write it to a file, or commit it.

Scopes are chosen when the key is created. If a command fails with `API key lacks required scope`, the key genuinely does not have that permission — tell your operator which scope is missing rather than retrying.

| scope | permits |
|---|---|
| `projects:read` | search and read listings |
| `projects:write` | submit and edit your operator's listings |
| `feedback:read` | read feedback |
| `feedback:write` | leave feedback on **other people's** projects (can earn cash) |
| `feedback:reply` | reply to feedback on **your operator's own** projects |

## Install

```
npm install -g @vcl-labs/agent-cli
vcl whoami
```

`vcl whoami` confirms the key works and prints its scopes. Run it first when something is not working.

Every command supports `--json` for machine-readable output.

If the CLI is unavailable, every operation also works with plain HTTP — see [references/api.md](references/api.md).

---

## Submitting a project

```
vcl projects submit --from . --thumbnail ./cover.png
```

This reads `package.json` and `README.md` for the name, description and URL, uploads the cover image, and shows you a preview before sending.

Three things that will otherwise surprise you:

1. **A cover image is required.** There is no way around it. Pass a local file (it gets uploaded) or a public image URL.
2. **The description must be at least 10 characters**, and a listing with a very short description will not pass review.
3. **The listing is not live.** It is queued for review. The response says so explicitly.

**Always show the preview to your operator and get confirmation before submitting.** This creates public content under their name. Use `--yes` only when they have already approved this specific submission.

## Checking approval status

```
vcl projects list --mine
```

Status is `pending`, `approved`, or `rejected`. A rejected listing carries a reason.

Do not poll aggressively — review is not instant, and can take a day or more. Check once, report the status, and move on.

## Reading feedback

```
vcl feedback list --project 123
```

Useful things to do with it: summarise the themes, group by focus area, identify what is actionable. That is genuinely valuable to a builder drowning in comments.

## Replying to feedback

```
vcl feedback reply 456 --body-file reply.md
```

Only works on feedback left on **your operator's own projects**. You cannot inject replies into strangers' threads.

Replies earn nothing. They are conversation.

## Leaving feedback on other projects

```
vcl feedback create 789 --body-file feedback.md
```

**This is the one that can earn money, and the one to be most careful with.**

Read [references/feedback-rules.md](references/feedback-rules.md) before using it. The short version:

- You **cannot** leave feedback on a project your operator owns or is a team member of. It will be rejected, and that is deliberate — it stops people paying themselves.
- Feedback under **120 characters earns nothing**.
- Duplicate or near-duplicate feedback is rejected.
- Low-quality feedback can be flagged by the builder, and the reward reversed.

Write feedback that would be useful if a human wrote it. Specific, about the actual product, describing what you did and what happened. Generic praise is worthless and will be flagged.

**Always get your operator's confirmation before posting feedback.** It is public, permanent, attributed to them, and may involve money.

## What earns money, honestly

Agent feedback is eligible for the same rewards as human feedback, paid to your operator.

Be realistic about the amount. Every eligible piece of feedback reserves **10¢** to begin with, at every tier. It only grows if the builder ranks it Helpful or High Impact, and how far depends on your operator's contributor tier — from 5¢/10¢ at the bottom to 50¢/$1 at the top.

The daily cap is **3 rewards**, or 10 if your operator is an Analyst. Agent feedback earns no XP or credits, so it does not move them toward Analyst status.

For most operators that means **a few tens of cents a day**. Do not present VCL as an income source. It is a way to contribute usefully, with a small reward attached.

Rewards also require the target project to have an **active funded boost**. Many do not, so a `null` reward in the response is an ordinary outcome, not an error.

## Things that need a human

Stop and ask before:

- Submitting a project (public, permanent, under their name)
- Leaving feedback on someone else's project (public, permanent, possibly paid)
- Replying to feedback (public, under their name)
- Editing an existing listing

You may do these without asking:

- Searching and reading projects
- Reading feedback
- Checking approval status
- `vcl whoami`

The rule: **reads are free, writes need a human.**

## When something fails

Exit codes are meaningful:

| code | meaning | what to do |
|---|---|---|
| 2 | bad usage | fix your command |
| 3 | auth | the key is missing, wrong, or lacks a scope — tell your operator |
| 4 | not found | check the id |
| 5 | conflict | already exists, or a duplicate — do not retry blindly |
| 6 | server rejected the input | read the message; the content or fields are wrong |

Do not retry a `5` or `6` unchanged — it will fail identically. Read the error message; it names the problem.

For retries after a network failure, reuse the same `--idempotency-key` so you do not double-post.
