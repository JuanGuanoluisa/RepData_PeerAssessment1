# Reproducible Research: Peer Assesment 1

**Juan Carlos Guanoluisa
13 - september - 2016**

## Introducction

*It is now possible to collect a large amount of data about personal movement using activity*
_monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are_
_part of the "quantified self" movement - a group of enthusiasts who take measurements about_
_themselves regularly to improve their health, to find patterns in their behavior, or because_
_they are tech geeks. But these data remain under-utilized both because the raw data are hard to_
_obtain and there is a lack of statistical methods and software for processing and interpreting_
_the data._

*This assignment makes use of data from a personal activity monitoring device. This device*
_collects data at 5 minute intervals through out the day. The data consists of two months of data_
_from an anonymous individual collected during the months of October and November, 2012 and_
_include the number of steps taken in 5 minute intervals each day._


## (Question 1) Loading and processing data

After downloading the data I set the working directory and I load it from the file activity.csv, transform date column as date 

Set working directory  


```r
setwd("C:/Disco_C_anterior/JuanCarlos/Cursos/Data Science/ReproducibleResearch/Semana 2") 
```


Read the data file in.


```r
dset <- read.csv("activity.csv")  
dset$date <- as.Date(dset$date)  
```

## (Question 2) What is mean total number of steps taken per day?

Histogram shows the total number of steps taken in each day Load the library ggplot2 to draw histogram

Load library to plot graphics  

```r
library(ggplot2)
```



```r
# Computes a summary of the total number of steps taken each day
Total_Steps <- aggregate(x = dset$steps , by = list(dset$date), FUN = sum ,na.rm=TRUE)
names(Total_Steps) <- c("date","steps")

# Plot using ggplot2
HistPlot1 <- ggplot(Total_Steps,aes(x = steps)) +
    ggtitle("Total Daily Steps") +
    xlab("Daily Steps with (binwidth = 2000)") + ylab("Count of Daily Steps") +
    geom_histogram(binwidth = 2000)
HistPlot1
```

![plot of chunk display/graphics1](figure/display/graphics1-1.png)


### Calculating the mean total number of steps taken per day

```r
mean(Total_Steps$steps)
```

```
## [1] 9354.23
```

### Calculatin the median total number of steps taken per day

```r
median(Total_Steps$steps)
```

```
## [1] 10395
```

## (Question 3) What is the average daily activity pattern?  

Make a time series plot of 5-minute interval and the average number of steps taken, averaged across all days.

```r
AVEGSteps<- aggregate(x = dset$steps , by = list(dset$interval), FUN = mean ,na.rm=TRUE)
names(AVEGSteps) <- c("interval","steps")

TimeSeriesPlotHist <- ggplot(AVEGSteps,aes(interval,steps)) +
                 ggtitle("Time Series Plot of Average Steps by Interval") +
                 geom_line()
TimeSeriesPlotHist
```

![plot of chunk display/graphics2](figure/display/graphics2-1.png)


###The 5-min time interval contains the maximum number of steps?

```r
AVEGSteps[which.max(AVEGSteps$steps),c("interval")]
```

```
## [1] 835
```

## (Question 4) Imputing missing values

### Calculating total number of missing values in the dataset

There are many days/intervals where there are missing values (coded as NA). The presence of
missing days may introduce bias into some calculations or summaries of the data.


```r
nrow(dset[is.na(dset$steps),])
```

```
## [1] 2304
```

```r
dset.imputed <- merge(x = dset, y = AVEGSteps, by = "interval", all.x = TRUE)
dset.imputed[is.na(dset.imputed$steps.x),c("steps.x")] <-
dset.imputed[is.na(dset.imputed$steps.x),c("steps.y")]
```

### Cleaning data

```r
dset.imputed$date <- as.Date(dset.imputed$date)
dset.imputed$date.x <- NULL
dset.imputed$Group.1 <- NULL
dset.imputed$steps <- dset.imputed$steps.x
dset.imputed$steps.x <- NULL
dset.imputed$steps.y <- NULL
```

### Histogram with new dataframe

```r
Total_Steps <- aggregate(x = dset.imputed$steps , by = list(dset.imputed$date), FUN = sum ,na.rm=TRUE)
names(Total_Steps) <- c("date","steps")
HistPlot2 <- ggplot(Total_Steps,aes(x = steps)) +
            ggtitle("Daily steps after Imputation") +
            xlab("Steps (binwidth 2000)") +
            geom_histogram(binwidth = 2000)
HistPlot2
```

![plot of chunk display/graphics3](figure/display/graphics3-1.png)

### mean total number of steps taken per day

```r
mean(Total_Steps$steps)
```

```
## [1] 10766.19
```

### median total number of steps taken per day

```r
median(Total_Steps$steps)
```

```
## [1] 10766.19
```

Mean and median values are higher after imputing missing data. The reason is that in the original
data, there are some days with steps values NA for any interval. The total number of steps taken
in such days are set to 0s by default. However, after replacing missing steps values with the
mean steps of associated interval value, these 0 values are removed from the histogram of total
number of steps taken each day.


## (Question 5) Are there differences in activity patterns between weekdays and weekends?

#### Create a new factor variable - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.  

```r
dset.imputed$weekday <- as.factor(ifelse(weekdays(dset.imputed$date) %in% c("Saturday","Sunday"),
"Weekend", "Weekday"))
AVEGWeekday  <- aggregate(x = dset.imputed$steps , by =
list(dset.imputed$interval,dset.imputed$weekday), FUN = mean ,na.rm=TRUE)
names(AVEGWeekday) <- c("interval","weekday","steps")
```

#### Make a panel plot containing a time series plot of the 5-minute interval and the average number of steps taken averaged across all weekday days or weekend days.


```r
AVEGPlotline <- ggplot(AVEGWeekday,aes(interval,steps)) +
                 ggtitle("Time Series Plot of Average Steps by Interval after Imputation") +
                 facet_grid(. ~ weekday) +
                 geom_line(size = 1)
AVEGPlotline
```

![plot of chunk display/graphics4](figure/display/graphics4-1.png)
