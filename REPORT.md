### Exercise 4: target transformation

The original HistGradientBoostingRegressor performed substantially worse than Ridge on validation (R² = 0.1983, MAE = 1,311.6 versus R² = 0.6439, MAE = 836.5). I first tried a log1p target transformation, but this did not improve performance (R² = 0.1675, MAE = 1,342.0).

Instead, I tested a trend-plus-residual approach. I fit a linear time trend using only the training period, trained HGB on the residuals after subtracting that trend, and added the projected trend back to HGB's predictions. This achieved validation R² = 0.7588 and MAE = 653.6, outperforming both original HGB and Ridge. A trend-only model achieved R² = 0.1771 and MAE = 1,330.3, suggesting that HGB contributes predictive value beyond the linear trend.

I consider this approach defensible as a hybrid model because the trend is estimated from historical training data and projected forward, without using validation or test targets to fit it. However, it is not a simple target transformation or a plain HGB model: it explicitly introduces a linear time-trend component. After selecting the approach using validation, I evaluated the test set once, at the end, achieving R² = 0.6019 and MAE = 810.2. I did not tune the model based on test results.
