---
description: Add prepaid credits, set up auto top-up, and download receipts.
---

# Credits and payments

This page explains how Engy billing works and how to add credits to your account. It is intended for account owners who manage spending.

## How billing works

Engy uses prepaid credits. You add credits to your account, and each request deducts its cost from your balance. There are no subscriptions or minimum spend.

The dashboard shows two amounts:

| Amount | Description |
| --- | --- |
| Balance | The total credit in your account. |
| Available (after holds) | The balance minus amounts that are temporarily reserved for requests in progress. |

When a request starts, Engy places a hold on part of your balance. When the request completes, Engy charges the actual cost and releases the rest of the hold.

The amount charged for each request is returned in the `x_engy.charged_micro` response field and the `X-Engy-Charged-Micro` header, in micro-USD. One USD equals 1,000,000 micro-USD.

## Add credits

{% stepper %}
{% step %}
### Open the credits page

In the dashboard, go to **Credits**.
{% endstep %}

{% step %}
### Choose an amount

Click **Add credits** and enter an amount in whole US dollars, from $1 to $100,000.
{% endstep %}

{% step %}
### Choose a payment method

Under **Pay with**, select a payment method. For a cryptocurrency, also select the **Network**.
{% endstep %}

{% step %}
### Complete the payment

Follow the instructions for the selected method. The credits appear in your balance after the payment is confirmed.
{% endstep %}
{% endstepper %}

## Payment methods

| Method | Networks |
| --- | --- |
| Card | Not applicable |
| USDC | Ethereum, Base |
| TAO | Bittensor |

The **Add credits** dialog shows the methods that are currently available. Some cryptocurrency methods include bonus credit, which the dialog displays before you pay.

## Auto top-up

Auto top-up charges a saved card automatically when your balance falls below a threshold. Use it to prevent requests from failing because of an insufficient balance.

To set up auto top-up:

1. In the dashboard, go to **Credits**.
2. In the **Auto top-up** card, set the following values:
   * **Threshold**: The balance below which Engy adds credits
   * **Amount**: The amount to add each time. It must be at least the threshold
3. Save a card for the charges, and then turn on auto top-up with **Toggle auto top-up**.

If a charge fails several times in a row, Engy pauses auto top-up. Update the card, and then turn auto top-up on again.

## Receipts

Each completed payment appears in **Credit history** on the **Credits** page. To view or print a receipt, click **View receipt** next to the payment.

To show your company name on receipts, add it in **Settings**.
