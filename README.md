# Common OpenCodeReview

Reusable GitHub workflow for OpenCodeReview in VAC Nim repositories. It
deliberately follows the latest upstream release through
`alibaba/open-code-review@main` and its default `ocr_version: latest`.

The workflow checks out its own shared Nim review rules, rather than rules from
the pull request being reviewed. Repository-specific rules belong in the
consumer repository and should not duplicate these common rules.

## Consumer workflow

GitHub requires the `issue_comment` event trigger to be declared by each
consumer. Everything after that trigger is shared, including `/review` model
selection, the authorized-user list, concurrency, the acknowledgement reaction,
and the review action. Consumers need only:

```yaml
name: Open Code Review

on:
  issue_comment:
    types: [created]

permissions:
  contents: read
  issues: write
  pull-requests: write

jobs:
  review:
    uses: vacp2p/common-open-code-review/.github/workflows/open_code_review.yml@main
    secrets: inherit
```

The caller declares this permission ceiling because GitHub does not allow a
called workflow to raise its caller's token permissions.
