# Linear PM Agents for Claude Code

> A set of specialized Claude Code subagents that automate agile project planning in [Linear](https://linear.app) — from a free-form project description to a fully structured backlog with epics, user stories, and technical tasks.

![Claude Code](https://img.shields.io/badge/Claude_Code-Subagents-orange)
![Linear](https://img.shields.io/badge/Linear-MCP_Integration-5E6AD2)
![License](https://img.shields.io/badge/License-MIT-green)

---

## What is this?

**Linear PM Agents** is a collection of 5 Claude Code subagents that act as your AI product manager. You describe your project in plain language and the agents:

1. Generate a structured **Requirements Document**
2. Create **Epics** in Linear — one by one, with your approval
3. Break each epic into **User Stories** — with Gherkin acceptance criteria
4. Break each story into **Technical Tasks** — with type prefixes, time estimates, and execution order

Every step is **interactive**: the agent shows you the generated content and waits for your approval before creating anything in Linear. You can approve, modify, or skip each item.

---

## Who is this for?

- **Developers** who know agile methodology but find backlog management tedious
- **Solo founders / small teams** who need structured planning without a dedicated PM
- **Consultants** who deliver multiple projects and need fast, consistent backlog setup
- **Government / enterprise dev teams** working in Azure-based environments

---

## Flow

```
You describe the project
        ↓
pm-orchestrator (main agent)
        ├── pm-discovery         → Requirements Document (you approve)
        ├── pm-epic-generator    → Epics in Linear (one by one, you approve each)
        ├── pm-story-generator   → User Stories per epic (one by one, you approve each)
        └── pm-task-generator    → Technical Tasks per story (one by one, you approve each)
```

**Result in Linear:**
```
Team
└── Issue "Épica: [Name]"  [label: Epic]
    └── Issue "Historia: [Name]"  [label: Story]
        ├── Issue "[Backend]: Create endpoint"  [label: Task]
        ├── Issue "[DB]: Migration"  [label: Task]
        └── Issue "[Tests]: Unit tests"  [label: Task]
```

---

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and configured
- A [Linear](https://linear.app) account (free plan works)
- Node.js 18+ installed
- `n` or `nvm` for global npm package management (recommended)

---

## Installation

### 1. Get your Linear API Key

1. Go to **Linear → Settings → API → Personal API Keys**
2. Click **Create key**, name it `claude-code-pm`
3. Copy the key (shown only once)

### 2. Install the Linear MCP server

```bash
npm install -g linear-mcp
```

### 3. Register the MCP server in Claude Code

```bash
claude mcp add linear-server \
  -e LINEAR_API_KEY=lin_api_YOUR_KEY_HERE \
  -- node $(npm root -g)/linear-mcp/build/index.js
```

> **Note:** If you use `n` for Node version management, the global path may be different. Find it with:
> ```bash
> npm root -g
> ```

Verify it's registered:
```bash
claude mcp list
# Should show: linear-server
```

### 4. Copy agents to your Claude Code agents directory

```bash
# Create the agents directory if it doesn't exist
mkdir -p ~/.claude/agents

# Copy all agents
cp agents/*.md ~/.claude/agents/
```

### 5. Create labels in Linear

Go to your Linear workspace → **Settings → Labels** and create:

| Label | Color |
|-------|-------|
| `Epic` | Purple `#7C3AED` |
| `Story` | Blue `#2563EB` |
| `Task` | Green `#059669` |

> **Optional but recommended.** The agents work without labels — they just won't be color-coded in Linear.

### 6. Get label UUIDs (optional)

After creating labels, ask Claude Code:

```
use linear-server to list all teams and labels with their IDs
```

You'll get UUIDs to configure in the agents if you want labels applied automatically.

---

## Usage

Open any Claude Code session from your home directory (where the MCP is registered) and type:

```
I want to plan a project in Linear
```

or

```
quiero planificar un proyecto en Linear
```

The `pm-orchestrator` agent activates automatically and guides you through the full flow.

### Example session

```
You: I want to plan a new project

pm-orchestrator: Hi! I'm your PM agent. Let's plan your project in Linear step by step.

  1. Team ID: Do you know your Team ID? If not, I can look it up.
  2. Project description: Tell me what you want to build.

You: [team id] I need a permit management system for a municipal government.
     Citizens should be able to apply for construction permits online and
     track their status. Inspectors need a mobile-friendly interface
     to validate permits in the field.

pm-orchestrator: [Generates full Requirements Document]
                 ✅ aprobado / ✏️ modificar: [changes]

You: aprobado

pm-orchestrator: Great. Now let's generate the epics one by one.

pm-epic-generator:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 EPIC 1 — PENDING APPROVAL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Epic: CITIZEN PORTAL FOR PERMIT APPLICATIONS
...
[full epic content]
...
✅ aprobado / ✏️ modificar: [...] / ⏭️ saltar / ❌ cancelar

You: modificar: add that it must integrate with the municipal registry

pm-epic-generator: [Regenerates with the requested change]

You: aprobado

✅ Epic created in Linear: ENG-1
```

---

## Agents

| Agent | Role | Linear tools used |
|-------|------|-------------------|
| `pm-orchestrator` | Main agent — coordinates the full flow | `linear_get_teams`, `linear_search_issues_by_identifier` |
| `pm-discovery` | Transforms free-form description → Requirements Document | None (pure generation) |
| `pm-epic-generator` | Generates and creates epics one by one | `linear_create_issue` |
| `pm-story-generator` | Generates user stories per epic | `linear_create_issue` |
| `pm-task-generator` | Generates technical tasks per story | `linear_create_issue` |

### Approval commands (all agents)

| Command | Action |
|---------|--------|
| `aprobado` / `sí` / `ok` / `dale` | Creates the item in Linear |
| `modificar: [description]` | Regenerates with changes, asks again |
| `saltar` | Skips this item, moves to next |
| `agregar: [description]` | Adds an extra item (stories/tasks) |
| `detener` / `cancelar` | Stops the current phase |

---

## Generated content format

### Requirements Document sections
- Executive summary, context, problem statement
- Primary and secondary objectives
- Stakeholders table
- Scope (in / out)
- Functional requirements (RF-001, RF-002…)
- Non-functional requirements (performance, security, availability)
- Constraints and assumptions
- Acceptance criteria
- Proposed agile flow
- **Preliminary epics list** (feeds the next agent)

### User Story format
```
As a [specific user type],
I want [concrete, measurable feature],
so that [clear business/user benefit].

Acceptance Criteria (Gherkin):
- GIVEN [context] WHEN [action] THEN [expected result]

Story Points: 1 / 2 / 3 / 5 / 8 (Fibonacci)
Priority: Must Have / Should Have / Could Have / Won't Have
```

### Technical Task title conventions
```
[Backend]:  Create endpoint POST /api/permits
[Frontend]: Implement form with Zod validation
[DB]:       Migration — Create permits table
[Tests]:    Unit tests for PermitService
[DevOps]:   Azure DevOps pipeline for permits module
[Docs]:     Document API in Swagger/OpenAPI
[Infra]:    Configure App Service with Managed Identity
```

---

## Configuration

### MCP server location

The `linear-mcp` server is registered per-project in `.claude.json`. If you want it available across all your projects, register it from your home directory:

```bash
cd ~
claude mcp add linear-server \
  -e LINEAR_API_KEY=lin_api_YOUR_KEY \
  -- node $(npm root -g)/linear-mcp/build/index.js
```

### Available Linear tools (from `linear-mcp`)

| Tool | Description |
|------|-------------|
| `linear_get_teams` | List teams with states and labels |
| `linear_create_issue` | Create an issue (epic, story, or task) |
| `linear_search_issues` | Search issues with filters |
| `linear_search_issues_by_identifier` | Find issue by ID (e.g., "ENG-3") |
| `linear_bulk_update_issues` | Update multiple issues at once |
| `linear_list_projects` | List workspace projects |
| `linear_create_comment` | Add comment to an issue |
| `linear_delete_issue` | Delete an issue |

---

## Troubleshooting

**`npx -y linear-mcp` fails with "could not determine executable to run"**
> The `linear-mcp` package has no `bin` field. Install globally and run with `node` directly:
> ```bash
> npm install -g linear-mcp
> node $(npm root -g)/linear-mcp/build/index.js
> ```

**Agent doesn't activate automatically**
> Make sure agent files are in `~/.claude/agents/` (not a subfolder). Restart Claude Code after copying.

**Linear API returns 401**
> Your API key may have expired. Generate a new one at Linear → Settings → API → Personal API Keys.

**Issues created without labels**
> Labels require their UUIDs. Create the labels in Linear UI first, then get their UUIDs by asking Claude: "use linear-server to list all labels".

---

## Project structure

```
linear-pm-agents/
├── agents/
│   ├── pm-orchestrator.md       # Main agent (talks to user)
│   ├── pm-discovery.md          # Requirements document generator
│   ├── pm-epic-generator.md     # Epic creator with approval loop
│   ├── pm-story-generator.md    # User story creator with approval loop
│   └── pm-task-generator.md     # Technical task creator with approval loop
├── README.md
├── .gitignore
└── LICENSE
```

---

## Contributing

Pull requests are welcome. If you add support for other PM tools (GitHub Issues, Jira, Notion) or new agent types (sprint planner, release notes generator), open a PR.

---

## License

MIT — see [LICENSE](LICENSE)

---

## Acknowledgments

Built with [Claude Code](https://claude.ai/code) subagents and the [linear-mcp](https://github.com/cosmix/linear-mcp) MCP server.
