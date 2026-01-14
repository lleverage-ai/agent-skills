# Lleverage Dev Skills

Lleverage agent skills for Claude Code development workflows.

## Installation

Add the marketplace to Claude Code:

```bash
claude plugin marketplace add lleverage-ai/lleverage-dev-skills
```

Install the dev-bot plugin:

```bash
claude plugin install dev-bot@lleverage-dev-skills
```

## Plugins

### dev-bot

Lleverage dev-bot skills for Claude Code development workflows.

#### Commands

- `/dev-bot:code-review` - Automated code review for pull requests using multiple specialized agents with confidence-based scoring

```bash
/dev-bot:code-review https://github.com/owner/repo/pull/123
```

## Development

To test plugins locally:

```bash
claude plugin install /path/to/lleverage-dev-skills/plugins/dev-bot --scope user
```
