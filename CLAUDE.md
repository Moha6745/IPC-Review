# IPC Review

A single-file Arabic (RTL) web app for recording review comments on
contractor and consultant payment applications (IPCs). Everything lives in
`index.html` — markup, styles and logic — and Netlify serves it straight
from `main`, so a merge is a deploy. `IPC Review.html` is an older copy kept
for reference; do not edit it.

## Workflow

**تعليمات المالك: أي تعديل يطلبه — ادمج الـ PR بنفسك بعده مباشرة، بدون ما
ينتظر منه طلب الدمج.**

Standing instruction from the repo owner (Moha6745): after any change they
ask for, merge the pull request yourself as soon as the work is verified and
CI is clean. Do not wait to be asked, and do not leave the PR sitting as a
draft for them to merge.

The steps for each request:

1. Branch from the latest `main`, make the change, and verify it (below).
2. Push and open the pull request, so the change still has a reviewable
   record and a Netlify deploy preview.
3. Once the Netlify checks finish and nothing is failing, mark it ready and
   squash-merge it — that is the repo's convention, and it keeps `main` one
   commit per change.
4. Unsubscribe from the PR and drop any check-in you scheduled for it.

The owner can still say no: if they ask to hold a change, leave it unmerged.
Anything genuinely destructive (deleting data, rewriting history on `main`)
is not covered by this and still needs their word.

## Verifying a change

Nothing merges on hope — there is no human gate in front of these merges, so
the verification is the gate. Drive the real page in headless Chromium
(Playwright and Chromium are preinstalled; see `PLAYWRIGHT_BROWSERS_PATH`)
and exercise what you touched, rather than only reading the diff.

Two environment quirks to expect:

- The sandbox's network policy blocks cdnjs and gstatic, so ExcelJS,
  html2pdf and Firebase do not load. Serve a local copy of the library over
  a Playwright route when testing an export; the blocked-CDN console errors
  are expected and are not a bug in the page.
- Firebase being unreachable means the app falls back to `localStorage`.
  Seed reviews under the `ipc_review:<id>` keys plus the `ipc_meta:review-ids`
  index, then reload.

`index.html` uses **CRLF** line endings. Editing it with a script that
rewrites the file will silently convert it to LF and turn a small change
into a whole-file diff — read and write with `newline=''`, or convert back
before committing.

## How the app is put together

- **Two pages, one document.** A nav tab switches `.page` elements: the IPC
  review form, and a projects page that groups saved reviews by
  classification (contractor/consultant) → project → that project's IPCs.
- **Storage is three tiers**, tried in order: Firebase Firestore, Claude's
  `window.storage`, then `localStorage`. A cloud outage must never look like
  data loss — the saved index only ever grows as a union, and `deleteReview()`
  is the only place an id is removed.
- **Excel export** builds a worksheet per review through the shared
  `buildReviewSheet(wb, r, sheetName)`. The single-review, per-project and
  all-projects exports all go through it, so their formats cannot drift.
  Sheet names must stay within Excel's limits: 31 characters, none of
  `: \ / ? * [ ]`, and no duplicates.
- **Section lists** differ between contractor and consultant projects and are
  versioned by `SECTIONS_DEFAULTS_VERSION`; bump it when the baked-in
  defaults change so browsers pick the new list up once.
