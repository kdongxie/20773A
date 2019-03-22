# Module 4

## Processing Big Data

### Module Overview

- Transforming big data
- Managing big datasets

## Lesson 1 :

### Transforming big data

- When should a transformation be permanent? (언제 영구적으로 변환해야하는가?)
- Using the `rxDataStep` function
- Adding new variables to a dataset
- Subsetting variables in a dataset
- Incorporating <u>third-party</u> packages into a transformation 
  - [third-party: MS사가 제공하는 package가 아닌 제 3자에 의하여 제공되는 Package를 의미한다.]
- Using custom transformation functions
- Reblocking an XDF file (재 봉쇄)
- Demonstration: Using a transformation function to calculate a running total



### When should a transformation be permanent(영구적인)?
- Transformations can be permanent or transient

- Permanent transformations incur I/O and storage costs

- Transient transformations can consume considerable resources *<u>if repeated often</u>*

- Consider the costs and benefits concerning:

  - Reuse (재 사용성)

  - Storage resources (자원의 양)

  - Processing resources

  - Volatility of the data (휘발성의 데이터)

### Using the `rxDataStep` function (Transforming Function)

- Use **`rxDataStep`** to implement transformations in XDF objects

  - XDF data is read as chunks(block size의 data/ 전체 data가 아님) into memory

  - Each chunk is processed and then written back to disk

- Ensure that the chunk size is not too big to fit into memory (chunk size는 memory보다 클수없음)
  - Tune using **`rowsPerRead`** and **`blocksPerRead`**

- Specify transformations in the **transforms** list

  - Avoid transformations that require simultaneous access to all observations in the dataset

  - Add R variables to the transforms closure using the **transformObjects** list

### Adding new variables to a dataset

- Transforms contain statements of the form:

  ##### 				*var*= *expression*

  - If *var* exists in the dataset, it is modified

  - If *var* doesn’t exist, it is created and added to every row

- If you are adding a categorical variable, specify the levels and labels explicitly to avoid inconsistencies between chunks

### Subsetting variables in a dataset

- Use **`varsToDrop`** or **`varsToKeep`** to specify which variables to remove or retain in the result

  (해당되는 Data를 제거 하거나 유지해주는 Function)

- Use **`rowSelection`** to delete entire rows
  - Specify a logical expression that references at least one variable in the dataset (or one of the special ***`.rx`*** variables)

- Limit the scope of the transformation using **`startRow`**, **`numRows`**, **`startBlock`**, and **`numBlocks`**

### Incorporating third-party packages into a transformation
- External packages are not included in the transforms closure

- Specify packages referenced by transformations using the **transformPackages** argument

### Using custom transformation functions
- Use custom functions to perform complex transformations

  - Reference the function using **transformFunc**

  - The function runs once for each chunk

- The function is given a chunk of data in the form of a list of vectors
  - One vector for each variable specified by using **transformVars**

- The function can:

  - Update information in these vectors

  - Add new vectors to the list

  - Delete vectors

- State information is available in the ***.rx*** variables

- Use **.rxGet** and **.rxSet** to pass information between chunks

### Reblocking an XDF file

- Transformations and subsetting can cause blocks in XDF files to become unbalanced

  - Some blocks might contain many fewer rows than others

  - Some blocks might be nearly empty

- Use **`rxDataStep`** with the **`rowsPerRead`** argument to reblock the file

