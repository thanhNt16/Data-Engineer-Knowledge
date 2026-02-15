---
tags: [tool-setup, claude, tmux, terminal, workflow, ai-assistant]
date: 2026-02-15
status: learned
---

# Claude with tmux: The Ultimate Terminal Workflow

## Overview

Using Claude Code directly in your terminal with tmux provides a powerful AI-assisted workflow. Claude's tmux (terminal multiplexer) gives you persistent sessions, organized workflows, and seamless context switching.

**Key Insight**: "Claude Code is optimized for long-lived, complex terminal workflows."

---

## Prerequisites

**Required**:
- Claude Code account (claude.ai or claude.ai)
- tmux installed (terminal multiplexer)
- Terminal emulator (Mac: iTerm2, Linux: tmux, Windows: Terminal.app)

**Optional**:
- Cursor (text editor) - tmux supports editing
- Shell plugins (zsh, bash) - Enhanced terminal features

---

## Installation

### 1. Install tmux

```bash
# macOS
brew install tmux

# Linux (Debian/Ubuntu)
sudo apt-get install tmux

# Verify installation
tmux -V

# macOS (alternate)
brew install reattach-to-user-namespace
```

### 2. Install Claude Code CLI

```bash
# Download CLI
curl -fsSL https://claude.ai/download/claude-cli-installer.sh -o claude-cli-installer.sh
chmod +x claude-cli-installer.sh

# Run installer
./claude-cli-installer.sh

# Verify installation
claude --version
```

---

## Configuration

### 1. Create tmux Session

```bash
# Start new tmux session named "claude"
tmux new-session -s -d /path/to/your/project claude

# Or attach to existing session
tmux attach-session -t claude
```

### 2. Start Claude Code Agent in tmux

```bash
# Method 1: Run Claude Code directly in tmux
tmux new-session -s -d /path/to/project claude
# In tmux window, claude (start Claude Code agent or run commands)

# Method 2: Use "claude" script to attach session
claude start --project /path/to/project
# Claude Code will detect tmux and use it
```

---

## Claude Code Agent Teams

### 1. Organizing Session Management

**Concept**: Each project gets its own Claude Code session context.

**Setup**:
```
Projects/
├── data-engineer/
│   └── context.claude
├── web-development/
│   └── context.claude
├── ai-experiments/
│   └── context.claude
```

**Workflow**:
1. Open project directory
2. Run `claude --project` to attach context
3. Work in Claude Code with persistent project context
4. Detach context when switching projects

### 2. File Context Management

**Concept**: Automatically include relevant files in Claude context.

**Setup**:
```bash
# Tell Claude Code to include specific files
claude start --project /path/to/project --include-pattern "*.py" "*.sql" "README.md"

# Claude Code will search for and index relevant files for context
```

### 3. Multi-Project Workflow

**Concept**: Separate Claude Code sessions for different projects.

**tmux Setup**:
```
# Create named session
tmux new-session -s -d /path/to/project-1 claude-project-1

# Another session
tmux new-session -s -d /path/to/project-2 claude-project-2

# Navigate between sessions
tmux list-sessions  # See all sessions
tmux attach-session -t claude-project-2  # Switch to project 2

# Detach session (keeps it running in background)
tmux detach-client -s claude-project-1
```

---

## Claude Code tmux Features

### 1. Session Persistence

**Feature**: Claude Code sessions persist across terminal restarts.

**How It Works**:
- Each tmux session = persistent Claude Code context
- Re-attach to session → context restored
- Sessions can run indefinitely (unlimited)

**Use Case**:
```bash
# Start persistent session (lasts until you exit)
claude --session claude-project

# Work overnight, detach terminal
# Next day: Re-attach (context is still there!)
tmux attach-session -t claude-project
```

---

### 2. Project Context Management

**Feature**: Maintain project-specific context for each tmux session.

**Setup**:
```bash
# Each project has its context file
echo "# Data Engineer Knowledge Base" > /projects/data-engineer/context.claude
echo "Files: README.md, schema.sql, pipelines/" > /projects/data-engineer/context.claude

# Attach with specific project
claude start --project /projects/data-engineer --name data-engineer
```

**Benefits**:
- Isolated project contexts
- Reduced confusion (no cross-project pollution)
- Easier context switching

---

### 3. Git Workflow Integration

**Feature**: Execute Git commands directly in Claude Code.

**Use Cases**:
```bash
# Stage changes
git add .
git status

# Commit with Claude's help
"Help me commit these changes: Added new streaming patterns"
git commit -m "Add streaming patterns"

# Create branch
git checkout -b feature/new-patterns
claude start --project .
git add .
git commit -m "Implement new patterns"

# Merge branch
git checkout main
git merge feature/new-patterns

# Push changes
git push origin main

# Use Claude's git integration (automatic)
claude start --project .
"Push these changes to GitHub"
```

---

## Workflow Patterns

### 1. Single-Project Session

**Best For**: Deep work on single project.

