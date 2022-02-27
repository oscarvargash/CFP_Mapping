# Mapping Tutorial for **_Krameria erecta_**
This is a bare-bones tutorial for running through mapping **_Galium clementis_**, look at CFP_Mapping ReadMe.md for a tutorial with more in-depth explanations.

> Add the yellow flag to the right corner of your laptop ![image](https://user-images.githubusercontent.com/99222277/154882335-f33380f0-1527-4047-b2b1-972577050e7b.png)

### Set a working directory.
I chose to call my directory "CFP_Mapping"
```
setwd("~/CFP_Mapping")
```
### Next you must install the necessary packages
If you have any issues installing and loading these packages, you may need to install and run RTools. An in depth tutorial for this is located in ReadMe.md
```
install.packages(c("dpylr","rgbif", "purrr", "readr", "terra", "taxize", "mapproj", 
                   "raster", "elevatr", "rgdal", "ggplot2", "ggmap", "CoordinateCleaner", 
                   "rnaturalearthdata", "rangeBuilder"))
```
### Now we can load the packages
```
library(dplyr)
library(purrr)
library(readr)  
library(magrittr) 
library(rgbif) 
library(taxize) 
library(mapproj)
library(raster)
library(elevatr)
library(rgdal)
library(ggplot2)
library(ggmap)
library(CoordinateCleaner)
library(rnaturalearth)
library(rangeBuilder)
```
> Change your flag to green once you are good to continue ![image](https://user-images.githubusercontent.com/99222277/154882595-b2448b1c-473f-4e83-9d72-1d401ebcb5e6.png)

If you ran into an issue with the package "CoordinateCleaner" and "terra" not being update, follow these steps.
The error seems to occur due to a older verison of "terra" being installed.
>If you did run into the formentioned issue place the yellow flag on your upper right corner ![image](https://user-images.githubusercontent.com/99222277/155015751-fbcdd26b-d7e2-470d-a801-1e553123c8fc.png)



<details><summary> CLCK ME </summary>
  <p>
    
**Step 1.** Download and instal [RTools](https://cran.r-project.org/bin/windows/Rtools/rtools40.html)

**Step 2.** Create a txt. file named " .Renviron ".

**Step 3.** Save the file to your documents.

**Step 4.** Restart R 


![Rest_R](https://user-images.githubusercontent.com/99222277/153778610-77351921-c65c-48a7-bbe5-70b3447fb129.png)


**Step 5.** Run the following lines in order and one at a time. This may take several minutes.
```
write('PATH="${RTOOLS40_HOME}\\usr\\bin;${PATH}"', file = "~/.Renviron", append = TRUE)
```
```
Sys.which("make")
```
```
install.packages("terra", type = "source")
```
This should have updated your "terra" packages, which we can check by loading the packages
```
library(CoordinateCleaner)
```
The package should be updated and no error message should appear
    
  </p>
  </details>
  
> Change your flag to green once you are good to continue ![image](https://user-images.githubusercontent.com/99222277/155015835-3816eda1-6959-4b81-abe1-1816a605dd8c.png)


# Creating a taxon key
[Species Geodata](https://drive.google.com/drive/u/0/folders/1C3myAw-06w_DJxgtrdG5K17l_2OGNN94)

This download contains a file named "Krameria_erecta_geo.csv"

After completeing the download, place the data set into the folder assocaioed with the directory. So that you can access it through R.
```
data <- read.csv("Krameria_erecta_geo.csv") 
```
"Krameria_erecta <- data

# Beginning of individual spp cleaning

Plotting out uncleaned data to get coordinate frame
```
plot(Krameria_erecta$decimalLongitude, Krameria_erecta$decimalLatitude)
```

Building a initial base map 
```
basemap <-  get_map(location = c(-140, -60, -32, 60), zoom = 3)
ggmap(basemap)
```

# plot data over basemap of CFP
```
ggmap(basemap) + geom_point(data = Krameria_erecta, aes(x=decimalLongitude, y=decimalLatitude, color=species))
```

Ploting the data on to the Basemap
```
basemap2 <-  get_map(location = c(-125, 20, -100, 40), zoom =5)
ggmap(basemap2)

```

# plot data over basemap2
```
ggmap(basemap2) + geom_point(data = Krameria_erecta_2, aes(x=decimalLongitude, y=decimalLatitude, color=species))
```


## Beginning of coordinate cleaning
```
flags_Krameria_erecta <- clean_coordinates(x = Krameria_erecta, 
                                        lon = "decimalLongitude", 
                                        lat = "decimalLatitude",
                                        countries = "countryCode",
                                        species = "species",
                                        tests = c("capitals", "centroids", "equal","gbif", "institutions",
                                                  "zeros", "outliers","seas"),
                                        outliers_method = "quantile",
                                        outliers_mtp = 2,
                                        outliers_td = 60,
                                        outliers_size = 100)
 ```
Isolate the flagged obs.
```
Krameria_erecta_dat_fl <- Krameria_erecta[!flags_Krameria_erecta$.summary,]
```
Exclude flagged obs.
```
Krameria_erecta_dat_cl <- Krameria_erecta[flags_Krameria_erecta$.summary,]
```
Plotting flagged and non-flagged species
```
plot(flags_Krameria_erecta, lon = "decimalLongitude", lat = "decimalLatitude")
```
Plotting data with flags removed over basemap2
```
ggmap(basemap2) + geom_point(data = Krameria_erecta_dat_cl, aes(x=decimalLongitude, y=decimalLatitude, color=species))
```
# If you'd like to download the species distribution map directly
```
ggsave(filename = "Krameria_erecta_distribuition.pdf")
```

# Beginning on Polygon work 
**Step 1.** Creating the polygon
```
Krameria_erecta_Poly_1 <- getDynamicAlphaHull(Krameria_erecta_dat_cl, fraction = 0.95, partCount = 4, buff = 10000, initialAlpha = 3,
                                    coordHeaders = c('decimalLongitude', 'decimalLatitude'), clipToCoast = 'terrestrial',
                                    proj = "+proj=longlat +datum=WGS84", alphaIncrement = 1, verbose = TRUE)
```
**Step 2.** Plot the polygon  
```
plot(Krameria_erecta_Poly_1[[1]], col=transparentColor('dark green', 0.5), border = NA) 
```
**Step 3.** Plot data points onto polygon
```
points(Krameria_erecta_dat_cl[,c('decimalLongitude','decimalLatitude')], cex = 0.5, pch = 3)
```
**Step 4.** View the polygon on top of an interactive map using Leaflet
```
install.packages("leaflet")
library(leaflet)
PolygonMap <- leaflet()
PolygonMap <- addTiles(PolygonMap)
PolygonMap <- addPolygons(PolygonMap, data = Krameria_erecta_Poly_1[[1]])
PolygonMap
```

**Step 5.** Calculate the area of our species distribution in Kilometers
```
Krameria_erecta_Area <- area(Krameria_erecta_Poly_1[[1]]) /1000000
Krameria_erecta_Area
```
