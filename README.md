<div align="center">
    <h1>Time-series machine learning and embeddings at scale</h1>
<br />

![functime](https://github.com/descendant-ai/functime/raw/main/docs/img/banner.png)
[![Python](https://img.shields.io/pypi/pyversions/functime)](https://pypi.org/project/functime/)
[![PyPi](https://img.shields.io/pypi/v/functime?color=blue)](https://pypi.org/project/functime/)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![GitHub Publish to PyPI](https://github.com/descendant-ai/functime/actions/workflows/publish.yml/badge.svg)](https://github.com/descendant-ai/functime/actions/workflows/publish.yml)
[![GitHub Build Docs](https://github.com/descendant-ai/functime/actions/workflows/docs.yml/badge.svg)](https://docs.functime.ai/)
[![GitHub Run Quickstart](https://github.com/descendant-ai/functime/actions/workflows/quickstart.yml/badge.svg)](https://github.com/descendant-ai/functime/actions/workflows/quickstart.yml)

</div>

---
**functime** is a powerful [Python library]((https://pypi.org/project/functime/)) for production-ready forecasting and temporal embeddings.

**functime** also comes with time-series [preprocessing](https://docs.functime.ai/ref/preprocessing/) (box-cox, differencing etc), cross-validation [splitters](https://docs.functime.ai/ref/cross-validation/) (expanding and sliding window), and forecast [metrics](https://docs.functime.ai/ref/metrics/) (MASE, SMAPE etc). All optimized as [lazy Polars](https://pola-rs.github.io/polars-book/user-guide/lazy/using/) transforms.

Want to use **functime** for seamless time-series predictive analytics across your data team?
Looking for production-grade time-series machine learning in a [serverless](#serverless-deployment) Cloud deployment?
Shoot Chris a message on [LinkedIn](https://www.linkedin.com/in/chrislohy/) to learn more about `functime` Cloud.

## Highlights
- **Fast:** Forecast 100,000 time series in seconds *on your laptop*
- **Efficient:** Embarrassingly parallel [feature engineering](https://docs.functime.ai/ref/preprocessing/) for time-series using [`Polars`](https://www.pola.rs/)
- **Battle-tested:** Machine learning algorithms that deliver real business impact and win competitions
- **Exogenous features:** supported by every forecaster
- **Backtesting** with expanding window and sliding window splitters
- **Automated lags and hyperparameter tuning** using [`FLAML`](https://github.com/microsoft/FLAML)
- **Censored forecaster:** for zero-inflated and thresholding forecasts

## Getting Started
Install `functime` via the [pip](https://pypi.org/project/functime) package manager.
```bash
pip install functime
```

### Forecasting

```python
import polars as pl
from functime.cross_validation import train_test_split
from functime.feature_extraction import add_fourier_terms
from functime.forecasting import linear_model
from functime.preprocessing import scale
from functime.metrics import mase

# Load commodities price data
y = pl.read_parquet("https://github.com/descendant-ai/functime/raw/main/data/commodities.parquet")
entity_col, time_col = y.columns[:2]

# Time series split
y_train, y_test = y.pipe(train_test_split(test_size=3))

# Fit-predict
forecaster = linear_model(freq="1mo", lags=24)
forecaster.fit(y=y_train)
y_pred = forecaster.predict(fh=3)

# functime ❤️ functional design
# fit-predict in a single line
y_pred = linear_model(freq="1mo", lags=24)(y=y_train, fh=3)

# Score forecasts in parallel
scores = mase(y_true=y_test, y_pred=y_pred, y_train=y_train)

# Forecast with target transforms and feature transforms
forecaster = linear_model(
    freq="1mo",
    lags=24,
    target_transform=scale(),
    feature_transform=add_fourier_terms(sp=12, K=6)
)
```

View the [full walkthrough](https://docs.functime.ai/forecasting.md) on forecasting with `functime`.

## Embeddings

Currently in closed-beta for `functime` Cloud.
Have an interesting use-case? Contact us at [Calendly](https://calendly.com/functime).

Temporal embeddings measure the relatedness of time-series.
Embeddings are more accurate and efficient compared to statistical methods (e.g. Catch22) for characteristing time-series.[^1]
Embeddings have applications across many domains from finance to IoT monitoring.
They are commonly used for the following tasks:

- **Matching:** Where time-series are ranked by similarity to a given time-series
- **Classification:** Where time-series are grouped together by matching patterns
- **Clustering:** Where time-series are assigned labels (e.g. normal vs irregular heart rate)
- **Anomaly detection:** Where outliers with unexpected regime / trend changes are identified

View the [full walkthrough](https://docs.functime.ai/embeddings.md) on temporal embeddings with `functime`.

## Serverless Deployment

Currently in closed-beta for `functime` Cloud.
Contact us for a demo via [Calendly](https://calendly.com/functime).

Deploy and train forecasters the moment you call any `.fit` method.
Run the `functime list` CLI command to list all deployed models.
Finally, track data and forecasts usage using `functime usage` CLI command.

![Example CLI usage](docs/img/functime_cli_usage.gif)

You can reuse a deployed model for predictions anywhere using the `stub_id` variable.
Note: the `.from_deployed` model class must be the same as during `.fit`.
```python
forecaster = LightGBM.from_deployed(stub_id)
y_pred = forecaster.predict(fh=3)
```

## License
`functime` is distributed under [AGPL-3.0-only](LICENSE). For Apache-2.0 exceptions, see [LICENSING.md](https://github.com/descendant-ai/functime/blob/HEAD/LICENSING.md).
