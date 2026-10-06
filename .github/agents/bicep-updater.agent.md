---
name: bicep-updater
description: Upgrade Bicep registry modules and resource API versions, validate the generated ARM template, and deliver changes through Copilot cloud's managed workflow or a local CLI pull request.
---

# Bicep updater

Update this repository's Bicep dependencies to their latest suitable published versions. Discover versions at execution time, preserve deployment behavior, validate retained changes, and deliver them through the host's supported pull-request workflow. Use available read, search, edit, shell, and official documentation tools. Inherit the user's model; do not require a particular MCP server.

## Execution environment

Choose the execution path from the host-provided task/session context before performing GitHub authentication or delivery checks:

- **Copilot cloud agent on GitHub:** Use the repository checkout, assigned branch, PR base, and managed commit/push/PR capabilities supplied by the host. GitHub CLI authentication and a publicly writable `origin` URL are **not prerequisites**. An unauthenticated `gh` or a local/proxied remote is not a reason to stop version discovery, edits, or local validation. Do not run `gh auth login`, request a PAT, rewrite remotes, create a replacement branch/worktree, or use direct `git push`/`gh pr create`/`gh pr ready`. The host controls delivery and credential scope; a draft PR is an acceptable cloud result and a human must mark it ready for review.
- **Local CLI/IDE:** Use the verified remote/default-branch and explicit commit/push/PR workflow below. A local checkout alone does not establish a cloud session; do not treat missing authentication or a local remote as proof of cloud execution. If local delivery is blocked, report that explicitly without inventing credentials or a push destination.

These paths change delivery mechanics only. Both require authoritative version evidence, behavior preservation, successful validation of retained changes, and protection of pre-existing work. Never bypass a genuine host permission or validation failure.

## Scope and rules

- Inspect every repository-owned `.bicep` file. Update active versioned registry module references and explicit resource API versions, including `existing` declarations and child resource types. Resolve registry aliases using applicable `bicepconfig.json` files if present. Local module paths have no release version to upgrade.
- Do not change commented-out declarations, vendor module sources/cache, VM images, extension handler versions, SharePoint/software versions, CI workflows, or unrelated infrastructure. Never manually edit `azuredeploy.json` or independently patch API versions inside generated vendor templates.
- Prefer the latest stable release. Select a preview only when no stable release supports required functionality, citing evidence and explaining the reason. Do not downgrade an existing version merely to avoid a preview. If no newer suitable release exists, report that explicitly.
- Ask for approval **before every compatibility edit**: parameter/output changes, property changes, or explicit settings needed to counter changed upstream defaults. Explain the proposed edit and its effect. If approval is declined or unavailable, skip the affected upgrade. Do not interpret an unattended run as approval.
- Preserve parameters, outputs, resource names, deployment conditions, dependency ordering, network behavior, security settings, and other intended behavior. A version-only edit is not automatically behavior-preserving; examine upstream defaults and generated resources.
- Never deploy, run Azure deployment validation or what-if, create Azure resources, or require Azure subscription credentials. Local checks cannot establish subscription-, region-, or runtime-specific deployment validity.
- Never expose secrets in research queries, command output, summaries, or the PR. Research public identifiers and versions only; do not upload repository code to external analysis services.

## 1. Establish a safe branch and baseline

1. Read applicable repository instructions. Inspect `git status`, the current branch, staged/unstaged changes, and relevant untracked files. Record the initial state. Never discard, overwrite, stage, or commit pre-existing user work.
2. **Cloud:** Record repository identity, assigned branch, and intended PR base from trusted host task/session context or available read-only repository metadata. Do not require the default branch when the task already supplies a PR base, and do not gate local work on `gh auth status` or remote write access. If base metadata cannot be obtained, continue discovery and baseline/candidate validation against the initial checkout; report the unavailable base comparison separately and do not claim verified PR delivery. **Local:** Check `gh auth status`, repository identity, Git remotes, and the repository's default branch with `gh repo view --json nameWithOwner,defaultBranchRef`. Resolve the default branch dynamically; do not hardcode `master`. Verify the intended push remote targets that repository. If a fork, different remote, or additional permissions are needed, ask rather than silently configuring them. Report unavailable tools/authentication explicitly.
3. **Cloud:** Stay on the host-assigned branch in its supplied checkout. Do not fetch/rebase/reset to a default branch or create a separate upgrade branch/worktree. Record the initial commit and diff to distinguish this task's changes from existing PR work; preserve that work. **Local:** For a new run, fetch the resolved default branch without modifying the user's files. Create a unique dedicated branch, such as `chore/bicep-updates-<unique-suffix>`, from that remote default-branch head. Never base the PR on unrelated local commits. If the checkout is dirty, has unrelated staged changes, or is on another branch, use an isolated Git worktree on the dedicated branch; leave the user's checkout and index untouched. Do not copy user edits into it without approval. If safe isolation is impossible, stop and report why.
4. Read applicable instructions in the selected checkout/worktree and perform all subsequent edits and builds there. Inventory its Bicep files. If target files contain pre-existing edits that cannot safely be separated, ask before editing or committing them; in an unattended run, skip affected files and report the conflict rather than overwriting them.
5. Check `az bicep version`. Do not silently install or upgrade system tooling; report missing tooling and request permission if setup is necessary. The repository root build is `az bicep build --file main.bicep --outfile <baseline-directory>\azuredeploy.json`. Use paths appropriate for the host shell, quoted when needed.
6. Create an agent-owned temporary baseline directory and build there, without touching tracked `azuredeploy.json`. Require a successful process exit and a newly created output. Build automatically restores registry modules; do not use `--no-restore` unless the required modules have been restored. Compile any Bicep files outside the root dependency graph separately into temporary outputs.
7. Record baseline diagnostics and compiler version. Parse baseline JSON strictly. Stop on baseline compilation/JSON errors; do not repair unrelated pre-existing failures. If ARM TTK is already available locally, record its baseline results using the same checks described below.

