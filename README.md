# CFP_Mapping
(state the purpuse of this project/goal and what "CFP" means)

## Set working directory 
Setting up your workplace is important when it comes to organization and accessing necessary files. 
setwd() will set your working director

**Example: setwd("C:/Users/Cam/Desktop/BOT_499")**
```
setwd("~/CFP_Mapping") 
```
getwd() will show you which directory you are currently using
```
getwd()
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
### If you ran into an issue with the package "CoordinateCleaner" and "terra" not being update, follow these steps.
The error seems to occur due to a older verison of "terra" being installed.
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
Sys.which("make")
install.packages("terra", type = "source")
```
This should have updated your "terra" packages, which we can check by loading the packages
```
library(CoordinateCleaner)
```
The package should be updated and no error message should appear
    
  </p>
  </details>

## Loading species table csv from WD folder, and assign as 'data'
Now we need to download our list of species.

[Species List Download](https://www.dropbox.com/scl/fi/m5rx2jjprixvcd8fjhgez/CFP_Online.xlsx?dl=0&rlkey=ypeezprkxn4cgdzmvhrlrpcy4)

Once downoload,place the data set into the folder assocaioed with the directory.
The following example has the "CFP_Online.xlsx" file within the BOT_499 folder


This file contains a list of 100 speices that are within the California Floristic Province.

**Example:(data <- read.csv("C:/Users/Cam/Desktop/BOT_499/CFP_Online.xlsx")**
```
data <- read.csv("location of folder/CFP_Online.xlsx") 
```
## Creating a Species List

**Step 1.**  Selects random sample of 100 spp., assign them to 'X'
```
X <- sample_n(data, 100)
```
**Step 2.** Write csv for 100spp.
```
write.csv(X, file = "100_Spp_Sample.csv")
```

**Step 3.** Loading 100 spp. data, assigning to 'Sample_Set'
```
Sample_Set <- read.csv("100_Spp_Sample.csv")
```
**Step 4.** Pull out only Genus and Spp., assign to 'Species_List"
```
Species_List <- c(Sample_Set$Name)
```
**Step 5.** How to View our Speecies list
```
Species_List
```
**Step 6.** Create CSV file for our Species list
```
write.csv(Species_List, file = "Species_List.csv")
```

**Step 7.** Assign value to the new csv file
```
file_url <- "Species_List.csv" 
``` 
# Creating a taxon key

This file is from The Global Biodiversity Information Facility (GBIF) and it is quite large, and contains GeoData of plant speices


[GBIF Data Download](https://www.dropbox.com/s/nef1p0dmkakcvtr/0000436-210914110416597.csv?dl=0)

This download contains a file named "0000436-210914110416597.csv"

After completeing the download, place the data set into the folder assocaioed with the directory. So that you can access it through R.
```
data <- read.csv("0000436-210914110416597.csv") 
```
Assign value to our new gbif file
```
Taxon_Keys <- read.csv("0000436-210914110416597.csv", sep = "\t")
```
Sort then tally by species
```
Taxon_Keys_Spp <- c(Taxon_Keys$species)
Taxon_Keys_Spp_Tally <- Taxon_Keys_Spp %>% group_by(species) %>% tally()
```
If a console message pops up with "Error in tally(.) : could not find function "tally"." Press "Click Me" and follow the steps.
<details><summary>CLICK ME</summary>
<p>
 
**Step 1.** Reset R.

![Rest_R](https://user-images.githubusercontent.com/99222277/153778610-77351921-c65c-48a7-bbe5-70b3447fb129.png)
 
**Step 2.** Remove packages "rlang" and "dplyr"
 ```
remove.packages("rlang")
remove.packages("dplyr")
```
**Step 3.** Re-install the packages.
```
install.packages("rlang")
install.packages("dplyr")
```
**Step 4.** Load Library
```
library(rlang)
library(dplyr)
```
**Step 5.** Sort by tally
```
Taxon_Keys_Spp <- c(Taxon_Keys$species)
Taxon_Keys_Spp_Tally <- Taxon_Keys_Spp %>% group_by(species) %>% tally()
```

</p>
</details>

### Grouping species
This will create a list of our filtered species
```
Taxon_Keys_Species_List <- Taxon_Keys %>% group_by(species) %>% tally()
```
## Filtering out Lat and Long 

Filtering out any invalid Latitude values

```
geodata <- Taxon_Keys %>% filter(!is.na(decimalLatitude)) 
```
Filtering out any invaild Longitude Value

```
geodata2 <- geodata %>% filter(!is.na(decimalLongitude))
```
### Group species names from geodata2 (Removes 1 spp. ["Calochortus indecorus"])
```
geodata2_Species_List <- geodata2 %>% group_by(species) %>% tally()
geodata2_Species_List
```
### Latitude plotting
```
geodata2$decimalLatitude
plot(geodata$decimalLatitude)
hist(geodata$decimalLatitude, breaks=60)
```
### Filtering out occurrences with a Latitude beneath 1
This step will filter out any data that is in the Southern Hemisphere, since our area of study is in the Northern Hemishere
```
NLat <- geodata2 %>% filter(decimalLatitude < 1) 
```
### Pull out list of species names from NLAT
```
Species_NLat <- NLat %>% group_by(species) %>% tally()
```
### Plot Longitude & Latitude data together
```
plot(geodata2$decimalLongitude, geodata2$decimalLatitude)
```

# Beginning of individual spp cleaning

Isolate individual species occurrences (replace example & "Genus species" where appropriate)
This will filter our data for a specific speices.
**Example:(Krameria_erecta<- geodata2 %>% filter(species == "Krameria erecta")**
```
Genus_species<- geodata2 %>% filter(species == "Genus species")
```
## Plotting out uncleaned data to get coordinate frame
```
plot(Genus_species$decimalLongitude, Genus_species$decimalLatitude)
```
## Creating a initial base map 
Now we can set our basemap along with the bounding box, the following figure shows how the bounding box is set
![smaller_xy](https://user-images.githubusercontent.com/99222277/153784782-0c2c3247-f20d-4d8b-8191-0742a47a721f.png)

We suggest changing the values to encompass the range of the species and its proximity to the CFP.


```
basemap <-  get_map(location = c(-140, -60, -32, 60), zoom = 3)
ggmap(basemap)
```
Plot data over basemap of CFP

**Example:(ggmap(basemap) + geom_point(data = Krameria_erecta_2, aes(x=decimalLongitude, y=decimalLatitude, color=species))**

```
ggmap(basemap) + geom_point(data = Genus_speies_2, aes(x=decimalLongitude, y=decimalLatitude, color=species))


```
## Building second base map of average range of data.
Changing the bounding box may be neccesary to encompass a plant species distrubtion.

(click me for this)
example_1 <- example %>% filter(decimalLatitude > example_LPLat, decimalLatitude < example_UPLat)
```
**Step 3.** Longitude Clean-up
```

LPLon <- quantile(example_1$decimalLongitude, c(0.005))
UPLon <- quantile(example_1$decimalLongitude, c(0.995))
example_2 <- example_1 %>% filter(decimalLongitude < example_UPLon, decimalLongitude > example_LPLon)
```
**Step 4.** Ploting the data on to the Basemap
```
ggmap(basemap2) + geom_point(data = Krameria_erecta_2, aes(x=decimalLongitude, y=decimalLatitude, color=species))
```
## Beginning of coordinate cleaning

Example: 
![Example_Flagged](https://user-images.githubusercontent.com/99222277/153942636-3dc69b93-e386-4b8b-9015-d2736e7deb8c.png)

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
Removing flagged occurence

Isolate the flagged obs. (need to figure what these lines do, lines in skelton key)
```
eample_dat_fl <- example[!flags_example$.summary,]
```
Exclude flagged obs.
```
example_dat_cl <- example[flags_example$.summary,]
```
Plotting data with flags removed over basemap2
```
ggmap(basemap2) + geom_point(data = Genus_species_example, aes(x=decimalLongitude, y=decimalLatitude, color=species, size = 9))
```
Opitional Download
```
ggsave(filename = "Genus_species_distribuition.pdf")
```
(Note from Alexander)Probably need to filter through occurrences manually to remove any outliers remaining

# Beginning on Polygon work
(short explaination of why we are creating polygon/ shapefiles)
```
Genus_species_Poly_1 <- getDynamicAlphaHull(Genus_species, fraction = 0.95, partCount = 4, buff = 10000, initialAlpha = 3,
                                    coordHeaders = c('decimalLongitude', 'decimalLatitude'), clipToCoast = 'terrestrial',
                                    proj = "+proj=longlat +datum=WGS84", alphaIncrement = 1, verbose = TRUE)
```
Step 1. Plot polygon  
```
plot(Genus_species_Poly_1[[1]], col=transparentColor('dark green', 0.5), border = NA) 
```
Step 2. Plot data points onto polygon
```
points(Genus_species[,c('decimalLongitude','decimalLatitude')], cex = 0.5, pch = 3)
```

Step 3. Calculate the area of our species distribution in Kilometers
```
Genus_species_Area <- area(Genus_species_Poly_1[[1]]) /1000000
Genus_species_Area
```
