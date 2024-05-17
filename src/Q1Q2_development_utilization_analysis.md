NavigatingZH_Data_Science_Intro_UZH_Q1Q2
================
Weijia Zhong
2024-05-08

### This script documents the codes use to solve research question:

Q1- Development of Zurich’s Public Transportation System Over Time <br>
Q2- Utilization Intensity of Zurich’s Public Transportation
Infrastructure

``` r
## This is just to figure out what is what in the datset
# Load the dependencies
library(readxl)
library(dplyr, warn.conflicts = FALSE)
library(stringr)
library(tibble)
library(ggplot2)
library(plotly, warn.conflicts = FALSE)
library(grid)
library(gridExtra, warn.conflicts = FALSE)
library(scales)
library(reshape2)
library(viridis, warn.conflicts = FALSE)
```

    ## Loading required package: viridisLite

``` r
directory_path <- "../data/fahrgastzahlen_2023/"

filename0 <- "REISENDE"
filename1 <- "LINIE"
filename2 <- "TAGTYP"
filename3 <- "HALTESTELLEN"
filename4 <- "GEFAESSGROESSE"
filename5 <- "haltepunkt"

# Load data from 2023
csv <- "csv"
df <- read.csv(file.path(directory_path, paste0(filename0, ".", csv)), sep=";", header = T)
linie <- read.csv(file.path(directory_path, paste0(filename1, ".", csv)), sep=";", header = T)
tagtyp <- read.csv(file.path(directory_path, paste0(filename2, ".", csv)), sep=";", header = T)
halte <- read.csv(file.path(directory_path, paste0(filename3, ".", csv)), sep=";", header = T)
gef <- read.csv(file.path(directory_path, paste0(filename4, ".", csv)), sep=";", header = T)
punkte <- read.csv(file.path(directory_path, paste0(filename5, ".", csv)), sep=",", header = T)

colnames(df)
colnames(linie)
colnames(tagtyp)
colnames(halte)
colnames(gef)
colnames(punkte)

# Examine a specific line 
linie42 <- subset(df, df$Linien_Id == 42)
sum(linie42$Tage_DTV == 0)
# no zeros in Tage_DTV

sum(linie42[!duplicated(linie42$Tage_DTV), ]$Tage_DTV)
# For one specific line, Tage_DTV sums up to 365 or smaller (why?)
nrow(linie42[!duplicated(linie42$Tagtyp_Id), ])
nrow(linie42[!duplicated(linie42$Tage_DTV), ])
# Binning in Tage_DTV is the same with Tagtyp_ID 
## -> A year of time is divided by type of days (sum to 365) then sampled for each day,
## then binned for each day type, showing the mean for this type of day

sum(!duplicated(gef$Plan_Fahrt_Id))
# no duplication in Plan_Fahrt_Id in gef
sum(!duplicated(df$Plan_Fahrt_Id))
# all Plan_Fahrt_Id represented in df 

sum(!duplicated(df$Haltestellen_Id))
sum(!duplicated(halte$Haltestellen_Id))
# not all represented in df - inactive stops?
inact <- anti_join(halte, df, by = "Haltestellen_Id")
# looks like a bunch of garage (?)

acti_punkte <- subset(punkte, punkte$halt_punkt_ist_aktiv == "True")
sum(!duplicated(acti_punkte$halt_id))
# roughly the same with the number of Haltstellen_Id
# probably fine to just filter data by unique Haltestellen_Id
```

Q1.1:Spatial coverage of VBZ system across the years Datasets of
previous years were downloaded from <br>
<https://data.stadt-zuerich.ch/dataset/vbz_fahrgastzahlen_ogd> <br> and
<br> <https://data.stadt-zuerich.ch/dataset/vbz_fahrzeiten_ogd>

The goal is to compute the distance/passenger volumn within the whole
VBZ system and compare across the years. <br> Futher, contributions from
different vehicle types (tram, bus, trolleybus, seilbahn, forchbahn,
nachtnetz) are visualized in stacked barplot.

