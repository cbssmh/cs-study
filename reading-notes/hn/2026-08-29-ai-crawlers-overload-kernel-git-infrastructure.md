# 1. Title

Creepy Crawlies

# 2. Source

* Author / Organization: Konstantin Ryabitsev / kernel.org
* Link: https://people.kernel.org/monsieuricon/creepy-crawlies
* Date: 2026-08-29

# 3. One-line Summary

AI-oriented scrapers consume roughly 20% of git.kernel.org's compute capacity by inefficiently rendering massive numbers of Git commits as HTML instead of cloning repositories, while increasingly bypassing conventional IP blocking and proof-of-work defenses.

# 4. Key Points

* git.kernel.org receives roughly 6 million daily requests for commit pages, with the author estimating legitimate traffic at only about 2%.
* Across five geographically distributed nodes with 90 CPU cores, roughly 14–16 cores are continuously occupied rendering commits for scrapers.
* Linux's history is already efficiently downloadable through `git clone`, making commit-by-commit HTML scraping unnecessarily expensive for the server.
* `linux.git` contains about 1.48 million commits and has roughly 922 forks on git.kernel.org, creating billions of URLs that can expose largely duplicated information.
* cgit additionally exposes patches, plain views and arbitrary diffs, producing a combinatorial URL space that naive crawlers can continuously explore.
* Blocking by User-Agent, IP address, subnet or cloud ASN became progressively less effective as crawlers adapted.
* Modern scraping traffic increasingly arrives through rotating residential and mobile IPs, sometimes enabled by residential proxy SDKs embedded in consumer applications or devices.
* git.kernel.org deployed Anubis, a proof-of-work challenge that initially drove most crawlers away.
* Raising Anubis difficulty increased costs for bots but also degraded legitimate mobile-user experience; crawlers eventually adapted and began solving the harder challenges.
* The project is now reducing anonymously accessible functionality and restricting computationally expensive endpoints rather than relying on a single anti-bot mechanism.

# 5. Deep Dive (Structured Understanding)

## Problem

git.kernel.org was designed around open access and human-scale browsing. Git repositories and mailing-list archives are intentionally downloadable, but cgit also provides dynamically generated HTML views of commits, patches and diffs.

Generic crawlers treat these generated URLs as independent resources. Because Git repositories contain enormous histories, forks and arbitrary comparison possibilities, the apparent HTTP crawl space becomes vastly larger than the underlying dataset.

The result is an asymmetric infrastructure problem: retrieving data is cheap for distributed scrapers, while each request can force git.kernel.org to perform server-side computation.

## Approach

The operators progressively introduced defenses:

1. Identify crawlers through User-Agent strings.
2. Block abusive IP addresses with tools such as fail2ban.
3. Block suspicious cloud subnets or entire ASNs.
4. Deploy Anubis to require clients to perform proof-of-work before accessing the site.
5. Increase proof-of-work difficulty as crawlers adapted.
6. Reduce the number of crawlable URLs and restrict expensive functionality for anonymous clients.

Each defense addresses a particular crawler behavior rather than eliminating scraping itself.

## Key Insight

The underlying data is not inherently expensive to distribute; the expensive part is the interface through which crawlers request it.

A repository can be transferred efficiently once with `git clone`, while an HTTP crawler can repeatedly force server-side rendering of commits, forks, patches and arbitrary diffs.

Residential proxy networks further weaken traditional IP-based abuse controls because a distributed crawler can make only a few requests from each apparently legitimate consumer IP before rotating to another.

Proof-of-work shifts some computational cost toward clients, but it does not permanently distinguish humans from sufficiently motivated automation.

## Result / Impact

Scraper traffic has become persistent infrastructure overhead rather than an occasional traffic spike.

Approximately 14–16 of 90 available CPU cores are continuously dedicated to serving this traffic, representing around 20% of total compute capacity.

Anubis still rejects a substantial portion of requests, but roughly one-third of challenged traffic now completes the proof-of-work and reaches the main site.

The practical response is therefore architectural degradation: git.kernel.org plans to remove or gate functionality that was previously freely available to anonymous users.

