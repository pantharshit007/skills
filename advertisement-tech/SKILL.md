---
name: advertisement-tech
description: >
  Expert in Advertisement Technology (AdTech), programmatic advertising, yield management,
  ad serving architecture, and publisher monetization. Use this skill whenever the user
  mentions anything related to: ad servers, GAM, Google Ad Manager, ADX, Ad Exchange,
  header bidding, Prebid.js, SSP, DSP, RTB, CPM, floor prices, line items, ad units,
  trafficking, creatives, viewability, IVT, CTV, VAST/VPAID, identity solutions, cookie
  deprecation, first-party data, ad quality, brand safety, supply path optimization (SPO),
  bid shading, ad ops, DFP, yield optimization, programmatic direct, PMPs, deal IDs,
  or any code involving `googletag`, `pbjs`, `apstag`, or ad-related JavaScript. Even if
  the user doesn't use these exact terms — if the task touches ad serving, monetization,
  or a publisher/advertiser workflow, use this skill.
---

# SKILL: AdTech (Advertisement Technology) & Programmatic Ecosystem

## 1. Skill Overview
You are an expert in Advertisement Technology (AdTech), specializing in programmatic advertising, yield management, ad serving architecture, and publisher monetization. Your primary function is to understand, debug, and write code related to the full publisher monetization stack — including Google Ad Manager (GAM), Google Ad Exchange (ADX), Header Bidding, Prebid.js, identity solutions, and CTV/video advertising.

---

## 2. The Core AdTech Supply Chain
Always keep this mental model when analyzing or generating AdTech architecture:

```
Advertiser (Brand) → Agency → DSP → Ad Exchange/SSP → Publisher Ad Server (GAM) → User's Browser/App
```

- **Advertiser (Demand):** Wants to buy impressions to show creatives. Uses a DSP.
- **Publisher (Supply):** Owns inventory (website/app/CTV). Uses an ad server (GAM) + SSPs.
- **The Transaction:** Happens via RTB in ~100ms before a webpage renders.
- **The Money Flow:** Advertiser pays CPM → DSP takes margin → Exchange/SSP takes margin → Publisher receives net revenue.

---

## 3. Key Terminology & Definitions

### Platforms & Infrastructure
| Term | Definition |
|---|---|
| **GAM (Google Ad Manager)** | Primary ad server for publishers. Decides which ad wins: direct-sold campaign, programmatic, or house ad. Frontend: Google Publisher Tags (GPT). |
| **ADX (Google Ad Exchange)** | Google's programmatic marketplace embedded within GAM. Acts as an SSP/Exchange for real-time bidding. Distinct from GAM itself. |
| **DFP (DoubleClick for Publishers)** | Legacy name for GAM — still seen in older codebases. Same product. |
| **DSP (Demand-Side Platform)** | Advertiser-side buying platform. Examples: DV360 (Google), The Trade Desk, Xandr, MediaMath. |
| **SSP (Supply-Side Platform)** | Publisher-side selling platform. Examples: AppNexus/Xandr, Magnite (Rubicon+Telaria), OpenX, PubMatic, Index Exchange, Triplelift. |
| **Ad Network** | An intermediary that aggregates publisher inventory and resells it. Not an exchange — no real-time auction. Often remnant-focused. |
| **Ad Exchange** | A real-time marketplace where SSPs and DSPs connect for RTB auctions. Do NOT confuse with ad networks. |
| **CDP (Customer Data Platform)** | First-party data management tool for publishers/advertisers. Used for audience segmentation and identity resolution. |
| **DMP (Data Management Platform)** | Third-party data aggregation for audience targeting. Becoming less relevant post-cookie deprecation. |

### Auction & Pricing
| Term | Definition |
|---|---|
| **RTB (Real-Time Bidding)** | Per-impression auction mechanism conducted in ~100ms via OpenRTB protocol. |
| **CPM (Cost Per Mille)** | Cost per 1,000 impressions. Core pricing metric. |
| **eCPM (Effective CPM)** | Revenue per 1,000 impressions actually earned (accounts for fill rate). |
| **Floor Price** | Minimum CPM a publisher will accept. Can be hard (impression rejected below floor) or soft (bid below floor can still win at floor price). |
| **Bid Shading** | Technique in first-price auctions where buyers bid slightly below their true max to avoid overpaying. |
| **First-Price Auction** | Winning bidder pays exactly what they bid. Industry standard since ~2019. |
| **Second-Price Auction** | Winner pays $0.01 above the second-highest bid. Legacy model, mostly replaced. |
| **Deal ID / PMP (Private Marketplace)** | A curated, invite-only programmatic auction between a publisher and select buyers. Higher CPMs, guaranteed inventory access. |
| **Programmatic Guaranteed (PG)** | Automated direct deal with fixed CPM + guaranteed impressions. Combines direct-sold certainty with programmatic automation. |
| **Preferred Deal** | Non-guaranteed programmatic deal at a fixed CPM. Buyer gets first look but no impression guarantee. |

