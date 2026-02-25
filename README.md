# Stripe → Supabase Entitlements Webhook (Deno/TypeScript)

A small webhook service that verifies Stripe signatures and updates a Supabase table to grant user entitlements after a successful checkout.

## What it does
- Accepts Stripe webhook `POST` requests
- Verifies request authenticity using the `stripe-signature` header
- Handles `checkout.session.completed`
- Extracts the customer email and **upserts** an entitlement record into Supabase

## Why it exists
Subscription or one-off purchase flow, linking to purchase platform (utilised Thunkable, but anything that can check against supabase is viable). This webhook updates a Supabase `entitlements` table so the rest of the app can check access quickly and consistently.

## Data model (Supabase)
Example table schema:

```sql
create table if not exists entitlements (
  email text primary key,
  entitled_web boolean not null default false,
  updated_at timestamptz not null default now()
);

create or replace function set_updated_at()
returns trigger as $$
begin
  new.updated_at = now();
  return new;
end;
$$ language plpgsql;

drop trigger if exists trg_updated_at on entitlements;
create trigger trg_updated_at
before update on entitlements
for each row execute function set_updated_at();