**Workflow**:
```bash
# 1. Open terminal (iTerm2)
cd /projects/data-engineer

# 2. Start Claude Code with tmux
tmux new-session -s -d /projects/data-engineer claude-data-eng

# 3. In Claude Code window:
claude  # Ask Claude to read project files
> Read README.md for context
> Read schema.sql for database structure
> Analyze pipeline architecture

# 4. Work on task
claude > Implement streaming pipeline with Kafka and Flink
> Add error handling and monitoring
> Generate SQL migrations

# 5. Use Git integration
claude > git status
claude > git add streaming_pipeline.py
claude > git commit -m "Add streaming pipeline"
claude > git push
```

### 2. Multi-Project Session

**Best For**: Managing multiple projects simultaneously.

**tmux Workflow**:
```
# Start tmux with 3 windows
tmux new-session -s -d /projects/data-engineer data-eng
tmux new-session -s -d /projects/web-dev web-dev
tmux new-session -s -d /projects/ai-experiments ai-exp

# Switch between sessions
# In tmux: Ctrl+B, then [0, 1, 2]

# Each window has its own Claude Code session
# Context is maintained separately per session
```

**Workflow**:
```bash
# Window 0: Data Engineer project
cd /projects/data-engineer
claude start --project .

# Window 1: Web Development project
cd /projects/web-dev
claude start --project .

# Window 2: AI Experiments project
cd /projects/ai-experiments
claude start --project .

# Or use project names for easier switching
claude start --project data-engineer --name data-engineer
claude start --project web-dev --name web-dev
```

---

## Advanced Claude Code tmux Features

### 1. Cursor Mode Integration

**Feature**: Edit code with Cursor IDE, see results in Claude Code.

**Setup**:
```bash
# Install Cursor CLI
brew install --cask cursor

# Use Cursor with Claude Code
cursor code .
claude > Open this file in Cursor
```

---

### 2. Context Window

**Feature**: View Claude's understanding of your code without running commands.

**Usage**:
```bash
# In Claude Code:
claude --context
# View what Claude knows about current project
```

---

### 3. Runbook Integration

**Feature**: Execute multiple commands as a runbook.

**Setup**:
```bash
# Create runbook (text file)
cat > runbook.md << 'EOF'
# Step 1: Pull latest data
cd /path/to/project
git pull origin main

# Step 2: Start services
docker-compose up -d
sleep 10

# Step 3: Run migrations
python manage.py migrate upgrade

# Step 4: Verify services
curl -f http://localhost:8000/health
EOF

# Execute runbook with Claude
claude start --project /path/to/project
claude > Execute the runbook runbook.md
```

---

### 4. Parallel Task Execution

**Feature**: Run multiple Claude Code tasks simultaneously.

**Use Case**:
```bash
# Execute multiple independent tasks
claude > Analyze schema.sql in parallel
> Write unit tests for models.py
> Generate documentation for API endpoints
> Run database migrations

# All tasks use Claude Code's orchestration
```

---

## Best Practices

### 1. Project Organization

| Pattern | Description |
|---------|-------------|
| **Project Isolation** | One project per context file |
| **Context Files** | Include README, schema, configs |
| **Named Sessions** | `claude --project /path/to/project --name data-engineer` |
| **Detached Background** | `tmux detach-client` for long-running tasks |

### 2. Claude Code Prompting

**Effective Prompts**:

| Goal | Prompt Template | Example |
|-------|-----------------|----------|
| **Project Overview** | "Summarize this project in 5 points" | "Read README and explain architecture" |
| **Code Analysis** | "Analyze this file for bugs" | "Find security vulnerabilities in auth.py" |
| **Generate Documentation** | "Create API documentation for endpoints" | "Write OpenAPI spec for user management" |
| **Debugging** | "Help me fix this error" | "Why is this SQL query slow?" |
| **Refactoring** | "Refactor this function for better performance" | "Split this 500-line file into modules" |
| **Git Workflow** | "Create a PR description" | "Write a comprehensive commit message" |

### 3. Workflow Optimization

| Technique | Description | Benefit |
|-----------|-------------|----------|
| **Project Switching** | `claude --project /path/to/other` | Instant context change |
| **Context Refresh** | `claude --project . --reload` | Reload project context |
| **Session Persistence** | Use tmux's attach/detach | Long-lived sessions |
| **Background Tasks** | `tmux detach-client -s session` | Run in background |

---

## Troubleshooting

### Session Won't Attach

**Symptoms**:
- `claude start --project` says "context not found"
- `tmux attach-session -t` says "session not found"

**Solutions**:
```bash
# Check if session exists
tmux list-sessions | grep claude

# Kill stuck sessions
tmux kill-session -t claude-project

# Start fresh session
tmux new-session -s -d /path/to/project claude-project
```

### Claude Code Not Responding

**Symptoms**:
- No output from Claude Code
- Connection issues

**Solutions**:
```bash
# Check Claude Code status
curl -I https://claude.ai/api/status

# Restart Claude Code
claude start --project . --restart

# Check tmux
tmux send-keys -t claude-project C-o : Enter # Force enter (not always needed)
```