### Inventory & Ad Units
| Term | Definition |
|---|---|
| **Ad Unit** | A defined slot on the page where ads serve (e.g., `/1234/homepage_leaderboard`). Defined in GAM and GPT. |
| **Inventory** | All available ad impressions across a publisher's properties. |
| **Line Item** | A trafficking entity in GAM representing a campaign booking. Has a priority type, CPM/CPC, targeting, and associated creatives. |
| **Creative** | The actual ad asset (HTML, image, video, VAST tag) attached to a line item. |
| **Key-Values (KVs)** | Custom targeting parameters passed to GAM. Used by Prebid (e.g., `hb_pb=1.50`, `hb_bidder=appnexus`) for programmatic line item targeting. |
| **AdSlot** | The specific page-level instance of an ad unit. Defined via `googletag.defineSlot()`. |

---

## 4. Yield Management Strategies

### The Waterfall (Legacy)
Publishers called ad networks sequentially by priority/historical CPM. Low competition, poor yield. Largely replaced by header bidding.

### Header Bidding (Client-Side)
Publishers simultaneously auction inventory to multiple SSPs *before* the GAM ad call.
- JS runs in browser `<head>`, collects bids from all SSPs
- Winning bid is passed as a key-value to GAM
- GAM compares against its own demand (ADX, direct) and picks the highest
- **Pro:** Maximum competition, high yield
- **Con:** Browser latency, client-side complexity, blocks page render if not async

### Server-Side Header Bidding (S2S)
Prebid Server (open-source, self-hosted or cloud) receives bid requests from the browser and fans out to SSPs server-to-server.
- **Pro:** Reduces browser latency, supports more bidders
- **Con:** Cookie/ID syncing harder, less accurate user data
- Common providers: Prebid Server (AppNexus-hosted), Index Exchange, Magnite

### Amazon TAM (Transparent Ad Marketplace)
Amazon's server-side header bidding solution. Works alongside Prebid.
- Uses `apstag` library on the frontend
- Must be integrated carefully to avoid race conditions with Prebid

---

## 5. Prebid.js Deep Dive

### Core Architecture
```
Page Load
  → pbjs.addAdUnits()         // Define ad units + bidder configs
  → pbjs.requestBids()        // Triggers simultaneous SSP auctions
      → Bidder responses (or timeout)
  → pbjs.setTargetingForGPTAsync()  // Writes hb_* KVs to GPT slots
  → googletag.pubads().refresh()    // GAM ad call fires with Prebid KVs
  → GAM evaluates: direct vs ADX vs hb_pb line item → winner serves
```

### Key `window.pbjs` Methods
| Method | Purpose |
|---|---|
| `pbjs.addAdUnits(adUnits)` | Registers ad slots and their SSP/bidder configs |
| `pbjs.requestBids({bidsBackHandler, timeout})` | Fires auction; callback triggers after bids or timeout |
| `pbjs.setTargetingForGPTAsync()` | Critical handoff: writes `hb_pb`, `hb_bidder`, etc. to GPT slots |
| `pbjs.getBidResponses()` | Debug: inspect all bid responses |
| `pbjs.getHighestCpmBids()` | Debug: see winning bids per slot |
| `pbjs.onEvent('bidWon', fn)` | Event: fires when a Prebid bid actually wins and serves |

### Standard Prebid Key-Values (KVs) in GAM
| Key | Example Value | Meaning |
|---|---|---|
| `hb_pb` | `1.50` | Price bucket of winning bid |
| `hb_bidder` | `appnexus` | SSP that won the Prebid auction |
| `hb_adid` | `abc123` | Ad ID for creative retrieval |
| `hb_size` | `300x250` | Creative size |
| `hb_format` | `banner` / `video` | Format type |
| `hb_source` | `client` / `s2s` | Client-side or server-side bidding |
| `hb_deal` | `deal_id_123` | If a PMP deal, deal ID |

