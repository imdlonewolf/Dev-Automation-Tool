# Engineering Specification Document
## Project Title: Autonomous SWE Agent for IDE Assistance and GitHub PR Orchestration

---

## 1. Executive Summary & Objective
This document defines the MVP architecture for an autonomous software engineering assistant designed to support development work inside an IDE environment. The system is not intended to be a fully autonomous software factory; instead, it is a focused, safety-first coding automation tool for a single repository and a single development workflow.

The primary objective is to reduce repetitive engineering work while maintaining strong human oversight. The agent reads a Jira ticket number, analyzes the repository, plans a code change, edits the relevant .NET and React files, runs validation, and creates a draft pull request. The developer remains responsible for approving key milestones and manually reviewing or merging the final PR.

This MVP specifically targets a single GitHub repository based on a .NET 8 backend and React + Vite frontend architecture. The agent will operate with a clear human-in-the-loop model, limited self-healing behavior, and strict rejection of untrusted or unsafe requirements.

---

## 2. Scope Definition

### 2.1 In Scope for MVP
* Jira ticket-driven issue intake using a ticket number
* Repository analysis before implementation
* Code modification across .NET backend and React frontend
* Local build and test validation
* Limited auto-repair loops when validation fails
* Branch creation and draft PR generation
* Developer milestone approval in chat
* Human review of PR before final merge
* Safety controls for malformed or untrusted requirements

### 2.2 Out of Scope for MVP
* Automatic PR merge
* Automated deployment to staging or production
* Multi-repository orchestration
* Full autonomous SDLC execution without human approval
* Task execution for untrusted or ambiguous requirements
* Broad enterprise workflow automation beyond the single repo scope

---

## 3. MVP Product Goal
The MVP should behave as a dependable IDE companion that helps with development tasks by doing the following:

1. Accept a Jira ticket number as the task trigger.
2. Inspect the repository to understand the relevant code paths and architecture.
3. Produce a concise implementation plan before modifying any files.
4. Apply the requested code changes in the relevant backend and frontend locations.
5. Run local build and unit test validation.
6. Retry only a limited number of fix attempts when code or tests fail.
7. Present the results to the developer for approval.
8. Create a draft PR summarizing the change.
9. Stop before merge; leave merge to the developer.

---

## 4. Functional Requirements

### Requirement 1: Jira-Driven Task Intake
The agent must accept a Jira ticket number as the start signal for a task. It must use the ticket identifier to locate the issue and read the provided requirement details from the Jira description.

The agent must not assume a task is valid based solely on a ticket title. It must analyze the description and determine whether the request is actionable and within the current repository scope.

### Requirement 2: Repository Analysis Before Implementation
Before editing code, the agent must inspect the repository to determine:
* relevant backend and frontend modules,
* likely files involved in the issue,
* existing patterns and conventions,
* the minimal implementation surface area.

This analysis phase is mandatory and must happen before any code change is made.

### Requirement 3: Human-in-the-Loop Milestone Gates
The system must enforce explicit review gates in chat. The developer approves each milestone before the workflow can continue.

Required gates:
1. Plan Gate: Developer approves the analysis and implementation plan.
2. Build Gate: Developer approves the implementation once code changes are complete and build is in progress or validated.
3. Verify Gate: Developer approves test/build verification results before PR creation or further action.
4. PR Review Gate: Developer reviews the draft PR and decides whether to rerun the workflow or proceed with final manual merge.

The agent may not skip from one phase to the next without explicit approval.

### Requirement 4: Cross-Stack Code Execution
The agent must be able to read, edit, and validate code across both runtime layers of the repository:
* C# backend code
* React + Vite frontend code

The changes must be aligned to the issue and kept as minimal and localized as possible.

### Requirement 5: Local Validation and Test Execution
The agent must execute local build and test commands relevant to the repository after implementation. It must verify:
* project builds successfully,
* unit tests pass,
* no obvious regressions are introduced.

If validation fails, the agent must inspect error output and identify probable root causes before retrying changes.

### Requirement 6: Limited Self-Healing Loop
If build or test validation fails, the agent may attempt to fix the issue automatically.

The maximum number of fix attempts is capped at 2 to 3 total retry cycles after the initial failed validation. Once the retry cap is reached, the agent must stop and report the status to the developer instead of continuing blindly.

### Requirement 7: Draft PR Generation
The agent must create a draft pull request after successful validation and approval. The PR should contain:
* summary of what changed,
* validation evidence,
* issue context,
* any assumptions or follow-up notes.

The agent must not merge the PR automatically.

### Requirement 8: Manual Merge by Developer
PR merge is manual. The developer is responsible for final merge approval after reviewing the draft PR and checking if it is correct and safe.

### Requirement 9: Untrusted Requirements Rejection
If the Jira ticket or description is ambiguous, incomplete, contradictory, unsafe, or outside the repo’s actual scope, the system must reject the task instead of guessing.

The agent must clearly state the reason why the requirement is considered invalid or untrusted.

### Requirement 10: Change Summary Documentation
The agent should generate a concise change explanation or summary documenting what was changed and why before the final review stage. This can be represented as a structured summary within the PR or as a markdown document if preferred by the development workflow.

---

## 5. Technical & System Requirements

