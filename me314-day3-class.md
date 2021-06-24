Working with RSQLite
================
Kenneth Benoit & Sarah Jewett (w/inspiration from **RSQLite** vignette)

## Why and how to use RSQLite

-   RSQLite is the easiest way to use a database from R because the
    package itself contains [SQLite](https://www.sqlite.org); no
    external software is needed.

-   RSQLite is a DBI-compatible interface which means you primarily use
    functions defined in the DBI package, so you should always start by
    loading DBI, not RSQLite:

``` r
library("DBI")
library(dplyr)
## 
## Attaching package: 'dplyr'
## The following objects are masked from 'package:stats':
## 
##     filter, lag
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

## Creating a new database

To create a new SQLite database, you simply supply the filename to
`dbConnect()`:

``` r
mydb <- dbConnect(RSQLite::SQLite(), "my-db.sqlite")
dbDisconnect(mydb)
# unlink() deletes the file(s) or directories
unlink("my-db.sqlite")
```

If you just need a temporary database, use either `""` (for an on-disk
database) or `":memory:"` or `"file::memory:"` (for a in-memory
database). This database will be automatically deleted when you
disconnect from it.

``` r
mydb <- dbConnect(RSQLite::SQLite(), "")
dbDisconnect(mydb)
```

## Adding data to the database

1.  Load a table using `read.csv()`

``` r
airports <- read.csv("nycflights13/airports.csv")[, -1]
# [, -1 is dropping the first column, which is just 'X' and row numbers]
planes <- read.csv("nycflights13/planes.csv")[, -1]
```

2.  Add to the database using `dbWriteTable()`:

``` r
mydb <- dbConnect(RSQLite::SQLite(), "")
dbWriteTable(mydb, "airports", airports)
dbWriteTable(mydb, "planes", planes)
dbListTables(mydb)
## [1] "airports" "planes"
```

## Queries

Queries in **RSQLite** pass SQL code directly, using `dbGetQuery()`

**SELECT** determines the columns to include in the query’s results
**LIMIT** is used in tandem with SELECT to limit the \# of records
returned

Let’s select everything from the the airports data

``` r
dbGetQuery(mydb, 'SELECT * FROM airports LIMIT 5')
##   faa                          name      lat       lon  alt tz dst
## 1 04G             Lansdowne Airport 41.13047 -80.61958 1044 -5   A
## 2 06A Moton Field Municipal Airport 32.46057 -85.68003  264 -6   A
## 3 06C           Schaumburg Regional 41.98934 -88.10124  801 -6   A
## 4 06N               Randall Airport 41.43191 -74.39156  523 -5   A
## 5 09J         Jekyll Island Airport 31.07447 -81.42778   11 -5   A
##              tzone
## 1 America/New_York
## 2  America/Chicago
## 3  America/Chicago
## 4 America/New_York
## 5 America/New_York
dbGetQuery(mydb, 'SELECT name, tzone FROM airports LIMIT 5')
##                            name            tzone
## 1             Lansdowne Airport America/New_York
## 2 Moton Field Municipal Airport  America/Chicago
## 3           Schaumburg Regional  America/Chicago
## 4               Randall Airport America/New_York
## 5         Jekyll Island Airport America/New_York
```

Limit 5 is like using head() or:

``` r
airports[1:5,]
##   faa                          name      lat       lon  alt tz dst
## 1 04G             Lansdowne Airport 41.13047 -80.61958 1044 -5   A
## 2 06A Moton Field Municipal Airport 32.46057 -85.68003  264 -6   A
## 3 06C           Schaumburg Regional 41.98934 -88.10124  801 -6   A
## 4 06N               Randall Airport 41.43191 -74.39156  523 -5   A
## 5 09J         Jekyll Island Airport 31.07447 -81.42778   11 -5   A
##              tzone
## 1 America/New_York
## 2  America/Chicago
## 3  America/Chicago
## 4 America/New_York
## 5 America/New_York
```

there is also SELECT DISTINCT

``` r
dbGetQuery(mydb, 'SELECT DISTINCT tzone FROM airports')
##                  tzone
## 1     America/New_York
## 2      America/Chicago
## 3  America/Los_Angeles
## 4    America/Vancouver
## 5      America/Phoenix
## 6    America/Anchorage
## 7       America/Denver
## 8     Pacific/Honolulu
## 9       Asia/Chongqing
## 10                 \\N
```

10 rows using DISTINCT versus 1458 rows without it:

``` r
dbGetQuery(mydb, 'SELECT tzone FROM airports LIMIT 5')
##              tzone
## 1 America/New_York
## 2  America/Chicago
## 3  America/Chicago
## 4 America/New_York
## 5 America/New_York
```

**WHERE** filters out unwanted data

``` r
dbGetQuery(mydb, 'SELECT * FROM planes WHERE engines > 2')
##   tailnum year                    type           manufacturer
## 1  N281AT   NA Fixed wing multi engine       AIRBUS INDUSTRIE
## 2  N381AA 1956 Fixed wing multi engine                DOUGLAS
## 3  N670US 1990 Fixed wing multi engine                 BOEING
## 4  N840MQ 1974 Fixed wing multi engine           CANADAIR LTD
## 5  N854NW 2004 Fixed wing multi engine                 AIRBUS
## 6  N856NW 2004 Fixed wing multi engine                 AIRBUS
## 7  N905FJ 1986 Fixed wing multi engine AVIONS MARCEL DASSAULT
##                model engines seats speed        engine
## 1           A340-313       4   375    NA     Turbo-jet
## 2             DC-7BF       4   102   232 Reciprocating
## 3            747-451       4   450    NA     Turbo-jet
## 4              CF-5D       4     2    NA     Turbo-jet
## 5           A330-223       3   379    NA     Turbo-fan
## 6           A330-223       3   379    NA     Turbo-fan
## 7 MYSTERE FALCON 900       3    12    NA     Turbo-fan
```

This is equivalent to to the following in dplyr:

``` r
planes %>% 
    filter(engines >2)
##   tailnum year                    type           manufacturer
## 1  N281AT   NA Fixed wing multi engine       AIRBUS INDUSTRIE
## 2  N381AA 1956 Fixed wing multi engine                DOUGLAS
## 3  N670US 1990 Fixed wing multi engine                 BOEING
## 4  N840MQ 1974 Fixed wing multi engine           CANADAIR LTD
## 5  N854NW 2004 Fixed wing multi engine                 AIRBUS
## 6  N856NW 2004 Fixed wing multi engine                 AIRBUS
## 7  N905FJ 1986 Fixed wing multi engine AVIONS MARCEL DASSAULT
##                model engines seats speed        engine
## 1           A340-313       4   375    NA     Turbo-jet
## 2             DC-7BF       4   102   232 Reciprocating
## 3            747-451       4   450    NA     Turbo-jet
## 4              CF-5D       4     2    NA     Turbo-jet
## 5           A330-223       3   379    NA     Turbo-fan
## 6           A330-223       3   379    NA     Turbo-fan
## 7 MYSTERE FALCON 900       3    12    NA     Turbo-fan
```

You can make more than one condition with WHERE using AND

``` r
dbGetQuery(mydb, 'SELECT * FROM planes WHERE engines > 1 AND seats < 55 LIMIT 12')
##    tailnum year                    type manufacturer           model engines
## 1   N178JB 2005 Fixed wing multi engine      EMBRAER ERJ 190-100 IGW       2
## 2   N179JB 2005 Fixed wing multi engine      EMBRAER ERJ 190-100 IGW       2
## 3   N183JB 2005 Fixed wing multi engine      EMBRAER ERJ 190-100 IGW       2
## 4   N184JB 2005 Fixed wing multi engine      EMBRAER ERJ 190-100 IGW       2
## 5   N187JB 2005 Fixed wing multi engine      EMBRAER ERJ 190-100 IGW       2
## 6   N190JB 2005 Fixed wing multi engine      EMBRAER ERJ 190-100 IGW       2
## 7   N192JB 2005 Fixed wing multi engine      EMBRAER ERJ 190-100 IGW       2
## 8   N193JB 2005 Fixed wing multi engine      EMBRAER ERJ 190-100 IGW       2
## 9   N197JB 2006 Fixed wing multi engine      EMBRAER ERJ 190-100 IGW       2
## 10  N198JB 2006 Fixed wing multi engine      EMBRAER ERJ 190-100 IGW       2
## 11  N202AA 1980 Fixed wing multi engine       CESSNA            421C       2
## 12  N203JB 2006 Fixed wing multi engine      EMBRAER ERJ 190-100 IGW       2
##    seats speed        engine
## 1     20    NA     Turbo-fan
## 2     20    NA     Turbo-fan
## 3     20    NA     Turbo-fan
## 4     20    NA     Turbo-fan
## 5     20    NA     Turbo-fan
## 6     20    NA     Turbo-fan
## 7     20    NA     Turbo-fan
## 8     20    NA     Turbo-fan
## 9     20    NA     Turbo-fan
## 10    20    NA     Turbo-fan
## 11     8    90 Reciprocating
## 12    20    NA     Turbo-fan
```

Note the difference between these two and the use of " versus ’ for the
entire Query Hint: try using ’ for the query and ’ to specify AIRBUS….

``` r
dbGetQuery(mydb, "SELECT * FROM planes WHERE manufacturer != 'AIRBUS' LIMIT 10")
##    tailnum year                    type     manufacturer     model engines
## 1   N10156 2004 Fixed wing multi engine          EMBRAER EMB-145XR       2
## 2   N102UW 1998 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 3   N103US 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 4   N104UW 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 5   N10575 2002 Fixed wing multi engine          EMBRAER EMB-145LR       2
## 6   N105UW 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 7   N107US 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 8   N108UW 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 9   N109UW 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 10  N110UW 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
##    seats speed    engine
## 1     55    NA Turbo-fan
## 2    182    NA Turbo-fan
## 3    182    NA Turbo-fan
## 4    182    NA Turbo-fan
## 5     55    NA Turbo-fan
## 6    182    NA Turbo-fan
## 7    182    NA Turbo-fan
## 8    182    NA Turbo-fan
## 9    182    NA Turbo-fan
## 10   182    NA Turbo-fan
dbGetQuery(mydb, 'SELECT * FROM planes WHERE manufacturer != "AIRBUS" LIMIT 10')
##    tailnum year                    type     manufacturer     model engines
## 1   N10156 2004 Fixed wing multi engine          EMBRAER EMB-145XR       2
## 2   N102UW 1998 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 3   N103US 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 4   N104UW 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 5   N10575 2002 Fixed wing multi engine          EMBRAER EMB-145LR       2
## 6   N105UW 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 7   N107US 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 8   N108UW 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 9   N109UW 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
## 10  N110UW 1999 Fixed wing multi engine AIRBUS INDUSTRIE  A320-214       2
##    seats speed    engine
## 1     55    NA Turbo-fan
## 2    182    NA Turbo-fan
## 3    182    NA Turbo-fan
## 4    182    NA Turbo-fan
## 5     55    NA Turbo-fan
## 6    182    NA Turbo-fan
## 7    182    NA Turbo-fan
## 8    182    NA Turbo-fan
## 9    182    NA Turbo-fan
## 10   182    NA Turbo-fan
```

You may have noticed that despite specifiying AIRBUS, there is still
AIRBUS INDUSTRIES. This is where we can use matching conditions.

\_ matches 1 character precisely

% matches any amount of characters

We use NOT LIKE to specify that we want it to leave out anything LIKE
AIRBUS

``` r
dbGetQuery(mydb, "SELECT DISTINCT manufacturer FROM planes WHERE manufacturer NOT LIKE 'AIRBUS%'")
##                     manufacturer
## 1                        EMBRAER
## 2                         BOEING
## 3                 BOMBARDIER INC
## 4                         CESSNA
## 5                    JOHN G HESS
## 6           GULFSTREAM AEROSPACE
## 7                       SIKORSKY
## 8                          PIPER
## 9                     AGUSTA SPA
## 10                   PAIR MIKE E
## 11                       DOUGLAS
## 12                         BEECH
## 13                          BELL
## 14            AVIAT AIRCRAFT INC
## 15                  STEWART MACO
## 16                   LEARJET INC
## 17             MCDONNELL DOUGLAS
## 18            CIRRUS DESIGN CORP
## 19            HURLEY JAMES LARRY
## 20                  KILDALL GARY
## 21               LAMBERT RICHARD
## 22                 BARKER JACK L
## 23         AMERICAN AIRCRAFT INC
## 24        ROBINSON HELICOPTER CO
## 25                FRIEDEMANN JON
## 26               LEBLANC GLENN T
## 27                    MARZ BARRY
## 28                   DEHAVILLAND
## 29                      CANADAIR
## 30                  CANADAIR LTD
## 31 MCDONNELL DOUGLAS CORPORATION
## 32 MCDONNELL DOUGLAS AIRCRAFT CO
## 33        AVIONS MARCEL DASSAULT
```

**GROUP BY** groups rows together by common column values.

You can also use **COUNT** with GROUP BY to count occurrences.

Here we can use GROUP BY with COUNT to see how many times a manufacturer
appears in the data

``` r
dbGetQuery(mydb, "SELECT manufacturer, model, COUNT (*) FROM planes GROUP BY manufacturer")
##                     manufacturer              model COUNT (*)
## 1                     AGUSTA SPA              A109E         1
## 2                         AIRBUS           A320-214       336
## 3               AIRBUS INDUSTRIE           A320-214       400
## 4          AMERICAN AIRCRAFT INC          FALCON XP         2
## 5             AVIAT AIRCRAFT INC               A-1B         1
## 6         AVIONS MARCEL DASSAULT MYSTERE FALCON 900         1
## 7                  BARKER JACK L      ZODIAC 601HDS         1
## 8                          BEECH               E-90         2
## 9                           BELL                230         2
## 10                        BOEING            737-824      1630
## 11                BOMBARDIER INC        CL-600-2D24       368
## 12                      CANADAIR        CL-600-2B19         9
## 13                  CANADAIR LTD              CF-5D         1
## 14                        CESSNA                150         9
## 15            CIRRUS DESIGN CORP               SR22         1
## 16                   DEHAVILLAND        OTTER DHC-3         1
## 17                       DOUGLAS             DC-7BF         1
## 18                       EMBRAER          EMB-145XR       299
## 19                FRIEDEMANN JON  VANS AIRCRAFT RV6         1
## 20          GULFSTREAM AEROSPACE               G-IV         2
## 21            HURLEY JAMES LARRY          FALCON-XP         1
## 22                   JOHN G HESS               AT-5         1
## 23                  KILDALL GARY          FALCON-XP         1
## 24               LAMBERT RICHARD          FALCON XP         1
## 25                   LEARJET INC                 60         1
## 26               LEBLANC GLENN T          FALCON XP         1
## 27                    MARZ BARRY          KITFOX IV         1
## 28             MCDONNELL DOUGLAS     DC-9-82(MD-82)       120
## 29 MCDONNELL DOUGLAS AIRCRAFT CO              MD-88       103
## 30 MCDONNELL DOUGLAS CORPORATION              MD-88        14
## 31                   PAIR MIKE E          FALCON XP         1
## 32                         PIPER          PA-31-350         5
## 33        ROBINSON HELICOPTER CO                R66         1
## 34                      SIKORSKY              S-76A         1
## 35                  STEWART MACO          FALCON XP         2
```

You have repetitive data of manufacturer and model thanks to the
tailnumber and year

``` r
dbGetQuery(mydb, "SELECT manufacturer, model, COUNT (*) FROM planes GROUP BY model LIMIT 20")
##    manufacturer      model COUNT (*)
## 1        CESSNA        150         1
## 2        CESSNA       172E         1
## 3        CESSNA       172M         1
## 4        CESSNA       172N         1
## 5          BELL       206B         1
## 6        CESSNA 210-5(205)         1
## 7          BELL        230         1
## 8        CESSNA       310Q         1
## 9        CESSNA       421C         1
## 10       CESSNA        550         1
## 11  LEARJET INC         60         1
## 12        BEECH     65-A90         1
## 13       BOEING    717-200        88
## 14       BOEING    737-301         2
## 15       BOEING    737-317         1
## 16       BOEING    737-3A4         1
## 17       BOEING    737-3G7         2
## 18       BOEING    737-3H4       105
## 19       BOEING    737-3K2         2
## 20       BOEING    737-3L9         2
```

Well, it’s pretty annoying that you’ve used COUNT here but the values
are out of order due to it going alphabetically by manufacturer. That’s
where you can use….

**ORDER BY** , which sorts the rows in the final result set by column(s)

``` r
dbGetQuery(mydb, "SELECT manufacturer, model, 
           COUNT (*) 
           FROM planes 
           GROUP BY model 
           ORDER BY COUNT")
## Error: no such column: COUNT
```

Spoiler alert! this doesn’t work! We need to name the COUNT something.
So let’s rewrite the same code again, but this time use AS!

``` r
dbGetQuery(mydb, 'SELECT manufacturer, model, 
           COUNT (*) AS count 
           FROM planes 
           GROUP BY model 
           ORDER BY count 
           LIMIT 6')
##   manufacturer      model count
## 1       CESSNA        150     1
## 2       CESSNA       172E     1
## 3       CESSNA       172M     1
## 4       CESSNA       172N     1
## 5         BELL       206B     1
## 6       CESSNA 210-5(205)     1
```

It’s doing the same thing as order() and sort(), in which is goes
smallest to largest. So let’s use DESC to see the largest number first
instead.

``` r
dbGetQuery(mydb, "SELECT manufacturer, model, 
           COUNT (*) AS count 
           FROM planes 
           GROUP BY model 
           ORDER BY count DESC
           LIMIT 6")
##                    manufacturer       model count
## 1                        BOEING     737-7H4   361
## 2              AIRBUS INDUSTRIE    A320-232   256
## 3                BOMBARDIER INC CL-600-2B19   171
## 4                BOMBARDIER INC CL-600-2D24   123
## 5                        BOEING     737-824   122
## 6 MCDONNELL DOUGLAS CORPORATION       MD-88   117
```

**FROM** and **JOIN** are important clauses when using SQL, particularly
when you have a few data frames that you want to link, but you
can’t/don’t want to merge all of the data into a single data frame.
*FROM* identifies the tables from which to draw data and how tables
should be joined *JOIN*, well, joins more than one table. There are
different ways of joining, but for now, focus on the clauses we’ve gone
through first with a single data frame.

Note that you can also do a query directly within a SQL chunk, much like
an R chunk from the drop down above. Notice in R Markdown, how we’ve
been inserting R Chunks, which are created by starting with
`{r} followed by our code, and ending with`. Well, we can replace the
{r} with ‘sql connection=’ and then just pop our query in, without
needing to keep using the dbGetQuery() function.

``` sql
SELECT * FROM planes LIMIT 5
```

<div class="knitsql-table">

| tailnum | year | type                    | manufacturer     | model     | engines | seats | speed | engine    |
|:--------|-----:|:------------------------|:-----------------|:----------|--------:|------:|------:|:----------|
| N10156  | 2004 | Fixed wing multi engine | EMBRAER          | EMB-145XR |       2 |    55 |    NA | Turbo-fan |
| N102UW  | 1998 | Fixed wing multi engine | AIRBUS INDUSTRIE | A320-214  |       2 |   182 |    NA | Turbo-fan |
| N103US  | 1999 | Fixed wing multi engine | AIRBUS INDUSTRIE | A320-214  |       2 |   182 |    NA | Turbo-fan |
| N104UW  | 1999 | Fixed wing multi engine | AIRBUS INDUSTRIE | A320-214  |       2 |   182 |    NA | Turbo-fan |
| N10575  | 2002 | Fixed wing multi engine | EMBRAER          | EMB-145LR |       2 |    55 |    NA | Turbo-fan |

5 records

</div>

It makes for much nicer output in RMarkdown. Try changing some of the
chunks earlier with this approach and see how it compares overall.

## Batched queries

If you run a query and the results don’t fit in memory, you can use
`dbSendQuery()`, `dbFetch()` and `dbClearResults()` to retrieve the
results in batches. By default `dbFetch()` will retrieve all available
rows: use `n` to set the maximum number of rows to return.

``` r
rs <- dbSendQuery(mydb, 'SELECT * FROM planes')
while (!dbHasCompleted(rs)) {
    df <- dbFetch(rs, n = 10)
    print(nrow(df))
}
dbClearResult(rs)
```
