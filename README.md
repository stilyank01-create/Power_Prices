# Macroeconomic Analysis — EU Battery Storage Arbitrage
## Analytical Pipeline — README
*Eurostat · ENTSO-E · Day-Ahead Prices · Solar Merit-Order · Two-Cycle Utility-Scale Arbitrage*

---

## Abstract

This project is a five-module macroeconomic analysis and analytical pipeline investigating the economic viability of grid-scale battery storage deployed across selected EU countries. The pipeline integrates three independent data sources: Eurostat installed capacity statistics, ENTSO-E real-time generation and day-ahead price APIs, and empirical solar generation profiles.

The goal of the project is to test whether solar production systematically depresses prices during midday hours, and whether batteries — deployed at national scale — can economically redistribute that energy to evening peak hours. The investment case is assessed at utility-scale lithium-ion storage costs (€300/kWh installed, 2024 mid-range), with two daily charge–discharge cycles to maximise revenue per unit of CapEx.

The analysis maps installed capacity across technologies and countries using Eurostat data (Module 1). From Module 2 onwards, the pipeline switches exclusively to ENTSO-E data, which provides more detailed and precise real-time energy statistics: solar penetration is quantified for eight EU countries (Module 2), day-ahead price dynamics during high-solar hours are characterised (Module 3), the solar merit-order effect is formally tested (Module 4), and a 10-year ROI for two-cycle arbitrage at utility-scale storage costs is quantified (Module 5).

Power generation volumes and prices are derived from ENTSO-E day-ahead 15-minute interval data, resulting in datasets of up to 8,760 × 4 = 35,040 observations per country per year. This granularity is essential for accurately characterising intraday price dynamics and identifying exploitable arbitrage windows.

The core hypothesis — that abundant midday solar generation systematically depresses day-ahead electricity prices, creating an exploitable price spread — is confirmed across all eight countries analysed (Modules 3 and 4). A Pearson correlation between solar generation and day-ahead price, computed over April–October 2024 when solar has a meaningful effect, returns a statistically significant negative coefficient in every country (p = 0.0000). However, the **investment case is more nuanced**: at honest cost and ROI accounting over a full operating year, only three of the eight national markets clear break-even over a 10-year horizon with two daily cycles.

---

## Pipeline Modules

### Module 1 — Installed Capacity per Technology per Country (Eurostat)
`01_Eurostat_Installed_Capacity_Per_Technology_Per_Country.ipynb`

Data source: Eurostat `nrg_inf_epc` dataset (installed electricity generation capacity by SIEC). The notebook queries each country once per year, decodes the JSON-stat response dynamically (reading `id` and `size` from each response so the parser stays correct under any query shape), and filters to leaf SIEC codes only — `TOTAL`, `RA100`, `RA300`, `RA420` are excluded to prevent double-counting parent + children. `CF` is kept as a single "Combustible fuels" line since `nrg_inf_epc` does not break coal/gas/oil out separately. For solar PV, `RA420AC` (alternating current) is preferred and falls back to `RA420DC` (direct-current nameplate) when AC is not reported.

- Technologies retained: combustible fuels (coal + gas + oil aggregated), nuclear, pure hydro, mixed hydro, pumped hydro, geothermal, wind onshore, wind offshore, solar PV, solar thermal, and tide/wave/ocean.
- Aggregates are excluded; AC and DC solar PV are not double-counted.

**▸ Output Data**


Fetching 2024...
  [OK] France: 11 technology records
  [OK] Spain: 11 technology records
  [OK] Austria: 7 technology records
  [OK] Bulgaria: 8 technology records
  [OK] Belgium: 9 technology records
  [OK] Greece: 6 technology records
  [OK] Czechia: 8 technology records
  [OK] Hungary: 8 technology records

============================================================
France
============================================================
TOTAL: 158,458 MW

