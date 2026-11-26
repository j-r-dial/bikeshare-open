### Results

Both runs trained on 2011, validated on the first half of 2012, and tested on the second half of 2012. The after run adds `cnt_roll_3`, the mean ride count over the three calendar days before the prediction date. MAE is measured in rides.

| Feature set | Model | Validation R² | Validation MAE | Test R² | Test MAE |
| --- | --- | ---: | ---: | ---: | ---: |
| Before | `baseline_mean` | -0.9318 | 2002.2 | -2.1441 | 2847.7 |
| Before | `baseline_persistence` | 0.4507 | 900.6 | 0.5099 | 840.1 |
| Before | `ridge` | 0.6475 | 832.9 | 0.5833 | 947.1 |
| Before | `hgb` | 0.2185 | 1296.6 | -0.4975 | 1972.6 |
| After | `baseline_mean` | -0.9318 | 2002.2 | -2.1441 | 2847.7 |
| After | `baseline_persistence` | 0.4507 | 900.6 | 0.5099 | 840.1 |
| After | `ridge` | 0.6439 | 836.5 | 0.58 | 946.2 |
| After | `hgb` | 0.1983 | 1311.6 | -0.5338 | 1998.7 |

### Headline finding

The leaky demonstration gave R² = 1.000 because casual and registered rides sum to the target. Under the temporal split with `cnt_roll_3`, ridge achieved test R² = 0.58 and MAE = 946.2 rides. It beat the training-mean baseline (R² = −2.1441, MAE = 2847.7) and had a higher R² than persistence (0.5099), while persistence had the lower MAE of 840.1 rides. Histogram gradient boosting struggled to extrapolate the rise in demand from 2011 to 2012, with test R² = −0.5338 and MAE = 1998.7.

Adding `cnt_roll_3` lowered validation R² by 0.0036 for ridge and 0.0202 for histogram gradient boosting and increased both validation MAEs. I kept the feature in the repository to preserve the evaluated experiment.
### Exercise 4: target transformation

The original HistGradientBoostingRegressor performed substantially worse than Ridge on validation (R² = 0.1983, MAE = 1,311.6 versus R² = 0.6439, MAE = 836.5). I first tried a log1p target transformation, but this did not improve performance (R² = 0.1675, MAE = 1,342.0).

Instead, I tested a trend-plus-residual approach. I fit a linear time trend using only the training period, trained HGB on the residuals after subtracting that trend, and added the projected trend back to HGB's predictions. This achieved validation R² = 0.7588 and MAE = 653.6, outperforming both original HGB and Ridge. A trend-only model achieved R² = 0.1771 and MAE = 1,330.3, suggesting that HGB contributes predictive value beyond the linear trend.

I consider this approach defensible as a hybrid model because the trend is estimated from historical training data and projected forward, without using validation or test targets to fit it. However, it is not a simple target transformation or a plain HGB model: it explicitly introduces a linear time-trend component. After selecting the approach using validation, I evaluated the test set once, at the end, achieving R² = 0.6019 and MAE = 810.2. I did not tune the model based on test results.

### Exercise 5: ridge vs persistence

I would deploy the persistence baseline if the main goal is to minimize the average daily forecast error in rides. On the test set, ridge had the higher R² (0.5800 versus 0.5099), a gain of 0.0701. Persistence had the lower MAE (840.1 versus 946.2 rides), so its forecasts missed by 106.1 fewer rides per day on average. Since MAE directly measures how far the forecast is from the daily ride count, I would use yesterday’s count as the forecast.
