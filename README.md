# Vini Água e Gás

Complete business solution for **Vini Água e Gás**, a water and gas cylinder delivery business in Brazil. This repository is a monorepo containing the customer mobile app, the admin panel, the backend API, and shared code.

## What's inside

- **Customer mobile app** (Android, via Google Play) — browse products (water gallons, gas cylinders), place orders, select delivery address with Google Maps, track order status, and manage loyalty points.
- **Admin panel** (web) — manage products, inventory, suppliers, orders (confirm → deliver), payments, daily cash reconciliation, and the loyalty program.
- **Backend API** — the single source of truth for auth, orders, inventory, payments, and loyalty, shared by both clients.
- **WhatsApp Terminal** (local Windows app) — receives orders from any WhatsApp number paired via QR code and prints them on the thermal printer.
- **Shared package** — Zod schemas and TypeScript types reused across API, admin, and mobile.

See the full [Architecture Plan](docs/architecture.md) for tech stack decisions, database schema, API design, authentication flow, Google Maps integration, and Play Store deployment steps.

## Project structure

```
vini-agua-gas/
├── apps/
│   ├── mobile/     # Expo React Native app (customer-facing, Android/Play Store)
│   ├── admin/      # React + Vite admin panel
│   ├── api/        # Node.js + Express + Prisma backend
│   └── terminal/   # Node.js + Electron local app: WhatsApp Web + thermal printer
├── packages/
│   └── shared/     # Shared TypeScript types, Zod schemas, constants
└── docs/
    ├── architecture.md
    └── whatsapp-printer-integration.md
```

## Tech stack

| Layer | Technology |
|---|---|
| Mobile app | React Native + Expo |
| Admin panel | React + Vite |
| Backend | Node.js 20 + Express + Prisma |
| Database | PostgreSQL (SQLite fallback documented for local MVP) |
| Auth | JWT (access + refresh tokens) |
| Validation | Zod |
| Maps | Google Maps Platform (Places, Geocoding) |
| WhatsApp terminal | Node.js + Electron + `whatsapp-web.js` |
| Thermal printing | `node-thermal-printer` (ESC/POS) |

## Requirements

- [Node.js 20+](https://nodejs.org/)
- [npm 10+](https://www.npmjs.com/)
- PostgreSQL (or use SQLite for local MVP)

## Getting started

### 1. Install dependencies

From the repository root:

```bash
npm install
```

This installs all workspace dependencies using npm workspaces.

### 2. Configure environment variables

Copy the example file and fill in the real values:

```bash
cp apps/api/.env.example apps/api/.env
```

Required variables for the API (`apps/api/.env`):

```env
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://user:password@localhost:5432/vini_agua_gas
JWT_ACCESS_SECRET=substitua_por_um_segredo_de_32_caracteres_ou_mais
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=30d
CORS_ORIGIN=http://localhost:5173,http://localhost:8081
```

For admin and mobile, copy their `.env.example` files and adjust `API_BASE_URL` if needed.

### 3. Set up the database

#### Option A: PostgreSQL (recommended)

Start a local PostgreSQL instance. With Docker:

```bash
docker run --name vini-postgres \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=vini_agua_gas \
  -p 5432:5432 \
  -d postgres:16
```

#### Option B: SQLite (local MVP fallback)

If you prefer a zero-dependency local setup, change the datasource in `apps/api/prisma/schema.prisma`:

```prisma
datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}
```

Then set `DATABASE_URL=file:./dev.db` in `apps/api/.env`.

### 4. Run migrations and seed

```bash
cd apps/api
npx prisma migrate dev
npx prisma db seed
```

The seed creates the sample products and an admin user:

- Email: `admin@viniaguaegas.com.br`
- Phone: `5599999999999`
- Password: `admin123`

### 5. Run the apps

Run everything in parallel from the root:

```bash
npm run dev
```

Or run each app separately:

```bash
# API (http://localhost:3000)
npm run dev:api

# Admin panel (http://localhost:5173)
npm run dev:admin

# Mobile (Expo dev server)
npm run dev:mobile
```

## Useful scripts

| Command | Description |
|---|---|
| `npm install` | Install all workspace dependencies |
| `npm run dev` | Run API, admin, and mobile in parallel |
| `npm run dev:api` | Run only the API |
| `npm run dev:admin` | Run only the admin panel |
| `npm run dev:mobile` | Start the Expo dev server |
| `npm run build` | Build all workspaces |
| `npm run typecheck` | Run TypeScript checks on all workspaces |
| `npm run db:migrate` | Run Prisma migrations |
| `npm run db:seed` | Run the database seed |
| `npm run db:generate` | Generate Prisma Client |

## Health check

The API exposes a health check endpoint:

```bash
curl http://localhost:3000/health
```

## API authentication

The auth module is available under `/api/v1/auth`:

- `POST /api/v1/auth/register` — create a customer account
- `POST /api/v1/auth/login` — login with phone or email
- `POST /api/v1/auth/refresh` — exchange a refresh token
- `GET /api/v1/auth/me` — get current user (requires Bearer token)
- `POST /api/v1/auth/logout` — revoke current refresh token
- `POST /api/v1/auth/change-password` — change password

## Documentation

- [Architecture Plan](docs/architecture.md) — tech stack, database schema, API endpoints, project structure, authentication flow, Google Maps integration, Play Store deployment, and environment/DevOps setup.
- [WhatsApp + Printer Integration](docs/whatsapp-printer-integration.md) — how the local terminal receives WhatsApp orders and prints receipts.

## License

Proprietary — all rights reserved by Vini Água e Gás.
