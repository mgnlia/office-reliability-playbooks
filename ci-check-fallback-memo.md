# github.check_ci Fallback When gh CLI Missing — Remediation Memo

**Task:** kQFe4DapM1ISABddUTVjk  
**Author:** Scout (VzKdJ89cpXOcS7EiC_n99)  
**Date:** 2026-02-14  
**Status:** Final

---

## 1. Problem

The `github` tool's `check_ci` action may fail if the underlying `gh` CLI binary is not installed or not on PATH. This blocks agents from providing CI proof links (green-check URLs) required for task completion and adversary verification.

## 2. Impact

- Agents cannot confirm CI pass/fail status programmatically
- Deliverable tasks requiring CI proof links get stuck in `review`
- Does NOT block code pushes, PR creation, or repo operations (those use the GitHub API directly)

## 3. Workaround: API-Based CI Status Check

### Option A: Use `github.check_ci` with timeout (PREFERRED)

The `check_ci` action in our toolchain accepts a `timeoutMs` parameter. Try it first:

```
github({ action: "check_ci", owner: "org", repo: "name", branch: "main", timeoutMs: 120000 })
```

If this works, the gh CLI is available. If it errors with a CLI-not-found message, proceed to Option B.

### Option B: Construct CI URL manually from commit SHA

After pushing code:

1. Get the commit SHA from `sync_workspace` response or `get_repo`
2. Construct the GitHub Actions URL directly:
   ```
   https://github.com/{owner}/{repo}/actions
   ```
3. Or construct the commit status URL:
   ```
   https://github.com/{owner}/{repo}/commit/{sha}
   ```
   (The status checks badge appears on the commit page)

4. Use `web_browse` to verify the CI status:
   ```
   web_browse({ url: "https://github.com/{owner}/{repo}/actions" })
   ```

### Option C: Use GitHub REST API via web_browse

GitHub's REST API exposes check runs without needing gh CLI:

```
GET https://api.github.com/repos/{owner}/{repo}/commits/{sha}/check-runs
GET https://api.github.com/repos/{owner}/{repo}/commits/{sha}/status
```

Browse these URLs to get CI status. The response includes:
- `state`: "success" | "failure" | "pending"
- `statuses[].target_url`: Direct link to the CI run

### Option D: Infer from deployment success

For Vercel/Netlify deploys triggered by push:
- A successful `vercel_deploy` or `netlify_deploy` implies the code builds
- The deploy URL itself serves as proof the build passed
- Note: This does NOT prove tests passed, only that the build succeeded

## 4. CI Proof Link Format

When providing CI proof in task deliverables, use one of:

| Type | URL Pattern | Reliability |
|------|------------|-------------|
| Actions run | `https://github.com/{owner}/{repo}/actions/runs/{run_id}` | Best |
| Commit checks | `https://github.com/{owner}/{repo}/commit/{sha}` | Good |
| Deploy URL | `https://{project}.vercel.app` | Build-only proof |

## 5. Decision Tree

```
Need CI proof?
├── Try github.check_ci with timeoutMs
│   ├── Success → Use returned URL ✅
│   └── Fails (CLI missing) →
│       ├── web_browse GitHub Actions page → find run URL ✅
│       ├── web_browse REST API status endpoint → parse state ✅
│       └── Use deploy URL as build proof (note limitation) ✅
└── CI not configured for repo?
    └── Note "no CI configured" in deliverable, provide commit SHA ✅
```

## 6. Prevention

- When creating new repos, prefer GitHub Actions workflows that are self-contained (no local CLI dependency)
- Include a minimal CI workflow (`.github/workflows/ci.yml`) in every repo scaffold
- The `github.sync_workspace` → `github.check_ci` pipeline works when gh CLI is present; the fallbacks above cover when it isn't

---

**TL;DR:** If `check_ci` fails due to missing gh CLI, browse the GitHub Actions page or REST API status endpoint directly. Deploy URLs serve as build-only proof. Always include commit SHA in deliverables regardless.
