# Specification prompt: PDF to Wikidata statements app

You can give this prompt to an AI assistant or a developer to rebuild or extend
the app. It lists the specifications established with the project owner.

---

## Goal
Build a very simple, user-friendly app to annotate a PDF document and generate
Wikidata statements (triples) from the annotations, ready to paste into
QuickStatements. It replaces INCEpTION, which the owner found too complicated
and too slow at matching text to Wikidata items.

## Platform and constraints
- A **single HTML file** that runs in the browser, with no installation.
- Internet access is acceptable (Wikidata search, PDF library from a CDN).
- **English-only** interface for now.
- The PDF stays on the user's computer; no server of its own.
- Primary source type: **PDF files** with selectable text. Scanned PDFs are out
  of scope.

## Setup per document
- The user enters the **source QID** (the Wikidata item of the document).
- The user enters the **first page number**, because the first page of a source
  is not always numbered 1. Printed page = first page number + PDF page index.

## Workflow (two steps)
1. **Match.** The user selects text in the PDF and matches it to an existing
   Wikidata item through a search box pre-filled with the selection. The matched
   text is highlighted.
2. **Link.** The user builds statements by linking matched items with an
   **arrow** (subject to value), then chooses the property by searching
   Wikidata properties.

Rules:
- Several statements can be created from the **same selected text** (a text can
  have many arrows).
- An **"Add value"** button turns selected text into a non-item value: text,
  URL or date. Values can be the target of an arrow, never the subject.
- Arrows are drawn **over the PDF text**. To stay readable, only the arrows
  connected to the currently selected text or statement are displayed.
- A statement is built from text on a single page. If the linked texts span two
  pages, the page reference includes both pages (for example `57-58`). The page
  reference can be edited per statement.

## Speed (main complaint about the previous tool)
- Search starts about 250 ms after typing stops and cancels outdated requests.
- Search results are cached.
- Items and properties are searched separately.
- Recently used properties are suggested when the property box is empty.
- Pressing Enter selects the first result.
- Use Wikidata's public search API directly, with no intermediate server.

## Output
- QuickStatements **V1 commands**, one line per statement:
  `subject|property|value|S248|source QID|S304|"page(s)"|S813|+date T00:00:00Z/11`
- Every statement carries a reference with:
  - **stated in** (`S248`) = the source QID;
  - **page(s)** (`S304`);
  - **retrieved** (`S813`) = the current date.
- **No quotation** is included, to avoid copyright issues.
- Supported value types: Wikidata items, text, URLs, dates (precision year,
  month or day following what the user types).
- Buttons: **Copy commands**, and **Open in QuickStatements**, which opens
  QuickStatements with the commands already filled in (and also copies them as a
  fallback).

## Interface
- Header: open PDF (or drag and drop), source QID, first page number, zoom
  controls (− and +), project export and import, status message.
- Main area: the PDF pages, with highlights and arrows.
- Right panel: selected-item bar with **Link to…** and **Delete**, list of
  statements (each with editable page(s) and a delete button), and the output
  box with the buttons above.
- Esc cancels linking, closes popups and clears the selection.

## Saving
- Automatic saving in the browser, per PDF.
- **Export project / Import project** buttons to save and reopen an annotation
  project as a JSON file.

## Out of scope for the current version
- Scanned PDFs (OCR)
- Creating new Wikidata items
- Editing Wikidata directly (the user runs the batch in QuickStatements)
- Multiple users, accounts, automatic suggestions
- Checking whether a statement already exists on Wikidata
- Quantities and coordinates as values

## Possible next steps
- Check for existing statements on Wikidata before export.
- Quantities and coordinates as value types.
- Interface language switch.
- OCR for scanned PDFs.
