# Reproducible Research: Peer Assessment 1



This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day

### Variables & Data set


The variables included in this dataset are:  
- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)  
- date: The date on which the measurement was taken in YYYY-MM-DD format  
- interval: Identifier for the 5-minute interval in which measurement was taken 

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.


```r
#prepartions, loadin libraries, etc.
rm(list = ls())
options(scipen = 4, digits =2)
library(plyr)
```

### Loading and preprocessing the data

First we load the data (i.e. read.csv())


```r
setwd("C:/Private/repeatable/assignment 2")

dfIn <- read.csv("activity.csv", stringsAsFactors = FALSE)
```

### What is mean total number of steps taken per day?

### Aggregate steps by day


```r
dfPerDate <- ddply(dfIn[,1:2],.(date),summarize, 
            stepsSum = sum(steps) 
            )
```

### An histogram of the total number of steps taken each day

A time series plot  of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
dates <- as.Date(dfPerDate[is.na(dfPerDate$stepsSum) == FALSE,"date"],format = "%Y-%m-%d")
vals <- dfPerDate[is.na(dfPerDate$stepsSum) == FALSE,"stepsSum"]
plot(dates,vals, xaxt = "n", type = "s")
axis.Date(side = 1, dates, format = "%d/%m/%Y")
```

![plot of chunk unnamed-chunk-4](./PA1_template_files/figure-html/unnamed-chunk-4.png) 

### mean and median total number of steps taken per day

Calculation of mean and median of the steps per day ignoring NA values


```r
meanOfStepsPerDay <- round(mean(dfPerDate$stepsSum,na.rm= TRUE), digits = 2)
medianOfStepsPerDay <- round(median(dfPerDate$stepsSum,na.rm= TRUE), digits = 2)
```
the resulting mean 10766.19 is  and the median is 10765

### What is the average daily activity pattern?


```r
# What is the average daily activity pattern? 
# aggregate steps by interval
dfPerInterval <- ddply(dfIn[,c(1,3)],.(interval),summarize, mean= mean(steps,na.rm = TRUE),sum= sum(steps,na.rm = TRUE))
plot(dfPerInterval$interval,dfPerInterval$mean,type="l")
```

![plot of chunk unnamed-chunk-6](./PA1_template_files/figure-html/unnamed-chunk-6.png) 

```r
maxInterval <- dfPerInterval[dfPerInterval$mean == max(dfPerInterval$mean,na.rm = TRUE),  "interval"]
```
### Imputing missing values

Calculating the total number of missing values


```r
numOfNAValues <- length(dfIn[is.na(dfIn$steps),"steps"])
```


The number of missing values is 2304.  

### Filling in all of the missing values in the dataset.

We will use the previously calculated mean steps per 5- minutes interval to fill the NA values.
And create a data set that is equal to the original dataset but with the missing data filled in (called dfInNoNA)  



```r
dfInNoNA <- join(dfIn,dfPerInterval,by="interval")
dfInNoNA[is.na(dfInNoNA$steps),"steps"] <- dfInNoNA[is.na(dfInNoNA$steps),"mean"]
dfInNoNA <- dfInNoNA[,c("steps","date","interval")]
```


we will repeat on the the filled data set, dfInNoNA, the same calculations and graphs that we previously performed on the data set with the NAs,

### What is the average daily activity pattern (after filing missing values)?


```r
dfPerDateNoNA <- ddply(dfInNoNA[,1:2],.(date),summarize, 
                   stepsSum = sum(steps))

# mean total number of steps taken per day
# histogram of the total number of steps taken each day
dates2 <- as.Date(dfPerDateNoNA[,"date"],format = "%Y-%m-%d")
vals2 <- dfPerDateNoNA[,"stepsSum"]
plot(dates2,vals2, xaxt = "n", type = "s")
axis.Date(side = 1, dates2, format = "%d/%m/%Y")
```

![plot of chunk unnamed-chunk-9](./PA1_template_files/figure-html/unnamed-chunk-9.png) 


# calculation of mean and median of the steps per day previously 

```r
meanOfStepsPerDayNoRA <- mean(dfPerDateNoNA$stepsSum,na.rm= TRUE)
medianOfStepsPerDayNoRA <- median(dfPerDateNoNA$stepsSum,na.rm= TRUE)
totalStepsPerDayNoRA <- sum(dfPerDateNoNA$stepsSum,na.rm= TRUE)
totalStepsPerDay <- sum(dfPerDate$stepsSum,na.rm= TRUE)
```

The resulting mean 10766.19 is  and the median is 10766.19
Note that  after filling the missing values 
- the mean (10766.19) is equal to the mean before filling the missing values (10766.19). This is direct result of using means to fill the missing values. 
- the median (10766.19) is very close to the median before filling the missing values (10765)
- however there is a large difference (86129.51) in the total number of steps after filling NA (656737.51) vs. the raw data (570608)

### Are there differences in activity patterns between weekdays and weekends?

We will  add a variable to the data frame  'weekday' that indicates if a data is weekend or weekday. Note Saturday and Sunday are considered the weekend


```r
Sys.setlocale("LC_TIME", "English")
```

```
## [1] "English_United States.1252"
```

```r
dfInNoNAWeekday <- dfInNoNA
dfInNoNAWeekday$weekday <- weekdays(as.Date(dfInNoNA[,"date"],abbreviate=TRUE))
dfInNoNAWeekday$isWeekend <- grepl("(Saturday)|(Sunday)",dfInNoNAWeekday$weekday)
dfInNoNAWeekday[!dfInNoNAWeekday$isWeekend,]$weekday <- "weekday" 
dfInNoNAWeekday[dfInNoNAWeekday$isWeekend,]$weekday <- "weekend"
```

and aggregate steps by interval and day tye (weekend/weeekday)


```r
dfPerIntervalDayType <- ddply(dfInNoNAWeekday[,c(1,3,4)],.(interval,weekday),summarize, mean= mean(steps,na.rm = TRUE),sum= sum(steps,na.rm = TRUE))
```

### A panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
par(mfrow = c(2,1))
plot(dfPerIntervalDayType[dfPerIntervalDayType$weekday == "weekend",]$interval,
     dfPerIntervalDayType[dfPerIntervalDayType$weekday == "weekend",]$mean,type="l",xaxt = "n", xlab ="", ylab="")
title("weekend")
plot(dfPerIntervalDayType[dfPerIntervalDayType$weekday == "weekday",]$interval,
     dfPerIntervalDayType[dfPerIntervalDayType$weekday == "weekday",]$mean,type="l",xlab="interval", ylab="mum of steps")
title("weekday")
```

![plot of chunk unnamed-chunk-13](./PA1_template_files/figure-html/unnamed-chunk-13.png) 


