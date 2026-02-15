---
tags: [tmux, claude, terminal, workflow, setup, 2026-02-15]
date: 2026-02-15
status: learned
---

# tmux + Claude Code Setup Guide

## Overview

**tmux** is a terminal multiplexer that allows you to create, access, and control multiple terminal sessions from a single window.

**Claude Code** is an AI coding assistant with a tmux integration for persistent sessions and organized workflows.

**Key Insight**: "tmux + Claude Code = Long-lived, organized terminal workflows for AI-assisted development."

---

## Installation

### 1. Install tmux

**macOS**:
```bash
# Using Homebrew
brew install tmux

# Verify installation
tmux -V
```

**Linux (Debian/Ubuntu)**:
```bash
# Using APT
sudo apt-get update
sudo apt-get install tmux

# Verify installation
tmux -V
```

**Linux (CentOS/RHEL/Fedora)**:
```bash
# Using YUM
sudo yum install tmux

# Verify installation
tmux -V
```

**From Source**:
```bash
# Clone tmux repository
git clone https://github.com/tmux/tmux.git
cd tmux

# Build and install
sh autogen.sh
./configure
make
sudo make install
```

### 2. Install Claude Code CLI

**macOS/Linux**:
```bash
# Download CLI installer
curl -fsSL https://claude.ai/download/claude-cli-installer.sh -o claude-cli-installer.sh
chmod +x claude-cli-installer.sh

# Run installer
./claude-cli-installer.sh

# Verify installation
claude --version
```

---

## tmux Fundamentals

### 1. Session Management

| Command | Description | Key Binding |
|---------|-------------|---------------|
| `tmux new-session -s` | Create new named session | `Ctrl+b c` |
| `tmux attach-session -t` | Attach to existing session | `Ctrl+b $` |
| `tmux detach-session` | Detach current session | `Ctrl+b d` |
| `tmux kill-session` | Kill session | `Ctrl+b x` |
| `tmux list-sessions` | List all sessions | `Ctrl+b s` |

### 2. Window Management

| Command | Description | Key Binding |
|---------|-------------|---------------|
| `tmux split-window -v` | Split window vertically | `Ctrl+b "` |
| `tmux split-window -h` | Split window horizontally | `Ctrl+b %` |
| `tmux select-pane -[n]` | Select pane by number | `Ctrl+b q` |
| `tmux resize-pane -[UDLR]` | Resize pane | `Ctrl+b arrows` |
| `tmux new-window` | Create new window | `Ctrl+b c` |
| `tmux kill-pane` | Kill pane | `Ctrl+b x` |

### 3. Panes Management

| Command | Description |
|---------|-------------|
| `Ctrl+b z` | Zoom in |
| `Ctrl+b [ ]` | Cycle through panes |
| `Ctrl+b o` | Pane selection mode |
| `Ctrl+b {` | Toggle pane fullscreen |

### 4. Copy & Paste Modes

| Mode | Command | Key Binding |
|------|---------|---------------|
| **emacs-mode** | `Ctrl+b [` | `Ctrl+b ]` |
| **vi-mode** | `Ctrl+b [` | `Ctrl+b ]` |
| `mouse-mode** | Mouse copy/paste |

---

## Claude Code Integration

### 1. Starting Claude Code in tmux

**Option A: Direct Command**

```bash
# Start tmux and run Claude Code
tmux new-session -s claude-code -n "Claude Code" "claude"

# When attached to session, run commands
claude start --project /path/to/project
```

**Option B: tmux Environment**

```bash
# Start tmux with Claude Code environment
tmux new-session -s claude -n "Claude Code Session"

# In tmux, attach Claude Code
claude start --project /path/to/project

# Switch back to main session (if needed)
Ctrl+b $
```

### 2. Persistent tmux Sessions

**Claude Code Integration**:
```bash
# Create named session that persists after detach
tmux new-session -s -d /path/to/project claude-workspace

# Run Claude Code in session
cd /path/to/project
claude start --project .

# Detach session (keeps it running)
Ctrl+b d

# Later, re-attach
tmux attach-session -t claude-workspace
```

---

## Workflow Patterns

### 1. Development Workflow

**Pattern**: Create tmux session for each project.

**tmux.conf Configuration**:
```bash
# ~/.tmux.conf

