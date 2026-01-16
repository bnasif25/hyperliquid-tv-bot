# Project Genesis Doc  
**Epic 1 – Bootstrap & Repo**  
*(living document, Markdown in `/docs/epic1-vision.md`)*

---

## 1. Tech Radar @ 2025-07

| Layer | Choice | Rationale |
|---|---|---|
| Language | Python 3.11 | Fast to write, huge trading-ecosystem, runs in 256 MB container |
| Infra-as-Code | Terraform + Fly.io | Serverless containers, free tier, 200 ms cold-start, native edge-deploy |
| Queue / Lock | Upstash Redis | Serverless, pay-per-command, zero ops |
| Secrets | Doppler + container-env | Short-lived tokens, no files on disk, audit log built-in |
| Observability | Grafana Cloud (Prom/Loki) | 50 GB logs & 10 k series free, public dashboard link for portfolio |
| CI/CD | GitHub Actions | Free minutes for public repo, OIDC to cloud, PR-gate keeps main green |
| Dev Env | Dev Container | One-command identical kitchen for every contributor |
| Exchange API | Hyperliquid REST + WS | Deep liquidity, 10 req/s burst, test-net mirror, ed25519 auth |

---

## 2. Architecture Vision (bird-view)

```
TradingView Alert (JSON) ─HTTPS─▶  Fly Edge Gateway (FastAPI)
                                    │ validate + dedupe
                                    ▼
Upstash Redis Stream  ───▶  Trade Engine (Python)
                                    │ idempotent executor
                                    ▼
Hyperliquid WS (atomic bundle) → position + TP/SL + breakeven mover
                                    │
Grafana Cloud ← Prometheus metrics ← audit logs → Backblaze B2
```

**Key qualities baked in during Epic 1**  
- **Zero-downtime blue-green** deploys (Fly)  
- **Cost ceiling ≈ $0.10/mo** infra (always-free tiers)  
- **< 1 s end-to-end** latency budget (gateway → fill)  
- **Portfolio artifact** – public repo + dashboard URL for recruiters  
- **Contributor onboarding < 2 min** (dev-container + Makefile)

---

## 3. Folder Map (set in stone after Epic 1)

```
hyperliquid-tv-bot/
├── src/               # FastAPI gateway + trade engine
├── tests/             # pytest, hypothesis property tests
├── terraform/         # Fly, Upstash, B2, DNS
├── .github/workflows/ # CI (pytest) + CD (terraform plan/apply)
├── .devcontainer/     # reproducible dev-env
├── docs/              # architecture diagrams (draw.io PNG + source)
├── Makefile           # one-button install / test / docker
├── Dockerfile         # multi-stage, distroless runtime
└── fly.toml           # Fly.io edge config
```

---

## 4. Decision Log (why we won’t revisit soon)

| Decision | Argued Alternatives | Final Call |
|---|---|---|
| Public repo | private + GitHub Teams | public = free CI + portfolio showcase |
| Dev Container | local venv, Docker Compose | container = identical kernel/libs on ARM/Mac & CI |
| Fly.io vs. AWS Fargate | Fargate, Render, Railway | Fly = free 256 MB + global edge + native Terraform |
| Terraform vs. Pulumi | Pulumi, CDK | Terraform = widest skill, state-lock free for solo dev |
| Upstash vs. Redis Cloud | Redis Cloud 30 MB free | Upstash = pay-per-cmd, no idle cost, same API |
| Grafana vs. DataDog | DataDog student plan | Grafana = permanent free tier, public link, OSS friendly |

---

## 5. Success Criteria (Epic 1 Exit Gate)

- [ ] Public repo with MIT, branch protection, CI badge green  
- [ ] Dev Container opens in ≤ 2 min on fresh laptop (ARM/Mac & x86/Linux)  
- [ ] `make install && make test` passes inside container  
- [ ] Terraform plan shows **$0** cost estimate (all free tiers)  
- [ ] Azure Boards linked → every future task traceable to commit  

---

