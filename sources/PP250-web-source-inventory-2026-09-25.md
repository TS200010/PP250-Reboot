# PP250 web presentation source inventory — 2026-09-25

Purpose: preserve newly located PP250/System 250 web sources, their provenance, download locations, and bibliography leads before any of them disappear. This file is an acquisition/index record; claims in later presentations are not treated as primary evidence until traced to an original source.

## Acquisition status key

- **LOCATED** — live copy found and inspected.
- **DOWNLOAD TARGET** — binary/original should be preserved under `sources/` when acquired.
- **MIRROR** — later web rendering/transcript; useful for recovery but not original provenance.
- **ARCHIVE TARGET** — original URL known but current original not yet recovered.
- **FOLLOW REFERENCE** — cited source should be acquired/checked.

---

## 1. Christian Diercks — *Capability Concept Mechanism and Structure in System 250*

**Status:** LOCATED; DOWNLOAD TARGET.

- Author: Christian Diercks
- Course/context: UCF COP6614, Fall 2004
- Format found: 17-page PDF exported from Microsoft PowerPoint
- Live URL: https://www.cs.ucf.edu/courses/cop6614/fall2004/System250_Direcks.pdf
- Note: server filename spells the surname `Direcks`; title slide says Christian Diercks.

### Contents of particular PP250 interest

Architecture; basic capability mechanism; capability access field; SCT; Load Capability; package/program/resource structure; CALL/RETURN/Store Capability; process dump stack; process structure; Change Process; system structure.

### References on final slide

1. `http://www.informatic.uni-ulm.de/rs/projecte/monads/capabilitiesE.html`
   - **FOLLOW REFERENCE.** Historical MONADS/capability material; original URL may require archive recovery.
2. `http://www.cs.washington.edu/homes/levy/capabook/Chapter4.pdf`
   - **FOLLOW REFERENCE.** Henry M. Levy, *Capability-Based Computer Systems*, chapter 4.
   - **LOCAL WORKING TEXT PRESERVED (2026-09-26):** `transcriptions/levy-capability-based-computer-systems-chapter-4-pasted-extract.txt`, derived from a Chapter 4 text pasted into the PP250-Reboot research chat. This is not yet a PDF-verified transcription; retain the PDF as a download/acquisition target and verify the working text against it when recovered.
3. `http://www.cs.ucf.edu/~eurip/cop6614/englandplessey250.pdf`
   - **FOLLOW REFERENCE / LOCATED in newer UCF path.** D. M. England System 250 paper; see item 3 below.

### Provenance warning

This is a student/course presentation and therefore a secondary source. Diagrams and architectural assertions should be compared with England and other Plessey primary sources before being incorporated as reconstruction facts.

---

## 2. Hua Zhang — *Capability Concept Mechanisms and Structure in System 250*

**Status:** LOCATED as web mirrors/transcript; ORIGINAL PPT DOWNLOAD TARGET.

- Presenter: Hua Zhang
- Course/context: UCF COP6614, Fall 2005
- 29 slides
- UCF reading-list context: https://www.cs.ucf.edu/courses/cop6614/fall2005/readinglist.html
- SlideServe mirror/transcript: https://www.slideserve.com/bina/capability-concept-mechanisms-and-structure-in-system-250
- SlideToDoc mirror: https://slidetodoc.com/capability-concept-mechanisms-and-structure-in-system-250/

### References on slide 28

1. D. M. England, *The Capability Concept Mechanism and Structure in System 250*, IRIA International Workshop on Protection in Operating Systems, Rocquencourt (1974), pp. 63–82.
2. H. Levy, *Capability-based Computer Systems*, Digital Press, 1984.

### Claims requiring primary-source tracing

Especially important is slide 18's statement that the Store Allocator allocates a segment and creates a capability and is "the ONLY place where Capabilities can be manufactured". This is potentially important to the PP250 capability-creation reconstruction, but must be traced to England/Plessey material rather than accepted on the authority of the student deck.

Other high-value subjects: mixed segments; Process Allocator; process dump stack; CR6/CR7 CALL/RETURN behaviour; process workspace stack.

### Provenance task

Compare slide-by-slide with Diercks 2004 and England 1974 to determine whether Zhang is an independent interpretation, an update of Diercks, or principally a condensation of England.

---

## 3. D. M. England — *Capability Concept Mechanisms and Structure in System 250*

**Status:** LOCATED; PRIMARY SOURCE; DOWNLOAD TARGET.

- Author: D. M. England
- Venue: International Workshop on Protection in Operating Systems, IRIA, Rocquencourt, France, August 1974
- UCF scan: https://cs.ucf.edu/courses/cop6614/fall2005/englandplessey250.pdf
- Scan contains the article through printed page 82; the final page includes Fig. 7, **Structure of a Process**.

### Pagination issue to preserve

Citations disagree on the start page:

- UCF/Hua Zhang and several later bibliographies: pp. **63–82**.
- Some later course bibliographies: pp. **68–82**.

The scan itself begins at printed page 63, so 63–82 is presently the stronger reading. Do not silently discard the 68–82 variant; retain it as a bibliographic discrepancy until the proceedings table of contents is checked.

### Importance

This is a primary architectural source and appears to be the main source behind both UCF presentations. It contains original System 250 diagrams including Load Capability, Structure of a Package, Structure of a Program, Structure of a Resource, and Structure of a Process.

---

## 4. SIPantic — old “lay presentation on System 250”

**Status:** ARCHIVE TARGET; original MHT not yet recovered.

