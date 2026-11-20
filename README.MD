# Närkefrakt EDI Payload Schemas

## Overview
Message schemas and worked examples for freight integrations with Närkefrakt. Covers three message types in both JSON and XML. Integrators send orders; Närkefrakt returns tracking and delivery events for the same consignment.

## Flow terms description

To describe what we at Närkefrakt mean when we talk about order, tracking, deliveries etc (Despite them appearing obvious) here's a simple example for clarity:

1. Integrator sends an **Order** to Närkefrakt.
2. Närkefrakt returns **Tracking Events** across the consignment's lifecycle — `GOODS_PREPARED`, `LOADED_AT_CONSIGNOR`, `UNLOADED_AT_TERMINAL`, `LOADED_AT_TERMINAL`, `LOADED`, `DELIVERED_TO_CONSIGNEE`. The tracking schema documents what each one means.
3. On delivery, Närkefrakt returns a **Delivery Event** carrying proof of delivery (signature, graphic signature, attachments).

Messages are correlated by the references sent on the order — normally `ConsignmentNumber` and `OrderNumber`, which are echoed back on every event.
And the same terms are used in the opposite direction when Närkefrakt sends an order to you as a partner to deliver on our behalf.


## Message Types
| Message | Direction | JSON schema | XML schema |
|---------|-----------|-------------|------------|
| Order | Integrator → Närkefrakt | `närkefrakt-order-schema.json` | `nf-order-schema.xml` |
| Tracking Event | Närkefrakt → Integrator | `närkefrakt-status-schema.json` | `nf-status-schema.xml` |
| Delivery Event | Närkefrakt → Integrator | `närkefrakt-delivery-event-schema.json` | `nf-deliver-schema.xml` |

JSON lives in `Närkefrakt EDI JSON/`, XML in `Närkefrakt EDI XML/`. Every schema has a matching `-example` file describing one consignment end to end — read those first.

The JSON schemas are [JSON Schema draft-07](https://json-schema.org/specification-links#draft-7) and can be used directly with any standard validator. The XML files are **annotated skeleton documents, not XSDs** — element names and comments define the contract.

## Versioning
Each version of the format is a branch. The current version is available both under its number and as `latest`.

| Branch | Use |
|--------|-----|
| `v1.0` | Numbered release — pin here for a contract that will not move |
| `latest` | Always the newest released version |

Pin to a numbered branch in production. Breaking changes go to a new numbered branch; `latest` moves with it.

## Header
Every message carries the same header, describing the message itself rather than the freight.

| JSON | XML | Required | Description |
|------|-----|----------|-------------|
| `messageId` | `<MessageId>` | no | Unique identifier for the message. Any string — a counter, a file name, or a timestamp-derived number. Lets Närkefrakt discard duplicates when a message is retried. |
| `messageTimestamp` | `<MessageTimestamp>` | **yes** | When the message was created, ISO 8601 with UTC offset. Required so a delayed message can be told from a current one. |
| `messageSource` | `<MessageSource>` | no, preferred | Which system, customer or integration produced the message — a TMS or ERP name, a customer identifier, or an integration name. Makes a message traceable to its origin when several senders share an endpoint. |

## Conventions
- **Omit optional fields you have no value for.** Do not send `null`, empty strings, or empty arrays.
- **Timestamps are ISO 8601 with offset** (`2026-07-02T08:00:00+02:00`). `deliverAfter` / `deliverBefore` also accept a bare `HH:MM`.
- **`messageTimestamp` is when the message was built; `eventTimestamp` is when the event happened.** They differ when a driver's device reports late — always trust `eventTimestamp` for what occurred.
- **Casing:** JSON keys are camelCase, XML elements PascalCase, XML attributes lowercase.
- Numeric identifiers are typed as **strings**, including `messageId`. Values exceeding 64-bit range must not be sent as JSON numbers.
- Validate against the schema before sending.

## Transport
Närkefrakt supports a wide range of message delivery methods. **SFTP** and **REST API** are the most common. Others can be accommodated — ask.

## Description of addon services


|Addon Code| Swedish term | Description |
|----------|-------------|-------|
ADR_GOODS | Farligt gods / Farligt avfall | The transport contains dangerous goods or dangerous waste.
BOOK_DELIVERY | Avisering | Närkefrakt staff will contact the recipient to coordinate a delivery day / time window before attempting a delivery. No delivery attempt will be made unless contact has been established. 
TAIL_LIFT | Bakgavellift | The transport requires a tail lift for loading / unloading.
DELIVERY_NOTIFICATION | Leveransbesked | The driver will contact the recipient 30-60 minutes (by phone) before delivery. 
EARLY_DELIVERY | Leverans före kl 12:00 | The delivery requires to be delivered before 12:00 Swedish time
LATE_DELIVERY | Leverans efter 12:00 | The delivery requires to be delivered after 12:00 Swedish time
TIME_SLOT_TRANSPORT | Tidsbestämd lastning/lossning | The transport requires loading or unloading at a fixed time, or fixed time +/- 30 minute interval. 
PACKAGE_LABELING | Utskrift av kollietiketter | Närkefrakt terminal staff will print and put package labels on all parts of a delivery.
WAYBILL_LABELING | Utskrift av inrikes fraktsedel och etiketter | Närkefrakt terminal staff will print a swedish transport waybill on the parcel.


Updated pricing and terms & conditions of Närkefrakts addon services can be located here: https://www.narkefrakt.se/produkter-tjanster/fjarr-distribution under "Dokument" (Swedish only)

## Contact
Närkefrakt IT Department — integration support and onboarding.

- it@narkefrakt.se
- +46 19 20 63 11
