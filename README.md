# ACLED Fatality Modelling

This repository contains the Python code developed during my internship (see my GitHub repository `Oxford_ACLED_Internship` for further details) in case it would be useful to anyone later. This README explains how to use functions contained in `general_model.py`. The Jupyter notebook shows how the functions can be used in practice. Further use of the functions can be found in the repository for my internship but is not as explicitly presented. The file `mcmc_functions.py` contains the implementations of the Metropolis Hastings algorithm used. The data files used can be shared upon request but are too large to be included in this repository.

## Using general_model.py

In this section I explain how to use the modelling provided in this Python file.

The function `general_model` can use either a Metropolis-Hastings implementation or the NUTS sampler provided by Stan for each of the models. I would recommend Stan in all cases, and the Stan files required are also provided here - see the associated report, provided in a PDF in this repository, for further details.

### Function arguments

The arguments taken by the `general_model` function are as follows:
- `model_type`: this specifies the chosen model, and how to use it is explained further below. This can be 0-2, 4-7, or 'null'.
- `data`: a data frame which has an entry for each day, event type and subnational region containing information on the number of events and the number of fatalities for each day/event/region combination. It should also have columns containing temporal information for each entry, such as the year, month and week.
- `data_filter`: this is a dictionary containing information on how to filter the dataset.

These three arguments have no defaults and must be provided. If no filtering is desired on the data, `None` can be supplied for this parameter. 

Further arguments are keyword arguments, with default `None` for each argument unless otherwise indicated below:
- `sample_size`: the number of posterior samples for each parameter to be returned, with default 10000.
- `temp_agg`: the temporal aggregation to apply to the data before modelling. The chosen temporal level must be a column of `data`, with default being 'WEEK' for weekly aggregation.
- `parameter_info`: a dictionary of parameter information, which is required if MH MCMC sampling is used. This should have keys including `names`, `hyperparameters` and `stepsizes`. 
- `stan_filepath`: a path to the Stan file to be used. This should be of the form `stan_files/model{i}.stan`, where `i` is 1, 2, 4, 5 or 6 and corresponds to the model number. This must be provided despite it being model dependent - further work could remove this requirement. If a stan filepath is provided, Stan is used, so to use Metropolis Hastings sampling this argument should not be called.
- `max_bin`: the maximum binning number for models 3 and 4 - any temporal periods observing at least this many events will be binned together into one category. Setting this parameter as 'auto' will use a crude automatic binning method based on `bin_size`.
- `covariates_dict`: a dictionary of selected covariates for model 5. This model can only be used if the data are about Bangladesh.
- `ignore_zero`: a boolean option to ignore temporal periods with no recorded events in the modelling. The default is `False`.
- `simulated_data`: a dictionary with keys including `responses` and `covariates` if simulated data is to be used rather than a data frame.
- `stan_seed`: a seed to be used for either Stan sampling or Metropolis Hastings, despite the name of the argument. The default is 1.
- `null_hyperparameters`: the hyperparameters if sampling the null model. This should be a list with two elements suitable as the parameters of a Beta distribution. The default is `[1,1]` to provide a default Uniform prior.
- `bin_size`: a proportion used if automatic binning is selected. This procedure groups together the top (1-`bin_size`)100% of observations as the highest category.

The output of this function depends on the sampling method:
- Sampling using Stan returns a tuple of a stan.fit.Fit object and a pandas data frame. This allows sampling diagnostics to occur on the stan.fit.Fit object.
- Sampling using Metropolis Hastings returns a pandas data frame only.

The output is indicated as a printed message from the function. When Metropolis Hastings sampling is used, the function also provides the proposal acceptance rates for each parameter. When Stan is used, the output is the usual output from using `pystan`. If a model uses binning, the information on the names of returned parameters is also given and the number of bins used.

### Model numbers

The names of the models given here correspond to the report, so for a more detailed description of each model refer to that document.

- 0: The Null Model. This is to test the null hypothesis of independence of events.
- 1: Basic Logistic Regression. This is a logistic regression on the number of events in each temporal period.
- 2: Further Logistic Regression. This is a logistic regression on the number of events in each period, and the fatality indication of the previous period.
- 4: Discretisation and Ordering. This model is explained in the report.
- 5: Expanded Logistic Regression. Only available for Bangladesh, this incorporates a wide range of variables into the logistic regression model.
- 6: Cumulative Probability. This works similar to the null model, but does not assume independence of events.
- 7: The Null Model.

## Sampling diagnostics

Sampling diagnostics are provided by several functions in the document. The function `sampling_visualisation` plots trace plots, autocorrelation and pairplots from posterior samples. The function `print_ess` prints the effective sample size of posterior parameter samples. For more information on how these functions work, refer to the information provided in the function document.

### Data simulation

Using simulated data is an important way of testing sampling algorithms. Whilst not explored in the report, the ability to simulate data is provided within this file with the `simulate_data`function. The arguments for this function which are as follows:

- `model_type`: model specifier as for `general_model`.
- `temp_agg`: as `general_model`.
- `data`: data frame for simulation, if observed means or counts are to be used.
- `data_filter`: as `general_model`.
- `parameters_mean`: the mean values of the parameters to be used in the data simulation.
- `inner_event_mean`: the mean number of event counts in the spatial area of choice. If model number 5 is used, this is the mean event count for the inner district. Either this value or a data frame and filtering dictionary must be provided.
- `outer_event_mean`: as for model 5.
- `num_obs`: the number of observations to simulate, default 1000.
- `random_events`: boolean toggle for simulating events or using recorded event counts, as found if a data frame is provided.
- `ignore_zero`: as for `general_model`.
- `covariates_dict`: as for `general_model`.
- `max_bin`: as for `general_model`, but automatic binning is not available for simulated data.

## Posterior visulation

Some functions are provided for visualising posterior distributions when parameters have been transformed. A key function is `posterior_probability_plot` which produces box and violin plots for models 1 and 4. Work still needs to be done on expanding the capabilities of this. Quantile tables are available through `create_quantile_table` which produces a table of quantiles for each parameter. Further information about this function can be found in the document.
