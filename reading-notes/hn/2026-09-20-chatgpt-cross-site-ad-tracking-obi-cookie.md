**1. Title**

ChatGPT Now Knows What You Do on Other Websites via Ad Collector

**2. Source**

- Author / Organization: Buchodi's Threat Intel
- Link: https://www.buchodi.com/chatgpt-now-knows-what-you-do-on-other-websites-via-ad-collector/
- Date: 2026-09-20

**3. One-line Summary**

- OpenAI's advertising infrastructure uses a cross-site `__obi` cookie and advertiser pixels that can send browsing and conversion events back to OpenAI under an identifier designed to be associated with a ChatGPT account or persistent anonymous subject.

**4. Key Points**

- ChatGPT can generate an `obi` identifier and obtain a short-lived signed JWT binding that identifier to either an `account_user` or persistent anonymous subject.
- `bzr.openai.com` converts this identifier into the `__obi` cookie with `Domain=.openai.com`, `SameSite=None`, `Secure`, `HttpOnly`, and a one-year lifetime.
- Advertisers integrating OpenAI's measurement pixel load resources from OpenAI domains; on the tested Chrome for Android setup, requests from those sites carried `__obi` back to OpenAI.
- The pixel SDK reports conversion events and can collect identity signals from advertiser-provided data, form fields, rendered HTML, and tag-manager data layers.
- Email, phone, and names are hashed before transmission, while some location fields such as country, city, region, and postal code may be transmitted directly.
- URLs were stripped of query strings in the observed dataset, but URL paths remained and sometimes exposed sensitive contextual information.
- Automatic matching was enabled for 638 of 881 observed pixels whose setting was known, including all observed credit and lending advertisers.
- The author observed one `__obi` identifier across multiple commercial sites, demonstrating the identifier's cross-site role under the tested browser configuration.
- Anonymous users also received persistent identifiers; observed anonymous subjects remained stable for at least 27 days.
- OpenAI's cookie policy reportedly classified `__obi` as an analytics cookie, while the author's decoded tokens consistently contained `consent_decision: analytics_allowed`.
- The author tested Chrome on Android; Safari/WebKit's third-party-cookie protections prevent this specific cookie mechanism, and desktop Chrome was not tested.
- The experiment observed events reaching OpenAI with the identifier attached, but did not directly observe the final server-side account-to-event join.

**5. Deep Dive (Structured Understanding)**

### Problem

Advertising platforms need attribution: they want to connect an ad interaction with later activity such as visiting a retailer, searching for a product, or purchasing it.

Applying conventional adtech to an AI assistant creates a different privacy boundary. Users may disclose substantially more contextual or personal information to a conversational assistant than to a conventional search or social interface.

The technical question is therefore whether activity occurring on advertiser websites can be associated with an identity originating from ChatGPT.

### Approach

The described system establishes a cross-site identifier in three stages:

1. ChatGPT generates an `obi` value and requests a signed synchronization token.
2. `bzr.openai.com` uses that token to establish the `__obi` cookie on `.openai.com`.
3. Websites integrating OpenAI's advertising SDK make requests to OpenAI infrastructure, allowing the browser to attach that cookie where third-party-cookie behavior permits it.

The author reproduced this on Chrome for Android using two capture methods and compared it with several months of traffic containing 936 advertiser pixels across 1,029 hostnames.

The SDK additionally gathers attribution and matching data from explicit advertiser inputs and automatically discovered page or tag-manager data.

### Key Insight

The significant component is not the conversion pixel itself but the shared identifier.

A site-specific identifier would isolate activity between advertisers. `__obi`, however, belongs to OpenAI's domain and can accompany requests generated from different participating websites.

This creates the technical basis for OpenAI to associate activity across advertiser sites with the same OpenAI-side subject.

The author contrasts this with `__obref`, which is stored on each advertiser's own domain and therefore remains site-specific.

### Result / Impact

The author observed the same `__obi` value reaching OpenAI from multiple commercial websites and found multiple identifiers appearing across multiple advertisers.

The system also operated with anonymous ChatGPT subjects, meaning account login is not strictly necessary for persistent attribution.

This effectively brings a familiar Meta/Google-style conversion-attribution architecture into an AI assistant ecosystem, where the originating service may possess unusually rich conversational context.

