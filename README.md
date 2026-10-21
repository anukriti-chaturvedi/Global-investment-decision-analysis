# Global Investment Decision Engine
## A finance-led case study in historical analysis, forecasting, valuation sensitivity, and FX risk

> **Project status note:** This document reflects the supplied Google Colab notebook and the work evidenced there. The notebook implements data auditing, Microsoft selection, historical metric engineering, a 2024–2028 forecast, Bear/Base/Bull scenarios, indicative DCF calculations, sensitivity analysis, and CSV exports. The Power BI dashboard, full-universe peer export, and integrated FX-adjusted investment analysis are planned follow-up work and should only be marked complete after implementation and validation.

## 1. Executive summary

The Global Investment Decision Engine is a finance-led analytics project designed to structure a path from raw financial datasets to an interpretable management decision-support view. It combines historical financial statement analysis, explicit forecast assumptions, scenario analysis, discounted cash-flow mechanics, sensitivity analysis, and a planned foreign-exchange risk lens.

The main deep-dive company is Microsoft (MSFT), selected from a dataset containing 12 companies. The selection was based on data coverage and continuity: Microsoft has 15 annual observations from 2009 through 2023 and complete coverage across the fields used in the company-screening step. The other companies remain relevant for peer benchmarking and should be retained in the final analytical model.

The current DCF output is **diagnostic and indicative, not decision-ready**. The notebook uses a cash-flow proxy and produces an enterprise-value figure that is implausibly large relative to the dataset's market-cap figure. The correct conclusion is not that the company is mispriced; it is that cash-flow definitions, units and valuation basis must be reconciled before the output can support an investment conclusion.

## 2. Business question and motivation

### Core question

**How can historical financial performance and transparent operating assumptions be translated into a structured view of future performance and indicative value, while making the major valuation risks visible?**

This project was motivated by the need to demonstrate applied finance judgement alongside practical analytics. Rather than building a generic data visualization, the project is organized around questions relevant to financial analysis, FP&A and valuation:

- How have revenue, profitability and cash generation changed over time?
- How complete and comparable are the available company records?
- Which operating assumptions drive a five-year forecast?
- How much does indicative value change when assumptions change?
- What additional validation is needed before a valuation can support a decision?
- How could foreign-exchange movements affect the INR value of a USD-denominated exposure?

### Intended audience

- Financial analysts and FP&A teams
- Management stakeholders reviewing performance and forecast assumptions
- Valuation and investment-analysis interviewers
- Portfolio reviewers assessing analytical judgement and documentation

## 3. Scope and boundaries

### Included in the current notebook

- Load and inspect the financial statements dataset.
- Load and inspect historical exchange-rate data and the country/economy dataset.
- Standardize company and category labels.
- Check coverage, missing values, duplicate company-year keys and negative financial observations.
- Screen companies and select Microsoft for a detailed case.
- Engineer historical growth, margin and cash-flow proxy metrics.
- Build 2024–2028 base forecast assumptions.
- Build Bear, Base and Bull scenarios.
- Calculate an indicative DCF using a 9% WACC and 3% terminal growth assumption.
- Run a WACC/terminal-growth sensitivity analysis.
- Export analytical CSV tables and a data dictionary.

### Planned or requiring further validation

- Export all 12 companies to a separate peer-comparison table.
- Resolve the units of market capitalization versus financial statement values.
- Replace or rigorously validate the cash-flow proxy before using it as DCF cash flow.
- Rebuild DCF on a defensible FCFF basis, where required inputs are available.
- Establish a consistent enterprise-value/equity-value bridge.
- Integrate historical FX data into explicit INR-denominated scenarios.
- Build, test and reconcile the Power BI report.
- Add external sources for valuation assumptions if a market-grounded valuation is intended.

## 4. Data sources

The notebook loads three CSV files:

| Dataset | Observed structure | Intended use |
|---|---|---|
| `Financial Statements.csv` | 161 rows, 23 columns | Main historical financial analysis and company screening |
| `Historical Data.csv` | 6,722 rows, 22 columns | Historical currency exchange rates, including USD/INR |
| `Currencies and Country EconomyWise.csv` | Loaded in Colab; schema/use not yet documented in the final modelling flow | Potential macroeconomic context, subject to relevance and data audit |

The financial statements data contains 12 company tickers: AAPL, AIG, AMZN, BCS, GOOG, INTC, MCD, MSFT, NVDA, PCG, PYPL and SHLDQ. The dataset covers years between 2009 and 2023, but coverage varies by company.

