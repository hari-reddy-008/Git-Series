# Enterprise Multi-Agent Code Review Orchestrator

A production-ready multi-agent system which, using the Claude Agent SDK, automates the processes of code review, test coverage analysis, and providing refactoring suggestions for GitHub pull requests.

## Features

- **Multi-Agent Architecture**: It coordinates three specialized sub-agents in order to carry out a thorough PR analysis.
- **Code Quality Analysis**: This analysis covers security, performance, maintainability, and adherence to coding best practices.
- **Test Coverage Analysis**: It is able to identify code paths that may not have been tested and gives advice on tests.
- **Suggestions for refactoring**: These highlight opportunities for modernization, improving maintainability, and enhancing the design.
- **MCP Integration**: It makes use of the GitHub MCP to obtain pull-request and repository data as well as the ESLint MCP for linting support.
- **Claude Skills**: Has specialized Skills for JavaScript, TypeScript, and for analysis with a focus on security.
- **Structured Output**: The final review is checked against the report schemas that were defined.
- **Production-Grade Reliability**: It features rate limiting, retry handling, structured logging, validation, and error handling.
- **Multiple Report Formats**: It can produce reports in JSON, Markdown, and HTML format.

## Architecture

### Orchestrator

Main coordinator that:

The GitHub repository owner, the name of the repository, and the number of the pull request are obtained from the command line.
2. Gets the pull request information and the files that have changed via the GitHub MCP.
3. It arranges the three specialized subagents to carry out the analysis.
4. It compiles the individual results from the subagents into a single, structured review report.
5. Checks that the generated structured output conforms to the project schema.
6. It produces reports in JSON, Markdown, and HTML format.

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

Enter the necessary credentials and project settings into the .env file:

```text
ANTHROPIC_API_KEY=sk-ant-your-key-here
GITHUB_TOKEN=ghp_your-token-here

The absolute path to the project root that contains the .claude/skills/ directory.
PROJECT_ROOT=/absolute/path/to/project/starter

LOG_LEVEL=info
```

You should not include a file called `.env` or actual API credentials in the repository.

### MCP Servers

Configured in `src/config/mcp.config.ts`:

- The GitHub MCP offers information regarding pull requests and access to repository files.
- The ESLint MCP offers analysis support related to ESLint.

### Claude Skills

Located in `.claude/skills/`:

- `javascript-best-practices` - an analysis of JavaScript coding and best practices.
- `typescript-patterns` - a set of patterns specific to TypeScript and an analysis of them.
- `security-analysis` - An analysis that is focused on security and concerns secure coding.

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

The attempt to use the `airaamane/simple-todo-app` validation target failed because the repository could not be accessed; instead, the pull requests listed above were employed for the successful end-to-end validation.

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

The successful completion of the TypeScript production build was achieved using the command `npm run build`.

## Output

The system produces JSON, Markdown, and HTML reports in the reports/ directory.

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
│   ├── main.ts                 # Command-line interface entry point
│   ├── orchestrator.ts          # The main orchestrator
│   ├── agents/                 # Specialized subagent definitions
│   │   ├── code-quality-analyzer.ts
│   │   ├── test-coverage-analyzer.ts
│   │   └── refactoring-suggester.ts
│   ├── config/
│   │   └── mcp.config.ts       # MCP server settings
│   ├── prompts/
│   │   └── index.ts            # contains the orchestrator and agent prompts
│   ├── types/
│   │   ├── analysis-results.ts # the output schemas of the subagent
│   │   └── report-types.ts    # the schema for the final report
│   └── utils/
│       ├── logger.ts            # Structured logging
│       ├── rate-limiter.ts  # A sliding-window rate limiter
│       ├── error-handler.ts    # Handles retries and errors
│       ├── report-generator.ts # which generates reports in JSON, Markdown, and HTML format
│       └── index.ts             # The utility exports
├── .claude/
│   └── skills/                 # Claude Skills definitions
├── tests/                      # Automated test suite
├── reports/                    # Generated review reports
.env.example                # Environment template
└── package.json
```

## Production Reliability

### Rate Limiting

The project makes use of a sliding-window rate limiter in order to control the number of requests and tokens used.

The system keeps record of recent requests, deletes any entries outside the specified time window, verifies that the request and token limits are not exceeded before carrying on, and waits if those limits have been reached.

The limiter also provides for the control of concurrent requests and for the accounting of tokens associated with completed requests.

### Error Handling

- Include retrying with an exponential backoff.
- Apply random jitter to the retry attempts.
- The exhaustion resulting from a retry is turned into a structured ReviewError.
- The handling of timeouts for asynchronous operations.
- Organized error codes and clear error messages.
The ability to handle failures gracefully throughout the review process.

### Observability

The project makes use of the logger utility for structured logging.

Logs can capture events associated with:

- Application startup
- Authentication configuration
- Review execution
- Report generation
- Errors and failures

The review reports which are generated also include metadata for example the time of the analysis and information regarding the execution.

## Limitations

The system needs valid GitHub access in order to obtain information about pull requests and repository files.
The execution of an end-to-end review can be affected by the availability of the GitHub API and the MCP.
You need valid login details to analyze a real pull request.
- Since the `airaamane/simple-todo-app` validation repository could not be accessed during the validation process, the fallback pull requests provided by the course were used to achieve a successful integration test.
The reports produced include analysis generated by the model and must therefore be checked before they are regarded as official code-review decisions.
The availability of the API or model and the rate limits may have an impact on the time it takes to carry out the execution.

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

A sliding time window is used by the rate limiter; if requests are temporarily limited then either wait for the current window to expire or modify the limits as specified in the rate-limiter implementation.

### "GitHub API error"

Verify that:

- `GITHUB_TOKEN` is configured.
- The token has the required access to the repository.
- The repository in question and the pull request are there and can be accessed.
- The GitHub MCP server manages to start.

### "Structured output validation failed"

Check the schemas in:

```text
src/types/analysis-results.ts
src/types/report-types.ts
```

Make sure that the prompts used by the orchestrator and the subagent ask for output that conforms to the expected schemas.

### "Repository not found"

Check that the owner, the name of the repository, and the pull-request number are correct.

When validating the required `airaamane/simple-todo-app` target, the repository could not be accessed. Fallback public pull requests are provided in the course instructions.

## License

ISC

## Contributing

It is a project carried out as part of a Udacity course which shows how to use the Claude Agent SDK, GitHub MCP, ESLint MCP, Claude Skills, structured outputs, automated testing, and production-oriented reliability features in a multi-agent code-review system.
