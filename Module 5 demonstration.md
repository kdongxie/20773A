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

