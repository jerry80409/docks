# Issue tracker: GitHub

這個 repo 的 issue 與 spec 都放在 GitHub issues（`jerry80409/docks`），所有操作都用 `gh` CLI。

## Issue template

`.github/ISSUE_TEMPLATE/` 裡的 template 是 issue 內文結構的唯一來源。發 spec 或 ticket 時，用 repo 的 template 取代 skill 內建的 template：

| 情境 | Template | Labels |
| --- | --- | --- |
| `/to-spec` 發佈 spec | `.github/ISSUE_TEMPLATE/spec.md` | `spec`、`ready-for-agent` |
| `/to-tickets` 發佈 ticket | `.github/ISSUE_TEMPLATE/ticket.md` | `ticket`、`ready-for-agent` |

填寫方式：

1. 讀 template，去掉 frontmatter（兩個 `---` 之間）。
2. 依 skill 的規則填每一節，章節標題保持原樣（中文附英文對照，英文部分對應 skill template 的章節名）。填完就刪掉 `<!-- -->` 說明註解；選填且沒內容的章節整節刪掉。
3. 內文用繁體中文。
4. 內文寫進暫存檔，用 `--body-file` 建立 issue，並加上表格裡的 labels（template frontmatter 的 labels 只在網頁 UI 生效，`gh` 不會自動套用）：

   ```sh
   gh issue create --title "..." --body-file <file> --label ticket --label ready-for-agent
   ```

### Ticket 的關聯

依相依順序（blocker 先）建立 ticket，每張建好後：

- **上層 spec**：把 ticket 掛成 spec 的 sub-issue，並在「上層 Spec (Parent)」寫 `#<spec>`。
  `gh api --method POST repos/jerry80409/docks/issues/<spec>/sub_issues -F sub_issue_id=<ticket-db-id>`
- **Blocked by**：用 GitHub 原生 issue dependencies 建立 edge，並在「被誰阻擋 (Blocked by)」列出 `#<n>`，讓網頁上讀內文的人也看得到。
  `gh api --method POST repos/jerry80409/docks/issues/<ticket>/dependencies/blocked_by -F issue_id=<blocker-db-id>`

`<...-db-id>` 是 issue 的數字 database id：`gh api repos/jerry80409/docks/issues/<n> --jq .id`（不是 `#number` 或 `node_id`）。

## Conventions

- **Create an issue**: 見上方「Issue template」。
- **Read an issue**: `gh issue view <number> --comments`, filtering comments by `jq` and also fetching labels.
- **List issues**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'` with appropriate `--label` and `--state` filters.
- **Comment on an issue**: `gh issue comment <number> --body "..."`
- **Apply / remove labels**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **Close**: `gh issue close <number> --comment "..."`

## Pull requests as a triage surface

**PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_

When set to `yes`, PRs run through the same labels and states as issues, using the `gh pr` equivalents:

- **Read a PR**: `gh pr view <number> --comments` and `gh pr diff <number>` for the diff.
- **List external PRs for triage**: `gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments` then keep only `authorAssociation` of `CONTRIBUTOR`, `FIRST_TIME_CONTRIBUTOR`, or `NONE` (drop `OWNER`/`MEMBER`/`COLLABORATOR`).
- **Comment / label / close**: `gh pr comment`, `gh pr edit --add-label`/`--remove-label`, `gh pr close`.

GitHub shares one number space across issues and PRs, so a bare `#42` may be either: resolve with `gh pr view 42` and fall back to `gh issue view 42`.

## When a skill says "publish to the issue tracker"

Create a GitHub issue from the matching template (see "Issue template").

## When a skill says "fetch the relevant ticket"

Run `gh issue view <number> --comments`.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue with **child** issues as tickets.

- **Map**: a single issue labelled `wayfinder:map`, holding the Notes / Decisions-so-far / Fog body. `gh issue create --label wayfinder:map`.
- **Child ticket**: an issue linked to the map as a GitHub sub-issue (`gh api` on the sub-issues endpoint). Where sub-issues aren't enabled, add the child to a task list in the map body and put `Part of #<map>` at the top of the child body. Labels: `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Once claimed, the ticket is assigned to the driving dev.
- **Blocking**: GitHub's **native issue dependencies** (see "Ticket 的關聯" for the command). GitHub reports `issue_dependencies_summary.blocked_by` (open blockers only, the live gate). Where dependencies aren't available, fall back to a `Blocked by: #<n>, #<n>` line at the top of the child body. A ticket is unblocked when every blocker is closed.
- **Frontier query**: list the map's open children (`gh issue list --state open`, scoped to the map's sub-issues / task list), drop any with an open blocker (`issue_dependencies_summary.blocked_by > 0`, or an open issue in the `Blocked by` line) or an assignee; first in map order wins.
- **Claim**: `gh issue edit <n> --add-assignee @me`, the session's first write.
- **Resolve**: `gh issue comment <n> --body "<answer>"`, then `gh issue close <n>`, then append a context pointer (gist + link) to the map's Decisions-so-far.
