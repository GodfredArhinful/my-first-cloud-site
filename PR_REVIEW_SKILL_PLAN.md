# PR Review Skill — Plan (React JS)

> **Status:** Draft for review. Nothing will be built from this until it's approved.

## Goal

A skill that reviews React JS pull requests that touch many files. It reads the changes closely, flags problems by severity, and suggests a concrete better alternative for each one.

## How to use it

| Command | What it reviews |
|---|---|
| `/pr-review 12` | Pull request #12 |
| `/pr-review feature-branch` | A branch compared with `main` |
| `/pr-review` | The current branch compared with `main` |
| `/pr-review 12 --post` | PR #12, and posts the review as comments on the exact lines on GitHub |

Without `--post`, the review only appears in chat.

---

## Step 1: Get the big picture

- Read the PR title and description to understand what the change is meant to do.
- List every changed file and mark it as **added**, **modified**, **deleted** or **renamed**, with line counts.
- Sort the files by role:

| Group | Examples |
|---|---|
| Hooks & utilities | `useAuth.js`, `formatDate.js` |
| Context & state | `AuthContext.jsx`, Redux slices, Zustand stores |
| API / services | `api/users.js`, fetch and axios wrappers |
| Shared components | `Button.jsx`, `Modal.jsx` |
| Pages & routes | `pages/Dashboard.jsx`, router config |
| Styles | `.css`, `.module.css`, styled-components |
| Tests | `*.test.jsx`, `*.spec.js` |
| Config | `package.json`, `vite.config.js`, `.env*`, ESLint config |

## Step 2: Review in dependency order

Review the code that other files depend on first, then the components that use it:

1. Hooks, utilities, context, API files
2. Shared components
3. Pages and routes
4. Styles, tests, config

Read each changed file **in full**, not only the changed lines. Many bugs only show up in context.

## Step 3: React-specific checks for each file

| Area | What to look for |
|---|---|
| **Hooks** | Missing or wrong `useEffect` dependencies · hooks called inside `if`/loops · effects with no cleanup (timers, subscriptions, event listeners) · stale values captured in callbacks |
| **State** | Changing state directly (`state.push(...)`) · keeping a copy of a value that could be calculated · updating from old state without `setX(prev => ...)` · state kept higher up than needed |
| **Rendering** | Missing or index-based `key` in lists · unnecessary re-renders · new objects/functions passed as props every render where it matters · `useMemo`/`useCallback` that do nothing |
| **Data fetching** | No loading or error state · late or out-of-order responses overwriting newer data · fetching during render · unhandled `async` errors |
| **Security** | `dangerouslySetInnerHTML` with user data · API keys or secrets in frontend code or committed `.env` files · unchecked user URLs in `href` |
| **Accessibility** | `onClick` on a `<div>` instead of `<button>` · images without `alt` · inputs without labels · things a keyboard user can't reach |
| **Structure** | Components that are too big and should be split · logic copied between components that belongs in a custom hook · props passed down through many layers (prop drilling) |
| **Code health** | Leftover `console.log` · unused imports/variables · ESLint `react-hooks` warnings |
| **Dependencies** | New packages that aren't needed · `package.json` changed without updating the lock file |

## Step 4: Check the files against each other

Some problems only show up when files are compared:

- A component's **props changed**: are all the places that use it updated?
- A **hook, context or API response changed shape**: is everything that uses it updated?
- **Imports** pointing to files that were renamed or deleted.
- **Behavior changed**: were the tests updated to match?
- The **same logic copied** into several files instead of shared.

## Step 5: Flag and suggest

### Severity flags

| Flag | Meaning | Blocks merge? |
|---|---|---|
| 🔴 **Must fix** | Bug, crash, infinite loop, security issue, broken feature | Yes |
| 🟡 **Should fix** | Accessibility gap, likely future bug, clear maintainability problem | Usually |
| 🔵 **Suggestion** | Cleaner way to do it, style, small cleanup | No |
| 💚 **Nice work** | Something done well worth keeping | — |

### Comment format

Every comment must include the file, the line, the problem, why it matters, and **what to do instead**.

````
🔴 Must fix — src/components/UserList.jsx:18
**Problem:** useEffect fetches users but has no dependency array.
**Why it matters:** It runs after every render, re-fetches, sets state, and loops forever.
**Do this instead:**
```jsx
useEffect(() => {
  fetchUsers().then(setUsers);
}, []);
```
````

### Rules
- One problem per comment. If the same problem repeats, write **one comment** and list the other locations.
- Only flag what can be pointed to in the code. Don't guess.
- Be honest about severity: don't mark style preferences 🔴, and don't bury real bugs as 🔵.
- Explain *why* in plain language, so the author learns and doesn't just fix.

## Step 6: Report

```
## PR Review: <PR title>

**Verdict:** ✅ Approve | 💬 Approve with suggestions | ❌ Request changes
**Summary:** <2–3 sentences on what the PR does and its overall state>

| File | Status | 🔴 | 🟡 | 🔵 |
|---|---|---|---|---|
| src/hooks/useUsers.js | modified | 1 | 0 | 1 |
| src/components/UserList.jsx | added | 0 | 2 | 0 |

### 🔴 Must fix (n)
### 🟡 Should fix (n)
### 🔵 Suggestions (n)
### 💚 Nice work
```

**Verdict rule:** any 🔴 → Request changes · only 🟡/🔵 → Approve with suggestions · nothing found → Approve.

### Posting to GitHub (only with `--post`)
1. Create a pending review.
2. Add one comment per finding on its exact line. Small fixes use GitHub's `suggestion` format so the author can apply them in one click.
3. Submit it as a single review with the summary and the matching verdict.

## Large PRs (about 15+ files)

Helper agents review groups of files at the same time (for example, one for hooks and state and one for components). I then do the **cross-file check (Step 4) myself**, because no single helper sees every file.

---

## Open questions

1. **Which repo?** `my-first-cloud-site` only has `index.html`. Where is the React project?
2. **JavaScript or TypeScript?** If TypeScript, type checks get added to Step 3.
3. **Project tools?** ESLint / Prettier / Jest / Vitest. If present, the review can run them as well.
4. **Existing skill file** on the branch: rewrite it with this plan, or delete it and start fresh?
