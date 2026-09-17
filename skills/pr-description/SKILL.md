---
name: pr-description

description: Proactively apply when creating a pull request or merge request title and description from a ticket, branch, diff, commits, or screenshots. Triggers on PR description, pull request, merge request, MR, PR title, What changed, Why did it change, How did it change, Jira ticket, Closes issue. Produces a concise, concrete description in the user's style without inventing context.
---

# PR / MR description

Create a title and a ready-to-paste Markdown description from the available ticket, branch, diff, commits, tests, and visual material.

The goal is not to document every implementation detail. Explain the ticket's context, the behavior that changed, why it changed, and only the useful outline of the solution. A screenshot or short video is preferred when the change is visible in the UI.

## Use when

| Use when | Skip when |
| --- | --- |
| Preparing a PR or GitLab merge request | Writing a technical design document |
| Turning a ticket and a diff into a reviewable description | Writing a changelog or release note |
| Summarizing several commits without repeating them | The user only asks for a commit message |

## Workflow

1. Read the ticket, issue, branch name, and user-provided context.
2. Inspect the repository when available:
   - `git status`
   - `git diff --stat`
   - `git diff`
   - `git log` for the relevant commits
   - the repository's PR/MR template, if one exists
3. Identify:
   - the main intent: feature or bug fix;
   - the domain or module for the title scope;
   - the user-visible or business-visible result;
   - the reason for the change;
   - the Jira key, Jira URL, and issue number;
   - available screenshots, recordings, or GIFs;
   - tests run and manual checks actually performed.
4. Write the output in the language used by the repository or ticket. If the examples and repository use English, keep the PR in English.
5. Return only the title followed by the Markdown description, unless the user asks for an explanation.

### What information should be used?

```
Available information?
├─ Ticket and diff are available       → explain context, result, and solution
├─ Only commits are available          → use commit intent; do not invent motivation
├─ Only a ticket is available          → draft the structure and mark missing facts
└─ A screen/video is available         → place it near What changed?
```

Never present an assumption as a fact. Use `[À compléter : ...]` for missing information rather than silently guessing.

## Title

Use one of these forms:

```text
feat(domain): short description of the new behavior
fix(domain): short description of the corrected behavior
```

Rules:

- Use `feat` when the main change adds or changes a capability.
- Use `fix` when the main change corrects an existing behavior.
- Derive `domain` from the ticket or the touched module. Do not invent a domain; use `[domain]` when it cannot be established.
- Keep the description short, concrete, and written as a result, not as an implementation detail.
- If a branch contains both a feature and a small fix, title it after the main ticket intent and mention the fix in the body.

❌ `feat: update files`

✅ `feat(domain): add in-game tutorials`

❌ `fix(domain): refactor tutorial engine and update several services`

✅ `fix(domain): prevent tutorials from showing at the wrong time`

## Description structure

Use these headings and this order:

```markdown
# What changed ?

[One short paragraph giving the context and the resulting behavior.]

- [feature or behavior 1]
- [feature or behavior 2]
- [bug fix and its observable result]

[Add a screenshot, recording, or GIF here when the change is visual.]

Closes [CTB-XXXX](link to Jira ticket)

# Why did it change ?

- [feature 1] was added because [reason]
- [feature 2] was added because [reason]
- [bug fix] was triggered by [cause] and provoked [impact]

Closes #issue

# How did it change ?

%{all_commits}

# Checklist:

- [ ] I have performed a self-review of my code
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing tests pass locally with my changes
- [ ] I have tested manually the app locally to check for any uncaught bugs
```

### What changed ?

Start with the problem or context, then describe what the user or business can now do. Prefer a short paragraph followed by a few bullets. Mention the important behavior and its boundaries, not every class, method, or file.

Good descriptions answer: “What is different after this change?”

❌

```markdown
We changed the TutorialService, added several handlers and updated the frontend.
```

✅

```markdown
Implements in-game tutorials.
A tutorial is an ordered script of steps: the backend defines what is taught and where the player is in it, while the frontend displays the current step and reports when it is completed.
```

When the change is visual, add the supplied media directly below the explanation:

```markdown
![tutorial](/uploads/path/to/screenshot-or-recording.mp4)
```

Do not fabricate upload paths. If no media is available, add a short placeholder only when a visual proof would materially help the review:

```markdown
[À compléter : ajouter un screen ou une vidéo si disponible]
```

### Why did it change ?

Explain the ticket's motivation, not the implementation. Connect each important change to a user, product, or technical problem. Keep the cause-and-effect relationship explicit.

❌ `Improved the architecture and made the code cleaner.`

✅ `The tutorial was added to improve onboarding for new players and give experienced players a way to rehearse the basics.`

If the reason is unknown, write `[À compléter : raison du changement]`. Do not infer a business reason from a class name or a commit message alone.

### How did it change ?

Keep the literal placeholder `%{all_commits}` when the repository uses GitLab's automatic commit expansion. Do not copy the full commit list into the description or turn this section into an implementation essay.

If the project does not support that placeholder, summarize the implementation in at most three bullets and keep them at the level of the main mechanisms.

### Links

- Add the Jira link as `Closes [CTB-XXXX](URL)` when both key and URL are known.
- Add `Closes #issue` when an issue number is known.
- Do not invent Jira URLs or issue numbers.
- If the ticket key is known but its URL is not, use `Closes CTB-XXXX` rather than inventing a link.
- Keep the closing references near the section they explain, as in the template above.

## Checklist

Use the four default checks above. Mark a checkbox only when the available evidence proves it was done:

- `[x]` for tests or manual checks actually run;
- `[ ]` when they are still to be done or cannot be verified.

Preserve additional project-specific checklist items from the repository's PR/MR template. Do not add generic checklist items just to make the list longer.

For example, project-specific items may include Doctrine defaults, migration rules, or configuration registration. Keep them only when they come from the project template or the ticket.

## Style rules

- Direct, concrete, and calm.
- Give context before mechanics.
- Prefer one short paragraph and a few bullets over a wall of text.
- Describe the main mechanism without exposing internal details that do not help review.
- Keep the user's vocabulary when it is clear and useful.
- Avoid marketing language, exaggerated claims, and generic sentences such as “this improves the overall quality”.
- Do not add a summary section if the three requested sections already explain the change.
- Do not mention files or classes unless they help understand the solution.
- Do not claim that tests passed, the app was launched, or a screen exists unless that is known.

## Decision tree: how much detail?

```
What is being reviewed?
├─ User-visible feature      → context, new behavior, trigger conditions, screenshot if possible
├─ Bug fix                   → observed cause, resulting impact, and corrected behavior
├─ Refactor with no behavior change → why it was needed and what was kept unchanged
└─ Mixed change              → lead with the ticket's main intent; mention secondary changes briefly
```

## Decision tree: when to ask instead of guessing?

```
Is a required fact missing?
├─ It can be verified in the repo or git history → inspect it
├─ It is a link, ticket key, test result, or user-facing reason → use a placeholder
├─ It changes the PR type or title scope       → ask the user if a reliable default is impossible
└─ It is a minor wording detail                → choose the shortest neutral wording
```

## Final check

Before returning the description:

- [ ] Title uses `feat(domain):` or `fix(domain):`.
- [ ] What changed explains the context and resulting behavior.
- [ ] Why did it change explains the motivation or cause and impact.
- [ ] How did it change contains `%{all_commits}` when supported.
- [ ] Jira and issue references are present only when known.
- [ ] Screens or recordings are included when available and relevant.
- [ ] Checklist states only what can be verified.
- [ ] No implementation detail, link, reason, or test result was invented.
