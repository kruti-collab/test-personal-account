---
allowed-tools: Bash, Read, Grep, Glob, TodoWrite
description: Answer questions about GAL API endpoints, auth flows, and services without coding
argument-hint: [question]
---

# API Expert - Question Mode

Answer the user's question by analyzing the Express.js API implementation, authentication flows, Stripe billing, and service layer in the GAL project. This prompt is designed to provide information about the backend without making any code changes.

## Variables

USER_QUESTION: $1
EXPERTISE_PATH: .claude/commands/experts/api/expertise.yaml

## Instructions

- IMPORTANT: This is a question-answering task only - DO NOT write, edit, or create any files
- Focus on API endpoints, Express middleware, authentication (OAuth, JWT, OIDC), Stripe billing, and service patterns
- If the question requires API changes, explain the implementation steps conceptually without implementing
- With your expert knowledge, validate the information from `EXPERTISE_PATH` against the codebase before answering your question

## Workflow

1. Read the `EXPERTISE_PATH` file to understand API architecture and patterns
2. Review, validate, and confirm information from `EXPERTISE_PATH` against the actual codebase
3. Search relevant files in `apps/api/src/` to verify your understanding
4. Respond based on the `Report` section below

## Key Files to Reference

- `apps/api/src/index.ts` - Main entry point with all endpoints
- `apps/api/src/services/` - Service layer implementations
- `apps/api/src/config/` - Configuration and feature flags
- `apps/api/SECURITY.md` - Security documentation

## Report

- Direct answer to the `USER_QUESTION`
- Supporting evidence from `EXPERTISE_PATH` and the codebase
- References to the exact files and lines of code that support the answer
- If expertise file is outdated, note what needs updating
