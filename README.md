# Jane Street Market Prediction Kaggle Competition

The Jane Street Market Prediction competition (Kaggle, Nov 2020 - Feb 2021) challenges us to create a quantitative trading model, one that utilizes real-time market data to help make trading decisions and maximise returns.

The goal of each model is to **predict whether it is better to make a trade or pass on it** at a certain point in time, given an anonymized set of features representing stock market data at that point.

*Note: The training data is not included in the repository because its size (5.4GB) exceeds the allowed file size of Github.*

## The Data

*Modified from the Jane Street competition guidelines*.

This dataset contains an anonymized set of features, `feature_{0...129}`, representing real stock market data. Each row in the dataset represents a trading opportunity. Each trade has an associated `weight` and `resp`, which together represents a return on the trade. The `date` column is an integer which represents the day of the trade, while `ts_id` represents a time ordering.

In the training set, **train.csv**, we are provided a `resp` value, as well as several other `resp_{1,2,3,4}` values that represent returns over different time horizons. These variables are not included in the test set. Trades with `weight = 0` were intentionally included in the dataset for completeness, although such trades will not contribute towards the scoring evaluation.

The model should predict an `action` value for every row: `1` to execute the trade and `0` to pass on it. This decision should be based on the predicted `resp` values.

The training data has 2390491 entries.

## Models

This repository contains the following predictive models:

- Keras Long-Short Term Memory (LSTM) model
- Keras Multi-layer Perceptron (MLP) model

### Keras LSTM model

#### Introduction

I opted to use a **Long Short-Term Memory (LSTM)** model because market data is a Time Series. Analysing past patterns to predict future performance is already established in Fundamental market analysis, so I decided to have the model take into account past data in addition to current data.

#### Data Preparation

The LSTM model expects a windowed training dataset. As I am using a lookback value of 15, this would result in the training data increasing in size by a factor of 15. The shape of the dataset will become (2390491, 15, 131), where 2390491 is the number of entries, 15 is the lookback of the LSTM and 131 is the number of features.

#### Model Architecture

The model architecture is as follows:

- Input [Input Layer, Batch Normalization, Dropout]
- LSTM (64 units) [LSTM, Dropout]
- LSTM (64 units) [LSTM, Dropout]
- Dense (512 units, Swish Activation) [Dense, Batch Normalization, Activation, Dropout]
- Dense (256 units, Swish Activation) [Dense, Batch Normalization, Activation, Dropout]
- Output (5 units, Sigmoid Activation) [Dense, Activation]

The optimizer used was Adam, the loss was Binary-Crossentropy and the metrics used were AUC-ROC and accuracy. Label smoothing and early stopping were also used.

The model was trained for 20 epochs, with early stopping at epoch 17.

#### Results and Observations

Despite my initial optimism that an LSTM will be an improvement over simply using a standard multi-layer perceptron (MLP), the model did not perform well. The AUC-ROC was very close to 0.5, indicating that the model had little to no distinguishing power, even on the training data. The accuracy was also low, hovering around 15-20%.

The poor performance might be due to the very short time between each data point. A lookback of 10, 50 or even 100 will only retain data from a short period of time into the past. In contrast, Fundamental Analysis tends to look at data going back hours, days or weeks. With such a short lookback, the data is also likely very noisy.

This LSTM approach was also much more resource-intensive than simpler approaches, due to the windowing of the data increasing the size of the data processed by a factor of the lookback value and the complexity of an LSTM model relative to a MLP. This limited the amount of tuning and epochs I could run due to Kaggle Notebooks' computing limitations.

Ultimately, I conclude that the model, as it is, is ill-suited for this problem.

### Keras MLP model

#### Introduction

After the poor performance of the LSTM model, I decided it will be best to avoid looking back through the data and returning to the basics. This is a **Multi-layer Perceptron (MLP)** model. With 131 features in the dataset, a basic MLP should have reasonable performance despite its simplicity and inability to take time into account.

#### Model Architecture

The model architecture is as follows:

- Input [Input Layer, Batch Normalization, Dropout]
- Dense (256 units, Swish Activation) [Dense, Batch Normalization, Activation, Dropout]
- Dense (256 units, Swish Activation) [Dense, Batch Normalization, Activation, Dropout]
- Dense (256 units, Swish Activation) [Dense, Batch Normalization, Activation, Dropout]
- Output (5 units, Sigmoid Activation) [Dense, Activation]

The optimizer used was Adam, the loss was Binary-Crossentropy and the metrics used were AUC-ROC and accuracy. Label smoothing and early stopping were also used.

The model was trained for 50 epochs, with early stopping at epoch 21.

#### Results and Observations

Compared to the previous LSTM model, this MLP model had a much better performance. While the accuracy of the model is comparable at 15-30%, the AUC-ROC is consistently higher than 0.56, indicating that the model has significantly more distinguishing power than the LSTM model. Despite the inability to look back into past data, it seems that the 131 features provide enough data to produce a good prediction of returns.

Sometimes the basic approach is best.

## License

This study is licensed by the GNU General Public License v3.0.

GNU © Nicholas Ho