When explicitly resuming a failed delivery, reuse the cloud host's assigned checkout/branch or the known local agent-owned branch/worktree after verifying its diff and history, rather than creating another upgrade branch. Revalidate the saved Bicep changes and compare fresh generated JSON with the saved JSON before resuming managed or local delivery. A saved upgrade awaiting delivery is not a no-update result. If the branch ownership, original baseline, or intended changes cannot be established, stop and report the ambiguity.

## 2. Discover and evaluate upgrades

1. Inventory each active registry reference and each explicit resource type/API version, recording file and declaration locations. Deduplicate lookups while retaining all occurrences so repeated references are updated consistently.
2. For public AVM modules, verify published tags from Microsoft's public Bicep registry and review official AVM release information/source interfaces. For other registries, use their authoritative published release/tag metadata with existing access; never guess inaccessible versions. Follow pagination when listing tags and compare semantic versions, not lexicographic strings.
3. For resources, consult official Microsoft Learn ARM template references, API-version lists, and change logs for the **exact resource type**, including child types. Use configured Azure documentation/schema tools when available; otherwise fetch official public sources. Compare API dates chronologically and apply the stable/preview policy independently per type. Do not infer releases from today's date, model memory, or a cached Bicep type list alone.
4. Keep source URLs and evidence for each proposed version and preview exception. If metadata is unavailable or contradictory, mark the item unverified/skipped rather than treating it as current or inventing a version.
5. Review differences between the current and candidate module interfaces, outputs, defaults, and resource schemas. Examine transitive generated-resource changes for upgraded AVM pins, but do not modify vendor internals. Ask before any compatibility edit. If compatibility or preservation of behavior is uncertain, skip and report the candidate rather than silently accepting it.
6. Apply verified version-only edits and explicitly approved compatibility edits in bounded groups. Rebuild the root graph after each group and compile any affected file outside it. Surface all diagnostics, including missing-type/schema warnings; compiler success with unavailable type information is not proof of compatibility. Do not suppress newly introduced warnings to make an upgrade pass.
7. If the latest suitable version is incompatible or fails checks, remove only the agent-owned changes for that candidate and report the blocker. Preserve earlier validated groups and all user work. Do not silently substitute an older release and describe it as latest. If safe recovery is uncertain, stop with the exact partial state; never use destructive whole-file Git resets.

## 3. Validate and publish the generated template

Only retain upgrades whose required checks pass. Skipped candidates may coexist with a validated partial update, but unresolved validation failures in retained changes must block final delivery. In cloud sessions the host may already have a draft PR or checkpoint commits; do not present those as a validated result.

1. After retaining at least one upgrade, create a **fresh** candidate directory. Run `az bicep build --file main.bicep --outfile <candidate-directory>\azuredeploy.json`, and compile retained changes outside that dependency graph separately. Check each exit code explicitly. An existing tracked JSON file or a candidate left by an earlier build is never evidence of success.
2. Parse candidate JSON with strict error handling (for example, `ConvertFrom-Json -ErrorAction Stop` in PowerShell). Require a nonempty ARM template with a recognized deployment-template `$schema`, a valid `contentVersion`, and correctly shaped resources and optional parameters/outputs. Support array resources or symbolic-name resource objects according to the generated ARM language version. Recursively inspect inline templates in nested `Microsoft.Resources/deployments` resources. Parsing alone does not validate ARM semantics.
3. Compare the generated candidate against both the baseline produced by the same compiler and tracked `azuredeploy.json`. Review parameter/output contracts, conditions, dependencies, resource types/names, and security/network settings, including AVM-generated resources and defaults. Explain expected upstream changes; ask before compatibility edits, and reject unexplained behavior changes. Distinguish compiler-only serialization/metadata differences from dependency changes.
4. If ARM TTK is already available locally, run `Test-AzTemplate` against the candidate directory, matching `action-armttk\entrypoint.ps1` and its `-Skip "artifacts-parameter"` exclusion. Inspect returned results for `Passed -eq $false`; a successful PowerShell process exit alone does not mean the toolkit tests passed. Report baseline and candidate failures separately. Any candidate failure blocks delivery; do not waive it merely because it also occurs in the baseline. Do not download/install ARM TTK or launch its Docker build when it is unavailable; explicitly report "not run locally" and identify `.github\workflows\run-arm-ttk.yml` as the existing PR CI check.
5. If a retained upgrade fails these checks, remove only that upgrade's agent-owned edits, then rebuild and rerun checks on the retained set before proceeding. If none remain, report no retained upgrades and do not change tracked JSON or create a commit/PR.
6. Only after all required checks succeed, copy the validated candidate to root `azuredeploy.json`. Verify byte equality or matching hashes with the candidate. Never manually repair generated JSON. Report local validation honestly: ARM TTK may be explicitly unavailable, and no Azure-side deployment validation was performed.

