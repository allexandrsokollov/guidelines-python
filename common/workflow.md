# Development Workflow Rules (Strict)

These rules describe how work must be done. They apply to every change.

A clear boundary applies throughout:

- **Git operations are performed by a human, never by an AI agent.** Creating
  branches, staging, committing, pushing, opening pull requests, and merging are
  human responsibilities. An AI agent MUST NOT run git commands or drive the git
  history. An AI agent produces code and tests; the human reviews and commits them.

## 1. One logical change per branch (human)

Required:

- create a dedicated branch for each logical unit of work
- keep the branch focused on a single feature, fix, or refactor
- branch from an up-to-date `main`

Forbidden:

- mixing unrelated changes in one branch
- committing directly to `main`

## 2. One logical change per commit (human)

Required:

- one logical unit of work per commit
- write commit messages in English, in the imperative mood
- make the message describe the intent of the change

Forbidden:

- bundling unrelated changes into one commit
- vague messages like `fix`, `update`, `changes`

## 3. Review what is staged before committing (human)

Required:

- run `git status` before every `git add`
- stage only the files that belong to the current logical change

Forbidden:

- blind `git add -A` without reviewing what is staged
- committing generated files, secrets, or local artifacts

## 4. Tests are written with the code

Tests follow `common/testing.md`. In addition:

Required:

- add or update tests in the same change as the code
- write tests according to the testing guideline

Forbidden:

- merging behavior changes without tests

## 5. Linters and type checks must pass

Required:

- run `ruff` and `mypy` before a change is committed for merge
- fix the code so all linters and type checks report no errors

YOU MUST NEVER weaken, disable, or reconfigure a linter rule to make an error go
away. Suppressions (`# noqa`, `# type: ignore`) are exceptional, must be local
and specific, and must carry a comment explaining the concrete reason.

## 6. Pull request and review (human)

Required:

- open a pull request for every change merged into `main`
- describe what the change does and why
- address every review comment explicitly (fix or reply)
- resolve a conversation only after the comment is handled

Forbidden:

- merging without an approving review
- ignoring or silently dropping review comments

## 7. Pre-merge checklist (human)

Before merge, confirm:

- the branch contains one logical change
- commits are clean and messages are meaningful
- code follows the applicable guidelines
- tests are present, written per `common/testing.md`, and pass
- `ruff` and `mypy` report no errors
- all review comments are resolved
- at least one approving review is present
