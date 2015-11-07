# The impact of storms and severe weather events on economy and population health in USA



## Synopsis
TBD: Synopsis: Immediately after the title, there should be a synopsis which describes and summarizes your analysis in at most 10 complete sentences.

## Data Processing
TBD: There should be a section titled Data Processing which describes (in words and code) how the data were loaded into R and processed for analysis. In particular, your analysis must start from the raw CSV file containing the data. You cannot do any preprocessing outside the document. If preprocessing is time-consuming you may consider using the cache = TRUE option for certain code chunks.


### Reading data


```r
# read.csv can open csv files from bz2 directly
storms <- read.csv("repdata-data-StormData.csv.bz2")
```

### Cleaning data
Quick look at the data shows that it needs tidying. Also it is clear that we need only a number of columns from this dataset to answer the questions of this analysis, so we'll perform some transformations described below.

```r
str(storms)
```

```
## 'data.frame':	902297 obs. of  37 variables:
##  $ STATE__   : num  1 1 1 1 1 1 1 1 1 1 ...
##  $ BGN_DATE  : Factor w/ 16335 levels "1/1/1966 0:00:00",..: 6523 6523 4242 11116 2224 2224 2260 383 3980 3980 ...
##  $ BGN_TIME  : Factor w/ 3608 levels "000","0000","0001",..: 152 167 2645 1563 2524 3126 122 1563 3126 3126 ...
##  $ TIME_ZONE : Factor w/ 22 levels "ADT","AKS","AST",..: 6 6 6 6 6 6 6 6 6 6 ...
##  $ COUNTY    : num  97 3 57 89 43 77 9 123 125 57 ...
##  $ COUNTYNAME: Factor w/ 29601 levels "","5NM E OF MACKINAC BRIDGE TO PRESQUE ISLE LT MI",..: 13513 1873 4598 10592 4372 10094 1973 23873 24418 4598 ...
##  $ STATE     : Factor w/ 72 levels "AK","AL","AM",..: 2 2 2 2 2 2 2 2 2 2 ...
##  $ EVTYPE    : Factor w/ 985 levels "   HIGH SURF ADVISORY",..: 826 826 826 826 826 826 826 826 826 826 ...
##  $ BGN_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ BGN_AZI   : Factor w/ 35 levels "","  N"," NW",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ BGN_LOCATI: Factor w/ 54429 levels ""," Christiansburg",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ END_DATE  : Factor w/ 6663 levels "","1/1/1993 0:00:00",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ END_TIME  : Factor w/ 3647 levels ""," 0900CST",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ COUNTY_END: num  0 0 0 0 0 0 0 0 0 0 ...
##  $ COUNTYENDN: logi  NA NA NA NA NA NA ...
##  $ END_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ END_AZI   : Factor w/ 24 levels "","E","ENE","ESE",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ END_LOCATI: Factor w/ 34506 levels ""," CANTON"," TULIA",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ LENGTH    : num  14 2 0.1 0 0 1.5 1.5 0 3.3 2.3 ...
##  $ WIDTH     : num  100 150 123 100 150 177 33 33 100 100 ...
##  $ F         : int  3 2 2 2 2 2 2 1 3 3 ...
##  $ MAG       : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ FATALITIES: num  0 0 0 0 0 0 0 0 1 0 ...
##  $ INJURIES  : num  15 0 2 2 2 6 1 0 14 0 ...
##  $ PROPDMG   : num  25 2.5 25 2.5 2.5 2.5 2.5 2.5 25 25 ...
##  $ PROPDMGEXP: Factor w/ 19 levels "","+","-","0",..: 16 16 16 16 16 16 16 16 16 16 ...
##  $ CROPDMG   : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ CROPDMGEXP: Factor w/ 9 levels "","0","2","?",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ WFO       : Factor w/ 542 levels ""," CI","$AC",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ STATEOFFIC: Factor w/ 250 levels "","ALABAMA, Central",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ ZONENAMES : Factor w/ 25112 levels "","                                                                                                                               "| __truncated__,..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ LATITUDE  : num  3040 3042 3340 3458 3412 ...
##  $ LONGITUDE : num  8812 8755 8742 8626 8642 ...
##  $ LATITUDE_E: num  3051 0 0 0 0 ...
##  $ LONGITUDE_: num  8806 0 0 0 0 ...
##  $ REMARKS   : Factor w/ 436781 levels "","\t","\t\t",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ REFNUM    : num  1 2 3 4 5 6 7 8 9 10 ...
```
First of all, we do not need the entire dataset for several reasons:  
- According to [http://www.ncdc.noaa.gov/stormevents/details.jsp](Storm Events Database description) there are three distinct periods when data reporting changed significantly.  
For the analysis only third period will be selected which started in January 1996 and contains data for 48 event types recorded as defined in NWS Directive 10-1605. If we decided to use the complete dataset, results of the analysis may be significantly biased due to major differences in number and types of events reported during mentioned periods.  
- We need only events which led to **fatalities** or **injuries** to answer first question.  
- We need only events which caused **property damage** or **crop damage**.  

In order to get the subset of the data based on date, we need to convert BGN_DATE to Date format and then select events which began later than January, 1996.


```r
storms <- transform(storms, BGN_DATE = as.Date(as.character(BGN_DATE), "%m/%d/%Y"))
storms1996 <- subset(storms, BGN_DATE >= as.Date("1996-01-01"))
```

Select events with either fatalities or injuries or property damage or crop damage.

```r
selected <- subset(storms1996, FATALITIES > 0 | INJURIES > 0 | PROPDMG > 0 | CROPDMG > 0)
```

It is necessary to calculate damage value with the same base. In order to do this we need to multiply PROPDMG and CROPDMG based on PROPDMGEXP and CROPDMGEXP modifiers.
Look at damage values modifiers:

```r
table(selected$PROPDMGEXP)
```

```
## 
##             +      -      0      1      2      3      4      5      6 
##   8448      0      0      0      0      0      0      0      0      0 
##      7      8      ?      B      H      K      M      h      m 
##      0      0      0     32      0 185474   7364      0      0
```

```r
table(selected$CROPDMGEXP)
```

```
## 
##             0      2      ?      B      K      M      k      m 
## 102767      0      0      0      2  96787   1762      0      0
```
We can see that only "K", "M", and "B" are used and they stand for one thousand, one million and one billion.

Adjust damage values using PROPDMGEXP and CROPDMGEXP modifiers.

```r
selected$PROPDMG[selected$PROPDMGEXP == "K"] <- 
    selected$PROPDMG[selected$PROPDMGEXP == "K"] * 1000
selected$PROPDMG[selected$PROPDMGEXP == "M"] <- 
    selected$PROPDMG[selected$PROPDMGEXP == "M"] * 1000000
selected$PROPDMG[selected$PROPDMGEXP == "M"] <- 
    selected$PROPDMG[selected$PROPDMGEXP == "M"] * 1000000000

selected$CROPDMG[selected$CROPDMGEXP == "K"] <- 
    selected$CROPDMG[selected$CROPDMGEXP == "K"] * 1000
selected$CROPDMG[selected$CROPDMGEXP == "M"] <- 
    selected$CROPDMG[selected$CROPDMGEXP == "M"] * 1000000
selected$CROPDMG[selected$CROPDMGEXP == "M"] <- 
    selected$CROPDMG[selected$CROPDMGEXP == "M"] * 1000000000
```

Next step is to clean levels of EVTYPE. Using subsetting we reduced the number of distinct EVTYPES and we need to adjust levels.


```r
# adjust levels and make them uppercase
selected$EVTYPE <- factor(toupper(as.character(selected$EVTYPE)))
dirtyevents <- as.character(levels(selected$EVTYPE))
dirtyevents
```

```
##   [1] "   HIGH SURF ADVISORY"     " FLASH FLOOD"             
##   [3] " TSTM WIND"                " TSTM WIND (G45)"         
##   [5] "AGRICULTURAL FREEZE"       "ASTRONOMICAL HIGH TIDE"   
##   [7] "ASTRONOMICAL LOW TIDE"     "AVALANCHE"                
##   [9] "BEACH EROSION"             "BLACK ICE"                
##  [11] "BLIZZARD"                  "BLOWING DUST"             
##  [13] "BLOWING SNOW"              "BRUSH FIRE"               
##  [15] "COASTAL  FLOODING/EROSION" "COASTAL EROSION"          
##  [17] "COASTAL FLOOD"             "COASTAL FLOODING"         
##  [19] "COASTAL FLOODING/EROSION"  "COASTAL STORM"            
##  [21] "COASTALSTORM"              "COLD"                     
##  [23] "COLD AND SNOW"             "COLD TEMPERATURE"         
##  [25] "COLD WEATHER"              "COLD/WIND CHILL"          
##  [27] "DAM BREAK"                 "DAMAGING FREEZE"          
##  [29] "DENSE FOG"                 "DENSE SMOKE"              
##  [31] "DOWNBURST"                 "DROUGHT"                  
##  [33] "DROWNING"                  "DRY MICROBURST"           
##  [35] "DUST DEVIL"                "DUST STORM"               
##  [37] "EARLY FROST"               "EROSION/CSTL FLOOD"       
##  [39] "EXCESSIVE HEAT"            "EXCESSIVE SNOW"           
##  [41] "EXTENDED COLD"             "EXTREME COLD"             
##  [43] "EXTREME COLD/WIND CHILL"   "EXTREME WINDCHILL"        
##  [45] "FALLING SNOW/ICE"          "FLASH FLOOD"              
##  [47] "FLASH FLOOD/FLOOD"         "FLOOD"                    
##  [49] "FLOOD/FLASH/FLOOD"         "FOG"                      
##  [51] "FREEZE"                    "FREEZING DRIZZLE"         
##  [53] "FREEZING FOG"              "FREEZING RAIN"            
##  [55] "FREEZING SPRAY"            "FROST"                    
##  [57] "FROST/FREEZE"              "FUNNEL CLOUD"             
##  [59] "GLAZE"                     "GRADIENT WIND"            
##  [61] "GUSTY WIND"                "GUSTY WIND/HAIL"          
##  [63] "GUSTY WIND/HVY RAIN"       "GUSTY WIND/RAIN"          
##  [65] "GUSTY WINDS"               "HAIL"                     
##  [67] "HARD FREEZE"               "HAZARDOUS SURF"           
##  [69] "HEAT"                      "HEAT WAVE"                
##  [71] "HEAVY RAIN"                "HEAVY RAIN/HIGH SURF"     
##  [73] "HEAVY SEAS"                "HEAVY SNOW"               
##  [75] "HEAVY SNOW SHOWER"         "HEAVY SURF"               
##  [77] "HEAVY SURF AND WIND"       "HEAVY SURF/HIGH SURF"     
##  [79] "HIGH SEAS"                 "HIGH SURF"                
##  [81] "HIGH SWELLS"               "HIGH WATER"               
##  [83] "HIGH WIND"                 "HIGH WIND (G40)"          
##  [85] "HIGH WINDS"                "HURRICANE"                
##  [87] "HURRICANE EDOUARD"         "HURRICANE/TYPHOON"        
##  [89] "HYPERTHERMIA/EXPOSURE"     "HYPOTHERMIA/EXPOSURE"     
##  [91] "ICE JAM FLOOD (MINOR"      "ICE ON ROAD"              
##  [93] "ICE ROADS"                 "ICE STORM"                
##  [95] "ICY ROADS"                 "LAKE EFFECT SNOW"         
##  [97] "LAKE-EFFECT SNOW"          "LAKESHORE FLOOD"          
##  [99] "LANDSLIDE"                 "LANDSLIDES"               
## [101] "LANDSLUMP"                 "LANDSPOUT"                
## [103] "LATE SEASON SNOW"          "LIGHT FREEZING RAIN"      
## [105] "LIGHT SNOW"                "LIGHT SNOWFALL"           
## [107] "LIGHTNING"                 "MARINE ACCIDENT"          
## [109] "MARINE HAIL"               "MARINE HIGH WIND"         
## [111] "MARINE STRONG WIND"        "MARINE THUNDERSTORM WIND" 
## [113] "MARINE TSTM WIND"          "MICROBURST"               
## [115] "MIXED PRECIP"              "MIXED PRECIPITATION"      
## [117] "MUD SLIDE"                 "MUDSLIDE"                 
## [119] "MUDSLIDES"                 "NON TSTM WIND"            
## [121] "NON-SEVERE WIND DAMAGE"    "NON-TSTM WIND"            
## [123] "OTHER"                     "RAIN"                     
## [125] "RAIN/SNOW"                 "RECORD HEAT"              
## [127] "RIP CURRENT"               "RIP CURRENTS"             
## [129] "RIVER FLOOD"               "RIVER FLOODING"           
## [131] "ROCK SLIDE"                "ROGUE WAVE"               
## [133] "ROUGH SEAS"                "ROUGH SURF"               
## [135] "SEICHE"                    "SMALL HAIL"               
## [137] "SNOW"                      "SNOW AND ICE"             
## [139] "SNOW SQUALL"               "SNOW SQUALLS"             
## [141] "STORM SURGE"               "STORM SURGE/TIDE"         
## [143] "STRONG WIND"               "STRONG WINDS"             
## [145] "THUNDERSTORM"              "THUNDERSTORM WIND"        
## [147] "THUNDERSTORM WIND (G40)"   "TIDAL FLOODING"           
## [149] "TORNADO"                   "TORRENTIAL RAINFALL"      
## [151] "TROPICAL DEPRESSION"       "TROPICAL STORM"           
## [153] "TSTM WIND"                 "TSTM WIND  (G45)"         
## [155] "TSTM WIND (41)"            "TSTM WIND (G35)"          
## [157] "TSTM WIND (G40)"           "TSTM WIND (G45)"          
## [159] "TSTM WIND 40"              "TSTM WIND 45"             
## [161] "TSTM WIND AND LIGHTNING"   "TSTM WIND G45"            
## [163] "TSTM WIND/HAIL"            "TSUNAMI"                  
## [165] "TYPHOON"                   "UNSEASONABLE COLD"        
## [167] "UNSEASONABLY COLD"         "UNSEASONABLY WARM"        
## [169] "UNSEASONAL RAIN"           "URBAN/SML STREAM FLD"     
## [171] "VOLCANIC ASH"              "WARM WEATHER"             
## [173] "WATERSPOUT"                "WET MICROBURST"           
## [175] "WHIRLWIND"                 "WILD/FOREST FIRE"         
## [177] "WILDFIRE"                  "WIND"                     
## [179] "WIND AND WAVE"             "WIND DAMAGE"              
## [181] "WINDS"                     "WINTER STORM"             
## [183] "WINTER WEATHER"            "WINTER WEATHER MIX"       
## [185] "WINTER WEATHER/MIX"        "WINTRY MIX"
```

Even after subsetting we can see a huge number of weather events compared to expected 48 events listed in [http://www.ncdc.noaa.gov/stormevents/pd01016005curr.pdf](NWS Directive 10-1605)  
EVTYPE needs to be cleaned.

Many event names in the dataset have the following defects:  
- short versions of the official event names  
- names with addidional suffexes  
- names with spaces in inappropriate places  
- combined events  
- names made up with no reference to official event names  


```r
# Events from the official document NWS Directive 10-1605
events <- c("Astronomical Low Tide", "Avalanche", "Blizzard", "Coastal Flood", "Cold/Wind Chill", "Debris Flow", "Dense Fog", "Dense Smoke", "Drought", "Dust Devil", "Dust Storm", "Excessive Heat", "Extreme Cold/Wind Chill", "Flash Flood", "Flood", "Frost/Freeze", "Funnel Cloud", "Freezing Fog", "Hail", "Heat", "Heavy Rain", "Heavy Snow", "High Surf", "High Wind", "Hurricane (Typhoon)", "Ice Storm", "Lake-Effect Snow", "Lakeshore Flood", "Lightning", "Marine Hail", "Marine High Wind", "Marine Strong Wind", "Marine Thunderstorm Wind", "Rip Current", "Seiche", "Sleet", "Storm Surge/Tide", "Strong Wind", "Thunderstorm Wind", "Tornado", "Tropical Depression", "Tropical Storm", "Tsunami", "Volcanic Ash", "Waterspout", "Wildfire", "Winter Storm", "Winter Weather")

# make upper-case version of event names
eventsUC <- toupper(events)
```

Event names as described in the official document that are found in the dataset:

```r
intersect(as.character(levels(selected$EVTYPE)), eventsUC)
```

```
##  [1] "ASTRONOMICAL LOW TIDE"    "AVALANCHE"               
##  [3] "BLIZZARD"                 "COASTAL FLOOD"           
##  [5] "COLD/WIND CHILL"          "DENSE FOG"               
##  [7] "DENSE SMOKE"              "DROUGHT"                 
##  [9] "DUST DEVIL"               "DUST STORM"              
## [11] "EXCESSIVE HEAT"           "EXTREME COLD/WIND CHILL" 
## [13] "FLASH FLOOD"              "FLOOD"                   
## [15] "FREEZING FOG"             "FROST/FREEZE"            
## [17] "FUNNEL CLOUD"             "HAIL"                    
## [19] "HEAT"                     "HEAVY RAIN"              
## [21] "HEAVY SNOW"               "HIGH SURF"               
## [23] "HIGH WIND"                "ICE STORM"               
## [25] "LAKE-EFFECT SNOW"         "LAKESHORE FLOOD"         
## [27] "LIGHTNING"                "MARINE HAIL"             
## [29] "MARINE HIGH WIND"         "MARINE STRONG WIND"      
## [31] "MARINE THUNDERSTORM WIND" "RIP CURRENT"             
## [33] "SEICHE"                   "STORM SURGE/TIDE"        
## [35] "STRONG WIND"              "THUNDERSTORM WIND"       
## [37] "TORNADO"                  "TROPICAL DEPRESSION"     
## [39] "TROPICAL STORM"           "TSUNAMI"                 
## [41] "VOLCANIC ASH"             "WATERSPOUT"              
## [43] "WILDFIRE"                 "WINTER STORM"            
## [45] "WINTER WEATHER"
```

Event names which are different from the official name or have additional suffixes:

```r
setdiff(as.character(levels(selected$EVTYPE)), eventsUC)
```

```
##   [1] "   HIGH SURF ADVISORY"     " FLASH FLOOD"             
##   [3] " TSTM WIND"                " TSTM WIND (G45)"         
##   [5] "AGRICULTURAL FREEZE"       "ASTRONOMICAL HIGH TIDE"   
##   [7] "BEACH EROSION"             "BLACK ICE"                
##   [9] "BLOWING DUST"              "BLOWING SNOW"             
##  [11] "BRUSH FIRE"                "COASTAL  FLOODING/EROSION"
##  [13] "COASTAL EROSION"           "COASTAL FLOODING"         
##  [15] "COASTAL FLOODING/EROSION"  "COASTAL STORM"            
##  [17] "COASTALSTORM"              "COLD"                     
##  [19] "COLD AND SNOW"             "COLD TEMPERATURE"         
##  [21] "COLD WEATHER"              "DAM BREAK"                
##  [23] "DAMAGING FREEZE"           "DOWNBURST"                
##  [25] "DROWNING"                  "DRY MICROBURST"           
##  [27] "EARLY FROST"               "EROSION/CSTL FLOOD"       
##  [29] "EXCESSIVE SNOW"            "EXTENDED COLD"            
##  [31] "EXTREME COLD"              "EXTREME WINDCHILL"        
##  [33] "FALLING SNOW/ICE"          "FLASH FLOOD/FLOOD"        
##  [35] "FLOOD/FLASH/FLOOD"         "FOG"                      
##  [37] "FREEZE"                    "FREEZING DRIZZLE"         
##  [39] "FREEZING RAIN"             "FREEZING SPRAY"           
##  [41] "FROST"                     "GLAZE"                    
##  [43] "GRADIENT WIND"             "GUSTY WIND"               
##  [45] "GUSTY WIND/HAIL"           "GUSTY WIND/HVY RAIN"      
##  [47] "GUSTY WIND/RAIN"           "GUSTY WINDS"              
##  [49] "HARD FREEZE"               "HAZARDOUS SURF"           
##  [51] "HEAT WAVE"                 "HEAVY RAIN/HIGH SURF"     
##  [53] "HEAVY SEAS"                "HEAVY SNOW SHOWER"        
##  [55] "HEAVY SURF"                "HEAVY SURF AND WIND"      
##  [57] "HEAVY SURF/HIGH SURF"      "HIGH SEAS"                
##  [59] "HIGH SWELLS"               "HIGH WATER"               
##  [61] "HIGH WIND (G40)"           "HIGH WINDS"               
##  [63] "HURRICANE"                 "HURRICANE EDOUARD"        
##  [65] "HURRICANE/TYPHOON"         "HYPERTHERMIA/EXPOSURE"    
##  [67] "HYPOTHERMIA/EXPOSURE"      "ICE JAM FLOOD (MINOR"     
##  [69] "ICE ON ROAD"               "ICE ROADS"                
##  [71] "ICY ROADS"                 "LAKE EFFECT SNOW"         
##  [73] "LANDSLIDE"                 "LANDSLIDES"               
##  [75] "LANDSLUMP"                 "LANDSPOUT"                
##  [77] "LATE SEASON SNOW"          "LIGHT FREEZING RAIN"      
##  [79] "LIGHT SNOW"                "LIGHT SNOWFALL"           
##  [81] "MARINE ACCIDENT"           "MARINE TSTM WIND"         
##  [83] "MICROBURST"                "MIXED PRECIP"             
##  [85] "MIXED PRECIPITATION"       "MUD SLIDE"                
##  [87] "MUDSLIDE"                  "MUDSLIDES"                
##  [89] "NON TSTM WIND"             "NON-SEVERE WIND DAMAGE"   
##  [91] "NON-TSTM WIND"             "OTHER"                    
##  [93] "RAIN"                      "RAIN/SNOW"                
##  [95] "RECORD HEAT"               "RIP CURRENTS"             
##  [97] "RIVER FLOOD"               "RIVER FLOODING"           
##  [99] "ROCK SLIDE"                "ROGUE WAVE"               
## [101] "ROUGH SEAS"                "ROUGH SURF"               
## [103] "SMALL HAIL"                "SNOW"                     
## [105] "SNOW AND ICE"              "SNOW SQUALL"              
## [107] "SNOW SQUALLS"              "STORM SURGE"              
## [109] "STRONG WINDS"              "THUNDERSTORM"             
## [111] "THUNDERSTORM WIND (G40)"   "TIDAL FLOODING"           
## [113] "TORRENTIAL RAINFALL"       "TSTM WIND"                
## [115] "TSTM WIND  (G45)"          "TSTM WIND (41)"           
## [117] "TSTM WIND (G35)"           "TSTM WIND (G40)"          
## [119] "TSTM WIND (G45)"           "TSTM WIND 40"             
## [121] "TSTM WIND 45"              "TSTM WIND AND LIGHTNING"  
## [123] "TSTM WIND G45"             "TSTM WIND/HAIL"           
## [125] "TYPHOON"                   "UNSEASONABLE COLD"        
## [127] "UNSEASONABLY COLD"         "UNSEASONABLY WARM"        
## [129] "UNSEASONAL RAIN"           "URBAN/SML STREAM FLD"     
## [131] "WARM WEATHER"              "WET MICROBURST"           
## [133] "WHIRLWIND"                 "WILD/FOREST FIRE"         
## [135] "WIND"                      "WIND AND WAVE"            
## [137] "WIND DAMAGE"               "WINDS"                    
## [139] "WINTER WEATHER MIX"        "WINTER WEATHER/MIX"       
## [141] "WINTRY MIX"
```

Modifying event names to match the ones from the official document:

```r
# This is to map all names needed for quantile 0.9 of totals
levels(selected$EVTYPE)[grep("TSTM WIND", 
                             as.character(levels(selected$EVTYPE)))] <- "THUNDERSTORM WIND"
levels(selected$EVTYPE)[grep("RIP CURRENTS", 
                             as.character(levels(selected$EVTYPE)))] <- "RIP CURRENT"
levels(selected$EVTYPE)[grep("EXTREME COLD", 
                             as.character(levels(selected$EVTYPE)))] <- "EXTREME COLD/WIND CHILL"
levels(selected$EVTYPE)[grep("^FOG", 
                             as.character(levels(selected$EVTYPE)))] <- "DENSE FOG"
levels(selected$EVTYPE)[grep("HURRICANE/TYPHOON", 
                             as.character(levels(selected$EVTYPE)))] <- "HURRICANE (TYPHOON)"
levels(selected$EVTYPE)[grep("WILD/FOREST FIRE", 
                             as.character(levels(selected$EVTYPE)))] <- "WILDFIRE"
levels(selected$EVTYPE)[grep("HURRICANE$", 
                             as.character(levels(selected$EVTYPE)))] <- "HURRICANE (TYPHOON)"
levels(selected$EVTYPE)[grep("STORM SURGE", 
                             as.character(levels(selected$EVTYPE)))] <- "STORM SURGE/TIDE"
levels(selected$EVTYPE)[grep("TYPHOON", 
                             as.character(levels(selected$EVTYPE)))] <- "HURRICANE (TYPHOON)"
levels(selected$EVTYPE)[grep("FREEZE", 
                             as.character(levels(selected$EVTYPE)))] <- "FROST/FREEZE"
```

Make new data structures to address both questions of the analysis:

```r
fatalities <- tapply(selected$FATALITIES, selected$EVTYPE, sum)
injuries <- tapply(selected$INJURIES, selected$EVTYPE, sum)
propdamage <- tapply(selected$PROPDMG, selected$EVTYPE, sum)
cropdamage <- tapply(selected$CROPDMG, selected$EVTYPE, sum)
```

Since we're interested in the most harmful event types, we select only top harmful events.


```r
fatalities <- fatalities[fatalities > quantile(fatalities, 0.9)]
injuries <- injuries[injuries > quantile(injuries, 0.9)]
propdamage <- propdamage[propdamage > quantile(propdamage, 0.9)]
cropdamage <- cropdamage[cropdamage > quantile(cropdamage, 0.9)]
```



```r
# create data.frame with the results for convinient plotting
fatalities.df <- data.frame(casualties = as.vector(fatalities), 
                       event = as.factor(names(fatalities)), 
                       harm = "fatality")

injuries.df <- data.frame(casualties = as.vector(injuries), 
                         event = as.factor(names(injuries)), 
                         harm = "injury")

harm.df <- rbind(fatalities.df, injuries.df)
```

## Results
1. Across the United States, which types of events (as indicated in the EVTYPE variable) are most harmful with respect to population health?


```r
qplot(event, 
      casualties , 
      data=harm.df, 
      geom = "bar", 
      stat = "identity", 
      fill = harm)
```

![](PA2_storms_files/figure-html/health_results-1.png) 

2. Across the United States, which types of events have the greatest economic consequences?


