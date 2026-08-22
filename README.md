 # Global Climate Analysis: Temperature Anomalies & CO₂ Emissions (1850–2025) with Power BI



A Power BI project analyzing 175 years of global temperature anomaly data (NOAA) alongside 274 years of CO₂ emissions data (Our World in Data), built on a star-schema model with time-intelligence measures and a hand-built Pearson-r correlation between temperature and emissions.

[Global Climate Analysis overview dashboard — KPI cards for warming rate, correlation, highest anomaly and CO2 per capita, with temperature and emissions trend lines from 1850–2025]

 
<img width="584" height="332" alt="Image" src="https://github.com/user-attachments/assets/3515e0d2-cadc-409f-baa5-912edf0a5195" />


## 📊 Dataset

| Table | Rows | Coverage | Source |
|---|---|---|---|
| **Annual Temp Anomalies (NOAA)** | 176 | 1850 – 2025 | NOAA — annual global temperature anomaly vs. baseline |
| **CO2 emissions per capita (OWID)** | 26,509 | 1750 – 2024 | Our World in Data — 231 distinct entities |
| **Date** (bridge/dimension table) | 146 | 1880 – 2025 | Built in-model for time intelligence |

**Note on the CO₂ table:** the 231 "Country Name" entities include individual countries mixed with continent- and region-level aggregates (e.g. *Africa*, *Asia*, *Asia (excl. China and India)*) — a standard feature of the OWID dataset. This matters for any measure that averages "across all entities," see the methodology note below.

---

## 🧱 Data Model

A simple star schema: the **Date** table sits as the bridging dimension between the two fact tables — `Year` → `Annual Temp Anomalies` (annual grain) and `Date` → `CO2 emissions per capita` (also annual grain, joined on the first day of the year). Both relationships are one-directional, many-to-one.

The `Date` table also carries a `YearMonth` key that isn't in use yet — its description flags it as the future relationship key for a planned **Sea Level Rise** table, so the model is deliberately built ahead of that next addition.



## 💡 Key Insights

- **Average anomaly across the full record (1850–2025): −0.24 °C.** Most of the 19th and early 20th century sat below the modern baseline, which pulls the long-run average negative.
- **Highest anomaly ever recorded: +0.98 °C in 2024** — the warmest year in the dataset.
- **Lowest anomaly recorded: −0.74 °C.**
- **Average year-over-year change: +0.005 °C/year** — small but consistently positive, i.e. a persistent gradual warming signal rather than a flat trend.
- **Long-term warming trend (1850–2025): ≈ 0.077 °C per decade**, compared with a reference figure of **0.229 °C per decade for the last 30 years (1995–2025)** — three times faster, illustrating that warming has accelerated well beyond the 175-year average pace.
- **Correlation between temperature anomaly and global CO₂ per capita: r ≈ 0.556** — a moderate-to-strong positive relationship: years with higher average CO₂ per capita tend to be warmer years.
- **Global average CO₂ per capita (2015–2024): 4.84 tonnes**, down from 5.14 tonnes in 1990 to 4.70 tonnes in 2024 (−8.6%) — see the methodology note below on how this average is calculated.

---

## 📸 Report Pages

**Decade breakdown** — average temperature anomaly and average CO₂ per capita, grouped by decade from the 1930s to the 2020s:

[Average temperature anomaly and average CO2 emissions per capita by decade, both trending upward from the 1930s to the 2020s] 
 
<img width="586" height="333" alt="Image" src="https://github.com/user-attachments/assets/03077b87-d0c3-46f8-99d2-c0f6fe70e9ae" />

**Correlation & year-over-year change** — the scatter plot below relates average annual CO₂ per capita to temperature anomaly with a trend line, alongside average YoY temperature change by decade:

![Scatter plot showing a positive relationship between CO2 emissions per capita and temperature anomalies, plus a bar chart of average year-over-year temperature change by decade showing acceleration since the 1970s–1980s]

 
<img width="580" height="325" alt="Image" src="https://github.com/user-attachments/assets/692d9bc7-c9a2-4286-8679-f816a1c4cd98" />


**Warming trend & country rankings** — long-term temperature and CO₂ trend lines against a forecast band, a table of the years with the highest temperature anomalies, and the top 10 countries by average CO₂ emissions per capita:

![Long-term temperature anomaly and CO2 emissions trend lines with forecast bands, a table of years with the highest temperature anomalies, and a bar chart of the top 10 countries by average CO2 emissions per capita]

 
<img width="584" height="327" alt="Image" src="https://github.com/user-attachments/assets/96daafe1-e8a3-42bd-a233-9feab01b4a88" />


