# Module 1 : R server and R client



## Lesson 1: Introduction to Microsoft R Server



### What is Microsoft R Server?

* Microsoft Open R

* ScaleR package for transparent parallel and distributed computing

- MicrosoftML package for distributed machine learning

  분산머신러닝 패키지 **MicrosoftML**

- Platform specific components

- Operationalization functions for deploying to remote servers 



### R Server environments

* Servers (single machines with multiple processors)

   + Windows or Linux

* Clusters (multiple machines each with multiple processors)

  많은 프로세서들을 가진 각각의 머신들을 사용

  * Hadoop
  * Spark

* Database services
   * SQL Server
   * Teradata



### Using a remote R server

* Remote execution (the **mrsdeploy** package) enables you to:

   + Log in and out of an R Server

  * Execute interactive code remotely

  * Reconcile any differences between the local and remote environments

  * Run R scripts 

    mrsdeploy 패키지를 이용해서 다양한 인터페이스를 제공

* You first have to operationalize your server





## Lesson 2: Using Microsoft R Client



### How R Client compares to R Server 

#### 서버와 비교한 클라이언트의 기능

- Freely available and based on Microsoft Open R

- 100 percent compatible with base R

  - Can install any open source R packages

- Has all **ScaleR** functions

  - Limited to two threads

    두 쓰레드로 제약....

- Limited to in-memory computation

  메모리 제약.

- Chunking data is not available

  조각난 데이터는 이용불가!.

- Can interact with R Server

  자체적으로 처리하거나 서버에 접속해서 처리



### Using R Client

- Standard methods:

  - Built in R GUI

  - Rterm (terminal access)

    터미널로 접속

  - Rscript (running batch files)

    배치파일로 접속

- Integrated development environments (IDEs): 개발도구

  - R Tools for Visual Studio (RTVS—Visual Studio Plugin)

    비주얼 스튜디오.

  - RStudio (third-party/open source standalone IDE)

    R 스튜디오.

  - 

### Connecting to a remote R server

- Configuring operationalization on the server
  - The R Server admin tool

- Logging in
  - Use the **mrsdeploy** function
    - 명령어
      1. remoteLogin()
      2. remoteLoginAAD()
      3. pause()
      4. resume()
      5. remmoteExecute()
      6. remoteLogout()
      7. exit

- Authentication options:

  - Username/password

  - Windows Active Directory

  - Azure Active Directory

- Can pause/resume a remote session



### Transferring objects between sessions

두세션 간 오브젝트 전송.



- Get/put objects between the local and remote session

- Put a copy of the local workspace

- Get/put files

  - Use out-of-band transfer for large files

  - Store files in a shared location that is accessible to all servers through the same path

- Warning: beware of transferring large datasets

- 명령어

  - getRemoteObject()
  - putLocalObject()
  - get RemoteFile()
  - putLocalFile()
  - listRemoteFiles()
  - deleteRemoteFile()

  

### Demonstration: Using R Client with Visual Studio and RStudio

1. 비주얼 스튜디오 실행
2. 비주얼 스튜디오에서 R Tools 메뉴, Data Science Settings 메뉴를 클릭
3. Microsoft Visual Studio 메세지 박스에서 Yes 클릭
4. R interractive 패널에서 mtcars 입력 후 엔터. mtcars 샘플데이터를 가져온다.
5. mpg<-mtcars[["mpg"]] 입력후 엔터. mpg 변수를 Varable Explorer윈도우에서 확인하자.
6. R Tools 메뉴를 눌러서 둘러보자.







## Lesson 3: The ScaleR functions



### Purpose of the ScaleR functions

- Scalable functions for analyzing large datasets

- Develop analyses locally on smaller datasets, and then deploy to servers, clusters or database services at scale, with minimal code changes

- Process data that is too big to fit in memory

- Distribute analyses over multiple cores, processors or nodes in a cluster

- Run updating algorithms on chunked data in XDF format



Summary of the ScaleR functions

- Input/output

- Data manipulation/chunking

- Descriptive statistics/cross tabulation

- Statistical modeling

- Basic plotting

- Compute contexts

- Data sources

- Low level distributed computing

- Utilities



#### The ScaleR functions and compute contexts

- You provide the following information to ScaleR functions:

  - The compute context

    - Local (default)

    - Server, cluster or database

  - The data source 

    - Text files

    - XDF chunked data

    - Database connection

- Prototype/test using the local compute context and test dataset

- Deploy against remote compute context and production dataset



### Best practice for computing with big data

- Upgrade your hardware
- Minimize data copying
- Chunk your data 큰 데이터를 조각내어 사용
- Use integers when you can
- Vectorize
- Avoid sorting big data  빅데이터정렬 방지
- Use row-oriented data transformations
- Process data in batches 데이터 일괄처리
- Be careful with factors that have many levels
- Invest in more nodes 많은 노드를 투자



### Lab: Exploring Microsoft R Server and Microsoft R Client

##### Exercise 1: Using R Client in RTVS and RStudio



##### Exercise 2: Exploring ScaleR functions



##### Exercise 3: Performing operations on a remote server





