# PR Review Agent Setup Guide

This directory contains a comprehensive PR Review Agent that automatically analyzes pull requests for code quality, security vulnerabilities, and project standards.

## 📋 Overview

The PR Review Agent is triggered when:
- A new pull request is opened
- New commits are pushed to an existing PR
- A PR is reopened
- Someone mentions the agent in a comment with keywords like "review this", "check this", or "security issue"

## 🏗️ Architecture

### Agent Definition
- **File**: [PR-Review.md](./PR-Review.md)
- **Type**: GitHub Copilot Agent
- **Purpose**: Comprehensive code review with security and quality analysis

### Workflow
- **File**: [../workflows/pr-review.yml](../workflows/pr-review.yml)
- **Trigger**: Pull request events, comments
- **Permissions**: Read contents, write to pull requests and issues

## 🔍 Review Capabilities

### Security Analysis
The agent checks for:
- ✅ Hardcoded credentials and API keys
- ✅ SQL/Command/XSS injection vulnerabilities
- ✅ Path traversal attacks
- ✅ Weak cryptography
- ✅ Exposed sensitive data in logs
- ✅ Vulnerable dependencies
- ✅ Insecure configurations

### Code Quality Checks
- ✅ Logic errors and edge cases
- ✅ Performance issues (N+1 queries, memory leaks)
- ✅ Error handling completeness
- ✅ Code complexity and duplication
- ✅ Race conditions and concurrency issues

### Breaking Changes Detection
- ✅ Public API modifications
- ✅ Data format changes
- ✅ Environment variable changes
- ✅ Behavior modifications

### Language-Specific Reviews
Optimized checks for:
- JavaScript/TypeScript
- Python
- Go
- And more...

## 🚀 Setup Instructions

### Prerequisites
1. GitHub repository with Actions enabled
2. GitHub Copilot for Business or Enterprise (for full agent functionality)

### Installation

1. **Copy the Agent files** to your repository:
   ```bash
   .github/
   ├── Agents/
   │   └── PR-Review.md          # Agent definition
   └── workflows/
       └── pr-review.yml          # Workflow file
   ```

2. **Configure Permissions** (if needed):
   - Go to **Settings** → **Actions** → **General**
   - Under "Workflow permissions", ensure:
     - ☑️ Read and write permissions
     - ☑️ Allow GitHub Actions to create and approve pull requests

3. **Enable GitHub Copilot** (Optional - for full AI reviews):
   - Configure GitHub Copilot for your repository
   - Set up the necessary tokens and permissions

### Configuration

#### Customize Review Triggers

Edit `.github/workflows/pr-review.yml`:

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened]
    # Add path filters if needed:
    # paths:
    #   - 'src/**'
    #   - '!docs/**'
```

#### Adjust Label Categories

Edit `.github/Agents/PR-Review.md` frontmatter:

```yaml
safe-outputs:
  add-labels:
    allowed: [security, bug, breaking-change, needs-review, enhancement, documentation, performance, code-quality]
    max: 5
```

#### Customize Review Depth

For lighter reviews on documentation PRs, adjust the agent instructions or add conditional logic in the workflow.

## 📝 Usage

### Manual Trigger

You can manually trigger the review:

```bash
# Via GitHub UI
Go to Actions → PR Review Agent → Run workflow

# Via GitHub CLI
gh workflow run pr-review.yml
```

### Request Review in Comments

Comment on any PR with keywords:
- "review this"
- "check this"  
- "security issue"
- Mention the agent directly

### Review Output

The agent provides structured feedback:

```markdown
## 🔍 Code Review Findings

### 🚨 Security Issues
- [file.js:42]: SQL Injection vulnerability
  - Severity: Critical
  - Issue: User input directly concatenated into SQL query
  - Recommendation: Use parameterized queries
  - Reference: OWASP A03:2021 - Injection

### ⚠️ Code Quality Issues
- [utils.py:120]: Potential memory leak
  - Issue: File handle not closed in error path
  - Suggestion: Use context manager (with statement)
  - Impact: Resource exhaustion under error conditions
```

## 🎯 Best Practices

### For Reviewers
1. Let the agent do the initial review
2. Review agent findings before manual review
3. Focus manual review on business logic and architecture
4. Use agent labels to prioritize reviews

### For Contributors
1. Run local linters before pushing (the agent focuses on logic, not style)
2. Include tests for new features
3. Document complex logic
4. Reference related issues in PR description

### For Maintainers
1. Regularly update the agent instructions based on common issues
2. Add project-specific checks to the agent definition
3. Monitor agent performance and adjust labels
4. Keep the allowed labels list aligned with project needs

## 🔧 Customization

### Add Custom Security Patterns

Edit `PR-Review.md` and add to Phase 2:

```markdown
#### Custom Security Checks
- ❌ Check for project-specific secrets patterns
- ❌ Verify authentication middleware is used
- ❌ Ensure audit logging for sensitive operations
```

### Language-Specific Rules

Add new language rules in Phase 5:

```markdown
**For Rust:**
- Proper lifetime annotations
- No unsafe blocks without justification
- Error handling with Result types
- No panics in library code
```

### Integration with Other Tools

The agent can work alongside:
- **CodeQL**: For deeper static analysis
- **Dependabot**: For dependency updates
- **Linters**: For code style enforcement
- **Test Coverage**: For coverage reporting

## 📊 Monitoring

### Check Workflow Runs
```bash
# List recent runs
gh run list --workflow=pr-review.yml

# View specific run
gh run view <run-id>
```

### Review Agent Performance
Monitor:
- Number of PRs reviewed
- Issues detected by category
- False positive rate
- Developer feedback

## 🐛 Troubleshooting

### Agent Not Triggering
- Check workflow file syntax: `gh workflow view pr-review.yml`
- Verify permissions in repository settings
- Check if PR matches trigger conditions

### No Review Comments Posted
- Verify `GITHUB_TOKEN` has write permissions
- Check if the agent found any issues (it only comments when needed)
- Review workflow logs for errors

### Rate Limiting
If you hit API rate limits:
- Reduce `max` values in `safe-outputs`
- Add concurrency limits in workflow
- Use conditional triggers (e.g., only on ready_for_review)

## 📚 Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/actions)
- [GitHub Copilot Documentation](https://docs.github.com/copilot)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CWE (Common Weakness Enumeration)](https://cwe.mitre.org/)

## 🤝 Contributing

To improve the PR Review Agent:

1. Test changes locally using `workflow_dispatch`
2. Update documentation for new features
3. Add examples for new review patterns
4. Submit changes via PR (the agent will review itself! 🤯)

## 📄 License

This agent configuration is part of your project and follows your project's license.

---

**Questions?** Open an issue or discussion in the repository.