BREAKDOWN:
  Nuclear fuels and other fuels n.e.c.              61,400 MW  ( 38.7%)
  Solar photovoltaic                                25,083 MW  ( 15.8%)
  Wind on shore                                     22,747 MW  ( 14.4%)
  Combustible fuels                                 20,411 MW  ( 12.9%)
  Pure hydro power                                  18,922 MW  ( 11.9%)
  Mixed hydro power                                  5,375 MW  (  3.4%)
  Pumped hydro power                                 1,728 MW  (  1.1%)
  Wind off shore                                     1,494 MW  (  0.9%)
  Other fuels n.e.c.                                 1,089 MW  (  0.7%)
  Tide, wave, ocean                                    210 MW  (  0.1%)

============================================================
Spain
============================================================
TOTAL: 135,952 MW

BREAKDOWN:
  Combustible fuels                                 38,325 MW  ( 28.2%)
  Solar photovoltaic                                35,823 MW  ( 26.3%)
  Wind on shore                                     32,179 MW  ( 23.7%)
  Pure hydro power                                  13,738 MW  ( 10.1%)
  Nuclear fuels and other fuels n.e.c.               7,117 MW  (  5.2%)
  Pumped hydro power                                 3,331 MW  (  2.5%)
  Mixed hydro power                                  3,071 MW  (  2.3%)
  Solar thermal                                      2,304 MW  (  1.7%)
  Other fuels n.e.c.                                    54 MW  (  0.0%)
  Wind off shore                                         5 MW  (  0.0%)
  Tide, wave, ocean                                      5 MW  (  0.0%)

============================================================
Austria
============================================================
TOTAL: 33,008 MW

BREAKDOWN:
  Pure hydro power                                   9,547 MW  ( 28.9%)
  Solar photovoltaic                                 8,143 MW  ( 24.7%)
  Mixed hydro power                                  5,639 MW  ( 17.1%)
  Combustible fuels                                  5,578 MW  ( 16.9%)
  Wind on shore                                      4,092 MW  ( 12.4%)
  Other fuels n.e.c.                                     8 MW  (  0.0%)
  Geothermal                                             0 MW  (  0.0%)

============================================================
Bulgaria
============================================================
TOTAL: 16,161 MW

BREAKDOWN:
  Combustible fuels                                  5,471 MW  ( 33.9%)
  Solar photovoltaic                                 4,560 MW  ( 28.2%)
  Pure hydro power                                   2,405 MW  ( 14.9%)
  Nuclear fuels and other fuels n.e.c.               2,006 MW  ( 12.4%)
  Pumped hydro power                                   864 MW  (  5.3%)
  Wind on shore                                        706 MW  (  4.4%)
  Mixed hydro power                                    149 MW  (  0.9%)

============================================================
Belgium
============================================================
TOTAL: 28,702 MW

BREAKDOWN:
  Solar photovoltaic                                 9,642 MW  ( 33.6%)
  Combustible fuels                                  7,797 MW  ( 27.2%)
  Nuclear fuels and other fuels n.e.c.               3,928 MW  ( 13.7%)
  Wind on shore                                      3,383 MW  ( 11.8%)
  Wind off shore                                     2,262 MW  (  7.9%)
  Pumped hydro power                                 1,307 MW  (  4.6%)
  Other fuels n.e.c.                                   259 MW  (  0.9%)
  Pure hydro power                                     124 MW  (  0.4%)

============================================================
Greece
============================================================
TOTAL: 28,716 MW

BREAKDOWN:
  Combustible fuels                                 11,041 MW  ( 38.5%)
  Solar photovoltaic                                 8,827 MW  ( 30.7%)
  Wind on shore                                      5,366 MW  ( 18.7%)
  Pure hydro power                                   2,782 MW  (  9.7%)
  Mixed hydro power                                    699 MW  (  2.4%)

============================================================
Czechia
============================================================
TOTAL: 23,022 MW

