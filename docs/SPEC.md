# github-mate specification (draft)

Decisions made in conversation with the human are recorded here. **Decided** holds what the human stated. **Assumed** holds Claude's understanding or proposals that the human has not yet confirmed. **Open** holds what is not yet settled.

This document describes the finished product. The order in which it is built is kept apart in §4 and is not part of the specification.

Last updated: 2026-10-10

## 1. Decided

### 1.1 Target users

People who develop around GitHub, working in the following way.

- Tasks are managed as Issues.
- Each implemented feature is submitted as a PR linked to an Issue.
- Branches are managed as git worktrees.

### 1.2 Repositories

- Several repositories are registered, and the user switches between them.
- A repository is registered by giving its URL. The user does not clone it first; the app clones it and manages where the repository and its worktrees live on disk (the human, 2026-10-10).
- The user gives one base directory in the app settings; the app keeps everything under it.

### 1.3 Issues and PRs

- The app offers roughly the same functionality as GitHub's own Issue page and PR page.
- The user also reviews other people's PRs in the app, not only their own.

### 1.4 Worktrees

- Issue, worktree and PR are not one-to-one.
  - A worktree may exist only locally, with no Issue yet (work still at the exploration stage).
  - One Issue may have several PRs, or several independent worktrees, when different approaches are tried.
- Each worktree has a row of action buttons:
  - open the folder
  - start an AI
  - run a command such as a build
- Buttons can be added in a settings screen. One button is defined by an icon and a command.
- Buttons for all repositories and buttons for one repository are managed on separate screens, not in one list (the human, 2026-10-10).
- There is one settings screen. It has no separate "repository settings" screen; per-repository settings are sections inside it, reachable from the same navigation as the app-wide sections (the human, 2026-10-10). The navigation group is named 「リポジトリごとの設定」, not "per-repository buttons"; selecting a repository shows a page for that repository whose first (and at present only) part is its buttons, so further per-repository items can be added to the same page.
- Each worktree has a delete button. (The human, 2026-10-10: build it this way for now.)
  - Pressing it opens a confirmation dialog that shows what would be lost: uncommitted changes, commits not pushed, running terminal sessions, and whether the PR is merged.
  - Checkboxes choose whether the local branch and the remote branch are deleted too.
  - If there are uncommitted changes or unpushed commits, deletion needs a second, explicit confirmation.
  - Running sessions in the worktree are terminated.

### 1.5 AI

- Supported: Claude Code, Codex, Antigravity CLI.
- The app starts an embedded terminal and launches the CLI inside it.
- The command that launches each AI's CLI is set per AI in the settings screen. The defaults are the bare commands `claude`, `codex` and `agy`; a user who wants arguments adds them there (the human's own setting will be `claude --dangerously-skip-permissions` and `codex --yolo`). The app never ships with permission-skipping flags.
- Terminal sessions end when the app quits. Quitting while sessions are running asks for confirmation. Continuing a conversation after a restart is left to each CLI's own resume function.
- The app does not reimplement any part of a CLI's own interface (prompt input, skill or file suggestions). Reproducing it and following its changes is too costly. The user types into the real CLI in the terminal.
- An AI is started for more than resolving an Issue; asking for a review of an existing PR is another use. Whether to add a feature specific to handing over an Issue is to be considered (§2 proposes not to).

### 1.6 Parallel work

- The expected use is handing several Issues to AIs at the same time.
- The UI must suit working on them in parallel.
- About 5 worktrees are active at once; the design must hold up to 10.
- After an AI finishes, the human mostly verifies by running the software, often without reading the diff. The diff must nevertheless be viewable inside the app.
- The human always has an AI review the result, sometimes through a separate service, depending on the occasion.
- Creating Issues can be left to the AI.
- Screen layout and appearance are decided through HTML mocks (`mock/`).
- The worktree screen follows `mock/worktree-v3-c.html` (chosen by the human from three candidates, 2026-10-10). The same file also holds the Issue screen, the PR screen, the settings, and the new-worktree and delete dialogs in the same design; the human has not yet reviewed those. The earlier mocks (`index.html`, `worktree-v2.html`, `worktree-v3-a.html`, `worktree-v3-b.html`) are kept for comparison only; the human found v2 information-rich but unrefined.
  - Worktree rows in the list all have the same height and are separated by lines.
  - No elapsed-time display on rows.
  - Each worktree row (except the main working tree) has a delete button (trash icon) at its right end, so a worktree can be deleted without opening it. In the mock it shows on hover and on the selected row, and always on merged rows.

### 1.7 What the worktree list makes prominent

