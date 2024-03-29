# NOAA's Storm Database Exploration
Bibek J Karki  



# Impact of Severe Weather Events in Public Health and Economy

## Synopsis

Severe weather events can cause dramatic impact on health of the public and economy of the country. Thus, the U.S. National Oceanic and Atmospheric Administration's (NOAA) storm database, which tracks the major storm and weather events, is used to address the following two questions:

1. Which types of events are most harmful to population health?
2. Which types of events have the greatest economic consequences?

Based on the dataset, which was recorded from 1950 to November, 2011, the result are as follows:

* __TORNADO__ is the most harmful weather event in regards to population health.
* __FLOOD__ causes the most property damage, while __DROUGHT__ causes the highest crop damage.


## DATA PROCESSING

### Loading the Data

First of all, we'll check if the required [Storm Data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2) is available to load into the environment. If not present, the dataset will be downloaded and then loaded into the environment.

For this project, the database was explored on the following date: 2017-01-30 18:04:59


```r
# Check if the dataset is availabe
reqFile <- "repdata%2Fdata%2FStormData.csv.bz2"

if (!file.exists(reqFile)) {
        fileURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
        download.file(fileURL,reqFile,method = "curl")
        date.download <- Sys.time()
        print(date.download)
}
```

```
## [1] "2017-01-30 18:01:37 MST"
```

```r
# Load data
stormData <- read.csv(file = bzfile(reqFile), stringsAsFactors = FALSE)
```

Let's also load the required libraries for exploratory analysis:

```r
library(dplyr)
library(ggplot2)
library(gridExtra)
```

### Summary of Data

Let's first look at the dimension of the dataset and variables used:

```r
dim(stormData)
```

```
## [1] 902297     37
```

```r
names(stormData)
```

```
##  [1] "STATE__"    "BGN_DATE"   "BGN_TIME"   "TIME_ZONE"  "COUNTY"    
##  [6] "COUNTYNAME" "STATE"      "EVTYPE"     "BGN_RANGE"  "BGN_AZI"   
## [11] "BGN_LOCATI" "END_DATE"   "END_TIME"   "COUNTY_END" "COUNTYENDN"
## [16] "END_RANGE"  "END_AZI"    "END_LOCATI" "LENGTH"     "WIDTH"     
## [21] "F"          "MAG"        "FATALITIES" "INJURIES"   "PROPDMG"   
## [26] "PROPDMGEXP" "CROPDMG"    "CROPDMGEXP" "WFO"        "STATEOFFIC"
## [31] "ZONENAMES"  "LATITUDE"   "LONGITUDE"  "LATITUDE_E" "LONGITUDE_"
## [36] "REMARKS"    "REFNUM"
```


### Preprocessing the Data

Since we are only concerned about the health and economic impact of severe weather conditions, only a subset of original variables are required to answer the problem. The required variables are:

* EVTYPE: the type of weather event
* FATALITIES: number of fatalities due to the weather event
* INJURIES: number of non-fatal injuries due to the weather event
* PROPDMG: property damage in USD
* PROPDMGEXP: multiplier for property damage (hundreds, thousands, millions, etc.)
* CROPDMG: crop damage in USD
* CROPDMGEXP: multiplier for crop damage (hundreds, thousands, millions, etc.)


```r
# Select required variables
stormData <- select(stormData, EVTYPE, FATALITIES, INJURIES, PROPDMG, PROPDMGEXP, CROPDMG,CROPDMGEXP)

# Summary of data
str(stormData)
```

```
## 'data.frame':	902297 obs. of  7 variables:
##  $ EVTYPE    : chr  "TORNADO" "TORNADO" "TORNADO" "TORNADO" ...
##  $ FATALITIES: num  0 0 0 0 0 0 0 0 1 0 ...
##  $ INJURIES  : num  15 0 2 2 2 6 1 0 14 0 ...
##  $ PROPDMG   : num  25 2.5 25 2.5 2.5 2.5 2.5 2.5 25 25 ...
##  $ PROPDMGEXP: chr  "K" "K" "K" "K" ...
##  $ CROPDMG   : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ CROPDMGEXP: chr  "" "" "" "" ...
```



## Impact on Population Health

### Fatalities

We'll explore the data to identify number of fatalities caused by each event types. Then we'll only display the top 10 event types with maximum fatalities.


