# Enterprise Multi-Agent Code Review Orchestrator

Production-ready multi-agent system that automates code review, test coverage analysis, and refactoring suggestions for GitHub pull requests using Claude Agent SDK.

## Features

- **Multi-Agent Architecture**: Coordinates 3 specialized subagents for comprehensive PR analysis.
- **Code Quality Analysis**: Analyzes security, performance, maintainability, and coding best practices.
- **Test Coverage Analysis**: Identifies potentially untested code paths and provides test recommendations.
- **Refactoring Suggestions**: Identifies modernization, maintainability, and design improvement opportunities.
- **MCP Integration**: Uses GitHub MCP for pull-request and repository data and ESLint MCP for linting support.
- **Claude Skills**: Uses dedicated Skills for JavaScript, TypeScript, and security-focused analysis.
- **Structured Output**: Validates the final review against the defined report schemas.
- **Production-Grade Reliability**: Includes rate limiting, retry handling, structured logging, validation, and error handling.
- **Multiple Report Formats**: Generates JSON, Markdown, and HTML reports.

## Architecture

### Orchestrator

Main coordinator that:

1. Receives the GitHub repository owner, repository name, and pull-request number from the command line.
2. Retrieves pull-request information and changed files through GitHub MCP.
3. Coordinates the three specialized subagents for analysis.
4. Aggregates the subagent results into a unified structured review report.
5. Validates the generated structured output against the project schema.
6. Generates JSON, Markdown, and HTML reports.

### Subagents

| **Agent** | **Purpose** | **Key Features** |
| --- | --- | --- |
| **code-quality-analyzer** | Security, performance, maintainability, and code quality | Uses Claude Skills and severity-based findings |
| **test-coverage-analyzer** | Identifies potentially untested code paths | Reviews implementation and tests and provides coverage recommendations |
| **refactoring-suggester** | Identifies refactoring and modernization opportunities | Provides actionable refactoring suggestions and impact information |

## Prerequisites

- Node.js 20+
- Anthropic API key or supported AWS authentication
- GitHub personal access token with repository read access
- TypeScript 5.3+

## Installation

```bash
# Navigate to the project
cd project/starter

# Install dependencies
npm install

# Copy environment template
cp .env.example .env
```

## Configuration

Edit `.env` with the required credentials and project configuration:

```text
ANTHROPIC_API_KEY=sk-ant-your-key-here
GITHUB_TOKEN=ghp_your-token-here

# Absolute path to the project root containing .claude/skills/
PROJECT_ROOT=/absolute/path/to/project/starter

LOG_LEVEL=info
```

Do not commit `.env` or real API credentials to the repository.

### MCP Servers

Configured in `src/config/mcp.config.ts`:

- **GitHub MCP**: Provides pull-request information and repository file access.
- **ESLint MCP**: Provides ESLint-related analysis support.

### Claude Skills

Located in `.claude/skills/`:

- `javascript-best-practices` - JavaScript coding and best-practice analysis.
- `typescript-patterns` - TypeScript-specific patterns and analysis.
- `security-analysis` - Security-focused and secure-coding analysis.

## Usage

### Run Code Review

```bash
npm run dev -- <owner> <repo> <pr-number>
```

Example:

```bash
npm run dev -- octocat Hello-World 1
```

Additional real GitHub pull requests used during validation:

```bash
npm run dev -- shinshin86 todo-opfs-sqlite 1
npm run dev -- lucaong minisearch 295
npm run dev -- lucaong minisearch 305
```

The required `airaamane/simple-todo-app` validation target was attempted, but the repository was inaccessible. The fallback pull requests above were used for successful end-to-end validation.

### Build for Production

```bash
npm run build
npm start
```

The command-line arguments are supplied to the development command as:

```bash
npm run dev -- <owner> <repo> <pr-number>
```

### Development

```bash
# Type checking
npm run lint

# Run tests
npm test

# Watch mode
npm run test:watch
```

The completed test suite passed with:

```text
Test Files: 4 passed
Tests: 65 passed | 1 skipped
```

The TypeScript production build also completed successfully with `npm run build`.

## Output

The system generates JSON, Markdown, and HTML reports under `reports/`.

Example report structure:

```text
{
  "pullRequest": {
    "owner": "octocat",
    "repo": "Hello-World",
    "number": 1
  },
  "fileReviews": [
    {
      "file": "example.ts",
      "codeQuality": {
        "issues": [...]
      },
      "testCoverage": {
        "untestedPaths": [...]
      },
      "refactorings": {
        "suggestions": [...]
      }
    }
  ],
  "summary": {
    "totalFiles": 1,
    "criticalIssues": 0,
    "highPriorityTests": 0,
    "refactoringOpportunities": 0
  },
  "recommendations": [...],
  "metadata": {
    "analyzedAt": "...",
    "duration": "...",
    "agentVersions": {...}
  }
}
```

