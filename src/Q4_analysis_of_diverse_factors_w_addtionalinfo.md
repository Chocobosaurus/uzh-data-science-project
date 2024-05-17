NavigatingZH_Data_Science_Intro_UZH_Q4
================
Xiaohan Zhu (auxilary info added by Weijia)
2024-05-16

### This script documents the codes use to solve research question:

- Q4: Analysis of the Interplay Between Diverse Factors

## Part 1: <br>

The whole dataset was grouped by factor “Weekday”/“Weekend”, or
“Semester/Exam period” and the mean across all time points were averaged
and visualized as bargraph

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(readr)
library(ggplot2)

reisende <- read.csv("../data/fahrgastzahlen_2023/reisende.csv", sep = ";")

reisende_data <- reisende %>% 
  mutate(day_type = ifelse(Tage_DWV > 0, "Weekday", "Weekend"),
         academic_calendar = case_when(
           Tagtyp_Id %in% c(10, 11, 12) ~ "Exam Session",
#           Tagtyp_Id %in% c(8, 9) ~ "Semester Break",
           TRUE ~ "Lecture Period"
         ),
         passenger_count = Besetzung, data = reisende)

# Factor analysis for weekday versus weekend patterns
weekday_vs_weekend <- reisende_data %>%
    group_by(day_type) %>%
    mutate(avg_passengers = Besetzung) %>%
    distinct() %>%
    summarise(avg_passengers = mean(avg_passengers, na.rm = TRUE))

# Factor analysis for academic calendar
semester_vs_break <- reisende_data %>%
    group_by(academic_calendar) %>%
    mutate(avg_passengers = Besetzung) %>%
    distinct() %>%
    summarise(avg_passengers = mean(avg_passengers, na.rm = TRUE))

# Plotting the difference between weekday and weekend passenger counts
barplot(weekday_vs_weekend$avg_passengers, names.arg = weekday_vs_weekend$day_type,
        main = "Average Occupation: Weekday vs. Weekend",
        xlab = "Day Type", ylab = "Average Occupation")
```

![](Q4_Analysis_of_diverse_factors_w_addtionalinfo_files/figure-gfm/Loading%20data,%20grouping%20and%20visualize%20gross%20average-1.png)<!-- -->

``` r
# Plotting the difference between academic and non-academic periods
barplot(semester_vs_break$avg_passengers, names.arg = semester_vs_break$academic_calendar,
        main = "Average Occupation: Academic Calendar",
        xlab = "Academic Calendar", ylab = "Average Occupation")
```

![](Q4_Analysis_of_diverse_factors_w_addtionalinfo_files/figure-gfm/Loading%20data,%20grouping%20and%20visualize%20gross%20average-2.png)<!-- -->

``` r
# Filter out rows with zero or negative passenger counts
reisende_data <- reisende_data %>%
  filter(passenger_count > 0)

# Create boxplot for weekday vs. weekend passenger counts
weekday_vs_weekend_boxplot <- ggplot(reisende_data, aes(x = day_type, y = passenger_count, fill = day_type)) +
  geom_boxplot(outlier.shape = NA) + # don't show outliers for intuitive visualization
  ylim(0, 75) +
  labs(title = "Distribution of Occupation: Weekday vs. Weekend",
       x = "Day Type", y = "Occupation") +
  theme_minimal()
#  scale_y_log10(labels = function(x) {
#    ifelse(x < 1, format(x, scientific = FALSE), format(x, scientific = FALSE, digits = 2))
#  })  # Log scale disabled for intuitive comparison

# Create boxplot for academic calendar
semester_vs_break_boxplot <- ggplot(reisende_data, aes(x = academic_calendar, y = passenger_count, fill = academic_calendar)) +
  geom_boxplot(outlier.shape = NA) +
  ylim(0, 75) +
  labs(title = "Distribution of Occupation: Academic Calendar",
       x = "Academic Calendar", y = "Occupation") +
  theme_minimal()
#  scale_y_log10(labels = function(x) {
#    ifelse(x < 1, format(x, scientific = FALSE), format(x, scientific = FALSE, digits = 2))
#  })  # Log scale disabled for intuitive comparison


# Display the plots
print(weekday_vs_weekend_boxplot)
```

    ## Warning: Removed 21669 rows containing non-finite outside the scale range
    ## (`stat_boxplot()`).

![](Q4_Analysis_of_diverse_factors_w_addtionalinfo_files/figure-gfm/Visualization%20with%20barplot-1.png)<!-- -->

``` r
print(semester_vs_break_boxplot)
```

    ## Warning: Removed 21669 rows containing non-finite outside the scale range
    ## (`stat_boxplot()`).

![](Q4_Analysis_of_diverse_factors_w_addtionalinfo_files/figure-gfm/Visualization%20with%20barplot-2.png)<!-- -->

### Part 2: <br>

Now perform the same factor analysis with a subset of the data contains
only info of stops near universities, visualize with barplot.

``` r
# Now the following part is to analyse the passenger occupation near the universities during exam session, lecture period, and semester break. Stops "Zürich, ETH/Universitätsspital", "Zürich, ETH Hönggerberg", "Zürich, Milchbuck", Zürich, Universität Irchel" are chosen.