BREAKDOWN:
  Combustible fuels                                 12,054 MW  ( 52.4%)
  Nuclear fuels and other fuels n.e.c.               4,290 MW  ( 18.6%)
  Solar photovoltaic                                 3,984 MW  ( 17.3%)
  Pumped hydro power                                 1,172 MW  (  5.1%)
  Pure hydro power                                   1,118 MW  (  4.9%)
  Wind on shore                                        363 MW  (  1.6%)
  Other fuels n.e.c.                                    41 MW  (  0.2%)

============================================================
Hungary
============================================================
TOTAL: 14,676 MW

BREAKDOWN:
  Solar photovoltaic                                 7,166 MW  ( 48.8%)
  Combustible fuels                                  4,985 MW  ( 34.0%)
  Nuclear fuels and other fuels n.e.c.               2,027 MW  ( 13.8%)
  Wind on shore                                        325 MW  (  2.2%)
  Other fuels n.e.c.                                   109 MW  (  0.7%)
  Pure hydro power                                      61 MW  (  0.4%)
  Geothermal                                             3 MW  (  0.0%)

> *Selected key technologies shown.

---

### Module 2 — Installed Generation Capacity & Solar Share (ENTSO-E, 2024)
`02_ENTSO-E_Installed_Capacity_And_Solar_Share.ipynb`

Data source: ENTSO-E Transparency Platform API. From this module onwards, the pipeline works exclusively with ENTSO-E data, which provides more detailed and precise real-time energy statistics than the Eurostat aggregate figures used in Module 1. Countries queried: AT, BE, BG, CZ, FR, GR, HU, ES. No API failures recorded. Results sorted descending by solar share.

- Hungary leads with 35.9% solar share despite a relatively small total capacity (11,223 MW).
- France has the largest absolute solar fleet (17,795 MW) but the lowest share (12.0%) due to its dominant nuclear base (148,465 MW total).

**▸ Output Data**

| Country | Total Capacity (MW) | Solar Capacity (MW) | Solar Share (%) |
|---------|---------------------|---------------------|-----------------|
| HU | 11,222.9 | 4,029.1 | 35.90% |
| BE | 28,114.5 | 8,788.8 | 31.26% |
| BG | 15,848.0 | 4,568.0 | 28.82% |
| GR | 24,122.0 | 6,700.0 | 27.78% |
| AT | 28,994.3 | 7,294.0 | 25.16% |
| ES | 116,949.1 | 23,867.3 | 20.41% |
| CZ | 21,198.0 | 3,548.0 | 16.74% |
| FR | 148,464.9 | 17,794.9 | 11.99% |

---

### Module 3 — Day-Ahead Price Analysis — Solar vs. Non-Solar Hours
`03_ENTSO-E_Determine_Solar_Strong_Hours.ipynb`

Strong solar hours identified per country (08:00–18:00 for most; CZ 08–17, HU 07–17, ES 09–20), classified by the hour-of-day average solar output exceeding 20% of the peak hour-of-day average over April–October 2024 (the season in which solar materially affects prices). Prices are also restricted to the same April–October window for the spread comparison. The price spread between strong-solar and remaining hours quantifies the merit-order effect during the solar season.

**Note on scope.** This module measures the price gap *during the solar season*. Module 5's cycle-1 spread is computed over the full year and is therefore smaller (e.g. AT €27.96/MWh here vs. €43.04/MWh in Module 5's cycle-1 spread — different metrics, both correct). Module 3 answers "how strong is the merit-order effect when solar is active?"; Module 5 answers "what spread does a year-round arbitrage operator realize?"

- Largest spreads: HU €55.74/MWh, BG €51.61/MWh, GR €48.79/MWh — strongest arbitrage signal.
- Smallest spreads: FR €20.72/MWh, ES €31.00/MWh, AT €27.96/MWh — still commercially relevant at scale.

**▸ Output Data**

