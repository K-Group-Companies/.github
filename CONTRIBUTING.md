# Contributing Guide

👋 Welcome! We are thrilled that you want to contribute to our projects. This document establishes the standards for our organization to ensure our codebase remains clean, sustainable, and scalable.

## 🚀 Getting Started

1. **Explore the Code:** Please check the `README.md` in the specific repository you are working on for setup instructions.
2. **Check Issues:** Browse the [Issues] tab to see what needs to be worked on.
3. **Assign Yourself:** If you pick up an issue, please assign it to yourself so others know it's being handled.

### 🧹 One-Time Machine Setup: Global gitignore

If you use **Claude Code** (or any AI coding assistant that writes to a `.claude/` folder), configure a [global gitignore](https://docs.github.com/en/get-started/git-basics/ignoring-files#configuring-ignored-files-for-all-repositories-on-your-computer) once so your personal, machine-specific settings never get committed to any repo. Only `.claude/settings.json` is meant to be shared — `.claude/settings.local.json` and `.claude/scheduled_tasks.lock` are personal and often contain local paths.

Add these lines to your global ignore file. Git uses the file named by `core.excludesFile`; when that is unset it defaults to `$XDG_CONFIG_HOME/git/ignore`, i.e. `~/.config/git/ignore` (on Windows, `%USERPROFILE%\.config\git\ignore`):

```gitignore
**/.claude/settings.local.json
**/.claude/scheduled_tasks.lock
```

> **Note:** use forward slashes even on Windows. In gitignore syntax `\` is an escape character, not a path separator, so a backslash pattern like `.claude\settings.local.json` is non-portable — its behavior varies by platform. Forward slashes match reliably everywhere.

Verify it works with `git check-ignore -v <path>` in any repo — it should report the rule that matches.

## 🌿 Branching Strategy

We follow a [Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow).

* **Main Branch:** `main` is our production-ready branch. It is protected and cannot be pushed to directly.
* **Feature Branches:** Create a new branch for every feature or bug fix.
* **Naming Convention:**
  * Features: `feat/short-description` (e.g., `feat/login-page`)
  * Bugfix: `fix/short-description` (e.g., `fix/header-alignment`)
  * Hotfix: `hotfix/short-description` (e.g., `hotfix/production-crash`)

## 💻 Development Process

1. **Sync often:** Keep your branch up to date with `main` to avoid massive merge conflicts later.
2. **Linting:** Ensure your code passes our linting rules. (Run `npm run lint` or equivalent before committing).
3. **Tests:** Write unit tests for new logic. We aim for high code coverage.

## ✉️ Commit Messages

We strive to use **[Conventional Commits](https://www.conventionalcommits.org/)**. This helps us generate changelogs automatically.

Format: `<type>(<scope>): <subject>`

**Examples:**

* `feat(auth): add google sso login support`
* `fix(api): handle null response in user controller`
* `docs(readme): update setup instructions`
* `chore(deps): upgrade react to v18`

## 📥 Pull Request (PR) Process

1. **Draft PRs:** If your work is in progress but you want feedback, open a PR and mark it as a "Draft".
2. **Description:** Fill out the PR template completely. Context is king!
3. **Size:** Keep PRs small. If a PR touches 50+ files, consider breaking it down.
4. **Review:** You need **at least one** approval from a team member before merging.
5. **Merge:** Squash and merge is preferred to keep the history clean.

## 🤝 Code of Conduct

We are committed to providing a friendly, safe, and welcoming environment for all, regardless of level of experience, gender identity and expression, sexual orientation, disability, personal appearance, body size, race, ethnicity, age, religion, or nationality.

**Be kind. Be professional. Be helpful.**
