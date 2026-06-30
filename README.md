# n8n Business Automations

A small library of production-shaped [n8n](https://n8n.io) workflows covering the
work I do most often for clients: CRM/ESP migration prep, email deliverability,
lead routing, and support operations. Each workflow is self-contained and uses
**synthetic sample data**, so you can import it and run it without touching a real
client system.

Built by [Lotfi O.](https://oldev.site) (OLDEV). I work as a WordPress systems
engineer with a focus on CRM migrations and email deliverability.

## Why these exist

Most automation portfolios show a screenshot and nothing you can actually run.
These are real, importable `.json` files. The external integration points that
would normally need a credential (SMTP email, CRM writes) are either left as
empty credential slots for you to fill, or stubbed as no-op nodes, so the logic
runs end to end with zero setup. The genuinely useful enrichment calls (DNS,
RDAP, disposable-domain checks) hit **free public APIs that need no key**.

## Workflows

| File | Nodes | What it does |
| --- | --- | --- |
| `OLDEV-Lead-Router-Enricher-Deduper.json` | 14 | Validates inbound leads, quarantines invalid/role-based addresses, enriches (company guess, free-mail flag), dedupes against an existing-contacts set, branches new vs returning, and round-robin assigns an owner. |
| `OLDEV-List-Hygiene-Deliverability-Preflight.json` | 13 | Scans an email list (syntax, role-based, disposable, duplicates), buckets contacts into clean / risky / suppress, and separately scores a sending domain on SPF / DKIM / DMARC. The front half of any CRM migration or bulk send. |
| `OLDEV-Inbound-Lead-Intelligence-Pipeline.json` | 24 | Webhook lead pipeline: validate, then five parallel live lookups (Google DNS-over-HTTPS for SPF/DMARC/MX, RDAP for domain age, Disify for disposable check), merge, score and classify into hot / warm / reject, write to CRM, and send tiered email alerts. Includes an Error Trigger branch and a CRM-failure branch. |
| `OLDEV-Support-Triage-Ops.json` | 42 | End-to-end helpdesk automation across labeled lanes. Per-ticket: validate, classify (topic/urgency/sentiment), enrich the sender domain, score and set an SLA, route to the right queue, persist, and send auto-ack + tiered owner/escalation emails. Plus a daily 7am sweep that fetches open tickets, flags SLA breaches, and emails a leadership digest and on-call escalation. |

## Status and validation

I want to be precise about what has and hasn't been verified, rather than imply
more than is true.

- **`OLDEV-Lead-Router-Enricher-Deduper`** and
  **`OLDEV-List-Hygiene-Deliverability-Preflight`** were built with the n8n
  workflow SDK, passed the SDK validator, and were **test-executed**, with their
  output data inspected to confirm the routing logic is correct.
- **`OLDEV-Inbound-Lead-Intelligence-Pipeline`** and
  **`OLDEV-Support-Triage-Ops`** are larger reference implementations. The JSON
  parses and the structure is complete, but they have **not yet been run inside
  n8n end to end**. Treat them as import-and-verify: bring them into your
  instance, pick credentials on the email/CRM nodes, and do a test run before
  relying on them.

All four files are valid JSON and import via **n8n -> Workflows -> Import from
File**.

## How to import and run

1. In n8n, go to **Workflows -> Import from File** and select a `.json` file.
2. Save the workflow. It stays inactive (manual or webhook trigger).
3. For workflows with email or CRM nodes, open those nodes and select a
   credential (for example your SMTP account on the `Email Send` nodes). The
   public-API HTTP nodes (DNS, RDAP, disposable) need no setup.
4. Click **Test workflow**. For webhook workflows, send a POST to the test URL,
   for example:
   ```json
   { "email": "someone@stripe.com", "name": "Test User", "source": "demo" }
   ```

## Public APIs used (no authentication required)

- **Google DNS-over-HTTPS** (`https://dns.google/resolve`) for SPF, DMARC, MX records
- **RDAP** (`https://rdap.org/domain/...`) for domain registration and age
- **Disify** (`https://disify.com/api/email/...`) for disposable and format checks

These calls are set to continue on error, so a slow or rate-limited lookup never
kills the run.

## Notes

- Sample data is synthetic. Email addresses and domains in the workflows are
  placeholders.
- The `oldev.site` addresses in notification nodes are example routing targets;
  change them to your own.
- These are templates and reference implementations, not a managed product.

## License

MIT. Use them, adapt them, learn from them.
