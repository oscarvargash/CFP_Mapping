# Mapping Tutorial for **_Galium clementis_**
This is a bare-bones tutorial for running through mapping **_Galium clementis_**, look at CFP_Mapping ReadMe.md for a tutorial with more in-depth explanations.

### Set a working directory.
I chose to call my directory "CFP_Mapping"
```
getwd()
setwd("~/CFP_Mapping")
```
### Next you must install the necessary packages
If you have any issues installing and loading these packages, you may need to install and run RTools. An in depth tutorial for this is located in ReadMe.md
```
install.packages(c("dpylr","rgbif", "purrr", "readr", "terra", "taxize", "mapproj", 
                   "raster", "elevatr", "rgdal", "ggplot2", "ggmap", "CoordinateCleaner", 
                   "rnaturalearthdata", "rangeBuilder"))
```
### Now you must load the packages into your library
```
library(terra)
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
If you ran into an issue with the package "CoordinateCleaner" and "terra" not being update, follow these steps.
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