| Country | Strong Solar Hrs | Strong Mean (€/MWh) | Strong Median (€/MWh) | Rest Mean (€/MWh) | Rest Median (€/MWh) | Spread (€/MWh) | N Strong | N Rest |
|---------|-----------------|---------------------|-----------------------|-------------------|---------------------|----------------|----------|--------|
| AT | 08–18 | 57.29 | 64.86 | 85.25 | 84.96 | 27.96 | 2,354 | 2,783 |
| BE | 08–18 | 45.06 | 47.45 | 74.84 | 76.51 | 29.78 | 2,354 | 2,783 |
| BG | 08–18 | 71.92 | 75.26 | 123.53 | 99.66 | 51.61 | 2,354 | 2,783 |
| CZ | 08–17 | 54.48 | 62.32 | 92.88 | 89.98 | 38.40 | 2,140 | 2,997 |
| ES | 09–20 | 42.49 | 35.01 | 73.49 | 80.89 | 31.00 | 2,568 | 2,569 |
| FR | 08–18 | 32.94 | 25.16 | 53.66 | 51.61 | 20.72 | 2,354 | 2,783 |
| GR | 08–18 | 74.81 | 79.79 | 123.60 | 102.09 | 48.79 | 2,354 | 2,783 |
| HU | 07–17 | 67.20 | 72.45 | 122.94 | 98.17 | 55.74 | 2,354 | 2,783 |

---

### Module 4 — Hypothesis Test — Merit-Order Effect of Solar Generation
`04_ENTSO-E_Solar_Strong_Hours_Negative_Price_Correlation.ipynb`

Statistical test: one-sided mean comparison and Pearson correlation between solar generation (MW) and day-ahead price (EUR/MWh), computed on hourly observations over April–October 2024. A correlation threshold of |r| > 0.2 is applied — conventionally a "weak" effect in social-science contexts, but a meaningful signal in noisy energy-market time series, where price is driven by many independent factors (demand, weather, fuel prices, imports, scarcity). AT and BE show strong correlations of −0.524 and −0.522 respectively. All countries return p = 0.0000.

**What is the p-value?** The p-value measures the probability that the observed price difference between solar and non-solar hours occurred by random chance. A value of 0.0000 means this probability is vanishingly small (less than 0.01%), so the result is considered statistically conclusive.

- The merit-order effect is confirmed in all eight countries: higher solar output consistently suppresses day-ahead prices.
- Both complementary signals (mean price comparison and correlation threshold) are satisfied across the board.

**▸ Output Data**

| Country | Solar hours mean (EUR/MWh) | Other hours mean (EUR/MWh) | Price gap (EUR/MWh) | Correlation | p-value |
|---------|---------------------------|---------------------------|---------------------|-------------|---------|
| Austria | 57.29 | 85.25 | 27.96 | -0.524 | 0.0000 |
| Belgium | 45.06 | 74.84 | 29.78 | -0.522 | 0.0000 |
| Bulgaria | 71.92 | 123.54 | 51.62 | -0.317 | 0.0000 |
| Czechia | 54.48 | 92.89 | 38.41 | -0.495 | 0.0000 |
| Spain | 42.49 | 73.49 | 31.00 | -0.428 | 0.0000 |
| France | 32.94 | 53.65 | 20.71 | -0.352 | 0.0000 |
| Greece | 74.81 | 123.61 | 48.80 | -0.364 | 0.0000 |
| Hungary | 67.20 | 122.94 | 55.74 | -0.354 | 0.0000 |

---

### Module 5 — 10-Year ROI Model — Two-Cycle Utility-Scale Arbitrage
`05_ENTSO-E_Solar_Storage_Arbitrage_Return_Model.ipynb`

**Model.** National storage fleets are sized to add `target_fraction` (50% by default) of mean grid energy on top of generation during the top-5 peak price hours each day. Storage is priced at **€300/kWh installed** (utility-scale Li-ion, 2024 mid-range including inverter, balance-of-system, and interconnection). The fleet runs **two daily cycles**:

