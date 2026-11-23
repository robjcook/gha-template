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

### Phase 1 — Anatomy of a Workflow 🚧

Workflow: [`.github/workflows/ci.yml`](.github/workflows/ci.yml) — runs `npm ci` and `npm test` on every push to `main` and every pull request.

Exercises:

- [ ] 1. Push and watch it run in the Actions tab
- [ ] 2. Break a test deliberately and read the failure annotation
- [ ] 3. Add `- run: echo "Branch is ${{ github.ref_name }}"` and inspect the log
- [ ] 4. Add a second job and confirm from timestamps it ran in parallel

Deliverable: every PR gets an automatic check, and you can explain why removing `checkout` breaks it.
