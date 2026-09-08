# Project: My Python Project

## Rules

@import .claude/rules/python-code.md
@import .claude/rules/documentation.md
@import .claude/rules/testing.md
@import .claude/rules/project-structure.md
@import .claude/rules/docker.md

## Project Context

Key directories to consult when working on this project:

- `docs/business-rules/` — domain rules and business logic definitions
- `docs/project/` — architecture decisions, ADRs, onboarding guides
- `docs/api/` — API contracts and endpoint specifications
- `docs/postman/` — Postman collections for manual and automated testing
- `scripts/docker/` — scripts to build, run, and manage containers
- `scripts/db/` — seed scripts, migration helpers, maintenance routines
- `scripts/aws/` — scripts for interacting with AWS services (S3, SSM, ECR, etc.)

Before implementing a feature, check `docs/business-rules/` for domain context.
Before writing an endpoint, check `docs/api/` for the existing contract.
