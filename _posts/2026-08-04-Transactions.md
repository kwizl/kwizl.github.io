---
title: Transactions in Banking and Payments
date: 2026-08-04 10:00:00 +0300
categories: [Payments, Banking]
tags: [Finance, Banking]
---

## Introduction

In this article, we will delve into the flow of money transactions across the banking world. We will discuss the various types of money transactions and what each entails.  A transaction is an event that creates or settles an obligation involving money.

## Credit Transfer

This is the request initiated by a debtor; hence, it is a push transfer transaction where a payer (debtor) instructs their bank to move money from their account directly into the payee’s (creditor’s) account. Usually, the creditor gets money within a few seconds up to a few days based on the chosen credit transfer method. The money is irrevocable once it is credited to the beneficiary. Transaction charges are applicable as per the Bank’s norms and chosen credit transfer method. It can be settled by either gross or net settlement method. Most common use cases are paying a vendor, salary credit, A2A Transfer, Transfer to Friends

In the SWIFT ecosystem, a **Credit Transfer** is an instruction sent by a debtor (the sender) to move funds from their account to a creditor (the receiver). Instead of the receiver "pulling" the money (like a direct debit), the sender "pushes" it through the banking network.

With SWIFT's global migration to the **ISO 20022 messaging standard**, credit transfers are primarily handled using **pacs (Payment Clearing and Settlement)** messages, replacing the legacy MT (Message Type) formats.

#### How It Works: The Core Messaging Flow

When a credit transfer crosses borders or distinct banking systems, it typically involves a chain of financial institutions. Under ISO 20022, the process follows a structured sequence.

**1.Payment Initiation**

The corporate client or individual initiates the transfer, sending a **pain.001 (Payment Initiation)** message to their bank (Bank A). This contains the ultimate debtor, ultimate creditor, amount, and currency details.

**2.Interbank Settlement Instruction**

Bank A processes the request. If the funds need to move to Bank B in another country, Bank A routes a **pacs.008 (FI to FI Customer Credit Transfer)** message through the SWIFT network. This is the core message that actually instructs the movement of the customer's funds between the financial institutions.

**3.Cover Payments**

If Bank A and Bank B don't have a direct relationship (nostro/vostro accounts), the message travels via an intermediary correspondent bank. A **pacs.009 (Financial Institution Credit Transfer)** is used to move the actual liquidity across the settlement accounts, while the pacs.008 carries the underlying commercial data.

**4.Payment Status Report**

Throughout the journey, banks use the **pacs.002 (Payment Status Report)** to reject, accept, or flag transactions for compliance matching (e.g., AML/Sanctions screening).

**5.Cash Management Advice**

Once Bank B receives the funds and the **pacs.008** clears, it credits the final beneficiary's account and transmits a **camt.054 (Bank-to-Customer Debit/Credit Notification)** to inform them the money has arrived.

## Book Transfer

It is similar to both Credit/Debit Transfer. Because both the sender (debtor) and the receiver (creditor) hold accounts at the same bank, the money never actually leaves the institution. The transfer is completed entirely by updating the bank's internal database. It is irrevocable once money is credited to the beneficiary. Usually, no charges apply to these transactions. An example is a savings account to a loan account in the same bank.

#### Architectural Mechanics

When funds move between two different banks, the transaction requires external clearing houses (like ACH) or settlement systems (like an RTGS or SWIFT). A book transfer completely bypasses this entire infrastructure.

```
Book Transfer:
[Sender Account] ─── (Internal Ledger Database Update) ───> [Receiver Account]
```

### The Database Transaction Lifecycle

From a software and core banking perspective, a book transfer is executed as a single, atomic database transaction wrapped in an ACID-compliant block (Atomicity, Consistency, Isolation, Durability).

**1.Debit Account Verification:**

The core banking system verifies that the sender's account exists, is active, and possesses sufficient available funds to cover the transaction amount.

**2.Acquire Ledger Locks**

The database engine places an exclusive row lock on both the sender's and receiver's balance records to prevent race conditions (such as simultaneous withdrawals).

**3.Atomic Balance Update**

The ledger engine executes two simultaneous actions inside a single database commit: it decrements the sender's balance and increments the receiver's balance by the exact same amount.

**4.Audit Trail Generation**

The system generates matching internal transaction logs, recording a debit and a credit journal entry pointing to the same internal transaction identifier. No messaging payloads (like SWIFT `pacs.008`) are generated or sent to external rails.

#### Common Use Cases

1. **Internal Account Sweeps:** A corporate client moving liquidity from an operational checking account to an overnight sweep or savings account within the same institution.
2. **Intra-Company Closed-Loop Ecosystems:** Peer-to-peer applications or digital wallets where users hold balances inside a single underlying custodial banking partner (e.g., two users transferring money to each other inside an app backed by the same ledger).
3. **Internal FX Operations:** A corporate entity executing a currency exchange between its own multi-currency accounts held at a single global treasury bank.

## Direct Debit Transfer

This request is initiated by the creditor via the Creditor Agent to collect payments. Mostly recurring payments. Hence, it is a pull transfer. The debtor must provide authorization/consent to the creditor for debiting the account. Usually, the creditor gets money in one or two days from the due date. Direct Debit notification must be sent to the debtor before the due date, and the debtor may dispute the transaction in case the account is debited by mistake. Uses are paying utility bills, subscriptions.