``` r
# Method: load data sets of the past years. Sum on $Distanz after filtering for unique haltestellen_ID 
## This is wrong because omitted shared stops of overlapping routes and overlap of route map with Nachtnetz
## Filter by unique$Distanz instead, but some error might persis since it still doesn't solve the issue of overlap with Nachtnetz

wd <- ("../data")
subfolders <- list.dirs(wd, full.names = TRUE)
subfolders <- grep("fahrgastzahlen_20[0-9]{2}", subfolders, value = TRUE)

dfs <- list()
years <- sprintf("20%02d", 14:23)

# Loop through each subfolder for data of a year from 2014-2023, load "REISENDE"
for (subfolder in subfolders) {
  csv_files <- list.files(path = subfolder, pattern = "\\.csv$", full.names = TRUE)
  
  if (length(csv_files) > 0) {
    reisende_file <- grep("REISENDE", csv_files, value = TRUE)
    if (length(reisende_file) > 0) {
      dfs[[subfolder]] <- read.csv(reisende_file, sep=";", header = T)
    }
  }
}


# Filter for unique $Distanz values, then sum up $Distanz, then make barplot
distanz <- function(dfs) {
  filter_and_summarize <- function(data) {
    unique_data <- data[!duplicated(data$Distanz), ]
    total_distance <- if ("Distanz" %in% colnames(unique_data)) {
      unique_data %>%
        summarise(total_distance = sum(Distanz, na.rm = TRUE)) %>%
        pull(total_distance)
    } else {
      NA
    }
    return(total_distance)
  }
  
  summed_distances <- sapply(dfs, filter_and_summarize)
  summed_distances_df <- data.frame(Years = years, Total_Distance = unlist(summed_distances)/10^3)
  
  ggplot(summed_distances_df, aes(x = Years, y = Total_Distance)) +
    geom_bar(stat = "identity", fill = "#8c2981", color = "grey") +
    labs(title = "Total Distance Coverage by Years (Nachtnetz excluded)", x = "Years", y = "Total Distance (km)")
}


plot <- distanz(dfs)
# ggsave("results/Q1_1.png", width = 8, height = 6, dpi = 600)
print(plot)
```

![](../plots/Q1Q2/Q1.1:Spatial%20coverage%20of%20VBZ%20system%20across%20the%20years-1.png)<!-- -->

``` r
# einsteiger * Tage_DTV as passenger volumn

passvol <- function(dfs) {
  multiply <- function(df) {
    total_vol <- df %>%
      mutate(product = Einsteiger * Tage_DTV) %>%
      summarise(total_vol = sum(product, na.rm = TRUE)) %>%
      pull(total_vol)
    return(total_vol)
  }
  
  total_volumn <- sapply(dfs, multiply)
  total_volumn_df <- data.frame(Years = years, Total_Volume = unlist(total_volumn)/10^6)
  
  ggplot(total_volumn_df, aes(x = Years, y = Total_Volume)) +
    geom_bar(stat = "identity", fill = "#f56b5c", color = "grey") +
    labs(title = "Total Volume by Years", x = "Years", y = "Total Volume (mio)")
}

plot <- passvol(dfs)
# ggsave("results/Q1_2.png", width = 8, height = 6, dpi = 600)
print(plot)
```

![](../plots/Q1Q2/Q1.2:%20Passenger%20volume%20change%20over%20the%20years-1.png)<!-- -->

``` r
# Merge with LINIE.csv
merline <- function(data) {
  merline_data <- merge(data, linie, by = "Linien_Id")
  return(merline_data)
}

merline_dfs <- lapply(dfs, merline)

# Verkehrssystem: T: Tram; TR: Trolleybus; B: Bus; SB:Seilbahn; FB: Forchbahn; N: Nachtbus.
# linie[!duplicated(linie$VSYS), ]$VSYS

# split dataset by vehicle type
tram <- function(data) {
  tram_data <- data[data$VSYS == "T", ]
  return(tram_data)
}

trolleybus <- function(data) {
  tr_data <- data[data$VSYS == "TR", ]
  return(tr_data)
}

bus <- function(data) {
  bus_data <- data[data$VSYS == "B" | 
                   data$VSYS == "BL" | # Limmattal
                   data$VSYS == "BZ" | # Zimmerberg
                   data$VSYS == "BG" | # Glattal
                   data$VSYS == "BP", ] # Pfannenstiel
  return(bus_data)
}

seilbahn <- function(data) {
  sb_data <- data[data$VSYS == "SB", ]
  return(sb_data)
}

forchbahn <- function(data) {
  fb_data <- data[data$VSYS == "FB", ]
  return(fb_data)
}

nachtbus <- function(data) {
  n_data <- data[data$VSYS == "N", ]
  return(n_data)
}

tram_dfs <- lapply(merline_dfs, tram)
trolley_dfs <- lapply(merline_dfs, trolleybus)
bus_dfs <- lapply(merline_dfs, bus)
seilbahn_dfs <- lapply(merline_dfs, seilbahn)
forchbahn_dfs <- lapply(merline_dfs, forchbahn)
nacht_dfs <- lapply(merline_dfs, nachtbus)
```