# 6. Why It Matters

This case demonstrates a broader shift from a human-oriented web to an environment where automated consumers can dominate infrastructure demand.

The important distinction is not simply "bots generate more traffic." Dynamic applications often expose enormous implicit URL spaces where each request triggers computation. Generic crawlers can unintentionally convert those features into resource-exhaustion mechanisms.

Residential proxy networks also undermine an important assumption behind traditional rate limiting: that an IP address provides a reasonably persistent identity.

The long-term consequence may be a less open web. Public projects can remain willing to share their underlying data while being forced to place authentication, computation challenges or restrictions around interfaces originally designed for unrestricted human browsing.

# 7. Critical Analysis

* The article attributes most abusive scraping to AI-related data collection, but traffic characteristics alone cannot reliably establish the operator or exact purpose of every crawler.
* The claim that pre-AI Linux history is especially valuable because training on synthetic data causes a "digital prion disease" is rhetorically strong and oversimplifies modern synthetic-data training, filtering and model-collapse research.
* Proof-of-work demonstrates empirical short-term effectiveness but has weak long-term differentiation between humans and automated clients. Specialized hardware and optimized implementations can make computation cheaper for sophisticated crawlers than for mobile users.
* The 2% legitimate-traffic estimate depends on behavioral assumptions, such as treating requests for obscure historical commits as likely automated traffic. The article acknowledges that bot/human classification is uncertain.
* The article focuses primarily on crawler behavior rather than whether cgit's server-side rendering architecture could be redesigned to reduce per-request computation.
* More caching is not automatically sufficient because arbitrary commit comparisons and duplicated fork paths create an enormous URL space with potentially poor cache hit rates.
* `git clone` is clearly more efficient for bulk repository acquisition, but generic web-scale crawlers may deliberately avoid site-specific retrieval logic because engineering one protocol adapter per content source also has a cost.
* Restricting anonymous functionality transfers part of the cost from crawler operators to legitimate users, creating an externality that raw CPU-utilization numbers do not capture.

# 8. Connections

* **DDoS and Rate Limiting:** The traffic resembles application-layer resource exhaustion, but rotating residential proxies defeat conventional per-IP throttling by distributing requests across enormous address pools.
* **Proof of Work / Hashcash:** Anubis applies the Hashcash principle of attaching computational cost to requests. Unlike spam prevention, however, successful scraping requests individually produce useful data, weakening the economic asymmetry PoW attempts to create.
* **Residential Proxy Networks:** Proxy SDK monetization turns consumer devices and residential connections into distributed exit nodes, making automated traffic resemble ordinary users and reducing the effectiveness of reputation-based blocking.
* **API vs. Web Scraping:** `git clone` represents a structured, efficient bulk-data interface, while scraping dynamically rendered HTML demonstrates the infrastructure cost of clients ignoring purpose-built data-access mechanisms.
* **Combinatorial URL Explosion:** Git history, arbitrary diffs, forks and parameterized views illustrate how a relatively compact underlying dataset can expose an enormous logical HTTP resource space.
* **Open Web vs. Authentication:** The pressure to gate expensive operations mirrors the broader migration of formerly anonymous public resources toward accounts, challenges and controlled APIs.
* **AI Infrastructure Externalities:** Model builders or data intermediaries can externalize acquisition costs onto content hosts, shifting compute, bandwidth and operational complexity toward projects that publish the source material.

# 9. Keywords

* AI Crawlers
* Web Scraping
* git.kernel.org
* cgit
* Proof of Work
* Anubis
* Residential Proxies
* Rate Limiting
* Resource Exhaustion
* Open Web

# 10. TL;DR

AI-oriented crawlers inefficiently scrape Git history through dynamically rendered HTML instead of using `git clone`, consuming roughly 20% of git.kernel.org's compute capacity.
IP blocking became ineffective after crawlers adopted massive rotating residential proxy networks, while Anubis proof-of-work provides only partial and increasingly costly protection.
The deeper problem is that automated access is forcing open infrastructure to restrict features originally designed for anonymous human use.
