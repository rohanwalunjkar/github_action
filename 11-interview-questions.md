# 11 — GitHub Actions Interview Questions (Brief)

Concise Q&A grouped by difficulty for quick revision.

---

## Basics

**1. What is GitHub Actions?**
GitHub's built-in **CI/CD and automation** platform that runs **event-driven** workflows defined in YAML inside `.github/workflows/`.

**2. What is a workflow?**
An automated process defined in a YAML file. A repo can have many; each is triggered by events.

**3. Explain the core hierarchy.**
**Event → Workflow → Jobs → Steps → Actions/Commands.**

**4. Where do workflow files live?**
In `.github/workflows/` with a `.yml`/`.yaml` extension.

**5. What is a job?**
A group of steps running on the **same runner**. Jobs run in **parallel** by default.

**6. What is a step?**
A single task in a job — either an **action** (`uses`) or a **shell command** (`run`). Steps run sequentially.

**7. What is an action?**
A reusable unit of code referenced with `uses: owner/repo@version` (e.g., `actions/checkout@v4`).

**8. What is a runner?**
A machine that executes a job — **GitHub-hosted** (ubuntu/windows/macos) or **self-hosted**.

**9. uses vs run?**
`uses` runs an **action**; `run` executes **shell commands**. A step uses one or the other, not both.

**10. What does actions/checkout do?**
Clones your repository into the runner so the workflow can access your code. Needed in almost every workflow.

---

## Events & Triggers

**11. How do you trigger a workflow?**
With the **`on`** key: `push`, `pull_request`, `schedule`, `workflow_dispatch`, etc.

**12. What is workflow_dispatch?**
A **manual** trigger that adds a "Run workflow" button and supports **inputs**.

**13. What is schedule?**
Runs workflows on a **cron** schedule (UTC).

**14. push vs pull_request events?**
`push` runs on commits pushed to branches/tags; `pull_request` runs on PR activity (open, sync, etc.).

**15. pull_request vs pull_request_target?**
`pull_request` runs in the PR branch context (no secrets for forks — safer). `pull_request_target` runs in the base context (has secrets — riskier).

**16. How do you filter which pushes trigger a run?**
Using `branches`, `branches-ignore`, `paths`, `paths-ignore`, `tags`.

**17. What is repository_dispatch?**
An event to trigger workflows from **external systems** via the REST API.

---

## Jobs, Steps & Data

**18. Do jobs run in parallel or sequentially?**
**Parallel** by default; use **`needs`** to make them sequential.

**19. What is `needs`?**
Declares job **dependencies** — a job waits for the listed jobs to finish.

**20. How do you pass data between steps?**
Write to **`$GITHUB_OUTPUT`** and read via `steps.<id>.outputs.<name>`.

**21. How do you pass data between jobs?**
Define job **`outputs`** and read them via `needs.<job>.outputs.<name>`.

**22. How do you persist files after a run or between jobs?**
Use **artifacts** (`upload-artifact` / `download-artifact`).

**23. Artifacts vs cache?**
**Artifacts** store results you can download (within a run). **Cache** speeds up runs by reusing dependencies **across runs**.

**24. How does caching work?**
`actions/cache` restores files by a **key** (often a lockfile hash); saves them at job end if the key is new.

**25. What is `$GITHUB_ENV`?**
Writing to it sets an env var available to **subsequent steps** in the same job.

---

## Variables, Secrets & Contexts

**26. env vs vars vs secrets?**
`env` = non-secret variables; `vars` = non-secret config from UI (`${{ vars.X }}`); `secrets` = encrypted, auto-masked (`${{ secrets.X }}`).

**27. What is GITHUB_TOKEN?**
An automatically created token for each run to interact with the repo. Scope it with **`permissions`**.

**28. Are secrets available to forked PR workflows?**
**No** — secrets are not passed to workflows triggered by forks (security).

**29. What are contexts?**
Objects with run data accessed via `${{ }}` — e.g., `github`, `env`, `secrets`, `vars`, `needs`, `steps`, `matrix`, `runner`, `inputs`.

**30. Name useful github context fields.**
`github.ref`, `github.sha`, `github.actor`, `github.event_name`, `github.repository`, `github.workspace`.

**31. env var precedence?**
**Step > Job > Workflow.**

---

## Expressions, Conditionals, Matrix

**32. What is `${{ }}`?**
Expression syntax to read contexts and compute values.

**33. What is `if`?**
A conditional that controls whether a job/step runs.