Available fields include year, company, category, market capitalization, revenue, gross profit, net income, EPS, EBITDA, shareholder equity, operating/investing/financing cash flows, current ratio, debt/equity, ROE, ROA, ROI, net profit margin, free cash flow per share, return on tangible equity, employee count and US inflation.

**Data provenance and reproducibility:** keep the original downloaded CSV files unchanged. Perform cleaning and derived calculations in a separate working DataFrame, and export cleaned/derived tables separately. Record the source URL, access date, units and any transformations in the repository README or data dictionary.

## 5. Data audit and preparation

### 5.1 Structural checks

The notebook checks:

- DataFrame dimensions and column names
- Data types and summary statistics
- Minimum and maximum years
- Unique company and category values
- Company-year duplicates
- Missing-value counts
- Company coverage by year
- Availability of core financial fields
- Negative revenue, gross profit, EBITDA and net income observations

### 5.2 Standardization

The source had an accidental trailing space in the `Company ` column name. The notebook removes surrounding whitespace from column names and text values and standardizes category labels to uppercase.

### 5.3 Duplicate and negative-value checks

No duplicate Company-Year rows were found in the overall financial dataset. The negative-value audit found:

- Negative revenue: 0 observations
- Negative gross profit: 0 observations
- Negative EBITDA: 9 observations
- Negative net income: 25 observations

Negative EBITDA and net income are not automatically errors. They may reflect actual operating losses or unusual reporting periods and should be retained unless a source-specific issue is identified.

### 5.4 Missingness and continuity

The notebook profiles missing fields and checks Microsoft for duplicate years and continuous annual coverage. Microsoft has observations for every year from 2009 through 2023, with no duplicate year and no missing year in that sequence.

### 5.5 Derived metrics

The notebook calculates:

- Revenue growth
- EBITDA margin
- Gross margin
- FCF proxy margin
- EBITDA growth
- Net income growth
- FCF proxy growth

The calculations use year-sorted company records and `pct_change()` within each company.

### 5.6 Cash-flow proxy caveat

The notebook defines:

`FCF_Proxy = Cash Flow from Operating + Cash Flow from Investing`

This is a proxy, not necessarily standard free cash flow to the firm. Investing cash flow can include acquisitions, asset sales and other investing items in addition to capital expenditure. A rigorous FCFF model commonly uses EBIT after tax, adds depreciation and amortization, subtracts capital expenditure and changes in operating net working capital. The current source data does not provide all these components as a clean, consistent series.

The proxy is useful for exploratory analysis only if clearly labelled. It should not be represented as audited FCFF or used for a definitive valuation conclusion without further work.

## 6. Company selection

The notebook initially screens companies based on annual coverage and availability of key financial fields. Financial institutions are excluded from the initial non-financial candidate list because their balance sheets, cash flows and valuation drivers differ from those of industrial and technology businesses.

The final non-financial candidates include MSFT, NVDA, AAPL, AMZN, GOOG, INTC, MCD and PCG. Microsoft and NVIDIA both have 15 years and 100% completeness across the selected screening fields. Microsoft is then selected as the deep-dive company.

### Why Microsoft?

- 15 annual observations, 2009–2023
- Continuous year coverage
- Complete coverage for fields used in the screening
- Suitable for demonstrating long-run trend analysis and assumption-driven forecasting

This is a practical modelling choice, not proof that Microsoft is uniquely superior to the other candidates. NVIDIA also passed the same coverage screen. The final narrative should explain that Microsoft is the deep-dive case, while the wider company universe supports a separate benchmarking page.

### Peer-analysis implementation gap

The current final export table `historical_financials.csv` is created from `historical_msft` and contains 15 Microsoft rows. It does not contain all 12 companies. A separate all-company export must be created from the full cleaned financial statements DataFrame before the Power BI peer page can be considered implemented.

## 7. Historical financial analysis

The intended historical analysis covers revenue, growth, EBITDA, profitability margins, net income, operating cash flow, cash-flow proxy, returns, liquidity, leverage and employee count.

Questions to explore include:

1. Is revenue growth accelerating or slowing?
2. Does EBITDA growth track revenue growth?
3. Are margins stable, expanding or normalizing?
4. Does accounting profit translate into operating cash generation?
5. How do returns and leverage evolve?
6. Are any ratios distorted by small or negative denominators?
7. How does Microsoft compare with relevant peers over common periods?

The final dashboard should make company and year filters explicit. When aggregating across companies, margins should generally be calculated from aggregated numerators and denominators rather than taking a simple average of company-level margins, unless the objective is specifically to show the average company.

