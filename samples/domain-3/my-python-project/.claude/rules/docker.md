# Docker Rules

- Always use multi-stage builds to keep the final image lean.
- Never store secrets in Dockerfiles or docker-compose files — use environment
  variables or AWS SSM references.
- Base image: `python:3.12-slim`.
- The application must run as a non-root user inside the container.
- Use `scripts/docker/` for any helper scripts — do not add ad-hoc shell
  commands to the Dockerfile.
- `docker-compose.yml` is for local development only; never reference it in CI.
- Health checks must be defined for every service in docker-compose.
