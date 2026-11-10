# services/companion — EC2 companion service

Docker-based service on one EC2 instance (Section 8.3, 16). Responsibilities:
- Import worker (batch work that exceeds Edge Function limits)
- Health endpoint
- Optional AI adapter host (only if DP-1 selects **Pattern B**)
- Optional internal APK download page

Runs behind a TLS reverse proxy (Caddy/Nginx). Non-root, read-only root FS where possible,
image pinned by digest. Secrets read at container start from AWS Secrets Manager / SSM — never
baked into the image or committed. **No database access from the AI adapter host.**

Built out in **Phase 11**. The `Dockerfile` and `docker-compose.yml` here are placeholders.