## 8. Forecast design

The notebook builds a five-year forecast for 2024–2028. The base forecast uses explicit assumptions for:

- Revenue growth
- EBITDA margin
- Net profit margin
- Operating cash-flow margin
- FCF margin

The assumptions are described in the notebook as anchored to recent historical averages and 2023 actual values, with modest normalization over time. They are judgement-based assumptions, not statistical predictions.

A robust final presentation should include an assumptions table that shows, for every forecast year:

- Assumption value
- Historical reference point
- Rationale for increase, decrease or normalization
- Bear/Base/Bull differences
- Sensitivity of the final output to the assumption

Actual historical values and forecasts must be visually distinguished in Power BI.

## 9. Scenario analysis

Three scenarios are implemented:

- **Bear:** lower revenue growth and lower profitability/cash-flow margins
- **Base:** central planning assumptions
- **Bull:** higher revenue growth and profitability assumptions

The scenario model creates annual financial outcomes for 2024–2028 and compares the 2028 outputs and the range of forecast results. These are illustrative scenarios; they should not be described as probability-weighted predictions unless probabilities are explicitly assigned and justified.

The main management value is to show how a different set of assumptions changes revenue, EBITDA, net income and cash-flow proxy outcomes.

## 10. Indicative DCF methodology

The notebook uses:

- Forecast period: 2024–2028
- Base WACC: 9%
- Base terminal growth: 3%
- Cash-flow measure: dataset FCF proxy
- Output type: indicative enterprise value

The DCF mechanics are:

1. Discount each forecast-period cash flow to present value.
2. Calculate terminal value using a perpetual-growth formula.
3. Discount terminal value to present value.
4. Add the present value of forecast cash flows and terminal value.

The terminal-value formula is:

\[
TV = \frac{FCF_{2028}(1+g)}{WACC-g}
\]

where \(g\) is the terminal growth rate. The formula requires WACC to exceed terminal growth.

### Why the current DCF is not decision-ready

The notebook reports a base-case indicative EV of approximately 1,437,881.96 in the model's output scale, while the 2023 market capitalization field is 2,451.23 in USD billions. The resulting comparison is implausible and cannot be interpreted as evidence of mispricing.

Key issues to investigate:

1. Financial statement values and market capitalization may use different scales.
2. The cash-flow proxy is not a rigorously constructed FCFF series.
3. The model does not have a clean standalone cash/debt bridge to reconcile enterprise value to equity value.
4. WACC and terminal growth assumptions have not been independently sourced and documented as market inputs.
5. The units and currency basis of every DCF input and output need to be consistent.

The notebook appropriately labels the DCF as indicative and notes that a definitive Buy/Hold/Sell conclusion requires an appropriate equity-value/market-price benchmark. The final case study should preserve this caveat prominently.

## 11. Sensitivity analysis

The notebook explores sensitivity to WACC and terminal growth. This is important because DCF valuation can be highly sensitive to assumptions about the discount rate and long-term growth.

The final Power BI sensitivity matrix should show:

- Rows: WACC values
- Columns: terminal growth assumptions
- Values: recalculated indicative enterprise value
- Conditional formatting: low-to-high valuation range
- A visible marker or label for the base assumption combination

Before displaying a sensitivity matrix, validate that the model recalculates each cell from the same consistent cash-flow series and unit scale.

## 12. Foreign-exchange risk — planned extension

The notebook loads `Historical Data.csv`, containing 6,722 rows and 22 columns, including USD/INR and other currency pairs. The final export list does not yet include a fully integrated FX-adjusted investment analysis.

A useful extension would define a hypothetical USD-denominated investment or annual cash-flow stream and translate it to INR under different exchange-rate assumptions:

\[
\text{INR value} = \text{USD value} \times \text{USD/INR}
\]

A meaningful FX analysis must specify:

- The exposure being translated (investment value or cash flow)
- The initial and forecast exchange-rate assumptions
- Whether annual average or period-end rates are appropriate
- Appreciation/depreciation scenarios
- The distinction between translation effects and underlying investment performance

Historical FX volatility can inform scenarios, but historical movements do not guarantee future exchange rates. Until this extension is implemented, FX should be described as an intended project component, not a completed quantified result.

## 13. Planned Power BI dashboard

The intended report has six pages.

### Page 1 — Executive Investment Overview
KPI cards for selected-year revenue, EBITDA margin, net margin, market capitalization and validated valuation outputs; historical-versus-forecast revenue line chart; scenario valuation comparison; key assumptions and caveats.

