# Security Review (2026-04-08)

Scope: dependency posture, hidden/runtime dependencies, external network calls, and external process invocation.

## Findings

### 1) `package-lock.json` is out-of-sync with `package.json` (Medium)
- `package.json` declares package name/version `fieldtheory@1.3.2`.
- `package-lock.json` root still declares `ft-bookmarks@1.3.0` and includes a root runtime dependency on `zod` that is not present in `package.json`.
- Risk: confusing/reproducibility drift during local installs (`npm ci`), accidental dependency creep, and harder supply-chain auditing.

### 2) Runtime reaches multiple external endpoints (Informational)
- X GraphQL and syndication endpoints.
- X OAuth and REST APIs.
- npm registry for update checks.
- Arbitrary media URLs from bookmark records.
- Risk: expected behavior for this CLI, but should be clearly documented and controllable (timeouts, optional disable flags, and endpoint allowlisting where feasible).

### 3) Local command execution present for browser cookies + LLM engine invocations (Informational)
- Uses `execFileSync` / `spawnSync` for `security`, `secret-tool`, `sqlite3`, PowerShell candidates, and user-installed LLM CLIs (`claude`, `codex`).
- Current implementation avoids shell interpolation (`execFile` style APIs), which reduces command-injection risk.
- Risk remains around PATH hijacking / untrusted binaries in PATH.

## Recommendations

1. Regenerate lockfile from current `package.json` (`npm install --package-lock-only`) and commit it.
2. Add CI checks:
   - `npm ci --ignore-scripts`
   - lockfile drift check (`git diff --exit-code package-lock.json` after install)
3. Consider pinning absolute binary paths (or stronger validation) before invoking external tools.
4. Add an opt-out env var for network features (e.g., update check) for hardened environments.
5. Keep conservative timeouts for all network requests and fail closed where secrets are involved.
