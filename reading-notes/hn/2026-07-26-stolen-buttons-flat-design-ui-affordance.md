1. Title

Stolen Buttons: What a Button Collection Reveals About Modern Web UI

2. Source

* Author / Organization: Anatoly Zenkov / Hacker News Discussion
* Link: https://anatolyzenkov.com/stolen-buttons
* Date: 2026-07-26
    * Original page publication date not stated
    * Based on the Hacker News submission date

3. One-line Summary

A playful archive of copied web buttons exposes how modern flat design has made interfaces more uniform, less expressive, and sometimes less obviously interactive.

4. Key Points

* Stolen Buttons is a collection of button designs gathered from real websites and presented as digital specimens.
* “Stolen” is a joke: the buttons are visually copied rather than removed from their original websites.
* The project also provides a browser extension for collecting buttons encountered while browsing.
* Many collected elements are visually similar combinations of solid colors, rounded corners, padding, and text.
* Hacker News commenters contrasted modern flat buttons with older interfaces that used shadows, bevels, gradients, and pressed states.
* The discussion centered on whether modern buttons still provide sufficient visual affordance.
* Static copies often omit hover, focus, active, disabled, loading, and animation states that define actual button behavior.
* Many elements that appear to be buttons are implemented as anchor elements rather than semantic HTML <button> elements.
* The browser extension raises trust concerns because collecting elements across websites may require broad page-access permissions.
* Commenters proposed turning the collection into a searchable design archive organized by date, shape, color, dimensions, framework, or similarity.

5. Deep Dive

Problem

Modern websites contain countless buttons, but individual button designs are rarely preserved as artifacts.

At the same time, contemporary interfaces increasingly rely on flat shapes, subtle color differences, and contextual conventions. This can make interactive elements visually indistinguishable from labels, cards, or decorative blocks.

The project unintentionally raises two related questions:

1. How much diversity remains in modern button design?
2. Can users still recognize a button without relying on learned context?

Approach

The author collects buttons from websites and displays them together in a large visual gallery.

A browser extension allows users to select and save button-like elements from pages they visit. The collection focuses primarily on captured appearance rather than complete component behavior.

By removing each button from its original layout, the gallery enables direct comparison of:

* Shape
* Color
* Typography
* Border radius
* Icon usage
* Visual depth
* Perceived clickability

Key Insight

A button is not defined only by its default appearance.

Its usability also depends on:

* Hover feedback
* Keyboard focus visibility
* Pressed-state feedback
* Disabled-state distinction
* Loading behavior
* Semantic HTML
* Screen-reader labeling
* Placement within surrounding content

Once removed from context and reduced to a static representation, many modern buttons no longer clearly communicate that they are interactive.

The apparent sameness of the collection is therefore both a design trend and a limitation of the collection method.

Result / Impact

The project became popular because a humorous concept triggered a broader discussion about the evolution of graphical interfaces.

The conversation moved from button-collection jokes to criticism of:

* Flat design
* Weak affordances
* UI homogenization
* Touchscreen-only controls
* Browser-extension permissions
* Loss of personality in modern web design

The collection also has potential value as a dataset for studying design trends, component systems, and interface accessibility.

6. Why It Matters

Buttons are among the most fundamental interaction primitives in software.

When users cannot distinguish an action from surrounding content, they must spend additional attention discovering how the interface works. This is especially significant for:

* Older users
* Users with cognitive impairments
* Users unfamiliar with a product
* Keyboard-only users
* Users moving between inconsistent design systems

The discussion reflects a larger shift from skeuomorphic interfaces toward flat and design-system-driven interfaces.

Flat design improved scalability, responsiveness, and implementation consistency, but often reduced explicit visual cues. Modern UI development therefore faces a trade-off between visual simplicity and interaction clarity.

The project also highlights a preservation problem. Websites change continuously, and UI components are rarely archived in a structured form. A well-indexed collection could serve as a historical record of web design evolution.

