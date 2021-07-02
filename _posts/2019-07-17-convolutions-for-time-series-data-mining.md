---
layout: post
category : 
tags : 
tagline: 
---

When we're moving towards time series or transactional data; it is interesting to think about how features are generated and how we perform data mining within this context. The "state of the art" approaches may involve simply throwing neural networks on it, however it may be challenging when additional engineering constraints are put on top. The good news is that this can be solved through purely engineering approaches; rather than fancy algorithms or theorems. 

In this post we'll look briefly at using convolution operations to deal with feature generation over transaction information. 

Setting up the Data
===================

To setup the dataframe, one of the ways to do this is similar to the approach taken by [`sktime`](https://alan-turing-institute.github.io/sktime/examples/loading_data.html)

```
DataFrame:
index |   dim_0   |   dim_1   |    ...    |  dim_c-1
   0  | pd.Series | pd.Series | pd.Series | pd.Series
   1  | pd.Series | pd.Series | pd.Series | pd.Series
  ... |    ...    |    ...    |    ...    |    ...
   n  | pd.Series | pd.Series | pd.Series | pd.Series
```
Then to consider how we would model the data; we would have each observation followed by a sparse representation of transactional data. From here it is "easy" to calculate how we convole and pool features:

```py
import skimage.measure
import scipy.ndimage
import numpy as np
import pandas as pd

rand_seq = np.random.choice([0., 1.], 26*4).reshape((4, 26))
convolve_res = scipy.ndimage.convolve1d(rand_seq, np.array([1., 1., 1.]), axis=1) # box filter
max_pool = skimage.measure.block_reduce(convolve_res, (1, 7,), np.max)
```

This will naturally convolve the output into the correct form provided the data is in the right format.

Modelling the Data and Next Steps
=================================

One item which is still unsolved in my mind is how do we model this data? In a database, transactional information is generally in EAV related format - where by the data is long rather than wide. However it will need to be changed to a wide format. If we collapse items into a single observation then it would work "successfully" from the perspective of the `pd.DataFrame`. 

To demonstrate an example of how this may operate in the real-world, we will make use of the customer transaction information which is automatically generated within `featuretools`. 

```py
import skimage.measure
import scipy.ndimage
import numpy as np
import pandas as pd

import featuretools as ft
from sklearn.preprocessing import QuantileTransformer

data = ft.demo.load_mock_customer()

# we can explode it naively - but we don't really want to do this...
# data['customers'].shape # (5, 4)
# data['customers'].merge(data['sessions']).shape # (35, 7)

# what we do instead is collapse `customer_id`, and have everything as a sparse
# series, so when we join it stays having 5 rows. 
sessions = data['sessions'].copy()
sessions['session_start'] = QuantileTransformer(n_quantiles=100).fit_transform(
      np.array(sessions['session_start'].astype(int).tolist()).reshape(-1, 1)
   )
sessions['session_start'] = np.round(sessions['session_start'] * 100).astype(int)
sessions_gb = sessions.groupby(['customer_id', 'device'])
# we want to group by device and session_start; that is 
# we want columns to be "desktop/mobile", with pd.Series("session_start")
# get the min and max of the series

def custom_agg(x):
   # takes into a list
   x_z = np.zeros(101)
   x_z[np.array(x['session_start'].tolist())] = 1
   return x_z

sessions_denormalise = sessions_gb.apply(custom_agg).unstack('device').reset_index()
customer_sessions = data['customers'].merge(sessions_denormalise) # 5 rows, 7 columns

# if we apply box filter and then apply average pooling...
def conv_avgpool(x):
   x = np.stack(x.tolist(), 0)
   convolve_res = scipy.ndimage.convolve1d(x, np.ones(30), axis=1) # box filter
   avg_pool = skimage.measure.block_reduce(convolve_res, (1, 101,), np.mean)
   return avg_pool.flatten()

customer_sessions['mobile'] = conv_avgpool(customer_sessions['mobile'])
customer_sessions['desktop'] = conv_avgpool(customer_sessions['desktop'])
customer_sessions['tablet'] = conv_avgpool(customer_sessions['tablet'])

#    customer_id zip_code           join_date date_of_birth   desktop    mobile    tablet
# 0            1    60091 2011-04-17 10:48:33    1994-07-18  0.594059  0.881188  0.891089
# 1            2    13244 2012-04-15 23:31:04    1986-08-18  0.881188  0.613861  0.594059
# 2            3    13244 2011-08-13 15:42:34    2003-11-21  1.198020  0.306931  0.297030
# 3            4    60091 2011-04-08 20:08:14    2006-08-15  0.891089  1.168317  0.297030
# 4            5    60091 2010-07-17 05:27:50    1984-07-28  0.594059  0.891089  0.297030
```

This demonstrates how we can use convolution and extend how we may denormalise data structures into convolution frameworks such as tensorflow for our workflows. This is obviously more work compared with the single liner as part of `featuretools`; however, this concept may allow us to think about how we scale and provide real-time approaches moving forward.


