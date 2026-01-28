# TWINS EDITOR — Product & Functional Specification v2.3

## 1. Product Overview

### 1.1 Branding
- **Product Name**: TWINS EDITOR
- **Logo**: VVDGT gradient logo (purple `#5A24F2` to magenta `#A530E8`)
- **Logo file**: `logo.svg` in project root
- **Header Layout**: Logo + Title in single line with all controls

### 1.2 Core Purpose
A dual-pane web tool for translating and proofreading Markdown, TXT, and JSON texts with NLP analysis, real-time metrics, and version tracking.

### 1.3 Technology Stack
- Single-file HTML application (self-contained)
- CodeMirror 5.65.5 (loaded from CDN)
- Native browser spellcheck
- No build process required

---

## 2. Architecture & Layout

### 2.1 Core Layout
- **Two CodeMirror panes** (Markdown mode):
  - **Left Pane**: Source / Baseline / Empty (read-only)
    - Header shows contextual label (SOURCE: / BASELINE:)
    - Live Footer with metrics
  - **Right Pane**: Target (always editable)
    - Header shows file name
    - Live Footer with metrics

### 2.2 Architecture Principles
- Core skeleton (layout, styles, base events) separate from feature modules
- Features delivered as modular, idempotent plugins
- Never remove existing features when adding new ones
- All code/comments in English

---

## 3. Operating Modes

### 3.1 Translation Mode
- Left pane: Source file (if opened), read-only
- Right pane: Target file, editable
- Line mirror and offset active
- Button: "Open Source" visible
- Changes tracked against original target (baseline stored on target open)

### 3.2 Proofread Mode
- Left pane: Baseline file (explicitly loaded), read-only
- Right pane: Current Target, editable
- Diff visualization active (replace/insert/delete)
- Button: "Open Baseline" visible
- Changes tracked against loaded baseline

### 3.3 Mode Switching Logic
- Mode toggle: `Mode: [Translation] [Proofread]`
- Switching modes changes left pane content and available buttons
- Each mode maintains its own file state (source vs baseline)

### 3.4 Baseline Handling
- **Translation mode**: Original target content stored as baseline when file opened
- **Proofread mode**: Baseline loaded explicitly via "Open Baseline" button
- Baseline used for diff visualization and CHANGES block generation

---

## 4. UI Controls & Placement

### 4.1 Header Toolbar (Single Line)
All controls in one horizontal line:

| Group | Controls |
|-------|----------|
| Brand | Logo + "TWINS EDITOR" title |
| Mode | Toggle: Translation / Proofread |
| Edit | Undo, Redo, Save (primary button) |
| Alignment | Offset: − / [number] / + / Reset, Lock Scroll checkbox |
| Language | Spell Check checkbox, Language selector, Grammar checkbox |
| View | Theme toggle, NLP Details button |

### 4.2 Pane Headers
- **Left**: Open Source/Baseline button, Label (SOURCE:/BASELINE: + filename), Close button
- **Right**: Open Target button, Label (TARGET: + filename), Close button

### 4.3 Hotkeys
- **Cmd/Ctrl + S**: Save Target
- **Cmd/Ctrl + O**: Open Target

---

## 5. File Operations

### 5.1 Supported Formats
- Markdown (.md)
- Plain text (.txt)
- JSON (.json)
- TOML (.toml) — for CMS content files
- YAML (.yaml, .yml) — for configuration files

### 5.2 Open Files
- Uses `<input type="file">` for file selection
- On cancel/error: editors not cleared
- Welcome text cleared on first file open

### 5.3 Save Target

**For Markdown/Text/JSON:**
1. Extract main block (content before first `---`)
2. Generate ## STATS block with metrics
3. Generate ## CHANGES block with diff summary
4. Combine: main block + `---` + STATS + CHANGES
5. Download as `originalname_ready.md`

**For TOML/YAML:**
1. Save content as-is (no stats/changes sections added)
2. Download as `originalname_ready.toml` or `.yaml`

Both modes:
- Clear all editors and show welcome text
- Show success alert

### 5.4 File Structure
```
[Main content block]
Everything before first solo ---
This is where you write/edit

---

## STATS
[Auto-generated metrics]

## CHANGES
[Auto-generated diff log]
```

---

## 6. Alignment & Synchronization

### 6.1 Line Mirror
- Cursor in one pane highlights corresponding line in other pane
- Highlight class: `cm-mirror-line`
- Strict index-to-index mapping with offset

### 6.2 Offset Control
- Formula: `Target line = Source line + offset`
- Controls: − (decrease), numeric input, + (increase), Reset
- Bidirectional with clamping to valid ranges

### 6.3 Lock Scroll
- Checkbox toggle
- Two-way synchronized scrolling when enabled
- Scroll position synced between panes

