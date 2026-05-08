# Bro — current status

| Field           | Value                                                    |
| --------------- | -------------------------------------------------------- |
| Generated       | 2026-05-08                                               |
| Last app deploy | Verified live at generation time (see Live state below). |
| Maintainer      | mike@impt.io                                             |

This file is the single, dated snapshot of where the project is **today**.
Replaced when material changes ship; superseded versions live in git history.

---

## 1. Live state

| Endpoint            | Status         | Detail                                                                  |
| ------------------- | -------------- | ----------------------------------------------------------------------- |
| `broai.ai`          | 🟢 LIVE        | Marketing single-page site. Loads (HTTP 200). No sub-routes.            |
| `api.broai.ai`      | 🟢 LIVE        | Application (Next.js). `/login`, `/signup`, `/chat`, `/about`, `/privacy`, `/terms` all responding. |
| `api.broai.ai/api/health` | 🟢 200    | Returns `{"status":"alive","service":"BROAI API","message":"Bro is here."}` |
| `ai.broai.ai`       | 🔴 DOWN        | Memory-ingest worker. Connection times out. Pending restart.            |

## 2. What's working end-to-end

- Marketing site is live, indexable for the home URL.
- Application loads on the public URL with auth/login screen, chat surface, legal pages.
- API health probe responds.
- Domain + TLS chain healthy on `broai.ai` and `api.broai.ai`.

## 3. What's not working / open work

These are the items currently blocking a polished public release.

- **Chat error path.** Some chat calls fail under conditions that need to be reproduced and traced. Tracking under "chat reliability".
- **Voice auto-play.** Voice responses do not auto-play on first
  load on some browsers; user has to tap to start. Browser
  autoplay-policy work needed.
- **Avatar.** Visual avatar (still or animated) is not yet in the
  conversation surface. Design pass pending.
- **Sign-in friction.** Sign-in flow has known friction; auth UX pass
  needed before broader testing.
- **`ai.broai.ai` offline.** The memory-ingest worker is down. New
  uploads cannot be processed until the worker is restored. Mitigation
  in progress.
- **Mobile testing coverage** — manual passes on iOS Safari + Android
  Chrome are incomplete.
- **SEO surface.** Marketing site has no `/sitemap.xml`, no
  `/robots.txt`, no sub-pages indexed. Appropriate for an early
  private-launch posture; will expand when the product is broadly open.
- **API redirect on `/health`** returns 307; the documented health
  endpoint is `/api/health` (which works). Intermediate redirect to be
  cleaned up.

## 4. Architecture summary (operational)

```
                ┌───────────────────────┐
   visitor  ──► │  broai.ai (marketing) │
                └───────────────────────┘
                            │
                            ▼  (sign in / try)
                ┌───────────────────────┐
                │  api.broai.ai (app)   │  Next.js · auth · chat ·
                │                       │  voice playback · avatar
                └───────────┬───────────┘
                            │
                            ▼  (memory ingest)
                ┌───────────────────────┐
                │  ai.broai.ai (worker) │  ⚠ currently OFFLINE
                └───────────────────────┘
```

Backing services (operational notes only — credentials and IDs not
recorded in this public file):

- **Voice synthesis** via ElevenLabs, model trained on consented source
  material from the namesake (~hundreds of clips). Voice ID held in
  internal config only.
- **Conversational model** is a Claude Sonnet variant.
- **Embeddings** for memory recall via `text-embedding-3-large`.
- **Datastore** is a vector-capable database backing the memory recall
  path; not exposed publicly.

## 5. Privacy posture

Bro is **privacy-first by design**:

- Stored memories are accessible only to the account that uploaded them.
- The voice model is trained on **consented** source material; it is
  not a generic-celebrity-voice product and will never operate that way.
- No third-party advertising trackers on either surface.
- Production datastores are restricted-access; no offshore replication
  outside contractual jurisdictions.
- Account deletion removes user-uploaded content; voice models trained
  per-account are isolated and removed on deletion.

A formal Privacy Policy is published at `api.broai.ai/privacy` (HTTP 200, ~26 KB).

## 6. Source code

The application source lives in **private repositories owned by the
build team** (the engineer leading delivery). This repository
(`IMPTio/broai`) is the public front door only — README, this status
file, brand assets, security policy.

Code-level access is granted via the maintainers when there is a
specific need (security review, partner integration, etc.).

## 7. Roadmap (near-term)

In priority order, near-term:

1. **Restore `ai.broai.ai`** ingest worker.
2. **Chat reliability fix** — reproduce and patch the chat error path.
3. **Voice auto-play** — comply with browser autoplay policies via a
   one-tap "start" gesture; persist user opt-in.
4. **Avatar** — surface a still avatar in the conversation; animated
   avatar deferred.
5. **Sign-in UX pass** — reduce friction; add passwordless option.
6. **Mobile QA pass** — iOS Safari + Android Chrome end-to-end.
7. **SEO surface** for `broai.ai` (sitemap, sub-pages).

Mid-term:

- Public bug-bounty programme.
- Third-party privacy review.
- Family-account sharing model (with consent constraints).

## 8. Mission note

Bro is built in memory of **Bronagh English** (24 April 2025, aged 18).
Every product decision goes through one filter: would the family of
someone who is grieving feel safe using this? When that filter and a
"growth" filter conflict, the grief filter wins.

---

*Status maintained by mike@impt.io. Operational issues:
ops@broai.ai. Security disclosures: security@broai.ai.*