**34. What are status check functions?**
`success()`, `failure()`, `always()`, `cancelled()` — used in `if` to control based on prior results.

**35. What is a matrix strategy?**
Runs the same job across **multiple combinations** (OS, versions) in parallel.

**36. What is fail-fast in a matrix?**
If true (default), one failing matrix job **cancels the rest**. Set `false` to let all finish.

**37. include vs exclude in a matrix?**
`include` adds extra combos/variables; `exclude` removes specific combos.

**38. Useful expression functions?**
`contains`, `startsWith`, `endsWith`, `format`, `join`, `toJSON`, `fromJSON`, `hashFiles`.

---

## Reuse & Custom Actions

**39. What is a reusable workflow?**
A workflow callable from others via **`workflow_call`**, used at the **job** level with `uses`.

**40. What is a composite action?**
An action that bundles **multiple steps** (`using: composite`), used at the **step** level.

**41. Reusable workflow vs composite action?**
Reusable workflow reuses **whole jobs** (job level); composite action reuses **steps** (step level).

**42. Types of custom actions?**
**JavaScript**, **Docker container**, and **Composite** actions.

**43. What file defines a custom action?**
**`action.yml`** (metadata: name, inputs, outputs, runs).

**44. Action vs reusable workflow?**
An action is a single **step**; a reusable workflow is an entire **workflow/jobs**.

**45. What is `secrets: inherit`?**
Passes **all** caller secrets to a reusable workflow automatically.

---

## Security & Best Practices

**46. How do you secure GITHUB_TOKEN?**
Use **least-privilege `permissions`** (default `contents: read`, widen per job).

**47. Why pin actions to a SHA?**
Tags are **mutable** and can be hijacked; a **commit SHA is immutable** and safer.

**48. How do you prevent script injection?**
Pass untrusted input (PR titles, etc.) via **env vars**, not inline `${{ }}` in `run`.

**49. What are environments?**
Named deploy targets (`staging`, `production`) with **protection rules** (approvals, wait timers) and scoped secrets.

**50. What is OIDC in Actions?**
Lets workflows get **short-lived cloud credentials** without storing long-lived keys (`id-token: write`).

**51. What is concurrency?**
Cancels/limits duplicate runs in the same group (`cancel-in-progress: true`).

**52. How do you debug a workflow?**
Enable `ACTIONS_STEP_DEBUG`, use `::debug::`/`::warning::` workflow commands, `continue-on-error`, and re-run with logging.

---

## Scenario / Practical

**53. Run a job only on the main branch?**
`if: github.ref == 'refs/heads/main'`.

**54. Deploy only after tests pass?**
Add `needs: [test]` to the deploy job.

**55. Test on Node 18 and 20 across Ubuntu and Windows?**
Use a **matrix** with `os` and `node` arrays.

**56. Skip CI for doc-only changes?**
Use `paths-ignore: ['**.md']` or check `[skip ci]` in the commit message.

**57. Store an AWS key securely?**
Use **secrets** (or better, **OIDC** for keyless auth).

**58. Cancel old runs when pushing rapidly?**
Use `concurrency` with `cancel-in-progress: true`.

**59. Share build output from build job to deploy job?**
Upload as an **artifact**, then **download** it in the deploy job.

**60. Require manual approval before production deploy?**
Use an **environment** with **required reviewers**.

---

## Rapid-Fire One-Liners

| Question | Answer |
|----------|--------|
| Workflow location? | `.github/workflows/` |
| Required top-level keys? | `on` and `jobs` |
| Default job execution? | Parallel |
| Force job order? | `needs` |
| Get your code? | `actions/checkout` |
| Manual trigger? | `workflow_dispatch` |
| Cron trigger? | `schedule` |
| Read a secret? | `${{ secrets.NAME }}` |
| Step output file? | `$GITHUB_OUTPUT` |
| Env for later steps? | `$GITHUB_ENV` |
| Reuse whole jobs? | Reusable workflow (`workflow_call`) |
| Reuse steps? | Composite action |
| Cross-version testing? | Matrix |
| Cancel duplicate runs? | `concurrency` |
| Least-privilege token? | `permissions` |
| Keyless cloud auth? | OIDC (`id-token: write`) |

---

## Key Takeaways

- Nail the **core hierarchy** and **event → runner** execution flow.
- Be crisp on **jobs/needs, artifacts vs cache, secrets, contexts, matrix**.
- Know **reusable workflows vs composite actions** and the **3 custom action types**.
- Have **security answers** ready: least-privilege, SHA pinning, injection prevention, environments, OIDC.
