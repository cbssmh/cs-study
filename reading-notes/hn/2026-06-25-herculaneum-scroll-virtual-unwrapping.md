# An Entire Herculaneum Scroll Has Been Read for the First Time

## Source

- **Author / Organization:** Vesuvius Challenge
- **Link:** https://scrollprize.org/firstscroll
- **Date:** 2026-06-25

---

## One-line Summary

Researchers successfully read an entire carbonized Herculaneum scroll without physically opening it by combining synchrotron X-ray imaging, virtual unwrapping, and machine learning.

---

## Key Points

- PHerc. 1667 became the first Herculaneum scroll to be virtually unwrapped and read continuously from beginning to end.
- The scroll remained sealed since the eruption of Mount Vesuvius in AD 79 and would have been destroyed by conventional opening.
- High-resolution phase-contrast X-ray microtomography captured the internal papyrus structure.
- Virtual unwrapping reconstructed the spiral geometry and flattened the papyrus into readable pages.
- Machine learning detected faint carbon-based ink that is nearly indistinguishable from carbonized papyrus.
- Classical scholars manually verified the reconstructed text after AI-assisted detection.
- The recovered work is a Stoic ethical treatise likely dating to the 2nd century BC and references Aristocreon, a disciple of Chrysippus.
- Higher-resolution scans independently confirmed previous Vesuvius Challenge text recovery results.
- Another scroll (PHerc.139) had its title identified as *Philodemus, On Gods, Book 8*.
- All scan data, reconstructed surfaces, transcriptions, and code have been released as open-source resources.

---

## Deep Dive (Structured Understanding)

### Problem

Hundreds of Herculaneum scrolls survived the eruption of Vesuvius only because they became carbonized. Unfortunately, this made them too fragile to physically unroll without destroying the text, leaving an inaccessible library for nearly two millennia.

### Approach

Researchers combined several technologies:

- Synchrotron phase-contrast X-ray microtomography
- 3D reconstruction of rolled papyrus geometry
- Virtual surface segmentation and flattening
- Machine learning for carbon-ink detection
- Human validation by papyrologists

Instead of physically opening the scroll, the entire reading process occurred digitally.

### Key Insight

Although both papyrus and ink are carbon-based and nearly indistinguishable in CT images, subtle texture differences are sufficient for ML models to identify probable ink locations after accurate geometric reconstruction.

Equally important, accurate segmentation and high-quality imaging proved more critical than increasingly sophisticated AI models.

### Result / Impact

The project demonstrated that complete non-destructive reading of sealed ancient scrolls is now practical.

This establishes a scalable workflow capable of unlocking hundreds of unread classical manuscripts while preserving the original artifacts.

---

## Why It Matters

- Represents one of the strongest examples of AI enabling new scientific discovery rather than replacing existing work.
- Opens access to potentially hundreds of lost philosophical, literary, and historical texts.
- Demonstrates how computer vision, medical imaging, and machine learning can transform archaeology.
- Validates an open-science model where community contributors became core researchers.
- Suggests future archaeological excavations may recover texts previously considered unreadable.

---

## Critical Analysis

- "Entire scroll" refers only to the surviving portion of PHerc.1667; significant outer layers were already destroyed during historical attempts to unroll it.
- Machine learning was only one component of the pipeline; advances in X-ray imaging and geometric reconstruction were equally essential.
- Large-scale deployment remains constrained by access to extremely expensive synchrotron beamlines.
- Human annotation and scholarly verification remain necessary, limiting full automation.
- Most recovered text remains fragmentary due to physical damage rather than algorithmic limitations.

---

## Connections

### Medical Imaging

Many segmentation and reconstruction techniques originated from CT/MRI analysis and medical image processing.

### AI for Scientific Discovery

Comparable to AlphaFold and protein structure prediction, where AI enables new scientific observations rather than consumer applications.

### Digital Humanities

Expands computational archaeology alongside OCR, multispectral manuscript recovery, and historical document digitization.

### Open Science

The project follows an open-data, open-code model similar to modern large-scale scientific collaborations.

---

## Keywords

- Herculaneum Scrolls
- Vesuvius Challenge
- Virtual Unwrapping
- Synchrotron CT
- Machine Learning
- Computer Vision
- Digital Humanities
- Archaeology
- Carbon Ink Detection
- Open Science

---

## TL;DR

- AI and synchrotron CT enabled the first complete virtual reading of a sealed Herculaneum scroll.
- The workflow combines imaging, geometric reconstruction, ML-based ink detection, and expert verification.
- The breakthrough creates a scalable path toward recovering hundreds of lost ancient manuscripts without damaging them.
