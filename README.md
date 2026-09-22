# 🕌 Al-Noor Masjid · Fund Management Dashboard

A lightweight, single-file web app for managing masjid donations and expenses. Built with vanilla React (no build step), connected to Supabase for auth and data.

## Live Demo

Deploy via GitHub Pages — see setup below.

## Features

- **User role** — record donations and expenses; view fund overview with charts
- **Admin role** — full transaction history, search/filter by name/date/type, delete records, charts and stats
- Session persists across page refreshes (localStorage)
- Fully responsive — works on mobile

## Tech Stack

- React 18 (UMD, no bundler needed)
- Supabase (Auth + Postgres via REST API)
- Pure HTML/CSS/JS — single `index.html` file

---

## Supabase Setup (already done if you ran the SQL)

### 1. Database tables

Run this in your Supabase SQL Editor:

```sql
create table donations (
  id uuid primary key default gen_random_uuid(),
  donor_name text not null,
  amount numeric not null,
  fund_type text not null,
  payment_method text not null,
  date date not null,
  description text,
  created_at timestamptz default now()
);

create table expenses (
  id uuid primary key default gen_random_uuid(),
  expense_name text not null,
  amount numeric not null,
  category text not null,
  date date not null,
  description text,
  created_at timestamptz default now()
);
```

### 2. Row Level Security (RLS)

Run this to lock down the tables so only authenticated users can read/write:

```sql
-- Enable RLS
alter table donations enable row level security;
alter table expenses enable row level security;

-- Authenticated users can read all records
create policy "auth_read_donations" on donations
  for select to authenticated using (true);

create policy "auth_read_expenses" on expenses
  for select to authenticated using (true);

-- Authenticated users can insert
create policy "auth_insert_donations" on donations
  for insert to authenticated with check (true);

create policy "auth_insert_expenses" on expenses
  for insert to authenticated with check (true);

-- Only admins can delete (uses user_metadata.role)
create policy "admin_delete_donations" on donations
  for delete to authenticated
  using ((auth.jwt() -> 'user_metadata' ->> 'role') = 'admin');

create policy "admin_delete_expenses" on expenses
  for delete to authenticated
  using ((auth.jwt() -> 'user_metadata' ->> 'role') = 'admin');
```

### 3. User accounts

Create users in Supabase Dashboard → Authentication → Users → Add user.

Then assign roles via the Supabase Dashboard:

1. Go to **Authentication → Users**
2. Click a user → **Edit**
3. Under **User Metadata**, set:

```json
{ "role": "admin", "full_name": "Aman" }
```

or for a regular committee member:

```json
{ "role": "user", "full_name": "Staff Name" }
```

> Users without `"role": "admin"` in their metadata default to the user (committee member) view.

---

## GitHub Pages Deployment

### First time

```bash
# 1. Create a new repo on GitHub (e.g. masjid-fund-dashboard)

# 2. Clone it and add the file
git clone https://github.com/YOUR_USERNAME/masjid-fund-dashboard.git
cd masjid-fund-dashboard

# 3. Copy index.html into the repo root (or just move this folder)
# The index.html is already the only file you need

# 4. Push
git add .
git commit -m "Initial deploy"
git push origin main

# 5. Enable GitHub Pages
# Go to repo → Settings → Pages → Source: Deploy from branch → main → / (root) → Save
```

Your site will be live at:
`https://YOUR_USERNAME.github.io/masjid-fund-dashboard/`

### Updating

```bash
git add index.html
git commit -m "Update"
git push
```

GitHub Pages auto-deploys within ~60 seconds.

---

## Project Structure

```
masjid-fund-dashboard/
└── index.html      ← entire app (HTML + CSS + JS)
└── README.md       ← this file
```

No `node_modules`, no build step, no config files. Just `index.html`.

---

## Customisation

All editable at the top of the `<script>` block in `index.html`:

| Constant | Purpose |
|---|---|
| `SUPABASE_URL` | Your project URL |
| `SUPABASE_ANON_KEY` | Your anon/public key |
| `FUND_TYPES` | Donation fund categories |
| `PAYMENT_METHODS` | Accepted payment methods |
| `EXPENSE_CATEGORIES` | Expense categories |

To rename the masjid, search for `Al-Noor Masjid` in `index.html` — appears in the title, navbar, and login page.
