# Create a Hadoop compute context

~~~R
context <- RxHadoopMR(sshUsername = "hadoop",  sshHostname = "172.168.137.101")
~~~

~~~R
rxSetComputeContext(context)
~~~

Hadoop 서버와 연결해주는 명령어 



### List the contents of the /user/hadoop folder in HDFS

###### hadoop fs -ls /user/hadoop

###### hdfs namenode -format

###### hadoop dfs -ls/

~~~R
rxHadoopCommand("fs -ls /user/hadoop")
~~~

Hadoop 서버의 folder를 출력하는 명령어



###### hadoop fs -put input(local) output(hdfs,path)

###### HDFS, SqlServer, Text(CSC...) ... SAS, SPAA, ...(Sources)



### Connect directly to HFDS on the Hadoop VM

~~~
hdfsConnection <- RxHdfsFileSystem()
~~~

~~~R
rxSetFileSystem(hdfsConnection)
~~~

Hadoop Virtual Machine의 HFDS에 연결하는 명령어 



### Create a data source for the CensusWorkers.xdf file

~~~
workerInfo <- RxXdfData("/user/hadoop/CensusWorkers.xdf")
~~~

xdf파일 형식의 CensusWorkers.xdf을 호출하기 



### Perform functions that read from the CensusWorkers.xdf file

~~~R
head(workerInfo)
~~~

~~~R
rxSummary(~., workerInfo)
~~~

호출된 파일을 




###### Sources => 

###### 		[File System]  RDBMS, HDFS(...), Storage Account, S3...

###### 		[Filetype]  (XDF...)

###### Microsoft R Server (R)

###### 		--> Machine Learning Server - R, Python

###### => 병렬, 분산 ...

###### =>Node