A bank will never execute a direct debit unless a legal and technical framework is in place. This framework is governed by a **Mandate** (or Pre-Authorized Debit Agreement). A **Mandate** formal authorization signed by the debtor that grants the creditor permission to initiate collections from their bank account. It specifies whether the withdrawals are for fixed or variable amounts and defines the frequency (e.g., monthly utility bills).

#### Transaction Lifecycle

Because direct debits are "pull" mechanisms, they carry an inherent risk of fraud or insufficient funds. As a result, their clearing cycle differs significantly from standard push transfers and relies on a multi-day clearing layout.

Under the **ISO 20022** standard, direct debits shift away from legacy flat files to **pain (Payment Initiation)** and **pacs (Payment Clearing and Settlement)** XML message formats.

**1.Collection Initiation**

The creditor generates a batch file containing the mandate references, amounts, and debtor bank account details (like IBANs). They upload this file to their bank (Bank A) using a **pain.008 (Customer Direct Debit Initiation)** message.

**2.Debtor Advise**

Regulations usually require the creditor to notify the debtor (typically 14 days in advance, though often shortened by contract) of the exact amount and date the funds will be pulled.

**3.Interbank Settlement Request**

Bank A routes the collection request into an Automated Clearing House (ACH) or clearing network using a **pacs.003** message. This message is routed to the debtor's bank (Bank B).

**4.Validation & Debit**

Bank B validates the mandate reference. It verifies if the debtor has sufficient funds. If everything passes, Bank B debits the debtor's account. If it fails (e.g., *Insufficient Funds* or *Mandate Expired*), Bank B generates a **pacs.002 (Payment Status Report)** indicating a **Reject**.

**5.Clearing & Finality**

The clearing house nets the positions between Bank A and Bank B. The funds are credited to Bank A, which then updates the creditor’s account balance.

#### Request For Payments

When a request is sent by the creditor to the debtor using pain.013 message, the debtor receives this request from the debtor agent with the option to accept or decline. Upon accepting the request, a pacs.008 credit transfer message is initiated by the debtor agent to the creditor agent. Upon declining the request, a status update(pain.014 message) is sent to the creditor agent. It is not like direct debit, as the request goes directly to the debtor for making the decision.

## R-Transactions

These transactions are considered a negative flow because it is for either rejecting ot returning or recalling a previous transaction. These flows cannot be processed independently, it must be used against a previous payment transaction. When a payment rail (like SEPA or NACHA) or a messaging system (like ISO 20022) encounters an exception, it marks it with a specific operational code beginning with the letter **"R"**. While they occur across both push and pull networks, handling R-transactions is arguably the most complex aspect of building core banking systems, particularly for Direct Debits.

**Push Payments:**

1. Reject
2. Recall
3. Return

### Reject

The rejection of the message can be divided into two; Business Rejection or Technical Rejection. The Rejection can be done at any point in the payment chain by any of the Agents or Clearing System as long as the message is not meeting the criteria either for Business Validation or Technical Validation. pacs.002 status report is sent to the sender of the message with reject reason and payment flow is stopped right there.

### Recall

This request is initiated by the dentor agent to cancel the previously sent value message by usign camt.056 message. The value message is cancelled only if money is not setttled between the naks otherwise, a return process is follwoed. If creditor agnet received this recall request, before crediting the beneficiary then a return payment is initiated on their own and it is up to the creditor to accept or decline the request. Recall Request os not always guaranteeing the amount to be returned from the creditor.

### Return

This is a flow of returning the money back to the debtor by a creditor agent after a settlment using pacs.004 message. Return can happen for two reasons, its either due to the pacs.008 message rejeccction by the creditor agent or for honoring the recall request sent by the dentor agent. The role of an actor in the pacs.008 original message flow is reversed in the return flow. Creditor Agent may charge a fee for returning a payment hence dentor return amount can be different from the original amount. Message Id, Interbank Settlement Date, Amount, Instruction Id, these are all some of the data is used to match the previously received pacs.008 for which return payment to be generated.

### Reversal

This request is initiated by a creditor to their creditor agent. It is requested after the settlement date. Clearing and Settlement executes the fund movement from the creditor agent’s settlement account to the debtor agent’s settlement account. Debtor agent credits back the debtor account

```
[Payment Initiated] ──> (Bank Validation Fails) ───────> REJECT
                    ──> (Clearing House Fails) ────────> RETURN
                    ──> (Debtor Stops Payment) ────────> REFUSAL
                    ──> (Settled Payment Reversed) ────> REVERSAL / REFUND / REVOCATION
```

When designing ledger software, handling an R-Transaction requires strict **state machine** enforcement. A payment record should transition linearly: `Pending` ➔ `Settled` ➔ `Returned`.

Because an R-transaction represents a complete reversal of financial state, ledger updates must be wrapped in isolated, atomic database operations. Simply updating a status column to "Returned" is insufficient; the system must generate a brand new ledger journal entry matching the R-transaction's precise ID, debiting the creditor's internal accounting wallet and crediting the debtor's balance to maintain a perfect, unalterable historical audit trail.