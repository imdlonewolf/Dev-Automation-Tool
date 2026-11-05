# SE Automator MVP Plan

## 1. Objective
Create a safe, developer-approved local automation assistant for a single GitHub repository that combines:
- .NET 8 backend
- React + Vite frontend
- Jira-driven task intake
- repository analysis before implementation
- local validation and limited self-healing
- draft PR generation
- manual human review and merge

The system must behave like a trustworthy IDE companion, not an autonomous software factory.

---

## 2. Project Goal
Build a minimal viable product that can:
1. accept a Jira ticket number,
2. inspect the repository,
3. validate the requirement,
4. create an implementation plan,
5. wait for developer approval,
6. edit relevant backend and frontend code,
7. run build/test validation,
8. retry only a few times when needed,
9. create a draft PR,
10. stop for final manual review and merge.

---

## 3. Non-Goals
The MVP will intentionally not include:
- automatic deployment
- automatic merge
- multi-repo orchestration
- broad autonomous SDLC execution
- work on ambiguous or untrusted requirements
- hidden or unsafe automation beyond the repo scope

---

## 4. Core Architecture
### Recommended MVP structure
- 1 orchestrator agent
- 1 analysis/planning workflow
- 1 implementation workflow
- 1 validation and PR workflow

### Components
- Task orchestrator
- Jira integration service
- Repository analyzer
- Planner / proposal generator
- Code execution layer
- Validation runner
- Retry/fix controller
- GitHub PR generator
- Approval state tracker

### Architecture principle
Keep the system narrow, auditable, and human-approved. The orchestrator shouldn’t act without explicit milestone approval.

---

## 5. Execution Flow
### Stage 1: Intake
- Developer provides Jira ticket number.
- System fetches issue metadata and description.
- System validates whether the ticket is actionable and within repo scope.
- If invalid or ambiguous, reject it with a reason.

### Stage 2: Analyze
- Inspect repo structure.
- Identify .NET and React areas likely affected.
- Determine likely files and root cause.
- Produce a short analysis summary.

### Stage 3: Plan
- Build implementation plan with:
  - impacted files,
  - backend/frontend scope,
  - validation strategy,
  - risks and assumptions.
- Require developer approval before implementation begins.

### Stage 4: Implement
- Apply minimal code changes.
- Keep scope tightly aligned to issue.
- Avoid unrelated cleanup or feature expansion.

### Stage 5: Verify
- Run local build and test commands.
- Inspect failures and determine probable root cause.
- Retry only within a strict cap (2–3 repair attempts).
- If retry budget is exhausted, stop and report status.

### Stage 6: PR Draft
- After relevant validation passes and approval is granted:
  - prepare a summary,
  - include validation evidence,
  - create a draft PR.
- Do not merge automatically.

### Stage 7: Manual Review and Merge
- Developer reviews PR and decides whether to revise, rerun, or merge.

---

## 6. Human Approval Gates
The workflow must enforce explicit approvals at the following milestones:

1. Plan Gate
   - Approve analysis and implementation plan.
2. Build Gate
   - Approve implementation after code changes are complete.
3. Verify Gate
   - Approve validation result before PR creation or next action.
4. PR Review Gate
   - Review the draft PR and decide whether to proceed or rerun.

No stage is allowed to advance without approval.

---

## 7. Functional Requirements Mapping
### Requirement 1: Jira-driven intake
- Accept ticket ID
- Read requirement details from Jira
- Reject untrusted or incomplete issues

### Requirement 2: Repository analysis before implementation
- Search repo structure before edits
- Determine likely files and modules
- Keep change surface minimal

### Requirement 3: Human-in-the-loop gates
- Plan, build, verify, and PR review approval gates

### Requirement 4: Cross-stack execution
- Edit backend C# code and frontend React code as needed

### Requirement 5: Local validation
- Run build/tests
- Confirm no obvious regressions

### Requirement 6: Limited self-healing
- Detect failure root cause
- Retry only a bounded number of times

### Requirement 7: Draft PR generation
- Summarize change and validation evidence
- Keep PR as draft

### Requirement 8: Manual merge only
- Developer controls final merge decision

### Requirement 9: Reject untrusted requirements
- Reject ambiguous, contradictory, or out-of-scope requests

### Requirement 10: Change summary documentation
- Include a concise summary of what changed and why

---

## 8. Technical Requirements
### Security
- No hardcoded secrets
- GitHub PATs/tokens via environment variables or secure config
- No secrets in logs, commit messages, or generated docs
- Limit network access to required repo and validation tasks

### Repo assumptions
- GitHub repository
- .NET 8 backend
- React + Vite frontend
- Local validation via repo-defined commands

### Execution model
- Local workspace automation layer
- Reads repo files and executes build/test operations
- Works with Git branch operations and PR tooling
- Maintains simple workflow state

---

## 9. Suggested Tech Stack
### Backend
- .NET 8 Web API / service layer
- ASP.NET Core if needed for orchestration APIs
- GitHub API integration
- Jira API integration
- local process execution wrappers

### Frontend
- React + Vite app for developer UI, if the assistant includes a UI layer
- Simple dashboards for approvals and workflow state

### Persistence
- JSON state or SQLite for task tracking
- local workflow logs

### DevOps / repo operations
- git commands for branch creation and PR updates
- local validation tools for build/test execution

---

## 10. MVP Development Plan
### Sprint 1: Foundation
- define workflow states and approval gates
- create repo skeleton
- configure environment and tooling
- set up secure config management

### Sprint 2: Ticket + repo intelligence
- implement Jira ticket intake
- repository analysis module
- rule-based requirement validation

### Sprint 3: Planning + implementation
- plan generation output
- scoped code editing pipeline
- backend/frontend file targeting

### Sprint 4: Validation + repair loop
- build/test runner
- error interpretation
- bounded retry/fix strategy

### Sprint 5: PR generation and developer review
- PR summary generation
- draft PR creation
- manual review merge flow

---

## 11. Risk Register
### Risk: Over-automation
Mitigation:
- hard approval gates
- low retry budget
- narrow scope enforcement

### Risk: Bad Jira ticket quality
Mitigation:
- reject tickets lacking actionable requirements
- validate scope before proceeding

### Risk: Regressions in validation
Mitigation:
- require local build/test evidence
- stop after retry limit if unresolved

### Risk: Unclear repo impact
Mitigation:
- mandatory analysis before edits
- root cause identification before changes

### Risk: Unsafe secret handling
Mitigation:
- env var configuration only
- secret scanning and log hygiene

---

## 12. Success Criteria
The MVP is successful if the assistant can:
- accept a Jira issue number,
- analyze the repo and determine the correct work path,
- propose an implementation plan,
- receive plan approval,
- implement the change in the relevant files,
- validate build/test locally,
- produce a draft PR,
- stop for human review and manual merge.

---

## 13. Immediate Next Steps
1. Create repo scaffold in the current workspace folder.
2. Add configuration files for backend and frontend.
3. Build the workflow state model.
4. Implement Jira intake and validation.
5. Add repo analysis and plan generation.
6. Add build/test validation and retry control.
7. Implement draft PR generation.
8. Verify the system against the spec via end-to-end test scenarios.

---

## 14. Summary
This project should be built as a disciplined, developer-controlled automation assistant. It must accept Jira tasks, analyze the codebase, plan a minimal fix, validate the result locally, create a draft PR, and then stop to let the developer review and merge manually.

The likely best MVP is a single orchestrator architecture with strong safety checks and explicit approval gates rather than a larger autonomous multi-agent system.
