# Slack Notification

Composite GitHub Action that posts a status notification — deploy, scheduled
job, or CI result — to Slack via Block Kit (`chat.postMessage`). One step,
no dependencies to install: the message adapts to what you pass it, showing
only the sections that have data.

Licensed under the [MIT License](LICENSE).

## Usage

```yaml
jobs:
  notify:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: howdycom/slack-notification@v1
        with:
          status: ${{ job.status }}
          environment: production
          workflow_name: ${{ github.workflow }}
          commit_sha: ${{ github.sha }}
          repository: ${{ github.repository }}
          actor: ${{ github.actor }}
          workflow_run_url: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
          bot_token: ${{ secrets.SLACK_BOT_TOKEN }}
          channel_id: ${{ vars.SLACK_CHANNEL_ID }}
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `status` | yes | — | Run status: `success`, `failure`, or anything else (rendered as cancelled). Drives the emoji, title, and button color. |
| `environment` | yes | — | Environment name (`staging`, `production`, …). |
| `workflow_name` | yes | — | Name of the workflow, shown next to the environment. |
| `commit_sha` | no | `''` | Full commit SHA. Rendered as a 7-char link when `repository` is also set. |
| `commit_message` | no | `''` | Commit message, truncated to 100 chars. Only shown with `commit_sha`. |
| `actor` | no | `''` | GitHub actor, shown as "Triggered by". |
| `error_message` | no | `''` | Failure detail in a code block, truncated to 1000 chars. |
| `workflow_run_url` | no | `''` | Adds a "View Workflow Run" button (red on failure, blue otherwise). |
| `services` | no | `''` | Comma-separated list of affected services. |
| `repository` | no | `''` | `owner/repo`, used to link the commit SHA. |
| `bot_token` | yes | — | Slack bot token (`xoxb-…`, needs the `chat:write` scope). Pass via secrets. |
| `channel_id` | yes | — | Slack channel ID (`C…`). |

No outputs.

## Message layout

- Header: ✅ Run Successful / ❌ Run Failed / ⚠️ Run Cancelled
- Environment + workflow fields, then (only when provided) services, commit
  + message, triggered-by, error block, and the workflow-run button.

The payload is built with `jq` (installed on the fly if missing) and
validated as JSON before sending; the step fails if Slack returns
`ok: false` or a non-200 status, printing the API error.

## Notes

- Pin to a tag (`@v1`), never to `main`.
- The bot token needs the `chat:write` scope, and the bot must be a member
  of the target channel.

## Versioning

Changes are tagged with semver (`v1`, `v1.1`, …). The major tag (`v1`) moves
to the latest compatible release; breaking changes bump the major version.
Don't reference `main` from a consumer workflow.

## Contributing

Changes go through a PR, not direct pushes to `main`. Message-layout changes
should include a screenshot of the rendered Block Kit in the PR body.
