# Agro Zanjir Digital

The digital layer of the Agro Zanjir value chain: one lot ID, one event log,
and the ports that make what happened physically into a record a bank, an
insurer or a customs broker can act on.

Two shells, both running.

```
backend/   Django 6 + DRF + PostgreSQL       →  http://localhost:8000
web/       Vite + React + TypeScript         →  http://localhost:5173
```

## Start both

```sh
# terminal 1
cd backend
.venv/bin/python manage.py migrate
.venv/bin/python manage.py seed_reference   # roles, capabilities, org types, checks
.venv/bin/python manage.py seed_demo        # the pilot dataset
.venv/bin/python manage.py runserver 8000

# terminal 2
cd web && npm install && npm run dev
```

Open http://localhost:5173 for the public website; `Sign in` leads to the panel
index at `/panels`. The overview screen's health card turns green when it can
reach the backend.

## What exists

- **The database and the API.** Thirty-eight models across the lot spine and the
  six clusters, with the four expensive rules enforced in code: the event log
  is append-only and hash-chained, an encumbrance blocks dispatch without ever
  being a status, no cluster holds a foreign key into another, and quantities
  are grams while money is minor units plus a currency. Every panel screen is
  served by it, permissions are capability-based, and reads of a lot passport
  are written to the audit log. 48 backend tests pass.
- **Sessions.** OneID at the front, JWT behind it, the access token in memory
  and the refresh token in an httpOnly cookie. OneID itself is not connected:
  the backend's stub adapter resolves a seeded person and says so in every
  session it issues.
- **Web**: three-language i18n, theme, a party-scoped session store and two
  route guards, the app shell, and a module manifest that generates the
  platform view's navigation, routes and placeholder pages.
- **The public website**, built from its own approved prototype: eleven pages
  at `/`, three languages, light and dark. Its copy and catalogue are content
  (`web/src/lib/site-data.ts`); the lot card in the hero and the ZEROCO
  comparison chart are real records, read from two endpoints that need no
  session - provenance is public, which is the argument the site is making.
- **The eight user panels**, built from the approved HTML prototype: producer,
  hub operations, ZEROCO trial, bank, insurer, export, public and
  administration - 44 screens over one shared design system, in all three
  languages, light and dark. They read the API. The figures in it are the pilot
  demonstration dataset, loaded by `manage.py seed_demo`, and every screen
  carries the demonstration bar that says so.

## What does not exist

**No live port adapter.** All five ports run their manual adapter: a finance
application, a lien, a claim, a booking or a customs declaration is recorded
and waits for an operator to resolve it from the admin. The seam is the
deliverable; the integrations are phase 3.

**Nobody signs in as themselves yet.** The gate, the token, the refresh cookie
and the verification check are all real. The identity behind them is not: the
stub adapter resolves a seeded person, and the sign-in screen prints which
adapter answered so a demonstration session can never pass for a real one.

**Most screens still only read.** The write endpoints exist and the spine
refuses what it should - a pledged lot cannot leave, an ungraded lot cannot be
put away, a full zone will not take another pallet - but wiring each form to
its endpoint is the next piece of work.

Also missing: the offline field-capture PWA, and row-level security in
PostgreSQL (scoping is enforced in the application layer today).

## The rules worth not breaking

1. **The event log is the truth; the lot row is a cache.** `lot_event` is
   append-only and hash-chained. That chain is the tamper evidence — and the
   reason no blockchain is needed.
2. **Lots split and merge.** Model `lot_relation` from day one, or traceability
   breaks in the first week of real use.
3. **Encumbrance is an overlay, not a status.** A pledged lot is still stored,
   still reserved, still shippable — it just cannot leave without the lender's
   release. Building it as a status value forces a rewrite when inventory
   finance goes live.
4. **One base unit.** Integer grams. Boxes, crates and pallets are
   presentation.
5. **Money is integer minor units plus a currency,** with the FX rate
   snapshotted on the transaction. This project spans UZS, USD and JPY.
6. **Clusters never reference each other.** A shipment does not know about a
   loan; a QC record does not know about a buyer. Everything meets at the lot.
   That constraint is what lets three people build six modules without constant
   merge conflicts.
7. **Every external relationship is a port with a manual adapter first.** Ship
   without a single bank meeting; add the API adapter when the bank is ready.

## Scope

This repository is the middle band: the platform. It does not store produce,
lend money or clear customs. The physical band (cold rooms, ZEROCO chambers,
reefer transport) and the institutional band (bank credit products, insurance
policies, customs procedures) are dependencies owned by the client, not
deliverables here.