``` r
distanz <- function(dfs) {
  filter_and_summarize <- function(data) {
    unique_data <- data[!duplicated(data$Distanz), ]
    total_distance <- if ("Distanz" %in% colnames(unique_data)) {
      unique_data %>%
        summarise(total_distance = sum(Distanz, na.rm = TRUE)) %>%
        pull(total_distance)
    } else {
      NA
    }
    return(total_distance)
  }
  
  summed_distances <- sapply(dfs, filter_and_summarize)
  summed_distances_df <- data.frame(Years = years, Total_Distance = unlist(summed_distances)/10^3)
  
  ggplot(summed_distances_df, aes(x = Years, y = Total_Distance)) +
    geom_bar(stat = "identity", fill = "#8c2981", color = "grey") +
    labs(title = "Total Distance Coverage by Years", x = "Years", y = "Total Distance (km)") +
         ylim(0, 450) #fix the ylab to be the same for comparison
}

passvol <- function(dfs) {
  multiply <- function(df) {
    total_vol <- df %>%
      mutate(product = Einsteiger * Tage_DTV) %>%
      summarise(total_vol = sum(product, na.rm = TRUE)) %>%
      pull(total_vol)
    return(total_vol)
  }
  
  total_volumn <- sapply(dfs, multiply)
  total_volumn_df <- data.frame(Years = years, Total_Volume = unlist(total_volumn)/10^6)
  
  ggplot(total_volumn_df, aes(x = Years, y = Total_Volume)) +
    geom_bar(stat = "identity", fill = "#f56b5c", color = "grey") +
    labs(title = "Total Volume by Years", x = "Years", y = "Total Volume (mio)") +
         ylim(0, 300) #fix the ylab to be the same for comparison
}

fig1 <- distanz(tram_dfs)
fig2 <- passvol(tram_dfs)
fig <- arrangeGrob(fig1, fig2, ncol=2, top = textGrob("Change in Tramways", gp = gpar(fontsize = 20)))
# ggsave(file = "results/Q1tram.png", fig, width = 8, height = 5, dpi = 600)
grid.arrange(fig1, fig2, ncol=2, top = textGrob("Change in Tramways", gp = gpar(fontsize = 20)))
```

![](../plots/Q1Q2/Q1+%20plots:%20Change%20in%20Tramways-1.png)<!-- -->

``` r
fig1 <- distanz(bus_dfs)
fig2 <- passvol(bus_dfs)
fig <- arrangeGrob(fig1, fig2, ncol=2, top = textGrob("Change in Buses", gp = gpar(fontsize = 20)))
# ggsave(file = "results/Q1bus.png", fig, width = 8, height = 5, dpi = 600)
grid.arrange(fig1, fig2, ncol=2, top = textGrob("Change in Buses", gp = gpar(fontsize = 20)))
```

![](../plots/Q1Q2/Q1+%20plots:%20Change%20in%20Bus-1.png)<!-- -->

``` r
fig1 <- distanz(trolley_dfs)
fig2 <- passvol(trolley_dfs)
fig <- arrangeGrob(fig1, fig2, ncol=2, top = textGrob("Change in Trolleybus", gp = gpar(fontsize = 20)))
# ggsave(file = "results/Q1trolleybus.png", fig, width = 8, height = 5, dpi = 600)
grid.arrange(fig1, fig2, ncol=2, top = textGrob("Change in Trolleybus", gp = gpar(fontsize = 20)))
```

![](../plots/Q1Q2/Q1+%20plots:%20Change%20in%20Trolleybus-1.png)<!-- -->