# Set default shell
set -g default-shell /bin/zsh

# Enable mouse
set -g mouse on

# Set default terminal name
set -g default-terminal "tmux-256color"

# Status bar configuration
set -g status-right '#[fg=yellow,bold] %H:%M:%S %Y-%b-%d #[default] #[fg=white,bg=black]'

# Enable vi-mode
setw -g mode-keys vi
```

**Project-Specific Sessions**:
```bash
# Create separate sessions for different projects
tmux new-session -s data-engineer -n "Data Engineer" "zsh"
tmux new-session -s web-app -n "Web App" "zsh"
tmux new-session -s ai-experiments -n "AI Experiments" "zsh"
```

### 2. Multi-Pane Workflow

**Pattern**: Split window into multiple panes for different tasks.

**Setup**:
```bash
# Create 4-pane layout (2x2)
tmux split-window -v

# Pane 1: Claude Code (top-left)
cd /path/to/project
claude start --project .

# Pane 2: Code Editor (top-right)
vim .

# Pane 3: Git/Logs (bottom-left)
git status
tail -f logs/app.log

# Pane 4: Database (bottom-right)
psql -U username dbname
```

### 3. Background Tasks

**Pattern**: Run long-running tasks in detached session.

**Setup**:
```bash
# Start background session
tmux new-session -s -d background-tasks -n "Background"

# In session, run long task
cd /path/to/project
python long_running_job.py

# Detach (keeps running)
Ctrl+b d
```

---

## tmux + Claude Code Workflow

### Step 1: Initialize Project

```bash
# Start tmux session
tmux new-session -s project-init -n "Project Init"

# Attach Claude Code to project
claude attach /path/to/project

# Ask Claude to read project structure
claude > Read README.md and explain architecture

# Ask Claude to analyze dependencies
claude > Analyze requirements.txt and list dependencies

# Ask Claude to create project structure
claude > Create directories: src/, tests/, docs/, config/
```

### Step 2: Development Workflow

```bash
# Create 4-pane layout for development
tmux split-window -v

# Pane 1: Claude Code (top-left)
claude start --project . --name development

# Pane 2: Editor (top-right)
vim src/main.py

# Pane 3: Terminal (bottom-left)
python -m pytest tests/test_main.py

# Pane 4: Git (bottom-right)
git status

# Switch between panes
# Ctrl+b q, then use arrow keys to select pane (1-4)
```

### Step 3: Debugging Workflow

```bash
# Create session for debugging
tmux new-session -s debugging -n "Debugging"

# Pane 1: Application (left)
python app/main.py

# Pane 2: Logs (right)
tail -f logs/debug.log -n 100

# Pane 3: Claude Code (bottom)
claude > Read debug logs and identify issue
claude > Suggest fix for stacktrace at line 42
```

### Step 4: Testing Workflow

```bash
# Create session for testing
tmux new-session -s testing -n "Testing"

# Run tests with Claude Code
claude start --project . --name testing
claude > Run all unit tests
claude > Generate coverage report
claude > Run integration tests

# View test results in real-time
# Tests run in detached session, watch results
```

---

## Advanced tmux Configurations

### 1. Project-Specific tmux Sessions

**Script: `setup-claude-project.sh`**
```bash
#!/bin/bash

PROJECT_PATH="$1"
SESSION_NAME="claude-$PROJECT_PATH"

# Create tmux session
tmux new-session -s -d "$PROJECT_PATH" -n "$SESSION_NAME"

# Attach Claude Code
tmux send-keys -t "$SESSION_NAME" "claude start --project $PROJECT_PATH" C-m

# Split window (2 panes: Claude + Terminal)
tmux send-keys -t "$SESSION_NAME" "split-window -v" C-m

# Select pane 1 (Claude)
tmux send-keys -t "$SESSION_NAME" "select-pane -t 0" Enter

# Wait for Claude to be ready
sleep 2

# Send first command to Claude
tmux send-keys -t "$SESSION_NAME" "read README.md" Enter

echo "tmux session '$SESSION_NAME' created with Claude Code"
```

### 2. Workflows Automation

**tmux.conf**:
```bash
# ~/.tmux.conf

# Status bar with Claude Code status
set -g status-right '#[bg=black]#{?session_windows} #[fg=green,bold] Claude #[default]'