- **Cycle 1**: discharge during the top-5 evening peak hours; charge during strong-solar hours (April–October hour-of-day classification, threshold = 20% of the peak hour-of-day average solar output).
- **Cycle 2**: discharge during the next-best 3 price hours of each day (dynamic per day); charge during the cheapest 3 non-strong hours of each day (typically overnight).

Fleet sizing is bounded by cycle 1 (the larger daily volume). Cycle 2 reuses the same fleet at a different time of day. Daily delivered energy in each cycle is capped at the cycle's target. Utilisation degrades linearly from 100% in year 1 to 70% in year 10 (10-year time-average = 85%).

**Solar feasibility check.** For each country, the model verifies whether the strong-solar window's total available solar generation is enough to fully charge the fleet for cycle 1. When `No`, the fleet would have to be charged from non-solar sources (typically wind, nuclear, or imports) at the prevailing midday price — the arbitrage still works economically (midday prices remain below evening peak), but it is no longer "solar arbitrage" in the strict sense.

**What is Net ROI?** Net ROI = (10-year gross margin ÷ total investment) − 1. A value of +20% means that for every €100 invested, €120 in gross arbitrage revenue is earned over 10 years (a 20% net return over the full horizon, before operating costs, taxes, or financing). A value of −40% means only €60 is recovered (a 40% net loss).

**▸ Output Data**

| Country | Cycle 1 Spread (€/MWh) | Cycle 2 Spread (€/MWh) | Solar Feasible | Storage (GWh) | Discharge (GW) | Annual Margin (B€) | Investment (B€) | 10y Margin (B€) | Net ROI (10y) |
|---------|-----------------------:|-----------------------:|:--------------:|--------------:|---------------:|-------------------:|----------------:|----------------:|--------------:|
| Hungary        | 89.40 | 46.10 | ☀ Yes | 8.52  | 1.70  | 0.309 | 2.555  | 3.093  | **+21%** |
| Bulgaria       | 88.61 | 44.92 | ☀ Yes | 2.65  | 0.53  | 0.095 | 0.794  | 0.949  | **+20%** |
| Greece         | 79.42 | 33.39 | ☀ Yes | 3.89  | 0.78  | 0.120 | 1.166  | 1.199  | **+3%**  |
| Czech Republic | 54.75 | 36.60 | ⚡ No  | 13.16 | 2.63  | 0.313 | 3.947  | 3.131  | −21%     |
| Austria        | 43.04 | 33.48 | ⚡ No  | 21.24 | 4.25  | 0.416 | 6.373  | 4.160  | −35%     |
| Belgium        | 42.43 | 32.35 | ☀ Yes | 5.02  | 1.00  | 0.096 | 1.505  | 0.962  | −36%     |
| France         | 36.16 | 31.64 | ⚡ No  | 43.29 | 8.66  | 0.740 | 12.986 | 7.405  | −43%     |
| Spain          | 38.78 | 18.45 | ☀ Yes | 71.44 | 14.29 | 1.105 | 21.431 | 11.047 | −48%     |

☀ = strong-solar hours produce enough cheap energy to fully charge the fleet for the evening (cycle 1) discharge. ⚡ = cycle 1 charging would partly draw from non-solar grid sources. (Cycle 2 always charges from overnight hours and is not affected by this check.)

---

## Final Results & Conclusions

### Key Findings

The **solar merit-order effect** is confirmed with statistical significance (p = 0.0000) in all eight countries analysed (Modules 3 and 4). Day-ahead prices are consistently and materially lower during strong solar hours. Price spreads range from €20.72/MWh (France) to €55.74/MWh (Hungary) in the simple solar-vs-non-solar comparison.

The **investment case at utility-scale storage costs**, however, is materially more demanding. Only three of the eight national markets clear break-even over a 10-year horizon with two daily cycles:

- **Hungary (+21%)** and **Bulgaria (+20%)** — wide cycle-1 spreads (~€89/MWh), strong cycle-2 spreads (~€45/MWh), modest fleet sizes, and full solar charge feasibility.
- **Greece (+3%)** — wide cycle-1 spread (€79/MWh) but a narrower cycle-2 spread (€33/MWh). At break-even, sensitive to assumptions.

