# Enfyra Installation Guide

## Prerequisites

- **Node.js** >= 20.0.0
- **Package manager** (npm ≥8.0.0, yarn ≥1.22.0, or bun ≥1.0.0)
- **Database server** (MySQL, MariaDB, or PostgreSQL)
- **Redis server**

---

## Quick Setup

### 1. Install and run the backend

```bash
npx @enfyra/create-enfyra-be@latest <project-name>
cd <project-name>
npm run start
```
By default, the backend will run at http://localhost:1105

- For detailed instructions: [@enfyra/create-enfyra-be](https://www.npmjs.com/package/@enfyra/create-enfyra-be)

### 2. Install and run the app
```bash
npx @enfyra/create-enfyra-app@latest <project-name>
cd <project-name>
npm run dev
```
- For detailed instructions: [@enfyra/create-enfyra-app](https://www.npmjs.com/package/@enfyra/create-enfyra-app)

## Configuration Prompts

## 3. Backend Configuration prompts

When you run:

```bash
npx @enfyra/create-enfyra-be@latest <project-name>
```

the CLI will ask you a series of configuration questions. Enter the values that match your environment.

| Prompt                                | Description                                                         |
| ------------------------------------- | ------------------------------------------------------------------- |
| **Package manager**                   | Select the package manager you want to use (`npm`, `yarn`, `bun`)   |
| **Project name**                      | Name of the backend project (if not passed as a CLI argument)       |
| **Database type**                     | Type of database (`MySQL`, `PostgreSQL`, `MariaDB`)                 |
| **Database host**                     | Hostname or IP address of your database                             |
| **Database port**                     | Port number of your database (e.g. `3306` for MySQL)                |
| **Database username**                 | Database user account                                               |
| **Database password**                 | Database password (can be left empty)                               |
| **Database name**                     | Name of the database to connect to / create                         |
| **Configure database pool settings?** | Whether to configure advanced connection-pool settings (Yes/No)     |
| **Redis URI**                         | URI of your Redis instance (e.g. `redis://user:pass@host:port`)     |
| **Application port**                  | Port where the Enfyra backend will run (default `1105`)             |
| **Admin email**                       | Initial admin email used to log into the dashboard                  |
| **Admin password**                    | Initial admin password                                              |

> 💡 If any of the database or Redis connection details are invalid, the CLI will prompt you to re-enter them or cancel setup.

**The backend will run at http://localhost:1105 by default.**

After you finish answering the prompts, the CLI will:

* Validate database and Redis connections.
* Generate an `.env` file with your answers.
* Ask for confirmation before scaffolding the project.

Then run your new backend:

```bash
cd <project-name>
npm run start
```
(or use `yarn start:dev` / `bun run start:dev` depending on the package manager you selected.)

## 4. Frontend Configuration Prompts

When you run:

```bash
npx @enfyra/create-enfyra-app@latest <project-name>
```

the CLI will ask you a series of configuration questions for the frontend application. Enter the values that match your environment.

| Prompt                                | Description                                                         |
| ------------------------------------- | ------------------------------------------------------------------- |
| **Package manager**                   | Select the package manager you want to use (`npm`, `yarn`, `pnpm`, `bun`) |
| **API base URL**                      | Base URL of your backend API (must include `http://` or `https://`) |
| **App port**                          | Port where the Enfyra frontend will run (default `3000`)           |

> 💡 The API base URL must be a valid HTTP/HTTPS URL pointing to your Enfyra backend instance.

After you finish answering the prompts, the CLI will:

* Validate the API URL format.
* Generate an `.env` file with your configuration.
* Scaffold the frontend project with your selected package manager.

Then run your new frontend:

```bash
cd <project-name>
npm run dev
```

(or use `yarn dev` / `pnpm dev` / `bun dev` depending on the package manager you selected.)