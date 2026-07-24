---
title: Payment Classification Systems
date: 2026-07-24 17:13:00 +0300
categories: [Payments, Banking]
tags: [Finance, Banking]
---

## Overview

Payment classification describes the lifecycle of a transaction consisting of three stages: **Location, Initiation, and Settlement.**

## Payment Initiation

This is the critical entry point. It is the phase where a payment instruction is created, validated, and officially injected into the financial system. This phase must cover three goals:

- **Authentication (Who):** Verifying the identity of the payer (initiator) and confirming they have the legal authority to move funds from the designated account.
- **Consent & Authorization (What):** Securing the payer's explicit approval for the transaction amount, currency, and destination, while ensuring the payer's bank (the **debtor agent**) has the liquid balances or credit line to cover it.
- **Instruction Generation (How):** Translating the user's intent into a standardized, machine-readable financial message (e.g., an ISO 20022 `pacs.008` or a SWIFT MT103) and routing it to the entry node of the clearing network.

## Location

### Domestic/Local

A payment system that processes E2E payment transactions within the same country. Also known as Nation/Intra-Country Payment. It has no need for a long correspondent banking chain like cross-border payments, which makes the transfers faster and cheaper. All the participants are in the same jurisdiction. The national currency is used. Eg. RTGS, Fedwire, IMPS.

### Cross Broder

A payment is originated from one country and ends in another country. More than one currency is involved in this process. The sender and receiver are in different jurisidictions hence they have longer payment chains with correspondent banks & intermediaroes. 

It is processed mainly via Swift, which is a global messaging network. Some of the challenges of cross-border payments are multiple fees and forex margins. There is also a possibility of delays due to time zones and compliance requirements such as sanctions.

### Regional

A payment system that processes E2E payment transactions with a defined set of countries that are under a specific economic region. A single predetermined currency is used. Eg. SEPA(T@, TIPS, STEPS), Equens, Core STET (France and Belgium) and PAPSS (Pan-African Payment and Settlement System). They support trade agreements by reducing friction in regional commerce and enable near-real-time settlement, lower and local currency use.

Regional Payment Integration has a platform where participants connect directly to support domestic and cross-border payments. Although this system lightens the burden of ccross-border payments, it has its own challenges. It has difficulty in harmonizing regulations and standards since it operates on different jurisdictions. It also has issues in maintaining sufficient liquidity.

## Initiation

This is when a transaction begins. The payer initiates the transaction when they instruct their bank to move funds from their account. The payment initiation contains details such as the recipient or account destination, amount, and details required for authentication.

### Push

The payment request is initiated by the debtor/payer. Money moves from the debtor's account to the creditor's account. In this process, no authorization is required. Eg., Account-to-Account Payments

### Pull

The payment request is initiated by the creditor/payee. Money moves from the debtor's account to the creditor's account. In [](http://account.In)this process, there needs to be an authorization from the payer.

### Collaborative

The payment request is initiated by the debtor/payer. Money from the debtor's account to the creditor's account. No payment is required, but the debtor must decide whether to accept or decline. The debtor and creditor need to both agree. Eg., RTP in the FedNow Payment

## Settlement Type

This determines how and when funds are transferred and finalized between parties (typically banks or financial institutions) after a payment is initiated.

### Gross Settlement

Transaction is processed and an individual transaction. It is used to process the High Value Transactions but now Low value Transactions such as Instant Payment are also being processed in Gross Settlement, T2, CHAPS. For further reading reward this previous article; [https://kwizl.github.io/posts/Messages-Clearing-Settlement/#settlement](https://kwizl.github.io/posts/Messages-Clearing-Settlement/#settlement)

### Net Settlement

Processing of Batch Transaction, and only the net difference is transferred at the end of the settlement cycle eg. ECS, SEPA DD, SEPA CT, BACS. For further reading reward this previous article: [https://kwizl.github.io/posts/Messages-Clearing-Settlement/#settlement](https://kwizl.github.io/posts/Messages-Clearing-Settlement/#settlement)