# CFP_Mapping

## Set working directory 
```
getwd()
setwd("~/CFP_Mapping") 
```
## Download following packages
```
install.packages(c("dpylr","rgbif", "purrr", "readr", "taxize", "mapproj", 
                   "raster", "elevatr", "rgdal", "ggplot2", "ggmap", "CoordinateCleaner", 
                   "rnaturalearthdata", "rangeBuilder"))
```
## We must install R-Tools before Terra can be installed and unpacked
```
write('PATH="${RTOOLS40_HOME}\\usr\\bin;${PATH}"', file = "~/.Renviron", append = TRUE)
Sys.which("make")
install.packages("jsonlite", type = "source")
```
## Coordinate Cleaner and Terra can now be installed
```
install.packages("CoordinateCleaner")
library(CoordinateCleaner)
library(terra)
```
## Load following packages
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
## Load species table csv from WD folder, and assign as 'data'
```
data <- read.csv("0000436-210914110416597_geo.csv") 
```
## Assign value to our new gbif file
```
Taxon_Keys <- read.csv("0000436-210914110416597.csv", sep = "\t")
```


## Sort then tally by species
```
Taxon_Keys_Spp <- c(Taxon_Keys$species)
Taxon_Keys_Spp_Tally <- Taxon_Keys_Spp %>% group_by(species) %>% tally()
```
## Group species names from Taxon_Keys
```
Taxon_Keys_Species_List <- Taxon_Keys %>% group_by(species) %>% tally()
```
## Filtering out any NA in Latitudes
```
geodata <- Taxon_Keys %>% filter(!is.na(decimalLatitude)) 
```
## Filter out any NA in Longitudes
```
geodata2 <- geodata %>% filter(!is.na(decimalLongitude))
```
## Group species names from geodata2 (Removes 1 spp. ["Calochortus indecorus"])
```
geodata2_Species_List <- geodata2 %>% group_by(species) %>% tally()
```
## Latitude plotting
```
geodata2$decimalLatitude
plot(geodata$decimalLatitude)
hist(geodata$decimalLatitude, breaks=60)
```
## Filtering out occurrences with a Latitude beneath 1
```
NLat <- geodata2 %>% filter(decimalLatitude < 1) 
```
## Pull out list of species names from NLAT
```
Species_NLat <- NLat %>% group_by(species) %>% tally()
```
## Plot Longitude & Latitude data together
```
plot(geodata2$decimalLongitude, geodata2$decimalLatitude)
```

# Beginning of individual spp cleaning

## Isolate individual species occurrences (replace example & "Genus species" where appropriate)
```
Galium_clementis<- geodata2 %>% filter(species == "Galium clementis")
```
## Plotting out uncleaned data to get coordinate frame
```
plot(Galium_clementis$decimalLongitude, Galium_clementis$decimalLatitude)
```
## Build a initial base map 
```
basemap <-  get_map(location = c(-140, -60, -32, 60), zoom = 3)
ggmap(basemap)
```
## plot data over basemap of CFP
```
ggmap(basemap2) + geom_point(data = Galium_clementis, aes(x=decimalLongitude, y=decimalLatitude, color=species))
```
## Building second base map of average range of data
```
basemap2 <-  get_map(location = c(-122.5, 35.5, -121, 37), zoom = 8)
ggmap(basemap2)
```

# Beginning of coordinate cleaning
```
Galium_clementis_example <- clean_coordinates(x = Galium_clementis, 
                                        lon = "decimalLongitude", 
                                        lat = "decimalLatitude",
                                        countries = "countryCode",
                                        species = "species",
                                        tests = c("capitals", "centroids", "equal","gbif", "institutions",
                                                  "zeros", "outliers","seas"),
                                        outliers_method = "quantile",
                                        outliers_mtp = 5,
                                        outliers_td = 60,
                                        outliers_size = 100)
```
## plotting data with flags removed over basemap2
```
ggmap(basemap2) + geom_point(data = Galium_clementis_example, aes(x=decimalLongitude, y=decimalLatitude, color=species, size = 9))
```
## If you'd like to download the species distribution map directly
```
ggsave(filename = "Galium_clementis_distribuition.pdf")
```
### Probably need to filter through occurrences manually to remove any outliers remaining

# Beginning on Polygon work
```
Galium_clementis_Poly_1 <- getDynamicAlphaHull(Galium_clementis, fraction = 0.95, partCount = 4, buff = 10000, initialAlpha = 3,
                                    coordHeaders = c('decimalLongitude', 'decimalLatitude'), clipToCoast = 'terrestrial',
                                    proj = "+proj=longlat +datum=WGS84", alphaIncrement = 1, verbose = TRUE)
```
## Plot polygon  
```
plot(Galium_clementis_Poly_1[[1]], col=transparentColor('dark green', 0.5), border = NA) 
```
## Plot data points onto polygon
```
points(Galium_clementis[,c('decimalLongitude','decimalLatitude')], cex = 0.5, pch = 3)
```

# Calculate the area of our species distribution in Kilometers
```
Galium_clementis_Area <- area(Galium_clementis_Poly_1[[1]]) /1000000
Galium_clementis_Area
```