- Attention marks behave like unread mail: once the user opens the worktree, the prominent mark goes away (the human, 2026-10-10).
- A worktree that has been seen but still has something outstanding must be distinguishable from one where nothing is wrong.
- Which events are surfaced is left to Claude's judgement (the human, 2026-10-10). Claude's choice:
  - **Put the worktree under "needs attention":** an AI is waiting for an instruction; an AI exited abnormally (error, usage limit); a button-started command failed; CI failed; review comments arrived; changes were requested; the PR conflicts with its base; the PR is mergeable; the PR was closed without merging; someone else's PR that is checked out here was updated.
  - **Two colours:** red for "something broke" (AI exited abnormally, command failed, CI failed, conflict); the accent colour for "your turn" (the rest).
  - **Three levels per row:** unread (loud); read with something outstanding (quiet, but stays in the group and keeps its reason); nothing wrong (dim). A new event makes a worktree unread again. The user can mark a worktree unread by hand.
  - **Not shown in the list:** a command finishing normally (its tab only); uncommitted or unpushed changes (diff link and delete dialog); branch behind its base (PR panel); new comments on the Issue (a mark on the Issue link).
  - **A review request for a PR with no worktree** shows as a count on the PR navigation entry.
  - A read row keeps its mark as a hollow ring in the same colour (red or accent); the human confirmed this level of prominence is right (2026-10-10).
  - Read state is kept separately for a worktree and for each session tab inside it: opening a worktree marks the worktree read, and a tab that has not been viewed keeps its mark (confirmed, 2026-10-10).
  - Rows do not move when they become read; the order changes only when a new event arrives.
- No OS notifications or sounds for now.

### 1.8 Form

- A desktop app.
- Runs on Linux, macOS and Windows.
- Built with Tauri or Electron.
- The UI language is Japanese. The app is not meant for worldwide distribution; other languages are added only if demand appears.

## 2. Assumed (not yet confirmed)

