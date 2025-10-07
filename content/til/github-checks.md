---
title: "Mandatory GitHub checks"
date: 2025-10-07T15:30:00+02:00
draft: false
tags: ["github", "CI/CD"]
---

Making conditional workflows work with required status checks

When you make a GitHub status check required in branch protection rules, GitHub expects that check to appear on every workflow. If
your workflow only runs conditionally (e.g., when certain files change), PRs that don't trigger the specific workflow get stuck forever waiting for a status that will never appear.

Instead of relying on workflow job names as status checks, manually create commit statuses using the GitHub API:

- uses: actions/github-script@ed597411d8f924073f98dfc5c65a23a2325f34cd # v8.0.0
  with:
    script: |
      await github.rest.repos.createCommitStatus({
        owner: context.repo.owner,
        repo: context.repo.repo,
        sha: context.payload.pull_request.head.sha,
        state: 'success',  # or 'pending', 'failure'
        context: 'The Required Status Check',  # Must match exactly
        description: 'Some description to see on the PR checks.'
      })