### Page 2 — Historical Financial Performance
Revenue and EBITDA trends, net income, growth rates, gross/EBITDA/net margins, operating cash flow, returns and a detailed financial table.

### Page 3 — Company and Peer Comparison
Company/category/year slicers; revenue and EBITDA comparisons; margin and return benchmarks; scatter plot of revenue growth versus EBITDA margin; company-level financial matrix. This page requires a separate all-company export.

### Page 4 — Forecast and Scenario Analysis
Historical-versus-forecast trend, Bear/Base/Bull revenue and EBITDA trajectories, selected-year scenario comparison and a transparent assumptions table.

### Page 5 — Valuation and Sensitivity
Scenario valuation chart, present-value composition, WACC-versus-terminal-growth sensitivity matrix, key assumptions and a visible model limitations note. This page should only use validated valuation outputs.

### Page 6 — Foreign Exchange Risk
USD/INR history, average/range and annual changes, hypothetical translation scenarios and a table showing the impact of exchange-rate assumptions. This page requires the FX integration extension.

### Design principles

- Use finance-oriented titles that state the question being answered.
- Show currency and scale in chart titles or axis labels.
- Distinguish actuals from forecasts.
- Use one consistent scenario order: Bear, Base, Bull.
- Avoid unnecessary decorative visuals.
- Make limitations visible next to valuation outputs.
- Test all slicers and reconcile visuals to source CSVs.

## 14. Analytical export layer

The notebook exports the following files from `/content/project_exports`:

| File | Purpose |
|---|---|
| `historical_financials.csv` | MSFT historical financial metrics (15 rows in the current notebook) |
| `base_forecast.csv` | Base-case forecast (5 rows) |
| `scenario_forecast.csv` | Bear/Base/Bull forecast (15 rows) |
| `dcf_valuation.csv` | Scenario-level indicative DCF results (3 rows) |
| `dcf_sensitivity.csv` | Sensitivity outputs (4 rows in the current notebook) |
| `risk_assessment.csv` | Risk/assumption assessment table |
| `investment_decision.csv` | Decision framework summary |
| `project_summary.csv` | Project summary items |
| `data_dictionary.csv` | Table/column/type dictionary |

These tables provide a useful reporting layer. Their row counts and values should be rechecked after any model changes. The current sensitivity export has only four rows, so verify whether it contains a sufficiently complete grid for the final heatmap; if not, expand the sensitivity calculation.

## 15. Final insights and decision framing

The current work supports several methodological conclusions:

1. Data coverage and continuity are useful selection criteria, but not sufficient by themselves to establish a company's investment attractiveness.
2. Negative earnings and cash-flow values require financial interpretation rather than automatic removal.
3. Forecast scenarios make assumptions visible and help stakeholders understand a range of outcomes.
4. DCF outputs are only meaningful when the cash-flow definition, unit scale, discount rate and terminal-growth assumptions are internally consistent.
5. Enterprise value and equity market capitalization are different measures and cannot be compared directly without a suitable bridge.
6. A broad company dataset and a focused deep dive serve different purposes: the deep dive supports detailed forecasting; the broader universe supports contextual benchmarking.
7. FX analysis should be connected to a defined USD exposure and translated consistently into INR, not treated as an isolated currency chart.

### Current decision

**No definitive investment recommendation is made from the current DCF output.** The next step is model validation: normalize units, define a defensible valuation cash flow, resolve the EV/equity-value bridge, create the all-company peer export, integrate FX scenarios and reconcile the completed Power BI dashboard to the source tables.


Do not commit raw data unless the dataset licence and redistribution terms permit it. If necessary, provide source links and reproducible preparation steps instead.

## 16. Reproducibility and tools

- **Google Colab:** notebook execution and analysis
- **Python / Pandas / NumPy:** data preparation, metrics, forecast/scenario calculations and DCF calculations
- **CSV:** portable analytical tables for the dashboard
- **Power BI:** intended interactive reporting layer

PostgreSQL is optional for this version because the final data is small and static. It can be added if demonstrating SQL querying is a specific objective, but it is not necessary for the core finance case study.

## 17. Conclusion

The Global Investment Decision Engine is designed to demonstrate an end-to-end finance analytics workflow: audit raw data, define an appropriate case, analyze historical financial performance, make forecast assumptions explicit, test scenarios, examine valuation sensitivity and communicate limitations to a decision-maker.

The most important outcome is not a single valuation number. It is a transparent analytical process that shows what the data supports, what assumptions drive the result and what must be validated before the analysis can be used for a real investment decision.