- **The unit shown on screen is a worktree.** A worktree is one branch. Its PR is found from the branch; its Issue is found from the PR's linked Issues, or from a link the app stores when the worktree is created from an Issue. Both are optional. This covers a worktree with no Issue, several worktrees for one Issue, and a worktree made from someone else's PR.
- **Each repository has three lists: worktrees, Issues, PRs.** A worktree can be created from an Issue (new branch), from a PR (checking out its branch, including PRs from forks), or from nothing.
- **Each way of creating a worktree has its own entry point** (the human, 2026-10-10): from nothing, the "+" at the top of the worktree list; from an Issue, the button at the top right of that Issue's page; from a PR, the button at the top right of that PR's page. The dialog does not let the user switch between the three; when opened from an Issue or PR it shows that Issue or PR as fixed text.
- **The creation dialog checks the branch name against existing branches** (the human, 2026-10-10, for creation from an Issue; Claude applies the same check to creation from nothing, since the dialog is the same). The check covers local branches and the remote's branches, and runs on the offered name and on every edit. While the name is taken, the dialog says whether the clash is local or remote and the create button is disabled. The name offered for an Issue is the first free one of `issue-<n>`, `issue-<n>-2`, ….
- **The Issue page and the PR page offer both "open worktree" and "create worktree"** (the human, 2026-10-13): when worktrees for that Issue or PR already exist, the user chooses between opening one and creating another; "create" is never replaced by "open". With one existing worktree "open" goes straight to it; with several it is a menu of their branches.
- **The creation dialog warns when the Issue or PR already has a worktree** (the human, 2026-10-13): under the Issue or PR line, with the branch names of the existing worktrees, whether or not their names resemble the new one. It is a notice only; creation is not blocked.
- **Errors and warnings use different colours** (the human, 2026-10-13). An error (something that blocks the action, such as a taken branch name) is red; a warning (a notice that does not block, such as the one above) is the accent colour that the worktree list uses for "your turn". The human first asked for the notice in red and then for the two to be distinguished.
- **Creating a worktree from a PR** (the human, 2026-10-10 and 2026-10-13):
  - The dialog is always shown (the human, 2026-10-13, reversing "no dialog" of 2026-10-10). It shows the PR as fixed text, an editable local branch name and the destination path. There is no base branch field, and no explanatory text about the name clash or the push destination (the human, 2026-10-13: not needed).
  - If no local branch has the PR's branch name, the field is pre-filled with that name. The PR's own remote branch does not count as a clash.
  - If a local branch of that name exists (in a worktree or left behind by a deleted one, or an unrelated branch such as a fork's `main`), creation under that name is refused, as for Issues. The field is pre-filled with the first free one of `<PR branch>-2`, `-3`, …. The new local branch tracks the PR's branch.
  - Git facts behind this (measured with git 2.53.0): one local branch cannot be checked out in two worktrees (`fatal: 'feat' is already used by worktree at ...`), but two local branches may track the same remote branch. A plain `git push` from a local branch whose name differs from its upstream fails under the default `push.default=simple` and works with `push.default=upstream`; the app sets that in the clones it manages (Claude's technical choice) so that the user and the AIs can push to the PR without extra arguments.
  - Consequence: a local branch left behind by a deleted worktree is never silently reused or reset. It can be used as the base branch in the "+" dialog.
- **Where a branch name has to be assigned automatically** (Claude's analysis, not yet confirmed):
  - From an Issue: a new branch is created. Offer `issue-<number>`, editable; a slug cannot be derived mechanically from a Japanese title. If that branch already exists (a second worktree for the same Issue), offer `issue-<number>-2`, and so on. Git does not allow one branch to be checked out in two worktrees.
  - From a PR: the PR's own branch name is used when no local branch has it. Otherwise the dialog offers `<PR branch>-2`, and so on (see above).
  - From nothing: a new branch with no source to derive a name from.
  - The directory name is derived from the branch name at creation and must be unique as well. A branch can later be renamed or switched inside the worktree (by the user or an AI), so the app identifies a worktree by its directory and reads its current branch, and the directory name may come to differ from the branch name.
- **The repository's main working tree appears in the worktree list too**, so an AI can be started on the default branch (for example to discuss the specification on `develop`) with no Issue or PR.
- **Three AI states are shown: running (実行中), waiting for input (入力待ち), exited (終了).** Waiting for input means the AI has finished its turn or stopped to ask for permission, and nothing proceeds until the user types.
- **"Roughly the same functionality" includes line-level review comments on a PR's diff.**

- **AI state belongs to a terminal session, not to a worktree.** One worktree may run several AI sessions at once, including several of the same AI. The worktree list shows one indicator per AI session. A worktree counts as needing the user if any of its sessions is waiting, even while another is running. The app does not prevent or warn about two AIs editing the same worktree.
- **The worktree list is grouped by what the user has to do:** needs attention (an AI is waiting, CI failed, review comments arrived, mergeable), AI working, waiting for results (CI running, review pending), done (merged, can be deleted). The terminal is the main area; Issue, PR and diff are in a side panel. Being tried in `mock/worktree-v2.html`.
- **No "next" button** that jumps to the next worktree needing attention (the human, 2026-10-10: leave it out for now).

### 2.1 Assumed workflows

The human said these are roughly right (2026-10-10).

- **A. Implement an Issue.** Create a worktree from the Issue, start an AI, check the result (run it, tests, diff), open a PR, wait for CI and review, merge, delete the worktree.
- **B. Several A's at once.** Hand one to an AI, move to the next, then go round the worktrees that have stopped. The main way the app is used.
- **C. Several approaches to one Issue.** Two or more worktrees from the same Issue; compare, keep one, delete the rest.
- **D. Exploration before an Issue exists.** A worktree with no Issue; it becomes an Issue or PR, or is deleted.
- **E. Review someone else's PR.** Create a worktree from the PR, ask an AI to review, read its findings, comment on lines, approve or request changes, delete the worktree.
- **F. Respond to review comments or a CI failure** on one's own PR: go back to the worktree, have the AI fix it, push.
- **G. Consult in the main working tree** (`develop` or `main`): discuss the specification, have the AI create Issues. No worktree is created.

What these ask of the worktree screen: (1) notice which worktrees need the user (an AI stopped, CI failed, review comments arrived), (2) get there fast, (3) check the result, (4) issue the next action.

## 3. Open

- **Screen layout.** Under discussion through the mock.
- **How to detect that an AI session has stopped and is waiting, or has exited abnormally.** The unread model in §1.7 depends on it. Detection differs per CLI; `agy --help` shows no hook or notify option. To be investigated and measured: each CLI's own notification mechanism, and a CLI-independent method based on what the terminal receives (bell, notification escape sequences, output going idle, process exit code).
- **Which of GitHub Projects, milestones and sub-issues the human uses**, and so which the Issue view must cover.
- **Framework: Tauri or Electron.** Claude decides after measuring. The current assessment is in §5.
- **GitHub authentication.** Claude will propose.
- **Changing the base directory.** The human considers this a large problem and wants to examine it separately (2026-10-10). Options so far: (a) move everything to the new location; (b) leave existing repositories where they are and use the new location only for new ones; (c) change only the setting without touching files (the user has moved, or will move, the directory themselves). Also to settle: what to do when the destination already contains folders.
- **The human's own branch naming rule**, to choose the default offered in the new-worktree dialog.

## 4. Development steps

Separate from the specification. Claude cuts the steps by what is easy to develop and easy to verify. The human has no preference about which step delivers what, and starts using the app only after the final version is built.

To be written once the specification has settled.

## 5. Framework assessment

With three operating systems and an embedded terminal decided, the assessment moved from "Tauri first" toward Electron. Not final.

- Tauri renders with a different engine on each OS (WebKitGTK on Linux, WKWebView on macOS, WebView2 on Windows). The heavy screens, the terminal and the diff view, would have to be verified on three engines.
- Electron runs the same Chromium on all three, so what is verified on Linux works almost unchanged elsewhere. Its terminal stack (xterm.js with node-pty) is the one VS Code uses.
- Tauri's advantages are a small distributable, low memory use, and a backend written in Rust.

Before deciding, build a prototype on Linux under WebKitGTK that streams heavy output into several terminals at once, and measure it. If no problem appears, choose Tauri; if one does, choose Electron.
