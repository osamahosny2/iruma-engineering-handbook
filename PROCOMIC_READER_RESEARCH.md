# Manga Pro / ProComic — verified source research and implementation plan

Updated: 2026-11-10

## Why this work exists

The app previously showed chapter placeholders or no reader images. A source appearing in the source list is not proof that its search, content identity, chapter list, reader slots, image requests, and downloads are functional. The goal is real source data only; no synthetic chapter or cover-as-page fallback.

## Upstream source reviewed

Repository: [keiyoushi/extensions-source](https://github.com/keiyoushi/extensions-source)

- Parser: [`Procomic.kt`](https://github.com/keiyoushi/extensions-source/blob/4591fbc9bb571ad03aaea0f7428f7f403350e90e/src/ar/procomic/src/eu/kanade/tachiyomi/extension/ar/procomic/Procomic.kt)
- DTOs: [`Dto.kt`](https://github.com/keiyoushi/extensions-source/blob/4591fbc9bb571ad03aaea0f7428f7f403350e90e/src/ar/procomic/src/eu/kanade/tachiyomi/extension/ar/procomic/Dto.kt)
- Reader slots: [`slot.js`](https://github.com/keiyoushi/extensions-source/blob/4591fbc9bb571ad03aaea0f7428f7f403350e/src/ar/procomic/assets/slot.js)
- Turnstile helper: [`TurnstileHelper.kt`](https://github.com/keiyoushi/extensions-source/blob/d9420cd1c66ecd3385946b616aacfcf8f6314f70/core/src/main/kotlin/keiyoushi/utils/TurnstileHelper.kt)

The upstream ProComic extension is Apache License 2.0, while the app is GPL-3.0. Apache-2.0 is compatible with GPL-3.0. The app already retains the upstream attribution in THIRD_PARTY_NOTICES.md and includes LICENSES/Apache-2.0.txt. This is a direct adaptation, not a fork or shared upstream project.

## Verified catalog contract

- Search/listing: `GET /api/public/content` with `sort`, `limit`, `page`, and optional `search`.
- Details: `GET /api/public/content/{contentId}`.
- Chapters: `GET /api/public/{contentType}/{contentId}/chapters?limit=100&order=desc&page={n}`.
- Content identity must preserve the numeric content ID, content type, and slug. Do not identify a work by its title alone.
- Arabic chapter entries are selected by the API language field. Locked entries and text novels must not be presented as readable manga chapters.
- Listing requests need conservative rate limiting (at least five seconds between listing calls). Later pages/filters may require a short-lived Turnstile token.

## Verified reader contract

- Reader route: `/ar/chapter/{seriesSlug}-{chapterNumber}-{chapterId}`.
- The reader loads deferred media and lazy page slots. The DOM marker is `data-reader-image-row`; a page slot may return one URL or a `pieces` array with relative geometry.
- A chapter's page count must come from the live reader, not a fixed constant or a chapter DTO guess.
- Reader images are served from `*.procomic.im` under paths such as `/i/`, `/chapters/`, or `/api/cdn-image`.
- Some pages consist of multiple image pieces. The app must fetch each piece with the chapter Referer and stitch it into one ordered page bitmap.
- The WebView session should remain alive while the user moves through lazy slots. Captured image bytes should be shared with the app's downloader instead of assuming direct OkHttp requests will always work.
- The source may use `procomic.pro` or the `procomic.net` mirror. `safe_browsing=off` is a site cookie used by the upstream parser.
- 5xx reader responses should be retried with bounded reloads. Cloudflare/Turnstile must be surfaced as a real verification requirement; do not convert a blocked response into an empty chapter list.

## Current app implementation

Dedicated adapter: `app/src/main/java/com/example/connector/ProComicConnector.kt`.

Shared browser integration:
- `AndroidKotatsuWebViewExecutor.kt` owns the persistent WebView reader session and captures CDN image bytes.
- `KotatsuSourceEngine.kt` exposes the reader session and existing browser-verification coordinator.
- `ConnectorEngine.kt` routes ProComic image downloads to the dedicated adapter.

Current commits:
- `8ff18f1b20e1213ac03b2ae62e869d3782651134` — API and lazy reader integration.
- `5211f99d58d4d7c28a26f2b48cdff1e5b2f020b5` — missing `IOException` import found by CI.

Verification:
- CI run `42895363904` failed only at Kotlin compilation because `java.io.IOException` was not imported.
- CI run `42896185574` is the follow-up run for the import fix; check its final conclusion before proceeding.
- A green CI build does not prove live website access. Phone testing is still required.

## Remaining work, in order

1. Confirm CI run `42896185574` passes unit tests and APK build.
2. Preserve upstream attribution in the app's `THIRD_PARTY_NOTICES.md`.
3. Add the five-second listing throttle, mirror fallback and `safe_browsing=off` handling if not yet present in the committed branch.
4. Implement/cache Turnstile token acquisition for later listing pages and handle interactive verification without hanging a background WebView.
5. Verify the current chapter reader DOM and image CDN paths against a live Manga Pro chapter.
6. Test old and new image URL formats, single-image pages, composite pages, 5xx retry, cancellation, and cookie/session reuse.
7. Add focused unit tests for API DTO mapping, chapter identity, slot descriptor parsing, trusted image URL validation, and composite geometry.
8. On the phone, test search → details → Arabic chapters → first page → later page → download → offline open. Only then mark Manga Pro production-ready.

## Explicit non-goals

- Do not fabricate chapters, page counts, page URLs, or covers-as-pages.
- Do not silently fall back to the old static HTML parser when the API returns invalid data.
- Do not fork or push changes to the upstream parser repository.
- Do not claim the source is fixed merely because GitHub Actions passes.


## 2026-11-10 — live reader-slot protocol correction

- Re-read the current upstream Procomic.kt, Dto.kt, and assets/slot.js from keiyoushi/extensions-source at commit d9420cd1c66ecd3385946b616aacfcf8f6314f70. The reader script is an index-based slot function, not a generic image selector.
- The slot function waits for /chapter-deferred-media/, locates [data-reader-image-row], scrolls each slot into view, checks image visibility/completion, and returns either one CDN URL or multiple pieces with relative geometry.
- The slot may signal an interactive Turnstile state. The app maps this to the existing browser-verification flow; a failed 5xx/status row is an error/retry, not an empty successful page list.
- The app already has a persistent ProComicReaderSession in AndroidKotatsuWebViewExecutor.kt; KotatsuSourceEngine.kt exposes page-count, page-descriptor and captured-image-byte operations. ConnectorEngine.kt delegates ProComic downloads to ProComicConnector. Keep this single session architecture; do not add a second hidden WebView implementation.
- Commit d560fc54eff633f89d171c51c78dcbc7fbbc5ecd aligns the slot function with the upstream protocol and uses latest_chapter ordering with a 12-item page for text searches.
- CI run 42896444353 for that commit is currently running. At the last check, setup had passed and Run unit tests was in progress; APK build and artifact upload had not started.
- This is code-level progress only. No live Manga Pro search/chapter/page test on the phone has been performed.

### Updated remaining work

1. Confirm CI run 42896444353 completes unit tests and APK build; inspect any compile/test failure before proceeding.
2. Add focused tests for content identity, Arabic/locked chapter filtering, slot descriptor parsing, trusted CDN validation, and composite-page geometry.
3. Implement the upstream short-lived Turnstile token flow for later listing pages/filters; the five-second rate limit alone does not provide this token.
4. Verify safe_browsing=off, mirror fallback between procomic.pro and procomic.net, and browser cookie/session reuse against the live site.
5. On-device test search → details → Arabic chapters → first and later pages → composite stitching → download → offline open.
6. Do not mark Manga Pro or task 3 complete until the real phone test passes.


## 2026-11-10 — compile correction and regression tests

- Commit a718ee66cf0a27243891c8766c546c6aad6add2c added explicit ProComic chapter DTO parsing tests, safe_browsing cookie preparation, browser-cookie reuse for API/image requests, and bounded composite-page reconstruction.
- The first compile attempt exposed two Kotlin errors: Android RectF exposes width() and height() methods, not width/height properties. The exact lines in ProComicConnector.kt were corrected in commit 8db92b23a10ec010a65543069a6cb80f4e0bc142.
- The current branch head is 8db92b23a10ec010a65543069a6cb80f4e0bc142. Its CI run is 42897158863; Android/Gradle setup passed and unit tests were running at the last check.
- Previous run 42896328650 failed at compileDebugKotlin due to those RectF calls. Do not treat that run as a passing test.
- Turnstile token generation remains a separate required follow-up for API listing pages that return 403; the upstream helper requires a foreground-capable WebView interaction flow and must be integrated deliberately, not mistaken for ordinary Cloudflare cookie verification.
