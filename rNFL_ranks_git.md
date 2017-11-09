# /r/NFL Power Ranking Visualization
Tyler Worzel  
11/9/2017  



# Introduction

Reddit is a great place for sports fans. News, gamethreads, analysis, highlights, and much more are posted and broken down by fellow fans. It is a huge community-driven collection that satisfies any sporting fan needs, and serves as an excellent supplement to ESPN, Bleacher Report, and the like. The [/r/NFL](www.reddit.com/r/NFL) subreddit is a perfect example of a great community. 

Like nearly every popular sports website out there, Tuesday morning means NFL power rankings, where analysts rank teams 1-32 based on how good they are. Whether it be ESPN's 'power panel' of rankers, 538's ELO ranking methodology, or Bleacher Report's one man show, power rankings are a great place to gauge how your team is doing. 

But what about the community? Well, /r/NFL has their method too. Each week, 32 representatives, one from each team's fanbase, create their own power rankings which are aggregated and posted to the community, with each representative writing a little blurb on their feelings of the team week by week. These rankings are then discussed within the community, where fans are free to cheer, complain, and sometimes all out argue over their team's rankings. Even the rankers are available to defend their sometimes outlandish decisions, like why /u/whirledworld ranked the Jaguars #3 in Week 5. 

[Here](https://www.reddit.com/r/nfl/comments/75ismt/official_rnfl_week_5_power_rankings/) are the Week 5, 2017 /r/NFL power rankings. 

Now, while this is a great way to view the /r/NFL power rankings, there's so much going on behind the scenes that we aren't seeing from this write-up alone. 


# Prep Work 

## Packages Used


```r
library(tidyverse) # Reading, cleaning, visualizing
library(rvest) # Reading
library(reshape) # Melting
library(plyr) # Summary stats
library(directlabels) # Viz help
library(scales) # Viz help
library(gganimate) # Animation
library(animation) # Animation
library(magick) # Animation
```

## Getting the Data

Luckily for us, /u/NFLPowerRankers keeps track of how each ranker votes for each week in a google doc, [located here](https://docs.google.com/spreadsheets/d/e/2PACX-1vSdFW_RZwS8TAMbbVr7rRkEv5kaRduhU3CiJ1MEkIHUe-X14NykrW9IM5Rw3VE98lg_ZjYhAF-01zKO/pubhtml), which is easily read with a package like `rvest` (from `tidyverse`). With the html page read, we have to extract the table from the document. Luckily enough in this case, we can just grab the sheets and parse them into tables. The result is a list of 18 data.frames, one for weeks 0-17 of the NFL season. 


```r
ranks_2017_url <-
  read_html(
  "https://docs.google.com/spreadsheets/d/e/2PACX-1vSdFW_RZwS8TAMbbVr7rRkEv5kaRduhU3CiJ1MEkIHUe-X14NykrW9IM5Rw3VE98lg_ZjYhAF-01zKO/pubhtml"
  )
  
  ranks_2017_list <- ranks_2017_url %>%
  html_nodes("table") %>%
  html_table() 
  
  ranks_2017_list <- ranks_2017_list[5:22] #Keep only the tables that we need
```

It's probably worth double checking the number of tables and the structure of one


```r
length(ranks_2017_list) # Number of tables
```

```
## [1] 18
```

```r
glimpse(ranks_2017_list[[1]]) # Structure
```

```
## Observations: 34
## Variables: 44
## $ `` <int> 1, NA, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, ...
## $ `` <int> NA, NA, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, ...
## $ `` <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,...
## $ `` <chr> "sknich49ers", "", "Patriots", "Falcons", "Steelers", "Pack...
## $ `` <chr> "pygreg Bears", "", "Patriots", "Seahawks", "Steelers", "Pa...
## $ `` <chr> "ArbysguyBengals", "", "Patriots", "Packers", "Seahawks", "...
## $ `` <chr> "PenguinProphetBills", "", "Patriots", "Falcons", "Seahawks...
## $ `` <chr> "BlindManBaldwinBroncos", "", "Patriots", "Falcons", "Seaha...
## $ `` <chr> "ThaddeusJPBrowns", "", "Patriots", "Falcons", "Packers", "...
## $ `` <chr> "LandsdownestreetBuccaneers", "", "Patriots", "Seahawks", "...
## $ `` <chr> "TangerineDieselCardinals", "", "Patriots", "Packers", "Ste...
## $ `` <int> NA, NA, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, ...
## $ `` <chr> "milkchococurryChargers", "", "Patriots", "Falcons", "Seaha...
## $ `` <chr> "IIHURRlCANEIIChiefs", "", "Patriots", "Falcons", "Steelers...
## $ `` <chr> "morespikesColts", "", "Patriots", "Cowboys", "Seahawks", "...
## $ `` <chr> "staub81Cowboys", "", "Patriots", "Falcons", "Packers", "St...
## $ `` <chr> "yoda133113Dolphins", "", "Patriots", "Falcons", "Steelers"...
## $ `` <chr> "famouslastwordsEagles", "", "Patriots", "Falcons", "Seahaw...
## $ `` <chr> "wannaknowmynameFalcons", "", "Patriots", "Falcons", "Packe...
## $ `` <chr> "SexterminatorGiants", "", "Patriots", "Packers", "Steelers...
## $ `` <int> NA, NA, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, ...
## $ `` <chr> "preludeoflightJaguars", "", "Patriots", "Packers", "Falcon...
## $ `` <chr> "nickmangoldsbeardJets", "", "Patriots", "Seahawks", "Cowbo...
## $ `` <chr> "sosuhmeLions", "", "Patriots", "Falcons", "Giants", "Steel...
## $ `` <chr> "analogWeaponPackers", "", "Patriots", "Steelers", "Packers...
## $ `` <chr> "allsecretsknownPanthers", "", "Patriots", "Packers", "Seah...
## $ `` <chr> "SPACE_LAWYERPatriots", "", "Patriots", "Falcons", "Packers...
## $ `` <chr> "newBreedRaiders", "", "Patriots", "Falcons", "Packers", "S...
## $ `` <chr> "One_Half_HamsterRams", "", "Patriots", "Falcons", "Steeler...
## $ `` <int> NA, NA, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, ...
## $ `` <chr> "I_have_no_throwawayRavens", "", "Patriots", "Falcons", "Se...
## $ `` <chr> "ThatChrisDodgeRedskins", "", "Patriots", "Falcons", "Cowbo...
## $ `` <chr> "Naly_DSaints", "", "Patriots", "Seahawks", "Steelers", "Pa...
## $ `` <chr> "iltat_workSeahawks", "", "Seahawks", "Patriots", "Falcons"...
## $ `` <chr> "smacksawSteelers", "", "Patriots", "Packers", "Falcons", "...
## $ `` <chr> "SkarmotasticTexans", "", "Patriots", "Falcons", "Steelers"...
## $ `` <chr> "philo13181Titans", "", "Patriots", "Falcons", "Packers", "...
## $ `` <chr> "whirledworldVikings", "", "Steelers", "Patriots", "Seahawk...
## $ `` <int> NA, NA, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, ...
## $ `` <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,...
## $ `` <chr> "Teams", "", "49ers", "Bears", "Bengals", "Bills", "Broncos...
## $ `` <chr> "Median", "", "30", "27", "20", "26", "10.5", "30", "14", "...
## $ `` <chr> "Mean", "", "29.63", "26.84", "19.41", "26.16", "11.81", "3...
## $ `` <chr> "StandardDeviation", "", "1.431", "1.679", "3.613", "2.152"...
```

## Cleaning the Data

The tables are a little raw, so let's clean them up a bit. 


```r
ranks_2017_list_clean <- lapply(ranks_2017_list, function(x) {
    x <- x[!is.na(x[,1]),]
    colnames(x) <- x[1, ]    # First row as names
    x <- x[,-c(1,3)]
    x <- x[, grep("^(NA)", names(x), value = TRUE, invert = TRUE)] # Drop NA columns
    x <- x[, !(colnames(x) %in% c("Teams", "Median", "Mean", "StandardDeviation"))]
    x <- x[-1,]
    x <- x[,apply(x,2,is.unsorted)] # Drop columns that are sorted
    x <- cbind("Rank"=1:32,x)
})

# Week numbers
names(ranks_2017_list_clean) <- paste0('Week',0:17)

ranks_2017_list_clean$Week0
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Rank"],"name":[1],"type":["int"],"align":["right"]},{"label":["sknich49ers"],"name":[2],"type":["chr"],"align":["left"]},{"label":["pygreg Bears"],"name":[3],"type":["chr"],"align":["left"]},{"label":["ArbysguyBengals"],"name":[4],"type":["chr"],"align":["left"]},{"label":["PenguinProphetBills"],"name":[5],"type":["chr"],"align":["left"]},{"label":["BlindManBaldwinBroncos"],"name":[6],"type":["chr"],"align":["left"]},{"label":["ThaddeusJPBrowns"],"name":[7],"type":["chr"],"align":["left"]},{"label":["LandsdownestreetBuccaneers"],"name":[8],"type":["chr"],"align":["left"]},{"label":["TangerineDieselCardinals"],"name":[9],"type":["chr"],"align":["left"]},{"label":["milkchococurryChargers"],"name":[10],"type":["chr"],"align":["left"]},{"label":["IIHURRlCANEIIChiefs"],"name":[11],"type":["chr"],"align":["left"]},{"label":["morespikesColts"],"name":[12],"type":["chr"],"align":["left"]},{"label":["staub81Cowboys"],"name":[13],"type":["chr"],"align":["left"]},{"label":["yoda133113Dolphins"],"name":[14],"type":["chr"],"align":["left"]},{"label":["famouslastwordsEagles"],"name":[15],"type":["chr"],"align":["left"]},{"label":["wannaknowmynameFalcons"],"name":[16],"type":["chr"],"align":["left"]},{"label":["SexterminatorGiants"],"name":[17],"type":["chr"],"align":["left"]},{"label":["preludeoflightJaguars"],"name":[18],"type":["chr"],"align":["left"]},{"label":["nickmangoldsbeardJets"],"name":[19],"type":["chr"],"align":["left"]},{"label":["sosuhmeLions"],"name":[20],"type":["chr"],"align":["left"]},{"label":["analogWeaponPackers"],"name":[21],"type":["chr"],"align":["left"]},{"label":["allsecretsknownPanthers"],"name":[22],"type":["chr"],"align":["left"]},{"label":["SPACE_LAWYERPatriots"],"name":[23],"type":["chr"],"align":["left"]},{"label":["newBreedRaiders"],"name":[24],"type":["chr"],"align":["left"]},{"label":["One_Half_HamsterRams"],"name":[25],"type":["chr"],"align":["left"]},{"label":["I_have_no_throwawayRavens"],"name":[26],"type":["chr"],"align":["left"]},{"label":["ThatChrisDodgeRedskins"],"name":[27],"type":["chr"],"align":["left"]},{"label":["Naly_DSaints"],"name":[28],"type":["chr"],"align":["left"]},{"label":["iltat_workSeahawks"],"name":[29],"type":["chr"],"align":["left"]},{"label":["smacksawSteelers"],"name":[30],"type":["chr"],"align":["left"]},{"label":["SkarmotasticTexans"],"name":[31],"type":["chr"],"align":["left"]},{"label":["philo13181Titans"],"name":[32],"type":["chr"],"align":["left"]},{"label":["whirledworldVikings"],"name":[33],"type":["chr"],"align":["left"]}],"data":[{"1":"1","2":"Patriots","3":"Patriots","4":"Patriots","5":"Patriots","6":"Patriots","7":"Patriots","8":"Patriots","9":"Patriots","10":"Patriots","11":"Patriots","12":"Patriots","13":"Patriots","14":"Patriots","15":"Patriots","16":"Patriots","17":"Patriots","18":"Patriots","19":"Patriots","20":"Patriots","21":"Patriots","22":"Patriots","23":"Patriots","24":"Patriots","25":"Patriots","26":"Patriots","27":"Patriots","28":"Patriots","29":"Seahawks","30":"Patriots","31":"Patriots","32":"Patriots","33":"Steelers"},{"1":"2","2":"Falcons","3":"Seahawks","4":"Packers","5":"Falcons","6":"Falcons","7":"Falcons","8":"Seahawks","9":"Packers","10":"Falcons","11":"Falcons","12":"Cowboys","13":"Falcons","14":"Falcons","15":"Falcons","16":"Falcons","17":"Packers","18":"Packers","19":"Seahawks","20":"Falcons","21":"Steelers","22":"Packers","23":"Falcons","24":"Falcons","25":"Falcons","26":"Falcons","27":"Falcons","28":"Seahawks","29":"Patriots","30":"Packers","31":"Falcons","32":"Falcons","33":"Patriots"},{"1":"3","2":"Steelers","3":"Steelers","4":"Seahawks","5":"Seahawks","6":"Seahawks","7":"Packers","8":"Steelers","9":"Steelers","10":"Seahawks","11":"Steelers","12":"Seahawks","13":"Packers","14":"Steelers","15":"Seahawks","16":"Packers","17":"Steelers","18":"Falcons","19":"Cowboys","20":"Giants","21":"Packers","22":"Seahawks","23":"Packers","24":"Packers","25":"Steelers","26":"Seahawks","27":"Cowboys","28":"Steelers","29":"Falcons","30":"Falcons","31":"Steelers","32":"Packers","33":"Seahawks"},{"1":"4","2":"Packers","3":"Packers","4":"Cowboys","5":"Steelers","6":"Packers","7":"Steelers","8":"Falcons","9":"Raiders","10":"Raiders","11":"Packers","12":"Falcons","13":"Steelers","14":"Packers","15":"Packers","16":"Seahawks","17":"Falcons","18":"Raiders","19":"Falcons","20":"Steelers","21":"Falcons","22":"Cowboys","23":"Cowboys","24":"Steelers","25":"Cowboys","26":"Packers","27":"Chiefs","28":"Packers","29":"Cowboys","30":"Steelers","31":"Packers","32":"Cowboys","33":"Giants"},{"1":"5","2":"Cowboys","3":"Falcons","4":"Steelers","5":"Packers","6":"Steelers","7":"Cowboys","8":"Packers","9":"Falcons","10":"Cowboys","11":"Seahawks","12":"Raiders","13":"Seahawks","14":"Cowboys","15":"Steelers","16":"Cowboys","17":"Cowboys","18":"Steelers","19":"Packers","20":"Seahawks","21":"Cowboys","22":"Falcons","23":"Steelers","24":"Cowboys","25":"Seahawks","26":"Cowboys","27":"Raiders","28":"Cowboys","29":"Packers","30":"Giants","31":"Cowboys","32":"Seahawks","33":"Eagles"},{"1":"6","2":"Chiefs","3":"Raiders","4":"Chiefs","5":"Giants","6":"Giants","7":"Seahawks","8":"Raiders","9":"Giants","10":"Steelers","11":"Chiefs","12":"Steelers","13":"Cowboys","14":"Seahawks","15":"Raiders","16":"Chiefs","17":"Seahawks","18":"Seahawks","19":"Raiders","20":"Packers","21":"Raiders","22":"Raiders","23":"Seahawks","24":"Raiders","25":"Packers","26":"Steelers","27":"Packers","28":"Raiders","29":"Steelers","30":"Cowboys","31":"Raiders","32":"Raiders","33":"Chiefs"},{"1":"7","2":"Seahawks","3":"Chiefs","4":"Falcons","5":"Cowboys","6":"Cowboys","7":"Raiders","8":"Cowboys","9":"Seahawks","10":"Packers","11":"Raiders","12":"Packers","13":"Raiders","14":"Raiders","15":"Cowboys","16":"Steelers","17":"Chiefs","18":"Cowboys","19":"Steelers","20":"Raiders","21":"Seahawks","22":"Chiefs","23":"Raiders","24":"Seahawks","25":"Raiders","26":"Chiefs","27":"Seahawks","28":"Giants","29":"Raiders","30":"Seahawks","31":"Seahawks","32":"Steelers","33":"Falcons"},{"1":"8","2":"Raiders","3":"Cowboys","4":"Raiders","5":"Raiders","6":"Raiders","7":"Chiefs","8":"Chiefs","9":"Cowboys","10":"Chiefs","11":"Cowboys","12":"Chiefs","13":"Giants","14":"Chiefs","15":"Chiefs","16":"Raiders","17":"Giants","18":"Chiefs","19":"Chiefs","20":"Panthers","21":"Chiefs","22":"Cardinals","23":"Giants","24":"Chiefs","25":"Chiefs","26":"Raiders","27":"Steelers","28":"Chiefs","29":"Titans","30":"Raiders","31":"Chiefs","32":"Chiefs","33":"Titans"},{"1":"9","2":"Broncos","3":"Broncos","4":"Broncos","5":"Chiefs","6":"Chiefs","7":"Giants","8":"Broncos","9":"Chiefs","10":"Titans","11":"Ravens","12":"Vikings","13":"Ravens","14":"Broncos","15":"Giants","16":"Cardinals","17":"Raiders","18":"Giants","19":"Broncos","20":"Cowboys","21":"Giants","22":"Steelers","23":"Chiefs","24":"Giants","25":"Cardinals","26":"Giants","27":"Lions","28":"Falcons","29":"Broncos","30":"Chiefs","31":"Giants","32":"Giants","33":"Cowboys"},{"1":"10","2":"Panthers","3":"Cardinals","4":"Giants","5":"Broncos","6":"Lions","7":"Broncos","8":"Giants","9":"Cardinals","10":"Ravens","11":"Vikings","12":"Lions","13":"Bengals","14":"Cardinals","15":"Broncos","16":"Giants","17":"Broncos","18":"Titans","19":"Giants","20":"Chiefs","21":"Titans","22":"Titans","23":"Broncos","24":"Broncos","25":"Giants","26":"Bengals","27":"Giants","28":"Broncos","29":"Buccaneers","30":"Lions","31":"Broncos","32":"Broncos","33":"Packers"},{"1":"11","2":"Giants","3":"Giants","4":"Cardinals","5":"Buccaneers","6":"Titans","7":"Titans","8":"Texans","9":"Buccaneers","10":"Panthers","11":"Buccaneers","12":"Broncos","13":"Cardinals","14":"Panthers","15":"Cardinals","16":"Broncos","17":"Lions","18":"Texans","19":"Lions","20":"Buccaneers","21":"Broncos","22":"Giants","23":"Cardinals","24":"Titans","25":"Titans","26":"Ravens","27":"Texans","28":"Cardinals","29":"Chiefs","30":"Titans","31":"Titans","32":"Panthers","33":"Raiders"},{"1":"12","2":"Buccaneers","3":"Titans","4":"Panthers","5":"Titans","6":"Broncos","7":"Lions","8":"Buccaneers","9":"Broncos","10":"Buccaneers","11":"Eagles","12":"Titans","13":"Chiefs","14":"Titans","15":"Titans","16":"Buccaneers","17":"Titans","18":"Broncos","19":"Titans","20":"Titans","21":"Texans","22":"Lions","23":"Buccaneers","24":"Cardinals","25":"Broncos","26":"Redskins","27":"Buccaneers","28":"Titans","29":"Giants","30":"Panthers","31":"Texans","32":"Buccaneers","33":"Ravens"},{"1":"13","2":"Titans","3":"Panthers","4":"Titans","5":"Lions","6":"Dolphins","7":"Cardinals","8":"Titans","9":"Titans","10":"Broncos","11":"Lions","12":"Colts","13":"Broncos","14":"Giants","15":"Buccaneers","16":"Titans","17":"Texans","18":"Cardinals","19":"Cardinals","20":"Eagles","21":"Cardinals","22":"Dolphins","23":"Panthers","24":"Texans","25":"Buccaneers","26":"Titans","27":"Titans","28":"Eagles","29":"Texans","30":"Broncos","31":"Cardinals","32":"Lions","33":"Panthers"},{"1":"14","2":"Cardinals","3":"Eagles","4":"Bengals","5":"Cardinals","6":"Texans","7":"Panthers","8":"Cardinals","9":"Panthers","10":"Lions","11":"Titans","12":"Texans","13":"Texans","14":"Texans","15":"Eagles","16":"Bengals","17":"Buccaneers","18":"Buccaneers","19":"Texans","20":"Broncos","21":"Panthers","22":"Buccaneers","23":"Titans","24":"Panthers","25":"Colts","26":"Panthers","27":"Ravens","28":"Texans","29":"Cardinals","30":"Buccaneers","31":"Panthers","32":"Cardinals","33":"Chargers"},{"1":"15","2":"Lions","3":"Texans","4":"Vikings","5":"Texans","6":"Panthers","7":"Texans","8":"Panthers","9":"Vikings","10":"Dolphins","11":"Panthers","12":"Cardinals","13":"Buccaneers","14":"Lions","15":"Texans","16":"Panthers","17":"Ravens","18":"Lions","19":"Vikings","20":"Ravens","21":"Buccaneers","22":"Texans","23":"Eagles","24":"Buccaneers","25":"Texans","26":"Saints","27":"Broncos","28":"Buccaneers","29":"Panthers","30":"Redskins","31":"Lions","32":"Titans","33":"Redskins"},{"1":"16","2":"Texans","3":"Vikings","4":"Lions","5":"Ravens","6":"Redskins","7":"Eagles","8":"Eagles","9":"Eagles","10":"Cardinals","11":"Broncos","12":"Buccaneers","13":"Eagles","14":"Eagles","15":"Panthers","16":"Lions","17":"Eagles","18":"Panthers","19":"Panthers","20":"Lions","21":"Lions","22":"Redskins","23":"Texans","24":"Lions","25":"Eagles","26":"Colts","27":"Cardinals","28":"Saints","29":"Eagles","30":"Cardinals","31":"Buccaneers","32":"Redskins","33":"Vikings"},{"1":"17","2":"Bengals","3":"Buccaneers","4":"Texans","5":"Bengals","6":"Ravens","7":"Ravens","8":"Vikings","9":"Texans","10":"Vikings","11":"Bengals","12":"Giants","13":"Redskins","14":"Ravens","15":"Vikings","16":"Redskins","17":"Cardinals","18":"Eagles","19":"Buccaneers","20":"Vikings","21":"Eagles","22":"Eagles","23":"Lions","24":"Eagles","25":"Panthers","26":"Eagles","27":"Redskins","28":"Panthers","29":"Bengals","30":"Dolphins","31":"Eagles","32":"Eagles","33":"Dolphins"},{"1":"18","2":"Vikings","3":"Redskins","4":"Redskins","5":"Redskins","6":"Eagles","7":"Vikings","8":"Lions","9":"Ravens","10":"Giants","11":"Saints","12":"Panthers","13":"Panthers","14":"Dolphins","15":"Colts","16":"Saints","17":"Vikings","18":"Ravens","19":"Redskins","20":"Texans","21":"Redskins","22":"Broncos","23":"Bengals","24":"Ravens","25":"Lions","26":"Vikings","27":"Vikings","28":"Bengals","29":"Lions","30":"Texans","31":"Redskins","32":"Texans","33":"Bills"},{"1":"19","2":"Redskins","3":"Ravens","4":"Eagles","5":"Eagles","6":"Buccaneers","7":"Redskins","8":"Redskins","9":"Dolphins","10":"Bengals","11":"Colts","12":"Ravens","13":"Saints","14":"Buccaneers","15":"Lions","16":"Vikings","17":"Redskins","18":"Bengals","19":"Ravens","20":"Chargers","21":"Ravens","22":"Vikings","23":"Ravens","24":"Redskins","25":"Vikings","26":"Broncos","27":"Bengals","28":"Vikings","29":"Ravens","30":"Ravens","31":"Ravens","32":"Vikings","33":"Jaguars"},{"1":"20","2":"Eagles","3":"Colts","4":"Dolphins","5":"Panthers","6":"Cardinals","7":"Bengals","8":"Bengals","9":"Lions","10":"Eagles","11":"Giants","12":"Redskins","13":"Dolphins","14":"Vikings","15":"Redskins","16":"Ravens","17":"Bengals","18":"Dolphins","19":"Bengals","20":"Dolphins","21":"Bengals","22":"Saints","23":"Redskins","24":"Vikings","25":"Saints","26":"Cardinals","27":"Saints","28":"Lions","29":"Vikings","30":"Bengals","31":"Vikings","32":"Bengals","33":"Cardinals"},{"1":"21","2":"Ravens","3":"Lions","4":"Ravens","5":"Dolphins","6":"Vikings","7":"Dolphins","8":"Ravens","9":"Redskins","10":"Colts","11":"Texans","12":"Bengals","13":"Titans","14":"Bengals","15":"Ravens","16":"Eagles","17":"Panthers","18":"Redskins","19":"Eagles","20":"Saints","21":"Vikings","22":"Ravens","23":"Colts","24":"Bengals","25":"Redskins","26":"Chargers","27":"Panthers","28":"Redskins","29":"Redskins","30":"Eagles","31":"Saints","32":"Dolphins","33":"Buccaneers"},{"1":"22","2":"Dolphins","3":"Saints","4":"Buccaneers","5":"Saints","6":"Saints","7":"Saints","8":"Colts","9":"Bengals","10":"Saints","11":"Chargers","12":"Eagles","13":"Colts","14":"Redskins","15":"Saints","16":"Chargers","17":"Dolphins","18":"Vikings","19":"Dolphins","20":"Bengals","21":"Saints","22":"Chargers","23":"Vikings","24":"Dolphins","25":"Bengals","26":"Lions","27":"Rams","28":"Dolphins","29":"Dolphins","30":"Saints","31":"Bengals","32":"Saints","33":"Saints"},{"1":"23","2":"Colts","3":"Bengals","4":"Saints","5":"Vikings","6":"Chargers","7":"Colts","8":"Chargers","9":"Saints","10":"Texans","11":"Redskins","12":"Dolphins","13":"Lions","14":"Saints","15":"Chargers","16":"Dolphins","17":"Chargers","18":"Saints","19":"Bears","20":"Cardinals","21":"Dolphins","22":"Panthers","23":"Saints","24":"Saints","25":"Ravens","26":"Buccaneers","27":"Bills","28":"Ravens","29":"Saints","30":"Bears","31":"Dolphins","32":"Ravens","33":"Broncos"},{"1":"24","2":"Saints","3":"Chargers","4":"Colts","5":"Chargers","6":"Colts","7":"Chargers","8":"Dolphins","9":"Colts","10":"Redskins","11":"Cardinals","12":"Saints","13":"Chargers","14":"Colts","15":"Bengals","16":"Bills","17":"Colts","18":"Colts","19":"Colts","20":"Redskins","21":"Bears","22":"Bears","23":"Chargers","24":"Colts","25":"Dolphins","26":"Dolphins","27":"Eagles","28":"Chargers","29":"Chargers","30":"Vikings","31":"Chargers","32":"Chargers","33":"Bengals"},{"1":"25","2":"Chargers","3":"Dolphins","4":"Chargers","5":"Bills","6":"Bengals","7":"Bills","8":"Saints","9":"Chargers","10":"Bills","11":"Dolphins","12":"Chargers","13":"Vikings","14":"Chargers","15":"Dolphins","16":"Texans","17":"Bills","18":"Chargers","19":"Saints","20":"Colts","21":"Chargers","22":"Bengals","23":"Dolphins","24":"Chargers","25":"Bills","26":"Texans","27":"Jets","28":"Colts","29":"Bills","30":"Chargers","31":"Colts","32":"Bills","33":"Colts"},{"1":"26","2":"Bills","3":"Bears","4":"Bills","5":"Rams","6":"Rams","7":"Rams","8":"Bills","9":"Bears","10":"Chargers","11":"Rams","12":"Jaguars","13":"49ers","14":"Bills","15":"Bills","16":"Colts","17":"Saints","18":"Bills","19":"Chargers","20":"Bears","21":"Colts","22":"Bills","23":"Bears","24":"Bills","25":"Chargers","26":"Bears","27":"Chargers","28":"Jaguars","29":"Colts","30":"Bills","31":"Bears","32":"Colts","33":"Lions"},{"1":"27","2":"49ers","3":"Bills","4":"Rams","5":"Bears","6":"Bills","7":"Bears","8":"Jaguars","9":"Rams","10":"Jaguars","11":"Bears","12":"Bills","13":"Bears","14":"Bears","15":"Rams","16":"Bears","17":"Bears","18":"Rams","19":"Jaguars","20":"Jaguars","21":"Bills","22":"Rams","23":"49ers","24":"Bears","25":"Bears","26":"Bills","27":"Bears","28":"Browns","29":"Bears","30":"Jaguars","31":"Bills","32":"Bears","33":"Bears"},{"1":"28","2":"Bears","3":"Rams","4":"Bears","5":"Jets","6":"Jaguars","7":"Jaguars","8":"Bears","9":"49ers","10":"Rams","11":"Bills","12":"Bears","13":"Bills","14":"49ers","15":"Bears","16":"Rams","17":"Rams","18":"Bears","19":"Bills","20":"Rams","21":"Jaguars","22":"Colts","23":"Rams","24":"Rams","25":"Jaguars","26":"49ers","27":"Colts","28":"Bills","29":"Rams","30":"49ers","31":"Rams","32":"Rams","33":"Texans"},{"1":"29","2":"Jaguars","3":"Jaguars","4":"Browns","5":"Colts","6":"49ers","7":"Browns","8":"Rams","9":"Jaguars","10":"Bears","11":"49ers","12":"Browns","13":"Jets","14":"Rams","15":"Jaguars","16":"49ers","17":"Jaguars","18":"49ers","19":"Browns","20":"Bills","21":"Rams","22":"49ers","23":"Bills","24":"Jaguars","25":"Browns","26":"Browns","27":"Dolphins","28":"Rams","29":"Browns","30":"Browns","31":"Jaguars","32":"Jaguars","33":"49ers"},{"1":"30","2":"Browns","3":"Browns","4":"Jaguars","5":"Jaguars","6":"Browns","7":"49ers","8":"Browns","9":"Jets","10":"49ers","11":"Browns","12":"49ers","13":"Rams","14":"Browns","15":"Browns","16":"Jaguars","17":"Browns","18":"Browns","19":"49ers","20":"49ers","21":"Browns","22":"Browns","23":"Jaguars","24":"Browns","25":"49ers","26":"Rams","27":"Jaguars","28":"Bears","29":"49ers","30":"Rams","31":"Browns","32":"49ers","33":"Browns"},{"1":"31","2":"Rams","3":"49ers","4":"49ers","5":"49ers","6":"Bears","7":"Jets","8":"49ers","9":"Bills","10":"Browns","11":"Jaguars","12":"Rams","13":"Browns","14":"Jaguars","15":"49ers","16":"Browns","17":"49ers","18":"Jets","19":"Rams","20":"Browns","21":"49ers","22":"Jaguars","23":"Browns","24":"49ers","25":"Rams","26":"Jaguars","27":"49ers","28":"Jets","29":"Jaguars","30":"Colts","31":"49ers","32":"Browns","33":"Rams"},{"1":"32","2":"Jets","3":"Jets","4":"Jets","5":"Browns","6":"Jets","7":"Buccaneers","8":"Jets","9":"Browns","10":"Jets","11":"Jets","12":"Jets","13":"Jaguars","14":"Jets","15":"Jets","16":"Jets","17":"Jets","18":"Jaguars","19":"Jets","20":"Jets","21":"Jets","22":"Jets","23":"Jets","24":"Jets","25":"Jets","26":"Jets","27":"Browns","28":"49ers","29":"Jets","30":"Jets","31":"Jets","32":"Jets","33":"Jets"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

