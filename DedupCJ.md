The Problem
-----------

Sometimes I want to create a cross join of the same vector but exclude
rows which contain the same information. How do I do this?

As a toy example, let’s say I have a vector of ISO2 country codes.

countryVec &lt;- c(“US”,“MX”,“JP”)

Essentially what I want to create is a dataset with two columns, one
representing country A and another representing country B- but there
shouldn’t be duplicate information for the dyad (or country pair).

So my countryVec output should look something like this:

JP - MX JP - US MX - US

Therefore, I want to exclude the inverse pairs: (MX - JP, US - JP, and
US - MX).

The Solution
------------

``` r
## Figure out which rows are duplicates in a cross join? 

library(data.table)

### First call in your vector of interest
countryVec <- c("US","MX","JP")

getUniqueDat <- function (x = countryVec) {
  ## Create empty data table
  badRowDat <- data.table()

  for (i in 1:length(unique(x))) {
    tempvec <- seq(from = (length(unique(x))*(i-1)) - (i-1), to = 
length(unique(x))*(i-1), by = 1)[-1]
    badRowDat <- rbind(badRowDat, tempvec)
  }

  ## Create Cross Join Table
  fullJoinDat <- CJ(x, x)
  names(fullJoinDat) <- c("V1","V2")
  ## Remove identical rows 
  fullJoinDat <- fullJoinDat[V1 != V2]
  ## Remove bad rows
  uniqueJoinDat <- fullJoinDat[-(badRowDat$x)]
  return(uniqueJoinDat)

}

## Let's test it out
getUniqueDat(x = countryVec)
```

    ##    V1 V2
    ## 1: JP MX
    ## 2: JP US
    ## 3: MX US

Appendix
--------

Adding on to a similar situation where you have a dataset that already
contains all combinations but you want to subset it to where it contains
only one unique pair.

``` r
dat <- data.table(a = c("A","B"),b = c("B","A"),ID = c(1,2))

cols <- c(1,2)
newdat <- dat[,cols, with = FALSE]
for (i in 1:nrow(dat)){
  newdat[i, ] = sort(dat[i,cols, with = FALSE])
}


dat[!duplicated(newdat),]
```

    ##    a b ID
    ## 1: A B  1
