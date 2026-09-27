# Enterprise Multi-Agent Code Review Orchestrator

Construct a multi-agent system that is ready for production use and which automates code reviews using the Claude Agent SDK.

## Project Overview

This system uses multiple specialized AI agents working together to provide comprehensive code reviews:

- **Main Orchestrator** - Coordinates the review process and aggregates results
- **Code Quality Analyzer** - Identifies code smells, anti-patterns, and best practice violations
- **Test Coverage Analyzer** - Evaluates test completeness and suggests missing test cases
- **Refactoring Suggester** - Recommends architectural improvements and refactoring opportunities

## What's Provided

This starter includes the infrastructure you need:

- **Type Definitions** (`src/types/`) - Zod schemas for validation
- **Logger** (`src/utils/logger.ts`). Uses Winston for structured logging.
- **Report Generator** (`src/utils/report-generator.ts`), makes reports in Markdown, HTML, or JSON
- **Project Config**. `package.json`, `tsconfig.json`, `.env.example`
- **Test Skeletons** (`tests/`) - Test file structure
- **Example Skill** (`.claude/skills/`) - Sample Claude skill

## What You Need to Implement

Your tasks:

1. **Agent Definitions** (`src/agents/`)
   - Code Quality Analyzer
   - Test Coverage Analyzer
   - Refactoring Suggester

2. **Prompts** (`src/prompts/`)
   - Orchestrator prompt
   - Agent-specific prompts

3. **MCP Configuration** (`src/config/mcp.config.ts`)
   - GitHub MCP server
   - ESLint MCP server

4. **Orchestrator** (`src/orchestrator.ts`)
   - Main coordination logic
   - Agent spawning and result aggregation

5. **Main Entry Point** (src/main.ts)
   - CLI argument parsing
   - Environment validation
   - Report generation

6. **Error Handler** (Recommended) (`src/utils/error-handler.ts`)
   - Custom `ReviewError` class
   - Retry logic with exponential backoff
   - Timeout wrapper

7. **Rate Limiter** (Optional) (`src/utils/rate-limiter.ts`)
   - Token bucket algorithm with sliding window
   - Request and token tracking
   - Concurrent request management

## Getting Started

### Prerequisites

- Node.js 18+
Either use the Anthropic API access (which is available in the Vocareum workspace) or obtain your own API key from https://console.anthropic.com/
- GitHub Personal Access Token (recommended; scopes: repo, read:org)

### Installation

**In Vocareum Workspace (Recommended):**

Anthropic API credentials are already set up in your workspace.

```bash
# Install dependencies from repository root (uses npm workspaces)
cd /voc/work/cd14715-claude-code-classroom
npm install

# Navigate to project and configure
cd project/starter
cp .env.example .env
```

**Local Setup:**

```bash
# Clone the repository
git clone https://github.com/udacity/cd14715-claude-code-classroom.git
cd cd14715-claude-code-classroom/project/starter

# Install dependencies
npm install

# Configure environment
cp .env.example .env
```

### Configuration

Set your settings by editing the .env file:

**In Vocareum Workspace:**
```bash
# API credentials are already in your environment - don't add them here

# Model Configuration (REQUIRED)
ANTHROPIC_MODEL=claude-sonnet-4-5-20250929

# Project root (REQUIRED)
PROJECT_ROOT=/voc/work/cd14715-claude-code-classroom/project/starter

# GitHub Token (RECOMMENDED for higher rate limits)
# GITHUB_TOKEN=ghp_your-token-here

# Logging level (optional)
LOG_LEVEL=info
```

**Local Setup with Your Own API Key:**
```bash
# Your Anthropic API key
ANTHROPIC_API_KEY=sk-ant-your-key-here

# Model Configuration (REQUIRED)
ANTHROPIC_MODEL=claude-sonnet-4-5-20250929

# Project root (REQUIRED - update to your path)
PROJECT_ROOT=/absolute/path/to/project/starter

# GitHub Token (RECOMMENDED)
# GITHUB_TOKEN=ghp_your-token-here

# Logging level (optional)
LOG_LEVEL=info
```

### Running

```bash
# Development mode
npm run dev -- <owner> <repo> <pr-number>

# Production build
npm run build
npm start <owner> <repo> <pr-number>

# Example
npm run dev -- facebook react 12345
```

### Testing

```bash
# Run all tests
npm test

# Run specific test
npm test -- orchestrator.test.ts

# Watch mode
npm test -- --watch
```

## Key Technologies

- **Claude Agent SDK** - Multi-agent orchestration framework
- **Model Context Protocol (MCP)** - External data integration
- **Zod** - Schema validation and type safety
- **TypeScript** - Type-safe development
- **Vitest** - Testing framework
- **Winston** - Structured logging

## Success Criteria

Your implementation is complete when:

- [ ] TypeScript compiles without errors: `npm run build`
- [ ] All tests pass: `npm test`
- [ ] Can review a real PR: `npm start owner repo pr-number`
- [ ] Generates reports in at least one format (MD, HTML, JSON)
- [ ] Rate limiting prevents API throttling (Optional)
- [ ] Errors are handled gracefully (Recommended)

## Resources

- [Claude Agent SDK](https://github.com/anthropics/claude-agent-sdk)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Anthropic API Docs](https://docs.anthropic.com/)
- [Zod Documentation](https://zod.dev/)

Good luck!
