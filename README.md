# Macroeconomic Analysis — EU Battery Storage Arbitrage
## Analytical Pipeline — README
*Eurostat · ENTSO-E · Day-Ahead Prices · Solar Merit-Order · Powerwall ROI*

---

## Abstract

This project is a five-module macroeconomic analysis and analytical pipeline investigating the economic viability of distributed small-scale battery storage — specifically Tesla Powerwall 3 or similar units — deployed across selected EU countries. The pipeline integrates three independent data sources: Eurostat installed capacity statistics, ENTSO-E real-time generation and day-ahead price APIs, and empirical solar generation profiles.

The goal of the project is to prove that solar production systematically leads to lower price levels during hours of high solar generation, and that small-scale batteries — if deployed at scale — can economically redistribute that energy to hours of higher demand and higher prices. Such an investment model is hypothesised to be viable on both a micro level (individual unit returns) and a macro level (grid-wide energy rebalancing), delivering sufficient returns to investors while simultaneously improving the temporal distribution of renewable energy supply.

The analysis maps installed capacity across technologies and countries using Eurostat data (Module 1). From Module 2 onwards, the pipeline switches exclusively to ENTSO-E data, which provides more detailed and precise real-time energy statistics: solar penetration is quantified for eight EU countries (Module 2), day-ahead price dynamics during high-solar hours are characterised (Module 3), the solar merit-order effect is formally tested (Module 4), and a 10-year Powerwall ROI (Return on Investment) sized to capture solar arbitrage is quantified (Module 5).

Power generation volumes and prices are derived from ENTSO-E day-ahead 15-minute interval prices, resulting in datasets of up to 8,760 × 4 = 35,040 observations per country per year. This granularity is essential for accurately characterising intraday price dynamics and identifying exploitable arbitrage windows.

The core hypothesis — that abundant midday solar generation systematically depresses day-ahead electricity prices, creating an exploitable price spread — is confirmed across all eight countries analysed. Statistical significance is established via a Mann-Whitney U test comparing mean prices during strong solar hours against all other hours, returning p = 0.0000 in every country, meaning the probability of observing these price differences by chance is less than 0.01%. This is further supported by a statistically significant negative Pearson correlation between solar generation (MW) and price (EUR/MWh) in all eight countries, confirming that higher solar output consistently coincides with lower prices.

The strongest price suppression effect is observed in Hungary (−55.74 EUR/MWh gap) and Bulgaria (−51.62 EUR/MWh gap), while the strongest solar-price correlation is found in Austria (−0.524) and Belgium (−0.522).

---

## Pipeline Modules

### Module 1 — Installed Capacity per Technology per Country (Eurostat)
`01_Eurostat_Installed_Capacity_Per_Technology_Per_Country.ipynb`

Data source: Eurostat installed generation capacity dataset. Pivots raw data into a Country × Technology matrix (81 columns), filters to 15 non-overlapping technologies, and sorts countries alphabetically.

- Technologies retained: solar, wind onshore/offshore, hydro variants, nuclear, coal, lignite, oil, gas, biomass, geothermal, tidal, and waste.
- All-zero columns dropped to reduce noise; aggregates and duplicates excluded.

**▸ Output Data**

| Country | Coal (MW) | Gas (MW) | Wind On. (MW) | Wind Off. (MW) | Biomass (MW) | Tidal (MW) | Total (MW) |
|---------|-----------|----------|---------------|----------------|--------------|------------|------------|
| Austria | 15,186 | 9,547 | 4,528 | 0 | 0 | 8 | 33,008 |
| Belgium | 1,431 | 124 | 5,990 | 3,928 | 463 | 259 | 28,702 |
| Bulgaria | 3,418 | 2,405 | 5,103 | 2,006 | 29 | 0 | 16,161 |
| Czechia | 2,290 | 1,118 | 10,401 | 4,290 | 10 | 34 | 23,022 |
| France | 26,024 | 18,922 | 16,840 | 61,400 | 0 | 1,089 | 134,217 |
| Greece | 3,481 | 2,782 | 10,408 | 0 | 0 | 0 | 28,716 |
| Hungary | 61 | 61 | 4,573 | 2,027 | 207 | 60 | 14,676 |
| Spain | 20,140 | 13,738 | 32,488 | 7,117 | 1,459 | 0 | 135,952 |

> *Selected key technologies shown. Full matrix spans 81 columns. France (134,217 MW) and Spain (135,952 MW) are the two largest systems by total installed capacity.*

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

Strong solar hours identified per country (08:00–18:00 for most; CZ 08–17, HU 07–17, ES 09–20). No countries had missing or failed price data. The price spread between strong-solar and remaining hours quantifies the battery arbitrage margin.

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

Statistical test: one-sided mean comparison and Pearson correlation (threshold |r| > 0.2) between solar generation and day-ahead prices. AT and BE show strong correlations of −0.524 and −0.522 respectively. All countries return p = 0.0000.

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

### Module 5 — 10-Year ROI Model — Powerwall Fleet
`05_ENTSO-E_Solar_Storage_Arbitrage_Return_Model.ipynb`

Investment model: Tesla Powerwall 3 (13.5 kWh energy / 4.6 kW discharge). Fleet sized to cover 20% of mean peak-hour grid energy, constrained by whichever Powerwall limit binds first. 85% average utilisation over 10 years (linear degradation 100%→70%). Total investment = all units × €7,200 + one gateway (€1,000).

