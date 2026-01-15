# Skills

This repository contains skills that demonstrate Taro + WeChat Mini Program hybrid development patterns and best practices.

## About This Repository

This repository contains a skill that provides guidance on Taro and native WeChat Mini Program hybrid development. The skill covers architecture patterns, implementation details, and common pitfalls when mixing Taro React components with native WeChat Mini Program components and APIs.

## Installation

### Installing openskills

First, install the openskills CLI tool globally:

```bash
npm install -g openskills
```

This installs the openskills command-line tool, which is required to install and manage skills.

### Installing This Skill

Install this skill in your project using openskills:

```bash
openskills install https://github.com/SteamedBread2333/skills --universal
```

The `--universal` flag installs the skill in the current project, making it available for use with compatible AI agents in this project.

### Syncing Skills

After installing skills, you can sync them to ensure you have the latest versions:

```bash
openskills sync
```

This command will:
- Update all installed skills to their latest versions
- Ensure consistency across your skill installations
- Refresh skill metadata and dependencies

### Managing Skills

- List installed skills: `openskills list`
- Update a specific skill: `openskills install <skill-url> --universal` (re-run the install command)
- Remove a skill: `openskills uninstall <skill-name>`

## Skill

* `skills/taro-wechat-hybrid`: Taro + WeChat Mini Program hybrid development guide

## Usage

This skill covers:
- How to structure projects that mix Taro and native WeChat Mini Program code
- Best practices for component integration
- Routing and navigation patterns
- State management across hybrid boundaries
- Performance optimization techniques
- Common pitfalls and solutions

## License

Apache 2.0
