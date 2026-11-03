# NSBM SportsHub · guideline-focused PHP/MySQL application

This repository is a self-contained edition of the existing NSBM SportsHub prototype for **Topic 3 — Sports Club Management System**. Its application uses **HTML, CSS, vanilla JavaScript, PHP and actual MySQL 8+**. It retains the existing navy/lime interface, light theme, student/admin portals, REST endpoints, native form fallback, club communities and authenticator 2FA.

The application lives directly at this repository root; there is no nested `backup/` directory to enter. It was exported from `Ckstr-arch/TestENV`'s `backup/` at source commit `b2df1bd73415aa4a62e13b3a2bf903baeddc09a9`, retaining the Codex-assisted prototype as its baseline. It does not load that repository's source, private settings, database or sessions. Exporting the package does not transfer authorship to individual students.

## Start here

Clone this standalone repository, then work from its root:

```bash
git clone https://github.com/Ckstr-arch/FinalSubBackup.git
cd FinalSubBackup
```

If already cloned, open that directory; do not look for a `backup/` subfolder. Choose **native setup** below or the [optional local Docker setup](#optional-docker-setup--isolated-local-development). Both run the same application.

### Native setup

1. Install **PHP 8.2+** with `pdo_mysql`, `mbstring`, `sodium` and Argon2id support, and a genuine **MySQL 8+** server. This edition intentionally rejects MariaDB. Many XAMPP installations bundle MariaDB under a “MySQL” label; use a separate actual MySQL server.
2. Follow [native Windows, macOS and Linux setup](docs/local-setup.md). Use a **fresh** `sportshub_backup` database, its own database user, and a new private application key. Existing prototype data is not imported automatically.
3. From this folder, run:

   ```bash
   php bin/setup.php
   # Edit the private .env with your own database settings first.
   php bin/migrate.php
   php bin/seed.php
   php bin/doctor.php
   php -S localhost:8091 -t public public/router.php
   ```

   Seeding is optional development sports data; it creates no shared passwords or user accounts. Run it only in development.
4. Use `http://localhost:8091` locally, matching `APP_URL` exactly. Health is `/api/v1/health`. Create students through registration; provision your own administrator through the private CLI procedure in the setup guide. No shared fixed demo password is shipped; optional random local accounts are described in the setup guide.

The document root is **`public/` inside this repository**, not the repository root. PHP's built-in server is only for local development. Do not run it as a production service.

## Optional Docker setup · isolated local development

Docker runs the same PHP application with **actual MySQL 8.4**. It is an optional local tool, not a change to the required HTML/CSS/JavaScript/PHP/MySQL stack. You do not need PHP or MySQL installed on the host for this route. Keep your other projects running; this guide uses its own Compose project, volumes and default web port **8095**.

### 1. Install Docker and check the port

- **macOS:** install [Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/) for your Intel or Apple Silicon Mac, then start it.
- **Windows:** install [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/), using its supported WSL2 backend and Linux containers.
- **Linux:** follow [Docker Engine installation](https://docs.docker.com/engine/install/) and [Compose plugin installation](https://docs.docker.com/compose/install/linux/). Use your installation's supported Docker access permissions.

Use a current Docker release with Compose v2 and BuildKit (enabled by default in current Docker Desktop). From the cloned `FinalSubBackup` root, verify:

```bash
docker version
docker compose version
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

If another project uses 8095, choose another free port and set **both** `APP_PORT` and `APP_URL` below. No other containers need to be stopped or removed.

### 2. Create four private local helper files

Create `var/docker/` in your editor or file manager. Terminal alternatives are `mkdir -p var/docker` on macOS/Linux or `New-Item -ItemType Directory -Force var/docker` in PowerShell. Save the following four files there. The repository already ignores `var/`; these local helpers and their secrets stay untracked and outside the public document root.

**`var/docker/Dockerfile`:**

```dockerfile
# syntax=docker/dockerfile:1
FROM php:8.4-apache-bookworm
RUN --mount=type=secret,id=proxy_ca,mode=0444 \
    set -eu; \
    if [ -f /run/secrets/proxy_ca ]; then \
      printf 'Acquire::https::CaInfo "/run/secrets/proxy_ca";\n' > /etc/apt/apt.conf.d/99-proxy-ca; \
    fi; \
    sed -i 's|http://deb.debian.org|https://deb.debian.org|g' /etc/apt/sources.list.d/debian.sources; \
    apt-get update; \
    apt-get install -y --no-install-recommends libonig-dev; \
    docker-php-ext-install -j2 pdo_mysql mbstring; \
    a2enmod rewrite; \
    rm -f /etc/apt/apt.conf.d/99-proxy-ca; \
    rm -rf /var/lib/apt/lists/*
WORKDIR /var/www/html
COPY app ./app
COPY bin ./bin
COPY database ./database
COPY public ./public
COPY tests ./tests
RUN find app bin database public tests -type d -exec chmod 755 {} \; \
    && find app bin database public tests -type f -exec chmod 644 {} \; \
    && mkdir -p var/sessions \
    && chown -R www-data:www-data var \
    && chmod 700 var/sessions \
    && printf '%s\n' '<VirtualHost *:80>' \
       'DocumentRoot /var/www/html/public' '<Directory /var/www/html/public>' \
       'Options -Indexes +FollowSymLinks' 'AllowOverride All' 'Require all granted' \
       '</Directory>' '</VirtualHost>' > /etc/apache2/sites-available/000-default.conf
EXPOSE 80
CMD ["sh", "-c", "php bin/migrate.php && exec apache2-foreground"]
```

Only application source directories are copied into the image: no root `.env`, Git history or local `var/` files. The adjacent build-context allowlist below also excludes those private inputs before they are transferred to the builder. Apache serves only `public/`. The optional `proxy_ca` build secret is for managed networks with a trusted organization CA; ordinary home networks do not supply it. Configure any required proxy/CA through your administrator's supported Docker build settings; keep TLS and package-signature verification enabled. The secret/configuration is removed after the verified package download step and is not copied into the image.

**`var/docker/Dockerfile.dockerignore`:**

```text
**
!app/
!app/**
!bin/
!bin/**
!database/
!database/**
!public/
!public/**
!tests/
!tests/**
```

This Dockerfile-specific ignore file permits only the five required source directories into the build context. Docker still reads the selected Dockerfile and its adjacent ignore file for the build, even though other `var/` files are excluded. Keep this file beside the local Dockerfile; `.gitignore` alone does not filter a Docker build context.

**`var/docker/compose.yaml`:**

```yaml
services:
  db:
    image: mysql:8.4
    environment:
      MYSQL_DATABASE: sportshub_backup
      MYSQL_USER: sportshub_backup
      MYSQL_PASSWORD: ${DB_PASSWORD:?Set a private DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD:?Set a separate private DB_ROOT_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD-SHELL", "MYSQL_PWD=\"$${MYSQL_PASSWORD}\" mysql --protocol=TCP -h127.0.0.1 -u\"$${MYSQL_USER}\" --database=\"$${MYSQL_DATABASE}\" --silent --skip-column-names -e 'SELECT 1'"]
      interval: 5s
      timeout: 5s
      retries: 30
    restart: unless-stopped
  app:
    build:
      context: ../..
      dockerfile: var/docker/Dockerfile
    ports:
      - "127.0.0.1:${APP_PORT:-8095}:80"
    environment:
      APP_ENV: development
      APP_URL: ${APP_URL:-http://localhost:8095}
      APP_KEY: ${APP_KEY:?Set a private generated APP_KEY}
      DB_HOST: db
      DB_PORT: "3306"
      DB_NAME: sportshub_backup
      DB_USER: sportshub_backup
      DB_PASSWORD: ${DB_PASSWORD:?Set a private DB_PASSWORD}
    volumes:
      - app_data:/var/www/html/var
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
volumes:
  db_data:
  app_data:
```

The database has no published host port, so it can coexist with another project's MySQL. The app connects to service name `db`, not the host's localhost. `../..` is relative to the Compose file and selects this repository as the build context. The `$$` sequences preserve variables for the health-check shell inside the container.

**`var/docker/.env`:**

```dotenv
DB_PASSWORD=REPLACE_WITH_YOUR_PRIVATE_DATABASE_PASSWORD
DB_ROOT_PASSWORD=REPLACE_WITH_A_DIFFERENT_PRIVATE_ROOT_PASSWORD
APP_KEY=PASTE_YOUR_GENERATED_KEY_HERE
APP_PORT=8095
APP_URL=http://localhost:8095
```

Replace all three placeholders before starting. `DB_PASSWORD` belongs to the dedicated application database account; `DB_ROOT_PASSWORD` belongs to this new container's database administrator. Neither is a website login or your computer password. Generate a private key locally with this command, then paste its output into `APP_KEY` without surrounding quotes:

```bash
docker run --rm php:8.4-cli php -r "echo base64_encode(random_bytes(32)), PHP_EOL;"
```

Keep this file private and retain its values after users enroll in 2FA. On macOS/Linux, restrict it with `chmod 600 var/docker/.env`; on Windows, keep it under your account with appropriate file permissions. It is separate from a native root `.env`, which this image does not copy or load. For simple Compose interpolation, use passwords without `$` or `#`, or follow Docker's env-file quoting rules carefully; never paste actual values into reports, issues or screenshots.

### 3. Build, start and verify

Run these commands from the repository root. Keep the **same project name, environment file and Compose file** for every operation:

```bash
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml config --quiet
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml up --build -d
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml ps
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml exec app php bin/doctor.php
```

`config --quiet` validates without printing resolved secrets. First build/start can take several minutes. MySQL must pass its SQL health check before the app migrates the fresh database and starts Apache. Then use `http://localhost:8095` and `/api/v1/health`, or your chosen matching port/origin. To diagnose startup, inspect only this project's logs:

```bash
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml logs --tail=80 app db
```

For optional development sports records and random local logins:

```bash
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml exec app php bin/seed.php
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml exec app php bin/demo-users.php --confirm-demo
```

The seeder prints only the private path. On any platform, copy the generated record to the ignored local helper directory and open it **privately in your editor**:

```bash
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml cp app:/var/www/html/var/demo-credentials.json var/docker/demo-credentials.json
```

Docker Desktop users can also inspect the app container's Files view at `/var/www/html/var/demo-credentials.json`. The persistent original lives in `app_data`; the command above makes a private local copy. Never share either file or include it in screenshots. Existing accounts/credential records are preserved. Alternatively, register students normally and provision your own admin through the [private CLI instructions](docs/local-setup.md), executing that provisioning inside the app container; do not expect native host environment variables to appear in it automatically.

### 4. Stop, restart and update without losing data

```bash
# Stop only this project; retain containers and persistent data.
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml stop
# Resume its stopped containers.
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml start
# Restart its running services.
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml restart
# After reviewed source changes, rebuild and apply additive migrations.
docker compose -p finalsubbackup-local --env-file var/docker/.env -f var/docker/compose.yaml up --build -d
```

`db_data` retains users/bookings/community rows; `app_data` retains sessions and your private generated demo credential record. The encryption key remains in your private helper `.env`, so protect/back it up alongside the database. A rebuild replaces source in the image and retains these volumes; source is not bind-mounted. Changing database passwords in the env file does **not** rotate an already initialized MySQL user's password. Changing `APP_KEY` makes existing encrypted authenticator secrets unreadable. Preserve the project name to keep selecting the same volumes.

Do not delete volumes or run a global Docker prune to update this application. These local commands do not target your other projects. Use separate browser profiles if native and Docker copies run on localhost simultaneously: ports alone do not isolate cookies. This is a local development setup; follow [security and hosting](docs/security-and-hosting.md) for a real HTTPS submission host.

The recipe was verified with Docker Engine on Linux: fresh MySQL startup, PHP readiness/security checks, Apache/API access, and retained data after stop/start/restart. Docker Desktop installation was not executed on macOS or Windows in that verification.

## Team build order and contribution evidence

For the eight-student team, start with the [short A–H start/finish guide](docs/start-finish-guide.md); H (API/authentication/security) is the requester’s chosen part. Use the [two-person-per-stage plan](docs/eight-member-plan.md) for the full responsibility split. Follow the [step-by-step guide for all four stages](docs/build-order-and-contributions.md): HTML structure → CSS → UI JavaScript → PHP templates/data → protected native forms → REST enhancement. Frontend and backend tracks work in parallel around one complete membership journey. Record each person’s genuine work in the [contribution log template](docs/contribution-log.md), keeping the existing assisted application credited as baseline.

## Included capabilities

- Student registration/login, server-enforced student/admin access and optional TOTP 2FA with single-use recovery codes.
- Club discovery/search/filter, membership requests, approvals and approved-member club spaces.
- Training/match schedules, event participation, tournament registration, scores and generic club standings.
- Facility/equipment booking requests, interval collision protection, review and eligible own cancellations.
- Admin CRUD for clubs, tournaments, events and resources; attendance, participation reports and audit history.
- Community posts, comments, reactions, private reports and administrator hide/restore moderation.
- Responsive desktop/mobile navigation, keyboard controls and dark/light themes.

## Documentation and evidence

| Guide | Purpose |
|---|---|
| [Guidelines audit](docs/guidelines-audit.md) | What the PDF requires, what changed and what remains a human deliverable |
| [Requirement traceability](docs/requirements-traceability.md) | Actual routes, source, data and evidence for Topic 3 |
| [Native setup](docs/local-setup.md) | Precise configuration, OS installation, private administrator and test setup |
| [Architecture](docs/architecture.md) | One application, request flows, real data relationships and preservation |
| [REST API](docs/api.md) | Methods, cookie/CSRF contract, payloads and response errors |
| [Security and hosting](docs/security-and-hosting.md) | Key handling, production prerequisites, backups, release and honest limits |
| [Report and video](docs/report-and-video.md) | Actual people/contributions, screenshots, rubric and final delivery checklist |
| [Validation](docs/validation.md) | Current commands/outcomes and checks still awaiting execution |

## Scope and contribution integrity

Docker is optional local tooling, with user-created helpers under ignored `var/docker/`; no Dockerfile or Compose manifest is tracked in this native package. The application has no Docker, Python, Node, Composer or framework runtime dependency. Tests use PHP. REST is an HTTP interface implemented in PHP, not an extra application language or framework. PHP's PDO/mbstring/sodium extensions support the required stack.

The PDF does not say that using an external development tool alone invalidates an application. This package keeps the application distribution native while documenting optional local tooling; the meaningful database correction is using actual MySQL instead of relying on MariaDB compatibility.

The existing application was Codex-assisted. Record each person's real implementation, review, debugging, design, testing and documentation, with honest assistance disclosure under course policy. Do not assign pre-existing module authorship or manufacture contribution commits. Real live hosting, lecturer access, final report/video and confirmed deadline remain human tasks. Current validation is documented separately; there is no claim of a live deployment.
