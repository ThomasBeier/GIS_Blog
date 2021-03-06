---
title:       "LiDAR metrics and forest micro climate"
subtitle:    ""
description: ""
date:        2021-11-11
author:      "Thomas Beier"
image:       ""
categories: ["aGIS"]
tags: ["Assessment"]
toc: true
output:
  blogdown::html_page:
    keep_md: yes
weight: 100
---




## Is it possible to derive a suitable set of predictor variables from LiDAR data to obtain a reliable prediction of the microclimate parameters temperature and humidity?

### Read the resources related to forest and identify those which you will use for attempting the task. Decide which algorithms and indices are adequate to answer the research question

Incoming solar radiation and wind speed are reduced by the forest canopy, dampening temperature and humidity variation beneath it (DE FRENNE ET AL. 2021). Extreme temperatures in the understorey are buffered: Evapotranspirative cooling and reflection and/or absorption of shortwave radiation in the canopy lead to lower temperature maxima below (DE FRENNE ET AL. 2021). At the same time, the foliage has an insulating effect against the atmosphere through protection against strong wind and traps longwave radiation beneath it. That results in higher temperature minima in the forest understorey compared to the open space (DE FRENNE ET AL. 2021). Furthermore, during photosynthesis leafs emit water vapor, increasing air humidity. Thus, LiDAR canopy metrics that describe a) the shading of the understorey, b) the capacity for photosynthesis and c) protection from wind should be suited best for the task at hand. Concretely, according to a paper by CARRASCO et al. (2019), metrics that were moderately correlated with temperature were the pixel based standard deviation of the first returns (Pearson coeff: 0.61) and leaf area density metrics like mean leaf area density (Pearson coeff: -0.72) and Shannon-index of leaf area density (-0.71). TYMEN et al. (2017) conducted a study in tropical rainforest where humidity was found to be correlated with th light penetration index (LPI) as a voxel metric.

### Apply and document these findings on base of the scripts of this units' experiences


```r
# 0 - specific setup
#-----------------------------
require(envimaR)

# MANDANTORY: defining the root folder DO NOT change this line
rootDIR = "~/edu/agis"

# define  additional packages
appendpackagesToLoad = c("lidR")
# define additional subfolders
appendProjectDirList =  c("data/lidar_org/normalized/")


# MANDANTORY: calling the setup script also DO NOT change this line
source(file.path(envimaR::alternativeEnvi(root_folder = rootDIR),"src/agis_setup.R"), 
       echo = FALSE,
       local = knitr::knit_global())
```

```
## variable alt_env_id is NOT defined
##  'COMPUTERNAME' is set by defaultvariable alt_env_value is NOT defined
##  'PCRZP' is set by defaultvariable alt_env_root_folder is NOT defined
##  'F:/BEN/edu' is set by default
```

```r
# 1 - start script
#-----------------------------
# read a small normalized .las from the already existing las-catalog

las <- readLAS(file.path(envrmt$path_normalized, "11_norm.las"))
crs(las) <- "EPSG:25832"
plot(las)

# calculate some standard metrics

sd_first_return_1m <- grid_metrics(las, func = sd(Z), res = 1, by_echo = "first")
plot(x = sd_first_return_1m, main = "sd of first returns in 1 m grid")
```

<img src="index_files/figure-html/unnamed-chunk-1-1.png" width="672" />

```r
# note: strictly speaking, it is not only the canopy height, since the road L3092 is 
# not covered by trees and returns first-return sd values of near 0. To avoid this, 
# the "func" argument could be modified to exclude returns below a certain height

# LAD

LAD <- voxel_metrics(las, func = LAD(Z), res = 5)
LAD <- na.omit(LAD)
plot(LAD, size = 10, color = "lad", colorPalette = heat.colors(50))


# Shannon-Index of LAD
LAD_Shannon <- voxel_metrics(las, func = entropy(Z), res = 5)
LAD_Shannon <-na.omit(LAD_Shannon)
plot(LAD_Shannon, size = 10, color = "V1", colorRampPalette(heat.colors(50)))
```

### References:

DE FRENNE, P., LENOIR, J., LUOTO, M., SCHEFFERS, B.R., ZELLWEGER, F., AALTO, J., ASHCROFT, M.B., CHRISTIANSEN, D.M., DECOCQ, G., DE PAUW, K., GOVAERT, S., GREISER, C., GRIL, E., HAMPE, A., JUCKER, T., KLINGES, D.H., KOELEMEIJER, I.A., LEMBRECHTS, J.J., MARREC, R., MEEUSSEN, C., OGÉE, J., TYYSTJÄRVI, V., VANGANSBEKE, P., HYLANDER, K. (2021): Forest microclimates and climate change: Importance, drivers and future research agenda. Global Change Biology 27: 2279-2297. https://doi.org/10.1111/gcb.15569 (Accessed: 26/09/2021)

Carrasco, L.; Giam, X.; Papeş, M.; Sheldon, K.S. (2019): Metrics of Lidar-Derived 3D Vegetation Structure Reveal Contrasting Effects of Horizontal and Vertical Forest Heterogeneity on Bird Species Richness. Remote Sens. 2019, 11, 743. https://doi.org/10.3390/rs11070743

Tymen, Blaise & Vincent, Gregoire & Courtois, Elodie & Heurtebize, Julien & Dauzat, Jean & Maréchaux, Isabelle & Chave, Jérôme. (2017). Quantifying micro-environmental variation in tropical rainforest understory at landscape scale by combining airborne LiDAR scanning and a sensor network. Annals of Forest Science. 74. 32. 10.1007/s13595-017-0628-z. 
