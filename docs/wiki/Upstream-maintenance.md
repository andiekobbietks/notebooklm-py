# Upstream maintenance

- Upstream: [teng-lin/notebooklm-py](https://github.com/teng-lin/notebooklm-py)
- Fork: [andiekobbietks/notebooklm-py](https://github.com/andiekobbietks/notebooklm-py)

Keep upstream history and licence notices. Sync through a dedicated branch and reviewable pull request. Do not auto-merge upstream changes or resolve conflicts blindly.

The proposed [upstream sync workflow](../../.github/workflows/upstream-sync.yml) runs weekly and supports manual dispatch. It records commit SHAs and opens/updates a PR. Review workflow permissions and run a manual dispatch after merge before relying on it operationally.

Local setup:

```bash
git remote add upstream https://github.com/teng-lin/notebooklm-py.git
git fetch upstream
git fetch origin
git switch main
git pull --ff-only origin main
git switch -c sync/upstream-YYYY-MM-DD
git merge upstream/main
```

Resolve conflicts deliberately, run tests and open a PR. Never force-push the shared main branch.
