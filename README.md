# Global-Trade-Risk-and-Direction
Two-lens risk framework for 50 global trade corridors (2000–2024): a structural exposure scorecard and a next-year growth forecast, tested against each other to show they measure different things.

Overview:

Supply chain risk dashboards usually collapse everything into one score. This project splits that score into two questions, then tests whether the split is necessary.

Lens 1, structural exposure: how fragile is this route now, given its concentration, volatility, and the industries it serves?
Lens 2, growth direction: will trade on this route grow or shrink next year?

You would assume a fragile route is a shrinking route. We tested that on 1,100 historical corridor-years and found no detectable relationship (Spearman rho = 0.053, p = 0.077). Fragility and direction are near-independent properties of a trade route, so the framework keeps them as two lenses instead of one blended number.

Key findings:

Exposure does not predict direction. The scorecard catches contractions with 39% precision and 16% recall, 56% accuracy overall. It is a structural vulnerability monitor, not a forecaster, and we say so.

The forecast beats a naive baseline by 47%. Gradient Boosting reaches 6.75 test MAE against 12.71 for "next year repeats this year," with 60% directional accuracy on data held out from both training and model selection.

Built-in feature importance was misleading. The model ranked a global supply-chain pressure index second at 25.6%. Permutation testing on held-out data scored it at −0.089, meaning shuffling it did not hurt predictions. It is one number per year, identical across all 50 corridors, so the model was separating years rather than corridors. Real signal came from each route's own history: current growth, momentum, trade size.

Tuning confirmed the model rather than improving it. A time-series-aware randomized search scored worse on validation (7.19) than the hand-picked config (6.35), so performance is not riding on a lucky parameter choice.

One shock drives most of the error. Russia → EU in 2022 is a 9.48-sigma residual: down 68% under sanctions, then up 217% off the collapsed base. Excluding it, test MAE falls to 4.79 and RMSE to 6.03. Both are reported.

Data:

Five public Kaggle datasets merged into one balanced panel: trade flows, shipping rates, tariff timeline, commodity prices, disruption events, and industry vulnerability.

Final panel: 1,250 rows (50 corridors × 25 years), 58 columns, zero missing corridor-years. Commodity prices start in 2010, so those columns are structurally missing before then. Documented, not imputed.

Method:

Sector standardization. A keyword classifier reduces 18 inconsistent raw trade categories to 8 sectors, reading each goods description and scoring it against sector dictionaries with a confidence score attached.

Feature engineering. Everything is computed within corridor to prevent cross-route leakage: lagged values, 3-year rolling average and volatility, momentum, share of global trade, disruption intensity, industry vulnerability. The target is next year's growth, so every predictor uses only information available at decision time.

Chronological split, never random, since a random split leaks the future into training:

Split	Years	Observations
Train	2000–2019	1,000
Validation	2020–2021 (COVID shock)	100
Test	2022–2023 (energy shock)	100

Both evaluation windows are deliberately shock periods. Selection used validation only; the test set stayed untouched until the end.

Risk scorecard. Five binary flags (negative 3-year growth, negative momentum, top-quartile volatility, top-quartile concentration, top-quartile industry vulnerability) sum to 0–5 and map to Low / Moderate / High / Critical. Thresholds are data-driven percentiles, not hand-picked.

Empirical validation. The scorecard is rebuilt across all 1,100 corridor-years with known outcomes and tested for whether it predicts contraction. The first key finding is that result.

Results:

Validation, used for model selection:

Model	MAE	R²
Gradient Boosting	6.35	0.446
Random Forest	7.83	0.210
Ridge Regression	8.24	0.240
Naive baseline	21.61	−2.686

Test (2022–2023, held out):

	MAE	RMSE	R²
Naive baseline	12.71	37.42	−1.798
Gradient Boosting	6.75	20.94	0.124
Gradient Boosting, outlier excluded	4.79	6.03	−0.053

R² is unstable on 100 observations dominated by one extreme shock, so MAE is the headline metric.

Structural exposure for 2024: 1 Low, 35 Moderate, 13 High, 1 Critical. Crossing exposure with economic importance gives four quadrants, of which the important one is Strategic Exposure: 3 corridors, $401B of trade that are both systemically important and structurally fragile (China → Japan, Japan → China, USA → China).

Limitations:
Small test window: 100 observations over two years, dominated by one shock.
R² is unreliable at this sample size. Trust MAE and RMSE.
Shipping-cost and commodity-price features were built but never entered the model, due to a column-naming mismatch. The model uses 13 numeric features.
60% directional accuracy beats chance but is not reliable, so forecasts are directional guidance, not commitments.
The scorecard's category labels are an analyst-designed convention, not a statistical prediction.
Repository
├── Data_Science_Project_Final_Code.ipynb   # Full analysis, Blocks 1–9
├── trade_corridor_dashboard.html           # Interactive dashboard, self-contained
├── data/                                   # Source datasets
└── README.md

The dashboard is a single HTML file with the analysis output embedded. Open it in any browser, no server needed.

Run the notebook top to bottom to reproduce. Requires pandas, numpy, scikit-learn, scipy, matplotlib, seaborn.

