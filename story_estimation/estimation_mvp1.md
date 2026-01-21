
ROLE
You are a senior full‑stack architect and code generator. Produce a complete, production‑grade web application in ONE SHOT. Do not ask any questions. Where details are missing, make best-practice assumptions, state them at the top, and proceed.

PRODUCT CONTEXT
- App Name: Estimator 360
- Purpose: A story-level Agile estimation tool that converts Story Points (SP) to engineering hours and person-days (PD), computes cost before/after savings scenarios, and presents project summaries with charts.
- Users/Roles:
  - ADMIN: Manage users, roles, tech factors, and global settings.
  - PM: Create projects, stories, trigger recalculation, view costs and savings.
  - ENGINEER: Update story fields (status, SP, technology), view summaries.

TECH STACK (MANDATORY)
- Backend: Java 17, Spring Boot 3.x (Spring Web, Spring Security 6 with JWT, Spring Data JPA/Hibernate, Validation, MapStruct, Lombok), Flyway for DB migrations, Testcontainers for integration tests, JUnit 5 + Mockito.
- Frontend: Angular 17 (standalone APIs), TypeScript, Angular Material, Angular Router, RxJS, NgRx Store, Jest (unit) and Cypress (e2e).
- Database: PostgreSQL 15.
- API: REST + OpenAPI 3.0 (YAML) auto-generated & served.
- Packaging/Infra: Maven Wrapper, Dockerfiles (FE & BE), docker-compose (FE+BE+DB+pgAdmin), GitHub Actions CI (build+test+lint+docker).
- Observability: Structured JSON logging; health endpoints (readiness/liveness).
- Config: 12-factor (env vars), secrets via .env/.env.sample (NEVER hardcode credentials).

DOMAIN & DATA MODEL (MANDATORY)
- TechnologyFactor(id, name UNIQUE, hours_per_sp DECIMAL) — seeded values:
  - "Java/J2EE & allied": 12
  - "MEAN Technology Stack": 20
  - "Peoplesoft": 11
  - "Control-M": 8
  - "Quantum": 8
- User(id, name, email UNIQUE, role ENUM[ADMIN,PM,ENGINEER], password_hash, created_at)
- Project(id, name, description, owner_id -> User, created_at)
- Story(id, project_id -> Project, title, description, status ENUM[NEW,IN_PROGRESS,DONE], story_points INT, technology STRING, created_at)
- Estimate(id, story_id -> Story, hours DECIMAL, days DECIMAL, technology_factor DECIMAL, assumptions TEXT, created_at)
- CostScenario(id, project_id -> Project, before_cost DECIMAL, after_cost DECIMAL, savings_pct DECIMAL, methodology STRING, created_at)

ERD: Provide a Mermaid ERD diagram in docs.

CORE BUSINESS LOGIC (MANDATORY)
- Recalculation Rule:
  - hours = story_points × technology_factor.hours_per_sp
  - days = hours / 8
- Project Summary:
  - total_sp = Σ story.story_points (per project)
  - total_hours = Σ estimate.hours (per project) [recompute if missing]
  - total_pd = total_hours / 8
  - cost_before/after and savings_pct computed from a simple scenario model:
    - Use configurable role rates and onsite/offshore split:
      - roleRates (per PD): SENIOR_ENGINEER=£460 onsite/£150 offshore; ENGINEER=£375/£140; SCRUM_MASTER=£400/£140
      - mix (defaults): onsiteShare=0.20, offshoreShare=0.80
    - baseline cost = total_pd × blended_rate_baseline
    - current cost = total_pd × blended_rate_current
    - savings_pct = (baseline - current) / baseline
  - Persist latest scenario to CostScenario per project.
- Validation:
  - story_points >= 0
  - technology must exist in TechnologyFactor.name (enforce via service-level validation).
  - Consistent error responses: RFC7807 problem+json.

BACKEND REQUIREMENTS (MANDATORY)
- Spring Boot 3 project with layers:
  - controller (REST) ↔ service ↔ repository ↔ entity/DTO/mapper (MapStruct)
- Security:
  - BCrypt password hashing
  - JWT access + refresh tokens
  - RBAC via roles (method or URL-level)
  - CORS configured for the Angular origin
- Endpoints (JSON):
  - Auth: POST /auth/signup, POST /auth/login, POST /auth/refresh
  - Users: CRUD (ADMIN only)
  - TechnologyFactors: CRUD (ADMIN), GET list (all roles)
  - Projects: CRUD
  - Stories: CRUD
  - Estimates: CRUD
  - Project Summary: GET /projects/{id}/summary → { total_sp, total_hours, total_pd, cost_before, cost_after, savings_pct, factors }
  - Recalc: POST /estimates/recalculate (body: projectId) → recompute all story estimates using current TechnologyFactors, persist estimates, return updated summary
