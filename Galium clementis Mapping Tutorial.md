#Mapping Tutorial for **Galium clementis**
This is a bare-bones tutorial for running through mapping **Galium clementis**, look at CFP_Mapping ReadMe.md for a tutorial with more in-depth explanations.

###Set a working directory.
I chose to call my directory "CFP_Mapping"
```
getwd()
setwd("~/CFP_Mapping")
```
###Next you must install the necessary packages
If you have any issues installing and loading these packages, you may need to install and run RTools. An in depth tutorial for this is located in ReadMe.md
```
install.packages(c("dpylr","rgbif", "purrr", "readr", "terra", "taxize", "mapproj", 
                   "raster", "elevatr", "rgdal", "ggplot2", "ggmap", "CoordinateCleaner", 
                   "rnaturalearthdata", "rangeBuilder"))
```
###Now you must load the packages into your library
There will likely be warning messages telling you that these packages were created under a previous version of R.
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
