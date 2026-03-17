---
name: pdf-to-data
description: "Extract structured data from financial PDFs — CIMs, annual reports, SEC filings, broker materials, pitch books, term sheets, credit agreements, and financial statements. Converts unstructured PDF content into clean tables, JSON, Excel, or markdown. Use when users say 'extract data from this PDF,' 'parse this CIM,' 'pull the financials from this filing,' 'read this term sheet,' 'structure this document,' 'convert PDF to Excel,' 'extract tables from PDF,' 'pull numbers from this report,' 'digitize this,' 'parse this 10-K,' 'read this pitch book,' or 'what are the key terms in this document.' Also triggers on 'PDF to data,' 'PDF extraction,' 'PDF to spreadsheet,' 'ingest this document,' 'OCR this,' or 'scrape this PDF.'"
---

# PDF to Structured Data

## Overview

This skill extracts structured, usable data from financial PDFs. It reads the document, identifies content types (tables, narrative, terms, schedules), and outputs clean structured data in the user's preferred format (Excel, JSON, markdown tables, or CSV).

**Core principle:** Extraction must be faithful to the source. Never infer, estimate, or fill in missing data. If a number is unclear or unreadable, flag it — don't guess.

---

## Supported Document Types

| Document | What Gets Extracted | Typical Output |
|----------|-------------------|----------------|
| **CIM / Confidential Info Memo** | Company overview, financials, customer data, management team, growth drivers | Deal summary + financial tables in Excel |
| **10-K / 10-Q (SEC Filing)** | Income statement, balance sheet, cash flow, segment data, risk factors | 3-statement Excel model inputs |
| **Annual Report** | Financial statements, KPIs, segment breakdown, guidance | Excel tables + KPI summary |
| **Pitch Book / Presentation** | Comps tables, deal terms, org charts, transaction summaries | Excel tables + narrative summary |
| **Term Sheet** | Key terms, pricing, covenants, fees, conditions | Structured term grid (markdown or Excel) |
| **Credit Agreement** | Definitions, covenants, pricing grid, amortization, baskets | Covenant summary + amortization schedule |
| **Broker / Teaser** | Deal highlights, financials, asking terms | One-page deal screening input |
| **Fund Report / LP Letter** | NAV, returns, portfolio summary, cash flows | Performance table + portfolio summary |
| **Appraisal / Valuation Report** | Comparable transactions, DCF assumptions, concluded values | Valuation summary in Excel |
| **Invoice / Statement** | Line items, totals, dates, counterparties | Structured table |

---

## Workflow

### Step 1: Receive and Read the PDF

Read the PDF using the built-in file reading capability.

```
- Accept the PDF file from the user
- Read the full document (or specified page ranges for large files)
- Identify the document type from content and layout
- Note the total page count and overall structure
```

**For large PDFs (50+ pages):** Ask the user which sections matter most. Don't extract everything if only the financials are needed.

### Step 2: Identify Content Zones

Classify every section of the document into one of these content types:

| Content Type | Description | Extraction Method |
|-------------|-------------|-------------------|
| **Financial Table** | Income statement, balance sheet, cash flow, comps | → Excel with formulas preserved |
| **Data Table** | Customer lists, vendor lists, asset schedules | → Excel or CSV |
| **Key Terms** | Term sheet terms, covenant definitions, pricing | → Structured key-value pairs |
| **Narrative** | Business description, risk factors, market overview | → Summarized markdown |
| **Schedule** | Amortization, debt maturity, lease schedule | → Excel with dates and amounts |
| **Chart / Graph** | Visual data presentation | → Describe the data shown; extract underlying data if axis labels are readable |
| **Boilerplate** | Legal disclaimers, TOC, cover pages | → Skip unless user requests |

### Step 3: Extract Financial Tables

This is the highest-value extraction. Follow these rules precisely:

**Table Detection:**
1. Identify column headers (years, quarters, periods, categories)
2. Identify row labels (line items, accounts, descriptions)
3. Map the grid — every cell to its row/column intersection
4. Detect units (thousands, millions, billions) from headers or footnotes
5. Detect currency (USD, EUR, GBP, etc.)

**Extraction Rules:**
- Preserve the exact numbers as printed — do NOT round or reformat
- Preserve negative number notation: `(1,234)` means negative
- Capture footnote markers (*, (1), (a)) and extract the corresponding footnotes
- If a cell contains "N/A", "NM", "—", or is blank, preserve that exactly
- Detect subtotals and totals — mark them so formulas can be added in Excel
- If columns are years, extract each year as a separate column
- If the table spans multiple pages, stitch it together into one continuous table

**Common Financial Table Patterns:**

