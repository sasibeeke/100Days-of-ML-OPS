The xFusionCorp Industries ML team enforces code quality on every commit via pre-commit. A draft .pre-commit-config.yaml exists in the git repository at /root/code/fraud-detection/, but it does not match the team's standard and pre-commit run --all-files fails against it. Correct the configuration.

A git repository already exists at /root/code/fraud-detection/ with .pre-commit-config.yaml and process.py already tracked. pre-commit is installed system-wide.

The corrected configuration must declare the following five hooks so that pre-commit run --all-files executes every one of them:

trailing-whitespace, end-of-file-fixer, and check-yaml – All three sourced from the pre-commit/pre-commit-hooks repository, pinned to a current release;
ruff – Sourced from the astral-sh/ruff-pre-commit repository, pinned to a current release;
black – Sourced from the psf/black-pre-commit-mirror repository, pinned to a current release.
Every repository entry in the configuration must include a rev: field.

Review the existing .pre-commit-config.yaml and correct everything that prevents the hooks above from running.

Once the configuration is correct, register the hooks with git and run them against the tracked files:
   ```bash 
   pre-commit install
   pre-commit run --all-files
   ```
Tip: pre-commit autoupdate queries each referenced repository and rewrites the rev: pins to the latest released tag. This is the standard way to discover current versions without looking them up by hand.
=> Given .pre-commit-config.yaml file
# vi  .pre-commit-config.yaml
```config

repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v2.3.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check_yaml

  - repo: https://github.com/charliermarsh/ruff-pre-commit
    rev: v0.1.0
    hooks:
      - id: ruff-lint

  - repo: https://github.com/psf/black-pre-commit-mirror
    hooks:
      - id: black
```

After running gives issue of version error
```bash
# pre-commit install
# pre-commit run --all-files
```
so for this do this
```upd 
# pre-commit autoupdate
``` 
and then run 
```inst
# pre-commit install
# pre-commit run --all-files
```
# cat  .pre-commit-config.yaml
```corr
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v6.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.15.13
    hooks:
      - id: ruff

  - repo: https://github.com/psf/black-pre-commit-mirror
    rev: 26.5.1
    hooks:
      - id: black
```