reisende <- read.csv("../data/fahrgastzahlen_2023/reisende.csv", sep = ";")

# Filter stops near universities
university_stops <- c(60, 377, 53, 161)
reisende_filtered <- reisende %>% 
  filter(Haltestellen_Id %in% university_stops)

# Rerun the analysis
reisende_data_filtered <- reisende_filtered %>% 
  mutate(day_type = ifelse(Tage_DWV > 0, "Weekday", "Weekend"),
         academic_calendar = case_when(
           Tagtyp_Id %in% c(10, 11, 12) ~ "Exam Session",
          # Tagtyp_Id %in% c(8, 9) ~ "Semester Break",
           TRUE ~ "Lecture Period"
         ),
         passenger_count = Besetzung)

# Factor analysis for academic calendar
semester_vs_break_filtered <- reisende_data_filtered %>%
    group_by(academic_calendar) %>%
    mutate(avg_passengers = Besetzung) %>%
    distinct() %>%
    summarise(avg_passengers = mean(avg_passengers, na.rm = TRUE))

# Plotting the difference between academic and non-academic periods
barplot(semester_vs_break_filtered$avg_passengers, names.arg = semester_vs_break_filtered$academic_calendar,
        main = "Average Occupation: Academic Calendar",
        xlab = "Academic Calendar", ylab = "Average Occupation")
```

![](Q4_Analysis_of_diverse_factors_w_addtionalinfo_files/figure-gfm/Same%20factor%20analysis%20with%20stops%20near%20universities%20only-1.png)<!-- -->

``` r
# Filter out rows with zero or negative passenger counts
reisende_data_filtered <- reisende_data_filtered %>%
  filter(passenger_count > 0)

# Create boxplot for academic calendar
semester_vs_break_boxplot_filtered <- ggplot(reisende_data_filtered, aes(x = academic_calendar, y = passenger_count, fill = academic_calendar)) +
  geom_boxplot() +
  labs(title = "Distribution of Occupation: Academic Calendar",
       x = "Academic Calendar", y = "Occupation") +
  theme_minimal()
#  scale_y_log10(labels = function(x) {
#    ifelse(x < 1, format(x, scientific = FALSE), format(x, scientific = FALSE, digits = 2))
#  })  # Use log scale for y-axis and customize label formatting

# Display the plots
print(semester_vs_break_boxplot_filtered)
```

![](Q4_Analysis_of_diverse_factors_w_addtionalinfo_files/figure-gfm/Same%20factor%20analysis%20with%20stops%20near%20universities%20only-2.png)<!-- -->

``` r
reisende_data$dataset <- factor("All Routes", levels = c("All Routes", "University Stops"))
reisende_data_filtered$dataset <- factor("University Stops", levels = c("All Routes", "University Stops"))

# Combine the datasets for plotting
combined_boxplot <- ggplot() +
  geom_boxplot(data = reisende_data, aes(x = academic_calendar, y = passenger_count, fill = academic_calendar), outlier.shape = NA) +
  geom_boxplot(data = reisende_data_filtered, aes(x = academic_calendar, y = passenger_count, fill = academic_calendar), outlier.shape = NA) +
  facet_wrap(~ dataset, scales = "free_y") +  # Facet by dataset (all routes vs. university stops)
  labs(title = "Distribution of Occupation: Academic Calendar",
       x = "Academic Calendar", y = "Occupation") +
  theme_minimal() +
  ylim(0, 75) +
#  scale_y_log10(labels = function(x) {
#    ifelse(x < 1, format(x, scientific = FALSE), format(x, scientific = FALSE, digits = 2))
#  }) + # Use log scale for y-axis and customize label formatting
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) 

# Display the plot
print(combined_boxplot)
```

    ## Warning: Removed 21669 rows containing non-finite outside the scale range
    ## (`stat_boxplot()`).

    ## Warning: Removed 339 rows containing non-finite outside the scale range
    ## (`stat_boxplot()`).

![](Q4_Analysis_of_diverse_factors_w_addtionalinfo_files/figure-gfm/Same%20factor%20analysis%20with%20stops%20near%20universities%20only-3.png)<!-- -->

``` r
# Filter stops near universities
university_stops <- c(60, 377, 53, 161)
ETH_Hongg <- 377
reisende_filtered <- reisende %>% 
  filter(Haltestellen_Id %in% university_stops)
reisende_filtered2 <- reisende %>% 
  filter(Haltestellen_Id %in% ETH_Hongg)


