# Module 6

## Creating and Evaluating Regression Models
### Module Overview

- Clustering big data
- Generating regression models and making predictions

### Lesson 1: Clustering big data

- Why perform clustering?
- Performing k-means clustering
- Evaluating clusters
- Standardizing data
- Optimizing k-means clustering
- Demonstration: Creating and examining a cluster

### Why perform clustering? (unsupervisor learning)

- Clustering is a type of <u>unsupervised learning</u>:

  - No labeled response data

  - Attempts to split the data into natural groups (신경망)

  - No measure of accuracy

- Useful for:

  - Exploratory data analysis

  - Finding natural groups in your data

  - To reduce datasets into subsets of similar data

  - Dimension reduction

  - Preprocessing of data before running supervised learning

  - Nonrandom sampling

### K-means clustering (Clustering 기법)

- K-means clustering (also know as partitioning clustering)

  - Divides “n” cases, described by “p” explanatory variables into a small number, “k”, of classes. 

  - You must specify k, the number of classes

##### Clustering the diamonds dataset

```R
remoteLogin(deployr_endpoint = "http://172.16.0.102:12800",
            session = TRUE, diff = TRUE, 
            commandline = TRUE, 
            username = "admin", 
            password = "Pa$$w0rd")

install.packages("ggplot2")

library(ggplot2)

tmp_data <- ggplot2::diamonds

tmp_data

str(tmp_data)

class(tmp_data)

df <- as.data.frame(tmp_data)

str(df)

colnames(df)

head(df)

clust <- rxKmeans(~ carat + depth + price, data = ggplot2::diamonds, numClusters = 5, 
                  seed = 1979)
clust
```

### # Linear Regression (선형회기) => 

### 					`rxLm` 선형 회기 함수를 구해주는 명령어 

Regression 은 추측 모델이 아닌 데이터를 기반으로 최적화 된 선을 만들어 준다. 

Y = aX + b 의 식을 도출해 내는 것이다. 

```R
# x : hight
# y : weight
x <- c(151,174,138,186,128,136,179, 163, 152, 131)
y <- c(63,81,56,91,47,57,76,72,62,48)
plot(x,y)
l_value <- lm(y~x)
```

```R
l_value
```

```R
Call:
lm(formula = y ~ x)

Coefficients:
(Intercept)            x  
   -38.4551       0.6746  
```

```R
plot(l_value)
```

![image](https://user-images.githubusercontent.com/46669551/54587526-ae6b5c00-4a63-11e9-94f5-8a5008cdaf21.png)

##### 위의 regression 함수를 이용하여 실제 데이터x의 값으로 y값을 추측할때 쓰는 함수

```R
x1 <- data.frame(x=170)
result <-predict(l_value, x1)
result
```

`출력`

```R
       1 
76.22869 
```

##### 그래프에 선형회기 선을 만들어 주는 명령어 

```R
plot(x,y,abline(lm(y~x)))
```

![image](https://user-images.githubusercontent.com/46669551/54587769-6dc01280-4a64-11e9-9421-4bd4567c8f37.png)

### #Multiple Regression

##### y = a1x1 + a2x2 + ... + b 의 식을 구해주는 기법

```R
# multiple regression
# y = a1x1 + a2x2 + ... + b

tmp_data2 <- mtcars[,c('mpg', "disp", 'hp','wt')]
tmp_data2
# mpg = disp + hp + wt 와 관련이 있다고 가정한다. 
# mpg ~ disp + hp + wt
model <- lm(mpg~disp+hp+wt, data = tmp_data2)
model

# 출력 >
Call:
lm(formula = mpg ~ disp + hp + wt, data = tmp_data2)

Coefficients:
(Intercept)         disp           hp           wt  
  37.105505    -0.000937    -0.031157    -3.800891  
# >

# y(mpg) = 37.10 + (-0.000937)disp + (-0.031157)hp + (-3.800891)wt
# 위와 같은 회기함수가 생성된다. 
# disp: 221, hp : 102, wt : 2.91 이라 가정하면
# y(mpg) = 37.10 + (-0.000937)*221 + (-0.031157)*102 + (-3.800891)*2.91
# ==> 22.7104 정도가 된다.
```



### # `glm` 

```R
tmp_data3 <- mtcars[,c('am', "cyl", 'hp','wt')]

am = cyl + hp + wt # 선형의 형태가 아닌 정규분포 등의 형태가 예상됨

tmp_data4 <- glm(am ~ cyl + hp + wt, data = tmp_data3, family = binomial())
tmp_data4
```

`출력`

```R
Call:  glm(formula = am ~ cyl + hp + wt, family = binomial(), data = tmp_data3)

Coefficients:
(Intercept)          cyl           hp           wt  
   19.70288      0.48760      0.03259     -9.14947  

Degrees of Freedom: 31 Total (i.e. Null);  28 Residual
Null Deviance:	    43.23 
Residual Deviance: 9.841 	AIC: 17.84
# 기존의 함수와 함께 잔차(residual)가 출력된다. 
```

```R
plot(tmp_data4)
```

![image](https://user-images.githubusercontent.com/46669551/54588648-f17afe80-4a66-11e9-9239-d9dc7ef0621b.png)

### Performing k-means clustering

- The RevoScaleR package provides **rxKmeans** for k-means clustering

  - Optimised for large datasets over clusters

  - Implements the Lloyd algorithm for heuristic clustering

    ![image](https://user-images.githubusercontent.com/46669551/54586422-6139bb00-4a60-11e9-84f2-56f3ee0cbbef.png)

  - Use a right-handed formula because there are no response variables

  - Set an appropriate number of clusters for k 

  - The data argument can be a big data resource (XDF)

### Evaluating clusters

- There is no real measure of “accuracy” in a cluster analysis because there is no response variable to compare against

- You can get an idea of the percentage of variation explained by the model by calculating the ratio of the “between cluster sums of squares” and “total sums of squares”:

- This returns the proportion of total variation explained by differences in cluster means