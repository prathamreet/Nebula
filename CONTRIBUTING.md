# Contributing to Nebula

Thank you for considering contributing to **Nebula**! We welcome contributions of all kinds — code, design, documentation, or testing.

---

## 🛠️ Getting Started

1. Fork the repository.
2. Clone your fork:
    ```bash
    git clone https://github.com/your-username/nebula.git
    cd nebula
    ```
3. Install prerequisites (C++ toolchain, Node.js, Rust).
4. Follow the steps in [README.md](./README.md#🚀-getting-started) to set up your local environment.

---

## 📋 Branching Strategy

| Type    | Naming Convention | Purpose        |
| ------- | ----------------- | -------------- |
| Feature | `feature/<name>`  | New features   |
| Bugfix  | `bugfix/<name>`   | Bug fixes      |
| Hotfix  | `hotfix/<name>`   | Urgent patches |
| Release | `release/vX.Y.Z`  | Release prep   |


PRs should target `main` or `develop` depending on the milestone.

---

## 🧾 Semantic Commit Messages

We follow [Conventional Commits](https://www.conventionalcommits.org):

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Examples:

-   `feat(frontend): add dark mode toggle`
-   `fix(bridge): resolve panic in event handler`
-   `chore(ci): upgrade Node to 20.x`

Use the following types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`.

---

## ✅ Pull Request Checklist

Before submitting a PR:

-   [ ] Follows project style & formatting
-   [ ] All tests pass (`npm run test`, `cargo test`, `ctest`)
-   [ ] Related issue referenced (\`Fixes #123\`)
-   [ ] Clear and concise commit history
-   [ ] No unrelated changes in diff

---

## 🔬 Testing

| Layer    | Command                       |
| -------- | ----------------------------- |
| C++      | `ctest` from `backend/build`  |
| Rust     | `cargo test` from `src-tauri` |
| Frontend | `npm run test` or Cypress e2e |

---

## 🧹 Linting & Formatting

-   JavaScript/TypeScript:

    ```bash
    npm run lint
    npm run format
    ```

-   Rust:
    ```bash
    cargo fmt && cargo clippy
    ```
-   C++:
    ```bash
    clang-format -i src/**/*.cpp
    clang-format -i backend/src/*.cpp
    ```

---

Thank you for helping make Nebula even more stellar! 🚀
`;
