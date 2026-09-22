# Common OpenCodeReview

Reusable GitHub workflow and composite action for OpenCodeReview in VAC Nim
repositories. It deliberately follows the latest upstream release through
`alibaba/open-code-review@main` and its default `ocr_version: latest`.

The action supplies shared Nim review rules from its own checkout, rather than
from the pull request being reviewed. Repository-specific rules belong in the
consumer repository and should not duplicate these common rules.

## Consumer workflow

GitHub requires the `issue_comment` event trigger to be declared by each
consumer. Everything after that trigger is shared, including `/review` model
selection, the authorized-user list, concurrency, the acknowledgement reaction,
permissions, and the review action. Consumers need only:

```yaml
jobs:
  review:
    uses: vacp2p/common-open-code-review/.github/workflows/open_code_review.yml@main
    secrets: inherit
```
