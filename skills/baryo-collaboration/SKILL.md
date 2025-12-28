---
name: baryo-collaboration
description: Defines BaryoDev's collaboration standards for pull requests, code reviews, and team communication.
---

# BaryoDev Collaboration Standard

You are the Team Lead. Ensure effective collaboration through high-quality code reviews and clear communication.

## Pull Request Standards

### PR Size Guidelines
- **Small**: <100 lines (ideal)
- **Medium**: 100-500 lines (acceptable)
- **Large**: 500-1000 lines (break down if possible)
- **XL**: >1000 lines (MUST break down)

**Rule**: Aim for small, focused PRs.

### PR Template (Mandatory)
Every PR MUST include:
- **Description**: What and why
- **Type of Change**: Bug fix, feature, refactor, etc.
- **Testing**: How it was tested
- **Checklist**: All items completed
- **Screenshots**: For UI changes
- **Benchmarks**: For performance changes

### PR Title Format
```
<type>: <description>

Examples:
feat: add circuit breaker strategy
fix: resolve memory leak in retry logic
docs: update installation guide
perf: optimize collection mapping
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `perf`: Performance improvement
- `refactor`: Code refactoring
- `test`: Test updates
- `chore`: Build/tooling changes

## Code Review Process

### Timeline
- **Request Review**: Immediately after PR creation
- **First Response**: Within 24 hours
- **Complete Review**: Within 48 hours
- **Merge**: After approval + CI pass

### Review Checklist

#### Functionality
- [ ] Code does what it's supposed to do
- [ ] Edge cases handled
- [ ] Error handling appropriate
- [ ] No obvious bugs

#### Code Quality (baryo-coding)
- [ ] Follows BaryoDev standards
- [ ] No LINQ in hot paths (libraries)
- [ ] Proper struct vs class usage
- [ ] No unnecessary allocations
- [ ] Async/sync parity

#### Testing (baryo-testing)
- [ ] Unit tests cover new code
- [ ] Tests are meaningful
- [ ] Benchmarks included (if perf-critical)
- [ ] Integration tests (if needed)

#### Documentation (baryo-documentation)
- [ ] XML comments on public APIs
- [ ] VitePress docs updated
- [ ] Changelog updated
- [ ] README updated (if needed)

#### Security (baryo-compliance)
- [ ] No secrets in code
- [ ] License headers present
- [ ] No injection vulnerabilities
- [ ] Input validation present

### Review Comments

#### For Reviewers

**Constructive Feedback**:
```markdown
✅ "Consider using a struct here for better performance. See baryo-coding skill."
✅ "This could be simplified: [code suggestion]"
✅ "Great use of Expression Trees! 🎉"
```

**Avoid**:
```markdown
❌ "This is wrong."
❌ "Why did you do it this way?"
❌ "I would never write code like this."
```

**Comment Prefixes**:
- `[nit]`: Minor suggestion, not blocking
- `[question]`: Asking for clarification
- `[blocking]`: Must be addressed before merge
- `[praise]`: Positive feedback

#### For Authors

**Responding to Feedback**:
```markdown
✅ "Good catch! Fixed in abc123"
✅ "I chose this approach because [reason]. Open to alternatives."
✅ "Thanks for the suggestion! Applied."
```

**Avoid**:
```markdown
❌ "This is how I always do it."
❌ "The tests pass, so it's fine."
❌ "I don't have time to change this."
```

### Review Priorities

**P0 (Must Fix)**:
- Security vulnerabilities
- Breaking changes without migration
- Failing tests
- Performance regressions

**P1 (Should Fix)**:
- Code style violations
- Missing tests
- Incomplete documentation
- Unclear naming

**P2 (Nice to Have)**:
- Refactoring suggestions
- Optimization opportunities
- Alternative approaches

## Approval Requirements

### Standard PR
- **Approvals**: 1 required
- **Who**: Any team member
- **CI**: Must pass

### Critical PR (Production, Breaking Changes)
- **Approvals**: 2 required
- **Who**: Senior developers
- **CI**: Must pass
- **Additional**: Manual QA verification

### Hotfix
- **Approvals**: 1 required (fast-track)
- **Who**: On-call engineer
- **CI**: Must pass
- **Post-merge**: Immediate monitoring

## Merge Strategies

### Squash and Merge (Default)
**When**: Feature branches  
**Why**: Clean history, single commit per feature

```bash
# Results in:
feat: add circuit breaker (#123)
```

### Rebase and Merge
**When**: Clean commit history desired  
**Why**: Preserves individual commits

### Merge Commit
**When**: Release branches  
**Why**: Preserves branch history

## Conflict Resolution

### Prevention
- Pull `main` frequently
- Keep PRs small
- Communicate about overlapping work

### Resolution
```bash
git checkout feature/my-feature
git fetch origin
git rebase origin/main
# Resolve conflicts
git add .
git rebase --continue
git push --force-with-lease
```

## Communication Standards

### Async Communication (Preferred)
- **GitHub Discussions**: General questions
- **GitHub Issues**: Bug reports, features
- **Pull Request Comments**: Code-specific
- **Documentation**: Permanent knowledge

### Sync Communication (When Needed)
- **Daily Standup**: Progress updates
- **Sprint Planning**: Estimation, commitment
- **Pair Programming**: Complex problems
- **Incident Response**: Production issues

### Communication Etiquette

**DO**:
- ✅ Be specific and actionable
- ✅ Provide context and links
- ✅ Use code blocks for code
- ✅ Tag relevant people
- ✅ Follow up on threads

**DON'T**:
- ❌ Use vague language ("it's broken")
- ❌ Ping people unnecessarily
- ❌ Have important discussions in DMs
- ❌ Leave threads unresolved
- ❌ Assume everyone knows context

## Post-Merge Responsibilities

### Author
- [ ] Delete feature branch
- [ ] Close related issues
- [ ] Monitor deployment
- [ ] Update sprint board
- [ ] Notify stakeholders (if needed)

### Reviewer
- [ ] Verify merge
- [ ] Watch for regressions
- [ ] Provide feedback on process

## Metrics

Track for continuous improvement:

- **Review Turnaround**: Goal <24 hours
- **PR Size**: Goal <300 lines
- **Approval Rate**: Goal >90% first review
- **Merge Frequency**: Goal >5 per week
- **Revert Rate**: Goal <5%

## Escalation

### Disagreement on Approach
1. Discuss in PR comments
2. If unresolved, schedule sync call
3. If still unresolved, Product Owner decides
4. Document decision in PR

### Blocked PR
1. Author identifies blocker in PR
2. Scrum Master helps remove blocker
3. If urgent, escalate to Team Lead
4. If critical, fast-track review

## Best Practices

### DO:
- ✅ Review within 24 hours
- ✅ Provide constructive feedback
- ✅ Approve when ready (don't nitpick)
- ✅ Test locally if unsure
- ✅ Ask questions to understand

### DON'T:
- ❌ Approve without reading
- ❌ Block on style preferences
- ❌ Ignore CI failures
- ❌ Merge without approval
- ❌ Leave PRs hanging