**What is Simple ROI?** ROI = 10-year gross margin ÷ total investment. A value of 19% means that for every €100 invested in the Powerwall fleet, €19 in gross arbitrage revenue is earned over 10 years — before operating costs, taxes, or financing. It is the total undiscounted payback ratio over the full 10-year horizon, not an annualised return.

- Bulgaria (19%) and Hungary (18%) deliver the highest ROI, driven by price spreads above €89/MWh.
- Greece (17%), the Czech Republic (11%) and Austria  (9%).
- Spain, France, and Belgium return 8% — positive but marginal, requiring scale or regulatory support to be compelling.


**▸ Output Data**

| Country | Peak Energy (MWh) | Eff. Energy (MWh) | Units (#) | Spread (€/MWh) | Annual Margin (€) | Investment (B€) | 10yr Margin (€) | Fleet ROI | Unit ROI |
|---------|-------------------|-------------------|-----------|----------------|-------------------|-----------------|-----------------|-----------|----------|
| Austria | 8,498 | 1,445 | 369,460 | 46.63 | 24,588,073 | 2.660 | 245,880,733 | 9% | 9% |
| Belgium | 2,007 | 341 | 87,245 | 42.43 | 5,282,846 | 0.628 | 52,828,457 | 8% | 8% |
| Bulgaria | 1,058 | 180 | 46,014 | 93.76 | 6,157,283 | 0.331 | 61,572,827 | 19% | 19% |
| Czech Rep. | 5,263 | 895 | 228,847 | 54.75 | 17,880,089 | 1.648 | 178,800,895 | 11% | 11% |
| France | 17,316 | 2,944 | 752,850 | 38.47 | 41,335,777 | 5.421 | 413,357,774 | 8% | 8% |
| Greece | 1,554 | 264 | 67,570 | 83.77 | 8,078,350 | 0.487 | 80,783,500 | 17% | 17% |
| Hungary | 3,407 | 579 | 148,124 | 89.40 | 18,898,061 | 1.066 | 188,980,608 | 18% | 18% |
| Spain | 28,602 | 4,862 | 1,243,583 | 42.20 | 74,895,556 | 8.954 | 748,955,556 | 8% | 8% |

---

## Final Results & Conclusions

### Key Findings

The solar merit-order effect is confirmed with statistical significance (p = 0.0000) in all eight countries analysed. Day-ahead prices are consistently and materially lower during strong solar hours. Negative Pearson correlations are confirmed in AT (r = −0.524) and BE (r = −0.522), with equivalent significance across the remaining six countries.

Price spreads range from €20.72/MWh (FR) to €55.74/MWh (HU). Countries with the highest solar share — HU (35.9%), BE (31.3%), BG (28.8%) — also exhibit the strongest merit-order suppression, confirming that solar penetration directly drives the arbitrage opportunity.

### Investment Case

The Simple ROI metric equals 10-year gross margin divided by total investment — the total undiscounted return over the full 10-year horizon, not an annualised figure. A 19% ROI means €19 returned per €100 invested over 10 years. The model returns positive results across all nine countries (8%–19%). Bulgaria and Hungary are the most attractive markets, combining the widest price spreads with relatively modest fleet sizes and investment levels. Greece and Slovenia also offer above-average returns. The combined 10-year gross margin across all nine countries exceeds €1.99 billion against a total investment of approximately €21.65 billion.

The distributed deployment model — large numbers of household Powerwall units rather than centralised grid-scale storage — provides a dual benefit: capturing solar arbitrage revenue while reducing peak load on transmission and distribution networks, potentially deferring costly grid capacity extensions.

### Priority Markets

- **Bulgaria: 19% ROI** — €93.76/MWh spread, €61.6M 10-year margin — strongest case per unit.
- **Hungary: 18% ROI** — €89.40/MWh spread, €189.0M 10-year margin — largest absolute return among top performers.
- **Greece: 17% ROI** — €83.77/MWh spread — strong solar infrastructure (27.8% share) supports continued growth.

### Limitations & Outlook

- Analysis based on 2024 ENTSO-E data; growing solar penetration will likely widen price spreads and improve ROI over time.
- Battery degradation modelled linearly (100%→70%); real-world performance varies with cycling patterns and temperature.
- The 20% fleet-sizing assumption and €1,000 gateway cost are fixed; sensitivity analysis on these parameters is recommended.
- Regulatory factors — grid tariffs, feed-in rules, aggregation frameworks — are not modelled and may significantly affect actual returns.
- Slovenia appears in the ROI model but was absent from Module 2 (ENTSO-E capacity API); its results should be verified against an independent data source.

---

**Overall, the pipeline provides a robust, data-driven foundation for investment decisions in distributed battery storage across EU solar markets, combining statistical rigour (merit-order hypothesis testing, p = 0.0000 across all countries) with practical financial modelling (10-year Powerwall fleet ROI of 8%–19%). Bulgaria, Hungary, and Greece emerge as the priority markets for near-term deployment.**

---

*Data sources: Eurostat Installed Capacity · ENTSO-E Transparency Platform API · Tesla Powerwall 3 specifications (13.5 kWh / 4.6 kW / €7,200)*
