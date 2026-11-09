# PrimeHR — Pagsanjan Human Resources Information System

PrimeHR is a Laravel-based Human Resources Information System for the Municipality of
Pagsanjan. It covers personnel records, attendance and DTRs, leave and benefits,
payroll, travel orders, pass slips, recruitment, reports, notifications, and an
AI assistant that answers HR questions from the system's own data.

The web tier is Laravel 13 with Blade, Vite, and Tailwind CSS 4; the database is
MySQL 8.0+.

The web application lives in [`primeHrMagdalenaLaravel/`](primeHrMagdalenaLaravel/).
The repository also contains a Flutter mobile client in
[`prime_magdalena_mobile_application/`](prime_magdalena_mobile_application/).

---

## Default administrator account

The database seeder creates this account. Use it to sign in after the setup steps
below.

| | |
|---|---|
| **Login page** | <http://127.0.0.1:8000/login> |
| **Email** | `admin@gmail.com` |
| **Password** | `asdf` |
| **Landing page** | `/admin/dashboard` |

> **Change this password before the system is used by anyone else.** It is a
> development credential created by
> [`AdminUserSeeder.php`](primeHrMagdalenaLaravel/database/seeders/AdminUserSeeder.php),
> not a production password.

To change it, sign in and open **Admin → Settings** (`/admin/settings`), then use
the password form. It asks for the current password and requires the new password
to be at least 8 characters.

The seeded account belongs to **Jade Flores** and holds both the `employee` and
`admin` roles. The login controller sends `admin@gmail.com` straight to the admin
dashboard; other accounts go to the role picker or their own dashboard.

---

## Quick start (TL;DR)

For an experienced developer on macOS/Linux. Windows commands and explanations
are in the numbered sections below.

```bash
git clone https://github.com/ViruSahastrabudhhe/primehris-pagsanjan.github.io.git PrimeHrProject-Pagsanjan
cd PrimeHrProject-Pagsanjan/primeHrMagdalenaLaravel

composer install
npm install

cp .env.example .env
php artisan key:generate
# edit .env: DB_CONNECTION=mysql, DB_DATABASE=primehrismagdalena,
# DB_USERNAME=root, DB_PASSWORD=

mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS primehrismagdalena CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
php artisan migrate --seed
php artisan storage:link

composer dev
```

Open <http://127.0.0.1:8000/login> and sign in with `admin@gmail.com` / `asdf`.

