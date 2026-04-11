---
name: summarize
description: Summarize any content (YouTube video, podcast, web article, PDF paper, book/EPUB) into a rich wiki note following vault schema. Creates concept pages for new terms, updates index.md and log.md. Use when the user provides a URL, file path, or content to summarize.
user_invocable: true
---

# Summarize

Universal content summarizer adapted for this vault at `~/Documents/artur/` (auto-detected via `.obsidian`). Takes a URL, file path, or pasted text and produces a structured wiki note with cross-linked concept pages, then updates `index.md` and `log.md`.

## Content Type -> Location Mapping

| Input | Location | type | tags |
|---|---|---|---|
| YouTube video (educational) | `wiki/videos/<title>.md` | `video` | `[video, <topics>]` |
| YouTube video (bookmark/casual) | `wiki/videos/<title>.md` | `video` | `[video, bookmark]` |
| Podcast episode | `wiki/videos/<title>.md` | `podcast` | `[video, podcast, <topics>]` |
| Web article / blog post | `wiki/web/<title>.md` | `web` | `[web, <topics>]` |
| Web bookmark (casual save) | `wiki/web/<title>.md` | `web` | `[web, bookmark]` |
| PDF research paper | `wiki/papers/<title>.md` | `paper` | `[paper, <topics>]` |
| Book / EPUB | `wiki/books/<title>.md` | `book` | `[book, <topics>]` |

## Required Tools

| Tool | Purpose | Install |
|---|---|---|
| `markitdown` | PDF, PPTX, DOCX, EPUB, HTML -> Markdown | already installed |
| `yt-dlp` | YouTube/podcast metadata + subtitles | `pip install yt-dlp` |
| `defuddle` | Clean web article extraction | `npm install -g defuddle` |

## Step 0: Bootstrap check

### 0a. Resolve vault root

Walk up from `$PWD` looking for `.obsidian/`. Fallback to `~/Documents/artur`.

```bash
vault=""
dir="$PWD"
while [ "$dir" != "/" ] && [ "$dir" != "." ]; do
  [ -d "$dir/.obsidian" ] && vault="$dir" && break
  dir="$(dirname "$dir")"
done
[ -z "$vault" ] && vault="$HOME/Documents/artur"
echo "Vault: $vault"
```

### 0b. Ensure new folders exist

```bash
mkdir -p "$vault/wiki/videos" "$vault/wiki/books"
```

### 0c. Check tools

For each missing tool (markitdown, yt-dlp, defuddle), ask the user before installing.

## Step 1: Detect content type and extract text

### YouTube video or podcast (URL provided)

Fetch metadata and auto-subtitles with yt-dlp:

```bash
yt-dlp --cookies-from-browser chrome \
  --print "%(id)s|%(title)s|%(duration)s|%(upload_date)s|%(channel)s" \
  --no-download "URL"

yt-dlp --cookies-from-browser chrome \
  --write-auto-sub --sub-lang en --sub-format json3 \
  --skip-download -o "/tmp/summarize/%(id)s" "URL"
```

If auto-subs exist, extract transcript text from the JSON3 file.

If no subs or quality is poor, download audio and transcribe. Ask the user:
- **Local:** `pip install mlx-whisper` then `mlx_whisper audio.mp3`
- **ElevenLabs Scribe:** requires `ELEVENLABS_API_KEY`

### Web article / blog post (URL provided)

```bash
defuddle parse "URL" --md -o /tmp/summarize/article.md
defuddle parse "URL" -p title
defuddle parse "URL" -p domain
```

Fallback if defuddle unavailable: `markitdown "URL" > /tmp/summarize/article.md`

### Web clip already in `raw/clips/`

Read directly — clips are Markdown from the Obsidian Web Clipper.

### PDF paper (in `raw/papers/` or provided path)

```bash
markitdown "$vault/raw/papers/filename.pdf" > /tmp/summarize/paper.md
```

### Book / EPUB

```bash
markitdown "path" > /tmp/summarize/book.md
```

## Step 2: Determine summary depth

| Source word count | Source examples | Target summary words | Sections |
|---|---|---|---|
| <1,500 | 5-min video, short article | 200-400 | 1-2 |
| 1,500-5,000 | 10-20 min video, blog post | 500-1,200 | 3-5 |
| 5,000-15,000 | 30-60 min video, whitepaper | 1,500-3,000 | 5-8 |
| 15,000-40,000 | 1-3 hr video/podcast, long paper | 3,000-6,000 | 8-15 |
| 40,000-80,000 | Short book, multi-hour series | 5,000-10,000 | 15-25 |
| 80,000+ | Full book | 8,000-15,000 | 20-40 |

**Videos/podcasts:** estimate ~150 words/min conversational, ~120 for interviews, ~170 for scripted.

**Bookmark depth:** user says "bookmark" or it is a casual save -> just a `> [!tldr]` + 2-3 bullet takeaways. No sections.

**Books:** each chapter gets its own `## Chapter N: Title` section (300-600 words each). Never batch chapters into single paragraphs.

