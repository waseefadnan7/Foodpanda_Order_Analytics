# Methodology: Cleaning & Engineering the Foodpanda Order Dataset

This document walks through every transformation applied to the raw foodpanda order export, in the order they were performed.

## 1. The Raw Export

The original file contained 198 rows with the following raw issues:

| Field | Problem |
|---|---|
| `Price` | Stored as text, e.g. `" Tk 1,855 "`. Currency symbol, thousands separator, and padding whitespace all needed cleanup |
| `Delivery Date` | A full sentence combining status and timestamp, e.g. `"Delivered on Tue, May 12, 8:18 PM"`. Critically, it never included a year |
| `Items Line 1/2/3` | Free-text item descriptions with quantity prefixes, e.g. `"1x Grilled Chicken,"` |
| `Restaurant Names` | Inconsistent casing and stray whitespace (`" ZAFRAN RESTORA & KABAB "` vs `"Dream Burger"`) |
| `Order ID` | Embedded a sequence number, but its significance wasn't initially obvious |

## 2. Recovering the Missing Year

This was the central data quality problem. The timestamp string gave weekday, month, day, and time of day, but no year, across what was clearly a multi-year history.

**Step 1: Parse the components.**
Each `Delivery Date` string was parsed with a regex to extract weekday, month, day, hour, minute, and AM/PM.

**Step 2: Generate year candidates.**
For each row, every year from 2021 to 2026 was tested against one question: does that year's calendar actually place the given month and day on the given weekday? In most years, a specific month/day combination falls on a different weekday, so this test is highly discriminating.

**Step 3: Check for uniqueness.**
Across all 198 rows, **every single row produced exactly one matching year.** No row had zero matches (which would mean a parsing error), and no row had multiple matches (which would mean genuine ambiguity requiring a tiebreaker).

**Step 4: Cross-validate with Order ID sequence.**
As a sanity check, the numeric sequence embedded in `Order ID` was extracted and compared against row order. Since the export was already sorted newest first, the sequence number should decrease monotonically as the reconstructed year and date get older. This held true across the dataset, confirming the year-recovery logic was internally consistent rather than coincidentally correct on a few rows.

**Result:** A fully reconstructed `Year` field spanning 2021 to 2026, derived with zero manual lookup, using only structural properties already present in the export.

## 3. Price Cleaning

```
Price (cleaned) = VALUE(SUBSTITUTE(SUBSTITUTE(TRIM(Price), "Tk", ""), ",", ""))
```
This removed the currency label, stripped thousands separators, trimmed whitespace, and cast the result to numeric.

## 4. Item Parsing

Each item line followed the pattern `"<qty>x <item name>,"`. This was split into:
- **Quantity**: the leading number before `x`
- **Item name**: everything after the `x`, with the trailing comma stripped
- **Item Count**: a per-order count of how many of the three item-line slots were populated (non-blank, non `"-"`)

## 5. Cuisine Classification

Rather than apply a generic external food category list, a keyword-to-cuisine mapping table was built **from the dataset's own actual item names**, scanning all ~200 unique item mentions across the three item-line columns and grouping them into cuisine buckets (Bengali/Desi, American/Fast Food, Italian/Pizza, Kabab/Grill, East Asian, Middle Eastern, Seafood, Dessert/Bakery, and so on).

Keywords were ordered from most specific to least specific (for example, `"khichuri"` checked before a generic `"chicken"` match) so that compound dishes classify correctly rather than falling into an overly broad bucket. Items that didn't match any keyword, a small number, mostly branded combo names, were left as `"Other"` and reviewed manually.

## 6. Restaurant Name Standardization

Applied `TRIM` and title-casing to collapse inconsistent capitalization and stray whitespace variants of the same restaurant name into a single consistent label, ensuring accurate grouping in restaurant-level aggregations.

## 7. Power BI Modeling

- Built a dedicated **Date table** via `CALENDAR()` in DAX, with `Year`, `MonthName` (sorted by `MonthNum`), and `Weekday` (sorted by `WeekdayNum`) columns, ensuring charts display in chronological order rather than alphabetical order
- Added an `Hour` column directly to the orders table, not the Date table, via Power Query, since hour of day is a property of a specific order's timestamp, not of a calendar date
- Built core measures: `Total Orders`, `Total Spend`, `Avg Order Value`, `Cancellation Rate %`, `Years Active`, and a year-over-year spend growth measure using `DATEADD`

---

*All transformations are reproducible from the raw export using the steps above. No manual row-by-row editing was performed.*
