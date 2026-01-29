# TWINS EDITOR v2.4

A dual-pane web tool for translating and proofreading Markdown, TXT, JSON, TOML and YAML texts with NLP analysis, real-time metrics, and version tracking.

## Features

- **Translation Mode** — Side-by-side source and target editing with line alignment and offset controls
- **Proofread Mode** — Compare current text against baseline with word-level diff visualization
- **Multi-format Support** — Markdown, Text, JSON, TOML, YAML with syntax highlighting
- **Real-time NLP Analysis** — Keywords, bigrams, trigrams, word density metrics
- **Live Metrics** — Character count, word count, heading/section count updated in real-time
- **Smart Statistics** — For TOML/YAML, counts only content values (not keys or structure)
- **Spell Check** — Native browser spellcheck with multi-language support
- **Grammar Check** — Double spaces, repeated words, punctuation issues
- **Theme Support** — Light and dark themes with persistence

## Files

| File | Description |
|------|-------------|
| `index.html` | Main application (single-file, self-contained) |
| `twins-editor-spec-v2.4.md` | Complete product & functional specification |

## Quick Start

1. Open `index.html` in a modern browser (or visit GitHub Pages)
2. Select mode: **Translation** (source + target) or **Proofread** (baseline + target)
3. Load files using Open buttons in pane headers
4. Edit target text with real-time metrics in footer
5. Save generates `*_ready.md` with embedded STATS and CHANGES

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Cmd/Ctrl + S` | Save target file |
| `Cmd/Ctrl + O` | Open target file |

## Modes

### Translation Mode
- Left pane: Source document (read-only reference)
- Right pane: Target document (editable)
- Line mirroring with configurable offset
- Lock scroll for synchronized navigation

### Proofread Mode
- Left pane: Baseline document (loaded explicitly)
- Right pane: Current document (editable)
- Word-level diff highlighting:
  - **Orange** — replaced words
  - **Pink** — added words
  - **Red strikethrough** — deleted words

## NLP Sidebar

Click **NLP** button to open detailed analysis:
- Top 40 lemmas with frequency
- Top 40 bigrams (word pairs)
- Top 40 trigrams (word triplets)
- Keyword matches highlighted
- Click any term to highlight in text

## Dependencies

- CodeMirror 5.65.5 (loaded from CDN)
- Modern browser with native spellcheck support

## Archive

The `archive/` folder contains previous specification versions (v1, v2, v2.1, v2.2) kept for historical reference.

---

**Version**: 2.4
**Last Updated**: January 2025