![image](https://user-images.githubusercontent.com/46669551/54501373-45eb8480-4968-11e9-84a0-be1012c5e4db.png)

### Demonstration: 

### Using a transformation function to calculate a running total

- Creating a transformation function

- Testing the transformation function

- Viewing the results of the transformation

```R
# Connect to R Server
remoteLogin(deployr_endpoint = "http://LON-RSVR.ADATUM.COM:12800", session = TRUE, diff = TRUE, commandline = TRUE, username = "admin", password = "Pa55w.rd")

# Examine the dataset
FlightDelayData <- "\\\\LON-RSVR\\Data\\FlightDelayData.xdf"
rxGetInfo(FlightDelayData, getVarInfo = TRUE, getBlockSize = TRUE)
head(RxXdfData(FlightDelayData))

# Create the transformation function
addRunningTotal <- function (dataList) {
  
  # Check to see whether this is a test chunk
  if (.rxIsTestChunk) {
    return(dataList)
  }
  
  # Retrieve the current running total for the distance from the environment
  runningTotal <- as.double(.rxGet("runningTotal"))
  
  # Add a new vector for holding the running totals 
  # and add it to the list of variable values
  runningTotalVarIndex <- length(dataList) + 1
  dataList[[runningTotalVarIndex]] <- rep(as.numeric(NA), times = .rxNumRows)
  names(dataList)[runningTotalVarIndex] <- "RunningTotal"
  
  # Iterate through the values for the Distance variable and accumulate them
  idx <- 1
  for (distance in dataList[[1]]) {
    runningTotal <- runningTotal + distance
    dataList[[runningTotalVarIndex]][idx] <- runningTotal
    idx <- idx + 1
  }
  
  # Save the running total back to the environment, ready for the next chunk
  .rxSet("runningTotal", as.double(runningTotal))
  return(dataList)
}

# Run the transformation
EnhancedFlightDelayData <- "\\\\LON-RSVR\\Data\\EnhancedFlightDelayData.xdf"

rxOptions("reportProgress" = 2)
rxDataStep(inData = FlightDelayData, outFile = EnhancedFlightDelayData, 
           overwrite = TRUE, append = "none",
           transforms = list(ObservationNum = .rxStartRow : (.rxStartRow + .rxNumRows - 1)),
           transformFunc = addRunningTotal,
           transformVars = c("Distance"),
           transformObjects = list(runningTotal = 0),
           numRows = 2000000,
           rowsPerRead = 200000
)

# View the results
rxGetInfo(EnhancedFlightDelayData, getVarInfo = TRUE, getBlockSize = TRUE)
head(RxXdfData(EnhancedFlightDelayData))
tail(RxXdfData(EnhancedFlightDelayData))

# Plot a line to visualize the results using a random sample of the data
rxLinePlot(RunningTotal ~ ObservationNum, EnhancedFlightDelayData, 
           rowSelection = rbinom(.rxNumRows, size = 1, prob = 0.01)
)


```

![image](https://user-images.githubusercontent.com/46669551/54502369-92858e80-496d-11e9-8f1c-733c8b039625.png)



## Lesson 2 :

### Managing big datasets

- Considerations for sorting big data
- Sorting and removing duplicates from big data
- Joining big datasets
- Demonstration: Sorting data with `rxSort`

### Considerations for sorting big data

- Sorting big data is expensive

- Alternatives to sorting:

  - Generate cross-tabulations using **rxCrossTabs** or **rxCube**. 

    - Data is sorted by variable using a single pass through the data

    - You can wrangle the results as a data frame

    - Scale noninteger numeric variables if necessary

  - Use custom transformation functions to calculate aggregate results by group

  - Use ScaleR functions such as **rxQuantiles**, **rxLorenz** and **rxLoc** rather than the more traditional R equivalents
    - Many ScaleR functions don’t require data to be presorted, and can process datasets in a single pass

### Sorting and removing duplicates(사본) from big data

- Use **rxSort** to sort large datasets and data frames

  - Specify sort keys using **sortByVars**

  - Specify sort order using **decreasing**

  - Factors are sorted by level, not name

- Data is sorted in memory if possible, or by using a merge sort
  - Use **varsToKeep**/**varsToDrop** to reduce the size of the dataset and improve performance (can make the difference between sorting in memory or on disk)

- Remove rows with duplicate key values using **removeDupKeys**
  - Record the frequency of removed rows with **dupFreqVar**

### Joining big datasets

- Use **rxMerge** to combine datasets

  - **union** appends one dataset to another vertically

  - **oneToOne** appends one dataset to another horizontally

- **rxMerge** also supports relational-style joins (Data가 2개 들어가야 함)

  - **Inner joins** combine rows where key values in both datasets match

  - Rows in either dataset that have no match in either do not appear

  - **Outer joins** (left, right, and full) can retain rows with no matches in the other dataset by generating **NA** values

    ```R
    # Merging the flight delay data and airport datasets using airport codes
    
    rxMerge(inData1 = sortedFlightDelayData, inData2 = sortedAirportData, outFile = mergedData, matchVars = c("Origin"), type = "inner")
    ```

    

- Joins require the data to be sorted on the join keys

- If joining on factors, ensure that they have the same levels in both datasets

### Demonstration: Sorting data with rxSort
- Tuning a sort

- Removing duplicates from sorted data

```R
# Connect to R Server
remoteLogin(deployr_endpoint = "http://LON-RSVR.ADATUM.COM:12800", session = TRUE, diff = TRUE, commandline = TRUE, username = "admin", password = "Pa55w.rd")

# Examine the data
flightDelayDataSubset = RxXdfData("\\\\LON-RSVR\\Data\\FlightDelayData.xdf")
head(flightDelayDataSubset)                            

# Sort it by Origin (this will take approx 90 seconds)
sortedFlightDelayData = "SortedFlightDelayData.xdf"
sortedData <- rxSort(inData = flightDelayDataSubset, outFile = sortedFlightDelayData, overwrite = TRUE,
                     sortByVars = c("Origin"), decreasing = c(TRUE))

# Note the factor levels for Origin
rxGetVarInfo(sortedData, varsToKeep = c("Origin"))

# View the data. It should be sorted in descending order of Origin
head(sortedData,200)

#Sort the data again. This should be much quicker as it only uses a subset of the variables
sortedData <- rxSort(inData = flightDelayDataSubset, outFile = sortedFlightDelayData, overwrite = TRUE,
                     sortByVars = c("Origin"), decreasing = c(TRUE),
                     varsToKeep = c("Origin", "Dest", "Distance", "CRSDepTime"))

# View the data
head(sortedData,200)

# De-dup routes
sortedData <- rxSort(inData = flightDelayDataSubset, outFile = sortedFlightDelayData, overwrite = TRUE,
       sortByVars = c("Origin"), decreasing = c(TRUE),
       varsToKeep = c("Origin", "Dest", "Distance"),
       removeDupKeys = TRUE, dupFreqVar = "RoutesFrequency"
)

# View the data
head(sortedData,200)

```



### Lab: Processing big data

### Exercise 1: Merging the airport and flight delay datasets

> ###### Task 1: Copy the data to the Shared folder

```R
# Preparation
# Open a command prompt window.
# Run following command
net use \\LON-RSVR\Data
copy E:\Labfiles\Lab04\airportData.xdf \\LON-RSVR\Data
copy E:\Labfiles\Lab04\FlightDelayData.xdf \\LON-RSVR\Data
```

> ###### Task 2: Refactor the data

```R
# Connect to R Server 
# LON-RSVR 서버와 원격으로 접속
remoteLogin(deployr_endpoint = "http://LON-RSVR.ADATUM.COM:12800", session = TRUE, diff = TRUE, commandline = TRUE, username = "admin", password = "Pa55w.rd")
```

```R
# Examine the factor levels in each dataset
airportData = RxXdfData("\\\\LON-RSVR\\Data\\airportData.xdf")
flightDelayData = RxXdfData("\\\\LON-RSVR\\Data\\flightDelayData.xdf")

iataFactor <- rxGetVarInfo(airportData, varsToKeep = c("iata"))
print(iataFactor) 

originFactor <- rxGetVarInfo(flightDelayData, varsToKeep = c("Origin"))
print(originFactor)

destFactor <- rxGetVarInfo(flightDelayData, varsToKeep = c("Dest"))
print(destFactor)
```

###### Creates a new set of factor levels using the levels in the **iata**, **Orgin** and **Dest** variables:

```R
# Create a set of levels for refactoring the datasets
refactorLevels <- unique(c(iataFactor$iata[["levels"]],
                           originFactor$Origin[["levels"]],
                           destFactor$Dest[["levels"]]))
```

```R
# Refactor the datasets
rxOptions(reportProgress = 2)
refactoredAirportDataFile <- "\\\\LON-RSVR\\Data\\RefactoredAirportData.xdf"
refactoredAirportData <- rxFactors(inData = airportData, outFile = refactoredAirportDataFile, overwrite = TRUE,
                                   factorInfo = list(iata = list(newLevels = refactorLevels))
                                  )

refactoredFlightDelayDataFile <- "\\\\LON-RSVR\\Data\\RefactoredFlightDelayData.xdf"
refactoredFlightDelayData <- rxFactors(inData = flightDelayData, outFile = refactoredFlightDelayDataFile, overwrite = TRUE,
                                       factorInfo = list(Origin = list(newLevels = refactorLevels),
                                                         Dest = list(newLevels = refactorLevels))
                                      )
```

```R
# Verify the new factor levels in each dataset. They should all be the same
iataFactor <- rxGetVarInfo(refactoredAirportData, varsToKeep = c("iata"))
print(iataFactor)

originFactor <- rxGetVarInfo(refactoredFlightDelayData, varsToKeep = c("Origin"))
print(originFactor)

destFactor <- rxGetVarInfo(refactoredFlightDelayData, varsToKeep = c("Dest"))
print(destFactor)
```

> ###### Task 3: Merge the datasets

##### renames the `iata` filed in the refactored airport data file to Origin

```r
# Rename the iata variable as Origin - names must match when joining
names(refactoredAirportData)[[1]] <- "Origin" 
```

```R
# reblock the airport data file
reblockedAirportDataFile <- "\\\\LON-RSVR\\Data\\reblockedAirportData.xdf"
reblockedAirportData <- rxDataStep(refactoredAirportData, 
                                   reblockedAirportDataFile, overwrite = TRUE
                                  )
```

##### uses the `rxMerge` function to merge the two XDF files, performing an inner join over the Origin field

```R
# Perform the merge to add the timezone of the Origin airport
mergedFlightDelayDataFile <- "\\\\LON-RSVR\\Data\\MergedFlightDelayData.xdf"
mergedFlightDelayData <- rxMerge(inData1 = refactoredFlightDelayData, inData2 = reblockedAirportData,
                                 outFile = mergedFlightDelayDataFile, overwrite = TRUE,
                                 type = "inner", matchVars = c("Origin"), autoSort = TRUE,
                                 varsToKeep2 = c("timezone", "Origin"),
                                 newVarNames2 = c(timezone = "OriginTimeZone"),
                                 rowsPerOutputBlock = 500000
                                )
```

```r
# Check that the data now contains OriginTimeZone variable
rxGetVarInfo(mergedFlightDelayData)
head(mergedFlightDelayData)
tail(mergedFlightDelayData)
```

### Exercise 2: Transforming departure and arrival dates to UTC

> ###### Task 1: Generate a sample of the flight delay data

##### Creates a new XDF file containing 0.5% of the data in the flight delay data file (약 20000 rows)

```r
# Generate a sample of the data to transform
rxOptions(reportProgress = 1)

flightDelayDataSubsetFile <- "\\\\LON-RSVR\\Data\\flightDelayDataSubset.xdf"
flightDelayDataSubset <- rxDataStep(inData = mergedFlightDelayData,
                                    outFile = flightDelayDataSubsetFile, overwrite = TRUE,
                                    rowSelection = rbinom(.rxNumRows, size = 1, prob = 0.005)
)
```

##### Display the metadata for the XDF file containing the sample data

```R
rxGetInfo(flightDelayDataSubset, getBlockSizes = TRUE)
```

> ###### Task 2: Transform the data

```R
# Date manipulation uses the lubridate package
install.packages("lubridate")
```

##### Implements the standardize Times transformation fuction

```r
# Add departure and arrival times recorded as UTC to the dataset 
standardizeTimes <- function (dataList) {
  
  # Check to see whether this is a test chunk
  if (.rxIsTestChunk) {
    return(dataList)
  }
  
  # Create a new vector for holding the standardized departure time 
  # and add it to the list of variable values
  departureTimeVarIndex <- length(dataList) + 1
  dataList[[departureTimeVarIndex]] <- rep(as.numeric(NA), times = .rxNumRows)
  names(dataList)[departureTimeVarIndex] <- "StandardizedDepartureTime"
  
  # Do the same for standardized arrival time
  arrivalTimeVarIndex <- length(dataList) + 1
  dataList[[arrivalTimeVarIndex]] <- rep(as.numeric(NA), times = .rxNumRows)
  names(dataList)[arrivalTimeVarIndex] <- "StandardizedArrivalTime"
  
  departureYearVarIndex <- 1
  departureMonthVarIndex <- 2
  departureDayVarIndex <- 3
  departureTimeStringVarIndex <- 4
  elapsedTimeVarIndex <- 5
  departureTimezoneVarIndex <- 6
  
  # Iterate through the rows and add the standardized arrival and departure times
  for (i in 1:.rxNumRows) {
    
    # Get the local departure time details
    departureYear <- dataList[[departureYearVarIndex]][i]
    departureMonth <- dataList[[departureMonthVarIndex]][i]
    departureDay <- dataList[[departureDayVarIndex]][i]
    departureHour <- trunc(as.numeric(dataList[[departureTimeStringVarIndex]][i]) / 100)
    departureMinute <- as.numeric(dataList[[departureTimeStringVarIndex]][i]) %% 100
    departureTimeZone <- dataList[[departureTimezoneVarIndex]][i]
    
    # Construct the departure date and time, including timezone
    departureDateTimeString <- paste(departureYear, "-", departureMonth, "-", departureDay, " ", departureHour, ":", departureMinute, sep="")
    departureDateTime <- as.POSIXct(departureDateTimeString, tz = departureTimeZone)
    
    # Convert to UTC and store it
    standardizedDepartureDateTime <- format(departureDateTime, tz="UTC")
    dataList[[departureTimeVarIndex]][i] <- standardizedDepartureDateTime 

    # Calculate the arrival date and time
    # Do this by adding the elapsed time to the departure time
    # The elapsed time is stored as the number of minutes (an integer)
    elapsedTime = dataList[[5]][i]
    standardizedArrivalDateTime <- format(as.POSIXct(standardizedDepartureDateTime) + minutes(elapsedTime))
    
    # Store it
    dataList[[arrivalTimeVarIndex]][i] <- standardizedArrivalDateTime 
  }
  
  # Return the data including the new variables
  return(dataList)
}
```

##### Uses the `rxDataStep` function to perform the transformation

```R
# Transform the sample data
flightDelayDataTimeZonesFile <- "\\\\LON-RSVR\\Data\\flightDelayDataTimezones.xdf"
flightDelayDataTimeZones <- rxDataStep(inData = flightDelayDataSubset,
                                       outFile = flightDelayDataTimeZonesFile, overwrite = TRUE,
                                       transformFunc = standardizeTimes,
                                       transformVars = c("Year", "Month", "DayofMonth", "DepTime", "ActualElapsedTime", "OriginTimeZone"),
                                       transformPackages = c("lubridate")
                            )

# Verify the results
rxGetVarInfo(flightDelayDataTimeZones)
head(flightDelayDataTimeZones)
tail(flightDelayDataTimeZones)
```

### Exercise 3: Calculating cumulative average delays for each route

> ###### Task 1: Sort the data

```r
# Sort the flight delay data by the departure time
rxOptions(reportProgress = 1)
sortedFlightDelayDataFile <- "sortedFlightDelayData.xdf"
sortedFlightDelayData <- rxSort(inData = flightDelayDataTimeZones, 
                                outFile = sortedFlightDelayDataFile, overwrite = TRUE,
                                sortByVars = c("StandardizedDepartureTime")
                               )
```

##### Displays the first and last few lines of the sorted file. Examine the data in the "StandardizedDepartureTime" variable:

```R
# Verify that the data has been sorted
head(sortedFlightDelayData)
tail(sortedFlightDelayData)
```

> ###### Task 2: Calculate the cumulative average delays

##### Implements the "caculateCumulativeaVerageDelays" transformation function

```R
# Add cumulative average flight delays for each route
calculateCumulativeAverageDelays <- function (dataList) {
  
  # Check to see whether this is a test chunk
  if (.rxIsTestChunk) {
    return(dataList)
  }
  
  # Add a new vector for holding the cumulative average delay 
  # and add it to the list of variable values
  cumulativeAverageDelayVarIndex <- length(dataList) + 1
  dataList[[cumulativeAverageDelayVarIndex]] <- rep(as.numeric(NA), times = .rxNumRows)
  names(dataList)[cumulativeAverageDelayVarIndex] <- "CumulativeAverageDelayForRoute"

  originVarIndex <- 1
  destVarIndex <- 2
  delayVarIndex <- 3
  
  # Retrieve the vector containing the cumulative delays recorded so far for each route
  cumulativeDelays <- .rxGet("cumulativeDelays")

  # Retrieve the vecor containing the number of times each route has occurred so far
  cumulativeRouteOccurrences <- .rxGet("cumulativeRouteOccurrences")

  # Iterate through the rows and add the standardized arrival and departure times
  for (i in 1:.rxNumRows) {
  
    # Get the route and delay details
    origin <- dataList[[originVarIndex]][i]
    dest <- dataList[[destVarIndex]][i]
    routeDelay <- dataList[[delayVarIndex]][i]

    # Create a string that identifies the route
    route <- paste(origin, dest, sep = "")

    # Retrieve the current cumulative delay and number of occurrences for each route
    delay <- cumulativeDelays[[route]]
    occurrences <- cumulativeRouteOccurrences[[route]]
    
    # Update the cumulative statistics
    delay <- ifelse(is.null(delay), 0, delay) + routeDelay
    occurrences <- ifelse(is.null(occurrences), 0, occurrences) + 1

    # Work out the new running average delay for the route
    cumulativeAverageDelay <- delay / occurrences
    
    # Store the data and updated stats
    dataList[[cumulativeAverageDelayVarIndex]][i] <- cumulativeAverageDelay 
    cumulativeDelays[[route]] <- delay
    cumulativeRouteOccurrences[[route]] <- occurrences
  }
 
  # Save the lists containing the cumulative data so far
  .rxSet("cumulativeDelays", cumulativeDelays)
  .rxSet("cumulativeRouteOccurrences", cumulativeRouteOccurrences)
  
  # Return the data including the new variable
  return(dataList)
}
```

##### Use `rxDataStep` to run the calculateCumulativeAverageDelays transformation function

```R
# Perform the transformation
flightDelayDataWithAveragesFile <- "\\\\LON-RSVR\\Data\\flightDelayDataWithAverages.xdf"
flightDelayDataWithAverages <- rxDataStep(inData = sortedFlightDelayData,
                                          outFile = flightDelayDataWithAveragesFile, overwrite = TRUE,
                                          transformFunc = calculateCumulativeAverageDelays,
                                          transformVars = c("Origin", "Dest", "Delay"),
                                          transformObjects = list(cumulativeDelays = list(), cumulativeRouteOccurrences = list())
                                         )
```

> ###### Task 3: Verify the results

```R
# Verify the results
rxGetVarInfo(flightDelayDataWithAverages)
head(flightDelayDataWithAverages)
tail(flightDelayDataWithAverages)
```

##### Creates a Scatter and regression plot showing the cumulative average delay for flights from ATL to PHX (Atlanta to Phoenix)

```R
# Plot the delays for the Atlanta/Phoenix route
rxLinePlot(CumulativeAverageDelayForRoute ~ as.POSIXct(StandardizedDepartureTime), type = c("p", "r"),
           flightDelayDataWithAverages,
           rowSelection = (Origin == "ATL") & (Dest == "PHX"),
           yTitle = "Cumulative Average Delay for Route",
           xTitle = "Date"
          )
```



