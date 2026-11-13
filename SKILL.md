---
name: spywinads-store-spy
description: See which Facebook / Instagram video ads a Shopify or online store is running right now, with each video's real age (the day the file first hit Meta) and how many other stores run the same video, plus whether the store is scaling or testing. Use whenever the user gives a store domain or store link and asks what ads it runs: "spy on this store", "what ads is shop.com running", "is this store scaling", "competitor store ads", "soi ads store", "store này chạy ads gì", "shop này đang scale không" — even if SpyWinAds is not mentioned.
compatibility: Needs the SpyWinAds MCP connection (https://spywinads.com/mcp, free account, 30 credits a day). Runs on Claude.ai, Claude Code, ChatGPT (Plus and up), Cursor, Codex.
license: MIT
---

# Shopify store spy — see a store's Facebook video ads

Data comes from the **SpyWinAds MCP**: a library of Meta video ads shown in the US, scanned by SpyWinAds users. Each video is fingerprinted, so the same video running in many stores is recognised as one.

## 0. Check the connection first

No tool named `spywin_*` available → the user has not connected SpyWinAds yet. Do not guess numbers or analyse from memory: the user
will trust those numbers. Give them the setup steps for where they are (section **Setup** at the end), and offer the free web tool
https://spywinads.com/shopify-store-spy meanwhile. Wait until they are connected.

Each tool call uses 1 credit (free: 30 a day, paid plans: 100), so call the right tool once per input. If a tool says out of credits or
not signed in, repeat that message and link `https://spywinads.com/developers`.

Answer in the language the user writes in, including table headers and the report lines below (they are written in English only
as a template).

## 1. Look up the store

Input: a domain (`shop.com`) or any link on the store → strip it to the domain. Several stores → one call per store, max 5 unless the
user asks for more.

Call `spywin_spy_store_ads(domain)`. You get live ads, new ads in the last 7 days, number of videos, products and Pages, and the top
videos with real age and stores per video.

`found: false` → if `spywin_queue_page_scan` exists, ask one short question and call it with `kind="store"`, follow with
`spywin_get_scan_status(request_id=…)`, then look up again. Otherwise give the free scan link the tool returns.

## 2. Status of the store

Pick **one** current status from `live_ads`, `new_ads_7_days`, `videos`, `products`, first match wins, and show the numbers:
1. **Paused**: `live_ads` = 0. Then describe how it ran before (videos per product, first ad in the library) and say it may be
   paused, not gone; the tool does not say when it stopped.
2. **Pushing hard**: `live_ads` ≥ 20 and `new_ads_7_days` ≥ 5.
3. **Slowing down**: `new_ads_7_days` = 0.
4. **Testing**: anything else, typically many products with 1–2 videos each.
Say it is a reading of ad activity, not of sales.

## 3. Report

**Summary** (2–3 sentences): what the store pushes, status, how fresh.

| # | Product | Video age | Stores running now / ever (all stores) | Live ads here | Why it is worth a look |
|---|---|---|---|---|---|

**Next step** — offer in plain words, and if the user picks one, call the tool yourself:
- the other stores running the top video → `spywin_lookup_video_ad(video_id=…)`. If the tool only accepts `ad` (older connector),
  ask for that video's Ad Library link, or tell the user to reconnect SpyWinAds to get the newer tool;
- copy the top product to a Shopify CSV → `spywin_export_shopify_product(landing_page)`.

## Read the numbers correctly

- **Real video age** = `video_first_on_meta`, the day the video file was first uploaded to Meta. Show it as date + days, counted
  from today: "22 Sep (52 days)". The ad start date resets each time a store pauses and restarts an ad; the file date does not, so
  say "video age", not "ad running for". `video_first_on_meta` empty → the file date is unknown: give `first_ad` and say it is the
  first ad date, not the video age.
- **Scope**: `live_ads` counts only the store / Page / product you looked up (whole library for a single-video lookup).
  `stores_running_now`, `stores_ever` and `pages_running_now` count **every store in the library**. So a paused store (0 live ads)
  can show a video that 4 other stores still run: say exactly that ("paused here, still running in 4 other stores"), it is not a
  contradiction.
- A product name that looks like a code (`ymtldysdym1`) is the store's URL handle: say the name is not readable and give the
  landing page. Same product with several videos → one row, "2 videos".
- Group brands by the **landing-page domain**, never by ad copy. Two stores with the same hook are still two brands.
- The library only sees a **minimum count of ads observed**: no spend, reach, orders or revenue. Many ads / many stores / a long run
  means *being invested in or copied*, not *profitable*. Say "worth a look because…", never "winning", "profitable", "making sales".
- Data = Meta ads shown in the US, scanned by SpyWinAds users. "Earliest store" = earliest in the library, not proven owner.

## Limits

This skill only reads SpyWinAds data: it does not scrape Facebook and never logs in to Facebook for the user (against Meta's terms, and it gets accounts locked).

## Setup (give this to users who are not connected)

Sign-up happens at the login step: free, 30 credits a day, no card.

- **Claude.ai** (every plan incl. Free, 1 custom connector): Customize → Connectors → + → Add custom connector → URL
  `https://spywinads.com/mcp` → Connect → sign in → Allow.
- **ChatGPT** (Plus / Pro / Business; Free cannot add custom MCP yet, suggest Claude.ai Free): Settings → Apps & Connectors → Advanced →
  Developer mode → Create → URL `https://spywinads.com/mcp`, OAuth → sign in → Allow.
- **Claude Code**: `claude mcp add --transport http spywinads https://spywinads.com/mcp`, then `/mcp` to sign in.
- **Cursor / Codex / scripts**: API key at `https://spywinads.com/developers`, header `Authorization: Bearer <key>` (never in the URL).

Want the full analysis (several stores, ad angles, Shopify CSV in one go)? Skill **spywinads-soi-ads**:
https://github.com/SpyWinAds/soi-ads-skill
