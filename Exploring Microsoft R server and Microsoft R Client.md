# Exploring Microsoft R server and Microsoft R Client



## EX 1 - Perform some regular Base R functions on a sample of the data

~~~ 
setwd("E:\\Labfiles\\Lab01")
~~~



### Read the data from the 2000.csv file into a data frame and examine the first 10 rows

~~~
flightDataCsv <- "2000.csv"
~~~

~~~
flightDataSampleDF <- read.csv(flightDataCsv)
~~~

~~~
head(flightDataSampleDF, 10)
~~~



### Add a column that gives the month name based on the month number

~~~
mName <- function(mNum) {
    month.name[mNum]
}
~~~

~~~
flightDataSampleDF$MonthName <- factor(lapply(flightDataSampleDF$Month, mName), levels = month.name)
~~~



### Perform some basic operations on the data. Time how long it takes to generate the summary

~~~
system.time(delaySummary <- summary(flightDataSampleDF))
~~~

~~~
print(delaySummary)
~~~

~~~
print(names(flightDataSampleDF))
~~~

~~~
print(nrow(flightDataSampleDF))
~~~

~~~
print(min(flightDataSampleDF$ArrDelay, na.rm = TRUE))
~~~

~~~
print(max(flightDataSampleDF$ArrDelay, na.rm = TRUE))
~~~

~~~
print(xtabs(~MonthName + as.factor(Cancelled == 1), flightDataSampleDF))
~~~



## EX 2 - Perform similar ScaleR functions on the same data

### Time how long it takes to generate a summary and display the results

~~~
system.time(rxDelaySummary <- rxSummary(~., flightDataSampleDF))
~~~

~~~
print(rxDelaySummary)
~~~




### Perform other ScaleR functions that examine the data

~~~
print(rxGetInfo(flightDataSampleDF))
~~~

~~~
print(rxGetVarInfo(flightDataSampleDF))
~~~

~~~
print(rxQuantile("ArrDelay", flightDataSampleDF))
~~~

~~~
print(rxCrossTabs(~MonthName:as.factor(Cancelled == 1), flightDataSampleDF))
~~~

~~~
print(rxCube(~MonthName:as.factor(Cancelled), flightDataSampleDF))
~~~

~~~
rm(flightDataSampleDF)
~~~



## EX 3 - Perfom operations on a remote server

## Connect to the remote server

~~~
remoteLogin(deployr_endpoint = "http://LON-RSVR.ADATUM.COM:12800", session = TRUE, diff = TRUE, commandline = TRUE, username = "admin", password = "Pa55w.rd")
~~~

~~~
pause()
~~~



## Copy the data file to the remote server

~~~
putLocalFile(c("2000.csv"))
~~~

~~~
resume()
~~~



## Read the data in (this is now running remotely)

~~~
flightDataCsv <- "2000.csv"
~~~

~~~
flightDataSampleDF <- read.csv(flightDataCsv)
~~~




## Add the month name - same as before

~~~
mName <- function(mNum) {
    month.name[mNum]
}
~~~

~~~
flightDataSampleDF$MonthName <- factor(lapply(flightDataSampleDF$Month, mName), levels = month.name)
~~~



## Display the first 10 rows

~~~
head(flightDataSampleDF, 10)
~~~



## Perform the same ScaleR operations as previously

~~~
rxRemoteDelaySummary <- rxSummary(~., flightDataSampleDF)
~~~

~~~
print(rxRemoteDelaySummary)
~~~

~~~
rxRemoteVarInfo <- rxGetVarInfo(flightDataSampleDF)
~~~

~~~
print(rxRemoteVarInfo)
~~~

~~~
rxRemoteQuantileInfo <- rxQuantile("ArrDelay", flightDataSampleDF)
~~~

~~~
print(rxRemoteQuantileInfo)
~~~

~~~
rxRemoteCrossTabInfo <- rxCrossTabs(~MonthName:as.factor(Cancelled == 1), flightDataSampleDF)
~~~

~~~
print(rxRemoteCrossTabInfo)
~~~

~~~~
rxRemoteCubeInfo <- rxCube(~MonthName:as.factor(Cancelled == 1), flightDataSampleDF)
~~~~

~~~
print(rxRemoteCubeInfo)
~~~

~~~
pause()
~~~



## Copy the results back to the local R client and display them

~~~
getRemoteObject(c("rxRemoteDelaySummary", "rxRemoteInfo", "rxRemoteVarInfo", "rxRemoteQuantileInfo", "rxRemoteCrossTabInfo", "rxRemoteCubeInfo"))
~~~

~~~
print(rxRemoteDelaySummary)
~~~

~~~
print(rxRemoteInfo)
~~~

~~~
print(rxRemoteVarInfo)
~~~

~~~
print(rxRemoteQuantileInfo)
~~~

~~~
print(rxRemoteCrossTabInfo)
~~~

~~~
print(rxRemoteCubeInfo)
~~~




## Log out from the remote session

~~~
remoteLogout()
~~~



