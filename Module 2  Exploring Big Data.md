#  Module 2 : Exploring Big Data



## Lesson 1: Understanding ScaleR data sources
### ScaleR data sources

- ScaleR functions can access many different types of data, including:

  - Fixed and variable length text files (CSV) 

  - SAS 

  - SPSS 

  - Teradata 

  - XDF 

  - SQL Server and other ODBC databases

    OBDC : 데이터베이스 연결 라이브러리

- ScaleR provides data source objects for each type of data

- Not all data sources are available in every compute context

  적용되지 않는 데이터 타입도 존재함. 확장해서 사용.



## Reading and writing data using ScaleR data sources

- You can manually iterate through data:

  - **rxOpen** : 

  - **rxIsOpen** :

  - **rxReadNext** : 

  - **rxWriteNext** :

  - **rxClose** :

- **rxReadNext** and **rxWriteNext** read and write “chunks” of data

- A chunk can be a data frame or list, depending on the data source



```R
list.files(rxGetOption("sampleDataDir"))
#샘플데이터 파일들의 리스트를 출력
file.path(rxGetOption("sampleDataDir"), "claims.txt")
#샘플데이터 폴더에 있는 `claims.txt`파일의 경로를 출력
df = rxImport(myfilepath)
#myfilepath의 경로에 있는 파일을 읽어와 df에 저장.
```



rxImport() : 데이터를 불러와 Data.Frame 형태로 변환.

RxTextData() : 데이터를 불러와 TextData로 저장



## Working with SQL Server data sources

- You can connect to SQL Server using the **RxSqlServerData** or **RxOdbcData** data sources

  - The **RxOdbcData** data source is generalized

  - The **RxSqlServerData** data source is optimized for SQL Server

- The **RxSqlServerData** data source references a table or query

- ScaleR provides the following helper functions for use with **RxSqlServerData** data sources
  - **rxSqlServerDropTable**

  - **rxSqlServerTableExists**

  - **rxExecuteSQLDDL**



## ScaleR file systems

- ScaleR provides direct access to files stored:

  - Using the computer’s native file system

  - Using HDFS

- To access native files, set the file system to **RxNativeFileSystem** with the **rxSetFileSystem** function

- To access HDFS files, set the file system to **RxHdfsFileSystem**

- You can only use **RxHdfsFileSystem** in a supported compute context, such as **RxHadoopMR**



## Working with Hadoop data sources

- Hadoop stores data using HDFS

- If you are running in a Hadoop compute context you can use the following helper functions:
  - **rxHadoopCopyFromClient** :
  - **rxHadoopCopyFromLocal** :
  - **rxHadoopCopy** :
  - **rxHadoopMove** :
  - **rxHadoopRemove** :
  - **rxHadoopRemoveDir** :
  - **rxHadoopListFiles** :
  - **rxHadoopFileExists** :
  - **rxHadoopCommand** :



## Demonstration: Reading data from SQL Server and HDFS

### Reading data stored in SQL Server

```R
# Connect to SQL Server

sqlConnString <- "Driver=SQL Server;Server=70.12.114.149;Database=VideoShop;uid=sa;pwd=Pa$$w0rd!3585;"

connection <- RxSqlServerData(connectionString = sqlConnString,table = "dbo.VS_CUSTOMER", rowsPerRead = 1000)

# Use R functions to examine the data in the Airports table

head(connection)

rxGetVarInfo(connection)

rxSummary(~., connection)

```



### Reading data stored in HDFS 

```R
# Create a Hadoop compute context
context <- RxHadoopMR(sshUsername = "hadoop", 
                      sshHostname = "192.168.137.101")

rxSetComputeContext(context)

# List the contents of the /user/instructor folder in HDFS
# hadoop fs -ls /user/hadoop
# hdfs namenode -format
# hadoop dfs -ls/

rxHadoopCommand("fs -ls /user/hadoop")

# hadoop fs -put intput(local) output(hdfs, path)
# Source => HDFS, SqlServer, Text(CSV....), SAS, SPSS, ....

# Connect directly to HFDS on the Hadoop VM
hdfsConnection <- RxHdfsFileSystem()
rxSetFileSystem(hdfsConnection)

# Create a data source for the CensusWorkers.xdf file
workerInfo <- RxXdfData("/user/hadoop/CensusWorkers.xdf")

# Perform functions that read from the CensusWorkers.xdf file
head(workerInfo)
rxSummary(~., workerInfo)

```





## Lesson 2: Reading and writing XDF data



### The XDF format

- XDF : Extensible Data Format

- Traditionally, R uses data frames

  - **These cache data in memory** : 재구동시 빠름

  - **The amount of available memory can limit the size of datasets that you can easily process**

- The XDF format breaks files into chunks

  - **Each chunk is loaded into memory, processed, and written back to disk**

  - ScaleR functions can process chunks in parallel

  - Sometimes it might be necessary for a chunk to be processed more than once

- XDF improves scalability, but there are tradeoffs

### Importing data into an XDF object

- Use the **rxImport** command to import data held in other formats into an XDF object

- Use the **numRows** and **rowSelection** arguments to filter rows

- Use the **varsToKeep** and **varsToDrop** arguments to filter columns

- Use the **rowsPerRead** argument to set the chunk size