# Key bindings for Claude Code
bind-key -T F5 run-shell -n "claude" "claude start --project . --name F5"
bind-key -T F6 new-window -n "claude" "claude start --project . --name F6"

# Window naming
setw -g automatic-rename on
set -g automatic-rename-format '%h:%t'  # hostname:window_title

# Enable logging
set -g history-limit 50000
```

### 3. Integration with Editor

**Vim + tmux**:
```bash
# Install vim-tmux navigator plugin
git clone https://github.com/sebastienc/git-worktree.git ~/.vim/bundle

# Configure tmux for Vim
set -g default-command "nvim"

# Use Vim from tmux panes
tmux new-session -s vim-work -n "Vim"
tmux send-keys -t vim-workspace "split-window -v" Enter
nvim
```

**Neovim + tmux**:
```bash
# Configure Neovim to work with tmux
echo 'set shell=/usr/bin/zsh' > ~/.tmux.conf

# Start Neovim in tmux
tmux new-session -s neovim -n "Neovim"
nvim
```

---

## Troubleshooting

### Session Issues

**Problem**: tmux won't start

**Solutions**:
```bash
# Check if tmux is already running
ps aux | grep tmux

# Kill existing tmux server
killall tmux

# Check for stale socket files
rm -f /tmp/tmux-*

# Start tmux server
tmux start-server
```

**Problem**: Claude Code won't attach to tmux session

**Solutions**:
```bash
# Verify Claude Code installation
which claude

# Check tmux version
tmux -V

# Reattach to session
tmux attach-session -t claude-workspace

# Send keys directly
tmux send-keys -t claude-workspace "command" Enter
```

### Performance Issues

**Problem**: tmux laggy with many sessions

**Solutions**:
```bash
# Limit session history
set -g history-limit 10000

# Reduce status bar updates
set -g status-interval 5

# Disable window title updates
set -g automatic-rename off

# Use tmux copy mode
set -g mode-keys vi
```

---

## Quick Reference

### Common Commands

```bash
# List sessions
tmux list-sessions

# Kill session
tmux kill-session -t claude-workspace

# Attach to session
tmux attach-session -t claude-workspace

# New window in current session
tmux new-window -n "Claude Code"

# Scroll up
Ctrl+b [

# Scroll down
Ctrl+b ]

