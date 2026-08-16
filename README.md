# Custodia

**Anti-scalping event ticketing, enforced entirely on-chain.**

Custodia is a decentralized event ticketing protocol built on GIWA Sepolia. It gives organizers full control over how they sell tickets  flexible pricing tiers, dynamic tier progression, waitlists  while making scalping mathematically impossible at the smart contract level. Every purchase, resale, check-in, cancellation, and refund is recorded and enforced on-chain, with no centralized authority deciding what's "fair."


## The Problem

Ticket scalping has been a broken part of live events for decades. The moment tickets go on sale, bots and resellers buy up entire tiers in seconds, then relist them on secondary markets at 3x, 5x, sometimes 10x the original price. Real fans  the people the event was actually priced for — either get locked out entirely or forced to pay a scalper's markup just to attend.

Centralized ticketing platforms have tried to solve this with manual policy enforcement: banning bulk purchases, requiring ID verification, capping resale on their own marketplace. It mostly doesn't work, because none of those rules are enforced anywhere except inside a company's internal database  easy to route around with new accounts, different payment methods, or simply reselling off-platform.

## The Custodia Approach

Custodia moves the rule itself onto the blockchain. When an organizer creates an event, they set a resale cap as a percentage of the original purchase price — for example, 110%, meaning a ticket can never be resold for more than 10% above what its current holder paid. This isn't a policy that a support team enforces after the fact. It's a `require()` statement in the smart contract. If someone tries to list a ticket above the cap, the transaction simply reverts. There's no way around it that doesn't involve rewriting the contract itself.

This shifts trust away from a platform's internal moderation and onto code that anyone can read, verify, and audit.

---

## Core Features

### Flexible Pricing Tiers
Organizers aren't locked into a rigid "Early Bird / General / VIP" structure. Each event can have any number of custom tiers — different names, prices, and supply counts, defined entirely by the organizer. Tiers can be configured to automatically open the next tier the moment the current one sells out, or the organizer can manually open and close tiers at will for more granular control over pacing.

### On-Chain Resale Cap
Every event carries its own resale ceiling, set once at creation as a percentage (in basis points) of the original price. The cap is checked against whatever the *current* holder actually paid — not the original face value  so the ceiling stays meaningful even after multiple resales.

### Optional Platform Resale Fee
The contract owner can enable or disable a resale fee, capped at a hard maximum of 20% to prevent abuse. When enabled, this fee is automatically deducted from the seller's proceeds at the moment of resale and tracked separately for withdrawal.

### Smart Waitlist System
When a tier sells out, interested buyers can join a waitlist instead of being turned away entirely. Organizers choose, per event, how freed-up slots are distributed:
- **Automatic Transfer** the next waitlisted buyer receives the ticket the moment a slot opens, no action required from them.
- **Time-Window Offer** — the buyer is offered the slot and given a fixed window to claim it before it passes to the next person in line.

### Dual-Mode Check-In
Custodia supports two ways to verify a ticket at the door, depending on how much control an event needs:
- `isValidTicketHolder()` — a lightweight, gasless view function anyone can call to confirm a wallet currently holds a valid, unrefunded ticket.
- `checkIn()` — an organizer-only function that permanently marks a specific ticket as checked in on-chain, useful for events that need an auditable attendance record.

### Cancellation & Batch Refunds
If an event has to be cancelled, the organizer doesn't need to refund holders one by one. They fund a single transaction with enough ETH to cover every outstanding ticket, and the contract loops through every holder, refunds them automatically, and marks each ticket as refunded — with any excess sent back to the organizer.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Smart Contract | Solidity `^0.8.20` |
| Network | GIWA Sepolia (OP Stack Layer 2) |
| Frontend | React + Vite + TypeScript |
| Styling | Tailwind CSS |
| Web3 Layer | Wagmi + Viem |
| Wallet Connection | RainbowKit |
| Animation | Framer Motion |

---

## Deployment Details

