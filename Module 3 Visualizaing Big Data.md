# Module 3

## Visualizing Big Data

### Module Overview

- Visualizing in-memory data : ggplot 등은 메모리를 이용하여 처리한다. 
- Visualizing big data : data의 양이 많아지면 다른 방식을 택한다. 

#### Lesson 1: Visualizing in-memory data
- Overview of the ggplot2 package
- Understanding ggplot geometries
- Combining plots
- Using aesthetic mappings
- Creating facets
- Customizing bar plots
- Understanding coordinate systems
- Arranging plots
- Demonstration: Creating a faceted plot with overlays using ggplot



#### Overview of the ggplot2 package

- The grammar of graphics

- **ggplot** objects consist of:
  - <u>**Data**</u> : raw data를 지칭하며, data frame의 형태이여야 한다.

  - <u>**Aesthetic mappings**</u> : raw data를 가독 가능한 graph의 현태로 변환해주는 것이다. 

  - <u>**Geometric objects (geoms)**</u> : 실질적인 plot 타입을 이야기하며, line, point, histogram등이 존재한다.

  - <u>**Coordinate system**</u> :   평면에 있는 점의 위치를 나타내는 좌표계(coordinate system)은 좌표라고 부르는 순서쌍으로 나타낸다. 

  - <u>**Statistical transformations (stats)**</u>: data를 transform하거나 summarize할 때 사용한다. regression line, boxplots 등으로 geom이 가지고 있는 고유한 stat이다. 

  - <u>**Facets**</u> : facet이라는 기법은 데이터를 특정 기준에 따라 서브셋으로 나눈 뒤 각 서브셋을 각기 다른 그래프 패널에 출력하는 것을 의미한다.



#### Understanding ggplot geometries

- Geometric objects (geoms) determine the kind of plot to be produced:

  - **geom_point()**: an XY **scatter plot** (two continuous variables)

  - **geom_bar()**: a **bar chart** for a single discrete variable

  - **geom_histogram()**: for a single continuous variable

  - **geom_line()**: two variables with **joined points** 

  - **geom_smooth()**: for a **smooth summary** or **regression line**



#### Combining plots

- You can chain different `geoms` together using  ` “+”`

##### Example:

