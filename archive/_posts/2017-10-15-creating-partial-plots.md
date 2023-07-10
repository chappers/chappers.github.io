---
layout: post
category:
tags:
tagline:
---

Partial dependency plots are an important part of post-hoc modelling, particular when we are dealing with complex ensemble based models.

The idea behind these plots is to simply show the effect of modifying a single variable assuming that all other variables are the same. However there are several ways how this could be achieved, both with different assumptions.

**Using Base Data**

One approach for calculating partial dependency is to simply calculate at $n$ points spread evenly for the variable of interest.

Below is an example based on the relevant `h2o` code which has been replaced with the numpy equivalent.

```py
partial_df = []
for col in cols:
	res_ls = []
	# determine bins for partial plot
	break_df = data[col].hist(breaks=20, plot=False).as_data_frame()
	break_df['breaks_shift'] = break_df['breaks'].shift()
	breaks_partial = list(break_df[['breaks_shift', 'breaks']].fillna(data[col].min()).to_records(index=False))
	breaks_partial[-1][1] = data[col].max()

	for idx, breaks in enumerate(breaks_partial):
		lower = breaks[0]
		upper = breaks[1]
		lower_mask = data[col] >= lower
		upper_mask = data[col] <= upper
		res = model.predict(data[lower_mask and upper_mask])[:, -1]
		actual = data[lower_mask and upper_mask, "response"].mean()[0]
		mean_res = res.mean()[0]
		std_res = res.sd()[0]
		res_ls.append(((lower+upper)/2, mean_res, std_res, actual))
	# create dataframe so that we can plot
	res_df = pd.DataFrame(res_ls, columns=['idx', 'mean', 'std', 'actual'])
	res_df['std'] = np.maximum(model.predict(data)[:, -1].sd()[0], res_df['std'])
	res_df['lower'] = res_df['mean'] - res_df['std']
	res_df['upper'] = res_df['mean'] + res_df['std']
	partial_df.append(res_df.copy())
	if plot:
		plt.figure(figsize=(7,10))
		plt.plot(res_df['idx'], res_df['lower'], 'b--',
				 res_df['idx'], res_df['upper'], 'b--',
				 res_df['idx'], res_df['mean'], 'r-',
				 res_df['idx'], res_df['actual'], 'go', )
		plt.grid()
		plt.show()

```

**Using Grid**

Rather than using percentile, one could take the percentiles to take the `min` and `max` values of the grid which will then be used to create equally spaced points to test.

```py
n_samples = X.shape[0]
pdp = []
for row in range(grid.shape[0]):
	X_eval = X.copy()
	for i, variable in enumerate(target_variables):
		X_eval[:, variable] = np.repeat(grid[row, i], n_samples)
	pdp_row = _predict(est, X_eval, output=output)
	if est._estimator_type == 'regressor':
		pdp.append(np.mean(pdp_row))
	else:
		pdp.append(np.mean(pdp_row, 0))
pdp = np.array(pdp).transpose()
if pdp.shape[0] == 2:
	# Binary classification
	pdp = pdp[1, :][np.newaxis]
elif len(pdp.shape) == 1:
	# Regression
	pdp = pdp[np.newaxis]
```

**Concluding Thoughts**

There isn't necessarily a wrong or right way of approaching it, and probably several approaches to get a sense for the partial relationships in any model. What is probably more important is that there is a sensible way to approach this problem in a reusable and repeatible manner.