### Price Granularity
Controls how bid prices map to `hb_pb` buckets in GAM. Prebid built-ins:
- `low`: $0.50 increments up to $5
- `medium`: $0.10 increments up to $20 (most common)
- `high`: $0.01 increments up to $20
- `auto`: Variable increments (recommended)
- `dense`: Finer increments in low ranges
- **Custom:** Always prefer `auto` or a custom config for maximum yield accuracy

### Bidder Adapters
Prebid has 300+ SSP adapters. Common ones:
`appnexus`, `rubicon` (now `rubicon`/Magnite), `openx`, `pubmatic`, `ix` (Index Exchange), `triplelift`, `criteo`, `sovrn`, `teads`, `33across`, `improvedigital`

---

## 6. Google Publisher Tags (GPT) Deep Dive

### Key `window.googletag` Patterns
```javascript
googletag.cmd.push(function() {
  // Define the slot
  var slot = googletag.defineSlot('/network_id/ad_unit_path', [[300,250],[728,90]], 'div-id')
    .addService(googletag.pubads());

  // Set page-level targeting
  googletag.pubads().setTargeting('section', 'homepage');

  // Enable features
  googletag.pubads().enableLazyLoad();       // Lazy loading
  googletag.pubads().disableInitialLoad();   // Critical for Prebid: prevents auto-fetch
  googletag.pubads().collapseEmptyDivs();

  googletag.enableServices();
});

// Display (after Prebid bids are in)
googletag.cmd.push(function() { googletag.display('div-id'); });
```

### `disableInitialLoad()` — Critical for Header Bidding
Must be called when using Prebid. Prevents GPT from fetching ads before Prebid bids are set. Without this, GAM calls fire before `hb_*` targeting is applied → Prebid bids are ignored.

### Lazy Loading
`googletag.pubads().enableLazyLoad({ fetchMarginPercent: 200, renderMarginPercent: 50 })` — Fetches/renders ads only when near viewport. Improves page speed but can reduce above-fold fill rate.

---

## 7. Identity Solutions & Cookie Deprecation

Third-party cookies deprecated in most browsers (Firefox/Safari already; Chrome phase-out ongoing). This affects audience targeting and frequency capping.

### Key Solutions
| Solution | Type | Notes |
|---|---|---|
| **UID2.0 / EUID** | Industry (Trade Desk) | Email-based encrypted ID. Publisher passes hashed email; DSPs match. |
| **LiveRamp ATS** | Identity graph | Authenticated Traffic Solution. Maps first-party logins to RampIDs. |
| **ID5** | Shared ID | Probabilistic + deterministic ID shared across publishers. |
| **Unified ID 2.0 in Prebid** | `userid` module | `pbjs.setConfig({ userSync: { userIds: [...] } })` |
| **Google PPID** | Publisher-provided ID | Publisher sends their own user ID to GAM for frequency capping + targeting. |
| **Privacy Sandbox / Topics API** | Google | Browser-based interest topics replace cookies. Still in flux. |
| **First-Party Data** | Publisher-owned | Logged-in users, newsletter subscribers. Most durable signal. |

---

## 8. Video Advertising

### VAST (Video Ad Serving Template)
XML-based standard for video ads. Publisher's video player requests a VAST tag → gets XML describing the creative (media file URL, tracking events, duration).
- **VAST 2.0/3.0/4.x**: Progressively adds skippable ads, pods, verification
- **VPAID**: JS-based interactive video wrapper (being deprecated, security concerns)
- **SIMID**: VPAID replacement; more secure interactive video standard

### Video Formats
| Format | Context |
|---|---|
| **Pre-roll** | Before video content |
| **Mid-roll** | During video content (e.g., YouTube-style) |
| **Outstream** | Standalone video in display ad unit; no video content required |
| **Rewarded Video** | User opts in for a reward (common in mobile games) |
| **CTV/OTT** | Connected TV (Roku, Fire TV, etc.) — high CPMs, no cookies, device ID-based |

---

## 9. Measurement & Quality

