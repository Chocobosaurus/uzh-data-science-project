# Navigating Zurich: A Comprehensive Analysis of Urban Traffic Dynamics
Project work in the course "Introduction to Data Science" at University of Zurich.
We investigate the public transportation system in the city of Zurich using data analytic techniques.

You can find more information about the project on our website [sagerpascal.github.io/uzh-data-science-project](https://sagerpascal.github.io/uzh-data-science-project/).

Project members (sorted alphabetically):

- Pascal Sager
- Luca Zhao
- Weijia Zhong
- Xiaohan Zhu

## Webpage

We provide an overview of our results on the webpage  [sagerpascal.github.io/uzh-data-science-project](https://sagerpascal.github.io/uzh-data-science-project/).
We use github actions to build and deploy the content in the [/docs](./docs) folder directly to this webpage.

## Data

The data can be downloaded from the data collection of Zurich at [data.stadt-zuerich.ch](https://data.stadt-zuerich.ch/dataset/vbz_fahrgastzahlen_ogd).
We provide snapshots of the data in the `data/` folder that allow reproducing the results of this project.

## Q1: Development of Zurich’s Public Transportation System Over Time

We analyze the development of Zurich's Public Transportation system across year 2014-2023, by characterizing the spatial coverage and 
passenger volume in Zurich's public transportation system. <br>
You can use the notebook [src/Q1Q2.md](./src/Q1Q2.md) to reproduce the results. <br>
The distance coverages was computed with first filtering out the dataset for unique stop nodes and then summed up the distance between stops. <br>
The passenger volumn was computed with mutiplying the mean of numbers of passengers aboarding the vehicle with the corresponding number of days. This is because the data was sampled in such a way that the days of a year was defined into different day types, and values in the input contains averaged numbers of passengers in the day type, and how many days are there in the day type, instead of massive individual observation values. 

## Q2: Utilization Intensity of Zurich’s Public Transportation Infrastructure

We analyze the mobility of passengers with the VBZ system, in the metric of passenger-kilometer (pkm, number of passengers mutiplied by the distance moved), with the intention to compare to other major cities in the world.<br>
Pkm was computed with mutiplying the mean of numbers of passengers onboard with the corresponding number of days and the distance traveled.

You can also use the notebook [src/Q1Q2.md](./src/Q1Q2.md) to reproduce the results.


## Q3: Spatiotemporal Distribution of Passengers

We analyze the spatiotemporal distribution of passengers in Zurich's public transportation system.
You can use the notebook [src/Q3_VBZ_Geospatial_Analysis.ipynb](./src/Q3_VBZ_Geospatial_Analysis.ipynb) to reproduce the results.

This script creates **maps visualizing the traffic**. First, we visualize the connections (between stops) as lines on top of a map of Zurich.
The lines are colored according to the vehicle utilization (free seats).
Second, we visualize the number of passengers at each stop as heatmaps on top of a map of Zurich.
We calculate these statistics for each hour of the day and distingiush between weekdays and weekends. For each case, we calculate one corresponding map.
The maps are available as html in the [docs/static/maps](./docs/static/maps) folder and on our website [sagerpascal.github.io/uzh-data-science-project](https://sagerpascal.github.io/uzh-data-science-project/).


## Q4: Analysis of the Interplay Between Diverse Factors

We analyze the occupation of Zurich's public transportation regarding the factors weekday versus weekend, and academic calendar (exam session, lecture period).

Average occupation and boxplots for occupation regarding weekday or weekend, and academic calendar are given. The R code is available here [src/Q4_Analysis_of_diverse_factors_w_addtionalinfo.Rmd](./src/Q4_Analysis_of_diverse_factors_w_addtionalinfo.md).


## Q5: Prediction of Freeseats

We applied different machine leaning methods with the goal to predict seat availability of the VBZ system with different feature variables.

### Q5a: Prediction of Occupancy

We applied machine learning models trained with data in the earlier years in order to predict for a given later year. In order to do so, we first prepared the data to keep only relevant feature variables as Time/GPS_Latitude/GPS_Longitude/Weekday-weekend_attribute/Vehicle_type.<br>
The predictor variable is set to Occupancy which is the number of passengers onboard/total number of empty seats for less dependency on vehivle type.<br>
Additionally information about the maximum capacity of the vehicle, which is reserved for calculating the number of predicted empty seat ((1- Occupany) * Capacity) with minimal possible value = 0.<br>

The training was first attempted with using 2022 dataset as the training set and 2023 dataset as the test set. Linear regression models and ramdom forest could be successfully trained with grid searching good parameters, though the training of SVR became extremely time consuming. As so we then trimmed the data down to tram only and trained SVR without grid searching parameters, together with dimension reduction with PCA.<br>
There has been also attempts done with 2017 data as the training set and 2019 data as the test set, with and without normalizing on the predictor.<br>

Mean absolute percentage error (MAPE), mean absolute error (MAE) and mean square erro (MSE) together with MAE of the median predictor were computed on the prediction on the test set for evaluation.<br>

You can use the notebook<br>
[src/Q5_datacleaning.ipynb](./src/Q5_datacleaning.ipynb) <br>
[src/Q5_norm_train.ipynb](./src/Q5_norm_train.ipynb) <br>
[src/Q5_nonorm_train.ipynb](./src/Q5_nonorm_train.ipynb) <br>
to reproduce the results.

### Q5b: Predicting Passenger numbers

##### Data Preprocessing

Data were loaded and processed as the followings:
1. Load the train (2022) and test (2023) datasets, fixing the incorrect character set in the training data.
2. Remove the missing data which are around 5%, the reason behind the missing data is simply because the measurement didnt take place.
3. Convert the departure time values into integer values with bins of range 0.5 hour.
4. Transform the days of evaluations to boolean values, to preserve the information on which day of the of weekday the data was collected.
5. Perform one-hot encoding on categorical column LineNames
6. Add geological coordinates to the depart and destination stops so that stops added in 2023 can also be used to test the model. 

A quick test of regression model was then performed using the linear regression model, selecting only one type of the transportation mean
to make the computation faster, but it turned out that the regression models are not complex enough to predict.

https://github.com/sagerpascal/uzh-data-science-project/blob/main/src/Q5b_deeplearning_data_cleaning.ipynb

##### Deep Learning Methods

Based on the pre-processed version of the dataset, we also evaluate the performance of a deep learning models.
We use a simple feedforward neural network that consists of the following blocks:

```
ModelBlock(
      (fc): Linear(in_features, out_features, bias=True)
      (bn): BatchNorm1d(out_features, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
      (dropout): Dropout(p=0.2, inplace=False)
    )
```

We use 4 of these blocks with the following dimensions:

1. `in_features=113`, `out_features=512`
2. `in_features=512`, `out_features=512`
3. `in_features=512`, `out_features=756`
4. `in_features=756`, `out_features=756`

Afterwards, we use a fully connected layer to map these `756` to a single output.
We use the `ReLU` activation function for all layers, train the model for 100 epochs and use the `Adam` optimizer with a learning rate of `0.01`.
Additionally, we use a Learning Rate Scheduler with a that reduces the learning rate by a factor of `0.1` when the evaluation loss
does not reduce for 3 epochs.

We calculate the mean absolute error and mean square error on de-normalized values to evaluate the performance of the model.
We also compare the model performance when excluding the GPS coordinates which we added from an external source.
The notebook is available at [src/Q5b_Occupacy_Prediction_Deep_Learning](./src/Q5b_Occupacy_Prediction_Deep_Learning.ipynb) and plots are in [plots/Q5b](./plots/Q5b).
