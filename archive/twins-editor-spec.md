# TWINS EDITOR – Complete Product & Functional Specification v2.1

## 1. Product Overview

### 1.1 Branding
- **Product Name**: TWINS EDITOR
- **Logo**: Display `icon-300.jpg` in header (hide gracefully if image fails to load)
- **Header Layout**: Logo + Title

### 1.2 Core Purpose
A dual-pane web tool for translating and proofreading Markdown, TXT, and JSON texts with advanced NLP analysis, real-time metrics, and version tracking.

---

## 2. Architecture & Layout

### 2.1 Core Layout
- **Two CodeMirror panes** (Markdown mode):
  - **Left Pane**: Read-only in normal mode
    - Can display: Source / Baseline / Empty
    - Header shows contextual label
    - Live Footer with metrics
  - **Right Pane**: Target (always editable)
    - Header shows file name
    - Live Footer with metrics

### 2.2 Architecture Principles
- Core skeleton (layout, styles, base events) separate from feature modules
- New functions ship as plugins/patches that don't break core
- Features delivered as modular, idempotent plugins
- Never remove existing features when adding new ones

---

## 3. Operating Modes

### 3.1 Available Modes
1. **Translation Mode**
   - Left pane: Source file (if opened), read-only
   - Right pane: Target file, editable
   - Line mirror and offset active
   
2. **Proofread Mode**
   - Left pane: Baseline (snapshot of Target), read-only
   - Right pane: Current Target, editable
   - Diff visualization active (replace/insert/delete)

### 3.2 Mode Switching Logic
- Mode toggle placed above left pane as: `Mode: [Translation] [Proofread]`
- Entering Proofread first time creates Baseline from current Target
- Returning to Translation shows Source (if exists) or Empty
- Mode switching never refreshes the Baseline

### 3.3 Baseline Persistence (v2.1 Update)
- **Creation**: Baseline created only on first Proofread entry this session
- **Persistence**: Baseline unchanged across mode toggles
- **Refresh**: Only after successful Save (verified by new SHA256 hash)
- **Association**: Baseline tied to saved hash from ## STATS block

---

## 4. UI Controls & Placement

### 4.1 Above Left Pane
- Mode toggle: Translation / Proofread
- Open Source button
- Offset controls: − / [number] / + / Reset
- Lock scroll checkbox

### 4.2 Global Toolbar
- Open Target
- Save (saves as *_ready.md)
- Theme (Light/Dark toggle)
- Undo
- Redo
- Grammar (toggle)
- NLP Details (opens sidebar)

### 4.3 Hotkeys
- **Cmd/Ctrl + S**: Save Target
- **Cmd/Ctrl + O**: Open Target

---

## 5. File Operations

### 5.1 Open Files
- Use `window.showOpenFilePicker` when available
- Fallback to `<input type="file">`
- Never clear editors on cancel/error
- Support: Markdown (.md), TXT (.txt), JSON (.json)

### 5.2 Save Target
1. Suggest filename: `originalname_ready.md`
2. Compute metrics for main block (before `---`)
3. Insert/update ## STATS block in tail
4. Insert/update ## Changes block in tail
5. Write file
6. Clear Target editor
7. Refresh Baseline to saved content
8. Update saved hash

### 5.3 File Structure
- **Main block**: Everything before first solo `---`
- **Tail**: Everything after first solo `---`
- NLP/Keywords/Stats operate on main block only
- Tail preserved on save

---

## 6. Alignment & Synchronization

### 6.1 Line Mirror
- Cursor in one pane highlights exactly one peer line in other pane
- Strict index-to-index mapping
- No partial line highlighting

### 6.2 Offset Control
- Formula: `Target line = Source line + offset`
- Controls: −, numeric input, +, Reset
- Bidirectional with clamping to valid ranges

### 6.3 Lock Scroll
- Two-way synchronized scrolling
- Cursor position sync respects offset
- Toggle via checkbox

---

## 7. Keyword Analysis