Generated validation reports include:

```text
reports/
├── octocat_Hello-World_1.json
├── octocat_Hello-World_1.md
├── octocat_Hello-World_1.html
├── shinshin86_todo-opfs-sqlite_1.json
├── shinshin86_todo-opfs-sqlite_1.md
├── shinshin86_todo-opfs-sqlite_1.html
├── lucaong_minisearch_295.json
├── lucaong_minisearch_295.md
├── lucaong_minisearch_295.html
├── lucaong_minisearch_305.json
├── lucaong_minisearch_305.md
└── lucaong_minisearch_305.html
```

## Project Structure

```text
project/starter/
├── src/
│   ├── main.ts                 # CLI entry point
│   ├── orchestrator.ts         # Main orchestrator
│   ├── agents/                 # Specialized subagent definitions
│   │   ├── code-quality-analyzer.ts
│   │   ├── test-coverage-analyzer.ts
│   │   └── refactoring-suggester.ts
│   ├── config/
│   │   └── mcp.config.ts       # MCP server configuration
│   ├── prompts/
│   │   └── index.ts            # Orchestrator and agent prompts
│   ├── types/
│   │   ├── analysis-results.ts # Subagent output schemas
│   │   └── report-types.ts     # Final report schema
│   └── utils/
│       ├── logger.ts            # Structured logging
│       ├── rate-limiter.ts     # Sliding-window rate limiter
│       ├── error-handler.ts    # Retry and error handling
│       ├── report-generator.ts # JSON, Markdown, and HTML reports
│       └── index.ts             # Utility exports
├── .claude/
│   └── skills/                 # Claude Skills definitions
├── tests/                      # Automated test suite
├── reports/                    # Generated review reports
├── .env.example                # Environment template
└── package.json
```

## Production Reliability

### Rate Limiting

The project implements a sliding-window rate limiter to control request and token usage.

The implementation tracks recent requests, removes records outside the configured time window, checks request and token limits before proceeding, and waits when the limits are reached.

The limiter also supports concurrent request control and token accounting for completed requests.

### Error Handling

- Retry handling with exponential backoff.
- Randomized jitter between retry attempts.
- Retry exhaustion is converted into a structured `ReviewError`.
- Timeout handling for asynchronous operations.
- Structured error codes and formatted error messages.
- Graceful handling of failures during the review workflow.

### Observability

The project includes structured logging through the logger utility.

Logs can capture events associated with:

- Application startup
- Authentication configuration
- Review execution
- Report generation
- Errors and failures

Generated review reports also contain metadata such as analysis time and execution information.

## Limitations

- The system requires valid GitHub access to retrieve pull-request information and repository files.
- GitHub API and MCP availability can affect end-to-end review execution.
- Real pull-request analysis requires valid authentication credentials.
- The required `airaamane/simple-todo-app` validation repository was inaccessible during validation, so the course-provided fallback pull requests were used for successful integration testing.
- Generated reports contain model-produced analysis and should be reviewed before being treated as authoritative code-review decisions.
- API/model availability and rate limits can affect execution time.

## Troubleshooting

### "Skills not loading"

Ensure `PROJECT_ROOT` points to the absolute project directory containing:

```text
.claude/skills/
```

For example:

```text
PROJECT_ROOT=/absolute/path/to/project/starter
```

### "Rate limit exceeded"

The rate limiter uses a sliding time window. If requests are temporarily limited, wait for the active window to clear or adjust the configured limits in the rate-limiter implementation.

### "GitHub API error"

Verify that:

- `GITHUB_TOKEN` is configured.
- The token has the required repository access.
- The target repository and pull request exist and are accessible.
- The GitHub MCP server can start successfully.

### "Structured output validation failed"

Check the schemas in:

```text
src/types/analysis-results.ts
src/types/report-types.ts
```

Also verify that the orchestrator and subagent prompts request output matching the expected schemas.

### "Repository not found"

Confirm the owner, repository name, and pull-request number.

For the required `airaamane/simple-todo-app` target, the repository was inaccessible during validation. The course instructions provide fallback public pull requests for this situation.

## License

ISC

## Contributing

This is a Udacity course project demonstrating a multi-agent code-review system using the Claude Agent SDK, GitHub MCP, ESLint MCP, Claude Skills, structured outputs, automated testing, and production-oriented reliability features.
