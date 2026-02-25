# Teable — Assignment Submission

## What is this project?

[Teable](https://teable.io) is an open-source, no-code database platform built on PostgreSQL. It gives users a spreadsheet-like interface to manage data, collaborate in real time, and build powerful database applications — all without writing SQL.

This repository contains **my assignment work** on top of the Teable 1.8.0 codebase.

---

## What I Did — Skip Email Verification on Email Change

### The Problem

By default, when a user wants to change their email address in Teable, the application sends a One-Time Password (OTP) to the new email address and requires the user to verify it before the change is saved. While this is a good security practice in production, the assignment required me to **bypass this verification step** so that the email change happens immediately after the user confirms their current password.

### My Approach

I traced the full flow — from the frontend dialog box all the way through the API layer and down into the backend service — and made targeted changes at each layer to remove the OTP step while keeping the password confirmation intact.

---

## Files I Modified

### 1. `packages/openapi/src/auth/change-email.ts`
This file defines the API contract (request body shape + route spec). I updated the `IChangeEmailRo` type to only require `newEmail` and `password` — removing any OTP/token fields — so the frontend and backend both agree on the simplified payload.

### 2. `apps/nestjs-backend/src/features/auth/local-auth/local-auth.service.ts`
This is the core business logic. I added a `changeEmail()` method that:
- Validates the user's **current password** before allowing the change
- Checks that the **new email isn't already registered**
- Directly updates the email in the database (no OTP generation or verification)
- Clears the user's active session after the change (for security)

### 3. `apps/nestjs-backend/src/features/auth/local-auth/local-auth.controller.ts`
This exposes the HTTP endpoint. I added a `PATCH /auth/change-email` route that calls the new `changeEmail()` service method and returns a success response.

### 4. `apps/nextjs-app/src/features/app/components/setting/account/ChangeEmailDialog.tsx`
This is the frontend dialog in the account settings. I updated it to:
- Remove the two-step OTP verification UI
- Call the new simplified endpoint directly with just the new email + password
- Show the updated email immediately on success without any extra steps

---

## How to Run This Locally

### Prerequisites
- Docker & Docker Compose installed

### Steps

1. **Clone the repo and navigate into it**
   ```sh
   git clone <your-repo-url>
   cd teable
   ```

2. **Set up your environment variables**
   ```sh
   cp .env.example .env
   # Open .env and fill in your database/redis credentials
   ```

3. **Start everything with Docker**
   ```sh
   cd dockers/examples/standalone/
   docker-compose up -d
   ```

4. **Open the app** at [http://localhost:3000](http://localhost:3000)

5. **Test the feature** — go to Account Settings → Email → Change Email. Enter your new email and current password. The change should apply instantly without any OTP step.

---

## Why No Verification?

The original Teable codebase has the verification flow gated behind a feature flag (`enableEmailVerification` in settings). My implementation essentially hard-bypasses that flow entirely, which is what the assignment required. In a real production scenario, you'd want to keep verification enabled for security.

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `PRISMA_DATABASE_URL` | PostgreSQL connection string |
| `BACKEND_CACHE_PROVIDER` | Cache provider (use `redis`) |
| `BACKEND_CACHE_REDIS_URI` | Redis connection string |
| `SECRET_KEY` | JWT signing secret — keep this private! |
| `PUBLIC_ORIGIN` | The base URL of the app (e.g. `http://localhost:3000`) |

See `.env.example` for a template.