---

## 7. Keyword Analysis

### 7.1 Extraction Sources
- **Inline format**: `Keywords: term1, term2, term3`
- **Section format**: `# Keywords` followed by list items
- Separate keyword lists per pane

### 7.2 Display
- Footer shows keyword chips with:
  - Keyword text
  - Occurrence count
  - Percentage of total words
  - Density color coding
- Click chip to highlight all occurrences (yellow background)

### 7.3 Density Color Buckets

**Single-word keywords:**
| Color | Range |
|-------|-------|
| Gray | < 1.0% |
| Blue | 1.0% - 1.9% |
| Green | 2.0% - 2.9% |
| Red | >= 3.0% |

**Two-word keywords (bigrams):**
| Color | Range |
|-------|-------|
| Gray | < 0.5% |
| Blue | 0.5% - 0.9% |
| Green | 1.0% - 1.9% |
| Red | >= 2.0% |

**Three-word keywords (trigrams):**
| Color | Range |
|-------|-------|
| Gray | < 0.3% |
| Blue | 0.3% - 0.6% |
| Green | 0.7% - 0.9% |
| Red | >= 1.0% |

---

## 8. Live Footer

### 8.1 Structure (2 rows per pane)
1. **Stats row**: Characters / Words / Headings count
2. **Keywords row**: Chips with count and % (colored by density)

### 8.2 Behavior
- Updates in real-time on text changes (300ms debounce)
- Click any chip for yellow highlight overlay
- Independent from ## STATS block

### 8.3 Statistics by File Type

| File Type | Characters/Words counted from | Headings count |
|-----------|------------------------------|----------------|
| Markdown | Main block (before `---`) | `#` headings |
| Text | Main block (before `---`) | `#` headings |
| JSON | Main block (before `---`) | `#` headings |
| TOML | String values only (not keys) | `title`, `h1`-`h6` keys |
| YAML | String values only (not keys) | `title`, `h1`-`h6` keys |

**TOML/YAML value extraction**: Only text content that appears on the rendered page is counted — keys, brackets, and structural syntax are excluded.

**Heading keys**: For CMS files, headings are counted from semantic keys (`title`, `h1`, `h2`, etc.) rather than structural sections.

---

## 9. NLP Analysis Sidebar

### 9.1 Activation
- Click "NLP" button in toolbar
- Sidebar slides in from right (500px width)

### 9.2 Content (3 sections)
- **Top Lemmas (40)**: Words with count and density %
- **Top Bigrams (40)**: Word pairs with count and density %
- **Top Trigrams (40)**: Word triplets with count and density %

### 9.3 Features
- Click any item to highlight occurrences in right pane
- Keyword matches highlighted with border
- Density colors applied to percentages
- Stop words filtered out (the, a, an, and, or, etc.)

---

## 10. STATS Block (Auto-generated on Save)

### 10.1 Format
```markdown
## STATS
- Characters (with spaces): <N>
- Words: <N>
- Headings count: <N>
- SHA256 (main text block): <hash>
- Made at (UTC): <ISO 8601>
```

### 10.2 Notes
- Written to tail after `---` separator
- Hash computed from main block only (simple hash function)
- Timestamp in ISO 8601 format

---

## 11. CHANGES Block (Auto-generated on Save)

### 11.1 Format
```markdown
## CHANGES

### Removed
Line N: − `deleted word`

### Added
Line N: + `added word`

### Replaced
Line N: `old` → `new`

### Overall
Difference: X.X% / Changes: N
```

### 11.2 Diff Source
- Compares current target against baseline
- Line-by-line word-level comparison
- Shows up to 10 items per category

---

## 12. Diff Visualization (Proofread Mode)

### 12.1 Colors
| Type | Color | Location |
|------|-------|----------|
| Replace | Orange (`rgba(255, 152, 0, 0.5)`) | Both panes |
| Insert | Pink (`rgba(255, 192, 203, 0.5)`) | Right pane |
| Delete | Red + strikethrough (`rgba(244, 67, 54, 0.5)`) | Left pane |

### 12.2 Rules
- Only show real differences (word-level)
- Deletions visible with strikethrough on left
- No highlights if baseline equals target
- Updates on every text change (300ms debounce)

---

## 13. Spell Check

### 13.1 Implementation
- Native browser spellcheck (not external dictionary)
- Checkbox toggle in toolbar
- Language selector dropdown

### 13.2 Supported Languages
- English, German, Spanish, French, Italian, Portuguese, Polish, Dutch, Russian

### 13.3 Behavior
- Red wavy underline for misspelled words
- Only active on Target pane
- Language attribute set on editor element

---

## 14. Grammar Check

