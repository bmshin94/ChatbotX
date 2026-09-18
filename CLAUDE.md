# ChatbotX (ChatbotXIO/ChatbotX)

## 프로젝트 개요
인스타그램, 카카오톡, 웹사이트 등 다양한 채널로 쏟아지는 고객 문의를 AI가 24시간 실시간으로 상담하고 응대하는 "오픈소스 옴니채널 고객 상담 센터"
고가의 유료 챗봇 솔루션을 대체하여 나만의 서버에서 안전하고 저렴하게 최신 대화형 AI 고객 지원 시스템을 구축
쇼핑몰 운영자부터 스타트업까지 단 한 명의 고객도 놓치지 않고 친절하고 신속하게 응대하는 충실한 고객 관리 솔루션

## 핵심 특징 & 추천 분야
- 옴니채널고객상담
- 24시간AI응대비서
- 오픈소스챗봇센터
- 고객문의원스톱해결
- 스마트고객경험

---
*이 문서는 오픈소스 큐레이터(Curator-Agent)에 의해 자동 생성된 가이드 문서입니다.*


---
## 기존 CLAUDE.md 내용

@AGENTS.md

## Claude Code — additional guidance

### Preferred workflow

1. Read the relevant skill file in `.agents/skills/` before writing code for a new feature, API, database table, worker, or integration.
2. When touching `packages/database`, always run `pnpm --filter @chatbotx.io/database db:migrate` after schema changes.
3. After any code change, run `pnpm lint` and `pnpm --filter <app> check-types` before reporting done.
4. Use `pnpm fix` (Biome auto-fix) instead of manually formatting code.

### Skill → task mapping

| Task | Skill to read first |
|------|---------------------|
| Broad/ambiguous request, onboarding, "where does X live" | `chatbotx-basecode` |
| New feature / page | `feature-scaffold` |
| Builder UI component, form, table, dialog, or any user-facing string | `builder-ui-i18n` |
| New API endpoint | `orpc-api` |
| Business logic, new service method, any DB read/write from app code | `business-data-access` |
| New DB table or migration | `drizzle-database` |
| New background job or queue | `worker-development` |
| New channel integration | `integration-channel` |
| Contact filter field/operator, filter SQL, or contact-based audience | `contact-filter` |
| Facebook/Messenger comment automation (auto-reply/like/hide on Page post comments) | `fb-comment-automation` |
| Minigame tool (Jackpot CRUD, prize draw, public play link/token, extending to a new minigame type) | `minigame` |
| New flow step with states (success/error/skip routing) | `flow-step-development` |
| Dev/build/lint commands | `turborepo-workflow` |
| Approved implementation plan | `implement-plan` |
| Security-sensitive change (auth, scoping, webhooks, AI tools, permissions) | `security-review` |
| Writing tests / verifying a change is done | `testing-workflow` |
| Concurrent code (worker jobs, migrations, replace-writes) | `reliability-concurrency` |

### Specialist agents (`.claude/agents/`)

Dispatch these project subagents by name when relevant:

| Agent | When |
|-------|------|
| `invariant-guard` | after editing `apps/`/`packages/`/`integrations/` — checks the ChatbotX invariants no linter enforces |
| `rag-eval` | changes under `packages/ai` or the embedding repositories/handlers |
| `incident-responder` | triaging a production error or failed job |

Reviewers, planners, build-fixers, etc. come from the `~/.claude/` global set — don't recreate them here.

### Model routing (cost)

Default to the cheapest tier that fits the task; reserve the top tier for judgment:

| Task class | Tier |
|------------|------|
| file lookup, grep, inventory, verification/refutation | Haiku / Sonnet |
| implementation, code review, debugging | Sonnet |
| architecture, synthesis, incident root-cause | Opus |

### Never do without checking

- Do not use `git add -A` or `git add .` — stage specific files only.
- Do not commit `.env` files or secrets.
- Do not skip `pnpm lint` — the CI will fail.
- Do not hardcode user-facing strings — use `useTranslations()`.
- Do not import `db` directly in `apps/` or `integrations/` — the chain is `action | API handler → service (@chatbotx.io/business) → repository (@chatbotx.io/database/repositories) → DB`; app code calls a service, never a repository directly (the one exception is a pure read with zero business logic). See `.agents/rules/data-access.md`.
- Do not use dynamic `import()` in tsdown-built code (`packages/*`, `integrations/*`, `apps/worker`, `apps/cli`, `apps/mcp-server`, `apps/javascript-executor`) — it breaks the tsdown build. In `apps/builder/src` dynamic imports and `next/dynamic` are allowed (and preferred for heavy client islands). See `.agents/rules/no-dynamic-import.md`.
