# gha-template

A deliberately tiny, dependency-free Node.js app used as a sandbox for working through a GitHub Actions learning plan, one phase at a time.

## The app

- `GET /` → `{"message":"Hello, world!"}` (pass `?name=Ada` to personalise)
- `GET /health` → `{"status":"ok"}`

```bash
npm start      # http://localhost:3000
npm test       # node's built-in test runner, no dependencies
```

Requires Node 22+.

## Progress

### Phase 1 — Anatomy of a Workflow ✅

Workflow: [`.github/workflows/ci.yml`](.github/workflows/ci.yml) — two parallel jobs on every push to `main` and every pull request: `test` (`npm ci` + `npm test`) and `lint` (`node --check` on every file).

Exercises:

- [x] 1. Push and watch it run in the Actions tab
- [x] 2. Break a test deliberately and read the failure annotation ([#1](https://github.com/robjcook/gha-template/pull/1))
- [x] 3. Add `- run: echo "Branch is ${{ github.ref_name }}"` and inspect the log ([#1](https://github.com/robjcook/gha-template/pull/1))
- [x] 4. Add a second job and confirm from timestamps it ran in parallel ([#2](https://github.com/robjcook/gha-template/pull/2))

Deliverable: every PR gets an automatic check, and you can explain why removing `checkout` breaks it.

Takeaways:

- **Failures:** GitHub only sees a step's exit code. `node --test` doesn't emit `::error file=…,line=…::` workflow commands, so the only annotation was a generic "Process completed with exit code 1" — line-level annotations need a reporter or problem matcher. A failed job also skips setup-node's post step, so no npm cache is saved.
- **Contexts:** on a PR run `ref_name` is `1/merge` and `ref` is `refs/pull/1/merge` (GitHub tests a merge commit of branch + base); `head_ref`/`base_ref` are set only on PR events. On a push to `main`, `ref_name` is `main` and both are empty. `${{ }}` is substituted into the script text *before* the shell runs — the root of script injection (Phase 7).
- **Jobs:** with no `needs:`, `test` and `lint` started in the same second on different runners. Each job is a fresh VM (hence `checkout` again) and bills rounded up to a whole minute — ~11 s of work billed as 2 minutes.
- **Why removing `checkout` breaks it:** the runner workspace starts empty. Context echoes still work (they come from the event payload), but `npm ci` has no `package.json`/lockfile to read.

### Phase 2 — Syntax, Triggers, Contexts, and Expressions 🚧

Goal: read any workflow file in the wild and know what it does. Deliverable: one workflow using path filters, `workflow_dispatch` with a typed input, a conditional job, a step output consumed by a later job, and a step summary.

Exercises:

- [ ] 1. **Path filters** — `paths-ignore: ['**.md']` on CI, then open a docs-only PR and confirm no run starts
- [ ] 2. **Manual trigger** — `workflow_dispatch` with a typed `choice` input, run it from the Actions tab and from `gh workflow run`
- [ ] 3. **Step output → job output** — a `meta` job computes the version with `$GITHUB_OUTPUT`; a later job reads it via `needs.meta.outputs`
- [ ] 4. **Conditional job** — a fake `deploy` job that runs only on a push to `main` or a manual run, skipped on PRs
- [ ] 5. **Step summary** — write a Markdown table to `$GITHUB_STEP_SUMMARY` and find it on the run page
- [ ] 6. **Variables** — compare `env:`, a repo configuration variable (`vars.*`) and a default variable (`$GITHUB_SHA`)
