# Contributing to MammoCare

Contributions that improve research reproducibility, dataset auditing, evaluation design, testing, documentation, and code quality are welcome.

## Before You Begin

1. Read the root [`README.md`](README.md) to understand the current scope and limitations.
2. Check existing GitHub Issues, Discussions, and pull requests to avoid duplicate work.
3. For substantial changes, open an issue or discussion first so the proposed scope and evaluation plan can be reviewed.
4. Create a fork or use a feature branch if you have write access to the repository.

## Development Workflow

1. Start from the latest `main` branch.
2. Create a focused branch with a descriptive name, such as `docs/evaluation-notes` or `fix/group-audit`.
3. Keep each change set narrow and commits descriptive.
4. Run the checks relevant to your changes.
5. Update documentation when behaviour, assumptions, outputs, or methodology changes.
6. Open a pull request against `main` and describe what changed, why it changed, how it was tested, and what remains uncertain.

## Code and Notebook Guidelines

- Follow the existing project structure and coding conventions.
- Prefer clear, maintainable code over unnecessary abstraction.
- Keep data loading, preprocessing, grouping, partitioning, training, and evaluation decisions inspectable.
- Set and document random seeds when randomness affects results.
- Do not change evaluation partitions or metric definitions silently between model comparisons.
- Add or update tests for changes to reusable code or data-validation logic.
- Ensure notebook cells can be followed in order; explain non-obvious methodological decisions in Markdown.
- Avoid committing large generated artefacts unless they are explicitly required and suitable for version control.

The project declares Python 3.12 or newer. Depending on the change, useful checks include:

```bash
# Run when tests are present
pytest

# Run for relevant Python code changes
ruff check .
ruff format --check .
```

Run applicable checks from the repository root. If tests are not present, or a check is not relevant or cannot be run, explain that in the pull request rather than implying it passed.

## Research and Data Integrity

MammoCare is centred on breast histopathology image classification using BreaKHis. Changes affecting data or experiments should:

- Respect BreaKHis access conditions, licence terms, citation requirements, and any applicable privacy restrictions.
- Document data sources, preprocessing steps, exclusions, grouping rules, split logic, random seeds, configuration changes, and evaluation units where relevant.
- Never commit private patient information, credentials, access tokens, or sensitive data.
- Preserve exact filename-derived grouping tokens unless documented evidence justifies a change.
- Avoid claiming that filename-derived groups are verified patient identities. The current workflow uses a grouping proxy and cannot guarantee separation by real-world patient.
- Report limitations and negative or inconclusive findings alongside positive results.
- Support research claims with generated outputs, code, citations, or other inspectable evidence.
- Avoid comparing results unless the dataset, target task, evaluation unit, and split methodology are sufficiently comparable.

## Pull Requests

Every pull request should explain its purpose and scope, summarize the changes, list checks actually performed and their outcomes, and describe relevant limitations or risks. For research changes, state whether the dataset audit, grouping strategy, evaluation protocol, metrics, or results interpretation has changed.

Use the repository's pull request template. Keep changes focused and update relevant documentation or tests when necessary.

## Communication

Use GitHub Issues or Discussions for questions, bug reports, research proposals, and general improvements. Do not post sensitive security details or private data publicly; follow [`SECURITY.md`](SECURITY.md) for vulnerability reports.

Thank you for helping make MammoCare more transparent, careful, and reproducible.
