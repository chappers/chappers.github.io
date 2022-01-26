---
layout: post
category:
tags:
tagline:
---

Using `tsfresh` to generate features quickly without thinking too hard...

Rough sketch is below. I'm not perfectly happy with the code, but using `tsfresh` in conjunction with `shap` and `isolationforest` should provide a really good starting point for bootstrapping timeseries modelling, particularly when you don't have any labels. We can make use of this feature engineering pipeline to build (hopefully) sensible profiles.

```py
from tsfresh.examples.robot_execution_failures import (
    download_robot_execution_failures,
    load_robot_execution_failures,
)
from tsfresh import extract_features
from sklearn.ensemble import IsolationForest
from sklearn.feature_selection import SelectFromModel
from sklearn.preprocessing import FunctionTransformer
from tsfresh.utilities.dataframe_functions import get_range_values_per_column

from sklearn.pipeline import make_pipeline
from sklearn.impute import SimpleImputer
import numpy as np
import shap

import tsfresh

download_robot_execution_failures()
timeseries, _ = load_robot_execution_failures()


def impute_dataframe_range(df_impute, col_to_max, col_to_min, col_to_median):
    columns = df_impute.columns

    # Make the replacement dataframes as large as the real one
    col_to_max = pd.DataFrame([col_to_max] * len(df_impute), index=df_impute.index)
    col_to_min = pd.DataFrame([col_to_min] * len(df_impute), index=df_impute.index)
    # col_to_median = pd.DataFrame(
    #     [col_to_median] * len(df_impute), index=df_impute.index
    # )

    df_impute.where(df_impute.values != np.PINF, other=col_to_max, inplace=True)
    df_impute.where(df_impute.values != np.NINF, other=col_to_min, inplace=True)
    # df_impute.where(~np.isnan(df_impute.values), other=col_to_median, inplace=True)

    df_impute.astype(np.float64, copy=False)

    return df_impute.dropna(axis=1, how="all")


def extract_features_and_impute(
    *args, return_statistics=False, col_to_max=None, col_to_min=None, **kwargs
):
    df = extract_features(*args, **kwargs)

    if col_to_max is None or col_to_min is None:
        col_to_max, col_to_min, _ = get_range_values_per_column(df)

    # these dictionaries need to be maintained!
    if return_statistics:
        return col_to_max, col_to_min
    else:
        df = impute_dataframe_range(df, col_to_max, col_to_min, {})
        return df


from sklearn.base import TransformerMixin, OutlierMixin


class PandasTransformer(TransformerMixin):
    def __init__(self, est):
        self.est = est

    def fit(self, X, y=None):
        self.est.fit(X, y)
        return self

    def transform(self, X):
        X_out = pd.DataFrame(self.est.transform(X))
        X_out.columns = X.columns
        return X_out


# this doesn't really work properly? - as it is stateful
pipeline = make_pipeline(
    FunctionTransformer(
        lambda x: extract_features_and_impute(x, column_id="id", column_sort="time")
    ),
    PandasTransformer(SimpleImputer(strategy="median")),
)

X_out = pipeline.fit_transform(timeseries)
isof = IsolationForest()
isof.fit(X_out)
explainer = shap.TreeExplainer(isof)
shap_values = explainer.shap_values(X_out)
vals = np.abs(shap_values).mean(0)
col_select = X_out.columns[vals > 0]

feature_info = dict(zip(list(X_out.columns), vals))
# dict(sorted(feature_info.items(), key=lambda x: x[1], reverse=True)[:10])

selected_features_info = tsfresh.feature_extraction.settings.from_columns(col_select)
col_to_max, col_to_min = extract_features_and_impute(
    return_statistics=True,
    timeseries_container=timeseries,
    column_id="id",
    column_sort="time",
)  # save statistics

# then we can reuse the information to have a better extract function
subset_timeseries = extract_features_and_impute(
    timeseries,
    column_id="id",
    column_sort="time",
    kind_to_fc_parameters=selected_features_info,
    col_to_max=col_to_max,
    col_to_min=col_to_min,
)

```

What improvements can be made?

- In production, the statistics needs to be saved, so they are not affected by the extract batch - this is particularly important when scoring models
- Better feature selection approach - we could do several rounds of isolation forest for example, to trim down the features selected.