``` r
fig1 <- distanz(seilbahn_dfs)
fig2 <- passvol(seilbahn_dfs)
fig <- arrangeGrob(fig1, fig2, ncol=2, top = textGrob("Change in Cablecar (Seilbahn)", 
                                                      gp = gpar(fontsize = 20)))
# ggsave(file = "results/Q1seilbahn.png", fig, width = 8, height = 5, dpi = 600)
grid.arrange(fig1, fig2, ncol=2, top = textGrob("Change in Cablecar (Seilbahn)", gp = gpar(fontsize = 20)))
```

![](../plots/Q1Q2/Q1+%20plots:%20Change%20in%20Cablecar%20(Seilbahn)-1.png)<!-- -->

``` r
fig1 <- distanz(forchbahn_dfs)
fig2 <- passvol(forchbahn_dfs)
fig <- arrangeGrob(fig1, fig2, ncol=2, top = textGrob("Change in Forchbahn", gp = gpar(fontsize = 20)))
# ggsave(file = "results/Q1forchbahn.png", fig, width = 8, height = 5, dpi = 600)
grid.arrange(fig1, fig2, ncol=2, top = textGrob("Change in Forchbahn", gp = gpar(fontsize = 20)))
```

![](../plots/Q1Q2/Q1+%20plots:%20Change%20in%20Forchbahn-1.png)<!-- -->

``` r
fig1 <- distanz(nacht_dfs)
fig2 <- passvol(nacht_dfs)
fig <- arrangeGrob(fig1, fig2, ncol=2, top = textGrob("Change in Night Network", gp = gpar(fontsize = 20)))
# ggsave(file = "results/Q1nacht.png", fig, width = 8, height = 5, dpi = 600)
grid.arrange(fig1, fig2, ncol=2, top = textGrob("Change in Night Network", gp = gpar(fontsize = 20)))
```

![](../plots/Q1Q2/Q1+%20plots:%20Change%20in%20Night%20network-1.png)<!-- -->

``` r
distanz0 <- function(dfs) {
  filter_and_summarize <- function(data) {
    unique_data <- data[!duplicated(data$Distanz), ]
    total_distance <- if ("Distanz" %in% colnames(unique_data)) {
      unique_data %>%
        summarise(total_distance = sum(Distanz, na.rm = TRUE)) %>%
        pull(total_distance)
    } else {
      NA
    }
    return(total_distance)
  }
  
  summed_distances <- sapply(dfs, filter_and_summarize)
  summed_distances_df <- data.frame(Years = years, Total_Distance = unlist(summed_distances)/10^3)
}

tram_dis <- distanz0(tram_dfs)
trolley_dis <- distanz0(trolley_dfs)
bus_dis <- distanz0(bus_dfs)
seilbahn_dis <- distanz0(seilbahn_dfs)
forchbahn_dis <- distanz0(forchbahn_dfs)
nacht_dis <- distanz0(nacht_dfs)
all_dis <- cbind(nacht_dis, forchbahn_dis$Total_Distance, seilbahn_dis$Total_Distance, 
                  trolley_dis$Total_Distance, bus_dis$Total_Distance, tram_dis$Total_Distance)
coln <- c("Years", "Nachtnetz", "Forchbahn", "Seilbahn", "Trolleybus", "Bus", "Tram")
colnames(all_dis) <- coln
melt_dis <- melt(all_dis, id = "Years")

b1 <- ggplot(melt_dis, aes(x = Years, y = value, fill = variable)) + 
  geom_bar(stat = "identity") +
  scale_fill_viridis(option = "magma", discrete = T) +
  ylab("Distance Coverage (km)") +
  guides(fill=guide_legend(title = "Vehicle Type", 
                           theme = theme(legend.title = element_text(size = 10))
                           ))
b1
```

![](../plots/Q1Q2/Q1+:%20Compiling%20into%20stacked%20barplot_Distance-1.png)<!-- -->

``` r
# Some difference with stats from: 
# https://www.stadt-zuerich.ch/vbz/en/index/vbz/facts_figures/routes.html
# Why? Changed a few params in Q1 but none was an exact match with the released data
```

