# 🍔 5 Years, 198 Orders, and a Missing Year Column: A Personal Foodpanda Behavior Analysis

I exported my own foodpanda order history and turned it into a full behavioral analytics project. I cleaned a genuinely messy real-world export, reverse-engineered a missing date field, and built an interactive Power BI dashboard to answer one question: **what does five years of food delivery data actually say about my habits?**

![Dashboard Overview](assets/dashboard_overview.png)

---

## 📌 Key Insights

- **198 orders, ~Tk 91,300 spent**, averaging **Tk 461 per order**, across **July 2021 to May 2026**
- **Saturdays are my heaviest ordering day**, with a clear secondary peak on Tuesdays. Weekday office-lunch fatigue is a likely driver
- **Orders cluster hard between 8 to 11 PM**, with a smaller secondary spike right at midnight: a classic late-night-craving pattern, not a dinner-time one
- **American/Fast Food dominates at ~40% of all orders**, followed by Bengali/Desi (~18%) and Italian/Pizza (~17%). Convenience food clearly wins over heavier traditional meals on a weekday basis
- **Pandamart, Pizza Day, and Best Fried Chicken** are my most-ordered-from restaurants by count, but the *top spenders* list (by total Tk) tells a slightly different story, since frequency and spend don't fully overlap
- **Order volume and spend have both grown year over year**, with a sharp jump starting around 2025: a trend worth digging into further (life change? price inflation? both?)
- Only a **2.5% cancellation rate** across 5 years: fairly disciplined ordering behavior overall

---

## 🛠️ Tech Stack

`Excel` · `Power Query` · `DAX` · `Power BI` · `Data Cleaning` · `Time Intelligence`

---

## 📊 The Dashboard

**Page 1, Overview & Time:** KPI summary, spend trend over 5 years, orders by weekday, orders by hour of day

![Overview and Time](assets/dashboard_overview.png)

**Page 2, Loyalty & Preference:** Top restaurants by order count vs. by spend, year-over-year growth, cuisine mix

![Loyalty and Preference](assets/Loyalty_Preference.png)

---

## 🔍 The Data Story: Recovering a Missing Year

The raw export from foodpanda had a serious gap: the delivery timestamp field recorded **weekday, month, day, and time, but never the year.** A row might read `"Delivered on Tue, May 12, 8:18 PM"`, with nothing identifying *which* May 12th it was.

Rather than treat ~5 years of order history as undatable, I reconstructed the year using two structural clues hiding in the export itself:

1. **The Order ID contains a strictly increasing sequence number.** Newer orders always have a higher sequence number than older ones, and the export was already sorted newest-first.
2. **Weekday, month, and day together uniquely determine the year** within any reasonable real-world window, since a given weekday only lands on a specific calendar date once every several years.

By checking every plausible year (2021 to 2026) against each row's weekday/month/day combination, **every single row resolved to exactly one valid candidate year, with zero ambiguity.** Cross-checking that result against the Order ID sequence (which should move consistently with time) confirmed the reconstruction was internally consistent across the full dataset.

The result: a genuine 5-year time axis, recovered entirely from fields that were never meant to encode a year at all.

Full methodology: [`docs/methodology.md`](Docs/methodology.md)

---

## 🧹 Other Data Cleaning Steps

- Converted price from text (`" Tk 1,855 "`) to numeric, handling thousands separators
- Parsed item lines (`"1x Grilled Chicken,"`) into clean item names and quantities
- Built a keyword-based cuisine classifier (Bengali/Desi, American/Fast Food, Italian/Pizza, Kabab/Grill, East Asian, etc.) mapped from the dataset's own actual menu items, not a generic external list
- Standardized restaurant name casing and whitespace for accurate grouping
- Built a dedicated Power BI Date table with proper weekday and month sorting for clean time-intelligence charts

---

## 🔁 Reproducing This

1. Clone the repo
2. Open `data/foodpanda_orders_cleaned.xlsx` to see the cleaned dataset and cuisine mapping
3. Open `powerbi/foodpanda_dashboard.pbix` in Power BI Desktop
4. To apply the brand theme: **View → Themes → Browse for themes**, then select `assets/foodpanda_theme.json`

---

## 🔗 Links

- 📝 LinkedIn post: *[link once published]*
---
*This is a personal data project using my own order history. It is not affiliated with or endorsed by foodpanda.*
