# OpenCode Gitea Review

[![OpenCode](https://img.shields.io/badge/OpenCode-AI%20Code%20Review-blue)](https://opencode.ai)
[![Docker Image](https://img.shields.io/badge/Docker-ghcr.io-blue)](https://ghcr.io/ccsert/opencode-review)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

[中文文档](README_zh.md) | English

An AI-powered **automatic code review tool for Gitea/Forgejo PRs**, built on the [OpenCode](https://opencode.ai) plugin system. It listens for PR and comment events via Gitea Actions, automatically fetches code diffs, and submits structured reviews.

## ✨ Features

- 🤖 **AI-Powered Code Review** - Uses Claude/GPT/DeepSeek models to analyze code changes
- 📝 **Line-Level Comments** - Provides precise feedback on specific code lines
- ✅ **Review Decisions** - Supports approve, request_changes, and comment states
- 🔄 **Auto-Trigger** - Triggered by `/oc` or `/opencode` comments
- � **Incremental Review** - Only reviews new changes since last review (for updated PRs)
- 🏷️ **Structured Tags** - Categorizes issues by type (BUG, SECURITY, PERFORMANCE) and severity
- �🐳 **Docker Support** - Zero-config installation with pre-built image
- 🛡️ **Isolated Configuration** - Uses `.opencode-review/` directory, won't conflict with your existing `.opencode/` setup

## 📦 Installation

### Interactive Installation (Recommended)

Run in your project root:

```bash
curl -fsSL https://raw.githubusercontent.com/ccsert/opencode-review-gitea/main/install.sh | bash
```

You'll see an interactive menu to choose your installation method.

### Direct Installation Options

```bash
# Docker-based (Recommended) - Zero files added to repo
curl -fsSL https://raw.githubusercontent.com/ccsert/opencode-review-gitea/main/install.sh | bash -s -- --docker

# Source-based - Full customization
curl -fsSL https://raw.githubusercontent.com/ccsert/opencode-review-gitea/main/install.sh | bash -s -- --source

# Both methods
curl -fsSL https://raw.githubusercontent.com/ccsert/opencode-review-gitea/main/install.sh | bash -s -- --both
```

## 🔄 Installation Methods Comparison

| Aspect | Docker 🐳 | Source 📦 |
|--------|----------|-----------|
| **Files added** | 1 workflow file | .opencode-review/ + workflow |
| **CI speed** | Fast (cached image) | Slower (install deps each run) |
| **Customization** | Environment variables | Full control over agents/tools |
| **Updates** | Automatic with `:latest` | Manual update required |
| **Best for** | Quick setup, standard use | Custom prompts, advanced users |

## ⚙️ Configuration

### 1. Set Up Secrets

Configure the following secrets in your Gitea repository:

| Secret Name | Description |
|-------------|-------------|
| `OPENCODE_GIT_TOKEN` | Gitea API Token (requires repo permissions) |
| `DEEPSEEK_API_KEY` | DeepSeek API Key (default model) |
| `ANTHROPIC_API_KEY` | Anthropic API Key (optional) |
| `OPENAI_API_KEY` | OpenAI API Key (optional) |

### 2. Configure Model (Optional)

Edit `.gitea/workflows/opencode-review.yaml`:

```yaml
env:
  # Format: provider/model-id
  MODEL: deepseek/deepseek-chat        # Default (requires DEEPSEEK_API_KEY)
  # MODEL: anthropic/claude-sonnet-4-5  # Requires ANTHROPIC_API_KEY
  # MODEL: openai/gpt-4o                # Requires OPENAI_API_KEY
```

### 3. Using LLM Gateways (Optional)

You can use LLM gateways like **LiteLLM**, **OpenRouter**, **Together AI**, **NVIDIA NIM**, etc. via the OpenAI-compatible API:

```yaml
env:
  # Point MODEL to an OpenAI-compatible provider
  MODEL: openai/your-model-name
  OPENAI_API_KEY: ${{ secrets.YOUR_API_KEY }}
  # Set the gateway URL
  OPENAI_BASE_URL: "https://openrouter.ai/api/v1"    # OpenRouter
  # OPENAI_BASE_URL: "http://localhost:4000/v1"        # LiteLLM
  # OPENAI_BASE_URL: "https://api.together.xyz/v1"     # Together AI
```

| Gateway | OPENAI_BASE_URL | Notes |
|---------|-----------------|-------|
| OpenRouter | `https://openrouter.ai/api/v1` | Use `OPENROUTER_API_KEY` |
| LiteLLM | `http://localhost:4000/v1` | Self-hosted proxy |
| Together AI | `https://api.together.xyz/v1` | |
| NVIDIA NIM | `https://integrate.api.nvidia.com/v1` | |
| Ollama (local) | `http://localhost:11434/v1` | No API key needed |

> **Note**: When using gateways, set `MODEL` to `openai/<model-name>` so opencode routes through the OpenAI-compatible provider.

### 4. Review Configuration

These options work with both Docker and Source installations:

```yaml
env:
  # Response language
  REVIEW_LANGUAGE: auto      # auto | en | zh-CN
  
  # Review depth and focus
  REVIEW_STYLE: balanced     # concise | balanced | thorough | security
  
  # File filtering (glob patterns, comma-separated)
  FILE_PATTERNS: ""          # e.g., "*.ts,*.go,src/**" (empty = all files)
```

#### Language Options

| Value | Description |
|-------|-------------|
| `auto` | Auto-detect from code comments (default) |
| `en` | Review in English |
| `zh-CN` | 使用简体中文审查 |

#### File Filtering Examples

```yaml
# Only review TypeScript files
FILE_PATTERNS: "*.ts,*.tsx"

# Only review source files (exclude tests)
FILE_PATTERNS: "src/**/*.go"

# Multiple patterns
FILE_PATTERNS: "*.py,*.js"

# Note: currently only positive matching is supported (no leading '!').
```

## 🚀 Usage

### Trigger Code Review

Comment on a PR:

```
/oc
```

or

```
/opencode please review this PR
```

### Local Testing (Docker)

```bash
docker run --rm \
  -v $(pwd):/workspace \
  -e GITEA_TOKEN="your-token" \
  -e GITEA_SERVER_URL="https://your-gitea.example.com" \
  -e DEEPSEEK_API_KEY="your-key" \
  -e PR_NUMBER=123 \
  -e REPO_OWNER="your-org" \
  -e REPO_NAME="your-repo" \
  ghcr.io/ccsert/opencode-review:latest
```

`REPO_NAME` can be either `repo` or `owner/repo` (entrypoint will normalize it).

> **Note**: The Docker image currently only supports **x86_64/amd64** architecture. ARM devices (Raspberry Pi, etc.) should use the [Source installation](#local-testing-source) method instead.

### Local Testing (Source)

```bash
export GITEA_TOKEN="your-token"
export GITEA_SERVER_URL="https://your-gitea.example.com"
export OPENCODE_CONFIG_DIR="$(pwd)/.opencode-review"

opencode run --agent code-review \
  "Please review PR #123 in owner/repo"
```

## 🔧 Customization (Source Installation)

### Modify Review Style

Edit `.opencode-review/agents/code-review.md`:

```markdown
---
description: AI code reviewer for Gitea/Forgejo PRs
tools:
  "*": false
  "gitea-review": true
  "gitea-pr-diff": true
---

You are a code review expert focusing on [your domain]...
```

### Add New Tools

Create a TypeScript file in `.opencode-review/tools/`:

```typescript
import { tool } from "@opencode-ai/plugin"

export default tool({
  description: "Tool description",
  args: {
    param: tool.schema.string().describe("Parameter description"),
  },
  async execute(args, context) {
    return "Result"
  },
})
```

## 📁 Project Structure

```
.
├── Dockerfile                      # Docker image definition
├── docker-compose.yaml             # Local testing
├── entrypoint.sh                   # Container entrypoint
├── install.sh                      # Installation script
├── templates/
│   ├── workflow-docker.yaml        # Docker workflow template
│   └── workflow-source.yaml        # Source workflow template
├── .github/workflows/
│   └── docker-publish.yaml         # Auto-build Docker image
└── .opencode-review/               # Isolated config directory
    ├── agents/
    │   ├── code-review.md          # Code review agent (main)
    │   └── gitea-assistant.md      # General assistant agent
    ├── tools/
    │   ├── gitea-pr-diff.ts        # Get full PR diff
    │   ├── gitea-pr-files.ts       # List changed files
    │   ├── gitea-incremental-diff.ts # Get incremental diff (new changes only)
    │   ├── gitea-review.ts         # Submit review with comments
    │   └── gitea-comment.ts        # Post comments on issues/PRs
    ├── skills/
    │   └── pr-review/SKILL.md      # Reusable review skill
    └── package.json                # Dependencies
```

> **Note**: After installation, `.gitea/workflows/opencode-review.yaml` will be created in your project.

## 🔗 Related Links

- [OpenCode Documentation](https://opencode.ai/docs)
- [OpenCode Custom Tools](https://opencode.ai/docs/custom-tools/)
- [Gitea API Documentation](https://docs.gitea.io/en-us/api-usage/)
- [Docker Image](https://ghcr.io/ccsert/opencode-review)

## 📄 License

MIT License - See [LICENSE](LICENSE)