``` r
passvol0 <- function(dfs) {
  multiply <- function(df) {
    total_vol <- df %>%
      mutate(product = Einsteiger * Tage_DTV) %>%
      summarise(total_vol = sum(product, na.rm = TRUE)) %>%
      pull(total_vol)
    return(total_vol)
  }
  
  total_volumn <- sapply(dfs, multiply)
  total_volumn_df <- data.frame(Years = years, Total_Volume = unlist(total_volumn)/10^6)
}
tram_pass <- passvol0(tram_dfs)
trolley_pass <- passvol0(trolley_dfs)
bus_pass <- passvol0(bus_dfs)
seilbahn_pass <- passvol0(seilbahn_dfs)
forchbahn_pass <- passvol0(forchbahn_dfs)
nacht_pass <- passvol0(nacht_dfs)
all_pass <- cbind(nacht_pass, forchbahn_pass$Total_Volume, seilbahn_pass$Total_Volume, 
                  trolley_pass$Total_Volume, bus_pass$Total_Volume, tram_pass$Total_Volume)
colnames(all_pass) <- coln
melt_pass <- melt(all_pass, id = "Years")

b2 <- ggplot(melt_pass, aes(x = Years, y = value, fill = variable)) + 
  geom_bar(stat = "identity") +
  scale_fill_viridis(option = "magma", discrete = T) +
    ylab("Passenger Volume (mio)") +
  guides(fill=guide_legend(title = "Vehicle Type", 
                           theme = theme(legend.title = element_text(size = 10))
                           ))
b2
```

![](../plots/Q1Q2/Q1+:%20Compiling%20into%20stacked%20barplot_Volume-1.png)<!-- -->

``` r
# to do: by years, compute passenger/km of public transport coverage

compute_pkm <- function(df) {
  df <- df %>%
    mutate(product = Besetzung * Distanz * Tage_DTV / 443037) # the number of inhabitants in ZH in 2023
  df %>% summarise(pkm = sum(product, na.rm = TRUE))
}

pkm <- sapply(dfs, compute_pkm)

pkm_df <- data.frame(Years = years, pass_km = unlist(pkm)/10^3)

p <- ggplot(pkm_df, aes(x = Years, y = pass_km)) +
  geom_bar(stat = "identity", fill = "#de4968", color = "grey") +
  labs(title = "Utilization of Public Transportation", x = "Years", y = "Passenger-km per capita") +
  scale_y_continuous(labels = label_comma())

# ggsave("results/Q2.png", width = 8, height = 6, dpi = 600)
print(p)
```

![](../plots/Q1Q2/Q2:%20Utilization%20of%20the%20intrastructure%20by%20passenger/km%20per%20capita-1.png)<!-- -->

The purpose of this question is to compare with the data collected with
other major cities of the world.

``` r
# to compare with statistica data as a proxy to metro
pkm <- sapply(tram_dfs, compute_pkm)

pkm_df <- data.frame(Years = years, pass_km = unlist(pkm)/10^6)

p <- ggplot(pkm_df, aes(x = Years, y = pass_km)) +
  geom_bar(stat = "identity", fill = "#de4968", color = "grey") +
  labs(title = "Utilization of Tramways", x = "Years", y = "Passenger (k)-km") +
  scale_y_continuous(labels = label_comma())
  
# ggsave("results/Q2tram.png", width = 8, height = 6, dpi = 600)
print(p)
```

![](../plots/Q1Q2/Q2+:%20Utilization%20of%20the%20intrastructure%20by%20passenger/km%20trams%20only%20not%20per%20capita-1.png)<!-- -->

