# 1. Title



Kage: Shadow Any Website for Offline Viewing



# 2. Source



* Author / Organization: Tam Nguyen Duc (tamnd)

* Link: https://github.com/tamnd/kage

* Date: 2026-06-15 (Hacker News discussion)



# 3. One-line Summary



Kage is a Go-based website archiving tool that renders pages in Chrome, captures the final DOM, strips JavaScript, localizes assets, and packages entire websites into portable offline archives.



# 4. Key Points



* Uses real headless Chrome/Chromium instead of raw HTML crawling.

* Captures the fully rendered DOM after client-side JavaScript execution.

* Removes scripts, event handlers, and JavaScript URLs from archived pages.

* Downloads CSS, images, and fonts and rewrites references to local paths.

* Supports full-site crawling with depth, scope, and page limits.

* Generates portable ZIM archives compatible with the Kiwix ecosystem.

* Can package a website into a self-contained executable viewer.

* Supports crawl resumption through stored crawl state.

* Uses deterministic URL-to-path mapping to avoid duplicate content.

* Positions itself as a modern alternative to HTTrack for JavaScript-heavy websites.



# 5. Deep Dive (Structured Understanding)



## Problem



Traditional website archiving approaches struggle with modern web applications.



Common failures include:



* React/Next.js pages saving as empty shells.

* Archived pages depending on remote APIs.

* JavaScript-driven content disappearing when services change.

* Browser "Save As" producing incomplete snapshots.



As more content becomes dynamically rendered, preserving websites accurately becomes increasingly difficult.



## Approach



Kage uses a browser-first architecture:



1. Open pages in real Chrome.

2. Allow all client-side rendering to finish.

3. Capture the final DOM visible to users.

4. Remove executable JavaScript.

5. Download dependent assets.

6. Rewrite links to local resources.

7. Store the result as a browsable offline mirror.



The archived content becomes static, self-contained, and independent of external services.



## Key Insight



The most important insight is that archiving should preserve the *rendered result* rather than the original source code.



Instead of saving HTML before execution, Kage saves the page after JavaScript, API calls, hydration, and rendering have completed.



This shifts website preservation from source capture to experience capture.



## Result / Impact



The resulting archive:



* Works without internet access.

* Eliminates tracking and analytics scripts.

* Preserves modern JavaScript websites more accurately.

* Can be distributed as folders, ZIM archives, or standalone binaries.

* Integrates with existing offline ecosystems such as Kiwix.



# 6. Why It Matters



## Importance



Modern websites increasingly behave like applications rather than documents.



Many pages cannot be meaningfully preserved using traditional scraping methods.



Kage addresses the growing gap between web preservation needs and browser-based application architectures.



## Broader Trend



The project aligns with several industry trends:



* Growth of client-rendered web applications.

* Rising concern over digital preservation.

* Interest in offline-first computing.

* Personal knowledge archiving and digital sovereignty.

* Decentralized ownership of information.



# 7. Critical Analysis



## Assumptions



* Assumes the rendered DOM fully represents the useful state of a website.

* Assumes removing JavaScript does not significantly reduce functionality.



These assumptions hold for many content-focused sites but not all applications.



## Limitations



* Authentication and cookie handling remain limited.

* Dynamic interactions after initial rendering may be lost.

* WebSockets, real-time updates, and complex application states are difficult to preserve.

* Large documentation sites can generate enormous archives.



## Missing Context



* No benchmark comparisons against HTTrack, ArchiveBox, Browsertrix, or SingleFile.

* Limited discussion of archival fidelity for highly interactive applications.

* Storage and crawl-performance metrics are not extensively documented.



# 8. Connections



## 1. HTTrack



Kage can be viewed as a browser-rendering evolution of HTTrack.



HTTrack mirrors raw web content, while Kage captures post-render application output.



## 2. SingleFile



Both tools preserve rendered pages.



However:



* SingleFile focuses on individual pages.

* Kage focuses on entire websites and multi-page archives.



## 3. ArchiveBox / Browsertrix



These projects emphasize archival completeness and replayability.



Kage prioritizes lightweight offline usability and simplified distribution.



## 4. Kiwix & ZIM



Kage leverages the ZIM ecosystem, connecting website preservation to established offline knowledge distribution platforms used for Wikipedia and educational archives.



## 5. Static Site Generation



Conceptually similar to static site generation, except Kage converts existing dynamic websites into static artifacts after rendering.



# 9. Keywords



* Website Archiving

* Offline Web

* Headless Chrome

* DOM Snapshot

* Static Preservation

* ZIM Format

* Kiwix

* HTTrack

* Browser Automation

* Digital Preservation



# 10. TL;DR



* Kage archives modern websites by capturing fully rendered browser output.

* It removes JavaScript and converts websites into portable offline artifacts.

* The project reflects a broader shift toward preserving dynamic web content as static, durable knowledge assets.
