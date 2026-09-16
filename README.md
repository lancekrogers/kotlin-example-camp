# fde-technical-exercise

Camp for a Forward Deployed Engineer (FDE) candidate technical exercise: take
over an existing Kotlin/Ktor RealWorld example app, ship a small product
improvement with tests and CI, and leave the repository in better shape than it
was found. The work is planned and executed with the Festival methodology.

## Mission

Complete the FDE technical exercise end to end, using coding agents as part of
the workflow, with a reviewable record of the plan, the changes, and the
verification evidence.

## Directory Structure

```text
.
├── .campaign/          Camp configuration and system state
│   ├── intents/        System-managed intent state (use camp intent)
│   └── settings/       Camp settings and shortcuts
├── projects/           Project repositories (submodules or worktrees)
├── festivals/          Festival methodology planning workspace
├── docs/               Human-authored documentation
├── workflow/           Workflow management (reviews, design, explore)
│   ├── design/         Design documents
│   ├── explore/        Exploratory research and discovery notes
│   └── reviews/        Review notes, feedback, and assessments
├── dungeon/            Archived and deprioritized work
├── AGENTS.md           AI agent instructions
└── CLAUDE.md           Symlink to AGENTS.md
```

## Getting Started

This camp is managed with the **camp** CLI.

```bash
# Navigation
camp go <shortcut>       # Jump to a shortcut location
camp p <project>         # Jump to a project directory
camp pins                # List pinned directories

# Project management
camp projects            # List all registered projects
camp project add <path>  # Register a new project

# Shortcuts
camp shortcuts           # List all shortcuts
camp shortcuts add       # Add a new shortcut (interactive)

# Workflow
camp log                 # Show camp git log
camp intent              # Manage camp intents in .campaign/intents/

# Help
camp --help              # Full command reference
```