- Use the **overwrite** and **append** arguments to overwrite or append to the end of an existing XDF file

  

  #### .Xdf 형식은 이진 파일 형식(binary file format)입니다.

  - 임의의 열과 인접 한 행의 효율적인 읽기 블록에 데이터를 저장
  - 변수 이름, 설명 및 데이터 저장소 유형 등과 같은 연관 된 메타 데이터를 포함
  - R (8 개 유형의 정수, 부동 소수점 두 종류 보다 더 다양 한 데이터 저장소 유형 지원
  - 데이터 블록의 행을 데이터 처리를 최적화할 수 있도록 기록
  - 청크 (블록 그룹) 데이터를 처리
  - 최적화 된 블록 i/o 크기 개별 컴퓨터 대역폭에 따라 달라 짐
  - 간단한 업데이트가 시작 하지 않고.xdf 파일의 끝에 대 한 메타 정보를 포함
  - 오버헤드가 걸리는 빈도가 적으며, 병렬처리가 가능

### Controlling data schemas

- Use the **colClasses** argument to override the type of a column
  - Useful for indicating that a column is a factor
- Use the **colInfo** argument to specify the layout of fixed format data in a text file

### Transforming data on import

- Transform data on import with the **transforms** argument

  - A list of transformations

  - Can also create new columns

  - Can reference any variable in the file

  - Internal variables provide limited state information about the import process

- You must reference external objects using **transformObjects**

- Consider implementing complex transformations in a transform function (see Module 4)

### Refactoring variables

- Use **rxFactor** to

  - Convert nonfactor variables into factors

  - Refactor existing factor variables

- Works with XDF objects and data frames

- Supports column level filtering (**varsToKeep** and **varsToDrop**)

- Does not support row level filtering or transformations

### Importing composite data

- The **rxImport** functions can import multiple text files
  - Specify a folder as the data source

- The output can be a composite XDF

  - Specify a folder as the destination

  - Set the **createCompositeSet** argument to TRUE

  - **rxImport** creates a **metadata** folder and a **data** folder holding the files

- To read a composite XDF file, create an **RxXdfData** data source over the folder holding the **metadata** and **data** folders

### Splitting large XDF files

- Use **rxSplit** to divide a large file vertically
  - Useful if you want to process parts of a file separately, or in parallel

- You can split by:

  - Factor, where each file contains the data for a different factor value

  - Number of rows, to create uniformly sized files

- The **rxSplit** function also supports filtering and transformations 

### Combining XDF files

- Use **rxMerge** to combine files

- Different types of merges supported

  - **Union** (files must have the same variables)

  - **OneToOne**

  - Inner and outer merges (relational)

### Reading data in blocks

- You can use the **rxOpen**, **rxReadNext**, and **rxClose** functions to process an XDF file

- The **rxReadNext** function returns a data frame
  - Use the **$** and **[[]]** accessors to read individual variables

- A preferable approach is to use **rxDataStep** (see Module 4)

### Examining and modifying the structure of an XDF object
- Examine XDF metadata using **rxGetInfo** and **rxGetVarInfo**

  - **rxGetInfo** returns the metadata of the XDF file

  - **rxGetVarInfo** returns the metadata describing each variable

- You can modify metadata using the **rxSetInfo** and **rxSetVarInfo** functions

  - You use **rxSetVarInfo** to change the names of variables, and the levels of factors

  - Both functions are very fast because they do not need to modify the data itself, only the metadata



# Connect to SQL Server

#### Use R functions to examine the data in the VS_CUSTOMER table

```R
sqlConnString <- "Driver=SQL Server;Server=70.12.114.147;Database=VideoShop;uid=sa;pwd=PA$$w0rd1234;"
```

SQL Server 에 접속을 진행

```R
connection <- RxSqlServerData(connectionString = sqlConnString, table = "VS_CUSTOMER", rowsPerRead = 1000)
```

RxSqlServerData 함수를 사용하여 정의한 데이터 원본 개체는 데이터가 저장되는 위치를 지정 하고,  SQL Server 의 Data 중 **"VS_CUSTOMER"** 의 *Table*을 변수 connection 으로 새 데이터를 형성

```R
head(connection)
```

```R
rxGetVarInfo(connection)
```

rxGetVarInfo 함수를 사용하여 새 데이터 원본의 변수를 확인할 수 있습니다

```R
rxSummary(~., connection)
```

호출한 Table의 내용을 출력하는 명령어





# SampleDataDir 

# (학습 테이블에서 데이터 로드하기)

[Machine Learning Server Documentation](https://docs.microsoft.com/ko-kr/machine-learning-server/)

https://docs.microsoft.com/ko-kr/machine-learning-server/r-reference/revoscaler/rxoptions

```R
myfilelists <- list.files(rxGetOption("sampleDataDir"))
```

##### sampedatadir의 파일을 List 형태의 파일로 생성



```R
myfilepath <- file.path(rxGetOption("sampleDataDir"), "claims.txt")
```

##### R 변수 myfilepath를 선언하고 샘플 데이터를 포함하는 .txt 파일의 경로를 변수에 할당



```R
df <- rxImport(myfilepath)
str(df)
class(df)
```

##### 파일의 구성 요소 확인



```R
tmp_data <- RxTextData(myfilepath)
```

##### 새 데이터를 저장할 변수를 정의하고 RxTextData 함수를 사용하여 텍스트 데이터 원본을 지정합니다



```R
class(tmp_data)
```

##### 새롭게 정의된 변수의 type을 확인한다.



```R
tmp_data$colNames
```

##### $ 를 이용하여 정의된 tmp_data의 colum을 쉽게 출력 가능하다.



```R
myfilepath2 <- file.path(rxGetOption("sampleDataDir"), "AirlineDemoSmall.csv")
```

##### 새로운 변수 myfilepath2에 파일이 위치한 경로를 지정해 준다.



```R
df2 <- rxImport(myfilepath2)
```

##### rxImport() 를 통하여 df2의 변수에 CSV파일을 넣어준다.



```R
df3 <- read.csv(myfilepath2)
```

##### df3의 변수에 경로에 따라 csv파일을 읽도록 해준다.



```R
df4 <- rxImport(inData = myfilepath2, outFile = "c:/TestData/AirlineDemoSmall.xdf", overwrite = TRUE, getVarInfo = TRUE)
```

##### 논리 값. 경우 `TRUE`, 변수 정보가 반환됩니다.

```R
df4 <- rxImport(inData = myfilepath2, outFile = "c:/Test_Data/AirlineDemoSmall.xdf", overwrite = TRUE, stringsAsFactors = TRUE)
```

##### 함수에 `stringsAsFactors` 인자를 `FALSE`로 설정하면 요인형이 아닌 문자형 그대로 사용할 수 있다.

##### `TURE` 의 경우에는 `DataFrame` 의 형태로 반환된다. 



```R
rxGetInfo(df4, getVarInfo = TRUE)
```

#####  Get size of dataset and num  of observations/variable

![image](https://user-images.githubusercontent.com/46669551/54351352-a5a90d80-4692-11e9-97d2-88cfd38e61b8.png)

```R
View(df2)

list.files(rxGetOption("sampleDataDir"))

df5 <- rxImport(rxGetOption("sampleDataDir"),  "AirlineDemoSmall.xdf")

claims <- file.path(rxGetOption("sampleDataDir"), "claims.txt")

claims

tmpData <- rxImport(inData = claims, outFile = "c:/Test_Data/Claims.xdf", overwrite = TRUE, stringsAsFactors = TRUE)

rxGetInfo(tmpData, getVarInfo = TRUE)
```

### XDF to data.frame

```R
df10 <- rxDataStep(inData = tmpData)

```

#####  rxDataStep 함수를 호출하여 데이터를 삽입합니다.

### Merging Data

##### data.frame No.1

~~~R
acct <- c(0538, 0538, 0538, 0763, 1534)
billee <- c("Rich C", "Rich C", "Rich C", "Tom D", "Kath P")
patient <- c(1, 2, 3, 1, 1)
acctDF <- data.frame(acct = acct, billee = billee, patient = patient)
acctDF
~~~

##### data.frame No.2

```R
acct <- c(0538, 0538, 0538, 0538, 0763, 0763, 0763)
patient <- c(3, 2, 2, 3, 1, 1, 2)
type <- c("OffVisit", "AdultPro", "OffVisit", "2SurfCom", "OffVisit", "AdultPro", "OffVisit")
procedureDF <- data.frame(acct = acct, patient = patient, type = type)
procedureDF
```

### Inner Merge

```R
rxMerge(inData1 = acctDF, inData2 = procedureDF, type = "inner", matchVars = c("acct", "patient"))
```

### Outer Merge

```R
rxMerge(inData1 = acctDF, inData2 = procedureDF, type = "left", matchVars = c("acct", "patient"))
rxMerge(inData1 = acctDF, inData2 = procedureDF, type = "right", matchVars = c("acct", "patient"))
rxMerge(inData1 = acctDF, inData2 = procedureDF, type = "full", matchVars = c("acct", "patient"))
```

### One-to-one merge 

```R
myData1 <- data.frame(x1 = 1:3, y1 = c("a", "b", "c"), z1 = c("x", "y", "z"))
myData1
myData2 <- data.frame(x2 = 101:103, y2 = c("d", "e", "f"), z2 = c("u", "v", "w"))
myData2
```

##### Data.frame 을 2개 생성 해준다.

```R
rxMerge(inData1 = myData1, inData2 = myData2, type = "oneToOne")
```

### Union merge

```R
names(myData2) <- c("x1", "x2", "x3")
```

##### Vector 생성

```R
rxMerge(inData1 = myData1, inData2 = myData2, type = "union")
```

### Using rxMerge with .xdf files

```R
claimsXdf <- file.path(rxGetOption("sampleDataDir"), "claims.xdf")

rxMerge(inData1 = claimsXdf, inData2 = claimsXdf, outFile = "claimsTwice.xdf", type = "union")

censusWorkers <- file.path(rxGetOption("sampleDataDir"), "CensusWorkers.xdf")

rxGetVarInfo(censusWorkers, varsToKeep = "state")

educExp <- data.frame(state = c("Connecticut", "Washington", "Indiana"), EducExp = c(1795.57, 1170.46, 1289.66))

rxMerge(inData1 = censusWorkers, inData2 = educExp, outFile = "censusWorkersEd.xdf", matchVars = "state", overwrite = TRUE)
```



```R
rxGetVarInfo("censusWorkersEd.xdf")
```

##### rxGetVarInfo 함수를 사용하여 새 데이터 원본의 변수를 확인할 수 있습니다
##### RevoScaleR 의 버전을 따라 rxGetVarNames 를 사용할 수도 있습니다.



## Lesson 3: Summarizing data in an XDF object 

### Using base R functions over an XDF object

- You can continue to use many common Base R functions with XDF objects
  - **summary**, **head**, **tail**, **names**, **dim**, **nrow**, **ncol**, and so on

- You do not have to recode existing scripts that use these functions to operate over XDF data

- Internally, these functions call ScaleR functions

  - They do not cast data to data frames

  - They can operate correctly over very large datasets without exhausting resources

- If you are writing new scripts from scratch, consider using the ScaleR functions instead

### Summarizing XDF data with rxSummary

- The **rxSummary** function provides extended summary information

  - Includes summaries of numerics

  - Also includes factors

- Specify a formula to indicate the data to summarize, or ~. for all variables

- Specify which statistics to include with the **summaryStats** argument

- Discount rows with no observations with the **removeZeroCounts** argument

- Can save summaries grouped by a factor variable to different files

### Examining an rxSummary object

- The rxSummary function returns an rxSummary object with the following fields:

  - sDataFrame 

  - categorical 

  - categorical.type

  - formula

  - nobs.valid

  - nobs.missing

![image](https://user-images.githubusercontent.com/46669551/54403287-c5801600-4712-11e9-9fe6-acc3075f1ae7.png)

### Transforming data as it is summarized
- The **rxSummary** function supports filtering and transformations

- These filters and transformations only apply to the summary; they are not saved permanently

- If you need to save filtered and transformed data, use **rxDataStep** (see Module 4)

### Using the dplyrXDF package

- The dplyrXdf package provides dplyr functionality over XDF objects

- Can use:

  - Pipes %>%

  - **select, filter, mutate, group_by, arrange, summarize,** … and many other dplyr functions

- Writes data to temporary files at each stage

  - These files are automatically deleted 

  - Use **persist** to save the changes permanently; often required at the end of a pipeline to avoid losing final results

### Cross-tabulating XDF data

- Generate cross-tabulations using **rxCrossTabs** and **rxCube**
  - Generate contingency tables that you can use to test for independence between variables

- Independent variables must be factors

  - Use the **F** function in a formula to convert a character variable into a factor

  - Use the **as.factor** function to convert a numeric variable into a factor (can generate a large number of values)

- **rxCrossTabs** can output data in **xtabs** format

- **rxCube** generates the same statistics but in a different format

### Testing for independence in cross-tabulated data
- Test for independence in an **rxCrossTabs** or **rxCube** object using:

  - **rxChiSquaredTest**

  - **rxFisherTest**

  - **rxKendallCore**

- These functions can also test variables in an **xtabs** object

### Computing quantiles over XDF data

- The **rxQuantiles** function calculates quantiles for a numeric variable

  - Data is sorted and binned

  - The bins are totaled to give a set of cumulative results

- By default, rxQuantiles creates four bins (0–25%, 25–50%, 50–75%, and 75–100%)
  - You can specify your own set of bins and associated probabilities with the **probs** argument

### Transforming, summarizing, and cross-tabulating XDF data
- Transforming XDF data

- Summarizing XDF data

- Cross-tabulating XDF data



#### Cross-tabulating XDF data

```R
# Flight delay data for the year 2000
setwd("E:\\Demofiles\\Mod02")

csvDataFile = "2000.csv" 

# Examine the raw data
rawData <- rxImport(csvDataFile, numRows = 1000)
rxGetVarInfo(rawData)
View(rawData)
colnames(rawData) #column name을 출력해준다. 


# Create an XDF file that combines the ArrDelay and DepDelay variables, 
# and that selects a random 10% sample from the data

outFileName <- "2000.xdf"
filteredData <- rxImport(csvDataFile, outFile = outFileName, 
                         overwrite = TRUE, append = "none",
                         transforms = list(Delay = ArrDelay + DepDelay),
                         rowSelection = ifelse(rbinom(.rxNumRows, size=1, prob=0.1), TRUE, FALSE))

# Examine the stucture of the XDF data - it should contain a Delay variable
# Note that Origin is a characater
rxGetVarInfo(filteredData)

# Generate a quick summary of the numeric data in the XDF file
rxSummary(~., filteredData)

# Summarize the delay fields
rxSummary(~Delay+ArrDelay+DepDelay, filteredData)

# Examine Delay broken down by origin airport
# Need to change Origin and Dest to factors

refactoredData = "2000Refactored.xdf"
refactoredXdf = RxXdfData(refactoredData)

rxFactors(inData = filteredData, outFile = refactoredXdf, 
          overwrite = TRUE, factorInfo = c("Origin", "Dest"))

rxGetVarInfo(refactoredData)

rxSummary(Delay~Origin, refactoredXdf)

# Generate a crosstab showing the average delay 
# for flights departing from each origin to each destination
rxCrossTabs(Delay ~ Origin:Dest, refactoredXdf, means = TRUE)

# Generate a cube of the same data
rxCube(Delay ~ Origin:Dest, refactoredXdf)

# Omit the routes that don't exist
rxCube(Delay ~ Origin:Dest, refactoredXdf, removeZeroCounts = TRUE)
```



## Summaries Example 

```R
sampleDataDir <- rxGetOption("sampleDataDir")
sampleDataDir #sampleData가 존재하는 File의 path를 저장하여 준다.
list.files(sampleDataDir) #sampledata의 목록을 출력해 준다.
Census <- file.path(sampleDataDir, "CensusWorkers.xdf") #새로운 변수에 .xdf 파일을 넣어준다
Census_info <- rxGetVarInfo(Census) #호출한 Data의 정보를 출력해 준다
names(Census_info) # Column 명을 출력해준다
class(Census_info)
Census_info$age #개별적으로 출력하여 보기 위함
Census_info$sex #개별적으로 출력하여 보기 위함
Census_info$age$low <- 30 #data info에서 Data type과 data의기준값(low/high)이 존재하는데 기준값을 변경해준다.
rxSummary(~F(age), data = Census) #Summary를 진행하는데 F라는 함수를 호출하여 age에 대한 요약을 진행한다.
```



### Module 2: Exploring Bing Data

# Lab : Exploring big data

#### Exercise 1 : Importing and transforming CSV data

> ##### Task 1: import the data for the year 2000

```R
setwd("E:\\Demofiles\\Mod02")
conString <- "Server=LON-SQLR;Database=AirlineData;Trusted_Connection=TRUE"
airportData <- RxSqlServerData(connectionString = conString, table = "Airports")
colClasses <- c(
	"iata" = "character",
	"airport" = "character",
	"city" = "character",
	"state" = "factor",
	"country" = "factor",
	"lat" = "numeric",
	"long" = "numeric")
csvData <- RxTextData(file = "airports.csv", colClasses = colClasses)
rxDataStep(inData = csvData, outFile = airportData, overwrite = TRUE)
```



~~~ R
flightDataSampleCsv <- "2000.csv" 
~~~

##### 1. 2000.csv파일을 호출



``` 
flightDataSample <- rxImport(flightDataSampleCsv, numRows = 10)
```

##### 2. 호출한 CSV파일을 변수에 Import 



``` 
flightDataSample
```

#####  3. 10 row의 값들이 출력 



```R
rxGetVarInfo(flightDataSample) 
```

##### 4. data.frame 의 구조를 보여준다.



```R
flightDataColumns <- c("Year" = "factor", "DayofMonth" = "factor", "DayOfWeek" = "factor", "UniqueCarrier" = "factor", "Origin" = "factor", "Dest" = "factor", "CancellationCode" = "factor")

```

##### 5. flightDataColumns Vector를 생성해 준다.



``` R
flightDataColumns <- c("Year" = "factor", "DayofMonth" = "factor", "DayOfWeek" = "factor", "UniqueCarrier" = "factor", "Origin" = "factor", "Dest" = "factor", "CancellationCode" = "factor")
flightDataXdf <- "2000.xdf"
rxOptions(reportProgress =1)
flightDataSampleXDF <- rxImport(inData = flightDataSampleCsv, outFile = flightDataXdf, overwrite = TRUE, append = "none",colClasses = flightDataColumns)
```

##### 6. CSV data 를 2000.xdf file에 import시켜준다.



```R
rxGetVarInfo(flightDataSampleXDF)
```

##### 7. data.frame의 구조를 출력한다. 

> 기존의 2000.csv파일은 대략 155MB의 크기였으나, 2000.xdf 파일의 크기는 28MB아래로 형성 된다.



> Task 2 : Compare the performance of the data files

```R
system.time(csvDelaySummary <- rxSummary(~., flightDataSampleCsv))
```

##### 1. system.time 의 function을 사용하여 CSV파일 안의 모든 numeric filed를 Summary한다. 

`출력`

```R
> csvDelaySummary
Call:
rxSummary(formula = ~., data = flightDataSampleCsv)

Summary Statistics Results for: ~.
Data: flightDataSampleCsv (RxTextData Data Source)
File name: 2000.csv
Number of valid observations: 1135221 
 
 Name              Mean       StdDev     Min   Max  ValidObs MissingObs
 ActualElapsedTime 128.928075  71.476152    10  817 1094758  40463     
 CRSElapsedTime    129.565957  70.761762     0  684 1135161     60     
 AirTime           106.475551  67.720673     1  646 1094758  40463     
 ArrDelay           10.459154  35.964615 -1298 1423 1094758  40463     
 DepDelay           11.301134  33.632204   -64 1435 1097657  37564     
 Distance          764.559009 573.430133    21 4962 1135221      0     
 TaxiIn              6.055372   4.572883     0  199 1135221      0     
 TaxiOut            15.648307  11.506659     0  368 1135221      0     
> 
```



##### 

```R
system.time(xdfDelaySummary <- rxSummary(~., flightDataSampleXDF))
```

##### 2.  system.time 의 function을 사용하여 XDF파일 안의 모든 numeric filed를 Summary한다. 

- 차이점으로 각 column별 summary 값을 출력해준다.

`출력`

```R
> xdfDelaySummary
Call:
rxSummary(formula = ~., data = flightDataSampleXDF)

Summary Statistics Results for: ~.
Data: flightDataSampleXDF (RxXdfData Data Source)
File name: 2000.xdf
Number of valid observations: 1135221 
 
 Name              Mean       StdDev     Min   Max  ValidObs MissingObs
 ActualElapsedTime 128.928075  71.476152    10  817 1094758  40463     
 CRSElapsedTime    129.565957  70.761762     0  684 1135161     60     
 AirTime           106.475551  67.720673     1  646 1094758  40463     
 ArrDelay           10.459154  35.964615 -1298 1423 1094758  40463     
 DepDelay           11.301134  33.632204   -64 1435 1097657  37564     
 Distance          764.559009 573.430133    21 4962 1135221      0     
 TaxiIn              6.055372   4.572883     0  199 1135221      0     
 TaxiOut            15.648307  11.506659     0  368 1135221      0     

Category Counts for Year
Number of categories: 1
Number of valid observations: 1135221
Number of missing observations: 0

 Year Counts 
 2000 1135221

Category Counts for DayofMonth
Number of categories: 31
Number of valid observations: 1135221
Number of missing observations: 0
+
...
```





```R
system.time(csvCrossTabInfo <- rxCrossTabs(~as.factor(Month):as.factor(Cancelled == 1), flightDataSampleCsv))
```

##### 3. Cross-Tabulation (교차분석) 을통해 월별당 비행기 캔슬정도의 관계를 나타내 준다. 

`출력` 

```R
> csvCrossTabInfo
Call:
rxCrossTabs(formula = ~as.factor(Month):as.factor(Cancelled == 
    1), data = flightDataSampleCsv)

Cross Tabulation Results for: ~as.factor(Month):as.factor(Cancelled == 1)
Data: flightDataSampleCsv (RxTextData Data Source)
File name: 2000.csv
Number of valid observations: 1135221
Number of missing observations: 0 
Statistic: counts 
 
as.factor(Month):as.factor(Cancelled == 1) (counts):
                as.factor.Cancelled....1.
as.factor.Month. FALSE TRUE
              1  88937 4968
              10 94768 2115
              11 91342 2151
              12 90365 5647
              2  85364 3076
              3  94642 2073
              4  90121 2325
              5  92929 3300
              6  90184 3715
              7  93221 3070
              8  95328 2998
              9  90456 2126
> 
```



```R
system.time(xdfCrossTabInfo <- rxCrossTabs(~as.factor(Month):as.factor(Cancelled == 1), flightDataSampleXDF))
```

##### 4. Cross-Tabulation (교차분석) 을통해 월별당 비행기 캔슬정도의 관계를 나타내 준다. 

`출력`

```R
> xdfCrossTabInfo
Call:
rxCrossTabs(formula = ~as.factor(Month):as.factor(Cancelled == 
    1), data = flightDataSampleXDF)

Cross Tabulation Results for: ~as.factor(Month):as.factor(Cancelled == 1)
Data: flightDataSampleXDF (RxXdfData Data Source)
File name: 2000.xdf
Number of valid observations: 1135221
Number of missing observations: 0 
Statistic: counts 
 
as.factor(Month):as.factor(Cancelled == 1) (counts):
                as.factor.Cancelled....1.
as.factor.Month. FALSE TRUE
              1  88937 4968
              10 94768 2115
              11 91342 2151
              12 90365 5647
              2  85364 3076
              3  94642 2073
              4  90121 2325
              5  92929 3300
              6  90184 3715
              7  93221 3070
              8  95328 2998
              9  90456 2126
> 
```



```R
system.time(csvCubeInfo <- rxCube(~as.factor(Month):as.factor(Cancelled), flightDataSampleCsv))
```

##### 5. cube 를 생성하고 월당 캔슬내역을 요약하는 명령

`출력`

```R
> csvCubeInfo
Call:
rxCube(formula = ~as.factor(Month):as.factor(Cancelled), data = flightDataSampleCsv)

Cube Results for: ~as.factor(Month):as.factor(Cancelled)
Data: flightDataSampleCsv
Number of valid observations: 1135221
Number of missing observations: 0 
 
   as.factor(Month) as.factor(Cancelled) Counts
1  1                0                    88937 
2  10               0                    94768 
3  11               0                    91342 
4  12               0                    90365 
5  2                0                    85364 
6  3                0                    94642 
7  4                0                    90121 
8  5                0                    92929 
9  6                0                    90184 
10 7                0                    93221 
11 8                0                    95328 
12 9                0                    90456 
13 1                1                     4968 
14 10               1                     2115 
15 11               1                     2151 
16 12               1                     5647 
17 2                1                     3076 
18 3                1                     2073 
19 4                1                     2325 
20 5                1                     3300 
21 6                1                     3715 
22 7                1                     3070 
23 8                1                     2998 
24 9                1                     2126 
> 
```



```R
system.time(xdfCubeInfo <- rxCube(~as.factor(Month):as.factor(Cancelled), flightDataSampleXDF))
```

##### 6. cube 를 생성하고 월당 캔슬내역을 요약하는 명령

`출력`

```R
> xdfCubeInfo
Call:
rxCube(formula = ~as.factor(Month):as.factor(Cancelled), data = flightDataSampleXDF)

Cube Results for: ~as.factor(Month):as.factor(Cancelled)
File name: 2000.xdf
Number of valid observations: 1135221
Number of missing observations: 0 
 
   as.factor(Month) as.factor(Cancelled) Counts
1  1                0                    88937 
2  10               0                    94768 
3  11               0                    91342 
4  12               0                    90365 
5  2                0                    85364 
6  3                0                    94642 
7  4                0                    90121 
8  5                0                    92929 
9  6                0                    90184 
10 7                0                    93221 
11 8                0                    95328 
12 9                0                    90456 
13 1                1                     4968 
14 10               1                     2115 
15 11               1                     2151 
16 12               1                     5647 
17 2                1                     3076 
18 3                1                     2073 
19 4                1                     2325 
20 5                1                     3300 
21 6                1                     3715 
22 7                1                     3070 
23 8                1                     2998 
24 9                1                     2126 
> 
```



```R
rm(flightDataSample, flightDataSampleXDF, csvDelaySummary, xdfDelaySummary, csvCrossTabInfo, xdfCrossTabInfo, csvCubeInfo, xdfCubeInfo)
```

##### 7. 진행한 변수들을 삭제하는 명령어 rm()

#### Exercise 2: Combing and transforming data

> ##### Task 1 : Copy the data files to R server

##### 1. windows command prompt를 사용하여 아래의 명령어를 진행한다. 

```
net use \\LON-RSVR\Data
```

##### 2. 다른 가상 서버에 공유폴더를 생성하여 공유기능을 활성화 한다. 

```
copy E:\Labfiles\Lab02\200?.csv \\LON-RSVR\Data
```

##### 3. 폴더에 위치한 .csv 파일을 우측의 가상서버로 복사하여 옮겨준다.

`출력`

```
E:\Labfiles\Lab02\2000.csv
E:\Labfiles\Lab02\2001.csv
E:\Labfiles\Lab02\2002.csv
E:\Labfiles\Lab02\2003.csv
E:\Labfiles\Lab02\2004.csv
E:\Labfiles\Lab02\2005.csv
E:\Labfiles\Lab02\2006.csv
E:\Labfiles\Lab02\2007.csv
E:\Labfiles\Lab02\2008.csv
        9 file(s) copied.
```

##### 4. 공유폴더로 옮겨진 파일들을 확인한다. 



> ##### Task 2: Create a remote session

##### 1. 아래의 명령어를 통하여 원격으로 가상서버에 접속을 진행한다. 

```R
remoteLogin("http://LON-RSVR.ADATUM.COM:12800" , session = TRUE, diff = TRUE, commandline = TRUE)
```

![image](https://user-images.githubusercontent.com/46669551/54346852-2236ee80-4689-11e9-88b9-c092cb2604ad.png)



##### 2. 위와 같은 로그인 페이지를 통해 원격으로 로그인을 진행한다. 

```R
> remoteLogin("http://LON-RSVR.ADATUM.COM:12800" , session = TRUE, diff = TRUE, commandline = TRUE)
Diff report between local and remote R sessions...

Local and Remote R versions are the same: R version 3.3.2 (2016-10-31)

These R packages installed on the local machine are not on the remote R instance:

   Missing Packages
1        assertthat
2                BH
3        colorspace
4               DBI
5         dichromat
6            digest
7             dplyr
8           ggplot2
9            gtable
10         labeling
11         lazyeval
12         magrittr
13          munsell
14             plyr
15     RColorBrewer
16             Rcpp
17         reshape2
18           scales
19          stringi
20          stringr
21           tibble


Your REMOTE R session is now active.
Commands:  
        - pause() to switch to local session & leave remote session on hold.
        - resume() to return to remote session.
        - exit to leave (and terminate) remote session.

REMOTE> 
```

##### 3. 위와같은 콘솔창이 생성이 되며 아래의  **"REMOTE>"** 를 통하여 원격 접속의 여부를 확인한다. 



```R
pause()
```

##### 4. pause()의 명령어를 통해 REMOTE session을 잠시 멈출 수 있다. 



```R
putLocalObject(c("flightDataColumns"))
```

##### 5. local 변수인 flightDataColumns 를 REMOTE session에 복사한다.



```R
resume()
```

##### 6. REMOTE session을 다시 활성화 시킨다. 



```R
ls()
```

##### 7. REMOTE session에 flightDataColumns 가 있는지를 확인한다. 

`출력`

```R
REMOTE> ls()
[1] "flightDataColumns"
```



> ##### Task 3: Import the Flight delay data to an XDF file

```R
flightDataSampleXDF <- rxImport(inData = "\\\\LON-RSVR\\Data\\2000.csv", outFile = "\\\\LON-RSVR\\Data\\Sample.xdf", overwrite = TRUE, append = "none", colClasses = flightDataColumns,
                                transforms = list(
                                  Delay = ArrDelay + DepDelay + ifelse(is.na(CarrierDelay), 0, CarrierDelay) + ifelse(is.na(WeatherDelay), 0, WeatherDelay) + ifelse(is.na(NASDelay), 0, NASDelay) + ifelse(is.na(SecurityDelay), 0, SecurityDelay) + ifelse(is.na(LateAircraftDelay), 0, LateAircraftDelay),
                                  MonthName = factor(month.name[as.numeric(Month)], levels=month.name)),
                                rowSelection = (Cancelled == 0),
                                varsToDrop = c("FlightNum", "TailNum", "CancellationCode"),
                                numRows = 1000
)

head(flightDataSampleXDF, 100)

```

##### 1. 원격으로 접속한 가상서버에서 2000.csv 파일을 불러와 상위 1000개의 row만을 가지고 XDF로 만들어 준다. 



```R
head(flightDataSampleXDF, 10)
```

##### 2. transformed한 XDF파일의 상위 10개의 row만을 출력하는 명령어

`출력`

```
REMOTE> head(flightDataSampleXDF, 10)
   .rxRowNames Year Month DayofMonth DayOfWeek DepTime CRSDepTime ArrTime
1            1 2000     1          1         6     842        846    1057
2            2 2000     1          3         1     844        846    1121
3            3 2000     1          9         7    2008       1932    2221
4            4 2000     1         17         1    1943       1932    2148
5            5 2000     1         19         3    1935       1932    2210
6            6 2000     1         20         4    1951       1932    2214
7            7 2000     1          6         4     826        826    1057
8            8 2000     1         10         1     935        826    1150
9            9 2000     1         20         4     826        826    1058
10          10 2000     1         15         6     838        826    1050
   CRSArrTime UniqueCarrier ActualElapsedTime CRSElapsedTime AirTime ArrDelay
1        1101            HP               255            255     244       -4
2        1101            HP               277            255     244       20
3        2153            HP               253            261     237       28
4        2153            HP               245            261     222       -5
5        2153            HP               275            261     243       17
6        2153            HP               263            261     227       21
7        1039            HP               271            253     249       18
8        1039            HP               255            253     239       71
9        1039            HP               272            253     236       19
10       1039            HP               252            253     225       11
   DepDelay Origin Dest Distance TaxiIn TaxiOut Cancelled Diverted CarrierDelay
1        -4    ATL  PHX     1587      3       8         0        0           NA
2        -2    ATL  PHX     1587      6      27         0        0           NA
3        36    ATL  PHX     1587      4      12         0        0           NA
4        11    ATL  PHX     1587      6      17         0        0           NA
5         3    ATL  PHX     1587      7      25         0        0           NA
6        19    ATL  PHX     1587      4      32         0        0           NA
7         0    ATL  PHX     1587      4      18         0        0           NA
8        69    ATL  PHX     1587      4      12         0        0           NA
9         0    ATL  PHX     1587      4      32         0        0           NA
10       12    ATL  PHX     1587      5      22         0        0           NA
   WeatherDelay NASDelay SecurityDelay LateAircraftDelay Delay MonthName
1            NA       NA            NA                NA    -8   January
2            NA       NA            NA                NA    18   January
3            NA       NA            NA                NA    64   January
4            NA       NA            NA                NA     6   January
5            NA       NA            NA                NA    20   January
6            NA       NA            NA                NA    40   January
7            NA       NA            NA                NA    18   January
8            NA       NA            NA                NA   140   January
9            NA       NA            NA                NA    19   January
10           NA       NA            NA                NA    23   January
REMOTE> 
```

##### 3. 파일탐색을 통해 원격으로 접속한 가상서버에서 Sample.xdf 파일을 공유폴더에서 삭제한다.

##### 4. 아래의 명령어를 통해 가상 서버의 공유폴더에 존재하는 모든 CSV 파일을 XDF파일로 변환해준다. 

```R
rxOptions(reportProgress = 1)

delayXdf <- "\\\\LON-RSVR\\Data\\FlightDelayData.xdf"
flightDataCsvFolder <- "\\\\LON-RSVR\\Data"
flightDataXDF <- rxImport(inData = flightDataCsvFolder, outFile = delayXdf, overwrite = TRUE, append = ifelse(file.exists(delayXdf), "rows", "none"), colClasses = flightDataColumns,
                          transforms = list(
                            Delay = ArrDelay + DepDelay + ifelse(is.na(CarrierDelay), 0, CarrierDelay) + ifelse(is.na(WeatherDelay), 0, WeatherDelay) + ifelse(is.na(NASDelay), 0, NASDelay) + ifelse(is.na(SecurityDelay), 0, SecurityDelay) + ifelse(is.na(LateAircraftDelay), 0, LateAircraftDelay),
                            MonthName = factor(month.name[as.numeric(Month)], levels=month.name)),
                          rowSelection = (Cancelled == 0),
                          varsToDrop = c("FlightNum", "TailNum", "CancellationCode"),
                          rowsPerRead = 500000
)
```

##### 5. REMOTE session을 종료한다.

```R
exit
```



#### Exercise 4: Refactoring data and generating summaries

#### ** Remain connected remotely

##### delayFactor breaks the continous variable Delay into a series of factored values

```R
delayFactor <- expression(list(Delay = cut(Delay, breaks = c(0, 1, 30, 60, 120, 180, 181), labels = c("No delay", "Up to 30 mins", "30 mins - 1 hour", "1 hour to 2 hours", "2 hours to 3 hours", "More than 3 hours"))))
delayFactor
```

##### Generate a crosstab that summarizes the delays for flights starting at the Origin airport

```R
originAirportDelays <- rxCrossTabs(formula = ~ Origin:Delay, data = enhancedXdf,
                                   transforms = delayFactor
                       )
print(originAirportDelays)
```

##### Generate a crosstab that summarizes the delays for flights finishing at the Dest airport

~~~R
destAirportDelays <- rxCrossTabs(formula = ~ Dest:Delay, data = enhancedXdf,
                                 transforms = delayFactor
                     )
print(destAirportDelays)
~~~

##### Generate a crosstab that summarizes the delays for flights starting in the OriginState state

```R
originStateDelays <- rxCrossTabs(formula = ~ OriginState:Delay, data = enhancedXdf,
                                 transforms = delayFactor
                     )
print(originStateDelays)
```

##### Generate a crosstab that summarizes the delays for flights finishing in the DestState state

```R


destStateDelays <- rxCrossTabs(formula = ~ DestState:Delay, data = enhancedXdf,
                               transforms = delayFactor
                   )
print(destStateDelays)
```

##### return to the local session

```R
exit
```

##### Install dplyrXdf

```R
install.packages("dplyr")
install.packages("devtools")
devtools::install_github("RevolutionAnalytics/dplyrXdf")
library(dplyr)
library(dplyrXdf)
```

##### Read in a subset of the data from the XDF file containing the data to be summarized (discard all the other columns)

```R
enhancedDelayDataXdf <- "\\\\LON-RSVR\\Data\\EnhancedFlightDelayData.xdf"
essentialData <-RxXdfData(enhancedDelayDataXdf, varsToKeep = c("Delay", "Origin", "Dest", "OriginState", "DestState"))
```

##### Summarize the data using dplyrXdf

```R
originAirportStats <- filter(essentialData, !is.na(Delay)) %>%
  select(Origin, Delay) %>%
  group_by(Origin) %>%
  summarise(mean_delay = mean(Delay), .method = 1) %>% # Use methods 1 or 2 only
  arrange(desc(mean_delay)) %>%
  persist("\\\\LON-RSVR\\Data\\temp.xdf")  # Return a reference to a persistent file. By default, temp files will be deleted

head(originAirportStats, 100)

destAirportStats <- filter(essentialData, !is.na(Delay)) %>%
  select(Dest, Delay) %>%
  group_by(Dest) %>%
  summarise(mean_delay = mean(Delay), .method = 1) %>%
  arrange(desc(mean_delay)) %>%
  persist("\\\\LON-RSVR\\Data\\temp.xdf")

head(destAirportStats, 100)

originStateStats <- filter(essentialData, !is.na(Delay)) %>%
  select(OriginState, Delay) %>%
  group_by(OriginState) %>%
  summarise(mean_delay = mean(Delay), .method = 1) %>%
  arrange(desc(mean_delay)) %>%
  persist("\\\\LON-RSVR\\Data\\temp.xdf")

head(originStateStats, 100)

destStateStats <- filter(essentialData, !is.na(Delay)) %>%
  select(DestState, Delay) %>%
  group_by(DestState) %>%
  summarise(mean_delay = mean(Delay), .method = 1) %>%
  arrange(desc(mean_delay)) %>%
  persist("\\\\LON-RSVR\\Data\\temp.xdf")

head(destStateStats, 100)
```

### HTML 

데이터 + 표현



### XML + XSL

XML : 데이터 

XSL : 표현



### SGML 