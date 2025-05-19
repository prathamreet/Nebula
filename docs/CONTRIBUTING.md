# Contributing to **NEBULA**

Thank you for wanting to contribute to **NEBULA**! 
Whether you're new to open source or an experienced contributor, this guide will help you follow our project's conventions—from forking to making your first pull request—ensuring a smooth collaboration for everyone.



## 🛡️ Code of Conduct

 We expect all participants to:

* Be respectful, patient, inclusive.
* Assume good intent and give constructive feedback.
* Follow community guidelines for discussions and reviews.

Violations may result in removal of contributions or banning.

<!-- 
## 📁 Project Structure

```
/         # root
├─ src/   # source code (modules, components)
├─ tests/ # unit & integration tests
├─ docs/  # documentation files
├─ .github/          
│   ├─ ISSUE_TEMPLATE/   
│   │   ├─ bug_report.md
│   │   └─ feature_request.md
│   └─ workflows/        # CI/CD configs
├─ .eslintrc.js
├─ pyproject.toml
└─ README.md
```

Understanding where code and docs live helps contributors find what they need quickly. -->

---

## 🚀 Getting Started

Follow these steps to set up your local environment:

1. **Fork** the repository (click “Fork” in the top-right).
2. **Clone** your fork locally:

   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd <repo-name>
   ```
3. **Add upstream** remote to stay updated:

   ```bash
   git remote add upstream https://github.com/<org>/<repo-name>.git
   git fetch upstream
   ```
4. **Install** dependencies

   <!-- ```bash
   # JavaScript/TypeScript\ n  npm install
   # Python
   pip install -r requirements.txt
   ``` -->
   
5. **Verify** everything works?
<!-- 
   ```bash
   npm test    # or pytest
   ``` -->

Your local `origin` points to your fork; `upstream` points to the main project.

---

## 🐛 Reporting Issues

Before you report:

* Search [open issues](https://github.com/<org>/<repo>/issues) for duplicates.
* Check closed issues for similar discussions.

### Bug Report Template

```yaml
name: Bug report
about: Create a report to help us improve
labels: [bug]

Short Description
A brief description of the bug.

Steps to Reproduce
1. Go to ...
2. Click ...
3. Observe ...

Expected Behavior
Clear description of what should happen.

Actual Behavior
What actually happens, with error messages or screenshots.

Environment:
- Version: vX.Y.Z
- OS: e.g., macOS Big Sur
- Browser/Runtime: Chrome 120, Node v18

Additional Context
(Optional) Logs, stack traces, or code snippets.
```

> **Example:**
> When I open the settings modal on mobile (iPhone 12), it overlaps the header (see screenshot). The header should push down instead.

---

## 🌟 Proposing Features

Got an idea? Propose it!

### Feature Request Template

```yaml
name: Feature request
about: Suggest an idea for this project
labels: [enhancement]

Problem Statement
What problem does this feature solve?

Use Cases
1. User scenario A
2. Developer scenario B

Proposed Solution
Outline your approach or API design.

Alternatives Considered
List other options and why you chose this one.
```

> **Example:**
> **Problem:** Right now, users can’t bulk-delete items.
> **Use Case:** Admin wants to clean up old records in one click.
> **Solution:** Add a “Select All” checkbox and a `bulkDelete(ids: number[])` API.

Maintainers will review and discuss feasibility before implementation.

---

## 🌲 Branching & Workflow

We follow **GitHub Flow**:

1. **Sync** `main`:  `git checkout main && git fetch upstream && git rebase upstream/main`
2. **Create** a feature branch:

   * `git checkout -b feature/user-auth`
   * or `git checkout -b fix/session-timeout`
3. **Work & Commit**:

   * Make small, focused commits (see Commit Messages).
4. **Push** to your fork:

   ```bash
   git push origin feature/user-auth
   ```
5. **Open** a Pull Request against `main`.
6. **Respond** to review comments.
7. Once approved, **merge** via GitHub UI and delete your branch.

> **Branch Naming Examples:**
>
> * `feature/<short-description>` (e.g., `feature/login-form`)
> * `fix/<issue-number>-<short>` (e.g., `fix/123-button-overflow`)
> * `docs/<topic>` (e.g., `docs/contributing`)
> * `chore/<task>` (e.g., `chore/update-deps`)

---

## ✍️ Commit Message Guidelines

Follow **Conventional Commits** for clarity and automation:

```
<type>(<scope>): <short summary>

[optional detailed description]

[footer (e.g., Closes #issue)]
```

| Type     | Description                     |
| -------- | ------------------------------- |
| feat     | a new feature                   |
| fix      | a bug fix                       |
| docs     | documentation only              |
| style    | formatting, missing semi-colons |
| refactor | code change without feature/fix |
| test     | adding or updating tests        |
| chore    | build process or auxiliary tool |

**Example:**

```
fix(auth): reject expired JWT tokens

Previously, expired tokens were silently accepted. Now we return 401.

Closes #42
```

* Limit subject to 50 chars, imperative mood, no trailing period.
* Wrap body at 72 chars.
* Reference issues with `Closes #` to auto-close.

---

## 📝 Writing a Pull Request

A stellar PR includes:

1. **Title:** Imperative and concise (e.g., `Add password reset flow`).
2. **Description:**

   * **What** you changed and **why**.
   * **Issue** link: `Closes #123`.
   * **Screenshots** or GIFs for UI work.
3. **Checklist:**

   ```markdown
   - [ ] I’ve run `npm test` or `pytest` locally.
   - [ ] I added/updated tests for my change.
   - [ ] I updated documentation where necessary.
   - [ ] All lint checks pass.
   ```
4. **Review:** Tag reviewers or teams if needed.

> **Tip:** Keep PRs small (one logical change per PR). Use drafts for WIP.

---
<!-- 
## 🧪 Testing & QA

* **Unit Tests:** Place under `tests/unit/`.
* **Integration Tests:** Under `tests/integration/`.
* **Manual QA:** Describe steps in PR if manual verification is needed.
* **CI Pipeline:** Every PR triggers lint, test, and build checks via GitHub Actions.

---

## 🎨 Code Style & Linters

We enforce consistency with:

* **JavaScript/TS:** ESLint + Prettier
* **Python:** Black + Flake8
* **CSS:** Stylelint

**Pre-commit hooks:** Use Husky to auto-run formatters before commits.

---

## 📚 Documentation

* **README.md:** Project overview and quickstart.
* **docs/**: Detailed guides and tutorials.
* **CHANGELOG.md:** Follow [Keep a Changelog](https://keepachangelog.com/) standard.

Update docs when you add features or modify behavior.

---

## 💬 Support & Communication

* **Issues & Discussions:** Use GitHub Issues and Discussions tabs.
* **Real-time Chat:** Join us on [Discord](invite-link) for quick help.
* **Mailing List:** Subscribe at `<email-list-link>` for announcements.

--- -->

Thank you for helping make **<Project Name>** even better! 🚀
