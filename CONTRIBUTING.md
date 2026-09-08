# Contributing to Secuvon

Thank you for your interest in contributing to Secuvon, the enterprise AI security platform by Nexphase Technologies. This document provides comprehensive guidance for contributing to the platform repository and our open-source projects.

> **Note:** The `secuvon/platform` repository is **proprietary**. External contributions are accepted only under a signed Contributor License Agreement (CLA). The CLI, SDK, benchmark and agent-guard repositories are currently private while they are being prepared for release.

---

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Ways to Contribute](#ways-to-contribute)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Coding Standards](#coding-standards)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)
- [Testing Requirements](#testing-requirements)
- [Documentation](#documentation)
- [Security](#security)
- [Reporting Bugs](#reporting-bugs)
- [Suggesting Features](#suggesting-features)
- [Contributor License Agreement](#contributor-license-agreement)
- [Recognition](#recognition)
- [Getting Help](#getting-help)

---

## Code of Conduct

This project and everyone participating in it is governed by our [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Report unacceptable behavior to [conduct@secuvon.com](mailto:conduct@secuvon.com).

---

## Ways to Contribute

There are many ways to contribute to Secuvon, not all of them involve writing code:

| Contribution Type | Where |
|------------------|-------|
| **Bug reports** | [Contact form](https://secuvon.ai/contact) |
| **Feature requests** | [Contact form](https://secuvon.ai/contact) |
| **Security vulnerabilities** | [security@secuvon.com](mailto:security@secuvon.com), see [SECURITY.md](SECURITY.md) |
| **Documentation improvements** | Pull requests to docs/ folder |
| **Community support** | Answer questions in Discussions |
| **Translations** | i18n contributions for the web UI |
| **Blog posts and case studies** | [community@secuvon.com](mailto:community@secuvon.com) |

---

## Getting Started

### Prerequisites

| Tool | Version | Notes |
|------|---------|-------|
| Git | 2.40+ | Version control |
| Docker Desktop | 4.x+ | Local stack |
| Node.js | 22.x | Frontend dev |
| Python | 3.12 | Backend dev |
| Make | Optional | Convenience commands |

### One-Time Setup

1. **Fork the repository** (for open-source repos) or **request access** (for the private platform repo)
2. **Clone your fork:**
   ```bash
   git clone https://github.com/YOUR-USERNAME/platform.git
   cd platform
   ```
3. **Add the upstream remote:**
   ```bash
   git remote add upstream <the repository URL you were given>
   ```
4. **Install Git hooks** (optional but recommended):
   ```bash
   ./tools/scripts/install-hooks.sh
   ```
5. **Bootstrap the local environment:**
   ```bash
   cp .env.example .env
   docker compose up -d
   ```
6. **Verify everything works:**
   ```bash
   curl http://localhost:8000/health
   open http://localhost:3000
   ```

### Project Structure

See [README.md](README.md#repository-structure) for a complete overview of the mono-repo layout.

---

## Development Workflow

### Branch Strategy

We use **trunk-based development** with short-lived feature branches:

```
main (protected, production)
├── develop (staging)
└── feature/your-feature-name (your branch)
```

**Branch naming convention:**

| Type | Prefix | Example |
|------|--------|---------|
| Feature | `feature/` | `feature/sse-real-time-progress` |
| Bug fix | `fix/` | `fix/scan-timeout-on-large-corpus` |
| Documentation | `docs/` | `docs/api-reference-update` |
| Refactor | `refactor/` | `refactor/extract-scanner-engine` |
| Performance | `perf/` | `perf/cache-corpus-loading` |
| Security | `security/` | `security/sanitize-redirect-url` |
| CI/CD | `ci/` | `ci/add-codeql-workflow` |
| Chore | `chore/` | `chore/update-deps` |

### Making Changes

1. **Create a feature branch:**
   ```bash
   git checkout -b feature/amazing-feature
   ```
2. **Make your changes** following our [coding standards](#coding-standards)
3. **Add tests** for any new functionality (see [Testing Requirements](#testing-requirements))
4. **Run tests locally** before committing:
   ```bash
   # Backend
   cd apps/api && pytest

   # Frontend
   cd apps/web && npm test && npx tsc --noEmit && npm run lint
   ```
5. **Commit your changes** with a descriptive message (see [Commit Guidelines](#commit-guidelines))
6. **Push to your fork:**
   ```bash
   git push origin feature/amazing-feature
   ```
7. **Open a Pull Request** (see [Pull Request Process](#pull-request-process))

### Keeping Your Branch Updated

```bash
git fetch upstream
git checkout main
git merge upstream/main
git checkout feature/amazing-feature
git rebase main
```

---

## Coding Standards

### Python (Backend)

We follow [PEP 8](https://peps.python.org/pep-0008/) with these additions:

- **Line length:** 100 characters
- **Type hints:** Required on all function signatures (use `from __future__ import annotations`)
- **Docstrings:** Google style for all public functions and classes
- **Imports:** Grouped (stdlib, third-party, local) and alphabetized within groups
- **Async/await:** Prefer async for any I/O operation
- **Error handling:** Never use bare `except:`; always specify exception types
- **Logging:** Use the module-level `logger`, not `print()`

**Tools:**
- **Linting:** [Ruff](https://docs.astral.sh/ruff/) (`ruff check .`)
- **Formatting:** [Ruff](https://docs.astral.sh/ruff/) (`ruff format .`)
- **Type checking:** [mypy](http://mypy-lang.org/) in strict mode (planned)

**Example:**

```python
"""Scanner service module, bridges the API to the scanning engine."""
from __future__ import annotations

import logging
from typing import Any

from app.scanner.executor import Executor

logger = logging.getLogger(__name__)


async def run_scan(
    scan_id: str,
    agent_id: str,
    modules: list[str] | None = None,
) -> dict[str, Any]:
    """
    Execute a security scan against the specified agent.

    Args:
        scan_id: Public scan identifier (e.g. "ftf-...")
        agent_id: UUID of the agent to scan
        modules: Optional list of module names to filter

    Returns:
        Scan result dictionary with grade, score, and findings

    Raises:
        ValueError: If agent_id is invalid
        RuntimeError: If scanner engine fails to initialize
    """
    logger.info("[%s] Starting scan for agent %s", scan_id, agent_id)
    # implementation...
```

### TypeScript (Frontend)

We use **strict mode TypeScript** with these conventions:

- **Strict null checks:** Always enabled
- **No implicit any:** All types must be explicit
- **Function components:** Use arrow functions for components, named for hooks
- **Props interfaces:** Suffix with `Props` (e.g. `interface ButtonProps`)
- **Hook prefixes:** Always start with `use` (e.g. `useScanProgress`)
- **Tailwind:** Use utility classes; avoid inline styles and CSS modules
- **State management:** Zustand for global state, useState for local

**Tools:**
- **Linting:** ESLint with Next.js + TypeScript configs
- **Formatting:** Prettier (config in package.json)
- **Type checking:** `npx tsc --noEmit`

**Example:**

```tsx
"use client";

import { useState, useCallback } from "react";

interface ScanButtonProps {
  agentId: string;
  onScanStart?: (scanId: string) => void;
}

export default function ScanButton({ agentId, onScanStart }: ScanButtonProps) {
  const [isLoading, setIsLoading] = useState(false);

  const handleClick = useCallback(async () => {
    setIsLoading(true);
    try {
      const response = await fetch("/api/v1/scans/run", {
        method: "POST",
        body: JSON.stringify({ agent_id: agentId }),
      });
      const data = await response.json();
      onScanStart?.(data.scan_id);
    } finally {
      setIsLoading(false);
    }
  }, [agentId, onScanStart]);

  return (
    <button onClick={handleClick} disabled={isLoading}>
      {isLoading ? "Starting..." : "Start Scan"}
    </button>
  );
}
```

### General Standards

- **No console.log in production code**, use proper logging
- **No commented-out code**, delete it; git remembers
- **No magic numbers**, define constants with descriptive names
- **DRY (Don't Repeat Yourself)**, extract shared logic into utilities
- **YAGNI (You Aren't Gonna Need It)**, don't add abstractions for hypothetical futures
- **Boy Scout Rule**, leave the code cleaner than you found it
- **Fail fast**, validate inputs early and raise clear errors

---

## Commit Guidelines

We use [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) for clear, automated changelog generation.

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

| Type | Purpose |
|------|---------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, missing semicolons, etc. (no code change) |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or correcting tests |
| `build` | Changes to build system or dependencies |
| `ci` | Changes to CI configuration |
| `chore` | Other changes that don't modify src or test files |
| `revert` | Reverts a previous commit |
| `security` | Security fix |

### Scopes

Common scopes: `web`, `api`, `corpus`, `scanner`, `guardian`, `compliance`, `auth`, `db`, `ci`, `docker`

### Examples

```
feat(scanner): add multi-turn attack chain support

Implements the multi-turn attack chain executor that maintains
conversation state across multiple turns, enabling realistic
prompt injection attack simulations.

Closes #142
```

```
fix(api): handle 429 rate limits with exponential backoff

The executor was treating 429 responses as successful requests,
causing scans to complete instantly without real testing. Now
retries with exponential backoff (3s, 7s, 15s) up to max_retries.

Fixes #198
Co-Authored-By: Jane Smith <jane@example.com>
```

```
security(web): sanitize redirect URL on login

Prevents open-redirect and XSS attacks via the redirect query
parameter. Now hardcoded to /dashboard regardless of input.

Reported-by: GitHub CodeQL
Severity: High
```

### Subject Line Rules

- Use the imperative mood ("add" not "added" or "adds")
- Don't capitalize the first letter
- No period at the end
- Limit to 72 characters
- Make it explain *what* and *why*, not *how*

---

## Pull Request Process

### Before Opening a PR

- [ ] Code follows our [coding standards](#coding-standards)
- [ ] Tests added for new functionality
- [ ] All tests pass locally
- [ ] Linter and type checker pass with no errors
- [ ] Documentation updated (README, API docs, code comments as needed)
- [ ] Commits follow [Conventional Commits](#commit-guidelines)
- [ ] No sensitive data, secrets, or credentials in commits
- [ ] No `console.log`, `print()`, or `debugger` statements
- [ ] Self-reviewed the diff for obvious issues

### PR Title and Description

**Title:** Use Conventional Commit format
- Good: `feat(scanner): add LLM judge validation for high-severity tests`
- Bad: `Updated scanner`

**Description template:**

```markdown
## Summary
Brief description of what this PR does and why.

## Changes
- Bullet list of specific changes
- Be concrete about what code changed
- Include any new dependencies or schema changes

## Testing
How was this tested? Include:
- Manual testing steps
- New tests added
- Edge cases considered

## Screenshots (if applicable)
Before / After screenshots for UI changes

## Related Issues
Closes #123
Related to #456

## Checklist
- [ ] Tests added and passing
- [ ] Documentation updated
- [ ] Migration scripts (if DB schema changed)
- [ ] Breaking changes documented in changelog
```

### Review Process

| Stage | Owner | Timeline |
|-------|-------|----------|
| Initial automated checks (CI, CodeQL) | Bot | 5-10 minutes |
| First human review | Maintainer | Within 2 business days |
| Address review feedback | Author | At your pace |
| Final review and merge | Maintainer | Within 1 business day after approval |

### Merge Strategy

- **Squash and merge** for feature branches (default)
- **Rebase and merge** for clean linear history (sometimes)
- **Merge commit** for release branches only

The PR title becomes the commit message on `main`. Make it count.

### What Reviewers Look For

1. **Correctness:** Does the code do what it claims?
2. **Tests:** Are there tests, and do they cover edge cases?
3. **Security:** Are inputs validated? Are secrets handled correctly?
4. **Performance:** Are there obvious N+1 queries, unbounded loops, or memory leaks?
5. **Maintainability:** Is the code readable? Are functions appropriately scoped?
6. **API design:** Are public interfaces consistent with existing patterns?
7. **Documentation:** Are non-obvious decisions documented?

---

## Testing Requirements

### Coverage Expectations

| Code Type | Coverage Target | Required For Merge |
|-----------|-----------------|--------------------|
| Backend business logic | 80%+ | Yes |
| Backend API endpoints | 100% (happy + error paths) | Yes |
| Frontend critical components | 70%+ | Recommended |
| E2E flows | All critical user journeys | For major features |
| Edge cases | All identified edge cases | Yes |

### Test Categories

**Unit tests**, Fast, isolated tests of individual functions/components
- Backend: pytest in `apps/api/tests/`
- Frontend: Jest in `apps/web/src/**/__tests__/`

**Integration tests**, Tests that hit a real database or external service
- Backend: pytest with `pytest-asyncio` and asyncpg
- Use test containers or a test database

**E2E tests**, Browser-driven tests of complete user flows
- Playwright in `apps/web/e2e/`
- Run against a fully started stack

### Writing Good Tests

- **One assertion per test** (when reasonable)
- **Descriptive test names:** `test_scan_completes_with_all_modules_when_user_has_pro_plan`
- **Arrange/Act/Assert pattern**
- **Test the behavior, not the implementation**
- **Use fixtures for shared setup**
- **Mock external dependencies, not internal logic**

---

## Documentation

When you change code, update the relevant docs:

| Change | Update |
|--------|--------|
| New API endpoint | Update `apps/api/app/api/routes/` docstrings (auto-generates Swagger) |
| New frontend page | Add to `apps/web/src/app/` and update navigation |
| New env variable | Update `.env.example` and README configuration table |
| New CLI command | Update `secuvon-cli` README |
| Architecture change | Update README architecture diagram |
| Breaking change | Add to `CHANGELOG.md` and call out in PR description |

### Writing Style

- **Active voice:** "The scanner runs tests" not "Tests are run by the scanner"
- **Present tense:** "The function returns..." not "The function will return..."
- **Concise:** Cut unnecessary words. Be direct.
- **Examples:** Show, don't just tell. Include code samples.
- **Audience awareness:** Write for the reader, not for yourself.

---

## Security

### Reporting Security Vulnerabilities

**Do NOT** open public GitHub issues for security vulnerabilities. Instead, follow our [Security Policy](SECURITY.md) and email [security@secuvon.com](mailto:security@secuvon.com).

### Security in Pull Requests

If your PR has security implications, mark it with the `security` label and:

1. Include a clear description of the security impact
2. Reference any related CVEs or advisories
3. Coordinate with maintainers before public disclosure
4. Add tests that prevent regression

### Pre-Commit Security Checks

Before committing, verify:

- No hardcoded credentials, API keys, or tokens
- No sensitive customer data in test fixtures
- No bypassed authentication checks
- No SQL queries built with string concatenation
- No `eval()`, `exec()`, or shell injection vectors

---

## Reporting Bugs

### Before Reporting

1. **Search existing issues**, your bug may already be reported
2. **Check the documentation**, the behavior may be intentional
3. **Try the latest version**, your bug may already be fixed
4. **Reproduce in isolation**, minimize the test case

### Bug Report Template

```markdown
**Describe the bug**
A clear and concise description of what the bug is.

**To Reproduce**
Steps to reproduce the behavior:
1. Go to '...'
2. Click on '....'
3. Scroll down to '....'
4. See error

**Expected behavior**
What you expected to happen.

**Actual behavior**
What actually happened.

**Screenshots / Logs**
If applicable, add screenshots or paste relevant log output.

**Environment**
- OS: [e.g. macOS 14.5]
- Browser: [e.g. Chrome 120]
- Secuvon version: [e.g. 1.2.3 or commit SHA]
- Deployment: [e.g. local Docker, cloud]

**Additional context**
Any other context about the problem.
```

---

## Suggesting Features

We welcome feature suggestions! Send them through the [contact form](https://secuvon.ai/contact).

### Good Feature Proposals Include

- **The problem** you're trying to solve (not just the solution)
- **Who benefits** from this feature (which user persona)
- **Use cases** with concrete examples
- **Alternatives considered** and why they're insufficient
- **Mockups or sketches** for UI features
- **Compatibility considerations** (breaking changes, migration paths)

---

## Contributor License Agreement

For contributions to the **proprietary platform repository**, all contributors must sign our Contributor License Agreement (CLA) before their first PR is merged. This is automated via the CLA Assistant bot, you'll be prompted on your first PR.

For contributions to **open-source repositories** (CLI, SDK, benchmark, agent-guard), you grant Nexphase Technologies a license to your contribution under the repository's Apache 2.0 license.

If your contribution includes work done as part of your employment, ensure your employer signs the corporate CLA before submitting.

---

## Recognition

We believe in recognizing all contributions. Contributors are acknowledged in:

- **CONTRIBUTORS.md**, auto-generated list of all contributors
- **Release notes**, significant contributions called out by name
- **Annual report**, top contributors recognized in our yearly retrospective
- **Conference speaking opportunities**, invited to represent Secuvon at events

### Contributor Roles

As contributors demonstrate sustained engagement, they may be invited to take on expanded roles:

| Role | Responsibilities | Recognition |
|------|------------------|-------------|
| Contributor | Submit PRs, file issues, help in discussions | Listed in CONTRIBUTORS.md |
| Triager | Triage incoming issues, label PRs | Triage permissions |
| Reviewer | Review PRs in their area of expertise | PR review permissions |
| Maintainer | Merge PRs, manage releases | Write access to repos |
| Steward | Guide project direction, govern community | Voting member of steering committee |

---

## Getting Help

| Need | Where |
|------|-------|
| **General questions** | [Contact form](https://secuvon.ai/contact) |
| **Documentation** | [secuvon.ai](https://secuvon.ai) |
| **Bug reports** | [Contact form](https://secuvon.ai/contact) |
| **Security issues** | [security@secuvon.com](mailto:security@secuvon.com) |
| **Conduct issues** | [conduct@secuvon.com](mailto:conduct@secuvon.com) |
| **Enterprise/legal** | [legal@secuvon.com](mailto:legal@secuvon.com) |

### Response Expectations

| Channel | Typical Response Time |
|---------|----------------------|
| Discord | Within hours during business days |
| GitHub Discussions | Within 1-2 business days |
| GitHub Issues | Triaged within 2 business days |
| Pull Requests | First review within 2 business days |
| Security reports | Acknowledged within 24 hours |

---

## Thank You

Every contribution makes Secuvon better, whether it's code, documentation, bug reports, or community support. Thank you for being part of building a more secure AI ecosystem.

---

<div align="center">

**Secuvon**, Building trust in AI, one scan at a time.

*by Nexphase Technologies*

[Code of Conduct](CODE_OF_CONDUCT.md) · [Security Policy](SECURITY.md) · [License](README.md#license)

</div>