## Step 3: Assemble the summary note

### Frontmatter by type

**video:**
```yaml
type: video
title: "<Title>"
url: "<YouTube URL>"
channel: "<Channel Name>"
duration: <seconds>
uploaded: <YYYY-MM-DD>
tags: [video, <topic-tags>]
concepts: ["[[wiki/concepts/<term>|<Term>]]"]
```

**podcast:**
```yaml
type: podcast
title: "<Episode Title>"
url: "<URL>"
show: "<Show Name>"
duration: <seconds>
tags: [video, podcast, <topic-tags>]
concepts: ["[[wiki/concepts/<term>|<Term>]]"]
```

**web:**
```yaml
type: web
title: "<Title>"
url: "<original URL>"
source: "[[raw/clips/<filename>]]"   # only if from web clipper
date: <YYYY-MM-DD>
tags: [web, <topic-tags>]
concepts: ["[[wiki/concepts/<term>|<Term>]]"]
# For music bookmarks:
# artist: "<Artist>"
# genre: "<genre>"
```

**paper:**
```yaml
type: paper
title: "<Title>"
authors: [<Author 1>, <Author 2>]
year: <year>
source: "[[raw/papers/<filename>.pdf]]"
tags: [paper, <topic-tags>]
concepts: ["[[wiki/concepts/<term>|<Term>]]"]
```

**book:**
```yaml
type: book
title: "<Title>"
author: "<Author>"
year: <year>
source: ""   # URL or [[raw/...]] path if file is in vault
tags: [book, <topic-tags>]
concepts: ["[[wiki/concepts/<term>|<Term>]]"]
```

### Body structure

No `# Title` heading — Obsidian shows filename as title.

```
> [!tldr]
> 2-5 sentences. What it is, who made it, core argument/takeaway.

## Section 1 Title

Summary paragraphs. Wikilink every concept, technique, tool, and named idea:
[[wiki/concepts/<term>|<Term>]]. Use > [!quote] for notable quotes.
Use Mermaid for flowcharts/pipelines — never extract images from PDFs.

## Section 2 Title

[...]
```

### Formatting rules

1. **No `# Title` heading** — filename is the title in Obsidian
2. **Full-path wikilinks always:** `[[wiki/concepts/neural-networks|Neural Networks]]`, never `[[neural-networks]]`
3. **`> [!tldr]`** callout is mandatory — every note starts with one
4. **`> [!quote]`** callouts for notable quotes (with speaker and timestamp if available)
5. **Mermaid** for diagrams/pipelines — no image extraction
6. **Language:** Portuguese for course-related content, English for research/web/video
7. **File naming:** kebab-case, lowercase, no accents (e.g., `andrej-karpathy-lets-build-gpt`)

### Long content (>3,000 source words)

Dispatch parallel **Opus** subagents — one per section — each with the section text, a word count target from the Step 2 table, and instruction to use full-path wikilinks.

**Never use Haiku or Sonnet for summarization. Always Opus.**

## Step 4: Create concept pages for new terms

### 4a. Extract wikilinks from the new note

```bash
grep -o "\[\[wiki/concepts/[^]|]*" note_path | sed 's|\[\[wiki/concepts/||' | sort -u
```

### 4b. Check which concept pages already exist

For each extracted term, check if `wiki/concepts/<term>.md` exists. **Always run this check. Never assume a concept page exists.**

### 4c. Create missing concept pages

Location: `wiki/concepts/<kebab-case-term>.md`

```yaml
type: concept
tags: [concept, <domain-tags>]
courses: []
papers: []
```

Body: 2-4 sentence plain-language explanation. Cross-link to related concepts with full-path wikilinks: `[[wiki/concepts/<related>|<Related>]]`.

For >8 missing concepts, dispatch parallel **Opus** subagents in batches.

### 4d. Verify — zero dangling wikilinks

Re-run Step 4b. The note is not done until all concept pages exist.

## Step 5: Update index.md and log.md

### index.md

Add the new note under the correct section:

```
- [[wiki/videos/<name>|<Display Title>]] — one-line description
```

| Content type | index.md section |
|---|---|
| `video` (non-bookmark) | `## Videos` |
| `video` (bookmark) | `## Videos` > `### Bookmarks` |
| `podcast` | `## Videos` |
| `web` (non-bookmark) | `## Web` |
| `web` (bookmark) | `## Web` > `### Bookmarks` |
| `paper` | `## Papers` |
| `book` | `## Books` |

Create the section if it does not exist yet.

### log.md

Prepend a new entry after the frontmatter:

```
## [YYYY-MM-DD] ingest | <Title>
<Content type> summary. Key topics: [[wiki/concepts/<term>|<Term>]], [...]. New concept pages: [[wiki/concepts/<new>|<New>]], [...].
```

## Model usage

| Task | Model |
|---|---|
| Section summarization | **Opus** subagents (parallel) |
| Concept page creation | **Opus** subagents (parallel batches) |
| **Never** | Haiku or Sonnet |
