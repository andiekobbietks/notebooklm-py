# Upstream and fork maintenance

## Repository relationship

- **Upstream:** [teng-lin/notebooklm-py](https://github.com/teng-lin/notebooklm-py)
- **Fork:** [andiekobbietks/notebooklm-py](https://github.com/andiekobbietks/notebooklm-py)
- **Default branch:** `main`

This repository is a GitHub fork. Upstream remains the canonical source for the original project. The fork should preserve upstream history and notices, and keep fork-specific changes reviewable.

## Branch and PR policy

1. Treat fork `main` as the integration branch.
2. Keep feature work on short-lived branches.
3. Bring upstream changes through the scheduled/manual upstream-sync PR.
4. Do not auto-merge upstream syncs.
5. Resolve conflicts deliberately and rerun the relevant upstream test gates.
6. Keep fork-only work isolated where practical; avoid changing upstream package identity or publishing to PyPI without a separate decision.
7. Record the exact upstream commit/tag represented by each sync.

## Automated sync

The workflow at `.github/workflows/upstream-sync.yml` runs weekly and on manual dispatch. It fetches `teng-lin/notebooklm-py:main`, merges it into a dedicated branch based on the fork's `main`, and opens or updates a PR. It requires the repository's normal GitHub Actions token permissions for contents and pull requests.

The workflow intentionally does not:
- push to `main`;
- enable auto-merge;
- resolve conflicts automatically;
- bypass branch protection or required checks.

If upstream and fork changes conflict, the workflow fails visibly. Resolve the conflict on a dedicated branch, preserve the intended fork changes, run tests, and then open/update a PR.

## Local maintenance commands

```bash
git remote -v
# origin should be your fork; add upstream if it is not already configured.
git remote add upstream https://github.com/teng-lin/notebooklm-py.git

git fetch origin
git fetch upstream
git switch main
git pull --ff-only origin main

git switch -c sync/upstream-YYYY-MM-DD
git merge --no-edit upstream/main
# Resolve conflicts, then run the relevant tests and checks.
git push -u origin HEAD
```

Do not force-push shared branches or rewrite upstream history.

## Attribution and licensing

Retain the upstream MIT licence and copyright notice. Keep original authorship attribution clear in documentation. Fork-specific work should identify its authorship without implying that Teng Lin endorses or maintains this fork. Do not remove third-party notices from inherited files.

## Review checklist

- [ ] Does the PR identify the upstream base commit?
- [ ] Are upstream changes preserved without accidental rollback?
- [ ] Are conflicts reviewed rather than mechanically accepted?
- [ ] Do the existing lint, type-check and test gates pass?
- [ ] Are fork-specific changes covered by tests?
- [ ] Are undocumented Google API assumptions stated as assumptions?
- [ ] Are credentials and notebook content excluded from logs?
- [ ] Are docs and diagrams updated with the implementation?
- [ ] Is any proposed capability clearly separated from current verified behaviour?
