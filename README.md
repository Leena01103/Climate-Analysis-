# Global Climate Trends — Temperature Anomalies & CO₂ Emissions

A Power BI analysis of long-term global warming, combining NOAA's global temperature anomaly record with Our World in Data's CO₂ emissions-per-capita dataset to explore how the two trends relate over time.

---

## 📊 Datasets

| Table | Rows | Coverage | Source |
|---|---|---|---|
| **Annual Temp Anomalies (NOAA)** | 176 | 1850 – 2025 | NOAAGlobalTemp v6, annual global land + ocean temperature anomaly, pulled **live** from NOAA's public data feed via Power Query `Web.Contents` — refreshes automatically each time the report is refreshed |
| **CO2 emissions per capita (OWID)** | 26,509 | Varies by country | Our World in Data, imported from a downloaded CSV |
| **Date** | 1,752 (monthly) | 1880 – 2025 | Custom calendar dimension built in M, used to bridge the two fact tables |

**Baseline note:** NOAA anomalies are expressed relative to the **1971–2000 climatological average**, not zero — so a negative anomaly for, say, 1900 doesn't mean "no warming," it means "cooler than the 1971–2000 baseline."

**Data model:** `Date[Year]` → `Annual Temp Anomalies[Year]` and `CO2 emissions[Date]` → `Date[StartOfMonth]`, both one-directional, letting the two fact tables be compared on a common Year axis without a direct relationship between them.

---

## 💡 Key Findings

| Metric | Value |
|---|---|
| Average temperature anomaly (1850–2025) | −0.24 °C |
| Maximum anomaly | **+0.98 °C, in 2024** |
| Minimum anomaly | −0.74 °C, in the 1850s |
| Long-term warming trend (first year → last year, ×10) | **+0.077 °C per decade** |
| Correlation: temp anomaly vs. global CO₂ per capita | **r ≈ 0.556** (moderate positive) |
| Global CO₂ per capita, 1990 → 2024 | 5.14 t → 4.70 t (**−8.6%**) |

The headline correlation (r ≈ 0.556) is calculated in DAX by pairing each year's average temperature anomaly with that year's average CO₂ per capita across all reporting entities, using a manual `SUMX`/`VAR` Pearson-r formula — the same approach used in the Simpson's Paradox / IoT project, applied here across two separate fact tables joined through `SUMMARIZE`.

---

## 🧮 DAX Highlights

- **Pearson correlation across two fact tables** — `Correlation Coefficient: Temp Anomaly vs CO2` builds a year-by-year paired table with `SUMMARIZE`, then runs the manual covariance/variance formula on it.
- **Dynamic long-term trend** — `Long-term Warming Trend (°C per decade)` compares the average anomaly in the first vs. last year in the current filter context and annualizes it, so it updates correctly if the data range changes.
- **YoY change & trend direction** — `YoY Change`, `Temperature Change Rate - Year-over-Year`, `Change Type`, `Temperature Change Category`, and `YoY Color Code (Red/Blue)` combine to drive conditional icons (🔴 Warming / 🔵 Cooling) and colors for KPI cards.
- **CO₂ growth framing** — `CO2 Change Since 1990 (Absolute)`, `CO2 Growth Since 1990 (%)`, and `CO2 Growth Category Since 1990` turn a raw before/after comparison into a labeled category (Strong Growth / Growth / Stable / Decline / Strong Decline).

---

## 🧰 Tools Used to Build the Report

- **Power Query** — live web pull from NOAA (`Web.Contents`) for the temperature data, CSV import for OWID CO₂ data, and a fully custom Date table generated with `List.Generate`
- **DAX** — manual Pearson-r correlation across two fact tables, dynamic trend calculations, conditional icon/color measures for KPI visuals

---

## ⚠️ Known Limitations

- **`Warming Rate Last 30 Years`** is a hardcoded constant (`0.229`, described as "1995–2025") rather than a calculated measure. It's useful as a fixed reference card, but it won't update if the underlying data changes — worth converting to a dynamic calculation for consistency with `Long-term Warming Trend`.
- **`Top 5 CO2 Emitters Text`** and **`Top 10 CO2 Emitters Text`** currently fail — both filter on a column called `Entity Type` that doesn't exist in the CO₂ table (the table only has Country Name, Code, Year, Annual CO₂ emissions, and Date). They need either an `Entity Type` column added in Power Query, or a filter against a known list of aggregate entity names.
- **Global CO₂ per capita figures are unweighted averages across all 231 "Country Name" entities**, and that list mixes individual countries with continent- and region-level aggregates (e.g. *Africa*, *Asia*, *Asia (excl. China and India)*) rather than excluding them. That means a small country and a whole-continent aggregate are weighted equally in measures like `Global Avg CO2 Per Capita` — it is **not** a population-weighted global figure. This is very likely why CO₂ per capita appears to *decline* from 1990 to 2024 (5.14 t → 4.70 t) in this model, even though global total emissions have risen over the same period — the trend is a methodology artifact of the unweighted country/region mix, not a real-world decline.

---

## 👤 Author

**Leena A. Elsheikh** — Data Analyst | MSc Statistics, MBA | Microsoft PL-300 Certified
[LinkedIn](https://www.linkedin.com) · [GitHub](https://github.com/Leena01103)