``` r
## Do we normalize? Traffic volume differs a lot i.e. Trolleybus vs Bus -> log scale
# onlt the first 2 elements in $FZ_AB is enough for sorting into per hour of 24h
# unique(substr(df$FZ_AB, 1, 2))
# Same problem with Pascal - everything shifted by 4h
df$FZ_AB <- as.integer(substr(df$FZ_AB, 1, 2))
df$FZ_AB <- df$FZ_AB %% 24
# Check if it's correct with Nachtnetz times
# unique(subset(df, Nachtnetz != 0)$FZ_AB)

# Split by vehicle type for 2023 data
line_df <- merge(df, linie, by = "Linien_Id")
# unique(line_df$VSYS)
tram_df <- line_df[line_df$VSYS == "T", ]
bus_df <- line_df[line_df$VSYS == "B" | 
                  line_df$VSYS == "BL" | # Limmattal
                  line_df$VSYS == "BZ" | # Zimmerberg
                  line_df$VSYS == "BG" | # Glattal
                  line_df$VSYS == "BP", ] # Pfannenstiel
trolley_df <- line_df[line_df$VSYS == "TR", ]
sb_df <- line_df[line_df$VSYS == "SB", ]
fb_df <- line_df[line_df$VSYS == "FB", ]
nacht_df <- line_df[line_df$VSYS == "N", ]

timeslots <- sprintf("%d", 0:23)
datasets <- list(tram_df, bus_df, trolley_df, sb_df, fb_df, nacht_df)
vol_per_hour <- data.frame(matrix(NA, ncol = length(timeslots), nrow = length(datasets)))
colnames(vol_per_hour) <- timeslots

# Compute passenger sum in a given hour of a given vehicle type
pass_perhour <- function(datasets) {
  for (i in seq_along(datasets)) {
    passnum <- numeric(length(timeslots)) # Creates a vector of specified length with each element = 0, use as reset here
    for (hour in timeslots){
    pass <- sum(subset(datasets[[i]], FZ_AB == hour)$Besetzung, na.rm = TRUE)
    passnum[as.numeric(hour) + 1] <- pass # Needed to correct for r indexing
    }
    vol_per_hour[i, ] <- passnum
  }
  return(vol_per_hour)
}

res_vol_per_hour <- pass_perhour(datasets)
vehicle_types <- c("Tram", "Bus", "Trolleybus", "Seilbahn", "Forchbahn", "Nachtnetz")
res_vol_per_hour_formelt <- cbind(res_vol_per_hour, vehicle_types)

rvph_df <- melt(res_vol_per_hour_formelt, id = "vehicle_types")

# Without scaling by row
p1 <- ggplot(rvph_df, aes(variable, vehicle_types, fill= value)) + 
  geom_tile() +
  xlab("Hours of day") +
  scale_fill_viridis(option = "magma", discrete=FALSE)

# ggsave("results/heatmap_noscaling.png", width = 10, height = 8, dpi = 600)
print(p1)
```

![](../plots/Q1Q2/Q1++:%20Heatmaps%20of%20traffic%20volumn%20by%20hours%20for%20each%20vehivle%20type-1.png)<!-- -->

``` r
scale_255 <- function(x) {
  min_val <- min(x)
  max_val <- max(x)
  scaled <- (x - min_val) / (max_val - min_val) * 255
  return(scaled)
}

# Change scaling to log10
scaled_rvph <- t(apply(res_vol_per_hour+1, 1, log10))
colnames(scaled_rvph) <- timeslots
rownames(scaled_rvph) <- vehicle_types
scaled_rvph_df <- melt(scaled_rvph, id = "vehicle_types")
scaled_rvph_df$value = as.numeric(scaled_rvph_df$value)

                          
p2 <- ggplot(scaled_rvph_df, aes(Var2, Var1, fill= value)) + 
  geom_tile() +
  xlab("Hours of day") +
  ylab("Vehicle Types") +
  scale_x_continuous(breaks = 0:23) +
  guides(fill=guide_legend(title = "Passenger Volume\n(log scale)", 
                           theme = theme(legend.title = element_text(size = 8))
                           )) +
  scale_fill_viridis(option = "magma", discrete=FALSE) +
  theme(
  panel.background = element_rect(fill='transparent'), #transparent panel bg
  plot.background = element_rect(fill='transparent', color=NA), #transparent plot bg
  panel.grid.major = element_blank(), #remove major gridlines
  panel.grid.minor = element_blank(), #remove minor gridlines
  legend.background = element_rect(fill='transparent'), #transparent legend bg
  legend.box.background = element_rect(fill='transparent') #transparent legend panel
  )

# ggsave("results/heatmap_scaled.png", width = 10, height = 8, dpi = 600)
print(p2)
```

![](../plots/Q1Q2/Q1++:%20Heatmaps%20of%20traffic%20volumn%20by%20hours%20for%20each%20vehivle%20type-2.png)<!-- -->
