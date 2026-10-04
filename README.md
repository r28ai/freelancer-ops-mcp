# Freelancer and Agency Ops

Client onboarding, invoices from calendar and commits, status reports and overdue chasers.

An MCP server with **8 workflows** across Stripe, Google Drive, Linear, Google Calendar, Gmail, GitHub, Google Docs, Granola, Google Sheets and Firecrawl. Each workflow is a prompt your agent runs as a slash command, over the 26 tools it needs and no others.

```bash
uv tool install https://github.com/r28ai/freelancer-ops-mcp/releases/download/v0.1.0/freelancer_ops_mcp-0.1.0-py3-none-any.whl
claude mcp add freelance -- freelancer-ops-mcp
```

It installs with [uv](https://docs.astral.sh/uv/) from this repository's release, with no git and nothing to build; nothing but Charter and the libraries it uses comes from PyPI. To update, run the install line from the [latest release](https://github.com/r28ai/freelancer-ops-mcp/releases/latest). If a desktop app cannot find `freelancer-ops-mcp`, give it the full path from `which freelancer-ops-mcp` (`where freelancer-ops-mcp` on Windows).

Then ask your agent to **connect your apps**, or run `/mcp__freelance__setup`.

## Connect your apps

Ask the agent to connect one ("connect Linear"). It tells you where to get that app's key and the command that stores it, and the next call works, with no restart. The agent never asks for a key in the chat.

Or connect everything this server uses from a terminal:

```bash
freelancer-ops-mcp login            # each app in turn
freelancer-ops-mcp login stripe     # just one
freelancer-ops-mcp status           # what is connected
```

Tokens and keys go to your operating system's keychain (macOS Keychain, Windows Credential Manager, the Secret Service on Linux), and are checked with one read-only call to the app's own API before they are kept. Every key, token and OAuth client is yours: we register no app with any of these services, and nothing passes through a server of ours, because there isn't one.

| App | How it connects | Or set |
|---|---|---|
| Stripe | Your own key ([get one](https://dashboard.stripe.com/apikeys)), entered once. | `STRIPE_API_KEY` |
| Google | Browser sign-in, over your own OAuth client ([make one](https://docs.r28.ai/charter/auth/setup/google)). | `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` |
| Linear | Your own key ([get one](https://linear.app/settings/account/security)), entered once. | `LINEAR_API_KEY` |
| GitHub | Your own key ([get one](https://github.com/settings/tokens/new?description=Charter&scopes=repo,read:user)), entered once. | `GITHUB_TOKEN` |
| Granola | Your own key ([get one](https://docs.granola.ai/help-center/sharing/integrations/granola-api)), entered once. In the Granola app: Settings → Connectors → API keys. Business plan or above. | `GRANOLA_API_KEY` |
| Firecrawl | Your own key ([get one](https://www.firecrawl.dev/app/api-keys)), entered once. | `FIRECRAWL_API_KEY` |

A variable set in your client's config always wins over the keychain.

## Workflows

| Workflow | What you get | Apps |
|---|---|---|
| **Client onboarding** <br>`client_onboarding` | Deposit link, SOW from template, shared folder and project, for each new client. | Stripe, Google Drive, Linear |
| **Deposit paid → kickoff** <br>`deposit_paid_to_kickoff` | The kickoff is booked the moment the deposit clears. | Stripe, Google Calendar, Linear, Gmail |
| **Client status report** <br>`client_status_report` | A weekly report the client can read, built from the work itself. | Linear, GitHub, Google Docs, Gmail |
| **Scope creep detector** <br>`scope_creep_detector` | Asks on the call that aren't in the SOW get flagged and a change order drafted. | Granola, Google Docs, Linear, Gmail |
| **Billable hours from calendar** <br>`billable_hours_from_calendar` | Client-tagged meetings become a timesheet and pending line items. | Google Calendar, Google Sheets, Stripe |
| **Overdue invoice chaser** <br>`overdue_invoice_chaser` | Escalating reminders that cite the last thread, and a follow-up held on your calendar. | Stripe, Gmail, Google Calendar |
| **Client site audit → proposal** <br>`client_site_audit_to_proposal` | A prospect's site audited and turned into a scoped proposal with a pay link. | Firecrawl, Google Docs, Stripe |
| **Client feedback in Drive comments → tasks** <br>`client_feedback_in_drive_comments_to_tasks` | Every comment the client left on a deliverable becomes a task, and they see the link. | Google Drive, Linear |

Every prompt takes one optional argument, `details`: the repo, team, channel, customer or date range you mean, so the agent does not have to ask. In Claude Code, put it in quotes, or only its first word arrives:

```
/mcp__freelance__client_onboarding "new client Acme Ltd, $2,000 deposit"
```

Reads run without asking. Before anything that creates, sends, changes or deletes, the prompt tells the agent to show you the call and wait.

0 of the 8 workflows need no Google or Granola credential.

## Other clients

**Claude Desktop**: install [uv](https://docs.astral.sh/uv/getting-started/installation/) if you have not, since Claude Desktop starts the server with it, then open the `.mcpb` from the [latest release](https://github.com/r28ai/freelancer-ops-mcp/releases/latest). Claude asks for any keys in its own settings and keeps them in your keychain. The first start takes a few seconds longer, while uv installs it.

**VS Code** (`.vscode/mcp.json`): VS Code asks for each key the first time the server starts and stores it securely. Leave out any you stored with `login`.

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "stripe-api-key",
      "description": "Stripe: Secret or restricted key",
      "password": true
    },
    {
      "type": "promptString",
      "id": "google-client-secret",
      "description": "Google: OAuth client secret",
      "password": true
    },
    {
      "type": "promptString",
      "id": "linear-api-key",
      "description": "Linear: Personal API key",
      "password": true
    },
    {
      "type": "promptString",
      "id": "github-token",
      "description": "GitHub: Personal access token",
      "password": true
    },
    {
      "type": "promptString",
      "id": "granola-api-key",
      "description": "Granola: API key",
      "password": true
    },
    {
      "type": "promptString",
      "id": "firecrawl-api-key",
      "description": "Firecrawl: API key",
      "password": true
    }
  ],
  "servers": {
    "freelance": {
      "type": "stdio",
      "command": "freelancer-ops-mcp",
      "env": {
        "STRIPE_API_KEY": "${input:stripe-api-key}",
        "GOOGLE_CLIENT_SECRET": "${input:google-client-secret}",
        "LINEAR_API_KEY": "${input:linear-api-key}",
        "GITHUB_TOKEN": "${input:github-token}",
        "GRANOLA_API_KEY": "${input:granola-api-key}",
        "FIRECRAWL_API_KEY": "${input:firecrawl-api-key}",
        "GOOGLE_CLIENT_ID": ""
      }
    }
  }
}
```

**Cursor** (`.cursor/mcp.json`) starts it the same way:

```json
{
  "mcpServers": {
    "freelance": {
      "command": "freelancer-ops-mcp"
    }
  }
}
```

**Codex** (`~/.codex/config.toml`) starts a turn without waiting for a server unless it is `required`, and then the agent has none of its tools. `required = true` makes the session wait for it, and `startup_readiness = "catalog"` waits for its tool list rather than just its connection:

```toml
[mcp_servers.freelance]
command = "freelancer-ops-mcp"
required = true
startup_readiness = "catalog"
startup_timeout_sec = 30
```

Name the server `freelance`. A host builds each tool's name from that key, and a longer one can push a tool past the 64 characters a function name allows.

## Built with Charter

Every tool here is a [Charter](https://github.com/r28ai/charter) declaration: a Pydantic schema saying where each field goes on the wire. Charter's runtime builds the request, attaches and refreshes the credential, and trims the response before the model reads it. It runs in your process, with no proxy and no telemetry.

The 26 tool schemas come to 38,633 tokens.

The same tools work in your own agent, without MCP:

```python
from charter.adapters.openai import to_openai_tools
from charter_packs_mcp import FAMILIES

tools = FAMILIES["agency"].tools()
definitions = to_openai_tools(tools)   # or charter.adapters.langchain
```

Need an API that isn't here? [Write a pack](https://docs.r28.ai/charter/start/coding-agents): your coding agent writes the declarations, and Charter's conformance suite checks them.

<details>
<summary>All 26 tools</summary>

- **Stripe**: `stripe_customers_create`, `stripe_checkout_sessions_create`, `stripe_checkout_sessions_list`, `stripe_invoice_items_create`, `stripe_invoices_list`
- **Google Drive**: `gdrive_files_copy`, `gdrive_permissions_create`, `gdrive_comments_list`, `gdrive_replies_create`
- **Linear**: `linear_project_create`, `linear_project_get`, `linear_issues_list`, `linear_issue_create`
- **Google Calendar**: `gcalendar_events_insert`, `gcalendar_events_list`, `gcalendar_events_quick_add`
- **Gmail**: `gmail_messages_send`, `gmail_drafts_create`, `gmail_threads_list`
- **GitHub**: `github_repos_list_commits`
- **Google Docs**: `gdocs_documents_create`, `gdocs_documents_get`
- **Granola**: `granola_notes_get`
- **Google Sheets**: `gsheets_spreadsheets_values_append`
- **Firecrawl**: `firecrawl_crawl`, `firecrawl_crawl_status`

</details>

## License

Apache 2.0.
