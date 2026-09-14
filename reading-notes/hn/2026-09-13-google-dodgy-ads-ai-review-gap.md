## 1. Title

Why Is Google Still Serving Dodgy Ads?

## 2. Source

- Author / Organization: Chris Greening / atomic14
- Link: https://www.atomic14.com/2026/09/13/why-is-google-still-serving-dodgy-ads
- Date: 2026-09-13

## 3. One-line Summary

- A deceptive YouTube ad repeatedly passed Google's review despite Gemini immediately identifying apparent policy violations, raising questions about whether ad-safety failures are primarily technical problems or incentive and enforcement problems.

## 4. Key Points

- The observed YouTube ad imitated an iOS “iPhone Storage is Full” system alert, including fake Yes/No controls.
- The author accidentally clicked the ad, then reported it to Google multiple times.
- Google responded that the ad did not violate its policies; other users reportedly received similar responses.
- When shown the same creative, Gemini classified it as disallowed because it mimicked system UI, used deceptive controls, and relied on fear-based claims.
- The article argues that modern AI appears capable of recognizing this class of deceptive advertising quickly.
- The central inconsistency is that Google's consumer AI can identify the problem while Google's production ad-review process apparently allows it.
- Hacker News commenters reported similar experiences involving fake warnings, fraudulent products, impersonation, malicious redirects, and recurring advertiser accounts.
- Several commenters argued that scam detection is complicated by cloaking, rotating domains, fingerprinting, residential-IP checks, and disposable advertising accounts.
- A commenter claiming experience with Google Ads said Google already uses both automated and human review, suggesting the issue is not simply the absence of AI moderation.
- Much of the discussion therefore shifted from technical capability toward incentives, false-positive tolerance, moderation cost, advertiser revenue, and platform liability.

## 5. Deep Dive (Structured Understanding)

### Problem

Google operates one of the world's largest advertising platforms, yet deceptive ads can still imitate trusted operating-system interfaces and remain active even after user reports.

The specific example is especially notable because the deception is visually simple: an advertisement presents itself as an iOS storage warning and uses fake system controls to induce interaction.

The problem therefore has two layers:

1. harmful advertising passes initial review;
2. subsequent user reports may still fail to trigger removal.

### Approach

The author uses Google's own Gemini model as an informal second reviewer.

Gemini analyzes the creative against advertising-policy concepts and identifies several apparent violations:

- imitation of system UI;
- misleading interactive elements;
- deceptive device-status claims;
- fear-based pressure intended to drive clicks.

The comparison creates a natural experiment:

`Google production review → allowed`

`Google Gemini analysis → disallowed`

The Hacker News discussion then broadens the analysis beyond this single advertisement by comparing similar experiences from publishers, advertisers, and users.

### Key Insight

The strongest interpretation is not simply that Google lacks capable detection technology.

Large-scale advertising moderation is an optimization problem involving several competing objectives:

`fraud detection`
`↔ false positives`
`↔ advertiser retention`
`↔ moderation cost`
`↔ ad inventory and revenue`
`↔ regulatory risk`
`↔ user trust`

Aggressive filtering reduces fraudulent ads but can also block legitimate advertisers and generate expensive appeals.

Loose filtering preserves revenue and advertiser throughput but increases harmful false negatives.

Scammers further exploit this system through cloaking, account rotation, fingerprinting, disposable domains, and other adversarial techniques.

The article therefore exposes a gap between what an AI model can classify in an isolated example and what an advertising platform is economically and operationally optimized to reject at scale.

### Result / Impact

The incident became a large Hacker News discussion centered on platform incentives rather than AI capability alone.

A recurring argument was that scam advertising imposes much of its external cost on users, publishers, financial institutions, and society, while the advertising platform still receives revenue.

That creates an asymmetric incentive:

`allowing a bad ad → potential revenue`

`blocking a good ad by mistake → immediate lost revenue + advertiser friction`

`allowing a bad ad → much of the downstream harm is externalized`

If that incentive structure remains unchanged, better AI models alone may not produce proportionally safer advertising ecosystems.