# Enter copy mode (vi)
Ctrl+b [

# Exit copy mode
Esc or Ctrl+c

# Split window vertically
Ctrl+b "

# Split window horizontally
Ctrl+b %

# Switch to previous pane
Ctrl+b ;

# Switch to next pane
Ctrl+b ;

# Switch to previous window
Ctrl+b p

# Switch to next window
Ctrl+b n

# List all windows
Ctrl+b w

# List all panes
Ctrl+b q
```

---

## Example Workflows

### Workflow 1: Full-Stack Development

**Setup**:
```bash
# 6-pane layout (2x3)
# Top: Claude Code, Editor, Logs
# Bottom: Database, API Server, Redis

tmux new-session -s fullstack -n "Full Stack"
# Split vertically
tmux split-window -v
# Split horizontally
tmux select-pane -t 0
tmux split-window -h
tmux select-pane -t 1
tmux split-window -h
tmux select-pane -t 2
```

**Commands**:
```bash
# Pane 0 (Top-Left): Claude Code
claude start --project . --name development

# Pane 1 (Top-Center): Editor
vim src/main.py

# Pane 2 (Top-Right): Git/Logs
git log
tail -f logs/app.log

# Pane 3 (Bottom-Left): Database
psql -U username dbname

# Pane 4 (Bottom-Center): API Server
uvicorn main:app

# Pane 5 (Bottom-Right): Redis
redis-cli
```

### Workflow 2: Code Review

**Setup**:
```bash
# 3-pane layout
# Left: Code, Right: Diff, Bottom: Terminal

tmux new-session -s code-review -n "Code Review"
tmux split-window -v
```

**Commands**:
```bash
# Left Pane: Original Code
vim src/original.py

# Right Pane: Modified Code
vim src/modified.py

# Bottom Pane: Diff
vim -d src/original.py src/modified.py
```

### Workflow 3: Data Pipeline Monitoring

**Setup**:
```bash
# 4-pane layout
# Top-Left: Pipeline Logs, Top-Right: Metrics
# Bottom-Left: Database, Bottom-Right: API

tmux new-session -s monitoring -n "Monitoring"
tmux split-window -v
tmux select-pane -t 0
tmux split-window -h
```

**Commands**:
```bash
# Top-Left: Pipeline Logs
tail -f logs/pipeline.log -n 100

# Top-Right: Metrics Dashboard
claude > Generate dashboard from logs/metrics.json
# or open dashboard URL

# Bottom-Left: Database Queries
psql -U username warehouse
SELECT COUNT(*) FROM events WHERE created_at > NOW() - INTERVAL '1 hour';

# Bottom-Right: API Health
curl -f http://api.health
```

---

## Integration with DevTools

### 1. VS Code + tmux

**Method**: Use VS Code integrated terminal with tmux.

**Setup**:
```bash
# Start tmux session
tmux new-session -s vscode -n "VS Code"

# Open VS Code from tmux
code /path/to/project

# Run tmux in VS Code integrated terminal
# VS Code will use tmux as its shell
```

### 2. Docker + tmux

**Method**: Run tmux inside Docker container.

**Setup**:
```bash
# Start Docker container with tmux
docker run -it --name tmux-container ubuntu:latest tmux

# Run tmux session in container
docker exec -it tmux-container tmux new-session -s docker-work -n "Docker"

# Run commands in container
docker exec -it tmux-container claude start --project /app
```

### 3. Kubernetes + tmux

**Method**: tmux sessions for different kubectl contexts.

**Setup**:
```bash
# Create session for development cluster
tmux new-session -s dev-cluster -n "Dev Cluster"
kubectl config use-context dev-cluster
kubectl get pods

# Create session for production cluster
tmux new-session -s prod-cluster -n "Prod Cluster"
kubectl config use-context prod-cluster
kubectl get pods

# Switch between clusters
Ctrl+b $  # Detach from dev-cluster
tmux attach-session -t prod-cluster  # Attach to prod-cluster
```

---

## Best Practices

### 1. Session Naming

| Pattern | Description | Example |
|---------|-------------|----------|
| **Project-Based** | One session per project | `claude-data-engineer` |
| **Task-Based** | Session for specific task | `claude-debug-session` |
| **Environment-Based** | Dev/Stage/Prod sessions | `claude-production` |
| **Time-Based** | Morning/Afternoon sessions | `claude-morning-work` |

### 2. Window Layouts

| Layout | Description | When to Use |
|--------|-------------|--------------|
| **2-Pane** | Side-by-side comparison | Code review, debugging |
| **3-Pane** | Main + 2 auxiliaries | Full-stack development |
| **4-Pane** | Quad view | Monitoring, database, API, logs |
| **6-Pane** | 2x3 grid | Complex workflows, microservices |

### 3. Claude Code Usage Patterns

| Pattern | Description | Example |
|---------|-------------|----------|
| **Single-Project** | Attach Claude Code to one project | `claude start --project /app` |
| **Multi-Project** | Separate sessions for different projects | `claude start --project /app1` (Session 1) |
| **Context-Switching** | Use named sessions for different contexts | `claude start --project /db` (Session 2) |
| **Background** | Run long tasks in detached session | `tmux new-session -s background` (Session 3) |
| **Collaboration** | Share session via tmux attach | Other developer runs `tmux attach-session -t session-name` |

---

## tmux.conf Advanced Configuration

### 1. Status Bar with Claude Code Status

```bash
# ~/.tmux.conf

# Status bar left: Session info
set -g status-left '#[bg=black]#[fg=green,bold]Session: #{?session_name} #[default]'

# Status bar center: Claude Code status
set -g status-left '#[bg=black]#[fg=yellow,bold]Claude: #{?session_windows} window #[default]'
set -g status-right '#[bg=black]#[fg=cyan,bold]#{?version} #[default]'

# Status bar with Git branch (requires script)
set -g status-right '#[bg=black]#[fg=blue,bold]#{git_branch} #[default]'
```

### 2. Key Bindings for Common Actions

```bash
# ~/.tmux.conf

# F5: New window
bind-key -T F5 new-window

# F6: Split vertical
bind-key -T F6 split-window -v

# F7: Split horizontal
bind-key -T F7 split-window -h

# Ctrl+b + Enter: New session
bind-key -T C-Enter new-session

# Ctrl+b + d: Detach session
bind-key -T C-d detach-session

# Ctrl+b + x: Kill pane
bind-key -T C-x kill-pane

# Ctrl+b + [: Previous pane
bind-key -T C-[ select-pane -p

# Ctrl+b + ]: Next pane
bind-key -T C-] select-pane -n

# r: Reload config
bind-key -T r source-file ~/.tmux.conf
```

---

## Session Persistence

### 1. Saving Session State

**Concept**: tmux sessions can persist across restarts.

**Configuration**:
```bash
# ~/.tmux.conf

# Enable session persistence
set -g history-file ~/.tmux_history

# Save last session
set -g save-strategy "new"

# Limit history file size
set -g history-limit 5000
```

### 2. Restoring Sessions

**Commands**:
```bash
# List saved sessions
tmux list-sessions

# Attach to saved session
tmux attach-session -t claude-workspace

# Kill saved session
tmux kill-session -t claude-workspace
```

---

## Interview Questions

**Q1: What is tmux and when would you use it?**

**A1**: tmux is a terminal multiplexer that allows you to create and manage multiple terminal sessions from a single window. I would use it when:
- Need to work on multiple tasks simultaneously (e.g., development + testing + monitoring)
- Want to organize work into named sessions (one per project)
- Need persistent sessions across terminal restarts
- Want to share terminal sessions with colleagues (pair programming)
- Run long-running processes in the background

**Q2: How does Claude Code integrate with tmux?**

**A2**: Claude Code provides a tmux integration that gives you persistent sessions, organized workflows, and the ability to switch between projects quickly. When you start a session with Claude Code and tmux, you can:
- Detach the session (keeps Claude Code running)
- Attach to it later from anywhere
- Have multiple independent sessions for different projects
- Use tmux's window management to organize code, logs, databases

**Q3: What's the difference between tmux and screen?**

**A3**: Both are terminal multiplexers. Key differences:
- **Emacs vs. Vim**: screen is designed for Emacs (and more flexible for Emacs workflows), tmux is designed for Vim workflows
- **Window Management**: tmux has better window splitting and pane management
- **Configuration**: tmux has a more powerful configuration system
- **Performance**: tmux is generally lighter and faster
- **Community**: tmux has a larger, more active community
- **Compatibility**: screen comes pre-installed on most Linux systems, tmux is not

---

## Next Steps

### Immediate (Today)

- [ ] Install tmux
- [ ] Install Claude Code CLI
- [ ] Start first tmux session with Claude Code
- [ ] Create 2-pane layout (Claude Code + Terminal)
- [ ] Practice tmux key bindings (navigation, windows, sessions)

### This Week

- [ ] Create project-specific tmux sessions
- [ ] Set up workflow automation scripts
- [ ] Configure tmux.conf with custom key bindings
- [ ] Integrate with your editor (Vim/Neovim)
- [ ] Practice multi-pane layouts (3-pane, 4-pane)

### This Month

- [ ] Master advanced tmux features (nested sessions, window layouts)
- [ ] Create debugging workflow with Claude Code
- [ ] Set up monitoring sessions with real-time logs
- [ ] Integrate with dev tools (Docker, Kubernetes)
- [ ] Share tmux sessions with team members

---

## Resources

- [tmux GitHub](https://github.com/tmux/tmux)
- [tmux Cheat Sheet](https://tmuxcheatsheet.com/)
- [Claude Code Documentation](https://code.claude.com/docs/en/agent-teams)
- [The Ultimate Terminal Workflow for AI Development](https://www.blle.co/blog/claude-code-tmux-beautiful-terminal)

---

## Related Notes

- [[System Design]] - Terminal tools, workflow automation
- [[Orchestration/Orchestration Workflows & Scheduling]] - Command execution patterns
- [[02-Areas/System Design/Data Vault Security]] - Secrets management, vault integration

---

**Progress**: 🟢 Learned (tmux + Claude Code setup documented)

**Next Steps**:
- [ ] Install tmux locally
- [ ] Install Claude Code CLI
- [ ] Create first Claude Code session with tmux
- [ ] Practice tmux workflows
- [ ] Integrate with editor and dev tools