```r
# Total no. of fatalities by event type
fatalities <- aggregate(FATALITIES ~ EVTYPE, data = stormData, FUN = sum)

# Sort the fatalities
fatalities <- fatalities[order(fatalities$FATALITIES,decreasing = TRUE),]

# Print
head(fatalities,10)
```

```
##             EVTYPE FATALITIES
## 834        TORNADO       5633
## 130 EXCESSIVE HEAT       1903
## 153    FLASH FLOOD        978
## 275           HEAT        937
## 464      LIGHTNING        816
## 856      TSTM WIND        504
## 170          FLOOD        470
## 585    RIP CURRENT        368
## 359      HIGH WIND        248
## 19       AVALANCHE        224
```

### Injuries

After fatalities, we'll also identify the number of non-fatal injuries caused by event types, and then display the top 10 event types with maximum injuries.


```r
# Total no. of fatalities by event type
injuries <- aggregate(INJURIES ~ EVTYPE, data = stormData, FUN = sum)

# Sort the fatalities
injuries <- injuries[order(injuries$INJURIES,decreasing = TRUE),]

# Print
head(injuries,10)
```

```
##                EVTYPE INJURIES
## 834           TORNADO    91346
## 856         TSTM WIND     6957
## 170             FLOOD     6789
## 130    EXCESSIVE HEAT     6525
## 464         LIGHTNING     5230
## 275              HEAT     2100
## 427         ICE STORM     1975
## 153       FLASH FLOOD     1777
## 760 THUNDERSTORM WIND     1488
## 244              HAIL     1361
```

## Impact on Economy

### Property Damage

First of all, the multiplier for property damage needs to be converted into numbers for  data manipulation.


```r
# find all the unique characters used
unique(stormData$PROPDMGEXP)
```

```
##  [1] "K" "M" ""  "B" "m" "+" "0" "5" "6" "?" "4" "2" "3" "h" "7" "H" "-"
## [18] "1" "8"
```

```r
# convert to regular numerical expressions
stormData$PROPDMGEXP <- gsub("^$|\\-|\\?|\\+","0",stormData$PROPDMGEXP)
stormData$PROPDMGEXP <- gsub("h","2",stormData$PROPDMGEXP,ignore.case = TRUE)
stormData$PROPDMGEXP <- gsub("k","3",stormData$PROPDMGEXP,ignore.case = TRUE)
stormData$PROPDMGEXP <- gsub("m","6",stormData$PROPDMGEXP,ignore.case = TRUE)
stormData$PROPDMGEXP <- gsub("b","9",stormData$PROPDMGEXP,ignore.case = TRUE)

# check if all the variables are now in numerical expression
unique(stormData$PROPDMGEXP)
```

```
##  [1] "3" "6" "0" "9" "5" "4" "2" "7" "1" "8"
```

```r
# convert to integer
stormData$PROPDMGEXP <- as.integer(stormData$PROPDMGEXP)
```

Now, we'll find the top 10 event type that caused the maximum property damage.


```r
# Convert to actual property damage value in Dollas
stormData$TOTPROPDMG <- stormData$PROPDMG * 10^stormData$PROPDMGEXP

# Total property damage in USD by event type
propertyDamage <- aggregate(TOTPROPDMG~EVTYPE, data = stormData, FUN = sum)

# Sort the fatalities
propertyDamage <- propertyDamage[order(propertyDamage$TOTPROPDMG,decreasing = TRUE),]

# Print
head(propertyDamage,10)
```

```
##                EVTYPE   TOTPROPDMG
## 170             FLOOD 144657709807
## 411 HURRICANE/TYPHOON  69305840000
## 834           TORNADO  56947380676
## 670       STORM SURGE  43323536000
## 153       FLASH FLOOD  16822673978
## 244              HAIL  15735267513
## 402         HURRICANE  11868319010
## 848    TROPICAL STORM   7703890550
## 972      WINTER STORM   6688497251
## 359         HIGH WIND   5270046295
```

### Crop Damage

For crop damage, we'll follow the same procedure as for property damage. First, we'll check the multiplier for crop damage.


```r
# find all the unique characters used
unique(stormData$CROPDMGEXP)
```

```
## [1] ""  "M" "K" "m" "B" "?" "0" "k" "2"
```

