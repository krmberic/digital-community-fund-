# Digital Community Fund Contribution and Member Benefit Management System (PHP)

**Case Study: Kimironko Cooperatives**

This is a **PHP port** of the original Python/Flask version of this project.
It implements the same features with the same business rules, using plain
**PHP 8 (custom lightweight MVC, no framework)** and **PDO**, so it can run
against either **SQLite** (zero-config, default) or **MySQL**.

> **No Composer packages are required to run this app.** It uses its own
> tiny autoloader (see `app/bootstrap.php`), so `composer install` is
> optional. `composer.json` is included mainly to document requirements.

---

## ⚠️ Important — this code has not been executed

This project was generated in an environment with **no PHP interpreter and
no internet access**, so unlike a typical delivery I could not actually run
or test it end-to-end. I wrote it carefully, mirroring the already-tested
Flask version route-for-route and rule-for-rule, and self-reviewed the code
for correctness — but **please run it locally and go through the flows
below before you rely on it**, and open each file if something doesn't
behave as expected.

---

## Features (same as the Flask version)

- Session-based login for **admin** and **treasurer** roles (hashed passwords)
- Member management — register, search, view, edit
- Contribution recording — cash, mobile money, bank transfer
- Benefit requests (loan / emergency grant / payout) with:
  - Loan eligibility rule: member needs ≥ 20,000 RWF in total contributions
  - Loan cap: cannot exceed 3× the member's total contributions
  - Admin-only approval workflow: pending → approved → paid / rejected
- Dashboard: fund balance, active members, this month's contributions, pending requests
- Reports: monthly contribution totals, benefit status breakdown, per-member statement

## Project Structure

```
fundphp/
├── app/
│   ├── bootstrap.php        # autoloader, .env loading, session start
│   ├── Core/                 # Database, Model, QueryBuilder, Router, Auth,
│   │                          # Session, Request, Controller, View, Env
│   ├── Controllers/          # Auth, Dashboard, Member, Contribution,
│   │                          # Benefit, Report, User
│   ├── Models/                # User, Member, Contribution, Benefit
│   ├── Middleware/            # Auth, Guest, Admin
│   └── Views/                  # PHP templates (Bootstrap 5 UI)
├── public/
│   ├── index.php               # front controller (all requests go through here)
│   ├── .htaccess                # Apache rewrite rules
│   └── assets/css/style.css
├── database/
│   └── migrations/
│       ├── 001_create_initial_schema.sql          # MySQL
│       └── 001_create_initial_schema_sqlite.sql    # SQLite (default)
├── scripts/
│   └── setup.php                # creates schema + default users + sample data
├── .env.example
├── composer.json
└── README.md
```

## Setup & Run

### Option A — SQLite (default, zero-config)

```bash
cd fundphp
cp .env.example .env
php scripts/setup.php          # creates database/fund.sqlite, default users, sample data
php -S localhost:8000 -t public
```

Open **http://localhost:8000**.

### Option B — MySQL

1. Create a database: `CREATE DATABASE community_fund;`
2. In `.env`, set:
   ```
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_DATABASE=community_fund
   DB_USERNAME=root
   DB_PASSWORD=yourpassword
   ```
3. Run `php scripts/setup.php` (applies `001_create_initial_schema.sql` and seeds data).
4. `php -S localhost:8000 -t public`

### Default logins

| Username    | Password       | Role      |
|-------------|----------------|-----------|
| `admin`     | `admin123`     | admin     |
| `treasurer` | `treasurer123` | treasurer |

Change these before real use, and set `APP_KEY`/a real session secret in production.

## Business Rules

Same as the Flask version, defined as constants in `BenefitController`:

```php
const LOAN_ELIGIBILITY_MIN_CONTRIBUTION = 20000;
const LOAN_MAX_MULTIPLE = 3;
```

Fund balance = total contributions − total *paid* benefits.

## Relationship to the Comprehensive Spec Document

You also shared a much larger specification (RBAC with 5 roles, SMS/WhatsApp/
email notifications, payment gateway integration, QR codes, loans/expenses/
payments as separate modules, etc.). **This PHP port matches the scope of
the original Flask app** (members, contributions, benefits, dashboard,
reports, admin/treasurer roles) — it does **not** yet implement the full
mega-spec's extra modules (Roles/Permissions CRUD, Loans/Expenses/Payments
as separate modules, SMS/WhatsApp/Email sending, QR codes, cron reminders).

If you want the full mega-spec built out, that's a much larger follow-on
project — happy to build it incrementally (e.g., start with RBAC + Loans/
Expenses modules, then notifications, then payments) rather than all at
once, so each part can be tested as it's delivered.

## Notes on Extending

- **Real notification sending**: add `SMSService`, `WhatsAppService`,
  `EmailService` classes under `app/Services/`, following the same
  sandbox/log pattern described in your spec, and call them from
  `ContributionController`/`BenefitController` after key actions.
- **QR codes**: easiest zero-dependency approach is a client-side JS
  library (e.g. `qrcode.js` from a CDN) rendering the member's `member_no`
  as a QR code in the browser — no PHP extension needed.
- **PDF/Excel export**: add a small library (e.g. `dompdf`,
  `PhpSpreadsheet`) via Composer once you have Composer/network access.
