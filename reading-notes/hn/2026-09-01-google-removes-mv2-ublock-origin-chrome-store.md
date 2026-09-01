# 1. Title

Google Removes Manifest V2 Extensions from the Chrome Web Store, Including uBlock Origin

# 2. Source

- Author / Organization: Web Iterate
- Link: https://webiterate.dev/google-removed-extensions-ublock-origin-108/
- Date: 2026-09-01

# 3. One-line Summary

- Google completed its Manifest V2 phase-out by removing remaining MV2 extensions, including uBlock Origin, from the Chrome Web Store, shifting Chromium extensions toward the more restrictive Manifest V3 model.

# 4. Key Points

- Google removed all remaining Manifest V2 (MV2) extensions from the Chrome Web Store.
- uBlock Origin (uBO), a widely used content and tracking blocker, was among the extensions removed.
- MV2 extensions installed on Chrome 138 or earlier may remain installed, but cannot receive updates or be reinstalled through the Chrome Web Store after removal.
- The impact extends beyond Chrome because the Chrome Web Store is the dominant extension marketplace for Chromium-based browsers.
- A Chromium browser may technically continue supporting MV2 while users can no longer conveniently obtain those extensions from Google's store.
- Brave mitigates this by maintaining access to selected MV2 extensions, including uBlock Origin, AdGuard, uMatrix, and NoScript.
- Google argues that Manifest V3 improves extension security, privacy, performance, and control over privileged browser capabilities.
- MV3 does not eliminate ad blocking: uBlock Origin Lite is an MV3-compatible alternative, but it does not provide the full flexibility of classic uBlock Origin.
- The transition therefore concerns not only extension compatibility but also how much control browser extensions—and ultimately users—have over network requests and web content.

# 5. Deep Dive (Structured Understanding)

## Problem

Browser extensions can access sensitive browsing data and intercept network activity. Google argues that the broad permissions available under Manifest V2 create security, privacy, and performance risks.

However, those same capabilities enable sophisticated tools such as uBlock Origin to dynamically inspect and block requests. Restricting them therefore creates a trade-off between reducing extension privileges and preserving powerful user-controlled filtering.

A second problem is distribution. The Chrome Web Store functions as infrastructure for much of the Chromium ecosystem, so removing an extension from the store can affect browsers other than Chrome even when their engines technically still support it.

## Approach

Google has gradually migrated Chrome's extension ecosystem from Manifest V2 to Manifest V3.

MV3 places tighter constraints on what extensions can do, particularly around network-request interception. Content blockers increasingly rely on declarative filtering mechanisms rather than unrestricted runtime interception.

The final Chrome Web Store removal makes MV3 the practical baseline for extensions distributed through Google's ecosystem.

Other browsers can choose different policies. Brave, for example, can preserve selected MV2 extensions through its own distribution mechanism, while Firefox has maintained support for the capabilities required by classic uBlock Origin.

## Key Insight

The important distinction is:

**Ad blocking has not been removed from Chrome; the extension capabilities available to ad blockers have changed.**

uBlock Origin Lite demonstrates that advertising can still be blocked under MV3. The larger change is that extensions have less freedom to dynamically manipulate browser network behavior than under the classic MV2 model.

This also exposes a governance issue: because Google controls both Chrome and the dominant Chromium extension marketplace, its extension-platform decisions propagate through a large portion of the web ecosystem.

## Result / Impact

Classic uBlock Origin effectively loses normal Chrome Web Store distribution.

Chrome users can move to MV3-compatible blockers such as uBlock Origin Lite, while users requiring the original uBO capabilities have stronger incentives to consider Firefox or browsers explicitly preserving MV2 functionality.

The transition therefore increases differentiation between browsers based not merely on rendering engines or performance, but on extension policy and user control.

# 6. Why It Matters

- Browser extension APIs determine how much control users can exercise over the web pages and network requests running on their own machines.
- Chrome's market position means its extension architecture can become a de facto ecosystem standard even without being a universal web standard.
- The transition illustrates the tension between sandboxing extensions for security and preserving powerful user-controlled software.
- It strengthens the strategic importance of browser-engine and extension-platform diversity.
- It also highlights a structural conflict: Google operates a major advertising business while simultaneously controlling the dominant browser and Chromium extension marketplace.

# 7. Critical Analysis

- The headline can imply that uBlock Origin or ad blocking itself has been eliminated. This is too broad: MV3-compatible blockers, including uBlock Origin Lite, remain available.
- Google's security argument has technical merit because highly privileged extensions represent a meaningful attack surface. Restricting extension capabilities is not inherently an anti-ad-blocking measure.
- Conversely, improved security does not automatically justify every restriction introduced by MV3. Reduced extension privileges also reduce legitimate user control.
- The article does not deeply compare the concrete technical capabilities of MV2's `webRequest` model with MV3's `declarativeNetRequest`, making it difficult to quantify the practical loss for typical users.
- The effect on Chromium browsers should be separated into two issues: browser-level MV2 support and Chrome Web Store distribution. A browser can preserve the former while losing convenient access to the latter.
- Claims about Google's advertising incentives require caution. Google's business model creates an obvious potential conflict of interest, but that alone does not prove that weakening ad blockers was the primary motivation for MV3.
- For ordinary users primarily interested in removing visible ads, the practical difference between uBO and uBO Lite may be substantially smaller than the architectural difference suggests.

# 8. Connections

- **Browser Extension Sandboxing:** MV3 follows the broader security principle of reducing privileges and replacing arbitrary runtime behavior with constrained, declarative APIs.
- **uBlock Origin vs. uBlock Origin Lite:** The two projects illustrate the difference between powerful dynamic request interception and filtering designed around MV3's declarative model.
- **Browser Engine Monoculture:** Chromium's dominance means Chrome architectural decisions can affect Brave, Edge, and other Chromium-derived browsers, increasing concern about ecosystem concentration.
- **Firefox and Web Diversity:** Firefox provides an independent browser engine and a different extension-policy model, making it strategically important even beyond its market share.
- **Platform Governance:** The situation resembles mobile app stores, where control over distribution APIs and marketplaces can determine what third-party software is practically viable.
- **Ad-Tech Conflict of Interest:** Google's simultaneous roles in advertising, browsers, search, and extension distribution raise questions similar to other vertically integrated technology platforms.

# 9. Keywords

- Manifest V2
- Manifest V3
- uBlock Origin
- uBlock Origin Lite
- Chrome Web Store
- declarativeNetRequest
- webRequest API
- Chromium
- Firefox
- Browser Extensions

# 10. TL;DR

- Google removed remaining Manifest V2 extensions, including classic uBlock Origin, from the Chrome Web Store.
- Manifest V3 still permits ad blockers such as uBlock Origin Lite, but with a more constrained extension architecture.
- The larger issue is who controls browser capabilities: MV3 trades extension freedom for tighter security while increasing debate over Google's influence on the web ecosystem.