| | |
|---|---|
| **Network** | GIWA Sepolia |
| **Chain ID** | `91342` |
| **RPC URL** | `https://sepolia-rpc.giwa.io` |
| **Block Explorer** | `https://sepolia-explorer.giwa.io` |
| **Contract Address** | `0x62bb24bF96b52783146591398e783E5CA30e892f` |
| **Verified Source Code** | [View on Explorer](https://sepolia-explorer.giwa.io/address/0x62bb24bF96b52783146591398e783E5CA30e892f?tab=contract) |
| **Native Currency** | ETH (all payments are handled via native `payable` functions, not an ERC20 token — GIWA Sepolia does not yet have a native stablecoin) |

---

## How It Works, Step By Step

1. **Event Creation** — An organizer calls `createEvent()`, setting the event name, date, resale cap percentage, and waitlist mode.
2. **Tier Setup** — The organizer calls `addTier()` for each pricing tier they want to offer, defining its name, price in ETH, and total supply.
3. **Primary Sale** — Buyers call `buyTicket()`, sending the exact tier price as `msg.value`. Funds are transferred directly to the organizer's wallet on purchase. If a tier sells out, the contract can automatically open the next tier.
4. **Waitlisting** — Once a tier is full, new interested buyers call `joinWaitlist()`. If a slot later frees up, the organizer calls `offerWaitlistSlot()`, and depending on the event's waitlist mode, the ticket either transfers immediately or opens a claim window via `claimWaitlistOffer()`.
5. **Resale** — A ticket holder who no longer needs their ticket can call `resellTicket()`. The contract checks the offered price against `getMaxResalePrice()` before allowing the transfer — anything above the cap is rejected automatically.
6. **Check-In** — At the event, staff can verify holders with the free `isValidTicketHolder()` check, or use the organizer-only `checkIn()` function to log attendance permanently on-chain.
7. **Cancellation (if needed)** — The organizer calls `cancelEvent()` followed by `refundAllTickets()`, funding and triggering a full batch refund to every remaining ticket holder in one transaction.

---

## Why Build This On-Chain

A traditional ticketing backend could technically enforce a resale cap too — but only within its own walled garden, and only as long as the company chooses to keep enforcing it. Moving that logic into a smart contract means:

- **The rule can't be quietly changed** without it being visible on-chain.
- **Anyone can verify** that the cap is actually being enforced, rather than trusting a platform's word for it.
- **Resale doesn't have to happen on a single company's marketplace** — the enforcement travels with the ticket itself, wherever it's transferred.
- **No custodial risk** — organizers receive funds directly, and refunds don't depend on a company's solvency or willingness to process them.

---

## Roadmap

- **Phase 1 — Testnet Launch (current)**
  Core contract live and verified on GIWA Sepolia. Tiered pricing, on-chain resale cap enforcement, dual-mode waitlist, organizer check-in tools, and cancellation/refund flow all functional end-to-end.

- **Phase 2 — Stablecoin Support**
  As GIWA-native stablecoins become available, extend the payment flow to support ERC20 tokens alongside native ETH, giving organizers and buyers price stability options.

- **Phase 3 — Organizer Tools**
  A dedicated analytics dashboard for organizers — sales breakdown by tier, resale activity, waitlist conversion — plus bulk tier management and CSV export of attendee/check-in data for offline event operations.

- **Phase 4 — Mainnet & Mobile Check-In**
  Full mainnet deployment alongside a dedicated mobile scanner app for door staff, and support for multi-organizer team accounts so larger events can delegate check-in and tier management.

- **Phase 5 — Cross-Event Discovery**
  Public organizer profiles, event discovery and recommendation surfaces, and category-based browsing to help Custodia scale beyond single-event use into a broader event discovery platform.

---

## Disclaimer

Custodia is an independent project built on GIWA Sepolia testnet as part of ongoing ecosystem development. It is not an official GIWA product and is not affiliated with, endorsed by, or sponsored by Upbit or Dunamu.

---

## Built By

**Rahman**
[GitHub](https://github.com/rahmansial477) · [X](https://x.com/rahmansial477)
