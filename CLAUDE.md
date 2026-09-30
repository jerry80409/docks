# CLAUDE.md

## Agent skills

### Issue tracker

Issue 放在 GitHub Issues（`gh` CLI）。發佈 spec 或 ticket（`/to-spec`、`/to-tickets`）時，內文用 `.github/ISSUE_TEMPLATE/` 的 template，取代 skill 內建的 template。See `docs/agents/issue-tracker.md`.

### Triage labels

預設五個 label：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。See `docs/agents/triage-labels.md`.

### Domain docs

Single-context：根目錄一個 `CONTEXT.md` 加上 `docs/adr/`。See `docs/agents/domain.md`.