An old System 250 page records the following original URL:

`http://www.sipantic.net/third%20international%20conference%20on%20computer%20communications.mht`

It describes this as “a lay presentation on System 250”.

Surviving pointer/page: https://en-academic.com/dic.nsf/enwiki/228235/

### Recovery tasks

- Search Internet Archive/Wayback and other web archives for the exact MHT URL and SIPantic directory variants.
- If recovered, preserve the **whole MHT**, not just rendered slides: MHT may contain embedded slide images, speaker/support material, hyperlinks and other resources.
- Extract every citation and compare any PP250 technical claims with the ICC papers and other primary sources.

---

## 5. Kenneth Hamer-Hodges — *Architecture Choices for Security — 2007*

**Status:** LOCATED as SlideServe rendering/transcript; DOWNLOAD TARGET if original can be recovered.

This was found while pursuing the SIPantic trail and is highly relevant. It may also be a candidate for the remembered “tacky but interesting” slide presentation.

- Author/presenter: Kenneth Hamer-Hodges
- Date/title slide: 2007
- Title slide identifies SIPantic
- 39 slides
- Mirror: https://www.slideserve.com/yeshaya/architecture-choices-for-security-2007

### Direct PP250 content

- Slide 18: **Plessey System 250 C1972 Capability Security** — discusses capabilities, POLA, designation, transfer, revocation, and distributed/network framing.
- Slide 19: **Capability PP-250 & E** — explicitly compares PP250 capability ideas with E/networked capabilities.
- Slide 20: **Further Research Links** — unusually useful bibliography/lead slide.
- Slide 38: object-capability timeline includes Hamer-Hodges, England and System 250.

### References/leads on slide 20

1. Jack B. Dennis and Earl C. Van Horn, *Programming Semantics for Multiprogrammed Computations* (1966). **FOLLOW REFERENCE.**
2. K. Hamer-Hodges, *A Fault-Tolerant Multiprocessor Design for Real-time Control*, *Computer Design*, Dec. 1973, pp. 75–81. **HIGH-PRIORITY FOLLOW REFERENCE** (direct PP250 author/source).
3. Marc Stiegler, *E in a Walnut*. **Contextual capability reference.**
4. Mark Miller, Chip Morningstar and Bill Frantz, *Capability-based Financial Instruments*, Financial Cryptography 2000. **Contextual distributed-capability reference.**
5. Jonathan Rees, *A Security Kernel Based on the Lambda-Calculus*, MIT AI Memo 1564 (1996). **Contextual object-capability reference.**
6. J. S. Shapiro and S. Weber, *Verifying the EROS Confinement Mechanism*, IEEE Symposium on Security and Privacy (2000). **Contextual capability reference.**

Other slides contain numerous contemporary security references/links; preserve these if/when the original deck is recovered, but the six above are the principal research leads on the explicit “Further Research Links” slide.

### Research significance

Unlike the UCF decks, this presentation is by Hamer-Hodges himself and therefore deserves special treatment when it discusses the intended architectural/security ideas of System 250. It is still a 2007 retrospective, not a 1970s contemporaneous specification, so retrospective interpretation must be distinguished from contemporaneous documentation.

---

## 6. UCF COP6614 Fall 2005 reading list

**Status:** LOCATED; preserve as provenance/context.

URL: https://www.cs.ucf.edu/courses/cop6614/fall2005/readinglist.html

It explicitly pairs England's 1974 System 250 paper with Hua Zhang's October 6 presentation, strongly supporting the conclusion that Zhang's deck was prepared as a presentation of the England paper rather than as independent System 250 research.

The reading list also identifies nearby capability literature including Dennis & Van Horn and Fabry, useful for reconstructing the intellectual context in which England's paper was taught.

---

# Consolidated acquisition queue

## Highest priority

1. Preserve the Diercks 2004 PDF binary in `sources/`.
2. Preserve the UCF scan of England 1974 in `sources/`.
3. Recover the original Hua Zhang PPT/PPTX if still present behind SlideServe or elsewhere; otherwise preserve a faithful local slide/image/text reconstruction with provenance clearly marked.
4. Recover the SIPantic MHT from web archives.
5. Recover/preserve Hamer-Hodges *Architecture Choices for Security — 2007* in its original PowerPoint/MHT form if possible.
6. Locate and preserve Hamer-Hodges, *A Fault-Tolerant Multiprocessor Design for Real-time Control*, *Computer Design*, Dec. 1973, pp. 75–81.

## Follow-on bibliography

7. Levy, *Capability-Based Computer Systems* (especially Chapter 4 and the System 250 chapter/material). Working text is preserved under `transcriptions/`; original PDF remains to be acquired and checked.
8. Dennis & Van Horn (1966).
9. Fabry, *Capability-Based Addressing* (1974).
10. Recover the old Ulm MONADS `capabilitiesE.html` page/archive cited by Diercks.
11. Follow the object/distributed-capability references in Hamer-Hodges 2007 where they bear on PP250 outform/inform and inter-computer authority research.

# Method rule

For each recovered source:

1. preserve the original binary unchanged;
2. record original URL, mirror/archive URL, retrieval date, author/title/date and checksum where practical;
3. transcribe/extract text separately rather than modifying the original;
4. capture the complete bibliography/reference slide(s);
5. follow PP250/Plessey-specific references recursively;
6. classify technical assertions as **primary/documented**, **retrospective**, **secondary**, or **inference** before feeding them into the architecture reconstruction.
