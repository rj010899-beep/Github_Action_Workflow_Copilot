# PR Review Agent - Quick Reference

## 🚦 When Does It Run?

| Event | Description |
|-------|-------------|
| `pull_request: opened` | New PR created |
| `pull_request: synchronize` | New commits pushed |
| `pull_request: reopened` | PR reopened |
| Comment with keywords | "review this", "check this", "security issue" |

## 🎯 What It Checks

### 🚨 Security (Critical)
```
✓ Hardcoded secrets/API keys
✓ SQL/Command/XSS injection
✓ Path traversal vulnerabilities
✓ Weak cryptography
✓ Exposed sensitive data
✓ Vulnerable dependencies
✓ Insecure configurations
```

### ⚙️ Code Quality
```
✓ Logic errors & edge cases
✓ N+1 queries & performance
✓ Error handling gaps
✓ Memory leaks
✓ Code complexity
✓ Race conditions
```

### 💥 Breaking Changes
```
✓ API signature changes
✓ Data format changes
✓ Environment variable changes
✓ Behavior modifications
```

## 🏷️ Labels Applied

| Label | Applied When |
|-------|--------------|
| `security` | Security vulnerability found |
| `bug` | Logic error detected |
| `breaking-change` | Breaking change detected |
| `needs-review` | Requires manual review |
| `performance` | Performance issue found |
| `code-quality` | Code quality issue |
| `documentation` | Docs need update |
| `enhancement` | Improvement suggestion |

## 📖 Example Review Output

### Finding Security Issues
```markdown
## 🔍 Code Review Findings

### 🚨 Security Issues
- **[auth.js:42]**: Hardcoded API key detected
  - **Severity**: Critical
  - **Issue**: API key exposed in source code
  - **Recommendation**: Use environment variables
  - **Reference**: CWE-798
```

### Clean PR
```markdown
## ✅ Code Review Complete

This PR looks good! No security vulnerabilities or code quality issues detected.

**Reviewed:**
- Security: ✓
- Code Quality: ✓
- Error Handling: ✓
- Performance: ✓
```

## 🎛️ Command Reference

### Request Manual Review
Comment on PR:
```
@pr-review please review this
```

### Run Workflow Manually
```bash
gh workflow run pr-review.yml
```

### Check Workflow Status
```bash
gh run list --workflow=pr-review.yml --limit 5
```

### View Latest Run
```bash
gh run view $(gh run list --workflow=pr-review.yml --limit 1 --json databaseId --jq '.[0].databaseId')
```

## 🛠️ Configuration Files

| File | Purpose |
|------|---------|
| `.github/Agents/PR-Review.md` | Agent instructions & rules |
| `.github/workflows/pr-review.yml` | Workflow trigger configuration |

## 📝 For PR Authors

### Before Opening PR
- [ ] Run local linters
- [ ] Add/update tests
- [ ] Update documentation
- [ ] Link related issues

### After Agent Review
- [ ] Address security issues immediately
- [ ] Fix bugs before merging
- [ ] Document breaking changes
- [ ] Consider performance suggestions

## 🔧 For Maintainers

### Customize Security Checks
Edit `.github/Agents/PR-Review.md`:
```markdown
#### Custom Security Checks
- ❌ Project-specific secret patterns
- ❌ Required middleware usage
- ❌ Audit logging requirements
```

### Adjust Trigger Conditions
Edit `.github/workflows/pr-review.yml`:
```yaml
on:
  pull_request:
    types: [opened, synchronize]
    paths:
      - 'src/**'
      - '!docs/**'
```

### Update Allowed Labels
Edit frontmatter in `PR-Review.md`:
```yaml
safe-outputs:
  add-labels:
    allowed: [security, bug, custom-label]
    max: 5
```

## 🐛 Common Issues

| Problem | Solution |
|---------|----------|
| Agent not commenting | Check workflow permissions |
| Too many/few comments | Adjust `max` in safe-outputs |
| Wrong labels applied | Update `allowed` labels list |
| Rate limit errors | Reduce review frequency |

## 💡 Tips

1. **Agent focuses on logic, not style** - Use linters for formatting
2. **Agent is conservative** - Only flags high-confidence issues
3. **Draft PRs get lighter review** - Enable draft mode for WIP
4. **Large PRs (>20 files)** - Agent suggests splitting them
5. **Agent learns from feedback** - Report false positives

## 🔗 Quick Links

- [Full Documentation](./README.md)
- [Agent Definition](./PR-Review.md)
- [Workflow File](../workflows/pr-review.yml)
- [GitHub Actions](https://github.com/features/actions)

## 📞 Support

- **Issues**: Open a GitHub issue
- **Questions**: Use GitHub Discussions
- **Security**: Follow security policy

---

**Last Updated**: 2026-05-13
