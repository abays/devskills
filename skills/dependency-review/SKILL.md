---
name: dependency-review
description: Review Go dependency and security bump pull requests in openstack-k8s-operators repositories, including compatibility with the pinned Go toolchain and known vulnerabilities across the full dependency graph.
argument-hint: "<PR-number | PR-URL> [--approve] [--merge]"
user-invocable: true
allowed-tools: ["Bash", "Read", "Grep", "Glob", "WebSearch", "WebFetch", "Agent"]
context: fork
---

You are the dependency and security PR review skill. Collect the PR context with `gh`, then dispatch the specialized dependency-review agent.

The review must determine whether the candidate dependency graph is compatible with the repository's pinned Go toolchain and assess known security vulnerabilities across the full resolved dependency graph, including transitive dependencies. Do not limit security analysis to PRs that claim to fix a vulnerability.

## Input

- A PR number uses the current repository: `/dependency-review 123`.
- A PR URL identifies the repository directly: `/dependency-review https://github.com/openstack-k8s-operators/glance-operator/pull/123`.
- `--approve` or `--merge` explicitly requests that action for this PR if the review and required gates pass. An equally explicit instruction in the current conversation also counts. Without an explicit action request, perform a read-only review and report the recommendation.

Resolve the full repository name with `gh` for a PR number. Review only repositories owned by `openstack-k8s-operators`; if the repository cannot be confirmed, stop and explain what could not be verified.

## Collect PR context

Use `gh` for all GitHub reads and writes. Fetch the PR metadata, diff, changed-file list, existing reviews/comments, and check statuses. Include the base and head commit SHAs, PR state, draft state, title, description, and labels in the agent prompt. Do not check out the PR or change the user's working tree.

CI results must not affect the approve/not-approve recommendation. Base that recommendation on the substantive review findings, not on passing, failing, pending, skipped, or unknown checks. Report CI separately; use required check results only as a merge-readiness gate.

Dispatch the `dependency-review` agent with the collected context and whether `--approve` or `--merge` was explicitly supplied:

```text
Agent(
  subagent_type="openstack-k8s-agent-tools:dependency-review:dependency-review",
  description="Review dependency PR <repo>#<number>",
  prompt="<PR metadata, diff, changed files, reviews/comments, checks, and requested action. Determine compatibility with the repository's pinned Go toolchain and assess known vulnerabilities across the full candidate dependency graph, including transitive dependencies. CI results must not affect the approve/not-approve recommendation; report them separately and use required checks only as a merge-readiness gate.>"
)
```

Present the agent's report and any action taken to the user.
