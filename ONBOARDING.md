# API Platform — New Employee Onboarding Guide

This guide is for engineers joining the team. It covers the dev environment, code patterns, and deployment.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture](#2-architecture)
3. [Tech Stack](#3-tech-stack)
4. [Local Development Setup](#4-local-development-setup)
5. [Project Structure](#5-project-structure)
6. [Key URLs & Services](#6-key-urls--services)
7. [Backend Patterns (API)](#7-backend-patterns-api)
8. [Frontend Patterns (PWA)](#8-frontend-patterns-pwa)
9. [Database & Migrations](#9-database--migrations)
10. [Real-time with Mercure](#10-real-time-with-mercure)
11. [Testing](#11-testing)
12. [Production Deployment](#12-production-deployment)
13. [Token Evaluation](#13-token-evaluation)

---

## 1. Project Overview

This is a full-stack web application built on **API Platform 4** — a framework for building API-first projects. The stack ships a:

- **REST / Hypermedia API** (JSON-LD, Hydra, OpenAPI) served by a PHP/Symfony backend
- **Next.js Progressive Web App** (PWA) as the frontend
- **Auto-generated Admin UI** powered by `@api-platform/admin` (React Admin + HydraAdmin)
- **Real-time push** via Mercure (embedded in the web server)
- Everything runs in **Docker** (local) or **Kubernetes** (production)

---

## 2. Architecture

```
Browser
  │
  ├── GET / ───────────────────► PWA (Next.js, port 3000 internally)
  ├── GET /docs ───────────────► Swagger UI  ─┐
  ├── GET /admin ──────────────► React Admin  │ All served via FrankenPHP
  ├── GET /api/* ──────────────► REST API  ───┤ (Caddy-based PHP server)
  └── SSE /.well-known/mercure► Mercure hub ─┘
                                      │
                                 PostgreSQL 16
```

### Services (Docker Compose)

```
┌─────────────────────────────────────────────────────┐
│  php  (FrankenPHP)                                  │
│  ├── Symfony 7.2 app  (API Platform 4.2)            │
│  ├── Caddy web server  (HTTP/HTTPS/HTTP3)            │
│  └── Mercure hub  (real-time SSE, embedded in Caddy)│
├─────────────────────────────────────────────────────┤
│  pwa  (Next.js 15 / Node.js)                        │
│  └── Proxied through FrankenPHP (PWA_UPSTREAM)      │
├─────────────────────────────────────────────────────┤
│  database  (PostgreSQL 16)                          │
└─────────────────────────────────────────────────────┘
```

The `php` container is the **single public entrypoint** — it forwards requests for the PWA to the `pwa` container internally, so users always hit one HTTPS host.

---

## 3. Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Language (API) | PHP | ≥ 8.4 |
| Framework (API) | Symfony | 7.2 |
| API layer | API Platform | 4.2 |
| ORM | Doctrine ORM | ^3.0 |
| Web server | FrankenPHP (Caddy) | 1-php8.4 |
| Language (PWA) | TypeScript | ^5.8 |
| Framework (PWA) | Next.js | ^15 |
| UI | React | ^19 |
| Styling | Tailwind CSS | ^4 |
| Admin UI | @api-platform/admin (HydraAdmin) | ^4.0 |
| Data fetching | TanStack React Query | ^5 |
| Form handling | Formik | ^2.4 |
| Database | PostgreSQL | 16 |
| Real-time | Mercure | embedded |
| E2E tests | Playwright | ^1.50 |
| Package manager (PWA) | pnpm | 9.1.3 |

---

## 4. Local Development Setup

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (or Docker + Compose plugin)
- Git

### First-time setup

```bash
# 1. Clone the repository
git clone <repo-url>
cd api-platform

# 2. Start all services (builds images on first run)
docker compose up --build -d

# 3. Wait for health checks to pass (~60 s)
docker compose ps   # all services should show "(healthy)"
```

That's it. All dependencies (Composer, pnpm, DB) are handled inside Docker.

### Daily workflow

```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# View logs
docker compose logs -f php
docker compose logs -f pwa

# Rebuild after Dockerfile changes
docker compose up --build -d php
```

### Useful container commands

```bash
# Run a Symfony console command
docker compose exec php bin/console <command>

# Run database migrations
docker compose exec php bin/console doctrine:migrations:migrate

# Open a shell in the PHP container
docker compose exec php sh

# Open a shell in the PWA container
docker compose exec pwa sh

# Run pnpm commands in PWA
docker compose exec pwa pnpm <command>
```

### Xdebug

Xdebug is installed in dev but **off by default**. Enable by setting the env variable before starting:

```bash
XDEBUG_MODE=debug docker compose up -d php
```

### Quick-start Checklist

- [ ] Docker Desktop installed and running
- [ ] `docker compose up --build -d` succeeds
- [ ] `https://localhost/` loads (accept self-signed cert)
- [ ] `https://localhost/docs` shows Swagger UI
- [ ] `https://localhost/admin` shows the admin interface
- [ ] Run `docker compose exec php bin/console list` to see all CLI commands

---

## 5. Project Structure

```
api-platform/
├── api/                        ← PHP/Symfony backend
│   ├── src/
│   │   ├── Entity/             ← Doctrine entities (= API resources)
│   │   ├── Controller/         ← Custom Symfony controllers (rarely needed)
│   │   └── Repository/         ← Doctrine repositories
│   ├── config/
│   │   ├── packages/           ← Bundle config (api_platform, security, mercure…)
│   │   └── routes.yaml         ← Route definitions
│   ├── migrations/             ← Doctrine migration files
│   ├── tests/                  ← PHPUnit tests
│   ├── frankenphp/             ← Caddyfile + Docker entrypoint + PHP ini
│   ├── Dockerfile              ← Multi-stage (base / dev / prod)
│   └── composer.json
│
├── pwa/                        ← Next.js frontend
│   ├── pages/
│   │   ├── index.tsx           ← Welcome / landing page
│   │   └── admin/index.tsx     ← Admin UI (loads HydraAdmin client-side)
│   ├── components/
│   │   ├── admin/App.tsx       ← HydraAdmin component
│   │   └── common/Layout.tsx   ← React Query provider wrapper
│   ├── public/                 ← Static assets
│   ├── next.config.js
│   └── package.json
│
├── e2e/                        ← Playwright end-to-end tests
│   └── tests/example.spec.js
│
├── helm/                       ← Kubernetes Helm chart
├── compose.yaml                ← Base Docker Compose (all envs)
├── compose.override.yaml       ← Dev overrides (volumes, hot-reload)
└── compose.prod.yaml           ← Production overrides (build targets)
```

---

## 6. Key URLs & Services

| URL | What it is |
|---|---|
| `https://localhost/` | Welcome page (Next.js) |
| `https://localhost/docs` | Interactive Swagger / OpenAPI UI |
| `https://localhost/admin` | Auto-generated React Admin |
| `https://localhost/api` | REST API root (JSON-LD) |
| `https://localhost/.well-known/mercure/ui/` | Mercure debug UI |
| `localhost:5432` | PostgreSQL (dev only, exposed) |

> **HTTPS only.** FrankenPHP generates a self-signed cert for `localhost` automatically. Accept the browser warning on first visit, or trust the cert.

---

## 7. Backend Patterns (API)

### Exposing a resource

The core pattern: **a PHP class + two attributes = a full REST API**.

```php
// api/src/Entity/Greeting.php
namespace App\Entity;

use ApiPlatform\Metadata\ApiResource;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

#[ApiResource(mercure: true)]     // ← exposes all CRUD operations + Mercure real-time
#[ORM\Entity]
class Greeting
{
    #[ORM\Id]
    #[ORM\Column(type: 'integer')]
    #[ORM\GeneratedValue(strategy: 'SEQUENCE')]
    private ?int $id = null;

    #[ORM\Column]
    #[Assert\NotBlank]            // ← automatic validation
    public string $name = '';

    public function getId(): ?int { return $this->id; }
}
```

This single class generates:
- `GET /api/greetings` — paginated list
- `POST /api/greetings` — create
- `GET /api/greetings/{id}` — read one
- `PUT/PATCH /api/greetings/{id}` — update
- `DELETE /api/greetings/{id}` — delete
- Full OpenAPI documentation for all of the above

### Customising operations

```php
use ApiPlatform\Metadata\Get;
use ApiPlatform\Metadata\GetCollection;
use ApiPlatform\Metadata\Post;

#[ApiResource(
    operations: [
        new GetCollection(),
        new Post(),
        new Get(),
        // PUT/PATCH/DELETE intentionally omitted → disabled
    ]
)]
```

### Validation

Use Symfony Validator constraints directly as PHP attributes on properties:

```php
use Symfony\Component\Validator\Constraints as Assert;

#[Assert\NotBlank]
#[Assert\Length(max: 255)]
public string $name = '';

#[Assert\Email]
public string $email = '';
```

Validation errors are returned automatically as RFC 7807 Problem Details responses.

### Configuration

API Platform global settings live in `api/config/packages/api_platform.yaml`:

```yaml
api_platform:
    title: Hello API Platform
    version: 1.0.0
    mercure:
        include_type: true
    defaults:
        stateless: true
        cache_headers:
            vary: ['Content-Type', 'Authorization', 'Origin']
```

### Security

Security is configured in `api/config/packages/security.yaml`. The default setup has no authentication — add JWT (`lexik/jwt-authentication-bundle`) or OAuth for real projects.

Access control per operation:

```php
#[ApiResource(
    operations: [
        new GetCollection(),
        new Post(security: "is_granted('ROLE_ADMIN')"),
    ]
)]
```

---

## 8. Frontend Patterns (PWA)

### Pages (Next.js Pages Router)

The project uses the **Next.js Pages Router** (not the App Router):

```
pwa/pages/
├── index.tsx        ← route: /
└── admin/
    └── index.tsx    ← route: /admin
```

### Admin UI

The admin at `/admin` is fully **auto-generated** — no custom UI code needed for basic CRUD. It reads the Hydra API description and builds forms, lists, and show views automatically.

```tsx
// pwa/components/admin/App.tsx
import { HydraAdmin } from "@api-platform/admin";

const App = () => (
  <HydraAdmin
    entrypoint={window.origin}   // points to the API
    title="API Platform admin"
  />
);
```

The admin page loads this component **client-side only** (via `dynamic` import with `ssr: false`) because it relies on browser APIs.

### Layout & Data Fetching

The shared `Layout` component wraps the app with **TanStack React Query** for server-state management:

```tsx
// pwa/components/common/Layout.tsx
const Layout = ({ children, dehydratedState }) => {
  const [queryClient] = useState(() => new QueryClient());
  return (
    <QueryClientProvider client={queryClient}>
      <HydrationBoundary state={dehydratedState}>
        {children}
      </HydrationBoundary>
    </QueryClientProvider>
  );
};
```

Use `useQuery` / `useMutation` for data fetching in components, and `dehydrate` in `getServerSideProps` for SSR prefetching.

### Styling

Tailwind CSS v4 with `@tailwindcss/forms`. Use utility classes directly in JSX:

```tsx
<div className="flex flex-col items-center text-cyan-500 hover:text-cyan-700">
```

There is no global CSS theme file — all styling is inline Tailwind utilities.

### TypeScript

All PWA code is TypeScript. Use the strict types from `@types/react`, `@types/node`, etc.

---

## 9. Database & Migrations

The project uses **Doctrine ORM** with **PostgreSQL**.

### Create a migration after changing an entity

```bash
docker compose exec php bin/console doctrine:migrations:diff
# Review the generated file in api/migrations/
docker compose exec php bin/console doctrine:migrations:migrate
```

### Direct database access (dev)

```bash
psql -h localhost -p 5432 -U app -d app
# Password: !ChangeMe!  (see compose.yaml defaults)
```

---

## 10. Real-time with Mercure

Mercure is built into the FrankenPHP/Caddy web server. To enable real-time push for a resource, add the `mercure: true` option:

```php
#[ApiResource(mercure: true)]
#[ORM\Entity]
class Notification { ... }
```

API Platform will automatically publish updates to the Mercure hub when entities are created/updated/deleted. Clients subscribe via Server-Sent Events (SSE).

The Mercure debugger at `https://localhost/.well-known/mercure/ui/` lets you manually publish and subscribe to topics.

---

## 11. Testing

### E2E tests (Playwright)

Tests live in `e2e/tests/`. They run against `https://localhost` so the full stack must be up.

```bash
# From the e2e directory (inside Docker or with Node installed locally)
npx playwright test

# Or via Docker (check CI config for exact command)
```

Current test coverage:
- Homepage title
- Swagger UI renders with correct operation count
- Admin CRUD flow (create, list, show, edit a Greeting)

### PHP unit/integration tests

```bash
docker compose exec php bin/phpunit
```

Tests are in `api/tests/`. Configuration is in `api/phpunit.xml.dist`.

---

## 12. Production Deployment

### Docker Compose

```bash
# Use the prod override — builds production-optimised images
docker compose -f compose.yaml -f compose.prod.yaml up --build -d
```

Required environment variables for production (set in `.env` or CI secrets):

| Variable | Purpose |
|---|---|
| `APP_SECRET` | Symfony app secret (random 32-char string) |
| `POSTGRES_PASSWORD` | Database password |
| `CADDY_MERCURE_JWT_SECRET` | JWT secret for Mercure publisher/subscriber keys |

### Kubernetes (Helm)

A Helm chart is available in the `helm/` directory for Kubernetes deployments.

---

## 13. Token Evaluation

Actual token counts per project area (measured from source files; ~4 chars = 1 token for code).

| Area | Key files | Tokens |
|---|---|---|
| API entities | `api/src/Entity/Greeting.php` | ~150 |
| API kernel & bootstrap | `api/src/Kernel.php`, `config/bootstrap.php`, `config/bundles.php` | ~580 |
| API config packages | `api/config/packages/*.yaml` (15 files) | ~2,330 |
| API routes & services | `api/config/routes.yaml`, `services.yaml`, `routes/*` | ~410 |
| DB migrations | `api/migrations/Version*.php` (1 file) | ~250 |
| Dockerfiles | `api/Dockerfile`, `pwa/Dockerfile` | ~1,070 |
| Docker Compose files | `compose.yaml`, `compose.override.yaml`, `compose.prod.yaml` | ~1,190 |
| Dependency manifests | `composer.json`, `pwa/package.json` | ~1,030 |
| PWA pages | `pwa/pages/index.tsx`, `admin/index.tsx`, `_app.tsx` | ~2,620 |
| PWA components | `pwa/components/admin/App.tsx`, `common/Layout.tsx` | ~180 |
| E2E tests | `e2e/tests/example.spec.js` | ~280 |
| **Total codebase** | | **~9,890 tokens** |


---

*For questions, check the [API Platform community](https://api-platform.com/community) or the official docs at api-platform.com.*