## 6. Why It Matters

- AI safety systems are shaped as much by deployment incentives and thresholds as by raw model capability.
- Content moderation is increasingly an adversarial ML problem in which attackers actively design inputs to evade classifiers.
- The case illustrates the difference between demonstrating that an LLM can identify abuse and building a reliable production moderation pipeline.
- Advertising platforms create multi-sided incentives among users, publishers, advertisers, and the platform itself; these interests are not always aligned.
- As generative AI lowers the cost of producing ads, identities, landing pages, and creative variants, scam campaigns can scale faster as well.
- Platform liability and regulation may therefore matter as much as advances in classification accuracy.

## 7. Critical Analysis

- Gemini's response does not prove that Google's production systems are incapable or intentionally permissive; the two systems may use different policies, inputs, thresholds, and operational constraints.
- A single clean screenshot is easier to classify than the full advertising pipeline, where landing-page behavior, targeting, account history, redirects, and regional variation matter.
- The article implies that Google could solve the issue simply by applying Gemini, but production moderation requires controlling false positives across enormous advertising volume.
- The claim that Google benefits financially from intentionally allowing scam ads is plausible as an incentive argument but is not established by the article itself.
- Hacker News anecdotes provide useful evidence of repeated failure modes but cannot establish their prevalence across Google's entire advertising network.
- Several comments introduce an important adversarial factor missing from the original article: scammers may show benign content to reviewers while delivering malicious content only to selected real users.
- The strongest criticism is therefore not “Google does not use AI,” but that repeated reports of apparently obvious deceptive creatives indicate weaknesses in enforcement thresholds, escalation mechanisms, or organizational incentives.
- Determining whether the failure is caused primarily by economics, moderation architecture, adversarial evasion, policy design, or operational scale would require internal data unavailable in the article.

## 8. Connections

### 1. Adversarial Machine Learning and Cloaking

Scam advertisers behave like adversarial attackers against a classifier.

They can modify creatives, rotate identities, fingerprint visitors, or serve different landing pages depending on whether traffic appears to come from an automated reviewer.

This resembles adversarial security systems where detection causes attackers to adapt rather than disappear.

### 2. Spam and Email Filtering

Advertising moderation has structural similarities to spam filtering.

Email providers rarely aim for zero spam because an aggressive classifier that deletes legitimate mail can be more damaging than occasional spam leakage.

Advertising platforms face the same precision-recall trade-off, except false negatives can also generate advertising revenue.

### 3. Malvertising

The issue belongs to the broader history of malvertising: legitimate advertising infrastructure being used to distribute scams, phishing, scareware, malware, and deceptive redirects.

The trusted advertising network effectively becomes part of the attack delivery chain.

### 4. Platform Incentive Alignment

The case resembles problems in social-media recommendation systems where engagement optimization can conflict with safety or information quality.

A platform's observable behavior follows the objectives and constraints embedded in its production systems, not merely the capabilities of its best available model.

### 5. Generative AI and Scam Economics

Generative AI reduces the marginal cost of producing advertising creatives, fake testimonials, synthetic spokespersons, localized copy, landing pages, and account variants.

AI therefore strengthens both sides of the contest: platforms can detect scams more efficiently, while scammers can create and mutate campaigns more efficiently.

### 6. Platform Liability

HN discussion repeatedly connects the issue to legal liability.

If platforms internalized more of the financial cost produced by fraudulent advertising, their optimal moderation threshold could shift toward substantially stricter enforcement.

## 9. Keywords

- Google Ads
- YouTube Ads
- deceptive advertising
- ad moderation
- malvertising
- adversarial machine learning
- cloaking
- Gemini
- platform incentives
- content moderation

## 10. TL;DR

- A deceptive iOS-style YouTube ad survived repeated Google reports while Gemini immediately identified apparent policy violations.
- The discrepancy suggests that ad safety depends on production thresholds, adversarial evasion, economics, and enforcement incentives—not merely AI capability.
- Better classifiers can help, but meaningful improvement may require changes to moderation architecture and platform accountability.