---

## Advanced Features

### 1. Multi-Project Workspace

**Setup**:
```bash
# Workspace with multiple projects
workspace/
├── frontend/
├── backend/
├── ml/
└── data/

# Attach workspace root
claude start --project /workspace --name workspace
```

**Workflow**:
- Ask Claude to analyze entire workspace
- Generate documentation for all projects
- Create architecture diagram
- Identify cross-project dependencies

### 2. Continuous Integration (CI/CD)

**Setup**:
```bash
# Run tests in Claude Code
claude > Run all unit tests in test/ directory
claude > Generate coverage report
claude > Fix any failing tests

# CI/CD integration
claude > Update CI configuration for GitHub Actions
claude > Create pull request description
```

### 3. Data Analysis Workflow

**Setup**:
```bash
# Attach to data engineering project
claude start --project /projects/data-engineer

# Ask Claude to analyze SQL schemas
claude > Read all .sql files and understand relationships
claude > Generate data lineage diagram
claude > Identify optimization opportunities

# Generate documentation
claude > Create data dictionary for all tables
claude > Document ETL pipeline flow
```

---

## Comparison: tmux vs. Native Terminal

| Feature | tmux + Claude | Native Terminal |
|---------|-----------------|----------------|
| **Context Persistence** | ✅ Persistent across sessions | ❌ Context resets on restart |
| **Project Switching** | ✅ Multiple named sessions | ❌ Manually change directories |
| **Git Integration** | ✅ AI-assisted commands | ⚠️ Manual git commands |
| **Claude AI Access** | ✅ Full AI capabilities | ⚠️ Basic autocomplete |
| **Context Files** | ✅ Automatic file inclusion | ❌ Manual file opening |
| **Multi-Window** | ✅ Multiple projects visible | ❌ Multiple terminal windows |
| **Background Tasks** | ✅ Detach sessions (continue work) | ❌ Must keep terminal open |

---

## Next Steps

### 1. Get Started

```bash
# 1. Install tmux (if not already installed)
brew install tmux

# 2. Install Claude Code CLI
curl -fsSL https://claude.ai/download/claude-cli-installer.sh -o claude-cli-installer.sh
chmod +x claude-cli-installer.sh
./claude-cli-installer.sh

# 3. Verify installations
tmux -V
claude --version

# 4. Start first session
cd /path/to/your/project
tmux new-session -s -d . claude-project

# 5. In Claude Code window, start working
claude  # Ask Claude to read project files, analyze code, etc.
```

### 2. Build Workflow

```bash
# Create your first workflow
cat > workflow.md << 'EOF'
# My Data Engineer Workflow

## Morning Routine
- [ ] Check emails
- [ ] Review Pull Requests
- [ ] Plan tasks for the day

## Development Workflow
1. Setup environment
   - Activate virtualenv: source venv/bin/activate
   - Install dependencies: pip install -r requirements.txt

2. Run tests
   - Run unit tests: pytest tests/
   - Generate coverage: pytest --cov=src
   - Fix failing tests

3. Write code
   - Implement feature: code/new-feature/
   - Document changes: CHANGES.md
   - Git workflow: git add, commit, push

## Deployment
   - Build Docker image: docker build -t myapp .
   - Push to registry: docker push myregistry/myapp:latest
   - Deploy to staging: kubectl apply -f k8s/staging.yaml
   - Verify deployment: curl -f http://staging.example.com/health

## Monitoring
   - Check logs: kubectl logs -f deployment/app-name
   - Set up alerts: Prometheus + Grafana
   - Review metrics: http://grafana.example.com/d/app-name
EOF

# Execute workflow with Claude
claude start --project .
claude > Follow the workflow.md workflow
claude > Help me execute step 3 of workflow: Run tests
```

---

## Resources

### Documentation

- [Claude Code Documentation](https://code.claude.com/docs/en/agent-teams)
- [Claude Code Terminal Mastery](https://developertoolkit.ai/en/claude-code/productivity-patterns/terminal-mastery/)
- [GitHub - nielsgroen/claude-tmux](https://github.com/nielsgroen/claude-tmux)

### Tutorials

- [How to Set Up and Use Claude Code Agent Teams](https://darasoba.medium.com/how-to-set-up-and-use-claude-code-agent-teams-and-actually-get-great-results-9a34f8648f6d)
- [Terminal Mastery with Claude Code & Codex](https://developertoolkit.ai/en/claude-code/productivity-patterns/terminal-mastery/)
- [The Ultimate Terminal Workflow for AI Development](https://www.blle.co/blog/claude-code-tmux-beautiful-terminal)

---

**Progress**: 🟢 Learned (Claude with tmux setup documented, workflows created)

**Next Steps**:
- [ ] Install tmux and Claude Code CLI
- [ ] Start first tmux session with Claude Code
- [ ] Create project context files
- [ ] Practice common workflows (git, testing, documentation)
- [ ] Explore advanced features (cursor integration, background tasks)
