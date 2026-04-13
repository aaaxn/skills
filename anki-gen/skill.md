---
name: anki-gen
description: >
  Generate Anki flashcard decks (.apkg) from vault content. Use when the user asks
  to create Anki cards, flashcards, or study decks — from wiki pages, raw PDFs/PPTX,
  concept pages, or any course material in the vault. Handles reading source content,
  generating well-structured cards (Q&A and Cloze), and packaging as importable .apkg files.
---

# Anki Card Generation Skill

## When to Use

Trigger on: "make me anki cards", "create flashcards for", "I have a test about X",
"study cards for", "anki deck for", or any request to produce spaced-repetition material
from vault content.

## Dependencies

```bash
pip install genanki markdown --break-system-packages
```

The packaging script is at: `<vault>/scripts/anki_gen.py`

## Anki Setup (one-time, on user's machine)

1. Install **AnkiConnect** add-on in Anki: Tools → Add-ons → Get Add-ons → code `2055492159`
2. In AnkiConnect config (Tools → Add-ons → AnkiConnect → Config), ensure CORS allows local access:
   ```json
   { "webCorsOriginList": ["http://localhost", "app://obsidian.md"] }
   ```
3. Keep Anki Desktop running in the background when generating cards

The script auto-detects AnkiConnect. If Anki is running → pushes directly. If not → saves .apkg.

## Workflow

### 1. Identify Source Material

Check what's available for the requested topics, in priority order:

1. **Wiki pages** (`wiki/courses/`, `wiki/concepts/`) — already processed, best source
2. **Raw PDFs/PPTX** (`raw/papers/`, `raw/course-materials/`) — convert with `markitdown` first
3. **User-provided context** — if the user pastes or describes content directly

Read `index.md` to find relevant pages. If the topic has wiki pages, prefer those.
If only raw files exist, run `markitdown <file>` and work from the output.

### 2. Generate Cards as JSON

Create a JSON file following this schema:

```json
{
  "deck": "CS::<Course>::<Topic>",
  "description": "Cards for <context>",
  "cards": [
    {
      "type": "qa",
      "front": "Question in markdown",
      "back": "Answer in markdown",
      "extra": "Optional extra context, mnemonics, gotchas",
      "tags": ["course-tag", "topic-tag"]
    },
    {
      "type": "cloze",
      "text": "The {{c1::term}} does X in the context of Y.",
      "extra": "Optional note",
      "tags": ["course-tag", "topic-tag"]
    }
  ]
}
```

**Deck naming convention:** Use `::` for hierarchy. **No `CS::` prefix** — start with the course name directly. When generating cards for multiple topics (e.g., exam prep), **split into sub-decks by topic** so the user can study per-topic or from the parent deck.

- `Redes::Prova-01::01-Introducao-e-Conceitos-Basicos`
- `Redes::Prova-01::02-Camada-Fisica-Codificacao`
- `SO::Prova-02::01-Escalonamento`

Generate one JSON file per sub-deck and push each separately.

### 3. Package and Deliver

```bash
# Smart default: pushes to Anki if running, saves .apkg if not
python <vault>/scripts/anki_gen.py cards.json

# With images
python <vault>/scripts/anki_gen.py cards.json --media $TEMP/diagram1.png --media $TEMP/fig2.png

# Force push (fails if Anki not running)
python <vault>/scripts/anki_gen.py cards.json --push

# Force .apkg file
python <vault>/scripts/anki_gen.py cards.json -o <vault>/anki-output/<deck-name>.apkg

# Both: push AND save backup
python <vault>/scripts/anki_gen.py cards.json --push -o <vault>/anki-output/<deck-name>.apkg

# Check connection
python <vault>/scripts/anki_gen.py --status
```

**Preferred flow:** Just run with no flags. The script checks for AnkiConnect at `localhost:8765`:
- **Anki running** → pushes cards directly, auto-syncs to AnkiWeb, done
- **Anki not running** → saves `.apkg` in `anki-output/`, user imports later

Cards pushed via AnkiConnect are auto-tagged `anki-gen` for easy filtering.
Duplicate detection is built in (same deck scope) — running twice won't create duplicates.

---

## Card Generation Guidelines

### Philosophy

- **One fact per card.** Never cram multiple concepts into one card.
- **Context is king.** The student should know *why* something matters, not just *what* it is.
- **Test understanding, not recall of phrasing.** Rephrase from source — don't copy sentences with a blank.
- **Use both card types strategically:**
  - **Q&A** for "why" questions, mechanisms, comparisons, scenarios
  - **Cloze** for definitions, key terms, protocol fields, specific values, command syntax

### CS-Specific Card Patterns

#### Definitions → Cloze
```json
{
  "type": "cloze",
  "text": "{{c1::TCP}} is a {{c2::connection-oriented}} transport protocol that provides {{c3::reliable, ordered delivery}} of data.",
  "tags": ["redes", "tcp", "transport"]
}
```