- OpenAPI:
  - Auto-generate OpenAPI 3 spec and serve it at /api-docs (YAML + Swagger UI).
- Persistence:
  - Flyway migrations to create tables & indexes, plus seed data:
    - 2 users: admin@example.com (ADMIN), pm@example.com (PM)
    - 2 projects, each with 5 stories of mixed technologies and SP
    - TechnologyFactor rows as listed above
    - 1 initial CostScenario per project
- Testing:
  - Unit tests for services
  - Controller slice tests
  - Integration tests using Testcontainers (Postgres) to verify migrations and core flows (login → create project → add stories → recalc → read summary).

FRONTEND REQUIREMENTS (MANDATORY)
- Angular 17 app (standalone components):
  - Routing:
    - /login
    - /projects
    - /projects/:id (tabs: Stories, Estimation, Cost)
    - /admin (Users, Technology Factors) [ADMIN only]
  - State: NgRx slices for auth, projects, stories, estimates, techFactors
  - UI:
    - Angular Material tables (sorting, pagination, filtering)
    - Story edit (inline or modal): title, description, status, SP, technology (dropdown → TechnologyFactor list)
    - Estimation tab: show factors, “Recalculate” button → calls POST /estimates/recalculate
    - Cost tab: expose cost_before/after, savings_pct; show charts (SP vs Hours, Cost Savings %)
  - Auth:
    - JWT storage (prefer httpOnly cookies; if not, localStorage with interceptor guard)
    - Route guards by role
  - Error handling: snackbars/toasts + accessible error summaries
  - i18n scaffold (en) and a11y labels for charts/forms
- Testing:
  - Jest unit tests for components/services
  - Cypress e2e: login → create project → add stories → recalc → verify summary visuals

NON-FUNCTIONAL (MANDATORY)
- Code quality:
  - Backend: Checkstyle/SpotBugs config suitable for CI; meaningful logs (JSON)
  - Frontend: ESLint + Prettier
- Performance:
  - Avoid N+1 (use fetch joins/specifications)
  - DB indexes on FKs and frequent filters
- Config:
  - 12-factor env: DB_URL, JWT_SECRET, TOKEN_TTL, ONSHORE_RATE multipliers, OFFSHORE_RATE multipliers, MIX shares, CORS_ORIGIN, etc.
  - Provide .env.sample files for BE/FE
- Security:
  - Never hardcode secrets
  - Validate/escape all inputs
  - CORS limited to FE origin
  - Basic rate-limit middleware/filter for auth endpoints (if reasonable)

OUTPUT FORMAT (MANDATORY)
Return a ZIP-like virtual file tree **with full file contents** for every file, not snippets:
- /backend
  - pom.xml
  - Dockerfile
  - src/main/java/... (controllers, services, repositories, entities, DTOs, mappers, security, config)
  - src/main/resources/application.yaml
  - src/main/resources/db/migration (Flyway SQL files)
  - src/test/java/... (unit + integration with Testcontainers)
  - OPENAPI.yaml (exported)
- /frontend
  - package.json, tsconfig.json, angular.json, jest.config.ts
  - Dockerfile
  - src/app/** (routes, components, material module, ngrx store, services, interceptors, guards)
  - src/environments/**
  - cypress/** (e2e specs, config)
- /infra
  - docker-compose.yaml (services: backend, frontend, postgres, pgadmin)
  - .env.sample files for FE and BE
- /docs
  - ERD.md with Mermaid diagram
  - ARCHITECTURE.md (context, auth, error model, logging, directory layout)
  - RUNBOOK.md (local dev, migrations/seed, docker compose up, CI, troubleshooting)
- /ci
  - .github/workflows/ci.yml (build, test, lint; BE with Maven cache; FE with node cache)

COMMANDS (MANDATORY)
- Local backend: ./mvnw spring-boot:run
- Local frontend: npm ci && npm run start
- Migrations/seed: Flyway runs automatically on startup; include a standalone seed runner or data.sql alternatives if needed
- Tests: 
  - Backend: ./mvnw -q -Dtest=* test (include Testcontainers)
  - Frontend: npm run test (Jest); npm run e2e (Cypress)
- Docker: docker compose up -d (FE at http://localhost:4200, BE at http://localhost:8080, Swagger at /api-docs, pgAdmin on default port)

STYLE (MANDATORY)
- Deterministic: do not ask me any clarifying questions
- Make reasonable assumptions and list them in a top “Assumptions” section
- Production-ready code only (no pseudocode, no TODOs)
- Ensure the app starts locally with the documented commands
``
