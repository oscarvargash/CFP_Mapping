# CFP_Mapping

## Set working directory 
```
getwd()
setwd("~/CFP_Mapping") 
```
## Packages
To install the package we neet to type:
```
install.packages(c("dpylr","rgbif", "purrr", "readr", "taxize", "mapproj", 
                   "raster", "elevatr", "rgdal", "ggplot2", "ggmap", 
                   "rnaturalearthdata", "rangeBuilder", "CoordinateCleaner"))
```
## Load Library
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
## If you ran into an issue with the package "CoordinateClear" follow these steps.
Step 1. Download and instal [RTools](https://cran.r-project.org/bin/windows/Rtools/rtools40.html)

Step 2. Create a txt. file named " .Renviron ".

Step 3. Save the file to your documents.

Step 4. Restart R 


![Rest_R](https://user-images.githubusercontent.com/99222277/153778610-77351921-c65c-48a7-bbe5-70b3447fb129.png)


Step 5. Run the following lines in order and one at a time. This may take several minutes.
```
write('PATH="${RTOOLS40_HOME}\\usr\\bin;${PATH}"', file = "~/.Renviron", append = TRUE)
Sys.which("make")
install.packages("terra", type = "source")
```
### This should have updated your "terra" packages
To check this run
```
library(CoordinateCleaner)
```
The package should be update and no error message should appear

## Load species table csv from WD folder, and assign as 'data'
Now we need to download our list of species.
[Species List](https://docs.google.com/spreadsheets/d/17WbYpslUYJon8NQS3xu7AoEDRZVxmHHp/edit?usp=sharing&ouid=101951801409981981542&rtpof=true&sd=true)
Once downoload we need to load the file into R
example (data <- read.csv("C:/Users/Cam/Desktop/BOT_499/CFP_Online.xlsx") 

```
data <- read.csv("location of folder/CFP_Online.xlsx") 
```

This file is from The Global Biodiversity Information Facility (GBIF) and it quite large
```
data <- read.csv("0000436-210914110416597.csv") 
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

### Isolate individual species occurrences (replace example & "Genus species" where appropriate)
This will filter our data for a specific speices.
Example,(Krameria_erecta<- geodata2 %>% filter(species == "Krameria erecta")
```
Genus_species<- geodata2 %>% filter(species == "Genus species")
```
## Plotting out uncleaned data to get coordinate frame
```
plot(Genus_species$decimalLongitude, Genus_species$decimalLatitude)
```
## Build a initial base map 
```
basemap <-  get_map(location = c(-140, -60, -32, 60), zoom = 3)
ggmap(basemap)
```
## plot data over basemap of CFP
```
ggmap(basemap2) + geom_point(data = Genus_speies, aes(x=decimalLongitude, y=decimalLatitude, color=species))
```
## Building second base map of average range of data
This will change for every species
```
basemap2 <-  get_map(location = c(-122.5, 35.5, -121, 37), zoom = 8)
ggmap(basemap2)
```

# Beginning of coordinate cleaning
```
Genus_species_example <- clean_coordinates(x = Genus_species, 
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
ggmap(basemap2) + geom_point(data = Genus_species_example, aes(x=decimalLongitude, y=decimalLatitude, color=species, size = 9))
```
## If you'd like to download the species distribution map directly
```
ggsave(filename = "Genus_species_distribuition.pdf")
```
### Probably need to filter through occurrences manually to remove any outliers remaining

# Beginning on Polygon work
```
Genus_species_Poly_1 <- getDynamicAlphaHull(Genus_species, fraction = 0.95, partCount = 4, buff = 10000, initialAlpha = 3,
                                    coordHeaders = c('decimalLongitude', 'decimalLatitude'), clipToCoast = 'terrestrial',
                                    proj = "+proj=longlat +datum=WGS84", alphaIncrement = 1, verbose = TRUE)
```
## Plot polygon  
```
plot(Genus_species_Poly_1[[1]], col=transparentColor('dark green', 0.5), border = NA) 
```
## Plot data points onto polygon
```
points(Genus_species[,c('decimalLongitude','decimalLatitude')], cex = 0.5, pch = 3)
```

# Calculate the area of our species distribution in Kilometers
```
Genus_species_Area <- area(Genus_species_Poly_1[[1]]) /1000000
Genus_species_Area
```