#### Mechanisms / "How does X work?" → Q&A
```json
{
  "type": "qa",
  "front": "How does TCP handle packet loss?",
  "back": "TCP uses **retransmission timers**. If an ACK isn't received within the timeout period (RTO), the sender retransmits the segment.\n\nThe RTO is calculated dynamically using the **Jacobson/Karels algorithm**, which tracks smoothed RTT and RTT variance.",
  "extra": "Related: fast retransmit triggers after 3 duplicate ACKs, without waiting for timeout.",
  "tags": ["redes", "tcp", "reliability"]
}
```

#### Comparisons → Q&A with table
```json
{
  "type": "qa",
  "front": "Compare TCP and UDP on: connection, reliability, ordering, and overhead.",
  "back": "| Feature | TCP | UDP |\n|---------|-----|-----|\n| Connection | Connection-oriented (handshake) | Connectionless |\n| Reliability | Guaranteed (ACKs, retransmission) | Best-effort |\n| Ordering | Preserved (sequence numbers) | Not guaranteed |\n| Overhead | High (20-byte header min) | Low (8-byte header) |",
  "tags": ["redes", "tcp", "udp", "comparison"]
}
```

#### Command syntax → Cloze
```json
{
  "type": "cloze",
  "text": "To display the routing table on Linux, use: `{{c1::ip route show}}` (or legacy: `{{c2::route -n}}`)",
  "tags": ["redes", "linux", "routing", "commands"]
}
```

#### Protocol fields / headers → Cloze
```json
{
  "type": "cloze",
  "text": "The TCP header field {{c1::Window Size}} tells the sender how many bytes the receiver is willing to accept, enabling {{c2::flow control}}.",
  "tags": ["redes", "tcp", "header"]
}
```

#### Scenarios / "What happens when..." → Q&A
```json
{
  "type": "qa",
  "front": "What happens when a TCP sender receives 3 duplicate ACKs for the same segment?",
  "back": "It triggers **fast retransmit**: the sender immediately retransmits the missing segment without waiting for the retransmission timer to expire.\n\nThis is followed by **fast recovery** (RFC 5681): the congestion window is halved (instead of reset to 1 MSS as in timeout), allowing faster throughput recovery.",
  "tags": ["redes", "tcp", "congestion-control"]
}
```

#### Formulas / Equations → Q&A or Cloze
```json
{
  "type": "qa",
  "front": "What is the formula for TCP's estimated RTT, and what does each term mean?",
  "back": "$$EstimatedRTT = (1 - \\alpha) \\cdot EstimatedRTT + \\alpha \\cdot SampleRTT$$\n\n- **EstimatedRTT**: smoothed average of round-trip times\n- **SampleRTT**: the latest measured RTT for a segment\n- **α (typically 0.125)**: weight given to the new sample — small α means slower adaptation\n\nThe timeout (RTO) is then: $RTO = EstimatedRTT + 4 \\cdot DevRTT$",
  "tags": ["redes", "tcp", "rtt", "formula"]
}
```

#### Diagrams / Visual concepts → Q&A with ASCII or description
```json
{
  "type": "qa",
  "front": "Describe the TCP three-way handshake step by step.",
  "back": "```\nClient              Server\n  |--- SYN (seq=x) --->|\n  |<-- SYN-ACK --------|\n  |    (seq=y, ack=x+1)|\n  |--- ACK (ack=y+1) ->|\n  |    [connected]      |\n```\n\n1. Client sends **SYN** with initial sequence number\n2. Server responds with **SYN-ACK**, acknowledging client's seq and sending its own\n3. Client sends **ACK**, connection is established",
  "tags": ["redes", "tcp", "handshake"]
}
```

### Card Volume Guidelines

- **Exam prep (user says "I have a test"):** Generate 15-30 cards per topic. Cover breadth.
- **Deep dive (specific concept):** 8-15 cards. Go deeper with edge cases and "why" questions.
- **Quick review:** 5-10 cards. Key definitions and gotchas only.
- Ask the user if unsure about depth.

### Quality Checklist (self-review before writing JSON)

- [ ] Each card tests exactly ONE piece of knowledge
- [ ] No card answer is just "yes" or "no" — rephrase to ask "what/how/why"
- [ ] Cloze blanks are meaningful (not articles or prepositions)
- [ ] Code in cards is syntactically correct
- [ ] LaTeX compiles (balanced delimiters, valid commands)
- [ ] Tags follow kebab-case, match vault conventions
- [ ] No card duplicates information from another card
- [ ] Answers include enough context to be understood months later

### Language