### 7.1 Extraction Sources
- **Inline format**: `Keywords: term1, term2, term3`
- **Section format**: `# Keywords` followed by list
- Separate keyword lists per pane

### 7.2 Overlay & Display
- Exact token/phrase matching (no partial matches)
- Live Footer shows chips with:
  - Occurrence count
  - Percentage of total words
  - Density color coding
- Click chip to highlight all occurrences

### 7.3 Density Color Buckets
- **Gray**: 0 ≤ p < 0.5% (too rare)
- **Blue**: 0.5% ≤ p ≤ 1.5% (under-used)
- **Green**: 1.6% ≤ p ≤ 2.9% (optimal)
- **Red**: p ≥ 3.0% (over-used)

---

## 8. Live Footer (Real-time Metrics)

### 8.1 Structure (4 rows per pane)
1. **Stats row**: chars / words / headings
2. **Keywords row**: Chips with count and % (colored)
3. **Bigrams row**: Top 12 bigrams
4. **Trigrams row**: Top 12 trigrams

### 8.2 Behavior
- Updates in real-time on text changes
- Computed from main block only
- Click any chip for red focus overlay
- Independent from ## STATS block

---

## 9. NLP Analysis

### 9.1 Signals
- Unique lemmas
- Bigrams (word pairs)
- Trigrams (word triplets)

### 9.2 Display
- **Footer preview**: Top 5 bigrams, top 5 trigrams
- **NLP Details sidebar**:
  - Top N lemmas
  - Top bigrams table
  - Top trigrams table
- Operates on main block only

---

## 10. STATS Block (File Metadata)

### 10.1 Format
```markdown
## STATS
- Characters (with spaces): <N>
- Words: <N>
- Headings count: <N>
- SHA256 (main text block): <hex>
- Made at (UTC): <ISO 8601>
```

### 10.2 Behavior
- Written to tail on Save
- SHA256 computed from main block only
- "Made at" timestamp (not "Parsed at")
- Hash serves as version identifier

---

## 11. Changes Block (Diff Log)

### 11.1 Format
```markdown
## Changes (proofread)
1. replace: `old` → `new`
2. insert: + `inserted text`
3. delete: − `deleted text`
```

### 11.2 Diff Source Logic
- **Translation mode**:
  - If Source exists: Source vs Target
  - Else: Baseline vs Target
- **Proofread mode**: Always Baseline vs Target

---

## 12. Diff Visualization

### 12.1 Colors
- **Replace**: Orange (both panes)
- **Insert**: Pink (right pane)
- **Delete**: Red with strikethrough (left pane)

### 12.2 Rules
- Only show real differences
- Never mark entire document as inserted
- Deletions must remain visible on left
- No highlights if Baseline equals Target

---

## 13. Grammar & Spellcheck

### 13.1 Spellcheck
- Browser native spellcheck enabled for Target

### 13.2 Grammar Toggle
- **Local checks**: 
  - Double spaces
  - Repeated punctuation
  - Adjacent repeated words
- **Optional**: LanguageTool API with silent fallback
- Visual: Red wavy underline for issues

---

## 14. Theme System

### 14.1 Options
- Light theme
- Dark theme
- Toggle button in toolbar
- Preference persisted in localStorage

### 14.2 Application
- CodeMirror theme
- Page-level CSS class

---

## 15. Development Requirements

### 15.1 Code Standards
- English-only in strings and comments
- Modular plugin architecture
- Idempotent patches
- No breaking changes to core

### 15.2 Dependencies
- CodeMirror 5.x
- Markdown mode
- Active-line addon

### 15.3 Browser APIs
- File System Access API (with fallback)
- localStorage for preferences

---

## 16. Acceptance Criteria

### 16.1 Core Functions
- [ ] Two-pane layout with CodeMirror
- [ ] Mode switching preserves content correctly
- [ ] Baseline persists until Save
- [ ] Save creates _ready.md files
- [ ] SHA256 hash updates on Save

### 16.2 Alignment
- [ ] Mirror highlights exactly one peer line
- [ ] Offset works bidirectionally
- [ ] Lock scroll synchronizes properly