### Key Metrics
| Metric | Definition |
|---|---|
| **Viewability** | % of impressions where ≥50% of ad pixels were in-view for ≥1 second (IAB standard). MRC-accredited. |
| **IVT (Invalid Traffic)** | Bot traffic, click fraud, ad stacking. SIVT = sophisticated, GIVT = general. |
| **Fill Rate** | % of ad requests that returned an ad. Low fill = lost revenue. |
| **Win Rate** | % of auctions a bidder wins. |
| **Bid Rate** | % of auction requests a bidder responds to. |
| **Time-to-First-Byte (TTFB)** | Latency metric for ad server responses. |
| **RPM (Revenue Per Mille)** | Publisher revenue per 1,000 page views (not impressions). |

### Brand Safety & Ads.txt
- **ads.txt (Authorized Digital Sellers)**: A text file at `domain.com/ads.txt` listing all authorized SSPs/exchanges. Prevents unauthorized reselling.
- **sellers.json**: SSP-side complement to ads.txt. Lists all publishers they represent.
- **SupplyChain Object (schain)**: OpenRTB field tracing every intermediary in the supply path.
- **Brand Safety**: Preventing ads from appearing next to harmful content. Tools: IAS, DoubleVerify (DV), MOAT.

---

## 10. Supply Path Optimization (SPO)
Buyers increasingly scrutinize supply paths to reduce fee layers.
- DSPs prefer direct publisher-SSP connections over reseller chains
- SPO means consolidating SSP relationships to paths with better data, fewer hops, and lower fees
- `ads.txt` `RESELLER` vs `DIRECT` status matters here

---

## 11. Codebase Interaction Guidelines

### Common Frontend Patterns to Recognize
```javascript
// GPT + Prebid integration (correct pattern)
googletag.cmd.push(() => googletag.pubads().disableInitialLoad());

pbjs.que.push(() => {
  pbjs.addAdUnits(adUnits);
  pbjs.requestBids({
    timeout: 1500,  // ms — tune based on latency vs yield tradeoff
    bidsBackHandler: () => {
      pbjs.setTargetingForGPTAsync();
      googletag.pubads().refresh();
    }
  });
});

// Amazon TAM (apstag) alongside Prebid
apstag.fetchBids({ slots: [...], timeout: 2e3 }, (bids) => {
  apstag.setDisplayBids();
  // coordinate with Prebid callback before refresh
});
```

### Debugging Checklist
1. **Ad not serving?** Check `?cust_params=` in GAM network request URL. Verify `hb_pb` and `hb_bidder` are present.
2. **Prebid bids not reaching GAM?** Confirm `disableInitialLoad()` is called before `enableServices()`. Confirm `setTargetingForGPTAsync()` runs before `refresh()`.
3. **Race condition?** Add `pbjs.onEvent('auctionEnd')` logging. Ensure `refresh()` only fires inside `bidsBackHandler` or after a timeout.
4. **Low bids / no bids?** Check `pbjs.getBidResponses()` in console. Look for bidder errors, mismatched sizes, or floor rejections.
5. **Floor filtering bids?** Check `pbjs.getConfig('priceFloors')` and ensure floor config isn't too aggressive.
6. **Line item not matching?** In GAM, verify price bucket line items exist for the CPM range returned. Check KV targeting on line items matches `hb_pb` values exactly.
7. **Video ads not serving?** Validate VAST XML with Google's VAST Inspector. Check player CORS settings.

### Ad Ops / GAM Trafficking Concepts
- **Line Item Priority Types** (highest to lowest): Sponsorship → Standard → Network → Bulk → Price Priority → House
- **Price Priority line items** are used for Prebid/programmatic — GAM competes them against ADX using the `hb_pb` CPM.
- **Creatives for Prebid:** Use Prebid's universal creative snippet (`<script>...</script>`) — it calls back to Prebid to render the actual SSP creative.

---

## 12. Operational Directives

1. **Never confuse GAM (ad server) with ADX (exchange inside GAM).** They are distinct products sharing an interface.
2. **Never confuse Ad Networks with Ad Exchanges.** Networks aggregate and resell; exchanges run real-time auctions.
3. **Always consider timeout tuning** when Prebid yield is discussed. Lower timeout = faster page, fewer bids. Higher timeout = more bids, slower page.
4. **First-price auction is the standard.** Don't assume second-price semantics unless explicitly dealing with a legacy system.
5. **Cookie-based targeting is unreliable.** When discussing audience targeting, always surface first-party data and identity solutions as the durable approach.
6. **Floor prices have downstream effects.** Raising floors increases eCPM but lowers fill rate — always frame this tradeoff.
7. **ads.txt must be maintained.** Any new SSP relationship requires adding a DIRECT or RESELLER entry.
