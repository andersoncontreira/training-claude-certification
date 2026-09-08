# Project Structure

Expected directory layout:

```
src/
  <domain>/
    models/         # Pydantic/dataclass models
    services/       # Business logic classes
    repositories/   # Data access classes
    controllers/    # HTTP layer (if applicable)
tests/
  unit/
  integration/
scripts/
  docker/           # build.sh, run.sh, prune.sh
  db/               # seed.py, reset.py, migrate.sh
  aws/              # s3_sync.py, ssm_get.py, ecr_push.sh
docs/
  business-rules/   # .md files per domain/entity
  project/          # ADRs, architecture diagrams, onboarding
  api/              # OpenAPI specs or Markdown endpoint docs
  postman/          # .json collection files
```

Rules:

- Do not create files outside these locations without asking.
- One class per file.
- File name must match the class name in snake_case.
