# Supabase & Stripe Design Architecture

## Overview
This document outlines the architecture for integrating Stripe Payments with Supabase for the DungeonToolbox project.

## Schema
We utilize a dedicated `user_subscriptions` table to track billing state, which maps 1:1 with the Supabase Auth `users` table via `user_id`.

```prisma
model UserSubscription {
  userId               String   @id @map("user_id")
  stripeCustomerId     String?  @unique @map("stripe_customer_id")
  stripeSubscriptionId String?  @unique @map("stripe_subscription_id")
  tier                 String   @default("free") // "free", "player", "gamemaster"
  status               String   @default("active") // "active", "canceling", "canceled"
  currentPeriodEnd     DateTime? @map("current_period_end")
  balance              Int      @default(50)
  lifetimeUsage        Int      @default(0) @map("lifetime_usage")
  lastReset            DateTime @default(now()) @map("last_reset")

  @@map("user_subscriptions")
}
```

## Payment Flow
1.  **Checkout**:
    *   Frontend request `POST /api/v1/billing/checkout` with `priceId`.
    *   Backend checks `user_subscriptions` for existing `stripe_customer_id`.
    *   If missing, Backend creates Stripe Customer and updates DB.
    *   Backend creates Stripe Checkout Session and returns URL.
    *   User completes payment on Stripe.
2.  **Provisioning (Webhook)**:
    *   Stripe sends `customer.subscription.created` webhook.
    *   Backend validates signature.
    *   Backend updates `user_subscriptions` -> `tier = 'player'`, `balance = 1000`.

## Lifecycle Management

### Upgrades
*   **Event**: `customer.subscription.updated` (standard).
*   **Action**: Backend updates `tier` and resets `balance` to new tier limit immediately.

### Downgrades / Cancellation
*   **User Action**: User clicks "Cancel" in Stripe Billing Portal.
*   **Event**: `customer.subscription.updated` with `cancel_at_period_end = true`.
*   **Backend Logic**: Sets `status = 'canceling'`, updates `current_period_end`. **Does not change Tier/Credits**.
*   **Expiration**: At period end, Stripe sends `customer.subscription.deleted`.
*   **Backend Logic**: Sets `tier = 'free'`, `status = 'canceled'`, `balance = 50`.

### Monthly Reset
*   **Event**: `invoice.payment_succeeded` with `billing_reason = 'subscription_cycle'`.
*   **Backend Logic**: Resets `balance` to Tier Limit (e.g., 1000). **Idempotent overwrite**, not increment.

## Row Level Security (RLS)
Supabase RLS policies should be enabled on `user_subscriptions` to ensure:
*   Users can `SELECT` their own subscription data.
*   Service Role (Backend) has full access.
*   Users **cannot** `UPDATE` their own balance or tier.

```sql
-- Example RLS
CREATE POLICY "Users can view own subscription" ON "user_subscriptions"
FOR SELECT USING (auth.uid() = user_id);
```
