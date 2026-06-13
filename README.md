# Sivafood - Monorepo Food Delivery Platform

A production-ready, scalable microservice-based food delivery platform built in TypeScript with Node.js/Express, React, PostgreSQL, Redis, Socket.io, and containerized with Docker.

---

## Technical Stack & Architecture

- **Clean Architecture Pattern**: Boundary separation (Domain Entities, Application Use-cases, Infrastructure Controllers) for maximum modularity and unit testability.
- **Frontend App**: Single unified React/Vite dashboard hosting 4 specific portals:
  1. **Customer Web Portal**: Discover restaurants, manage carts, checkout with simulated Razorpay, and track delivery routes on a Leaflet.js live map.
  2. **Restaurant Owner Dashboard**: Review and accept incoming orders, update preparation states, edit menus.
  3. **Delivery Partner App**: Manage assignments, view pickup/drop directions, simulate and stream real-time coordinate GPS telemetry.
  4. **Master Admin Console**: View system statistics (revenue, users, stores) and toggle restaurant listing authorizations.
- **API Gateway**: Express-based reverse proxy routing requests to respective services.
- **Storage Strategy**: PostgreSQL (via Prisma ORM) for core relational data; Redis for in-memory active customer shopping carts and geospatial tracking indices.
- **WebSockets**: Real-time GPS coordinate broadcasts via Socket.io.

---

## Project Structure Map

```
├── docker-compose.yml
├── package.json
├── tsconfig.json
├── README.md
├── backend/
│   ├── gateway/                 # Express reverse-proxy (Port 8000)
│   ├── shared/                  # Common library: Prisma schema, auth, errors, DB singletons
│   └── services/
│       ├── auth-service/        # JWT Auth, profiles & addresses (Port 8001)
│       ├── restaurant-service/  # Restaurant catalog, menu items, search (Port 8002)
│       ├── order-service/       # Redis-cached carts, checkout pipelines (Port 8003)
│       ├── payment-service/     # Razorpay integrations, callback signatures (Port 8004)
│       └── delivery-service/    # Matching engine, real-time driver socket telemetry (Port 8005)
└── frontend/
    ├── src/                     # React/TypeScript components
    ├── index.html               # Leaflet map imports
    └── vite.config.ts           # Port 3000 mapping with gateway proxy rules
```

---

## Quick Start Guide

### Option 1: Docker Compose (Recommended - Complete container runs)

Build and run all services, databases, and frontends containerized:

```bash
# Spin up PostgreSQL, Redis, API Gateway, 5 Microservices, and the React Frontend
docker-compose up --build
```
- Frontend web client: `http://localhost:3000`
- API Gateway endpoints: `http://localhost:8000/api/...`

### Option 2: Local Host Development (Fast reloads)

#### 1. Start Infrastructure Databases (Postgres & Redis)
Ensure you have Docker installed, then spin up the infrastructure container nodes:
```bash
docker compose up postgres redis -d
```

#### 2. Install Dependencies
Run from the monorepo root workspace folder:
```bash
npm install
```

#### 3. Setup Environment variables & Database Migrations
Generate Prisma clients and push schema tables:
```bash
# Set connection environment link
# (For Windows Powershell, use: $env:DATABASE_URL="postgresql://postgres:password@localhost:5432/food_delivery?schema=public")
export DATABASE_URL="postgresql://postgres:password@localhost:5432/food_delivery?schema=public"

# Run Prisma schema migration
npm run db:migrate -w backend/shared

# Seed database with sample customers, restaurant owners, drivers, and items
npm run db:generate -w backend/shared
npx ts-node backend/shared/prisma/seed.ts
```

#### 4. Build Shared Package
```bash
npm run build:shared
```

#### 5. Launch All Dev Servers
Start all microservices and frontend clients concurrently:
```bash
npm run start:all
```
- Frontend client: `http://localhost:3000`

---

## Dummy Login Credentials
All password hashes are seeded as **`password`** for simplicity. Use these logins in the visual portals:
- **Customer**: `customer@food.com`
- **Restaurant Owner**: `owner@food.com`
- **Delivery Rider**: `driver@food.com`
- **System Admin**: `admin@food.com`

---

## Sandbox Workflow Walkthrough Guide

To verify the integration workflow end-to-end:
1. Open `http://localhost:3000` and sign in using **`customer@food.com` / `password`**.
2. Select a restaurant (e.g. **Spicy Fusion Bistro**), add items (Paneer Butter Masala) to the cart, and proceed to the cart tab.
3. Select the default address (Home) and click **Place Order & Pay**.
4. A Razorpay simulator modal will pop up. Click **SIMULATE SUCCESSFUL PAYMENT**.
5. Once confirmed, the order moves to **CONFIRMED** state. A background matching engine script assigns the rider **Express Rider** automatically.
6. Look at the bottom-right of your screen. Use the **SANDBOX SELECTOR** dock to override active views:
   - Click **Restaurant**: View the order request under incoming boards. Click **Accept & Confirm** -> **Start Preparing** -> **Mark Ready for Pick**.
   - Click **Driver**: View the assigned delivery offer under Partner Portal. Click **Accept Run Offer** -> **Arrived at Restaurant** -> **Pick up Order**.
   - Click **Start Ride Simulation**: The driver app streams GPS coordinates over WebSockets every 1.5 seconds.
   - Click **Customer**: Open **My Orders -> Track Order** to watch the driver icon glide along the map in real-time!
