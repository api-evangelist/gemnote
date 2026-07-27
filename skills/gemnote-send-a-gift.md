---
name: Send a gift with Gemnote
description: Browse the Gemnote gift and greeting-card catalog and create a shipment that sends a gift to a recipient, then check its fulfillment status.
api: openapi/gemnote-openapi.yml
operations: [listGifts, listGreetingCards, createShipment, getShipment]
---

# Send a gift with Gemnote

Use the Gemnote API to send a custom gift (and optional greeting card) to a recipient and track fulfillment.

## Authentication

Every request must include an `AUTHORIZATION` header with a token issued by your
Gemnote account manager. Sandbox tokens begin with `t_` (base URL
`https://sandbox.gemnote.com/`); production tokens begin with `p_` (base URL
`https://api.gemnote.com/`). Test against the sandbox first — sandbox changes do
not affect production.

## Steps

1. **Pick a gift.** Call `listGifts` (`GET /api/v1/gifts/`) to browse the catalog.
   Use `page[number]` and `page[size]` to page through results. Note the `id` of
   the gift you want to send.
2. **(Optional) Pick a greeting card.** Call `listGreetingCards`
   (`GET /api/v1/greeting_cards/`) and note the `id` of a card to include.
3. **Create the shipment.** Call `createShipment` (`POST /api/v1/shipments/`) with a
   JSON:API body under `data.attributes`. Required fields: `giftId`, `qty`,
   `recipientName`, `email`, `addressOne`, `city`, `country`. Optional fields include
   `greetingCardId`, `recipientCompany`, `phone`, `addressTwo`, `state`, `zipcode`,
   `fromName`, `fromEmail`, `message`, `messageLanguage`. The response returns the new
   shipment with `stage: pending`.
4. **Track fulfillment.** Call `getShipment` (`GET /api/v1/shipments/{shipment_id}`)
   with `include=gift,postcard` to sideload related resources. Watch the `stage`
   attribute for progress; if fulfillment fails, `errorReason` explains why.

## Conventions & errors

- Responses use the JSON:API envelope (`data` / `type` / `attributes` /
  `relationships`, with `links` + `meta` on collections and an `included` array for
  sideloaded resources).
- There is no idempotency-key mechanism — do not retry `createShipment` blindly, as a
  retry creates a second shipment.
- Handle `401` (missing/invalid token), `404` (unknown id), and `422` (missing/invalid
  required field on create).