**6. Why It Matters**

- The architecture shows AI products converging with established adtech infrastructure rather than developing an entirely separate monetization stack.
- AI assistants occupy a higher-context relationship with users than conventional websites, making identity linkage potentially more consequential even when the tracking mechanism itself is conventional.
- Browser privacy architecture becomes part of AI privacy: `SameSite`, third-party-cookie partitioning, WebKit ITP, fingerprinting defenses, and tracker blocking directly affect what AI advertising systems can observe.
- The distinction between analytics and marketing consent becomes important when the same persistent identifier supports measurement infrastructure associated with advertising.
- AI assistants are evolving from isolated conversational interfaces toward platforms connected to commerce, advertising, recommendations, and external actions. Attribution infrastructure is a foundational component of that transition.

**7. Critical Analysis**

- The headline is stronger than the demonstrated result. The experiment shows that OpenAI receives advertiser-site events carrying an identifier designed to map to an OpenAI subject; it does not prove that ChatGPT itself directly retrieves arbitrary browsing history.
- The author explicitly did not observe the final server-side join between collected events and a ChatGPT account. That association is inferred from the token and identifier design.
- Testing was limited primarily to Chrome on Android. Safari/WebKit blocks the third-party-cookie mechanism described, while desktop Chrome was not tested.
- The article sometimes frames standard third-party-cookie attribution as novel. The architecture closely resembles long-established Meta Pixel and Google advertising mechanisms; the unusual part is its deployment around an AI conversational product.
- Observed advertiser traffic establishes technical capability and deployment but does not independently establish how collected information is subsequently retained, queried, used for personalization, or exposed to model inference.
- Hashing email or phone values is not equivalent to anonymization because deterministic hashes of known identifiers can still support matching.
- Conversely, sensitive URL paths demonstrate contextual leakage but do not prove that OpenAI deliberately designed those paths to encode sensitive information; URL structure is controlled largely by advertiser sites.
- Hacker News discussion also raises broader mechanisms such as fingerprinting and IP correlation, but these are separate from the specific `__obi` mechanism demonstrated in the article and should not be treated as proven components of this implementation.
- Claims about broader social consequences, surveillance, or future manipulation extend beyond what the network captures themselves establish.

**8. Connections**

- **Meta Pixel / Google Ads Conversion Tracking:** The closest architectural precedent. A third-party measurement endpoint receives advertiser-site events and attempts to associate them with a platform-side identity for attribution.
- **Third-Party Cookies & SameSite:** `SameSite=None; Secure` explicitly permits cross-site cookie transmission where browsers still allow third-party cookies, making browser cookie policy a core security boundary.
- **Safari ITP / Firefox Total Cookie Protection:** Browser-side partitioning and third-party-cookie restrictions can disrupt this specific cross-site identity mechanism, demonstrating how browser architecture constrains adtech.
- **Customer Match / Enhanced Conversions:** Hashing emails and phone numbers before transmission resembles existing advertising systems that match first-party customer identifiers against platform identities.
- **Tag Management Systems:** Reading `dataLayer`, `adobeDataLayer`, and GTM-derived information shows how analytics infrastructure can become an identity source beyond explicit pixel parameters.
- **AI Personalization & Memory:** Cross-site behavioral data introduces a fundamentally different personalization source from user-controlled conversational memory: inferred external behavior rather than information deliberately provided inside the assistant.
- **Privacy-Preserving Attribution:** Systems such as browser-mediated attribution APIs represent an alternative design direction: measuring conversions while reducing direct cross-site identity linkage.

**9. Keywords**

- OpenAI Ads
- `__obi`
- Cross-Site Tracking
- Third-Party Cookies
- Conversion Attribution
- Advertising Pixel
- SameSite=None
- Identity Matching
- Browser Privacy
- AdTech

**10. TL;DR**

- OpenAI's ad measurement system uses `__obi`, a persistent OpenAI-domain identifier that can accompany requests from participating advertiser websites.
- The mechanism resembles conventional adtech, but its integration with an AI assistant creates a more sensitive identity and context boundary.
- The demonstrated risk is real but browser-dependent, and the article infers rather than directly observes the final server-side linkage and downstream use of collected events.