> `composer dev` runs the server, queue worker, log viewer, and Vite together.
> If you prefer separate terminals, see
> [Run the application](#10-run-the-application).

---

## Contents

1. [Requirements](#1-requirements)
2. [Clone the repository](#2-clone-the-repository)
3. [Install PHP dependencies](#3-install-php-dependencies)
4. [Install frontend dependencies](#4-install-frontend-dependencies)
5. [Configure the environment](#5-configure-the-environment)
6. [Create the MySQL database](#6-create-the-mysql-database)
7. [Create the tables and seed the admin account](#7-create-the-tables-and-seed-the-admin-account)
8. [Link storage](#8-link-storage)
9. [Build or watch the frontend](#9-build-or-watch-the-frontend)
10. [Run the application](#10-run-the-application)
11. [Common commands](#11-common-commands)
12. [Troubleshooting](#12-troubleshooting)
13. [Project layout](#13-project-layout)
14. [Optional integrations](#14-optional-integrations)
15. [Running the tests](#15-running-the-tests)
16. [Production checklist](#16-production-checklist)

---

## 1. Requirements

Install these before starting.

| Tool | Minimum | Tested with | Notes |
|---|---|---|---|
| **PHP** | 8.3 | 8.4.13 | The app requires `^8.3`. |
| **Composer** | 2.x | 2.9.2 | Installs the PHP packages. |
| **Node.js** | 20.19+ or 22.12+ | 24.11.0 | Required by Vite 8. |
| **npm** | 10+ | 11.6.1 | Ships with Node. |
| **MySQL** | 8.0 | 8.0.45 | **Required.** SQLite is not supported. |
| **Git** | any recent | — | Only needed to clone. |

### Required PHP extensions

Enable these in `php.ini` (or install the matching PHP packages):

- `pdo_mysql` — the database driver
- `mbstring`
- `openssl`
- `fileinfo`
- `zip` — Excel/Word export and import (`phpoffice/phpspreadsheet`, `phpoffice/phpword`)
- `gd` — image handling and PDF rendering
- `curl`
- `xml`, `dom`, `libxml`
- `ctype`, `tokenizer`, `json`
- `bcmath`

Check what is enabled:

```bash
php -m
```

If `pdo_mysql` is missing, Composer may install successfully but every database
command will fail with `could not find driver`.

### Why MySQL and not SQLite?

`.env.example` contains `DB_CONNECTION=sqlite`, but this project's migrations use
MySQL-only SQL:

- `ENUM` columns (for example the `roles` conversion and `status`)
- a `JSON` column on `users`
- `CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` in
  `2026_04_15_182306_add_timestamps_to_tables.php`

On SQLite, `php artisan migrate` stops partway through with an error such as
`near "ON": syntax error`. Use MySQL 8.0 or newer.

---

## 2. Clone the repository

```bash
git clone https://github.com/ViruSahastrabudhhe/primehris-pagsanjan.github.io.git PrimeHrProject-Pagsanjan
cd PrimeHrProject-Pagsanjan/primeHrMagdalenaLaravel
```

The target folder name is your choice. This guide uses
`PrimeHrProject-Pagsanjan`.

> **Private repository:** if GitHub asks for credentials, sign in first or ask the
> repository owner to grant your account access. If you received the project as a
> ZIP file, extract it and open a terminal in
> `PrimeHrProject-Pagsanjan/primeHrMagdalenaLaravel` instead.

> **Run the remaining commands from `primeHrMagdalenaLaravel/`.** The repository
> root has its own legacy `composer.json` and `package.json`; installing from
> there will not set up the application. You can confirm you are in the right
> place when the folder contains `artisan`, `app`, `database`, `public`,
> `resources`, and `routes`.

---

## 3. Install PHP dependencies

From `primeHrMagdalenaLaravel/`:

```bash
composer install
```

This reads `composer.lock`, so it installs the exact package versions the project
was built with. Do not run `composer update` for a normal setup.

If Composer reports that `bootstrap/cache` must be present and writable:

```bash
# macOS / Linux
chmod -R 775 storage bootstrap/cache

# Windows (PowerShell): make sure the folder exists and your user can write to it
New-Item -ItemType Directory -Force bootstrap\cache
```

Then run `composer install` again.

---

## 4. Install frontend dependencies

```bash
npm install
```

This installs Vite, Tailwind CSS, and the JavaScript used by the Blade pages. Use
Node 20.19+ or 22.12+; older Node versions fail on Vite 8.

---

## 5. Configure the environment

### 5.1 Create `.env`

```bash
cp .env.example .env
```

On Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

### 5.2 Generate the application key

```bash
php artisan key:generate
```

This writes `APP_KEY` into `.env`. Run it only after `.env` exists. If you see
`No application encryption key has been specified`, this step was skipped.

### 5.3 Point the application at MySQL

Open `.env` and make sure these values are set. The important part is replacing
the default `DB_CONNECTION=sqlite` block:

```dotenv
APP_NAME="Pagsanjan HRIS"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://127.0.0.1:8000
APP_TIMEZONE=Asia/Manila

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=primehrismagdalena
DB_USERNAME=root
DB_PASSWORD=
```

Notes:

- `DB_PASSWORD=` is correct for a local MySQL `root` account with no password
  (common with XAMPP and Laragon). If your MySQL root account has a password, put
  it after the `=`.
- If your MySQL listens on a port other than `3306` (for example because two
  MySQL installations are present), set `DB_PORT` to the correct port.
- Keep `SESSION_DRIVER=database`, `CACHE_STORE=database`, and
  `QUEUE_CONNECTION=database`. The migrations create the tables these need.
- `MAIL_MAILER=log` is the default. Password-reset and credential emails are
  written to `storage/logs/laravel.log` instead of being sent. Configure SMTP
  later if you need real email delivery.
- `APP_URL` should match the address you open in the browser. If you run
  `php artisan serve --port=8001`, change it to
  `http://127.0.0.1:8001`.

> **Do not leave `DB_CONNECTION=sqlite`.** The migration set will fail. This is
> the single most common setup problem in this project.

---

## 6. Create the MySQL database

Start MySQL first, then create an empty database.

### Using the MySQL command line

```bash
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS primehrismagdalena CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

If your root account has no password, omit `-p`:

```bash
mysql -u root -e "CREATE DATABASE IF NOT EXISTS primehrismagdalena CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

If `mysql` is not on your `PATH` (common with XAMPP on Windows), use
phpMyAdmin/MySQL Workbench or run the full path, for example
`C:\xampp\mysql\bin\mysql.exe`, with the same arguments.

### Using MySQL Workbench, phpMyAdmin, or another GUI

Run:

```sql
CREATE DATABASE IF NOT EXISTS primehrismagdalena
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

### Optional: use a dedicated database user

Using `root` is fine for local development. For a cleaner setup:

```sql
CREATE DATABASE IF NOT EXISTS primehrismagdalena
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

CREATE USER 'primehr'@'localhost' IDENTIFIED BY 'replace-with-a-strong-password';
GRANT ALL PRIVILEGES ON primehrismagdalena.* TO 'primehr'@'localhost';
FLUSH PRIVILEGES;
```

Then set `DB_USERNAME=primehr` and `DB_PASSWORD=replace-with-a-strong-password`
in `.env`.

The database name is not special. If you use a different name, update both the
`CREATE DATABASE` statement and `DB_DATABASE` in `.env`.

> **Do not import the SQL files in the repository-root `database/` folder for a
> fresh installation.** Those files and `setup_db.py` are a snapshot from the
> original developer's machine. They may not match the current schema. The
> supported installation path is `php artisan migrate --seed`, described next.

---

## 7. Create the tables and seed the admin account

From `primeHrMagdalenaLaravel/`:

```bash
php artisan migrate --seed
```

This does three things:

1. Runs every migration and creates the tables.
2. Seeds the reference data: departments, designations, leave types, deduction
   types, and leave accrual rates.
3. Runs `AdminUserSeeder` and creates the `admin@gmail.com` account.

**`--seed` is required for the admin account.** If you run only
`php artisan migrate`, the tables exist but there is no login account.

> The repository's `composer setup` script is not a substitute for this step: it
> migrates but does **not** seed. If you use it, make sure `.env` already points
> at MySQL and run `php artisan db:seed` afterwards.

If you already ran `php artisan migrate` without seeding, run:

```bash
php artisan db:seed
```

### Optional: load the demonstration dataset

The seeder can also create thirty demonstration employees, their attendance
records, leave applications, payroll rows, travel orders, pass slips, and demo
accounts. It is disabled by default so a real installation does not fill with
people who do not work there. It takes noticeably longer than the base seed
because it also writes the attendance register and the payroll rows derived from
it.

To enable it, set this in `.env`:

```dotenv
SEED_DEMO_DATA=true
```

Then, on a **fresh** database:

```bash
php artisan migrate:fresh --seed
```

> `migrate:fresh` **drops every table** in the configured database before
> rebuilding it. Never run it against a database that holds data you want to keep.

If the database is already set up and you only want to add the demonstration
roster:

```bash
php artisan db:seed
```

The demo seeder replaces its own roster (`EMP-<year>-9001` and up) and leaves the
admin account and other tables alone. The demo employee accounts share the
password `password`; they are throwaway test accounts.

### If the admin account is missing

On a database seeded with the shipped reference data but missing the admin row,
run the seeder directly:

```bash
php artisan db:seed --class=AdminUserSeeder
```

Only do this when `admin@gmail.com` does not already exist. Running it twice
fails on the unique email/employee number, because the seeder does not check for
an existing account.

### Confirm the seed worked

```bash
php artisan tinker
```

```php
App\Models\User::where('email', 'admin@gmail.com')->first(['email', 'roles', 'status', 'email_verified_at']);
```

You should see an active, verified user with the `admin` role. Type `exit` to
leave Tinker.

---

## 8. Link storage

```bash
php artisan storage:link
```

This creates `public/storage` as a link to `storage/app/public`. Employee photos,
uploaded documents, and generated files are served through it.

Without this step, uploaded images and files return 404s even though they exist
on disk.

**Windows:** creating a symlink may require Developer Mode or an elevated
terminal. If the command fails:

1. Open **Settings → Privacy & security → For developers** and enable
   **Developer Mode**.
2. Or run the terminal as Administrator.
3. Then run `php artisan storage:link` again.

On macOS/Linux, make sure `storage/` and `bootstrap/cache/` are writable by the
web-server user.

---

## 9. Build or watch the frontend

Choose one mode.

### Development mode (live reload)

```bash
npm run dev
```

Keep this terminal open while you work. Vite serves the CSS and JavaScript and
writes `public/hot`, which Laravel reads automatically.

### Production-style build (no Vite process needed)

```bash
npm run build
```

This compiles the assets into `public/build`. Run it again whenever the frontend
source changes.

If a page fails with `Vite manifest not found at .../public/build/manifest.json`,
you have not run `npm run build` or `npm run dev` yet.

> Never commit `public/build` or `public/hot`; both are generated and are already
> ignored by Git.

---

## 10. Run the application

### Option A — one command for development

```bash
composer dev
```

This starts, in one terminal:

- `php artisan serve` (the web server)
- `php artisan queue:listen` (queued jobs)
- `php artisan pail` (live log viewer)
- `npm run dev` (Vite)

It requires the development dependencies, so use `composer install` (not
`composer install --no-dev`). It does **not** run migrations or seeders, so
complete steps 6 and 7 first.

### Option B — separate terminals

Terminal 1:

```bash
php artisan serve
```

Terminal 2:

```bash
npm run dev
```

Optional terminal 3, if you want queued jobs processed:

```bash
php artisan queue:listen --tries=1
```

Then open:

<http://127.0.0.1:8000>

### Sign in

Open <http://127.0.0.1:8000/login> and use:

| Field | Value |
|---|---|
| Email | `admin@gmail.com` |
| Password | `asdf` |

The account is redirected to `/admin/dashboard`.

If you prefer the production-style path, run `npm run build` instead of
`npm run dev`, then `php artisan serve` and open the same URL. Only one of
`npm run dev` or `npm run build` is needed at a time.

### About the server and queue

- `php artisan serve` uses PHP's built-in server on `127.0.0.1:8000`. It is for
  development, not production.
- The database queue is used for background jobs. Notifications are stored as
  database rows and appear in the bell without a worker; a worker is needed for
  anything explicitly queued.
- `php artisan schedule:work` runs the scheduler in the foreground during
  development. In production, use cron instead (see the checklist below).

---

## 11. Common commands

Run these from `primeHrMagdalenaLaravel/`.

| Command | What it does |
|---|---|
| `php artisan serve` | Start the Laravel development server. |
| `npm run dev` | Start Vite with live reload. |
| `npm run build` | Compile the frontend into `public/build`. |
| `composer dev` | Run server, queue, logs, and Vite together. |
| `php artisan migrate` | Apply migrations that have not run yet. |
| `php artisan migrate --seed` | Apply migrations and seed reference data + admin. |
| `php artisan migrate:fresh --seed` | Drop all tables, rebuild, and seed. **Destructive.** |
| `php artisan db:seed` | Run the seeders without touching the schema. |
| `php artisan db:seed --class=AdminUserSeeder` | Create only the admin account. |
| `php artisan storage:link` | Recreate the `public/storage` link. |
| `php artisan optimize:clear` | Clear config, route, view, and application caches. |
| `php artisan config:clear` | Clear only the cached configuration. |
| `php artisan test` | Run the PHPUnit test suite. |
| `php artisan queue:listen --tries=1` | Process queued jobs (development). |
| `php artisan queue:work` | Process queued jobs (long-running worker). |
| `php artisan schedule:work` | Run the scheduler in the foreground. |
| `php artisan about` | Show the application environment summary. |

---

## 12. Troubleshooting

### `could not find driver`

`pdo_mysql` is not enabled for the PHP binary you are using.

- Check with `php -m`.
- Enable `extension=pdo_mysql` in `php.ini` and restart the terminal.
- On Windows with XAMPP, edit `php\php.ini`; on Laragon, use the PHP extension
  menu. The `php` on your `PATH` and the PHP used by the web server must be the
  same binary.

### Composer runs out of memory

```bash
COMPOSER_MEMORY_LIMIT=-1 composer install
```

On Windows PowerShell:

```powershell
$env:COMPOSER_MEMORY_LIMIT="-1"; composer install
```

### `SQLSTATE[HY000] [2002] Connection refused`

MySQL is not running, or the host/port is wrong.

1. Start MySQL (XAMPP/Laragon control panel, `services.msc`, or
   `brew services start mysql`).
2. Confirm `DB_HOST=127.0.0.1` and `DB_PORT=3306` in `.env`.
3. Test with `mysql -u root -p -e "SELECT 1;"`.

### `SQLSTATE[HY000] [1045] Access denied for user ...`

`DB_USERNAME` or `DB_PASSWORD` is wrong. Test the same credentials directly in
MySQL, then update `.env`.

### `Unknown database 'primehrismagdalena'`

The database was not created. Run the `CREATE DATABASE` statement in step 6,
then `php artisan migrate --seed`.

### `SQLSTATE[42S02] Base table or view not found`

The migrations have not run, or they ran against a different database. Confirm
`DB_DATABASE` in `.env`, then:

```bash
php artisan migrate --seed
```

### `near "ON": syntax error` during migration

You are running on SQLite. Change `.env` to MySQL:

```dotenv
DB_CONNECTION=mysql
```

Then drop and recreate the database if a partial migration run left it in a
mixed state, and run `php artisan migrate --seed` again.

### `Vite manifest not found at .../public/build/manifest.json`

The frontend has not been built. Run one of:

```bash
npm run dev
```

or:

```bash
npm run build
```

### `419 Page Expired` when submitting a form

The session table or session configuration is missing.

1. Confirm `SESSION_DRIVER=database` in `.env`.
2. Run `php artisan migrate` (the `sessions` table is part of the migrations).
3. Run `php artisan optimize:clear`.
4. Reload the page and try again.

### `The bootstrap/cache directory must be present and writable`

The web-server/PHP user cannot write to `bootstrap/cache`.

```bash
# macOS / Linux
chmod -R 775 bootstrap/cache storage
```

On Windows, check the folder's properties and give your user write access. Then:

```bash
php artisan optimize:clear
```

### `storage:link` fails on Windows

Enable **Developer Mode** (Settings → Privacy & security → For developers) or run
the terminal as Administrator, then retry:

```bash
php artisan storage:link
```

### `npm run build` fails with a Node version error

Vite 8 requires Node 20.19+ or 22.12+. Check `node -v` and upgrade Node if
needed. After upgrading, run `npm install` again if the error mentions missing
packages. You do not need to delete `node_modules` or `package-lock.json`.

### `Class "..." not found` after pulling new code

The autoloader is stale:

```bash
composer dump-autoload
php artisan optimize:clear
```

### `Invalid email or password` for `admin@gmail.com`

Check, in order:

1. Did you run `php artisan migrate --seed`, or `php artisan db:seed` after
   migrating? Migrations alone do not create users.
2. Does the user exist?

   ```bash
   php artisan tinker
   ```

   ```php
   App\Models\User::where('email', 'admin@gmail.com')->first();
   ```

3. Is `status` set to `Active`? A user with `status = Inactive` cannot sign in,
   even with the correct password.
4. Is `email_verified_at` set? The admin area is behind email verification. The
   seeder sets this automatically.
5. If the row is missing, run `php artisan db:seed --class=AdminUserSeeder` on a
   database where that account does not already exist.

### Port 8000 is already in use

Run the server on another port:

```bash
php artisan serve --port=8001
```

Then update `APP_URL` in `.env` to `http://127.0.0.1:8001` so generated links
match.

### Changes to `.env` do not take effect

Laravel may be using a cached configuration:

```bash
php artisan optimize:clear
```

Then restart `php artisan serve`.

---

## 13. Project layout

```text
PrimeHrProject-Pagsanjan/
├── primeHrMagdalenaLaravel/          # The Laravel web application
│   ├── app/                          # Controllers, models, services, middleware
│   ├── bootstrap/                    # Framework bootstrap and package cache
│   ├── config/                       # Application configuration
│   ├── database/
│   │   ├── migrations/               # Schema
│   │   ├── seeders/                  # Reference data, admin account, demo data
│   │   └── factories/
│   ├── public/                       # Web root (index.php, assets, uploads link)
│   ├── resources/
│   │   ├── css/                      # Tailwind and per-area styles
│   │   ├── js/                       # Per-area JavaScript
│   │   └── views/                    # Blade templates
│   ├── routes/                       # web.php and api.php
│   ├── storage/                      # Logs, uploads, framework cache
│   └── tests/                        # PHPUnit tests
├── prime_magdalena_mobile_application/  # Flutter mobile client (optional)
├── database/                         # Legacy SQL snapshot (not used for setup)
└── setup_db.py                       # Legacy MySQL import helper (not used for setup)
```

The application's web root is `primeHrMagdalenaLaravel/public`, not the repository
root.

---

## 14. Optional integrations

None of these are required for the core system to run. They are listed so you know
what the related screens and features expect.

### AI Assistant

The assistant uses a configurable LLM provider. The simplest local setup is a
Groq API key:

```dotenv
GROQ_API_KEY=your-groq-api-key
GROQ_MODEL=openai/gpt-oss-120b
```

Get a key from <https://console.groq.com/>. The assistant also supports
per-user and organisation-wide provider settings under **Settings → AI/Chatbot**
(`groq`, `openai`, `anthropic`).

If no provider is configured, the assistant still returns the underlying data
using a deterministic fallback narration; it just does not generate prose. No
AI key is needed to sign in or use the rest of the HRIS.

### Email

The default `MAIL_MAILER=log` writes messages to `storage/logs/laravel.log`.
Password-reset codes and account-credential emails therefore appear in the log
instead of an inbox. Set a real SMTP mailer in `.env` when you need delivery.

### Document indexing and OCR

Uploaded PDFs, DOCX files, and images become searchable after indexing:

```bash
php artisan ai:index-documents
```

To re-extract everything:

```bash
php artisan ai:index-documents --force
```

Scanned documents need Tesseract:

- macOS: `brew install tesseract poppler`
- Windows: install Tesseract and make sure `tesseract` is on `PATH`
- Linux: `sudo apt install tesseract-ocr poppler-utils`

Without Tesseract, scanned files are marked `ocr_required` rather than silently
indexed as empty.

### Recruitment backend

The Recruitment pages can call a separate FastAPI service. That service is
optional and runs as its own process. See
[`docs/hiring-backend-integration.md`](primeHrMagdalenaLaravel/docs/hiring-backend-integration.md)
for the configuration. Without it, the Recruitment pages show a clear
"not connected" message instead of breaking.

### Mobile application

The Flutter client in `prime_magdalena_mobile_application/` is a separate app. It
talks to the Laravel API under `/api`. Configure its API base URL in
`lib/config/api_config.dart` (`manualBaseUrl`), for example
`http://192.168.1.20:8000/api` when testing on a physical phone. See that
folder's README for Flutter-specific setup.

---

## 15. Running the tests

```bash
php artisan test
```

The test suite is configured to use an in-memory SQLite database
(`phpunit.xml`). Many tests build the tables they need in `setUp()` because the
application's full migration set is MySQL-only and `RefreshDatabase` cannot run
it on SQLite.

A fresh checkout is not expected to be completely green: some pre-existing
failures in `tests/Feature/ChatbotControllerTest.php` (four tests) and
`tests/Feature/ExampleTest.php` are unrelated to installation. For setup
verification, the meaningful checks are:

1. `php artisan about` runs without errors.
2. `php artisan migrate --seed` completes.
3. The login page loads.
4. `admin@gmail.com` / `asdf` reaches `/admin/dashboard`.

To run a focused subset:

```bash
php artisan test tests/Unit/ThemePrecedenceTest.php
php artisan test tests/Feature/WebsiteContentTest.php
```

---

## 16. Production checklist

The steps above are for local development. Before exposing the system:

- [ ] Change the `admin@gmail.com` password (or remove the seeded account and
      create a real administrator).
- [ ] Set `APP_ENV=production` and `APP_DEBUG=false`.
- [ ] Generate a fresh `APP_KEY` on the production server; never reuse a
      development key.
- [ ] Use a dedicated MySQL user with only the privileges it needs, not `root`.
- [ ] Run `composer install --no-dev --optimize-autoloader`.
- [ ] Run `npm ci && npm run build`.
- [ ] Run `php artisan migrate --force`.
- [ ] Run `php artisan storage:link`.
- [ ] Run `php artisan optimize`.
- [ ] Point the web server's document root at `primeHrMagdalenaLaravel/public`.
- [ ] Make `storage/` and `bootstrap/cache/` writable by the web-server user.
- [ ] Configure HTTPS and set `APP_URL` to the public URL.
- [ ] Run a supervised queue worker (`php artisan queue:work`) if queued jobs are
      used.
- [ ] Add the scheduler to cron:

  ```cron
  * * * * * cd /path/to/primeHrMagdalenaLaravel && php artisan schedule:run >> /dev/null 2>&1
  ```

- [ ] Never set `SEED_DEMO_DATA=true` on a production database.
- [ ] Configure real SMTP mail if password reset and credential emails must be
      delivered.
- [ ] Back up the database and the `storage/app/public` uploads directory.
