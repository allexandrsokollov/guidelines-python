# Development Workflow Rules (Strict)

These rules describe how work must be done. They are mandatory for every change,
whether written by a human or an AI agent.

## 1. One logical change per branch

Required:

- create a dedicated branch for each logical unit of work
- keep the branch focused on a single feature, fix, or refactor
- branch from an up-to-date `main`

Forbidden:

- mixing unrelated changes in one branch
- committing directly to `main`

## 2. One logical change per commit

Required:

- one logical unit of work per commit
- write commit messages in English, in the imperative mood
- make the message describe the intent of the change

Forbidden:

- bundling unrelated changes into one commit
- vague messages like `fix`, `update`, `changes`

## 3. Check status before staging

Required:

- run `git status` before every `git add`
- stage only the files that belong to the current logical change

Forbidden:

- blind `git add -A` without reviewing what is staged
- committing generated files, secrets, or local artifacts

## 4. Write code to the guidelines

Before writing code, the relevant guidelines apply:

- `common/clean_code.md` — readability, naming, structure
- `common/typing.md` — explicit types and DTOs
- `common/testing.md` — behavior-focused tests
- the stack-specific error handling and settings rules (`django/` or `fastapi/`)

An AI agent MUST follow these guidelines and MUST NOT silently relax them.

## 5. Tests are part of the change

Required:

- add or update tests in the same change as the code
- cover the success path (asserting the full response payload)
- cover representative failure paths
- keep tests deterministic and named by behavior

Forbidden:

- merging behavior changes without tests
- asserting only status codes or one or two fields when the full contract matters

## 6. Linters and type checks must pass

Required:

- run `ruff` and `mypy` before every commit intended for merge
- fix the code so all linters and type checks report no errors

YOU MUST NEVER weaken, disable, or reconfigure a linter rule to make an error go
away. Suppressions (`# noqa`, `# type: ignore`) are exceptional, must be local
and specific, and must carry a comment explaining the concrete reason.

## 7. Self-review before requesting review

Before opening a pull request, confirm:

- the change is one logical unit
- names are clear, types are complete, errors are explicit
- tests cover success and key failure paths
- all tests pass
- `ruff` and `mypy` report no errors
- no secrets, generated files, or local artifacts are staged

## 8. Pull request and review

Required:

- open a pull request for every change merged into `main`
- describe what the change does and why
- address every review comment explicitly (fix or reply)
- resolve a conversation only after the comment is handled

Forbidden:

- merging without an approving review
- ignoring or silently dropping review comments

## 9. Pre-merge checklist

Before merge, confirm:

- the branch contains one logical change
- commits are clean and messages are meaningful
- code follows the applicable guidelines
- tests are present and pass
- `ruff` and `mypy` report no errors
- all review comments are resolved
- at least one approving review is present
