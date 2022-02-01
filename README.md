# Playlist Export
A little RStudio project for exporting playlists from a Spotify JSON file to .csv (for use in MS Excel, Google Sheets, or Numbers). Provided with the caveats that I am not an expert and figured this out based on trial and error. There may be more elegant ways of achieving the same outcome. For those who prefer a more automatic option, [Exportify by watsonbox](https://github.com/watsonbox/exportify) looks good.

If you do not have RStudio installed and would like to give it try follow the instructions and get the free downloads at the [RStudio website](https://www.rstudio.com/products/rstudio/download/#download).

## About this tutorial

I have provided an accompanying file in RMarkdown (.Rmd) format to allow you download and to run each section of the code in RStudio on your own computer and see the results. The demo data file is called [Playlist1](../blob/main/Playlist1.json]. **For the best results** put this into a new folder called MyData on your desktop. This will be close to the experience of working with your own data export from Spotify. 

### Getting your data

To get your JSON data from Spotify you need to go to your [Account page > Privacy settings > Download your data](https://www.spotify.com/uk/account/privacy/ "Spotify UK Link") (UK link - you need to change this URL based on your location). Once you have requested your data it takes a few days for Spotify to email a download link to you. When you use the link you will be prompted to login to your account. The export file will be provided as a zip file. 

This tutorial is based on downloading the zip file to your Desktop and then unzipping it. The file will be called 'MyData' and the file will be called 'Playlist1.json' Interesting this file actually contains many playlists. I have included 3 playlists in my example data.

I am using the rjson by [Alex Couture-Bell](https://github.com/alexcb/rjson) and [Tidyverse packages](https://www.tidyverse.org/packages/) by Hadley Wickham and RStudio for this tutorial.


## Prepare your workspace

Load the packages and libraries that you need.

```{r setting up workspace}
# Set working directory based on where you unzipped your data download
setwd("~/Desktop/MyData/")

# Install the packages that you need
install.packages("rjson")
install.packages("tidyverse")

# Load the libraries from the packages that you need
library(rjson)
library(purrr)
library(dplyr)
library(tidyr)
```
## Import Data

First we need to import the data from the JSON file and figure out how many playlists we have.

```{r import data}
# Assign a name to our JSON playlist data which will be a list once parsed
playlists <- fromJSON(file = "Playlist1.json")

# Find out how many playlists you have - result will be shown below if you use the RMarkdown file or in the console if you copy and paste this code.
length(playlists$playlists)
```
Result for demo data
```[1] 3```

## Exporting each playlist

I have 3 playlists in demo data, the code chunks below are setup for the first playlist. You can re-run chunks 4, 5, and 6 to export each playlist to .csv (comma separated values for use in Microsoft Excel, Google Sheets, or Numbers). 

On each re-run: 
- change the number in 'map()' in chunk 4 (immediately below).
- change the name of the csv file in chunk 6 (so that you don't overwrite a previous export).

### Step 1 - Cleaning up the information from the JSON file

The data in JSON file is nested. Step 1 is to pull out, or unnest, the playlists within the JSON file. We are also creating a Tibble to contain the data from one playlist ('playlistExport') so that it is easier to work in Step 2.

```{r Step 1 - unnesting the data in the JSON file}
# code chunk 4
# Starting with the first playlist listed using 'map(1)'

playlistExport <- playlists %>%
  map(1) %>% # change the number in map() for each playlist (e.g., 1 to 3) 
  bind_rows(.id = "playlists") %>%
  unnest(cols = playlists)

playlistExport
```
If you use the demo data this will provide the result of a tibble that looks this...

| playlists | name            | lastModifiedDate | items      | numberofFollowers |
|-----------|-----------------|------------------|------------|-------------------|
| playlists | weight training | 2020-06-28       | <list [3]> | 1                 |
| playlists | weight training | 2020-06-28       | <list [3]> | 1                 |
| playlists | weight training | 2020-06-28       | <list [3]> | 1                 |
| playlists | weight training | 2020-06-28       | <list [3]> | 1                 |

### Step 2 - Clearning up the information in the 'items' column

The field above entitled 'items' is the one with the important information that we need. So, there is a second step to unnest that level.

```{r Step 2 - unnesting the information in the items column}
# code chunk 5
# Unnesting the data that I am actually interesting in the items column

playlistExport <- playlistExport$items %>%
  map(1) %>% # Do not change this number
  bind_rows(.id = "items") %>%
  unnest(cols = items)

playlistExport
```
If you use the demo data this will provide a tibble that looks like this...

| items | trackName              | artistName   | albumName              | trackUri                   |
|-------|------------------------|--------------|------------------------|----------------------------|
| 1     | Still D.R.E. (Tabata)  | Tabata Songs | Still D.R.E. (Tabata)  | spotify:track:[linkIDhere] |
| 2     | Enter Sandman (Tabata) | Tabata Songs | Enter Sandman (Tabata) | spotify:track:[linkIDhere] |
| 3     | Dark Symphony (Tabata) | Tabata Songs | Dark Symphony (Tabata) | spotify:track:[linkIDhere] |
| 4     | Hey Ya! (Tabata)       | Tabata Songs | Hey Ya! (Tabata)       | spotify:track:[linkIDhere] |

### Step 3 - export the playlist information

Exporting the first playlist to csv for MS Excel, Google Sheets, or Numbers.

```{r export to csv}
#code chunk 6

write.csv(playlistExport, file = "playlist1.csv") 
# remember to update the csv title for each playlist you export
```
Repeat the process for each of your playlists as outlined in the 'exporting each playlist' section.