7. Critical Analysis

The discussion frequently idealizes older interfaces without fully accounting for their disadvantages.

Skeuomorphic and heavily styled buttons could:

* Create excessive visual noise
* Age quickly
* Require image assets
* Reduce consistency across platforms
* Compete with primary content
* Perform poorly across different screen sizes

Flat design itself does not necessarily make buttons unusable. A flat button can remain clear through contrast, spacing, typography, placement, cursor behavior, and interaction feedback.

The collection also provides incomplete evidence for claims about modern UI uniformity. It appears to preserve default visual states while omitting hover, active, focus, and animation behavior. Components that seem identical as static images may behave differently during interaction.

The dataset may also contain selection bias. The collected buttons reflect websites visited by the author or extension users rather than a representative sample of the web.

Several Hacker News comments attribute design changes mainly to designers following fashion or attempting to justify their roles. That explanation is reductive. The transition toward flat design was also influenced by:

* Mobile interfaces
* Responsive layouts
* High-density displays
* CSS capabilities
* Cross-platform design systems
* Performance requirements
* Product consistency
* Accessibility standards

The browser extension discussion identifies a legitimate security concern, but broad permissions do not prove malicious behavior. The real issue is the extension trust model: users must trust both the current developer and every future software update.

8. Connections

1. Skeuomorphism and Flat Design

Early graphical interfaces used shadows, bevels, gradients, and physical metaphors to communicate interactivity.

The move toward Metro UI, iOS 7, and Material Design reduced decorative realism and favored typography, spacing, geometric shapes, and motion.

Stolen Buttons provides a visual snapshot of the long-term outcome of that transition.

2. Affordance and Signifiers

In human-computer interaction, affordance describes what actions an object enables, while signifiers communicate where and how those actions can be performed.

A physical button affords pressing through shape and movement. A flat digital rectangle may depend almost entirely on color, text, context, and convention as signifiers.

The collection demonstrates how weak many components become when separated from their original context.

3. Design Systems and Component Libraries

Frameworks such as Bootstrap, Material UI, Chakra UI, Ant Design, and enterprise design systems encourage standardized component structure.

This improves consistency and development speed but can produce visual convergence across unrelated products.

Many modern buttons therefore differ only through design tokens such as:

* Color
* Radius
* Typography
* Spacing
* Border
* Elevation

4. Semantic HTML and Accessibility

Elements styled as buttons may be implemented using <a>, <div>, or <span> rather than <button>.

Visual similarity does not guarantee equivalent behavior for:

* Keyboard navigation
* Form submission
* Screen readers
* Focus management
* Disabled states

A visual button archive cannot evaluate accessibility without also capturing markup and behavior.

5. Browser Extension Security

Extensions that inspect arbitrary page elements may require permission to access data across websites.

This resembles the broader security problem faced by password managers, ad blockers, developer tools, and productivity extensions: useful functionality often requires privileges that could also enable surveillance or data extraction.

Security therefore depends on code transparency, update integrity, ownership stability, and minimal permission design.

6. Digital Preservation

Projects such as the Internet Archive preserve pages, but interactive states and component-level metadata are often lost.

A structured UI archive could preserve:

* Source URL
* Capture date
* CSS properties
* DOM semantics
* Interaction states
* Accessibility attributes
* Framework fingerprints

This would make the collection useful for design history and empirical UI research.

9. Keywords

* UI Affordance
* Button Design
* Flat Design
* Skeuomorphism
* Design Systems
* Semantic HTML
* Web Accessibility
* Browser Extensions
* Interaction Design
* Digital Preservation

10. TL;DR

* Stolen Buttons turns ordinary web buttons into a playful visual archive.
* The collection reveals both the uniformity of flat design and the limits of judging interactive components from static appearance.
* Its deeper value lies in studying UI affordance, accessibility, design-system convergence, and web design history.