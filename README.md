# Wikidata annotation for PDF 

A single-file web app to annotate a PDF and turn your annotations into
[QuickStatements](https://quickstatements.toolforge.org/) commands for Wikidata.

You select text in a PDF, match it to Wikidata items, link the items with
arrows, and copy the resulting statements. Every statement automatically gets a
reference ("stated in" + page + "retrieved").

- One file: `pdf-to-wikidata.html`. No installation, no account, no server.
- Your PDF never leaves your computer. Only the text you search for is sent to
  Wikidata's public search.
- Requires an internet connection (Wikidata search and the PDF library).
- The interface is in English.

---

## How to use it

### 1. Start
1. Open `pdf-to-wikidata.html` in a recent browser (Chrome, Edge or Firefox).
2. Click **Open PDF**, or drop a PDF file on the page.
3. Enter the **Source QID**: the Wikidata item of the book or article you are
   annotating (for example `Q12345`).
4. Enter the **First page number**: the number printed on the first page of the
   PDF. It is not always 1 (for example, a chapter that starts on page 57).

The app calculates the printed page number of every PDF page from this.

### 2. Match text to Wikidata items
1. Select a word or phrase in the PDF with your mouse.
2. A search box opens with your selection already filled in.
3. Click the right result, or press **Enter** to take the first one. The text
   is now highlighted in blue.

If the thing you selected is not a Wikidata item but a value (a date, a number
written as text, a URL), use **Add value** in the same box and choose its type:

| Type | What it produces |
|------|------------------|
| Text | A text string |
| URL  | A URL string |
| Date | A date. Write `2020`, `2020-05` or `2020-05-17`; the precision (year, month, day) follows what you write. Dates like `May 17, 2020` are also recognised. |

Values are highlighted in green. A value can only be the target of a statement,
never the subject.

### 3. Link items to build statements
1. Click a blue highlight to select it.
2. Click **Link to…** in the right panel.
3. Click the highlighted text the statement points to.
4. Search and choose the **property** (for example "place of birth").

An arrow appears from the first text to the second, labelled with the property.
The same text can be linked as many times as you like, so one selection can
give several statements.

To keep the page readable, only the arrows connected to the text or statement
you have selected are shown. Click empty space or press **Esc** to deselect.

### 4. Check and edit statements
The right panel lists your statements. For each one you can:
- click it to see its arrow and jump to its place in the PDF;
- edit its **page(s)** (see below);
- delete it.

To remove a highlight and all its statements, select it and click **Delete**.

### 5. Export to Wikidata
- **Copy commands** copies the QuickStatements commands.
- **Open in QuickStatements** opens QuickStatements in a new tab with the
  commands already filled in. The commands are also copied to your clipboard;
  if the page opens empty, paste them with Ctrl+V.

Then run the batch in QuickStatements while logged in to Wikidata. Always read
the batch before running it.

### What each statement contains
```
Q42|P19|Q350|S248|Q12345|S304|"57"|S813|+2026-11-24T00:00:00Z/11
```
- the statement itself (subject, property, value);
- **stated in** (`S248`): your source QID;
- **page(s)** (`S304`): the page of the texts you linked. If they are on two
  pages, both are included (for example `57-58`);
- **retrieved** (`S813`): today's date.

No quotation is added, to avoid copying copyrighted text.

### Zoom, saving and resuming
- **− / +** zoom the PDF. Your highlights follow.
- Your work is saved automatically in your browser, for each PDF (same name and
  size). Reopen the PDF in the same browser and your annotations come back.
- **Export project** saves your annotations in a `.wdproject.json` file.
  **Import project** loads one into the PDF you have open. Use this to move to
  another computer or browser, or to keep a backup.

### Limitations
- Scanned PDFs do not work: the PDF needs real, selectable text.
- The app cannot create new Wikidata items, only link to existing ones.
- It does not check whether a statement already exists on Wikidata.
- Quantities and coordinates are not supported as values.
- Clearing your browser data erases the automatic save. Export your project to
  keep a copy.

---

## How it works (technical)

### Architecture
A single HTML file with inline CSS and JavaScript, no build step and no
framework. The only external dependency is
[pdf.js](https://mozilla.github.io/pdf.js/) 3.11.174, loaded from cdnjs. The
only network calls are the pdf.js library and Wikidata search requests.

### PDF display
- Each page is a `div` containing a canvas (rendering), a text layer (pdf.js
  `renderTextLayer`, transparent text used for selection) and a highlight layer.
- Pages are created with their final size first, then drawn lazily with an
  `IntersectionObserver` (800 px margin), so large PDFs open quickly.
- Zoom changes the scale, rebuilds the pages and restores the scroll position.
  The default scale 1.4 is shown as 100%.

### Annotations
- On mouse release, the browser selection's client rectangles are converted to
  PDF-page coordinates (divided by the scale), so highlights survive zooming. A
  selection can cover several pages; its rectangles are grouped by page.
- A **mention** has: an id, a kind (`item`, `text`, `url` or `date`), the
  selected text, its rectangles per page, and either a Wikidata entity (id,
  label, description) or a value.
- A **statement** has: an id, the subject mention, a property (id, label), the
  target mention and an optional page override.

### Wikidata search
- Calls `wbsearchentities` on `wikidata.org/w/api.php` with `origin=*` (CORS),
  English labels, 8 results, `type=item` or `type=property`.
- Typing is debounced (250 ms), earlier requests are cancelled
  (`AbortController`) and results are cached for the session.
- The eight most recently used properties are kept in `localStorage` and shown
  when the property box is empty.

### Arrows
One SVG overlay covers the whole page stack. For the selected mention or
statement, a quadratic curve joins the centres of the first rectangle of each
mention, with an arrowhead and the property label. Arrows can cross pages.

### Page numbers
Printed page = first-page number + PDF page index. A statement's page string is
built from the pages of its two mentions: one page gives `57`, consecutive pages
give `57-58`, otherwise a list `57, 59`. It can be edited per statement.

### QuickStatements output
One line per statement, in QuickStatements V1 format:

`subject | property | value | S248 | source | S304 | "pages" | S813 | +date T00:00:00Z/11`

- Item values are written as QIDs; text and URLs as quoted strings (quotes,
  pipes, tabs and line breaks inside values are neutralised).
- Dates are written `+YYYY-MM-DDT00:00:00Z/P` with precision 9 (year), 10
  (month) or 11 (day).
- The display and clipboard use `|` separators. The URL used by **Open in
  QuickStatements** uses tabs, URL-encoded after `https://quickstatements.toolforge.org/#/v1=`.
  If the URL would exceed 7,000 characters, QuickStatements is opened without
  commands and you paste them.

### Saving
- **Automatic**: `localStorage`, key `wdann:<file name>:<file size>`, holding the
  mentions, statements, source QID and first page.
- **Project file**: JSON with `app`, `v` (format version, currently 1), `file`,
  `size`, `src`, `first`, `M` (mentions) and `S` (statements).

### Browser compatibility
Needs a modern browser with ES2017, `fetch`, `IntersectionObserver` and
`localStorage`. The file also works when opened from your disk (`file://`).

### Security
- **Malicious PDFs.** pdf.js versions before 4.2.67 are affected by
  CVE-2024-4367 (code execution when opening a crafted PDF). The app uses
  pdf.js 3.11.174 with the documented workaround, `isEvalSupported: false`.
  Still, open PDFs from sources you trust.
- **Untrusted project files and stored data.** Everything read from an imported
  project file or from browser storage goes through a strict validator
  (`clean()`): only well-formed mentions and statements are kept, Wikidata ids
  must match `Q…` / `P…` plus digits, dates must match the expected format, and
  invalid entries are dropped (their number is reported). Ids and values are
  checked again when the commands are generated, and escaped before being
  shown. Only import project files you trust anyway, and read each batch
  before running it in QuickStatements.
- **Third-party script.** The pdf.js script tag has a Subresource Integrity
  hash, so a modified file from the CDN is refused. The worker script, which
  pdf.js loads itself, cannot carry a hash.
- **Content security policy.** A policy in the page only allows scripts from
  cdnjs, and network requests only to Wikidata and cdnjs. If PDFs fail to
  display in your browser, remove the `Content-Security-Policy` line to test
  whether it is the cause, and report it.
- **New tab.** QuickStatements opens with `noopener` and `noreferrer`.
- **Privacy.** The selected text is sent to Wikidata's search. Annotations,
  including the selected text, are stored in your browser and in exported
  project files, so be careful when sharing those files.

### Known technical caveats
- The way QuickStatements accepts commands through the URL could not be
  confirmed in its current documentation, hence the clipboard fallback.
- Highlights are placed from the browser's text rectangles, so they may be
  slightly off on PDFs with unusual fonts or rotated text.
