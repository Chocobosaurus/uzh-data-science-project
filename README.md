# Navigating Zurich: A Comprehensive Analysis of Urban Traffic Dynamics
Project work in the course "Introduction to Data Science" at University of Zurich.
We investigate the public transportation system in the city of Zurich using data analytic techniques.

You can find more information about the project on our website [sagerpascal.github.io/uzh-data-science-project](https://sagerpascal.github.io/uzh-data-science-project/).

Project members (sorted alphabetically):

- Pascal Sager
- Luca Zhao
- Weijia Zhong
- Xiaohan Zhu

## Data

The data can be downloaded from the data collection of Zurich at [data.stadt-zuerich.ch](https://data.stadt-zuerich.ch/dataset/vbz_fahrgastzahlen_ogd).
We provide snapshots of the data in the `data/` folder that allow reproducing the results of this project.

## Q1: Development of Zurich’s Public Transportation System Over Time

We analyze the development of Zurich's Public Transportation system across year 2014-2023, by characterizing the spatial coverage and 
passenger volume in Zurich's public transportation system.
You can use the notebook [src/Q3_VBZ_Geospatial_Analysis.ipynb](./src/Q3_VBZ_Geospatial_Analysis.ipynb) to reproduce the results.

TODO Weijia: These links seem wrong. Please correct them.

## Q2: Utilization Intensity of Zurich’s Public Transportation Infrastructure

TODO Weijia

## Q3: Spatiotemporal Distribution of Passengers

We analyze the spatiotemporal distribution of passengers in Zurich's public transportation system.
You can use the notebook [src/Q3_VBZ_Geospatial_Analysis.ipynb](./src/Q3_VBZ_Geospatial_Analysis.ipynb) to reproduce the results.

This script creates **maps visualizing the traffic**. First, we visualize the connections (between stops) as lines on top of a map of Zurich.
The lines are colored according to the vehicle utilization (free seats).
Second, we visualize the number of passengers at each stop as heatmaps on top of a map of Zurich.
We calculate these statistics for each hour of the day and distingiush between weekdays and weekends. For each case, we calculate one corresponding map.
The maps are available as html in the [docs/static/maps](./docs/static/maps) folder and on our website [sagerpascal.github.io/uzh-data-science-project](https://sagerpascal.github.io/uzh-data-science-project/).


## Q4: Analysis of the Interplay Between Diverse Factors

We analyze the occupation of Zurich's public transportation regarding the factors weekday versus weekend, and academic calendar (exam session, semester break, lecture period).

Average occupation and boxplots for occupation regarding weekday or weekend, and academic calendar are given. The R code is available here.


## Q5: Prediction of Seat Availability

TODO Weijia

### Q5a: Predicting Free Seats

TODO Weijia

### Q5b: Predicting Occupancy

TODO: Luca: Describe the task

##### Data Preprocessing

Luca Zhao -> Describe how you pre-process the data, e.g. how you handle missing values, how you encode categorical variables, etc. + link the jupyter notebook here

##### Classical Machine Learning Methods

Luca Zhao -> Describe your linear regression model, random forest, SVC, etc. + link the jupyter notebook here


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
