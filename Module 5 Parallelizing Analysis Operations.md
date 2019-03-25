# Module 5

## Parallelizing Analysis Operations

###### 1. 병렬구조를 통한 Data 분석 Operations

###### 2. Computer를 여러개로 하여 분산처리하는 방법

###### ** 데이터를 쪼개서 분산시켜 처리하는 방법에 대한 학습을 진행한다

###### Big Data를 처리할 떄에 Resource(CPU, Memory, Network, Disk 등)를 확인 해야 한다.

![1552887657959](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552887657959.png)

### Module Overview

- Using the `RxLocalParallel` compute context with `rxExec`
- Using the `RevoPemaR` package

###### ** embarrassingly parallel : 데이터를 겹치는 부분이 없이 쪼개서 분산처리하는 방법

![image](https://user-images.githubusercontent.com/46669551/54509219-29f9da00-498c-11e9-9875-167cf58c2e62.png)

![image](https://user-images.githubusercontent.com/46669551/54509252-4e55b680-498c-11e9-8527-129ea5cc2bda.png)

## Lesson 1: Using the *RxLocalParallel* compute context with rxExec (병렬 처리를 가동할때 사용)

### Types of big data processing tasks

- High Performance Analytics **(HPA)**

  - Limited by data processing (sharing huge datasets across nodes)

  1. ScaleR functions excel at this

- High Performance Computing **(HPC)**

  - Limited by CPU capacity
  - “Embarrassingly parallel” jobs (중복되는 자료가 없이 분산처리하는 방법)
    - Simulations
    - Bootstrapping
    - Image processing
    - Parallelized **lapply-based** code

- Use **`rxExec`** for these tasks



### Using the RxLocalParallel compute context

- The **rxExec** function is intended for use in parallel environments, such as clusters
- You can use the **RxLocalParallel** compute context for nonclustered environments
  - Enables the **rxExec** function to parallelize jobs locally
  - Use for embarrassingly parallel HPC jobs
  - Use for tasks that don’t need large amounts of data to be transferred
  - Works on local machines and remote servers

##### Creating an RxLocalParallel object

```R
ParallelContext <- RxLocalParallrl()
rxSetComputeContext(parallelContext)
```



### Using the `rxExec` function to perform tasks in parallel

#### : 데이터가 많거나 빠르게 처리하고 싶을 때에 사용하는 함수

- **rxExec** enables you to run arbitrary R code in parallel on distributed computing resources

- You can use **rxExec** in two main ways:

  - To run a function multiple times, collecting the results in a list

  ![image](https://user-images.githubusercontent.com/46669509/54509493-13a04e00-498d-11e9-90de-51849d353414.png)

  - To operate on each object of a list, vector or similar iterable object (like a parallel **lapply**)

![image](https://user-images.githubusercontent.com/46669509/54509501-1a2ec580-498d-11e9-9203-27d5628faf6b.png)

### Using rxExec as a back end for foreach

- **<u>foreach</u>** is a popular way to parallelize base R across cores on a single computer
- The **doRSR** package enables you to use **rxExec** as a back end for the **%dopar%** syntax
  - Simplifies sharing across platforms
  - Enables you to use **<u>%dopar%</u>** on a <u>*remote server or cluster(서버가 많음을 의미함)*</u>

![image](https://user-images.githubusercontent.com/46669509/54509532-329ee000-498d-11e9-84ca-c364dbd9df98.png)

### Demonstration: Using rxExec to perform tasks in parallel

Running a simulation sequentially

Running a simulation using parallel tasks

```R
# Connect to R Server
remoteLogin(deployr_endpoint = "http://LON-RSVR.ADATUM.COM:12800", session = TRUE, diff = TRUE, commandline = TRUE, username = "admin", password = "Pa55w.rd")

# Create dice game simulation function:
# If you roll a 7 or 11 on your initial roll, you win. 
# If you roll 2, 3, or 12, you lose. 
# Roll a 4, 5, 6, 8, 9, or 10, and that number becomes your point and you continue rolling until you either roll your point again in which case you win, or roll a 7, in which case you lose. 
playDice <- function()
{
  result <- NULL
  point <- NULL
  count <- 1
  while (is.null(result))
  {
    roll <- sum(sample(6, 2, replace=TRUE))
    
    if (is.null(point))
    {
      point <- roll
    }
    if (count == 1 && (roll == 7 || roll == 11))
    {
      result <- "Win"
    }
    else if (count == 1 && (roll == 2 || roll == 3 || roll == 12))
    {
      result <- "Loss"
    }
    else if (count > 1 && roll == 7 )
    {
      result <- "Loss"
    }
    else if (count > 1 && point == roll)
    {
      result <- "Win"
    }
    else
    {
      count <- count + 1
    }
  }
  result
}

# Test the function
playDice()

# Play the game 100000 times sequentially - replicate 함수로 10만번 반복
system.time(diceResults <- replicate(100000, playDice()))
table(unlist(diceResults))

# Play the game 100000 times using rxExec - rxExec 함수로 10만번 반복
rxGetComputeContext()
system.time(diceResults <- rxExec(playDice, timesToRun=100000, taskChunkSize = 20000))
table(unlist(diceResults))

# Switch to RxLocalParallel. rxExec will run tasks in parallel
# Elapsed time may be longer on a single-node cluster or for small jobs
# rxExec 함수로 10만번 반복 + 병렬연산. 이 예제에서는 싱글 노드 클러스터를 사용해 시간이 더 걸린다.
rxSetComputeContext(RxLocalParallel())
system.time(diceResults <- rxExec(playDice, timesToRun=100000, taskChunkSize = 20000))
table(unlist(diceResults))

```





### Creating and running non-waiting jobs

#### : 한개의 작업이 수행될 때 다음작업을 수행하지 못하는 것

1. 동기화 방식 : 하나의 작업을 순차적으로 수행하는 것 [ A ----> B ----> C ----> ]
2. 비동기화 방식 : 동시다발적으로 작업을 수행할 수 있다. [A --> (A, B, C) ----> ]

- With long HPC jobs, you might prefer to send the job to the server to run in the background
  - Specify the compute context to be non-waiting with the **wait = FALSE** argument
  - Check back on the progress of the job using **<u>rxGetJobStatus</u>**  (작업의 상태를 확인하는 명령어)
  - Retrieve the results of the completed job with **<u>rxgetJobResults</u>** (작업의 수행결과를 확인하는 명령어)



### Calling HPA functions from rxExec

- You can use **rxExec** to run HPA jobs to different nodes on a cluster 
  - For example, run an independent analysis on each node of a cluster with HPA functions, making use of the available cores on each node
  - Set the **elemType** argument in the **rxExec** node to “nodes”—this will parallelize over the nodes in the cluster
  - Specify the HPA function (such as **rxGlm**, **rxLm**, and so on) using the **FUN** argument 





### Demonstration: Creating waiting and <u>non-waiting jobs</u> in Hadoop

- Uploading data to Hadoop
- Running an analysis as a waiting job
- Running an analysis as a non-waiting job

```R
# Create a Hadoop compute context
context <- RxHadoopMR(sshUsername = "instructor", 
                      sshHostname = "LON-HADOOP")
rxSetComputeContext(context, wait = TRUE)

# Upload the Flight Delay Data
rxHadoopRemove(path = "/user/instructor/FlightDelayData.xdf" )
rxHadoopCopyFromClient(source = "E:\\Demofiles\\Mod05\\FlightDelayData.xdf",
                       hdfsDest = "/user/instructor/")

# Perform an analysis on the flight data
# What are the most popular routes?
rxOptions(reportProgress = 1)
hdfsConnection <- RxHdfsFileSystem()
rxSetFileSystem(hdfsConnection)
flightDelayDataFile <- "/user/instructor/FlightDelayData.xdf"
flightDelayData <- RxXdfData(flightDelayDataFile)
routes <- rxSummary(~Origin:Dest, flightDelayData) # Summary, 기술통계

# Try and sort the data in the Hadoop context.
# Note the error message
sortedData <- rxSort(as.data.frame(routes$categorical[[1]]), #Sorting
                     sortByVars = c("Counts"), decreasing = TRUE)

# Use rxExec to run rxSort in a distributed context
sortedData <- rxExec(FUN = rxSort, 
                     inData = as.data.frame(routes$categorical[[1]]),
                     sortByVars = c("Counts"), 
                     decreasing = TRUE
                    )

head(sortedData)

# Create a non-waiting Hadoop compute context
# Hadoop 연결
context <- RxHadoopMR(sshUsername = "instructor", 
                      sshHostname = "LON-HADOOP",
                      wait = FALSE)
rxSetComputeContext(context)

# Perform the analysis again
job <- rxSummary(~Origin:Dest, flightDelayData)

# Check the status of the job
rxGetJobStatus(job) # 작업 상황 확인

# When the job has finished, get the results
# This will fail if the job is still running
routes <- rxGetJobResults(job) # 작업 결과 확인
print(routes$categorical[[1]])

# Run the job again
job <- rxSummary(~Origin:Dest, flightDelayData) # Summary & 기술통계

# Check the status of the job
rxGetJobStatus(job)

# Cancel the job
rxCancelJob(job)

# Check the status of the job
rxGetJobStatus(job)

# Return to the local compute context
rxSetComputeContext(RxLocalSeq())

```



## Lesson 2: Using the RevoPemaR package
- The RevoPemaR framework
- The structure of a PEMA class
- Creating a PEMA class
- Using a PEMA object to perform an analysis
- Debugging a PEMA class
- Demonstration: Creating and running a PEMA object

### The RevoPemaR framework

- Use the RevoPemaR framework to distribute processing for big data

  - A <u>master node</u> divides data into chunks and divides processing amongst <u>client nodes</u> in a cluster

    ![image](https://user-images.githubusercontent.com/46669551/54572748-5c5b1400-4a2c-11e9-9287-c1b315e18293.png)

  - The master node aggregates the results when clients have finished

  - Similar in concept to Map/Reduce

- PEMA classes are based on Reference (Ref) classes, an OOP system of classes in R that:

  - Work well with mutable objects 

  - Have methods that belong to objects rather than functions, unlike the standard S3 and S4 classes in R

### The structure of a PEMA class

- Create a class generator to construct a PEMA object

- You must specify:

  - The class name

  - The superclass from which your class will inherit methods and fields (**PemaBaseClass** or a child class of this class)

  - The fields, or class variables; ref classes are mutable, so the field contents can change 

  - The methods, or class functions, that perform the analysis itself

- Use the **setPemaClass** constructor function to create the PEMA class generator

### Creating a PEMA class

- When you write a new PEMA class generator, you should provide the following methods:

  - **Initialize**: sets the initial field values 

  - **processData**: controls how each chunk is processed and aggregates results for all chunks processed by a single node

  - **updateResults**: collects the results from the other nodes together in one place; this method might not get called if there is only one node **# 결과를 Load하여 바꾸어 주는 기능**

  - **processResults**: calculates the final result

  - **getVarsToUse**: specifies the names of the variables to use; this method is optional, but can improve performance

### Using a PEMA object to perform an analysis
- Instantiate a new PEMA object using the class generator

![image](https://user-images.githubusercontent.com/46669551/54573384-6d595480-4a2f-11e9-9734-77c87e263d4b.png)

- Use **pemaCompute** to run the analysis

![image](https://user-images.githubusercontent.com/46669551/54573397-7ba77080-4a2f-11e9-8a82-d96ddcc6df35.png)

- The **pemaCompute** function handles the distribution and coordination of PEMA objects amongst nodes



### Debugging a PEMA class

- Display debugging information using **outputTrace**

  - Set the **traceLevel** field in the PEMA object

  - Specify a value for **outputTraceLevel** when calling **outputTrace**

  - Trace messages will be displayed if **traceLevel** is greater than or equal to **outputTraceLevel** 

  - Useful for implementing different levels of debug message, such as “information” and “warning”

### Demonstration: Creating and running a PEMA object
- Creating a PEMA class generator

- Testing the PemaMean class

- Using the PemaMean class to analyze flight delay data

```R
# Run Locally

# Create the PemaMean class
library(RevoPemaR)

PemaMean <- setPemaClass(
  Class = "PemaMean",
  contains = "PemaBaseClass",
  fields = list(
    sum = "numeric",
    totalObs = "numeric",
    totalValidObs = "numeric",
    mean = "numeric",
    varName = "character"
  ),
  methods = list(
    initialize = function(varName = "", ...)  {
      'sum, totalValidObs, and mean are all initialized to 0'
      callSuper(...)
      usingMethods(.pemaMethods)
      varName <<- varName
      sum <<- 0
      totalObs <<- 0
      totalValidObs <<- 0
      mean <<- 0
      traceLevel <<- as.integer(1)
    },
    
    processData = function(dataList) {
      'Updates the sum and total observations from the current chunk of data.'
      sum <<- sum + sum(as.numeric(dataList[[varName]]),
                        na.rm = TRUE)
      totalObs <<- totalObs + length(dataList[[varName]])
      totalValidObs <<- totalValidObs +
        sum(!is.na(dataList[[varName]]))
      invisible(NULL)
    },
    
    updateResults = function(pemaMeanObj) {
      'Updates sum and total observations from another PemaMean object.'
      sum <<- sum + pemaMeanObj$sum
      totalObs <<- totalObs + pemaMeanObj$totalObs
      totalValidObs <<- totalValidObs + pemaMeanObj$totalValidObs
      
      invisible(NULL)
    },
    
    processResults = function() {
      'Returns the sum divided by the totalValidObs.'
      if (totalValidObs > 0)
      {
        mean <<- sum/totalValidObs
      }
      else
      {
        mean <<- as.numeric(NA)
      }
      
      outputTrace("outputting mean:\n", outTraceLevel = 1)
      
      return( mean )
    }
  )
)

# Instantiate a PemaMean object
meanPemaObj <- PemaMean()

# Connect to R Server
remoteLogin(deployr_endpoint = "http://LON-RSVR.ADATUM.COM:12800", session = TRUE, diff = TRUE, commandline = TRUE, username = "admin", password = "Pa55w.rd")
library(RevoPemaR)

# Copy the PemaMean object to the R server environment for testing
pause()
putLocalObject("meanPemaObj")
resume()

# Create some test data
set.seed(12345)
testData <- data.frame(x = rnorm(1000))
testData
# Run the analysis
result <- pemaCompute(pemaObj = meanPemaObj, data = testData, varName = "x")
print(result)

# Examine the internal fields of the PemaMean object
print(meanPemaObj$sum)
print(meanPemaObj$mean)
print(meanPemaObj$totalValidObs)

# Create some more test data
set.seed(54321)
testData <- data.frame(x = rnorm(1000))

# Run the analysis again, but include the previous results
result <- pemaCompute(pemaObj = meanPemaObj, data = testData, varName = "x", initPema = FALSE)
print(result)

# Examine the internal fields of the PemaMean object again
print(meanPemaObj$sum)
print(meanPemaObj$mean)
print(meanPemaObj$totalValidObs)

# Perform an analysis against the flight delay data
# - Calculate the mean delay time
rxSetFileSystem(RxNativeFileSystem()) # Just in case the session is still connected to HDFS
pause()
setwd("E:\\Demofiles\\Mod05")
putLocalFile("FlightDelayDataSubset.xdf")
resume()
flightDelayData = RxXdfData("FlightDelayDataSubset.xdf")
result <- pemaCompute(pemaObj = meanPemaObj, data = flightDelayData, varName = "Delay")
print(result)
```

### Lab: Parallelizing analysis operations
##### Exercise 1: Capturing flight delay times and frequencies

> ###### Task 1 : Create a shared folder

> ###### Task 2 : Copy the data

1. Log in to the client server(LON-DEV)
2. Open a command prompt window
3. run the following command

```R
net use \\LON-RSVR\Data
copy E:\Labfiles\Lab05\FlightDelayData.xdf \\LON-RSVR\Data
```

> ###### Task 3 : Create the class generator for the PemaFlightDelays class

1. RStudio 또는 Visual Studio를 이용하여 새로운 R프로젝트를 실행한다.
2. `dplyr` Package를 설치한다.

```R
install.packages("dplyr")
```

3. `dplyr`의 실행을 선언해 주고, `RevoPemaR` 를 라이브러리에 추가해준다.

```R
library(dplyr)
library(RevoPemaR)
```

4. 아래의 Code는 `PemaFlightDelays`의 Class를 정의해주는 generator이다. [실행시키지 않는다]

```R
PemaFlightDelays <- setPemaClass(Class = "PemaFlightDelays", contains = "PemaBaseClass",)
```

5. **contains** line 다음에  아래의 코드를 작성하여 Class의 field를 선언해준다. [실행시키지 않는다]

```R
fields = list( 
	totalFlights = "numeric",
	totalDelays = "numeric",
	origin = "character",
	dest = "character",
	airline = "character",
	delayTimes = "vector",
	result = "list"
),
```

6. **fields** list의 뒤에 **initialize** 라는 method를 정의해준다. [실행시키지 않는다]

```R
methods = list(
    initialize = function(originCode = "", destinationCode = "", 
                          airlineCode = "", ...)  {
      'initialize fields'
      print("Hello")
      callSuper(...)
      usingMethods(.pemaMethods)
      totalFlights <<- 0
      totalDelays <<- 0
      delayTimes <<- vector(mode="numeric", length=0)
      origin <<- originCode
      dest <<- destinationCode
      airline <<- airlineCode
    },
```

7. **initialize** method 뒤에 **processData** method를 정의해준다.[실행시키지 않는다]

```R
processData = function(dataList) {
      'Generates a vector of delay times for specified variables in the current chunk of data.'
      print("Hello2")
      data <- as.data.frame(dataList)

      # origin의 값이 없다면, default값을 dataset의 첫번째 값으로 입력해준다.
      if (origin == "") {
        origin <<- as.character(as.character(data$Origin[1]))
      }

      # destination의 값이 없다면, default값을 dataset의 첫번째 값으로 입력해준다.
      if (dest == "") {
        dest <<- as.character(as.character(data$Dest[1]))
      }
      
      # airline의 값이 없다면, default값을 dataset의 첫번째 값으로 입력해준다.
      if (airline == "") {
        airline <<- as.character(as.character(data$UniqueCarrier[1]))
      }
      
      # dplyr함수를 사용하여 orgin, destination 그리고 airline의 dataset을 filter한다.
      # flights의 수를 갱신(update)하여 준다.
      # Delay 변수를 선택해준다.
      # 연착된 항공편(delayed flights)만을 포함하여 출력해준다.
      data %>%
        filter(Origin == origin, Dest == dest, UniqueCarrier == airline) %T>%
        {totalFlights <<- totalFlights + length(.$Origin)} %>%
        select(ifelse(is.na(Delay), 0, Delay)) %>%
        filter(Delay > 0) ->
        temp

      # 새로운 vector delayTimes에 결과값을 저장한다.
      delayTimes <<- c(delayTimes, as.vector(temp[,1]))
      totalDelays <<- length(delayTimes)
      
      invisible(NULL)
    },
```

8. **processData** method 뒤에 **updateResult** method를 정의해준다.[실행시키지 않는다]

```R
updateResults = function(pemaFlightDelaysObj) {
      'Updates total observations and delayTimes vector from another PemaFlightDelays object object.'

      # totalFlights 와 totalDelays fields 를 갱신(update)한다.
      totalFlights <<- totalFlights + pemaFlightDelaysObj$totalFlights
      totalDelays <<- totalDelays + pemaFlightDelaysObj$totalDelays
      
      # 7번과정에서 생선한 delayTimes vector에 delay data를 추가시킨다.
      delayTimes <<- c(delayTimes, pemaFlightDelaysObj$delayTimes)

      invisible(NULL)
    },
```

9. **updateResult** method 뒤에 **processResults** method를 정의해준다.[실행시키지 않는다] 

```R
processResults = function() {
      'Generates a list containing the results:'
      '  The first element is the number of flights made by the airline'
      '  The second element is the number of delayed flights'
      '  The third element is the list of delay times'
      
      results <<- list("NumberOfFlights" = totalFlights,
                       "NumberOfDelays" = totalDelays, 
                       "DelayTimes" = delayTimes)
      
      return(results)
    }
  )
```

10. **PemaFlightDelays**로 선언된 Class의 Generating의 전체 코드를 실행시킨다. 

```R
PemaFlightDelays <- setPemaClass(
  Class = "PemaFlightDelays",
  contains = "PemaBaseClass",
  fields = list(
    totalFlights = "numeric",
    totalDelays = "numeric",
    origin = "character",
    dest = "character",
    airline = "character",
    delayTimes = "vector",
    results = "list"
  ),
  methods = list(
    initialize = function(originCode = "", destinationCode = "", 
                          airlineCode = "", ...)  {
      'initialize fields'
      print("Hello")
      callSuper(...)
      usingMethods(.pemaMethods)
      totalFlights <<- 0
      totalDelays <<- 0
      delayTimes <<- vector(mode="numeric", length=0)
      origin <<- originCode
      dest <<- destinationCode
      airline <<- airlineCode
    },
    
    processData = function(dataList) {
      'Generates a vector of delay times for specified variables in the current chunk of data.'
      print("Hello2")
      data <- as.data.frame(dataList)

      # If no origin was specified, default to the first value in the dataset
      if (origin == "") {
        origin <<- as.character(as.character(data$Origin[1]))
      }

      # If no destination was specified, default to the first value in the dataset
      if (dest == "") {
        dest <<- as.character(as.character(data$Dest[1]))
      }
      
      # If no airline was specified, default to the first value in the dataset
      if (airline == "") {
        airline <<- as.character(as.character(data$UniqueCarrier[1]))
      }
      
      # Use dplyr to filter by origin, dest, and airline
      # update the number of flights
      # select the Delay variable
      # only include delayed flights in the results
      data %>%
        filter(Origin == origin, Dest == dest, UniqueCarrier == airline) %T>%
        {totalFlights <<- totalFlights + length(.$Origin)} %>%
        select(ifelse(is.na(Delay), 0, Delay)) %>%
        filter(Delay > 0) ->
        temp

      # Store the result in the delayTimes vector
      delayTimes <<- c(delayTimes, as.vector(temp[,1]))
      totalDelays <<- length(delayTimes)
      
      invisible(NULL)
    },
    
    updateResults = function(pemaFlightDelaysObj) {
      'Updates total observations and delayTimes vector from another PemaFlightDelays object object.'

      # Update the totalFlights and totalDelays fields
      totalFlights <<- totalFlights + pemaFlightDelaysObj$totalFlights
      totalDelays <<- totalDelays + pemaFlightDelaysObj$totalDelays
      
      # Append the delay data to the delayTimes vector
      delayTimes <<- c(delayTimes, pemaFlightDelaysObj$delayTimes)

      invisible(NULL)
    },
    
    processResults = function() {
      'Generates a list containing the results:'
      '  The first element is the number of flights made by the airline'
      '  The second element is the number of delayed flights'
      '  The third element is the list of delay times'
      
      results <<- list("NumberOfFlights" = totalFlights,
                       "NumberOfDelays" = totalDelays, 
                       "DelayTimes" = delayTimes)
      
      return(results)
    }
  )
)
```

> ###### Task 4 : Test the PemaFlightDelays class using a data frame

1. Task 3를 통해 선언한 PemaFlightDalays Class를 변수 pemaFlightDelaysObj로 만들어준다.

```R
pemaFlightDelaysObj <- PemaFlightDelays()
```

2. R서버가 위치한 가상서버에 원격으로 접속을 수행한다.

```R
remoteLogin(deployr_endpoint = "http://LON-RSVR.ADATUM.COM:12800", session = TRUE, diff = TRUE, commandline = TRUE, username = "admin", password = "Pa55w.rd")
```

3. 원격접속을 일시정지 한다.

```R
pause()
```

4. R client (Local computer)에서 **pemaFlightDelaysObj** 변수를 복사한다.

```R
putLocalObject("pemaFlightDelaysObj")
```

5. 원격접속을 재 가동시켜준다.

```R
resume()
```

6. R Server에 **dplyr를** 설치하고 에 **dplyr** 와 **RevoPemaR** 를 라이브러리에 추가해준다.

```R
install.packages("dplyr")
library(dplyr)
library(RevoPemaR)
```

7. R Server 인 LON-RSVR에 복사해준 FlightDelayData.xdf로부터 상위 50000개의 값을 불러와 새로운 data.frame의 형태로 변환하여준다.

```R
flightDelayDataFile <- ("\\\\LON-RSVR\\Data\\FlightDelayData.xdf")
flightDelayData <- RxXdfData(flightDelayDataFile)
testData <- rxDataStep(flightDelayData, numRows = 50000)
```

8. pemaCompute 함수를 이용하여 **PemaFlightDelaysObj** 에서 "**ABE**" 에서 출발하여 "**PIT**" 까지 운행하는 "**US**"항공만을 결과값으로 가져온다.

```R
result <- pemaCompute(pemaObj = pemaFlightDelaysObj, data = testData, originCode = "ABE", destinationCode = "PIT", airlineCode = "US")
print(result)
```

`print(result)`

```R
> print(result)
$NumberOfFlights
[1] 755

$NumberOfDelays
[1] 188

$DelayTimes
  [1]   3  10   3  16  54   2  61  65  54  12  18  23  92  16 153   7  18   2  21  61   2   1   1   4  40  67   1  82   6   3 112 298  39  21
 [35]  13   2   1  12   2 131 474  85  27 352   9   2  49  24  18  60  43  28 126 109  40  39  53  34 120   3 274  73  57   3  83  27  58  53
 [69]  15   8  58  61   1 117  34  32   9  19  66  44   2  82  17  21   9 103   2  45   4  64   3  48  52  17   5  11   7   1  18  23  43  29
[103]   7  46  22  71  16  18   9  62  27 120  10  12  11   6  10   4  50   4   1   6   1 129   3   9 185   5  11  17  19 171   2  81   3  17
[137]   1  33  21   2  45   8  27  29  42  25  40   5   1  15   1  59   4  10   6  81  13  45  37   6   9   1   7   1   2   2   2   5 109   3
[171]  15   7  25  58  17  45 289   5   7   7  38  89   3  34  12  15 129  19

> 
```



9. pemaFlightDelaysObj 의 값들을 출력해본다.

```R
print(pemaFlightDelaysObj$delayTimes)
print(pemaFlightDelaysObj$totalDelays)
print(pemaFlightDelaysObj$totalFlights)
print(pemaFlightDelaysObj$origin)
print(pemaFlightDelaysObj$dest)
print(pemaFlightDelaysObj$airline)
```



> ###### Task 5: USe the PemaFlightDelays class to analyze XDF data

1. **flightDelayData** 파일을 subset을 통해 **FlightDelayDataSubset.xdf** 파일의 형태로 쪼개어 저장한다.

```R
flightDelayDataSubsetFile <- ("\\\\LON-RSVR\\Data\\FlightDelayDataSubset.xdf")
testData2 <- rxDataStep(flightDelayData, flightDelayDataSubsetFile, 
                       overwrite = TRUE, numRows = 50000, rowsPerRead = 5000)
```

2. pemaCompute 함수를 이용하여 **PemaFlightDelaysObj** 에서 "**ABE**" 에서 출발하여 "**PIT**" 까지 운행하는 "**US**"항공만을 결과값으로 가져온다.

```R
result <- pemaCompute(pemaObj = pemaFlightDelaysObj, data = testData2, originCode = "ABE", destinationCode = "PIT", airlineCode = "US")
```

3. 출력값을 확인한다. (755 flights, with 188 delayed)

```R
> print(result)
$NumberOfFlights
[1] 755

$NumberOfDelays
[1] 188

$DelayTimes
  [1]   3  10   3  16  54   2  61  65  54  12  18  23  92  16 153   7  18   2  21  61   2   1   1   4  40  67   1  82   6   3 112 298  39  21
 [35]  13   2   1  12   2 131 474  85  27 352   9   2  49  24  18  60  43  28 126 109  40  39  53  34 120   3 274  73  57   3  83  27  58  53
 [69]  15   8  58  61   1 117  34  32   9  19  66  44   2  82  17  21   9 103   2  45   4  64   3  48  52  17   5  11   7   1  18  23  43  29
[103]   7  46  22  71  16  18   9  62  27 120  10  12  11   6  10   4  50   4   1   6   1 129   3   9 185   5  11  17  19 171   2  81   3  17
[137]   1  33  21   2  45   8  27  29  42  25  40   5   1  15   1  59   4  10   6  81  13  45  37   6   9   1   7   1   2   2   2   5 109   3
[171]  15   7  25  58  17  45 289   5   7   7  38  89   3  34  12  15 129  19

> 
```

4. 같은 XDF파일을 이용하여 다른 항공편에 대한 내용을 분석한다.

```R
result <- pemaCompute(pemaObj = pemaFlightDelaysObj, data = flightDelayData, originCode = "LAX", destinationCode = "JFK", airlineCode = "DL")
print(result)
```