The remaining five markets are unprofitable at the assumed cost basis: Czechia (−21%), Austria (−35%), Belgium (−36%), France (−43%), and Spain (−48%).

### The Scale Paradox

The unprofitable markets are also the largest investments. Spain (€21.4B) and France (€13.0B) together would require €34.4B in capital for a combined loss of approximately €17B over 10 years. The profitable markets — Hungary, Bulgaria, Greece — require only €4.5B combined to generate ~€0.5B in net 10-year profit.

This is not a result of weak fundamentals in the large markets. Rather, it reflects market saturation (Spain's already-suppressed daytime trough), nuclear baseload smoothing (France's compressed evening peak), and limits on solar charge supply at scale (France, Austria, Czechia cannot physically charge a 50%-target fleet from solar alone).

Notably, **investment size does not predict profitability**. Bulgaria (€0.8B) and Spain (€21.4B) differ by 27× in capital, but Bulgaria is the most profitable and Spain the least. Belgium and Bulgaria require similar capital (€1.5B vs €0.8B), yet one loses 36% and the other gains 20%. The determinant is spread, not scale.

### Priority Markets

- **Hungary** — €89.40/MWh cycle-1 spread, €0.31B annual margin, €2.56B investment. Strongest absolute net return.
- **Bulgaria** — €88.61/MWh cycle-1 spread, €0.10B annual margin, €0.79B investment. Best capital efficiency in the dataset.
- **Greece** — €79.42/MWh cycle-1 spread, €0.12B annual margin, €1.17B investment. Marginally profitable; sensitive to spread assumptions.

### Limitations & Outlook

- **Static spreads.** The model assumes 2024 day-ahead spreads persist for 10 years. Deploying national-scale storage would compress those spreads (exactly the purpose of the deployment), so the model overstates achievable returns. A dynamic price-response model would lower all multiples.
- **Single revenue stream.** Day-ahead arbitrage only; no capacity market, no frequency response, no ancillary services. Real grid batteries typically earn 30–60% of revenue from these other streams. Adding them would push all marginal cases (Greece, Czechia) into profitability.
- **No round-trip efficiency loss.** A 10–15% loss is typical for lithium battery storage and would reduce margins proportionally. Excluded here for simplicity.
- **No operating costs.** O&M, insurance, grid fees, and end-of-life provisions are not modelled.
- **Cycle lifetime.** Two cycles per day implies ~5,000–7,000 cycles over 10 years — within mainstream Li-ion ratings but optimistic for cheaper chemistries. Lifetime sensitivity (8–15 years) is recommended.
- **Solar feasibility check.** The check uses the average solar volume in strong-solar hours from April–October. Day-to-day variation is not modelled; cloudy summer days could shift cycle 1 to grid-charging on those days.
- **Cost basis.** €300/kWh installed reflects 2024 utility-scale system prices for 4-hour-duration projects. Costs are falling 5–10% per year, so a 2030 build would land closer to €200/kWh — at which point Spain and Belgium clear break-even.

---

**Overall, the pipeline confirms the merit-order hypothesis with statistical rigour (p = 0.0000 across all eight countries) but reveals a sharper investment story than headline spreads suggest. At utility-scale storage costs and two daily cycles, Hungary, Bulgaria, and Greece are the only markets where day-ahead arbitrage alone justifies the capital required. The unprofitable markets are not unattractive in absolute terms — their spreads are real and exploitable — but their scale demands either lower cost bases (€200/kWh) or additional revenue stacks (capacity, ancillary services) to clear ROI.**

---

*Data sources: Eurostat `nrg_inf_epc` (Installed Capacity) · ENTSO-E Transparency Platform API (day-ahead prices, actual generation) · Utility-scale Li-ion storage cost reference €300/kWh installed (2024).*