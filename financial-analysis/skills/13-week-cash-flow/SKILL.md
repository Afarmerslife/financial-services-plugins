---
name: 13-week-cash-flow
description: "Build, populate, and analyze 13-week cash flow forecasts for liquidity management, restructuring, and distressed situations. Creates detailed weekly cash receipt and disbursement models with variance tracking, covenant compliance monitoring, and liquidity runway analysis. Use when users say 'build a 13-week cash flow,' 'weekly cash forecast,' 'liquidity analysis,' 'cash flow projection,' 'restructuring cash flow,' 'DIP budget,' 'cash runway,' 'weekly receipts and disbursements,' 'variance report,' 'rolling forecast,' or 'how many weeks of cash do we have.' Also triggers on 'TWCF,' '13 week,' 'cash burn,' 'liquidity runway,' 'cash management,' or 'covenant compliance.'"
---

# 13-Week Cash Flow Forecast

## Overview

This skill builds **13-week cash flow (TWCF) models** — the standard liquidity management and restructuring tool used by distressed companies, turnaround advisors, DIP lenders, and creditor committees. It produces a weekly receipts-and-disbursements forecast with variance tracking and liquidity analysis.

**Core principle:** Cash is fact. Every line item must tie to a verifiable source — AR aging, AP schedules, payroll registers, debt service calendars, or management estimates with stated assumptions.

---

## When to Use This Skill

- Company facing liquidity pressure or cash crunch
- Restructuring / Chapter 11 DIP budget preparation
- Lender or creditor committee oversight
- M&A buyer diligence on working capital and cash conversion
- PE portfolio company cash management
- Any situation where weekly cash visibility is critical

---

## Workflow

### Step 1: Gather Inputs

