# 0005. The stockout forecast ships as a 50/50 blend of a model and a moving average

- **Status:** Accepted
- **Date:** 2026-09-05
- **Driven by:** "Add stockout forecasting, and make it beat the baseline" (2026-08-04); a four-fold rolling backtest

## Context

The forecast predicts how many days of stock each size has left. The first version
predicted one day at a time and fed each guess back in to get the next. It scored
**41% mean absolute error** against a 28-day moving average's **33%** — the lag
features held smooth predictions while the model was trained on noisy real counts,
and the error compounded.

Fixing that — predicting the next 28 days as one total, and pooling every variant
into one model instead of one model per size — closed the gap to a tie, not a win. A
size selling fifteen units a month carries about 26% Poisson noise no matter what,
and the measured error floor across this catalogue is **32.6%**. Both the model and
the baseline were sitting on it.

## Decision

Ship the forecast as `0.5 * model + 0.5 * moving_average` (`forecast.predict_rates`).
The model and the baseline get different things wrong, so averaging cancels some of
the variance: **36.9%** against the baseline's **39.7%** over four rolling windows.

The dashboard shows that three-way comparison — blend, model alone, baseline alone —
under the forecast card, via `/api/forecast/accuracy`, so the blend is never taken
on trust. `beats_baseline` is a boolean in the backtest output.

One model is trained across all variants at once because most sizes have too little
history to fit anything stable alone; pooled, there are ~20,000 rows and each
variant's own recent rate stays a feature.

## Alternatives considered

- **Ship the model alone** — it does not beat the thing anyone would do by eye, so
  it would not earn its dependency on scikit-learn.
- **Ship the moving average alone** — simpler, and honestly close, but it has no
  seasonal signal and cannot see that a whole product line is picking up.
- **A heavier model (Prophet, an RNN)** — the error floor is noise, not model
  capacity. A better model cannot get below 32.6% here, so the added complexity buys
  nothing on this dataset.

## Consequences

- The forecast card has to keep showing its own accuracy. A forecast that hides its
  error rate on a shop this small is asserting, not predicting.
- `HistGradientBoostingRegressor` stays a real dependency, justified by the blend
  beating the baseline and not by the model beating it alone.
- On a much larger store the noise floor drops and the model's share of the blend is
  worth re-tuning — the backtest is the place that decision gets made.