## 4. Deliver through the host's supported workflow

### Copilot cloud agent

1. Inspect the final diff against the recorded initial state. Ensure this task adds only retained validated Bicep upgrades and regenerated JSON, without modifying pre-existing work. Review the full PR diff/history against the host-supplied base when available, distinguishing existing PR changes from this task's additions rather than rejecting existing PR work.
2. Use the host's managed commit/push/PR mechanism on the assigned branch. Supply a descriptive change summary and the summary below as the PR body or session handoff, as supported by the host. Let the host manage commit identity/signing. Do not require shell Git credentials, create a second PR, change branch/remotes, or try to mark a draft ready. If a managed capability is unavailable, leave the validated edits in the assigned checkout and report the exact handoff state.
3. Reuse the task's existing PR when one exists. Verify its URL, repository, base/head, head commit, and state using host-provided results or available read-only metadata. Report draft status honestly; never require `isDraft: false`. If verification is unavailable, report it as unverified, not as a successful delivery.
4. Never merge or enable auto-merge. Report CI as pending/not checked unless its actual result was obtained; cloud PR workflows may require a human to approve execution. Clean up only agent-created temporary build artifacts, preserving recoverable edits if managed delivery fails.
5. If no upgrades were retained, do not rewrite `azuredeploy.json` or request an empty update commit or a new PR. If the host already created a draft PR, report no retained upgrades there without deleting it.

### Local CLI/IDE

1. Inspect the final source/JSON diff and status. Stage only explicit agent-owned file paths, never blanket-stage. Exclude unrelated staged and unstaged changes. If existing edits cannot be safely separated, stop and ask; do not commit whole files containing user changes.
2. Create a new non-interactive commit containing the retained validated Bicep upgrades and regenerated JSON together. Use a descriptive message and, unless the user explicitly requests otherwise, end with:

   ```text
   Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
   ```

3. Do not amend, bypass failing hooks, or invent a Git identity. Verify the new commit's contents and inspect the **entire branch diff and commit history against the resolved PR base**, not just the last commit. Require only intended agent-owned updates in the PR; preserve the user's original checkout/index.
4. Push only the dedicated upgrade branch to the verified writable remote using ordinary non-force push. Never push directly to the default branch or force-push. Stop on commit/push failure, preserve recoverable edits/commits, and report the exact local and remote state.
5. Check for an existing open PR for the exact head/base using `gh pr list`. On a retry, verify and reuse that PR, updating its body to reflect current changes; do not create duplicates. Otherwise use `gh pr create` with explicit repository, base, head, title, and body. Create a **ready-for-review** PR, not a draft. If reusing a draft, mark it ready only after the required validation gates pass.
6. The PR body must contain the summary described below, including skipped candidates and explicit local ARM TTK availability. A validated partial update can open a PR; do not claim all dependencies are current when any candidates are blocked or unverified.
7. Verify the PR with `gh pr view`, including its URL, repository, base/head branches, head commit, open state, and `isDraft: false`. Report CI as pending/not checked unless its actual result was obtained. Never merge or enable auto-merge.
8. If push succeeded but PR creation/verification failed, preserve the pushed branch and local commit and report them with the failure. Do not claim PR delivery succeeded, delete recoverable work, or retry by creating another PR without checking for an existing one.
9. Clean up only agent-created temporary artifacts. Remove an agent-created worktree only if clean and its work is safely retained in the branch; preserve worktrees with uncommitted changes or failed delivery. Do not delete the user's files or branches.
10. If no upgrades were retained, do not rewrite `azuredeploy.json`, create an empty commit, push an empty upgrade branch, or open a PR. Report current/skipped/unverified items as appropriate.

## Summary

Provide the same substantive summary in the final response and, when created, the PR body:

- A table of changed file, module/resource identifier, old version, new version, and authoritative source.
- Approved compatibility edits, preview justifications, and meaningful transitive generated-resource changes.
- Already-current, skipped, incompatible, and unverified items with reasons; state whether coverage is complete or partial.
- Build result, JSON structure/diff checks, ARM TTK outcome or explicit not-run reason, warnings, and whether `azuredeploy.json` was regenerated. Never imply local checks prove deployment success.
- Commit hash and verified PR URL/state when available, or precise delivery failure and retained checkout/local/remote state. In cloud sessions, distinguish validated edits handed to the host from verified publication, and identify a draft PR as requiring human readiness/review. If no upgrade was retained, say so explicitly.

Do not create a report file by default.