# Rerun the analysis
reisende_data_filtered <- reisende_filtered %>% 
  mutate(day_type = ifelse(Tage_DWV > 0, "Weekday", "Weekend"),
         academic_calendar = case_when(
           Tagtyp_Id %in% c(10, 11, 12) ~ "Exam Session",
          # Tagtyp_Id %in% c(8, 9) ~ "Semester Break",
           TRUE ~ "Lecture Period"
         ),
         passenger_count = Besetzung)

reisende_data_filtered2 <- reisende_filtered2 %>% 
  mutate(day_type = ifelse(Tage_DWV > 0, "Weekday", "Weekend"),
         academic_calendar = case_when(
           Tagtyp_Id %in% c(10, 11, 12) ~ "Exam Session",
          # Tagtyp_Id %in% c(8, 9) ~ "Semester Break",
           TRUE ~ "Lecture Period"
         ),
         passenger_count = Besetzung)

# Factor analysis for academic calendar
semester_vs_break_filtered <- reisende_data_filtered %>%
    group_by(academic_calendar) %>%
    mutate(avg_passengers = Besetzung) %>%
    distinct() %>%
    summarise(avg_passengers = mean(avg_passengers, na.rm = TRUE))

semester_vs_break_filtered <- reisende_data_filtered2 %>%
    group_by(academic_calendar) %>%
    mutate(avg_passengers = Besetzung) %>%
    distinct() %>%
    summarise(avg_passengers = mean(avg_passengers, na.rm = TRUE))

# Plotting the difference between academic and non-academic periods
barplot(semester_vs_break_filtered$avg_passengers, names.arg = semester_vs_break_filtered$academic_calendar,
        main = "Average Occupation: Academic Calendar",
        xlab = "Academic Calendar", ylab = "Average Occupation")
```

![](Q4_Analysis_of_diverse_factors_w_addtionalinfo_files/figure-gfm/same%20analysis,%20restricing%20university%20stops%20to%20ETH/Hongg%20only%20for%20minimal%20confounder%20effects-1.png)<!-- -->

``` r
# Filter out rows with zero or negative passenger counts
reisende_data_filtered <- reisende_data_filtered %>%
  filter(passenger_count > 0)

# Create boxplot for academic calendar
semester_vs_break_boxplot_filtered <- ggplot(reisende_data_filtered, aes(x = academic_calendar, y = passenger_count, fill = academic_calendar)) +
  geom_boxplot() +
  labs(title = "Distribution of Occupation: Academic Calendar",
       x = "Academic Calendar", y = "Occupation") +
  theme_minimal()
#  scale_y_log10(labels = function(x) {
#    ifelse(x < 1, format(x, scientific = FALSE), format(x, scientific = FALSE, digits = 2))
#  })  # Use log scale for y-axis and customize label formatting

# Display the plots
print(semester_vs_break_boxplot_filtered)
```

![](Q4_Analysis_of_diverse_factors_w_addtionalinfo_files/figure-gfm/same%20analysis,%20restricing%20university%20stops%20to%20ETH/Hongg%20only%20for%20minimal%20confounder%20effects-2.png)<!-- -->

``` r
reisende_data$dataset <- factor("All Routes", levels = c("All Routes", "University Stops"))
reisende_data_filtered$dataset <- factor("University Stops", levels = c("All Routes", "University Stops"))
reisende_data_filtered2$dataset <- factor("ETH_Hongg", levels = c("All Routes", "ETH_Hongg"))

# Combine the datasets for plotting
combined_boxplot <- ggplot() +
  geom_boxplot(data = reisende_data, aes(x = academic_calendar, y = passenger_count, fill = academic_calendar), outlier.shape = NA) +
  geom_boxplot(data = reisende_data_filtered, aes(x = academic_calendar, y = passenger_count, fill = academic_calendar), outlier.shape = NA) +
  geom_boxplot(data = reisende_data_filtered2, aes(x = academic_calendar, y = passenger_count, fill = academic_calendar), outlier.shape = NA) +
  facet_wrap(~ dataset, scales = "free_y") +  # Facet by dataset (all routes vs. university stops)
  labs(title = "Distribution of Occupation: Academic Calendar",
       x = "Academic Calendar", y = "Occupation") +
  theme_minimal() +
  ylim(0, 75) +
#  scale_y_log10(labels = function(x) {
#    ifelse(x < 1, format(x, scientific = FALSE), format(x, scientific = FALSE, digits = 2))
#  }) + # Use log scale for y-axis and customize label formatting
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) 

# Display the plot
print(combined_boxplot)
```

    ## Warning: Removed 21669 rows containing non-finite outside the scale range
    ## (`stat_boxplot()`).

    ## Warning: Removed 339 rows containing non-finite outside the scale range
    ## (`stat_boxplot()`).

    ## Warning: Removed 1291 rows containing non-finite outside the scale range
    ## (`stat_boxplot()`).

![](Q4_Analysis_of_diverse_factors_w_addtionalinfo_files/figure-gfm/same%20analysis,%20restricing%20university%20stops%20to%20ETH/Hongg%20only%20for%20minimal%20confounder%20effects-3.png)<!-- -->