## 🧮 DAX Measures

**Temperature (12 measures)** — averages/min/max, YoY change and a color-coded "Warming/Cooling" indicator, long-term trend (endpoint method), and a static 30-year reference rate for the card visual.

**CO₂ Emissions (11 measures)** — averages, recent-decade and baseline-year (1990) figures, absolute and percentage change since 1990 with a growth-category classifier, and (currently broken) top-emitter text measures — see Known Limitations.

**Cross-table (1 measure)** — `Correlation Coefficient: Temp Anomaly vs CO2`, a manually built Pearson-r formula:

```dax
Correlation Coefficient: Temp Anomaly vs CO2 =
VAR TempData =
    SUMMARIZE(
        VALUES('Annual Temp Anomalies (NOAA)'[Year]),
        'Annual Temp Anomalies (NOAA)'[Year],
        "TempAnomaly", AVERAGE('Annual Temp Anomalies (NOAA)'[Temp Anomaly]),
        "CO2PerCapita", CALCULATE(
            AVERAGE('CO2 emissions per capita (OWID)'[Annual CO₂ emissions (per capita)]),
            ALL('CO2 emissions per capita (OWID)'[Country Name])
        )
    )
VAR N = COUNTROWS(TempData)
VAR AvgTemp = AVERAGEX(TempData, [TempAnomaly])
VAR AvgCO2 = AVERAGEX(TempData, [CO2PerCapita])
VAR NumeratorSum = SUMX(TempData, ([TempAnomaly] - AvgTemp) * ([CO2PerCapita] - AvgCO2))
VAR StdDevTempSum = SUMX(TempData, ([TempAnomaly] - AvgTemp) ^ 2)
VAR StdDevCO2Sum = SUMX(TempData, ([CO2PerCapita] - AvgCO2) ^ 2)
VAR Denominator = SQRT(StdDevTempSum * StdDevCO2Sum)
RETURN IF(Denominator = 0, BLANK(), NumeratorSum / Denominator)
```

This builds a year-by-year table (matching each year's temperature anomaly to that year's average CO₂ per capita across all entities) and computes Pearson's r from first principles — the same approach used in the [Smart Home IoT Simpson's Paradox project](../smart-home-iot-analysis).

---

## 📝 Data & Methodology Notes

- **`Long-term Warming Trend (°C per decade)`** compares only the *first* and *last* year's individual values (1850 and 2025), not a full linear regression across all 176 years. It's a quick endpoint estimate and can be sensitive to noise in either single year — a regression-based (e.g. `LINEST`-style) trend would be more robust for a formal claim.
- **`Warming Rate Last 30 Years`** is a static, hardcoded value (0.229 °C/decade for 1995–2025) rather than one computed from the local data — noted in the measure's own description as intentional, for use in a card visual.
- **`Global CO2 Per Capita`** and the correlation measure average *equally* across all 231 entities in the OWID table. Since that list mixes individual countries with continent/region aggregates, this is **not** a true population-weighted global figure (total world emissions ÷ total world population) — it's an unweighted average across whatever entities exist in the "Country Name" column for that year.



## ⚠️ Known Limitations

- **`Top 5 CO2 Emitters Text`** and **`Top 10 CO2 Emitters Text`** currently fail — both filter on `'CO2 emissions per capita (OWID)'[Entity Type]`, a column that doesn't exist in the table. Until that column is added (or the filter removed), these two measures return an error rather than a ranked list.

---

## 🚀 Roadmap

- Add the planned **Sea Level Rise** table and connect it to the existing `Date[YearMonth]` key (already reserved for this purpose).
- Add the missing `Entity Type` column to the CO₂ table (or an equivalent country-vs-aggregate flag) to fix the Top 5/10 Emitters measures and correctly exclude continents/regions from country-level rankings.
- Consider replacing the endpoint-based warming trend with a proper linear-regression measure for a more defensible long-term trend figure.

---

## 🛠️ Tools Used

- **Power BI Desktop** — star-schema data modeling
- **DAX** — time-intelligence measures, manual Pearson-r correlation, conditional formatting logic (warming/cooling icons, color codes)
- **Power Query** — data ingestion from NOAA and OWID source files

---

## 👤 Author

**Leena A. Elsheikh** — Data Analyst | MSc Statistics, MBA | Microsoft PL-300 Certified
[LinkedIn](https://www.linkedin.com) · [GitHub](https://github.com/Leena01103)