### 5.1 Protocol and Execution Model
The system should be built using a protocol-driven architecture compatible with MCP-style tooling and local IDE orchestration patterns. The agent operates as a local development worker communicating with the developer and repository environment.

### 5.2 Security Model
* No hardcoded secrets
* GitHub PATs or tokens must be stored via environment variables or secure secret management
* No secret material should be embedded in commit messages, logs, or generated documentation
* All network access must be scoped to the project repository and necessary validation tasks

### 5.3 System Boundary
The agent runs as a local workspace automation layer with access to:
* repository files,
* build/test commands,
* Git branch operations,
* GitHub PR creation tooling,
* developer chat or approval interface.

It is a stateless orchestration layer whose actions are driven by the task queue and developer approvals.

### 5.4 Repository Assumptions
The system is designed for a single repository at present:
* GitHub repository
* .NET 8 backend
* React + Vite frontend
* Local validation via repo-defined build/test commands

---

## 6. Execution Flow
The agent executes work in a distinct multi-stage flow. It cannot advance to a later stage without completing the necessary prior work and receiving explicit developer approval.

### Stage 1: Analyze
The agent receives a Jira ticket number and inspects the repo to determine:
* affected project areas,
* likely files to edit,
* root cause or implementation direction,
* potential risks and dependencies.

The system must output a short analysis summary before any modification begins.

### Stage 2: Plan
The agent produces an implementation plan that includes:
* scope of the change,
* files likely involved,
* backend/frontend impacts,
* validation steps,
* risks and assumptions.

The developer approves this plan in chat before implementation continues.

### Stage 3: Build / Implement
The agent makes the code changes required by the plan and keeps the work minimal and scoped to the ticket.

The implementation must not broaden the task scope beyond the issue definition.

### Stage 4: Verify
The agent runs local validation and checks:
* build success,
* test suite success,
* no obvious regressions.

If validation fails, the agent attempts a limited number of code repairs and reruns verification.

### Stage 5: PR Draft
After verification passes and the developer has reviewed the result, the agent creates a draft PR containing the summary and validation evidence.

### Stage 6: Manual Review and Merge
The developer reviews the PR manually and decides whether to:
* request revisions,
* rerun the workflow,
* or merge manually when ready.

No automatic merge is enabled in the MVP.

---

## 7. Human Approval Model
The workflow must include explicit human review in chat at the critical gates. The developer is responsible for approving or rejecting the agent’s actions at each milestone.

The human review model should support the following:
* approve the plan,
* approve the implementation,
* approve the verification outcome,
* review the draft PR,
* request a rerun if necessary.

The agent must surface concise reasoning, evidence, and next steps so the developer can make informed decisions.

---

## 8. Self-Healing and Failure Handling

### 8.1 Failure Detection
The agent must detect and interpret local validation failures such as:
* compile errors,
* test failures,
* broken imports,
* type mismatches,
* runtime assertions.

### 8.2 Repair Strategy
When validation fails, the agent must:
1. inspect the failing output,
2. identify probable root cause,
3. modify the relevant code,
4. rerun validation.

### 8.3 Retry Limit
The repair loop is intentionally limited to 2 to 3 attempts due to operational and credit constraints.

If it fails after the set limit, the system must stop and hand the task back to the developer with a clear summary of the failure and the most likely cause.

This prevents uncontrolled loop behavior and protects the development budget.

---

## 9. Jira Integration Requirements
The system must support the following flow:
* developer provides a Jira ticket number,
* agent reads the issue from Jira or uses the Jira ticket metadata available to the workflow,
* agent analyzes the repo and the requirement,
* agent builds a plan for code change,
* agent updates code and validates it,
* agent drafts PR with the work summary.

The design assumes the Jira description contains the actionable requirement for the task. If it does not, the system must reject the task as incomplete.

---

## 10. MVP Architecture Recommendation
For this project, the ideal architecture is not a large multi-agent system. The preferred MVP structure is:

* 1 orchestrator agent for task coordination and developer communication
* 1 analysis/planning phase
* 1 implementation/build phase
* 1 verification and PR drafting phase

This yields a simple but effective architecture that is easier to operate safely and easier for a developer to trust.

---

## 11. Success Criteria for MVP
The MVP is considered successful when the system can:
* accept a Jira issue number,
* analyze the repository and determine the solution path,
* propose and receive approval for the plan,
* implement the change in the relevant files,
* validate build and tests locally,
* produce a draft PR,
* stop cleanly for human review and manual merge.

---

## 12. Non-Goals and Safety Constraints
This project intentionally does not include:
* automatic deployment,
* automatic merge,
* autonomous execution for ambiguous tasks,
* broad issue automation beyond the described repo and workflow,
* trust-based action on invalid requirements.

Safety is prioritized over automation breadth. This is a development helper, not an unrestricted autonomous engineer.

---

## 13. Final Summary
The MVP should be built as a disciplined, developer-approved automation tool for a single GitHub .NET + React repository. It accepts a Jira issue number, analyzes the codebase, plans a fix, updates the relevant files, runs local validation, retries only a limited number of times when testing fails, creates a draft PR, and then requires manual developer review and merge.

This keeps the system practical, safe, and aligned with the project’s stated goals: correctness, safety, and meaningful automation coverage without sacrificing human control.

