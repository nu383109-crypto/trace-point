# TracePoint

TracePoint is a web application for tracking kiosks and their components. It provides a browser-based interface and a REST API for managing asset records, movement history, faults, users, and audit activity.

## Features

- Register and search components, and view each component's event history.
- Track kiosks, their locations, and the components installed in them.
- Record component events and report or resolve faults.
- View dashboard statistics, reports, and audit logs.
- Authenticate users with role-based permissions and JWT sessions.
- Serve the frontend and API from the same Express application.

## Technology

- Node.js and Express
- PostgreSQL
- Vanilla HTML, CSS, and JavaScript

## Requirements

- Node.js and npm
- PostgreSQL database with a connection URL
- PostgreSQL's `psql` command-line client to apply the schema

The database connection code currently rejects localhost database hosts. Use a reachable non-local PostgreSQL instance, such as a hosted PostgreSQL or Neon database.

## Getting started

1. Install the Node dependencies:

   ```sh
   npm install
   ```

2. Copy `.env.example` to `.env` and configure the required environment variables:

   ```env
   PORT=5500
   DATABASE_URL=postgresql://USER:PASSWORD@HOST/DATABASE?sslmode=require
   JWT_SECRET=replace-with-a-long-random-secret
   APP_BASE_URL=http://localhost:5500
   ```

   `DATABASE_URL_POOLED` can be set instead of `DATABASE_URL`; when both are set, the application uses `DATABASE_URL_POOLED`. Use the connection string and SSL options provided by your database host. Keep `.env` private and never commit credentials or production secrets.

   On Windows PowerShell, copy the example file with:

   ```powershell
   Copy-Item .env.example .env
   ```

   On macOS or Linux:

   ```sh
   cp .env.example .env
   ```

3. Apply the database schema. Replace the example URL with the connection URL from your database provider:

   ```sh
   psql "postgresql://USER:PASSWORD@HOST/DATABASE?sslmode=require" -f src/db/schema.sql
   ```

   The same command works in Windows PowerShell:

   ```powershell
   psql "postgresql://USER:PASSWORD@HOST/DATABASE?sslmode=require" -f src/db/schema.sql
   ```

   Replace the URL placeholders before running the command. `psql` does not load `.env` automatically. The schema creates the application's tables and enables the `pg_trgm` extension used for search. The database role must have permission to create that extension.

4. Start the application:

   ```sh
   npm run dev
   ```

   For a regular start without automatic restarts:

   ```sh
   npm start
   ```

5. Open `http://localhost:5500` (or the port configured in `.env`). The root page redirects to the login page. The health endpoint is available at `http://localhost:5500/health`.

## Configuration

| Variable | Required | Purpose |
| --- | --- | --- |
| `DATABASE_URL` or `DATABASE_URL_POOLED` | Yes | PostgreSQL connection string. The pooled value takes precedence when both are set. |
| `JWT_SECRET` | Yes for authentication | Secret used to sign JWTs. Set a long, unpredictable value. |
| `PORT` | No | HTTP port. Defaults to `3000` in the server; `.env.example` uses `5500`. |
| `APP_BASE_URL` | For password reset email | Public base URL used to build password-reset links. |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_SECURE`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM` | For password reset email | SMTP server settings. Password recovery returns an unavailable response until valid mail settings are configured. |

## First account and access roles

The first account registered in an empty database is assigned the **IT/Systems Administrator** role and must change its password. Further self-registrations receive the **Field Technician** role by default. The application also supports **Maintenance Supervisor** accounts.

Registration is available through the API. Before exposing the application publicly, review whether unrestricted registration is appropriate for your deployment.

## API overview

The API is mounted under `/api`. Most endpoints require a bearer token obtained by signing in through `/api/auth/login`.

| Prefix | Purpose |
| --- | --- |
| `/api/auth` | Registration, login, logout, password change, and password recovery |
| `/api/components` | Component records and component event history |
| `/api/kiosks` | Kiosk records and details |
| `/api/faults` | Fault reporting and resolution |
| `/api/events` | Component event history |
| `/api/dashboard` | Dashboard statistics |
| `/api/audit-logs` | Audit log records |
| `/api/users` | User listing, role changes, and account status |

For example, sign in and use the returned token in subsequent requests:

```sh
curl -X POST http://localhost:5500/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com","password":"your-password"}'
```

```sh
curl http://localhost:5500/api/components \
  -H "Authorization: Bearer YOUR_TOKEN"
```

List endpoints that support pagination accept `page` and `limit` query parameters. The default page size is 20 and the maximum is 100.

## Project structure

```text
.
├── public/             # HTML pages, shared stylesheet, and browser JavaScript
├── src/
│   ├── controllers/    # Request handlers and application operations
│   ├── db/             # PostgreSQL connection, schema, and setup script
│   ├── middleware/     # Authentication and authorization
│   ├── routes/          # Express API route definitions
│   └── utils/           # Shared helpers
├── server.js            # Starts the HTTP server
└── package.json         # Dependencies and npm scripts
```

## Available npm scripts

| Script | Description |
| --- | --- |
| `npm start` | Start the server |
| `npm run dev` | Start the server with Nodemon |
| `npm run migrate` | Run the schema file with `psql -f`; requires PostgreSQL connection environment variables in the shell |
| `npm run setup` | Alias for `npm run migrate` |
| `npm run seed` | Prints account setup guidance; it does not insert sample records |

## Development notes

- The frontend is served as static files from `public/`.
- The `pg_trgm` extension is required by the schema's text-search indexes.
- There is currently no test or lint command defined in `package.json`.
