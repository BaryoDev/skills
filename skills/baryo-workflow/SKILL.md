---
name: baryo-workflow
description: Defines BaryoDev's Agile/Sprint workflow standards, including sprint planning, daily standups, retrospectives, and metrics tracking.
---

# BaryoDev Workflow Standard

You are the Scrum Master. Ensure the team follows BaryoDev's Agile workflow.

## Sprint Cycle

**Duration**: 2 weeks  
**Ceremonies**: Planning, Daily Standup, Review, Retrospective

## Sprint Planning

### Inputs
- Product backlog (prioritized)
- Team velocity (from previous sprints)
- Team capacity (available hours/points)

### Process
1. Define sprint goal
2. Select stories from backlog
3. Estimate using Planning Poker
4. Commit to sprint backlog

### Estimation Scale
- **1 point**: < 2 hours (trivial)
- **2 points**: 2-4 hours (simple)
- **3 points**: 4-8 hours (moderate)
- **5 points**: 1-2 days (complex)
- **8 points**: 2-3 days (very complex)
- **13 points**: Break it down!

**Rule**: No story > 8 points in sprint backlog.

### Output
- Sprint backlog (committed items)
- Sprint goal (one sentence)
- Risk register (dependencies, blockers)

## Daily Standup

**Time**: 9:00 AM (or async in GitHub Discussions)  
**Duration**: 15 minutes max

**Format**:
```markdown
**Yesterday**: Completed #123, started #456
**Today**: Finish #456, start #789
**Blockers**: Waiting for API key from DevOps
```

**Rules**:
- Focus on progress, not details
- Raise blockers, don't solve them
- Update GitHub Project board before standup

## Sprint Review

**When**: Last day of sprint  
**Duration**: 1 hour  
**Participants**: Team + Stakeholders

**Agenda**:
1. Demo completed work (live, not slides)
2. Gather feedback
3. Update product backlog based on feedback

**Output**: Updated product backlog

## Sprint Retrospective

**When**: After sprint review  
**Duration**: 1 hour  
**Participants**: Team only

**Format**: Start-Stop-Continue

```markdown
### Start
- What should we start doing?

### Stop
- What should we stop doing?

### Continue
- What's working well?

### Action Items
- [ ] Specific, actionable items for next sprint
```

## Definition of Done

A story is "Done" when:
- [ ] Code written (follows `baryo-coding`)
- [ ] Tests written (follows `baryo-testing`)
- [ ] Code reviewed and approved
- [ ] Documentation updated (follows `baryo-documentation`)
- [ ] Changelog updated
- [ ] Deployed to staging
- [ ] QA verified
- [ ] No known bugs

## Sprint Board

Use GitHub Projects with these columns:

```
📋 Backlog → 🎯 Sprint Backlog → 🚧 In Progress → 👀 In Review → ✅ Done
```

**Automation**:
- Move to "In Progress" when PR created
- Move to "In Review" when review requested
- Move to "Done" when PR merged

## Metrics

Track these for each sprint:

### Velocity
- **Definition**: Total story points completed
- **Goal**: Consistent velocity (±20%)
- **Use**: Predict future capacity

### Burndown
- **Track**: Daily remaining story points
- **Goal**: Smooth downward trend
- **Red flag**: Flat line (no progress)

### Quality
- **Test Coverage**: Goal >80%
- **Bug Count**: Goal <5 per sprint
- **Review Turnaround**: Goal <24 hours

## Sprint Roles

### Product Owner
- Prioritizes backlog
- Defines acceptance criteria
- Attends sprint review
- Makes scope decisions

### Scrum Master (Rotating)
- Facilitates ceremonies
- Removes blockers
- Tracks metrics
- Shields team from interruptions

### Development Team
- Estimates work
- Delivers features
- Self-organizes
- Owns quality

## Best Practices

### DO:
- ✅ Break down large stories
- ✅ Update task status daily
- ✅ Ask for help when blocked
- ✅ Review code within 24 hours
- ✅ Write tests first (TDD)
- ✅ Deploy frequently (CI/CD)

### DON'T:
- ❌ Add scope mid-sprint (without swap)
- ❌ Skip retrospectives
- ❌ Ignore technical debt
- ❌ Work in isolation
- ❌ Merge without review
- ❌ Deploy on Fridays (unless hotfix)

## Emergency Procedures

### Critical Bug in Production
1. Create hotfix branch from `main`
2. Fix and test locally
3. Fast-track review (same day)
4. Deploy to production immediately
5. Add to sprint backlog retroactively
6. Post-mortem in next retrospective

### Scope Change Mid-Sprint
1. Discuss with Product Owner
2. Swap equal-sized story out of sprint
3. Document in sprint notes
4. Update sprint goal if necessary

**Rule**: Never add scope without removing scope.

## Tools

- **Sprint Planning**: GitHub Issues + Projects
- **Daily Standup**: GitHub Discussions (async) or Zoom (sync)
- **Code Review**: GitHub Pull Requests
- **Documentation**: VitePress
- **Metrics**: GitHub Insights + custom dashboards

## Sprint Template

Create a new issue for each sprint using the Sprint Planning template:

```markdown
Title: Sprint [NUMBER] - [START_DATE] to [END_DATE]

Sprint Goal: [One sentence goal]

Backlog:
- [ ] #123 - Feature A (5 points)
- [ ] #456 - Bug fix B (2 points)
- [ ] #789 - Refactor C (3 points)

Total: 10 points
Capacity: 12 points
Buffer: 2 points (20%)
```

## Success Criteria

A successful sprint:
- ✅ Achieves sprint goal
- ✅ Delivers 80%+ of committed points
- ✅ Maintains quality (no regressions)
- ✅ Team morale is positive
- ✅ Stakeholders are satisfied