```
Income Statement:
  Revenue → Gross Profit → EBITDA → EBIT → Net Income
  (Watch for: adjusted vs. GAAP, pro forma columns, LTM/NTM)

Balance Sheet:
  Current Assets → Total Assets → Current Liabilities → Total Liabilities → Equity
  (Watch for: as-of dates, pro forma adjustments)

Cash Flow Statement:
  Operating → Investing → Financing → Net Change
  (Watch for: capex breakout, working capital detail)

Comps Table:
  Company | EV | Revenue | EBITDA | EV/Revenue | EV/EBITDA | Growth
  (Watch for: calendarized vs. fiscal, source dates, footnoted adjustments)

Debt Schedule:
  Tranche | Facility | Rate | Maturity | Outstanding | Available
  (Watch for: L+spread notation, SOFR floors, accordion features)
```

### Step 4: Extract Key Terms (for Term Sheets / Credit Agreements)

Output as a structured grid:

```
| Term | Value | Notes |
|------|-------|-------|
| Borrower | | |
| Facility Type | | |
| Commitment Amount | | |
| Tenor / Maturity | | |
| Interest Rate | | Base rate + spread, floors |
| Amortization | | Annual %, schedule |
| Financial Covenants | | Levels, step-downs |
| Negative Covenants | | Key baskets, carve-outs |
| Conditions Precedent | | Material items |
| Fees | | Commitment, arrangement, agency |
```

For credit agreements, also extract:
- Definition of EBITDA (every add-back and exclusion)
- Covenant levels by test period
- Restricted payment and debt baskets with specific dollar/ratio thresholds
- Material adverse change definition

### Step 5: Extract Narrative Sections

For business descriptions, risk factors, and market sections:

- Summarize into structured bullet points (not full paragraphs)
- Preserve specific data points mentioned in narrative (market sizes, growth rates, customer counts)
- Flag forward-looking statements vs. historical facts
- Extract any embedded data that should be in a table but was written as prose

**Example — narrative to structured:**

PDF says: *"The Company generated revenue of $45.2 million in FY2025, up 23% from $36.7 million in FY2024, driven primarily by the addition of 47 new enterprise customers and a net revenue retention rate of 118%."*

Extract:
```
| Metric | FY2024 | FY2025 | Change |
|--------|--------|--------|--------|
| Revenue ($M) | 36.7 | 45.2 | +23% |
| New Enterprise Customers | — | 47 | — |
| Net Revenue Retention | — | 118% | — |
```

### Step 6: Build Output

**Default output: Excel workbook** with:
- One sheet per major table extracted
- A "Summary" sheet with document metadata and key data points
- A "Raw Extract" sheet with all narrative content in structured form
- Proper number formatting, column widths, and headers
- Cell comments for footnotes

**Alternative outputs (if user requests):**
- **JSON:** Nested structure with document → sections → tables/terms/narrative
- **Markdown:** Tables in pipe format, narrative as bullet points
- **CSV:** One file per table, with clear headers

### Step 7: Validation Checklist

Before delivering, verify:

- [ ] All financial tables foot (totals match sum of line items)
- [ ] Cross-references check (e.g., net income on IS matches net income on CF statement)
- [ ] Units are consistent and labeled (thousands vs. millions)
- [ ] Negative numbers are correctly captured
- [ ] No data was fabricated — every number traces to a specific page/location
- [ ] Footnotes are captured and linked to the relevant cells
- [ ] Document metadata is recorded (date, source, page count)

---

## Handling Common Challenges

### Scanned / Image PDFs
- If the PDF is image-based (no selectable text), state this clearly
- Describe what you can see from the image content
- For tables in image PDFs, extract what is visually readable and flag any uncertain values with `[?]`
- Recommend the user re-obtain a text-based PDF if accuracy is critical

### Multi-Column Layouts
- Pitch books often use side-by-side layouts
- Read left column fully before right column
- Don't merge unrelated columns into one table

### Watermarked / Confidential Documents
- Extract data normally — watermarks don't affect content
- Note the confidentiality marking in the output metadata
- Do NOT strip or remove confidentiality notices

### Inconsistent Formatting
- Some PDFs mix number formats (1,234 vs 1.234 for thousands)
- Detect the convention used in the document and normalize
- Flag any ambiguous numbers: is "1.234" one-point-two-three-four or one-thousand-two-hundred-thirty-four?

### Tables That Span Multiple Pages
- Stitch them together — match column alignment by header position
- If headers repeat on each page, use the first occurrence and merge rows
- If a row is split across a page break, reconstruct it

---

## Important Notes

- **Faithfulness over completeness.** It is better to flag `[UNREADABLE]` than to guess a number.
- **Preserve original precision.** If the PDF says $45.2M, output 45.2 — not 45.20 or 45.
- **State your confidence.** If extraction was clean, say so. If the PDF was messy, list what might be wrong.
- **Large documents need scoping.** A 200-page credit agreement doesn't need full extraction — ask what the user needs.
- **Always cite page numbers.** Every extracted data point should reference the source page so the user can verify.
- **Don't interpret — extract.** If the PDF says revenue grew 23%, extract "23%". Don't calculate your own growth rate from the absolute numbers (unless asked to verify).
- **Financial data has legal implications.** Accuracy matters. When in doubt, flag it and let the user verify.
