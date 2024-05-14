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

## Development of Zurich’s Public Transportation System Over Time

We analyze the development of Zurich's Public Transportation system across year 2014-2023, by characterizing the spatial coverage and 
passenger volume in Zurich's public transportation system.
You can use the notebook [src/VBZ_Geospatial_Analysis.ipynb](./src/VBZ_Geospatial_Analysis.ipynb) to reproduce the results.


## Utilization Intensity of Zurich’s Public Transportation Infrastructure

TODO Weijia

## Spatiotemporal Distribution of Passengers

We analyze the spatiotemporal distribution of passengers in Zurich's public transportation system.
You can use the notebook [src/VBZ_Geospatial_Analysis.ipynb](./src/VBZ_Geospatial_Analysis.ipynb) to reproduce the results.

This script creates **maps visualizing the traffic**. First, we visualize the connections (between stops) as lines on top of a map of Zurich.
The lines are colored according to the vehicle utilization (free seats).
Second, we visualize the number of passengers at each stop as heatmaps on top of a map of Zurich.
We calculate these statistics for each hour of the day and distingiush between weekdays and weekends. For each case, we calculate one corresponding map.
The maps are available as html in the [docs/static/maps](./docs/static/maps) folder and on our website [sagerpascal.github.io/uzh-data-science-project](https://sagerpascal.github.io/uzh-data-science-project/).


## Analysis of the Interplay Between Diverse Factors

TODO Luca/Xiaohan

# Prediction of Seat Availability

TODO Luca/Xiaohan