- **Portuguese** for course content (matching the vault's convention)
- **English** for general CS concepts, RFCs, protocol names
- **Natural mix** is fine — "O TCP usa {{c1::three-way handshake}} para estabelecer conexao"
- When in doubt, match the language of the source wiki page

---

## Media and Images

**Note:** The vault's CLAUDE.md says "do NOT extract images from PDFs" — that rule applies to
wiki pages only. For Anki cards, extracting diagrams and figures is encouraged when they
genuinely help understanding.

### When to include images

Include an image in a card when:
- A diagram, flowchart, or state machine is central to the concept (e.g., TCP state machine)
- A figure from slides/papers explains a mechanism better than text ever could
- A comparison table with visual layout matters (network topologies, architecture diagrams)

Do NOT include images when:
- The concept is better explained with text, code, or a simple ASCII diagram
- The image is decorative or low-information (stock photos, title slides)
- A markdown table or bullet list conveys the same information

### Extracting images from PDFs

Use PyMuPDF (fitz) to extract or crop figures from PDF files. Both `fitz` and `Pillow`
are already installed.

**Extract embedded images from a page:**
```python
import fitz
doc = fitz.open("raw/papers/some-paper.pdf")
page = doc[3]  # page 4 (0-indexed)
images = page.get_images()
for i, img in enumerate(images):
    xref = img[0]
    pix = fitz.Pixmap(doc, xref)
    pix.save(f"$TEMP/anki-media/fig-{i}.png")
```

**Crop a specific region from a page (when the figure is part of the page layout):**
```python
import fitz
doc = fitz.open("raw/course-materials/slides.pdf")
page = doc[5]
# Coordinates: fitz.Rect(x0, y0, x1, y1) in points (72 pts = 1 inch)
# Use generous margins around the figure
clip = fitz.Rect(50, 100, 550, 400)
pix = page.get_pixmap(clip=clip, dpi=200)
pix.save("$TEMP/anki-media/cropped-diagram.png")
```

**How to find the right coordinates:**
1. Open the PDF, note the page number
2. Start with a large crop area covering most of the page
3. Refine by adjusting coordinates — use generous margins (add 20-30 pts padding)
4. Use `dpi=200` for sharp rendering on high-DPI screens

### Creating simple diagrams

When a concept would benefit from a visual but the source has no figure, generate one
using matplotlib or PIL:

```python
import matplotlib.pyplot as plt
import matplotlib.patches as patches

fig, ax = plt.subplots(figsize=(6, 3))
# Draw your diagram...
ax.set_xlim(0, 10); ax.set_ylim(0, 5)
ax.axis('off')
fig.savefig("$TEMP/anki-media/my-diagram.png", bbox_inches='tight',
            dpi=150, transparent=True)
plt.close()
```

Good candidates for generated diagrams:
- Protocol state machines (TCP states, handshakes)
- Layer diagrams (OSI model, protocol stacks)
- Simple flowcharts (algorithm steps, decision trees)
- Network topologies (star, ring, mesh)

### Referencing images in cards

Card JSON uses bare filenames — the script handles bundling/uploading:

```json
{
  "type": "qa",
  "front": "What does the TCP state machine look like?",
  "back": "<img src=\"tcp-states.png\">\n\nKey states: LISTEN, SYN-SENT, ESTABLISHED, FIN-WAIT...",
  "tags": ["redes", "tcp"]
}
```

Then pass all image files via `--media`:
```bash
python <vault>/scripts/anki_gen.py cards.json --media $TEMP/anki-media/tcp-states.png
```

### Workflow for cards with images

1. Create a temp directory for media: `mkdir -p $TEMP/anki-media`
2. Extract/create images, saving to `$TEMP/anki-media/`
3. Reference images by filename in card JSON (`<img src="filename.png">`)
4. Pass all images via `--media` flags when running the script
5. The script uploads (AnkiConnect) or bundles (.apkg) them automatically

### Code Screenshots
Don't use screenshots for code. Always use markdown code blocks — they render
properly in Anki with the card template's syntax highlighting.

---

## Example End-to-End Session

**User:** "I have a networks test next week covering TCP, UDP, and routing. Make me Anki cards."

**Claude's steps:**
1. Read `index.md` → find relevant pages:
   - `wiki/courses/2026-1/redes/tcp.md`
   - `wiki/courses/2026-1/redes/udp.md`
   - `wiki/courses/2026-1/redes/roteamento.md`
   - `wiki/concepts/tcp-ip.md`
2. Read each page
3. Generate ~60 cards (20 per topic) as JSON, mixing Q&A and Cloze
4. Write JSON to `$TEMP/redes-exam-cards.json`
5. Run: `python scripts/anki_gen.py $TEMP/redes-exam-cards.json`
   - If Anki is open → "Pushed 60 cards to CS::Redes::Camada-de-Transporte. They're ready to study."
   - If Anki is closed → "Saved redes-exam-cards.apkg — open Anki and import via File → Import."

**If source is a raw PDF (not yet in wiki):**
1. `markitdown raw/course-materials/slides-tcp.pptx > $TEMP/slides-tcp.md`
2. Read the markdown output
3. Scan the PDF for useful diagrams/figures using PyMuPDF
4. Extract good figures to `$TEMP/anki-media/`
5. Generate cards (text + image references) as JSON
6. Run: `python scripts/anki_gen.py $TEMP/cards.json --media $TEMP/anki-media/fig1.png --media $TEMP/anki-media/fig2.png`
7. Optionally offer to also create wiki topic pages (follows vault schema)

**If no good figure exists but a diagram would help:**
1. Generate a simple diagram with matplotlib (state machines, flowcharts, layer stacks)
2. Save to `$TEMP/anki-media/`
3. Reference in card JSON and pass via `--media`