Request the following from the user (adjust based on what's available):

**Required:**
- Most recent bank balance(s) — starting cash position
- Accounts receivable aging schedule (current, 30, 60, 90+ days)
- Accounts payable aging schedule
- Payroll schedule and headcount
- Debt service calendar (interest, principal, revolver draws)
- Any committed capex or one-time payments

**Helpful but not required:**
- Historical weekly cash receipts (last 13-26 weeks) for seasonality
- Revenue forecast or bookings pipeline
- Existing budget or monthly P&L forecast
- Customer payment terms and concentration
- Vendor critical payment list
- Lease and rent schedules

If the user provides a monthly P&L forecast but no weekly detail, use the conversion logic in Step 2 to build weekly estimates.

### Step 2: Build the Model Structure

Create an Excel workbook with these sheets:

#### Sheet 1: "13-Week Cash Flow" (Primary Model)

```
Layout:
- Row 1: Title — "13-Week Cash Flow Forecast"
- Row 2: Company name, prepared date, version
- Row 3: Blank
- Row 4: Headers — "Week Ending:" then 13 weekly dates (Friday end-dates)
- Row 5: Week numbers (Wk 1 ... Wk 13) + "13-Wk Total"

OPERATING RECEIPTS
- Collections on existing AR (by aging bucket)
- Collections on new sales
- Other operating receipts (rebates, refunds, royalties)
- Intercompany receipts
→ Total Operating Receipts

OPERATING DISBURSEMENTS
- Payroll & benefits (gross)
- Payroll taxes
- Rent & occupancy
- Utilities
- Insurance premiums
- Raw materials / COGS purchases
- Freight & logistics
- Sales & marketing spend
- Professional fees (legal, accounting, consulting)
- IT & software
- Maintenance & repairs
- Other operating disbursements
→ Total Operating Disbursements

→ NET OPERATING CASH FLOW (Receipts − Disbursements)

NON-OPERATING / RESTRUCTURING
- Debt service — interest payments
- Debt service — principal payments
- Revolver draws / (repayments)
- Capital expenditures
- Tax payments
- Restructuring / professional fees
- One-time / extraordinary items
→ Total Non-Operating Cash Flow

→ NET CASH FLOW (Operating + Non-Operating)

LIQUIDITY SUMMARY
- Beginning cash balance
- Net cash flow
- Ending cash balance
- (+) Revolver availability
- (=) Total liquidity
- Minimum cash requirement
- Liquidity cushion / (shortfall)
- Weeks of cash remaining (at current burn)
```

#### Sheet 2: "Variance Report"

```
For each completed week, show:
- Forecast vs. Actual for every line item
- Variance ($)
- Variance (%)
- Cumulative variance
- Management explanation column (blank — for user to fill)
```

#### Sheet 3: "AR Waterfall"

```
- Opening AR balance by aging bucket (current, 30, 60, 90+)
- New billings each week
- Collections each week (by bucket)
- Closing AR balance
- DSO calculation
- Collection rate by bucket
```

#### Sheet 4: "AP Schedule"

```
- Opening AP balance by vendor or category
- New invoices received
- Payments made
- Closing AP balance
- DPO calculation
- Critical vendor flagging
```

#### Sheet 5: "Assumptions"

```
- Collection assumptions by aging bucket (% collected per week)
- Payment timing assumptions
- Payroll calendar
- Debt service schedule
- Key risks and sensitivities
- Version history / change log
```

### Step 3: Populate the Model

**Collection Assumptions (default — adjust based on actuals):**

| AR Bucket | Wk 1 | Wk 2 | Wk 3 | Wk 4 | Wk 5+ |
|-----------|-------|-------|-------|-------|--------|
| Current (0-30) | 15% | 25% | 25% | 20% | 15% |
| 31-60 days | 30% | 30% | 20% | 15% | 5% |
| 61-90 days | 40% | 25% | 15% | 10% | 10% |
| 90+ days | 20% | 15% | 10% | 5% | haircut remainder |

**If user provides historical weekly collections:** Override defaults with actual collection curves. Calculate historical collection rates by aging bucket and apply those forward.

**Monthly-to-weekly conversion rules:**
- Payroll: Map to actual pay dates (biweekly = every other week, semi-monthly = 2x/month)
- Rent: 100% in the first week of the month
- Revenue-linked items: Spread evenly across 4.33 weeks unless seasonality data exists
- Quarterly items (taxes, insurance): Map to the specific week they're due
- Debt service: Map to contractual payment dates

**Formula discipline:**
- Every total row must be a SUM formula
- Net cash flow = Total Receipts − Total Disbursements + Non-Operating
- Ending cash = Beginning cash + Net cash flow
- Beginning cash (Wk N) = Ending cash (Wk N-1)
- 13-Week Total column = SUM across all 13 weeks for each line
- Weeks of cash = Ending cash / (trailing 4-week avg weekly net cash burn)

### Step 4: Format the Workbook

**Formatting standards:**

| Element | Format |
|---------|--------|
| All dollar amounts | `#,##0` (no decimals — this is a cash flow, not a P&L) |
| Negative numbers | `(#,##0)` in red or parentheses |
| Section headers | Bold, dark background, white text |
| Subtotal rows | Bold, single top border |
| Grand total rows | Bold, double top border |
| Input cells (assumptions) | Blue font |
| Formula cells | Black font |
| Variance (unfavorable) | Red fill or red font |
| Variance (favorable) | Green font |
| Week ending dates | `MM/DD/YY` format |
| Column width | Weeks: 14 characters. Labels: 35 characters |
| Freeze panes | Freeze row 5 and column A |

### Step 5: Sensitivity Analysis

After building the base case, provide:

1. **Downside case:** Collections slip 2 weeks, disbursements stay on schedule
2. **Severe downside:** Collections slip 2 weeks + largest customer pays 50% slower + one-time expense hits
3. **Cash breakeven:** At what collection rate does the company run out of cash? In which week?

Present as a summary table:

```
| Scenario | Wk 4 Liquidity | Wk 8 Liquidity | Wk 13 Liquidity | Cash-Out Week |
|----------|---------------|----------------|-----------------|---------------|
| Base     |               |                |                 | N/A           |
| Downside |               |                |                 |               |
| Severe   |               |                |                 |               |
```

### Step 6: Deliver and Brief

Provide a **1-page executive summary** alongside the model:

- Current liquidity position (cash + revolver availability)
- Projected liquidity at Week 4, 8, and 13
- Key risks to the forecast (top 3)
- Critical assumptions that drive the outcome
- Recommended actions (if liquidity is tight)
- Covenant compliance status (if applicable)

---

## Rolling Forecast Updates

When the user provides actuals for a completed week:

1. Move actual data into the Variance Report sheet
2. Calculate all variances
3. Roll the forecast forward — drop the completed week, add a new Week 13
4. Adjust forward assumptions if actuals reveal a trend (e.g., collections slowing)
5. Update the executive summary with revised outlook

---

## Covenant & Compliance Monitoring

If the company has financial covenants, add a **Covenant Tracker** section:

```
| Covenant | Threshold | Current | Status | Breach Week (projected) |
|----------|-----------|---------|--------|------------------------|
| Minimum cash | $X.XM | | | |
| Max leverage | X.Xx | | | |
| Min fixed charge coverage | X.Xx | | | |
| Max capex | $X.XM | | | |
| Borrowing base availability | $X.XM | | | |
```

---

## Important Notes

- **Cash basis only.** This is not an accrual model. Revenue recognition, depreciation, and non-cash items do not appear.
- **Receipts ≠ Revenue.** Collections lag sales. Always model the timing gap.
- **Disbursements ≠ Expenses.** Payments lag accruals. Payroll, rent, and vendor payments have specific timing.
- **Update weekly.** A TWCF loses value if not maintained. Build for easy updating.
- **Actuals are non-negotiable.** When actuals are available, use them. Don't smooth or adjust completed weeks.
- **Flag unknowns.** If an assumption is a guess, say so. Mark uncertainty explicitly in the Assumptions sheet.
- **Err conservative.** In distressed situations, overestimating collections or underestimating disbursements can be fatal. Default to the pessimistic side.
- **Version control.** Save each weekly update as a new version. Never overwrite the prior week's forecast — you need the audit trail.
