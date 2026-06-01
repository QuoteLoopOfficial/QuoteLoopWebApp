# Quote Loop Web App

Single-file static web app for Quote Loop — sign-in, sign-up, onboarding, dashboard, and admin surfaces. Deployed on Vercel.

## Files

- `index.html` — the entire app (HTML + inline CSS + inline JS).
- `.gitignore` — macOS / editor / Vercel-local guards. **`*.p8` excluded** so Apple Sign-in private keys can't be committed.

No `package.json`, no build step, no separate `.css`/`.js`. Pure static. Vercel serves the file as-is.

## Architecture notes

- All Supabase calls are hand-rolled `fetch()` against PostgREST. The anon key is inlined; RLS protects rows server-side.
- Page routing is a JS-controlled `showPage()` mechanism. Legal-page routes use real hash URLs (`#/tos`, `#/privacy`, …) — but those legal containers were dropped during the lift since the standalone files at `quoteloop.com.au/tos.html` etc. remain the canonical sources.
- Webapp routes (`signin`, `signup`, `dashboard`, `admin`, `onboarding`, `app`, `reset-password`, `subscription-required`) are NOT hash-routed today — they're only reachable via in-app `showPage()` calls. Re-add hash routing for these later if you want deep-linkable webapp URLs (e.g. `xyz/#/signin`).

## Compatibility stubs

`index.html` includes three hidden DOM stubs at the top of `<body>`:

```html
<div id="mainSite" style="display:none"></div>
<nav id="nav" style="display:none"><div id="nav-auth-btns"></div></nav>
```

These exist because the lifted JS makes unguarded references to landing-page elements (`#mainSite`, `#nav`, `#nav-auth-btns`). Without the stubs, `showPage()`, the scroll handler, and `updateNavForAuth()` would throw `TypeError: Cannot set properties of null` on cold load / scroll / sign-in respectively. The stubs let those calls succeed with no visible effect. **Remove the stubs once the JS is refactored to drop the landing dependencies** (or to add `if (x) …` null-guards around the three call sites).

## Cutover-domain TODOs

Two hardcoded `https://quoteloop.com.au` URLs remain in the JS — Apple OAuth and password-reset email redirect. Both are marked with:

```js
// TODO: cutover-domain — update to quoteloop.xyz
```

Grep for `TODO: cutover-domain` to find them. Update both before serving the app on the new domain. The Google OAuth flow uses `window.location.origin` and is domain-portable — no update needed there.

## Diagnostic error handler

The first `<script>` in `<head>` is a global error handler that surfaces unguarded-DOM crashes loudly during early testing:

```js
window.addEventListener('error', function (e) {
  if (e && e.message && /Cannot (read|set) (property|properties) of null/.test(e.message)) {
    console.error('[QuoteLoop WebApp] Unguarded DOM reference:', e.message, 'at', e.filename + ':' + e.lineno);
  }
});
```

Remove or downgrade once the webapp is stable and known-clean.

## Provenance

Initial lift on 2026-06-01 from `QuoteLoopOfficial/quoteloop-website` `index.html`:

| Source | Lifted to |
|---|---|
| Lines 1598–8212 (inline `<script>` body) | Inline `<script>` in this `index.html` |
| Lines 8217–10156 (webapp page containers) | `<body>` after stubs |
| Lines 14–20, 23–38, 41–46, 485–486 (head CSS subset) | Custom `<style>` block in `<head>` |
| Lines 2–10 (head meta + favicons + fonts) | Verbatim in `<head>`, title/description updated |

Landing-only surfaces (lines 579–1195: hero/features/pricing/etc., and embedded legal pages 1196–1596) intentionally NOT lifted — stay on `quoteloop-website`.
