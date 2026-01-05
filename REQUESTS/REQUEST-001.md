# REQUEST-001: Establish initial maintainer request workflow and template usage

Goal
- Establish a clear, reviewable REQUEST workflow in this repo so contributors can propose work, track approvals, and ensure commits only happen with explicit human approval.

Scope
- Add a formal REQUEST document describing the initial workflow and template usage for feature/maintenance requests.
- Provide a concrete example request that follows the repository's REQUEST rules.
- No code changes to runtime code or other files outside REQUESTS/.

Constraints
- Must not commit runtime code or files outside REQUESTS/ without explicit human approval.
- Follow repository rules: filenames REQUEST-XXX.md and include required sections.
- Approval block must reflect human decisions.

Definition of Done
- A REQUEST file exists at REQUESTS/REQUEST-001.md including:
  - Goal, Scope, Constraints, Definition of Done, Affected Repos
  - An Approval block with an explicit Status and AutoCommit setting
- The file follows repository REQUEST rules.

Affected Repos
- vesper-maintainer

Example Request Body
- See above sections; this file itself serves as the example.

Approval
- Status: APPROVED
- AutoCommit: true
- CommitMessage: "REQUEST-001: Add request to establish initial maintainer workflow"
- Scope: ["vesper-maintainer"]

Notes
- This request establishes process and tooling for requester/maintainer coordination.
- Per repo rules, commits will proceed because a human explicitly approved and enabled AutoCommit in this request.