### 14.1 Local Checks
- Double spaces
- Repeated consecutive words
- Multiple punctuation (except `...`, `?!`, `!?`)
- Missing space after punctuation
- Lowercase after sentence-ending punctuation
- Common misspellings (recieve, beleive, occured, etc.)
- Unclosed quotes
- Unmatched parentheses

### 14.2 Visual Feedback
- Red wavy underline for errors
- Orange underline for capitalization
- Blue underline for spacing issues

---

## 15. Theme System

### 15.1 Options
- Light theme (default CodeMirror)
- Dark theme (Monokai)

### 15.2 Persistence
- Stored in localStorage
- Applied on page load
- Toggle button in toolbar

### 15.3 CSS Variables
```css
--bg-primary, --bg-secondary, --bg-tertiary
--text-primary, --text-secondary
--border-color
--accent, --success, --warning, --error
--keyword-gray, --keyword-blue, --keyword-green, --keyword-red
```

---

## 16. Welcome Text

### 16.1 Behavior
- Shown on app start in both panes
- Demonstrates all features with live statistics
- Cleared on first file open or user edit
- Restored after successful save

### 16.2 Content
- Complete feature documentation
- Sample keywords for density demonstration
- Workflow examples

---

## 17. State Management

### 17.1 Global State Object
```javascript
state = {
  mode: 'translation' | 'proofread',
  sourceContent: string,
  targetContent: string,
  baselineContent: string,
  sourceFileName: string,
  targetFileName: string,
  baselineFileName: string,
  targetFileExtension: string,
  offset: number,
  lockScroll: boolean,
  theme: 'light' | 'dark',
  grammarEnabled: boolean,
  spellCheckLanguage: string,
  welcomeActive: boolean,
  keywords: { left: [], right: [] },
  nlpData: { left: {...}, right: {...} }
}
```

### 17.2 Persistence
- Theme preference: localStorage
- All other state: session only (lost on page refresh)

---

## 18. Dependencies

### 18.1 External (CDN)
- CodeMirror 5.65.5 (codemirror.min.js)
- CodeMirror Markdown mode
- CodeMirror TOML mode
- CodeMirror YAML mode
- CodeMirror Active Line addon
- CodeMirror Monokai theme

### 18.2 Browser APIs
- localStorage (theme preference)
- Blob/URL for file download
- Native spellcheck

---

## 19. Acceptance Criteria

### 19.1 Core Functions
- [x] Two-pane layout with CodeMirror
- [x] Mode switching (Translation/Proofread)
- [x] Baseline handling per mode
- [x] Save creates `_ready.md` files
- [x] Hash computed on save

### 19.2 Alignment
- [x] Mirror highlights peer line
- [x] Offset works bidirectionally
- [x] Lock scroll synchronizes panes

### 19.3 Analysis
- [x] Live Footer shows stats and keywords
- [x] Keywords extract from inline/section format
- [x] Density colors applied
- [x] NLP sidebar with lemmas, bigrams, trigrams

### 19.4 Diff & Save
- [x] Proofread shows word-level differences
- [x] Colors: orange/pink/red as specified
- [x] STATS block with all fields
- [x] CHANGES block with categorized diffs

---

## 20. Known Limitations

1. **Hash function**: Uses simple hash, not cryptographic SHA256
2. **File API**: Only `<input type="file">`, no File System Access API
3. **No persistence**: State lost on page refresh (except theme)
4. **No autosave**: Manual save only
5. **Baseline in Proofread**: Must be loaded explicitly, not auto-created

---

## 21. Future Enhancements

### 21.1 Planned Improvements
- Real SHA256 hash implementation
- File System Access API for better save experience
- Autosave with configurable interval
- Bigrams/Trigrams in footer preview
- Auto-create baseline on Proofread entry

### 21.2 Advanced Features
- AI translation suggestions
- Translation memory
- Multi-file projects
- Export to PDF/HTML
- Collaboration features

---

## Appendix A: Code Examples

### CodeMirror Configuration
```javascript
{
  mode: 'markdown',
  lineNumbers: true,
  lineWrapping: true,
  readOnly: false, // true for left pane
  styleActiveLine: true,
  theme: 'default', // or 'monokai'
  inputStyle: 'contenteditable',
  spellcheck: true // target only
}
```

### File Structure Example
```markdown
# Document Title

Main content here...

Keywords: term1, term2, term3

---

## STATS
- Characters (with spaces): 1234
- Words: 234
- Headings count: 3
- SHA256 (main text block): abc123...
- Made at (UTC): 2024-01-15T10:30:00Z

## CHANGES

### Removed
Line 5: − `old`

### Added
Line 7: + `new`

### Overall
Difference: 2.5% / Changes: 12
```

---

**Version**: v2.3 — Consolidated specification reflecting actual implementation (app v9.6.3)
