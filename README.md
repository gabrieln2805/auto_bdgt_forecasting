# auto_bdgt_forecasting
Automated Budget Forecasting & Variance Explanation Engine
Traditional ML vs Agentic AI (Local LLM) — A Practical Comparison

Abstract
This project builds and compares two end‑to‑end forecasting and variance‑explanation pipelines for public budget data. The first is a traditional manual workflow using SARIMAX. The second is an agentic AI workflow that uses a local LLM (Ollama) to select models and orchestrate robust backtesting. Results show that a model selected by the agentic pipeline (ETS with additive trend) significantly outperforms the manual SARIMAX on rolling‑origin backtests, highlighting the value of automated model selection when time‑series structure is uncertain. (statsmodels.org)

1) Problem Statement
We need a forecasting and variance‑explanation engine that can:

forecast future budget values,
compute variance between actuals and estimates, and
support exploratory financial metrics.
To evaluate engineering productivity and model performance, we implemented two parallel solutions:

Traditional ML (manual): fixed SARIMAX configuration.
Agentic AI (automated): a chatbot using a local LLM to choose model candidates and run backtests.
2) Data & Reshaping
Raw schema: SpendingData with wide fiscal‑year columns (e.g., 2026-27 Estimates, 2024-25 Actuals) and categorical dimensions (Agency, Fund, Function, etc.).

Steps:

Extract year columns using regex: YYYY-YY (Estimates|Actuals).
Melt to long format and parse:
fy_start (year start)
year_type (Actuals vs Estimates)
Aggregate to yearly totals.
Create:
value = Actuals if present else Estimates
variance = Actuals – Estimates
pct_variance
This produced a clean annual time series for forecasting and variance analysis.

3) Visuals (from the notebook)
Figure 1 — Budget Estimates vs Actuals by Fiscal Year
A line chart shows a long‑term upward trend in spending. Actuals dominate historical years; estimates appear only in recent years because actuals are not yet closed.

Figure 2 — Manual SARIMAX vs Naive Forecast
The SARIMAX line tracks actuals more closely than the naive baseline on the single holdout window, indicating stronger short‑term fit.

Figure 3 — SARIMAX Forecast with 95% Confidence Interval
The prediction interval widens over the forecast horizon, reflecting increasing uncertainty — a standard property of time‑series forecasts. (robjhyndman.com)

4) Traditional ML Approach (Manual SARIMAX)
We implemented a fixed SARIMAX configuration using statsmodels’ state‑space SARIMAX class. (statsmodels.org)

Model: SARIMAX(1,1,1) × (1,1,1,3)
Evaluation: last 3 years as holdout

Holdout results:

MAE ≈ 9.74M
RMSE ≈ 10.95M
MAPE ≈ 3.86%
This looked strong on a single holdout window, which is common with fixed‑origin evaluation.

5) Agentic AI Approach (Local LLM + Tools)
Because API usage was restricted, the agent was run locally using Ollama. The agent’s job was decision‑making; execution was done by deterministic Python tools.

LLM: local Ollama server (HTTP API)
Role: pick candidate model families + horizons
Tools: execute rolling‑origin backtests and report metrics
Ollama provides a local chat endpoint to run LLM inference without cloud dependencies. (ollama.readthedocs.io)

6) Robust Backtesting (Rolling‑Origin)
To avoid overfitting to a single holdout period, we used rolling‑origin evaluation, also known as time‑series cross‑validation. This approach evaluates a model across multiple test windows as the forecast origin rolls forward. (robjhyndman.com)

We evaluated horizons [1, 3, 5], and averaged RMSE across all rolling windows.

7) Agentic Model Diversity & Selection
The agent selected a small, diverse candidate set:

SARIMAX (1,1,1)(1,1,1,3)
ETS with additive trend
Naive baseline
Drift baseline
ETS is implemented via statsmodels ExponentialSmoothing. (statsmodels.org)

Agentic leaderboard (avg RMSE):

ETS ≈ 7.51M
Drift ≈ 7.69M
Naive ≈ 15.1M
SARIMAX ≈ 3.97e8 (unstable)
8) Final Comparison (Manual vs Agentic)
We then compared:

Manual SARIMAX (fixed configuration)
Agentic best model (ETS)
Rolling‑origin averages:

Approach	RMSE	MAPE
Manual SARIMAX	3.97e8	2.93
Agentic ETS	7.51e6	0.037
Interpretation
The series is trend‑dominant with weak seasonality. ETS with an additive trend matches this structure better than a rigid seasonal SARIMAX. The agentic pipeline also avoids manual mis‑specification by testing multiple model families.

9) Discussion
Key takeaway:

A manually chosen SARIMAX can look good on a single holdout but fail under robust backtesting.
The agentic pipeline delivers better accuracy and stability, at the cost of a more complex evaluation loop.
Engineering tradeoff:

Manual: faster to implement, but brittle if the model structure is wrong.
Agentic: more compute, but statistically safer and scalable to new datasets.
10) Limitations
Model diversity was intentionally constrained (SARIMAX, ETS, Naive, Drift).
Results are based on annual aggregates; segment‑level forecasts may behave differently.
LLM was used for orchestration only; it did not generate forecasts directly.
11) Reproducibility Notes
All metrics, plots, and comparisons are produced in a Jupyter notebook.
Forecasting is done with statsmodels SARIMAX and ExponentialSmoothing. (statsmodels.org)
Agentic controller runs locally via Ollama’s chat API. (ollama.readthedocs.io)
Rolling‑origin evaluation follows time‑series cross‑validation principles. (robjhyndman.com)
References
statsmodels SARIMAX documentation. (statsmodels.org)
statsmodels ExponentialSmoothing (ETS). (statsmodels.org)
Rolling‑origin / time‑series cross‑validation. (robjhyndman.com)
Ollama local API documentation. (ollama.readthedocs.io)
