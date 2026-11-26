# Presentation guide and interview Q&A

**Deck:** `Walmart_M5_Forecasting.pptx` (10 slides, about 10 minutes). The full talk track is in each slide's speaker notes.

## The 30-second version (for "tell me about a project")

> I built a demand-forecasting pipeline on Microsoft Fabric with Walmart's M5 data. It forecasts daily sales for all
> 3,049 items in a store, four weeks ahead. The data goes through a bronze/silver/gold lakehouse, Spark builds the
> features, LightGBM does the forecasting and MLflow registers the model. On four weeks of unseen data, the store's
> daily total is off by about 6%, against 14% for a simple moving average. The one thing I'd fix next is a
> consistent 6% under-forecast.

## Numbers to know by heart

| What | Value |
|---|---|
| Store / items | CA_1 / 3,049 items |
| Training rows | ~2.4 million item-days (Feb 2014 – Apr 2016) |
| Holdout | Last 28 known days: 25 Apr – 22 May 2016 |
| Future forecast | 23 May – 19 Jun 2016 |
| Daily total error (MAPE) | 6.1% model vs 14.1% baseline (57% less error) |
| Item-day RMSE / MAE | 2.256 / 1.148 vs baseline 2.422 / 1.205 |
| Best number of trees | 104 (early stopping) |
| By category | Household 5.2% vs 18.0% · Foods 8.4% vs 13.7% · Hobbies 11.6% vs 17.8% |
| Bias | Forecast ~6% below actual volume over the holdout |
| Top features | 90-day rolling mean 41%, 28-day rolling mean 34%, item 14% |

## Likely questions, with answers

**Why is every feature lagged by at least 28 days?**
We forecast up to 28 days ahead. On the day we make the forecast, we don't know sales for the next 28 days.
A 7-day lag would use sales we wouldn't have for most of the horizon, so the scores would look better than
real life. My first version had exactly this mistake.

**Why LightGBM and not ARIMA or Prophet?**
Those fit one model per series: 3,049 models for one store, and each sparse series has little signal. One
global gradient-boosting model learns shared patterns across all items and handles prices, events and
categories as features. It's also what most top M5 solutions used.

**What is the Tweedie objective?**
A loss function for data with many exact zeros plus a long tail of positive values, which is what item-level
retail sales look like. Squared error tends to predict small positive values everywhere instead.

**How did you validate?**
I held out the last 28 known days, matching the forecast horizon. Early stopping on that window chose 104
trees. Then I retrained on all known days with 104 trees for the real future forecast. Using the holdout for
both early stopping and scoring is slightly optimistic. Rolling-origin back-tests are the next step.

**Why is the error 6% on store totals but RMSE 2.26 per item?**
Item-day errors are large relative to sales of 0–2 units, but they partly cancel out when you add items up.
The store total is what a replenishment planner looks at first, so I report both.

**Why does the model under-forecast by 6%?**
Most likely the features (rolling means ending 28 days back) lag behind an upward trend in spring 2016, and
the Tweedie power I chose (1.1) shrinks predictions towards zero. Next I'd add a trend feature and tune the
power.

**What does MLflow add?**
Every run's parameters and metrics are logged, so runs can be compared side by side. The final model is
registered with a version number, so the forecasts can always be traced to the exact model that made them.

**Why bronze / silver / gold?**
Bronze keeps the raw data unchanged, so the pipeline can always be rebuilt. Silver holds the cleaned, feature-ready
data. Gold holds only what the business reads (forecasts and metrics), which keeps Power BI simple and fast.

**How would this run in production?**
Schedule the notebook in a Fabric pipeline after new sales land. Score with the registered model daily,
retrain weekly or monthly, and alert if the holdout error drifts above a threshold.

**What did it cost?**
It ran on a pay-as-you-go F2 capacity, paused when not in use. The full build takes about 10 minutes of
compute, which costs a few cents.

## Be ready to say how you built it

Expect "did you build this yourself?". Be straightforward that you used an AI assistant to help write and
debug the code. Show that you understand and own every decision on this page. Interviewers care far more
that you can explain the 28-day leakage rule and the validation than who typed each line.
