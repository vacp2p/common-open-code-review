# Common OpenCodeReview

Reusable GitHub composite action for OpenCodeReview in VAC Nim repositories.
It deliberately follows the latest upstream release through
`alibaba/open-code-review@main` and its default `ocr_version: latest`.

The action supplies shared Nim review rules from its own checkout, rather than
from the pull request being reviewed. Repository-specific rules belong in the
consumer repository and should not duplicate these common rules.

## Consumer workflow

Keep event triggers, authorization, concurrency, permissions, and the LLM
secret in each consumer repository. The review step is:

```yaml
- uses: vacp2p/common-open-code-review@main
  with:
    llm_url: https://api.deepseek.com/chat/completions
    llm_auth_token: ${{ secrets.DEEPSEEK_API_KEY }}
    llm_model: ${{ steps.pr.outputs.model }}
    llm_use_anthropic: 'false'
    base_ref: ${{ steps.pr.outputs.base_ref }}
    head_sha: ${{ steps.pr.outputs.head_sha }}
```

The calling workflow needs `contents: read`, `issues: write`, and
`pull-requests: write` permissions.
