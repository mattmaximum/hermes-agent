# Hermes PR Workflow Guide

A streamlined guide for contributing to Hermes Agent via GitHub pull requests.

## Quick Start

```bash
# 1. Setup (one-time)
git config --global credential.helper store
printf "protocol=https\nhost=github.com\nusername=YOUR_USERNAME\npassword=YOUR_TOKEN\n" | git credential store

# 2. Start working
git checkout main && git pull origin main
git checkout -b feat/your-feature-name

# 3. Make changes (using Hermes tools or manually)
# ... your work here ...

# 4. Commit
git add your-changed-files
git commit -m "feat: brief description of changes

More detailed explanation if needed. Wrap at 72 characters."

# 5. Push and create PR
git push -u origin HEAD

# Then use either:
# Option A: gh CLI (recommended)
gh pr create --title "feat: your feature" --body "## Summary\nDescribe what this PR does"

# Option B: Manual (if gh not available)
# Get PR number from GitHub UI after pushing, then:
gh pr checks --watch  # Monitor CI
gh pr merge --squash --delete-branch  # Merge when green
```

## Detailed Steps

### 1. Authentication
Hermes supports two authentication methods:

**gh CLI (Recommended):**
```bash
gh auth login  # Follow prompts
gh auth setup-git  # Configure git to use gh credentials
```

**Git-only (HTTPS token):**
```bash
git config --global credential.helper store
printf "protocol=https\nhost=github.com\nusername=USER\npassword=TOKEN\n" | git credential store
```

### 2. Branch Naming
Use descriptive prefixes:
- `feat/` - new features
- `fix/` - bug fixes
- `docs/` - documentation
- `refactor/` - code restructuring
- `test/` - adding tests
- `ci/` - CI/CD changes
- `chore/` - maintenance

### 3. Commit Messages
Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
type(scope): short description

Optional body: detailed explanation
Footer: References issues (Closes #123)
```

Types: `feat`, `fix`, `docs`, `refactor`, `test`, `ci`, `chore`, `perf`

### 4. Creating a PR
After pushing your branch:

**With gh CLI:**
```bash
gh pr create \
  --title "feat: add user authentication" \
  --body "## Summary
- Adds JWT-based auth endpoints
- Secure password hashing
- Middleware for protected routes

## Test Plan
- [ ] Unit tests pass
- [ ] Integration tests verified

Closes #42"
```

**Without gh (using curl):**
```bash
BRANCH=$(git branch --show-current)
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/NousResearch/hermes-agent/pulls \
  -d "{\"title\":\"feat: your feature\",\"head\":\"$BRANCH\",\"base\":\"main\"}"
```

### 5. Monitoring CI
Check status with:
```bash
# With gh
gh pr checks --watch

# Without gh
SHA=$(git rev-parse HEAD)
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/NousResearch/hermes-agent/commits/$SHA/status
```

### 6. Merging
When all checks pass:
```bash
# With gh (recommended)
gh pr merge --squash --delete-branch

# Without gh
PR_NUMBER=123
curl -s -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/NousResearch/hermes-agent/pulls/$PR_NUMBER/merge \
  -d '{"merge_method":"squash"}'

# Cleanup
git checkout main && git pull origin main
git branch -d your-branch-name
git push origin --delete your-branch-name
```

## Hermes-Specific Tips

### Using Hermes Tools for Development
Hermes includes built-in tools that can help with PR workflow:

- **File operations**: Use `read_file`, `write_file`, `patch` instead of terminal commands
- **Search**: Use `search_files` to find code patterns
- **Skills**: Load `github-pr-workflow` skill for advanced automation
- **Session search**: Use `session_search` to find past discussions about similar changes

### Testing Your Changes
Before submitting:
```bash
# Run relevant tests
pytest tests/specific_module/ -v

# Or quick check
pytest tests/ -q -k "keyword"

# Full test suite (takes longer)
scripts/run_tests.sh tests/specific_directory/
```

### Common Workflows

**Bug Fix:**
```bash
git checkout main && git pull origin main
git checkout -b fix/issue-description
# ... fix the bug ...
git commit -m "fix: resolve issue with X
- Root cause was Y
- Fix ensures Z
Closes #123"
git push -u origin HEAD
# Create PR, monitor CI, merge
```

**Documentation:**
```bash
git checkout main && git pull origin main
git checkout -b docs/improve-xyz
# ... update docs ...
git commit -m "docs: clarify PR workflow process
- Add quick start guide
- Include both gh and curl examples
- Explain Hermes-specific tips"
git push -u origin HEAD
# Create PR (docs usually have lighter CI requirements)
```

## Troubleshooting

**Authentication issues:**
- Ensure token has `repo` and `workflow` scopes
- Check `git credential approve` is working
- For SSH: verify key is added to GitHub and `ssh -T git@github.com` works

**CI failures:**
- Use `gh pr checks` to see details
- Check logs with `gh run view RUN_ID --log-failed`
- Common issues: formatting, missing imports, test flakiness

**PR conflicts:**
```bash
git checkout main && git pull origin main
git checkout YOUR_BRANCH
git merge main  # or: git rebase main
# Resolve conflicts, then:
git add .
git commit -m "fix: resolve merge conflicts"
git push -f  # Only if you're the only one working on this branch
```

## Best Practices

1. **Keep PRs focused** - one feature/fix per PR
2. **Write clear descriptions** - explain why, not just what
3. **Reference issues** - always link to related issues
4. **Run tests locally** - catch obvious issues before CI
5. **Update documentation** - if your change affects usage, update docs
6. **Follow existing patterns** - match the codebase style and conventions
7. **Be responsive** - address review comments promptly

## Getting Help

If you're stuck:
- Check the [CONTRIBUTING.md](CONTRIBUTING.md) file
- Search past discussions: `hermes session_search "your topic"`
- Ask in the Nous Research Discord
- Look at recent PRs for examples

Remember: small, well-documented PRs are easier to review and merge than large, complex ones. When in doubt, break your work into smaller chunks.