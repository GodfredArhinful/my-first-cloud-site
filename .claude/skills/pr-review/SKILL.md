---
name: pr-review
description: Review a pull request (or the current branch's changes) line by line, flag problems by severity, and suggest a concrete better alternative for each one. Use when asked to review a PR, check a diff, look over changes before merging, or give feedback on code. Accepts a PR number, a branch name, or nothing (reviews the current branch against main). Add "--post" to publish the review as inline comments on GitHub.
---

# PR Review

Review changes closely, flag every real problem, and for each one say **what to do instead**. A flag without a suggestion is not finished.

## 1. Get the changes

Pick the target from the arguments:

- **PR number** (e.g. `12`): fetch the PR with the GitHub MCP tools (`mcp__github__pull_request_read` for details, diff, and files) or the `gh` CLI if that is what's available. Also read the PR title and description so you know what it is *meant* to do.
- **Branch name**: `git fetch origin <branch>` then `git diff origin/main...origin/<branch>`.
- **Nothing**: review the current branch: `git diff origin/main...HEAD` plus any uncommitted changes (`git diff HEAD`).

If the diff is empty, say so and stop.

## 2. Read for context, not just the diff

For every changed file, open the **whole file**, not only the changed lines. Many bugs are only visible in context: a renamed id still used elsewhere, a link to a file that was deleted, a style that another page depends on. Use `grep` to find other places that reference anything the PR renames, moves, or deletes.

## 3. Check closely

Go through each changed line and ask the questions below. Only flag what you can point to in the code; don't invent problems.

**Correctness**
- Does it do what the PR description says? Anything missing?
- Broken links, wrong file paths, wrong case in file names (GitHub Pages is case-sensitive: `About.html` ≠ `about.html`).
- JavaScript: undefined variables, typos in ids/classes used by `getElementById`/`querySelector`, unhandled errors, `==` where `===` is meant, off-by-one loops.
- Unclosed or wrongly nested HTML tags, duplicate `id` attributes.

**Security**
- Secrets, API keys, tokens, or passwords committed anywhere (including comments and config files). This is always 🔴.
- `innerHTML` / `document.write` with user input (XSS) — suggest `textContent` or building elements.
- External scripts loaded over `http://` or from untrusted CDNs; `target="_blank"` links without `rel="noopener noreferrer"`.

**Accessibility**
- Images without meaningful `alt` text; buttons/links with no readable label.
- Missing `<meta name="viewport" ...>` (page will look tiny on phones).
- Skipped heading levels, poor color contrast, form inputs without `<label>`.

**Performance**
- Large unoptimized images, scripts that block rendering (prefer `defer`), duplicate libraries.

**Maintainability**
- Repeated code that should be shared (e.g. inline styles copied across pages → a shared CSS file).
- Unclear names, leftover debug code (`console.log`, commented-out blocks), dead files.
- Changes unrelated to the PR's purpose — suggest splitting them out.

## 4. Grade each finding

| Flag | Meaning | Blocks merge? |
|---|---|---|
| 🔴 **Must fix** | Bug, security issue, broken page/link, data loss | Yes |
| 🟡 **Should fix** | Accessibility gap, likely future bug, clear maintainability problem | Usually |
| 🔵 **Suggestion** | Nicer way to do it, style, small cleanup | No |
| 💚 **Nice work** | Something done well worth keeping | — |

Be honest about severity. Don't mark style preferences 🔴, and don't bury a real bug as 🔵.

## 5. Write each comment like this

```
🔴 Must fix — `index.html:14`
**Problem:** The image has no `alt` text, so screen readers announce only the file name.
**Why it matters:** Visitors using assistive tech can't tell what the image shows.
**Do this instead:**
<img src="cloud.png" alt="Diagram of a website served from the cloud">
```

Rules:
- Always give the file and line.
- Always include a **Do this instead** with real code or a precise step, not just "consider improving".
- One problem per comment. If the same problem repeats, comment once and list the other locations.
- Explain *why* in one or two plain sentences — the goal is that the author learns, not just fixes.

## 6. Output

Print the review in this shape:

```
## PR Review: <title or branch>

**Verdict:** ✅ Approve | 💬 Approve with suggestions | ❌ Request changes
**Summary:** <2–3 sentences: what the PR does and the overall state>

### 🔴 Must fix (n)
...
### 🟡 Should fix (n)
...
### 🔵 Suggestions (n)
...
### 💚 Nice work
...
```

Verdict rule: any 🔴 → **Request changes**; only 🟡/🔵 → **Approve with suggestions**; nothing found → **Approve**. Omit empty sections.

## 7. Posting to GitHub (only with `--post` or when explicitly asked)

Never post to GitHub unless asked. When asked:
1. Create a pending review: `mcp__github__pull_request_review_write` with method `create`.
2. Add one inline comment per finding on its line with `mcp__github__add_comment_to_pending_review`. For small, exact fixes, use a GitHub suggestion block so the author can apply it with one click:
   ````
   ```suggestion
   <img src="cloud.png" alt="Diagram of a website served from the cloud">
   ```
   ````
3. Submit with method `submit_pending`, using the summary as the body and the event that matches the verdict (`REQUEST_CHANGES`, `COMMENT`, or `APPROVE`). You cannot request changes on your own PR — use `COMMENT` then.
4. Reply with the link to the review.