### 16.3 Analysis
- [ ] Live Footer shows 4 rows updating in real-time
- [ ] Keywords extract from inline/section format
- [ ] Density colors apply correctly
- [ ] NLP ignores tail section

### 16.4 Diff & Save
- [ ] Proofread shows only real differences
- [ ] Colors: orange/pink/red as specified
- [ ] ## STATS block includes all fields
- [ ] ## Changes block formats correctly
- [ ] Tail preserved on Save

---

## 17. Configurable Parameters

| Parameter | Default | Options |
|-----------|---------|---------|
| Autosave interval | Disabled | 60-120 seconds |
| Keyword filter thresholds | None | Min/max % |
| NLP preview size | 5 bi, 5 tri | Adjustable |
| Sidebar top-N | 50 lemmas, 30 bi, 30 tri | Adjustable |
| Baseline refresh | On Save only | Manual option |

---

## 18. Usage Scenarios

### 18.1 Translation Workflow
1. Open Source file (left pane)
2. Open Target file (right pane)
3. Edit Target using alignment tools
4. Monitor keywords and metrics
5. Save → generates _ready.md with STATS

### 18.2 Proofread Workflow
1. Open Target file
2. Switch to Proofread mode
3. Baseline created from Target
4. Edit Target seeing diffs
5. Save → updates with new hash

### 18.3 Direct Proofread (No Source)
1. Open Target directly
2. Enter Proofread mode
3. Left shows Baseline from Target
4. Edit and save as above

---

## 19. Edge Cases & Error Handling

### 19.1 File Operations
- Handle picker cancel gracefully
- Fallback for unsupported APIs
- Preserve content on errors

### 19.2 Empty States
- Empty Target → empty Baseline
- No Source → left pane shows "— empty —"
- No keywords → empty keyword row

### 19.3 Diff Edge Cases
- Identical texts → no highlights
- Large insertions → proper visualization
- Multiple deletions → all visible on left

---

## 20. Future Enhancement Suggestions

### 20.1 Immediate Improvements
1. **JSON Support Enhancement**
   - Specialized JSON diff view
   - Key-path based alignment
   - Schema validation

2. **Export Options**
   - PDF generation with diff highlights
   - HTML preview mode
   - Combined report generation

3. **Collaboration Features**
   - Comment annotations
   - Change attribution
   - Version history browser

### 20.2 Advanced Features
1. **AI Integration**
   - Translation suggestions
   - Grammar improvement hints
   - Consistency checking

2. **Project Management**
   - Multi-file projects
   - Batch processing
   - Translation memory

3. **Quality Metrics**
   - Readability scores
   - Consistency analysis
   - Style guide compliance

### 20.3 Performance Optimizations
1. **Large File Handling**
   - Virtual scrolling
   - Lazy loading
   - Background processing

2. **Caching Strategy**
   - Persist analysis results
   - Incremental updates
   - Web Workers for NLP

---

## Appendix A: Technical Implementation Notes

### CodeMirror Configuration
```javascript
// Basic setup for both panes
{
  mode: 'markdown',
  lineNumbers: true,
  lineWrapping: true,
  readOnly: false, // true for left pane
  styleActiveLine: true,
  theme: 'default' // or 'dark'
}
```

### File Structure Example
```markdown
# Main content here
This is the main block used for analysis...

---

## STATS
- Characters (with spaces): 1234
- Words: 234
- Headings count: 3
- SHA256 (main text block): abc123...
- Made at (UTC): 2024-01-15T10:30:00Z

## Changes (proofread)
1. replace: `old text` → `new text`
```

### Storage Schema
```javascript
{
  theme: 'light|dark',
  lastBaseline: 'full text snapshot',
  lastHash: 'sha256 hash',
  keywords: {
    source: ['term1', 'term2'],
    target: ['term3', 'term4']
  },
  preferences: {
    lockScroll: true,
    grammar: false,
    offset: 0
  }
}
```