![image](https://user-images.githubusercontent.com/46669551/54415043-794acb00-473e-11e9-9e97-e773dd495ea8.png)

#### Using aesthetic mappings

- Each geom has a mapping argument that accepts an **aes()** expression that maps variables to:

  - Points in space (x and y): x, y 두개의 축으로 나뉜다. 

  - The color of the points and lines (color)

  - The shape of the points (shape)

  - The style of lines (linetype)

  - The color of bars, histograms or polygons (fill)
- **color**. the color of points and lines
- **size**. the size of points
- **shape**. the shape of point
- **linetype**. the style of line
- **fill**. the color of bars, histograms, and polygons

++ 질적 자료 : 범주 ... (수치로 측정하기 어려움, 서열이 없음, 명목척도, 예 : 성별, 취미, 국가)

++ 양적 자료 : 이산 ... (수량, 서열이 있음, 서열 척도, 등간척도, 비율척도, 수치로 측정 가능한 자료 , 예: 나이,무게,가격)

##### #Setting the color of points and labels by category 

```R
ggplot(mtcars, mapping = aes(x = disp, y = mpg, color = factor(am, levels = c(0,1), labels = c("Automatic","Manual")))) + 
scale_color_manual(values = c("blue", "yellow")) +
geom_point() +
labs(color = "Transmission")
```



#### Creating facets

- You can split your plot into different panels by categorical variables

- There are two faceting functions:

  - **facet_grid(var1 ~ var2)**: splits your data into rows by var1 and columns by var2

  - **facet_wrap(~ var1, ncol = 2)**: splits your data by a single variable. The panels wrap around to the next line. You can set the number of columns using ncol

- Don’t overdo faceting
  - It can obscure relationships



#### Customizing bar plots

- Use the **geom_bar** geometry to create bar plots 
- The most commonly used options are **position** and **stat**

- Position argument: **Position**의 옵션은 그룹화된 bar plot을 다룰때 유용하다.(grouping using the **fill** argument)
  - Specify whether bars are stacked or dodged (alternating)

```R
# The position argument

ggplot(mtcars) + geom_bar(mapping = aes(x = factor(cyl),fill = factor(am)), show.legend = FALSE, position = "stack")
```

- Stat argument: 만약 실제 data point를 대표하고 싶다면 `stat = 'identity'`를 주고 y축의 변수와 map한다.
  - Display discrete values of each instance of a variable rather than counts

```R
# Using the stat argument to graph data point

mpg_df <- data.frame(name = rownames(mtcars), mpg = mtcars$mpg)
ggplot(mpg_df) + geom_bar(mapping = aes(x = name, y = mpg), stat = 'identity') + coord_flip()
```



#### Understanding coordinate systems

- Coordinate systems map the positon of geometric objects to the plane of the plot. They apply to the entire plot:

  - **coord_cartesian()**: the default Cartesian coordinate system

  - **coord_flip()**: flips the x and y axes—for example, to make columns rather than bar plots

  - **coord_polar()**: polar coordinates for pie, donut and radar plots

  - **coord_trans()**: transformed Cartesian coordinates (for example, log, square root transformations)

  - **coord_map()**: creates map projections
  - **coord_fixed()**: These are Cartesin coordinates that have a fixed aspect ratio between the x and y axes

#### Arranging plots

- 결과를 비교하기 위해 두개의 plot을 함께 나란히 출력하고 싶을 때 사용한다. 

- You can arrange different plots next to each other for comparison or presentations 

  - Use the **gridExtra** package 

  - For example, to arrange two plot objects (plot1 and plot2) side by side:

![image](https://user-images.githubusercontent.com/46669551/54415198-e52d3380-473e-11e9-9371-b2f10241752c.png)

```R
# Arranging plots side by side

library(gridExtra)
p1 <- ggplot(mtcars, mapping = aes(x = disp, y = mpg)) + geom_point() + geom_smooth() + labs(title = "LOESS smoother")
p2 <- ggplot(mtcars, mapping = aes(x = disp, y = mpg)) + geom_point() + geom_smooth(method = "lm" , formula = y ~ poly(x,2)) + labs(title = "2nd degree polynomial lm")grid.arrange(p1, p2, ncol =2)
```



#### Lesson 2: Visualizing big data

- Creating scatter and line plots
- Creating histograms
- Saving plots
- Transforming data as it is plotted
- Demonstration: Generating a histogram with **rxHistogram**

#### Creating scatter and line plots

- The **rxLinePlot** function is a wrapper around the **xyplot** function from lattice that can operate on big data 

- Consider summarizing data using **rxCube** first 
  - Convert the results into a data frame using **rxResultsDF**

- Using **rxLinePlot**, you can create:

  - `Line plots` 

  - `Scatter plots ` 

  - `LOESS smoothers`

  - `Regression lines`

  - `Trellis plots` (similar to facets in **ggplot2**)

- You can customize layouts using features of the **lattice** package

#### Creating a trellis plots

- Trellis Graphics is a family of techniques for viewing complex, multi-variable data sets.
- **Trellis plots** are based on the idea of conditioning on the values taken on by one or more of the variables in a data set. • In the case of a categorical variable, this means carrying out the same **plot** for the data subsets corresponding to each of the levels of that variable.



#### Creating histograms

- Create histograms using the **rxHistogram** function 
  - Summarizes data using the **rxCube** function
  - Passes the summaries to lattice functions for plotting

- You can use the **F** function in a formula to quickly produce bins for each integer level
- Set the **histType** argument to populate bins with counts or percentages



#### Saving plots

- Save **rxLinePlot** and **rxHistogram** plots in the same manner as base R plots

  - Open a file handle

  - Generate the plot

  - Close the file handle

![image](https://user-images.githubusercontent.com/46669551/54415829-dc3d6180-4740-11e9-9cda-2e053c186374.png)

#### Transforming data as it is plotted

- Transformations are the same as for **rxSummary**

  - The **transforms** argument accepts a list of variables and their transformations—

    for example:

![image](https://user-images.githubusercontent.com/46669551/54415863-fd05b700-4740-11e9-8f34-6e78a348aa57.png)

- You can also filter data with the **rowSelection** argument

#### Generating a histogram with rxHistogram
- Creating a histogram

- Creating side-by-side histograms and overlays

```R
# Use the flight delay data
setwd("E:\\Demofiles\\Mod03")
airlineDataFile <- "FlightDelayData.xdf"
airlineData <- RxXdfData(airlineDataFile)
rxGetInfo(airlineData, getVarInfo = TRUE)

# Create a histogram showing the number of flights departing from each state
rxHistogram(~OriginState, airlineData,
            xTitle = "Departure State",
            yTitle = "Number of Flights",
            scales = (list(
              x = list(rot = 90, cex = 0.5)
            )))

# Filter the data to only count late flights
rxHistogram(~OriginState, airlineData, rowSelection = ArrDelay > 0,
            xTitle = "Departure State",
            yTitle = "Number of Late Flights",
            scales = (list(
              x = list(rot = 90, cex = 0.5)
            )))

# Flights by Carrier #항공사별
flightsByCarrier <- rxHistogram(~UniqueCarrier, airlineData,
                                xTitle = "Carrier",
                                yTitle = "Number of Flights",
                                yAxisMinMax = c(0, 2E6),
                                scales = (list(
                                  x = list(rot = 90, cex = 0.6)
                              )))

# Late flights by carrier
lateFlightsByCarrier <- rxHistogram(~UniqueCarrier, airlineData, rowSelection = ArrDelay > 0,
                                    xTitle = "Carrier",
                                    yTitle = "Number of Late Flights",
                                    yAxisMinMax = c(0, 2E6),
                                    plotAreaColor = "transparent",
                                    fillColor = "magenta",
                                    scales = (list(
                                      x = list(rot = 90, cex = 0.6)
                                  )))

# Display both histograms in adjacent panels # 두개의 히스토그램을 출력 가능케 해주는 페키지 
install.packages("latticeExtra")
library(latticeExtra)

print(c(flightsByCarrier, lateFlightsByCarrier))

# Overlay the histograms # 두개의 히스토그램을 합쳐서 출력해준다.
print(flightsByCarrier + lateFlightsByCarrier)
```



# Lab: Visualizing data

## Exercise 1: Visualizing data using the ggplot2 package

> ###### Task 1: Import the flight delay data into a data frame

##### # Generate plots of delay data to visualize any relationship between delay and distance. 

##### # Use ggplot2, with overlays and facets



##### #The location of the XDF file used in this exercise

```R
setwd("E:\\Labfiles\\Lab03")
```

##### #Create a data source for the extended data

~~~R
flightDelayDataXdf <- "FlightDelayData.xdf"
flightDelayData <- RxXdfData(flightDelayDataXdf)
~~~

##### #Create a data frame using the columns required for the plots. 

##### #Create a random sample of 2% of the data (to avoid running out of memory in ggplot)

```R
rxOptions(reportProgress = 1)
delayPlotData <- rxImport(flightDelayData, rowsPerRead = 1000000, 
                     varsToKeep = c("Distance", "Delay", "Origin", "OriginState"),
                     rowSelection = (Distance > 0) & as.logical(rbinom(n = .rxNumRows, size = 1, prob = 0.02))
                )
```

> ###### Task 2: Generate line plots to visualize the delay data

##### #Install packages

```R
install.packages("tidyverse")
library(tidyverse)
```

##### # Basic plot: Flight Distance vs Total Delay

```R
ggplot(data = delayPlotData) +
  geom_point(mapping = aes(x = Distance, y = Delay)) +
  xlab("Distance (miles)") +
  ylab("Delay (minutes)")
```

##### #Add a line plot to establish whether there is a pattern

##### #Remove all non-existant, negative delays, and questionable outliers first

```R
delayPlotData %>%
  filter(!is.na(Delay) & (Delay >= 0) & (Delay <= 1000)) %>%
  ggplot(mapping = aes(x = Distance, y = Delay)) +
  xlab("Distance (miles)") +
  ylab("Delay (minutes)") +
  geom_point(alpha = 1/50) +
  geom_smooth(color = "red")
```

##### #Faceted by originState #항공사별 그래프를 출력해 줌

```R
delayPlotData %>%
  filter(!is.na(Delay) & (Delay >= 0) & (Delay <= 1000)) %>%
  ggplot(mapping = aes(x = Distance, y = Delay)) +
  xlab("Distance (miles)") +
  ylab("Delay (minutes)") +
  geom_point(alpha = 1/50) +
  geom_smooth(color = "red") +
  theme(axis.text = element_text(size = 6)) +
  facet_wrap( ~ OriginState, nrow = 8)
```

## Exercise 2: Examining relationships in big data using the `rxLinePlot` function

> ###### Task 1: Plot delay as a percentage of flight time

##### #Add delay as a percentage of flight time to the data

```R
delayDataWithProportionsXdf <- "FlightDelayDataWithProportions.xdf"
delayPlotDataXdf <- rxImport(flightDelayData, outFile = delayDataWithProportionsXdf, 
                             overwrite = TRUE, append ="none", rowsPerRead = 1000000, 
                             varsToKeep = c("Distance", "ActualElapsedTime", "Delay", "Origin", "Dest", "OriginState", "DestState", "ArrDelay", "DepDelay", "CarrierDelay", "WeatherDelay", "NASDelay", "SecurityDelay", "LateAircraftDelay"),
                             rowSelection = (Distance > 0) & (Delay >= 0) & (Delay <= 1000) & !is.na(ActualElapsedTime) & (ActualElapsedTime > 0),
                             transforms = list(DelayPercent = (Delay / ActualElapsedTime) * 100) 
                    )
```

##### #Create a cube to summarize the data of interest - gets the data down to a manageable volume for plotting

```R
delayPlotCube <- rxCube(DelayPercent ~ F(Distance):OriginState, data = delayPlotDataXdf,
                        rowSelection = (DelayPercent <= 100)
                 )
names(delayPlotCube)[1] <- "Distance"
```

##### #Convert the cube into a data frame (required by rxLinePlot)

```R
delayPlotDF <- rxResultsDF(delayPlotCube)
```

##### #Generate line plots of delay as a percentage of flight time versus distance

##### #First, as a set of points

```R
rxLinePlot(DelayPercent~Distance, data = delayPlotDF, type="p",
           title = "Flight delay as a percentage of flight time against distance",
           xTitle = "Distance (miles)",
           yNumTicks = 10,
           yTitle = "Delay %",
           symbolStyle = ".",
           symbolSize = 2,
           symbolColor = "red",
           scales = (list(x = list(draw = FALSE)))
)
```



> ###### Task 2: Plot regression lines for the delay data

##### #Next, with a smoothed curve overlay

```R
rxLinePlot(DelayPercent~Distance, data = delayPlotDF, type=c("p", "smooth"),
           title = "Flight delay as a percentage of flight time against distance",
           xTitle = "Distance (miles)",
           yNumTicks = 10,
           yTitle = "Delay %",
           symbolStyle = ".",
           symbolSize = 2,
           symbolColor = "red",
           scales = (list(x = list(draw = FALSE)))
)
```

##### #Break the plot down by origin state

```R
rxLinePlot(DelayPercent~Distance | OriginState, data = delayPlotDF, type="smooth",
           title = "Flight delay as a percentage of flight time against distance, by state",
           xTitle = "Distance (miles)",
           yTitle = "Delay %",
           symbolStyle = ".",
           symbolColor = "red",
           scales = (list(x = list(draw = FALSE)))
)
```



> ###### Task 3: Visualize delays by the day of the week

##### #Examine whether delay might be a function of departure time and day

##### #Break the departure time down into 30 minute intervals so it can be plotted more easily

```R
delayDataWithDayXdf <- "FlightDelayWithDay.xdf"
delayPlotDataWithDayXdf <- rxImport(flightDelayData, outFile = delayDataWithDayXdf, 
                                    overwrite = TRUE, append ="none", rowsPerRead = 1000000, 
                                    varsToKeep = c("Delay", "CRSDepTime", "DayOfWeek"),
                                    transforms = list(CRSDepTime = cut(as.numeric(CRSDepTime), breaks = 48)),
                                    rowSelection = (Delay >= 0) & (Delay <= 180)
                           )
```

##### #Recode the DayOfWeek factor from a number to a meaningful set of abbreviations

```R
delayPlotDataWithDayXdf <- rxFactors(delayPlotDataWithDayXdf, outFile = delayDataWithDayXdf, 
                                     overwrite = TRUE, blocksPerRead = 1,
                                     factorInfo = list(DayOfWeek = list(newLevels = c(Mon = "1", Tue = "2", Wed = "3", Thu = "4", Fri = "5", Sat = "6", Sun = "7"),
                                                                   varName = "DayOfWeek"))
                           )
```

##### #Summarize the data

```R
delayDataWithDayCube <- rxCube(Delay ~ CRSDepTime:DayOfWeek, data = delayPlotDataWithDayXdf)
```

##### #Convert the cube into a data frame 

```R
delayPlotDataWithDayDF <- rxResultsDF(delayDataWithDayCube)
```

##### #Generate a line plot of delay as a function of departure time

```R
rxLinePlot(Delay~CRSDepTime|DayOfWeek, data = delayPlotDataWithDayDF, type=c("p", "smooth"),
           lineColor = "blue",
           symbolStyle = ".",
           symbolSize = 2,
           symbolColor = "red",
           title = "Flight delay, by departure day and time",
           xTitle = "Departure time",
           yTitle = "Delay (mins)",
           xNumTicks = 24,
           scales = (list(y = list(labels = c("0", "20", "40", "60", "80", "100", "120", "140", "160", "180")),
                          x = list(rot = 90),
                                   labels = c("Midnight", "", "", "", "02:00", "", "", "", "04:00", "", "", "", "06:00", "", "", "", "08:00", "", "", "", "10:00", "", "", "", "Midday", "", "", "", "14:00", "", "", "", "16:00", "", "", "", "18:00", "", "", "", "20:00", "", "", "", "22:00", "", "", "")))
           )
```



##### Exercise 3: Creating histograms over big data

> ###### Task 1: Create histograms to visualize the frequencies of different cause of delay

##### #Generate histograms of delay data to visualize the rates of different causes of delay by origin state and time of year. 



##### 1. This code creates the XDF file to be used to plot flight delays by state and by month

```R
delayReasonDataXdf <- "FlightDelayReasonData.xdf"

delayReasonData <- rxImport(flightDelayData, outFile = delayReasonDataXdf, 
                            overwrite = TRUE, append ="none", rowsPerRead = 1000000, 
                            varsToKeep = c("OriginState", "Delay", "ArrDelay", "WeatherDelay", "MonthName"),
                            rowSelection = (Delay >= 0) & (Delay <= 1000) &
                                           (ArrDelay >= 0) & (DepDelay >= 0)
                            )
```



##### 2. This code creates a histogram that counts the frequency of arrival delays

```R
rxHistogram(formula = ~ ArrDelay, data = delayReasonData, 
            histType = "Counts", title = "Total Arrival Delays",
            xTitle = "Arrival Delay (minutes)",
            xNumTicks = 10)
```

##### 3. This code creates a histogram that shows the percentage frequency of arrival delays as a percentage

```R
rxHistogram(formula = ~ ArrDelay, data = delayReasonData, 
            histType = "Percent", title = "Frequency of Arrival Delays",
            xTitle = "Arrival Delay (minutes)",
            xNumTicks = 10)
```

##### 4. This code creates a histogram that shows the percentage frequency of arrival delays as a percentage, organized by state

```R
rxHistogram(formula = ~ ArrDelay | OriginState, data = delayReasonData, 
            histType = "Percent", title = "Frequency of Arrival Delays by State",
            xTitle = "Arrival Delay (minutes)",
            xNumTicks = 10)
```

##### 5. This code creates a histogram that shows the frequency of weather delays

```R
rxHistogram(formula = ~ WeatherDelay, data = delayReasonData, 
            histType = "Counts", title = "Frequency of Weather Delays",
            xTitle = "Weather Delay (minutes)",
            xNumTicks = 20, xAxisMinMax = c(0, 180), yAxisMinMax = c(0, 20000))
```

##### 6. This code create a histogram that shows the frequency of weather delays organized by month

```R
rxHistogram(formula = ~ WeatherDelay | MonthName, data = delayReasonData, 
            histType = "Counts", title = "Frequency of Weather Delays by Month",
            xTitle = "Weather Delay (minutes)",
            xNumTicks = 10, xAxisMinMax = c(0, 180), yAxisMinMax = c(0, 3000))
```













#### R-Studio Packages 

https://www.rstudio.com/resources/cheatsheets/