```r
# convert to regular numerical expressions
stormData$CROPDMGEXP <- gsub("^$|\\?","0",stormData$CROPDMGEXP)
stormData$CROPDMGEXP <- gsub("k","3",stormData$CROPDMGEXP,ignore.case = TRUE)
stormData$CROPDMGEXP <- gsub("m","6",stormData$CROPDMGEXP,ignore.case = TRUE)
stormData$CROPDMGEXP <- gsub("b","9",stormData$CROPDMGEXP,ignore.case = TRUE)

# check if all the variables are now in numerical expression
unique(stormData$CROPDMGEXP)
```

```
## [1] "0" "6" "3" "9" "2"
```

```r
# convert to integer
stormData$CROPDMGEXP <- as.integer(stormData$CROPDMGEXP)
```

Now, we'll find the top 10 event type that caused the maximum crop damage


```r
# Convert to actual crop damage value in Dollas
stormData$TOTCROPDMG <- stormData$CROPDMG * 10^stormData$CROPDMGEXP

# Total property damage in USD by event type
cropDamage <- aggregate(TOTCROPDMG~EVTYPE, data = stormData, FUN = sum)

# Sort the fatalities
cropDamage <- cropDamage[order(cropDamage$TOTCROPDMG,decreasing = TRUE),]

# Print
head(cropDamage,10)
```

```
##                EVTYPE  TOTCROPDMG
## 95            DROUGHT 13972566000
## 170             FLOOD  5661968450
## 590       RIVER FLOOD  5029459000
## 427         ICE STORM  5022113500
## 244              HAIL  3025954473
## 402         HURRICANE  2741910000
## 411 HURRICANE/TYPHOON  2607872800
## 153       FLASH FLOOD  1421317100
## 140      EXTREME COLD  1292973000
## 212      FROST/FREEZE  1094086000
```


## RESULT

Based on data manipulation, the most adverse weather event in regards to human health is __TORNADO__. The bar chart below clearly depicts the 10 events that causes the most fatalities and injuries.


```r
# Number of fatalities plot
g1 <- ggplot(fatalities[1:10,], aes(x = EVTYPE, y = FATALITIES)) + 
        geom_bar(stat = "identity") + 
        labs(y = "No. of Fatalities") + 
        theme_bw() +
        theme(axis.text.x = element_text(angle = 45, hjust = 1),
              axis.title.x=element_blank())

# Number of injuries plot
g2 <- ggplot(injuries[1:10,], aes(x = EVTYPE, y = INJURIES)) + 
        geom_bar(stat = "identity") + 
        labs(y = "No. of Injuries") + 
        theme_bw() +
        theme(axis.text.x = element_text(angle = 45, hjust = 1),
              axis.title.x=element_blank())

# In panels
grid.arrange(g1, g2, ncol = 2,
             top = "Severe Weather Event's impact on Population Health",
             bottom = "Event Type")
```

![](Course_Project_2_files/figure-html/plot1-1.png)<!-- -->

In terms of impact in the economy, __FLOOD__ causes the highest _property damage_, and __DROUGHT__ causes the highest _crop damage_.


```r
# Property damage plot
f1 <- ggplot(propertyDamage[1:10,], aes(x=EVTYPE, y=TOTPROPDMG)) + 
        geom_bar(stat = "identity") + 
        labs(y = "Total Property  Damage (USD)") + 
        theme_bw() +
        theme(axis.text.x = element_text(angle = 45, hjust = 1),
              axis.title.x=element_blank())

# Crop damage plot
f2 <- ggplot(cropDamage[1:10,], aes(x=EVTYPE, y=TOTCROPDMG)) + 
        geom_bar(stat = "identity") + 
        labs(y = "Total Crop  Damage (USD)") + 
        theme_bw() +
        theme(axis.text.x = element_text(angle = 45, hjust = 1),
              axis.title.x=element_blank())

# In panels
grid.arrange(f1, f2, ncol = 2,
             top = "Severe Weather Event's impact on Economy",
             bottom = "Event Type")
```

![](Course_Project_2_files/figure-html/plot2-1.png)<!-- -->

## CONCLUSION

Thus, __FLOOD__, __DROUGHT__ and __TORNADO__ are the three extreme weather events that causes the most impact to the human health and the economy.
