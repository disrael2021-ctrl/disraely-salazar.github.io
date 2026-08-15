---
title: "Osteon: Building a Surgical Case & Insurance Reconciliation Platform"
date: 2026-08-15
draft: false
author: "Disraely Salazar"
tags: ["Next.js", "TypeScript", "PostgreSQL", "Prisma", "SaaS", "Healthcare Tech"]
categories: ["blog"]
description: "How I built and shipped a production web app for two orthopedic surgeon partners in Barcelona to track surgical cases and reconcile insurance payments against a 70/30 revenue-split agreement — from data model through deployment."
---

My brother-in-law is a traumatologist with a private clinic partnership in Barcelona. He and his partner split every surgery's billing 70/30 based on who operated — main surgeon gets 70%, assisting partner gets 30%, and the split flips depending on who's in which role for a given case. The problem wasn't the math. It was that insurers pay weeks after the surgery, often a different amount than expected, and the two of them were reconstructing who-was-owed-what from memory and a paper notebook at month end. That's how payment disputes start between partners.

I built **Osteon** to fix that: a same-day entry habit (log who operated the moment the surgery happens) feeding a reconciliation queue (fill in the real insurer payment when it lands, watch the split compute itself). This is a walkthrough of the data model, the money-correctness decisions, and a couple of production infrastructure lessons that only showed up after real doctors started using it.

<!--more-->

---

## Overview

Two orthopedic surgeons, one shared clinic, one 70/30 revenue-split agreement that flips per case depending on who's "main doctor" versus "assistant." The system needed to answer, for any given month:

- What's expected vs. what's actually been paid, per case?
- Whose share is whose, once a payment lands?
- Which cases are still sitting unreconciled, and for how long?

And it needed to do all of that while touching as little patient data as legally possible — this is health-adjacent billing information in the EU, and the fewer identifiers I store, the less exposure under GDPR/LOPDGDD special-category data rules.

---

## Data Model

Built on **Next.js (App Router) + TypeScript + Prisma + PostgreSQL**, hosted on Vercel with a Neon Postgres database in the EU (Frankfurt) region.

**Data minimization, enforced by the schema itself:** the `Case` model stores a `patientRefCode` — an opaque invoice reference — and nothing else patient-identifying. No name, no date of birth, no contact info. That constraint is commented directly in `schema.prisma` so it can't be quietly violated by a future migration.

**Money is always integer cents**, never floats, end to end — `expectedAmountCents`, `actualAmountReceivedCents`, `societyShareCents`. A rounding bug in a payment-splitting app isn't a cosmetic issue, so this was non-negotiable from the first migration.

**Derived values are computed once, at read time, from one function** — status (`PENDIENTE` / `CONCILIADO`), the difference between expected and actual, and each party's share — never stored redundantly. One pure, unit-tested function (`computeCaseDerived`) is the single source of truth, called from every screen that needs it:

```ts
export function computeCaseDerived(input: CaseMoney): CaseDerived {
  const status = input.actualAmountReceivedCents == null ? "PENDIENTE" : "CONCILIADO";
  const finalAmountCents = input.actualAmountReceivedCents ?? input.expectedAmountCents;
  const differenceCents =
    input.actualAmountReceivedCents == null
      ? null
      : input.actualAmountReceivedCents - input.expectedAmountCents;
  const societyShareCents = Math.round((finalAmountCents * input.societyPercentage) / 100);
  const doctorShareCents = finalAmountCents - societyShareCents;
  return { status, finalAmountCents, differenceCents, societyShareCents, doctorShareCents };
}
```

---

## Modeling How Surgeries Actually Get Billed

The first version of the case-entry form assumed one procedure, one amount per case. Real usage broke that assumption fast: a single surgery session can involve two separate pathologies (an arthroscopy plus a separate repair), and a single procedure can be billed under two or three different insurer codes, each with its own amount, summing to the total charge.

That meant introducing a proper one-to-many relationship — a `CaseLineItem` table, each row holding one pathology and one billed amount, with a `Case` owning as many line items as the surgery actually needed:

```prisma
model CaseLineItem {
  id          String    @id @default(cuid())
  caseId      String
  case        Case      @relation(fields: [caseId], references: [id], onDelete: Cascade)
  pathologyId String
  pathology   Pathology @relation(fields: [pathologyId], references: [id])
  amountCents Int
  order       Int       @default(0)
}
```

The UI mirrors the real mental model: pick a procedure once, then add as many billing codes underneath it as were actually charged, with a running subtotal per procedure and a total for the whole case. `expectedAmountCents` is still the sum, computed once at creation and persisted — same "computed once, stored" pattern as before, just fed by an array instead of two hardcoded fields.

Since this table replaced two nullable columns on `Case` directly, the migration needed care: production had zero existing case rows at the moment it ran (verified live before deploying), so there was nothing to lose — but the migration file itself carries no backfill guarantee, which is worth remembering as a pattern, not repeating blind.

---

## The Reconciliation Queue

This is the highest-leverage screen in the app, and the one the original paper-notebook workflow didn't have an equivalent of. It's a single Pending-only queue, oldest case first, with the real payment amount entered inline — no separate edit page, no navigating away from the list:

```ts
export async function reconcileCase(input: ReconcileCaseInput) {
  const parsed = reconcileCaseSchema.parse(input);
  const updated = await prisma.case.update({
    where: { id: parsed.caseId },
    data: {
      actualAmountReceivedCents: parsed.actualAmountReceivedCents,
      reconciledAt: new Date(),
      reconciledByUserId: session.user.id,
    },
  });
  await writeAuditLog({ /* old/new values, for the "who changed this" question */ });
  return updated;
}
```

The moment a real amount is entered, the case flips from Pending to Reconciled and both partners' shares recompute automatically — the exact reconstruction that used to happen manually, once a month, from memory.

---

## Two Production Lessons

**Serverless cold-starts are a real UX bug, not a footnote.** Neon's free-tier Postgres suspends after inactivity. A login attempt right after an idle period could take the database over 15 seconds to wake up — longer than Vercel's serverless function timeout — so the request died with no response at all. I traced this from a `responseStatusCode: -1` entry in the production logs (a request that never got a response) and confirmed it by reproducing a 16-second login myself. The fix was a lightweight `/api/warmup` endpoint pinged on a schedule to keep the database from ever going fully idle during clinic hours.

**Auth.js redacts thrown errors in production builds, silently.** A `throw new Error("wrong password")` inside a Server Action shows the real message in local dev — and a generic, useless one in production, by design (it's a security feature against leaking implementation details). That difference meant a bug only showed up when tested against the live deployment, not locally. The fix: return `{ success: false, error }` result objects for expected validation failures instead of throwing, and reserve `throw` for genuinely unexpected states.

---

## Skills Demonstrated

- **Relational schema design under a real constraint** — data minimization enforced structurally, not just by policy, plus a one-to-many redesign (`CaseLineItem`) driven by how the business actually bills
- **Money-safe application logic** — integer-cents storage, single-source-of-truth derived calculations, unit-tested split logic
- **Full-stack TypeScript** — Next.js App Router, Server Actions, Prisma 7 with driver adapters, Auth.js credentials-based sessions
- **Production debugging** — diagnosing a serverless cold-start failure from raw logs, and a framework-level error-redaction gotcha that only reproduces in production
- **Shipping to real users** — this isn't a portfolio demo; it's live, and two practicing surgeons use it to track real cases and real money

*Disraely Salazar is a Data Analytics student at triOS College in Toronto, ON, with a background in business operations and supply chain management.*
