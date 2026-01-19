---
allowed-tools: Bash, Read, Grep, Glob, TodoWrite
description: Answer questions about E2E tests, Playwright patterns, and test infrastructure without coding
argument-hint: [question]
---

# E2E Expert - Question Mode

Answer the user's question by analyzing the Playwright E2E test implementation, test patterns, and CI integration in the GAL dashboard. This prompt is designed to provide information about testing without making any code changes.

## Variables

USER_QUESTION: $1
EXPERTISE_PATH: .claude/commands/experts/e2e/expertise.yaml

## Instructions

- IMPORTANT: This is a question-answering task only - DO NOT write, edit, or create any files
- Focus on Playwright tests, page objects, authentication setup, and CI workflows
- If the question requires test changes, explain the approach conceptually without implementing
- With your expert knowledge, validate the information from `EXPERTISE_PATH` against the codebase before answering your question

## Workflow

1. Read the `EXPERTISE_PATH` file to understand E2E test architecture and patterns
2. Review, validate, and confirm information from `EXPERTISE_PATH` against the actual codebase
3. Search relevant files in `apps/dashboard/e2e/` to verify your understanding
4. Respond based on the `Report` section below

## Key Files to Reference

- `apps/dashboard/playwright.config.ts` - Playwright configuration
- `apps/dashboard/e2e/setup/global-setup.ts` - Auth setup
- `apps/dashboard/e2e/ui/` - Test files organized by type
- `apps/dashboard/e2e/SKIPPED_TESTS.md` - Skipped tests tracking
- `.github/workflows/e2e-*.yml` - CI workflow files

## Report

- Direct answer to the `USER_QUESTION`
- Supporting evidence from `EXPERTISE_PATH` and the codebase
- References to the exact files and lines of code that support the answer
- If expertise file is outdated, note what needs updating
