# Reproducible Research: Peer Assessment 1
The aim of this assignment is to analyse data from a personal activity monitoring device. So, first of all it is needed to download the data into an R object ready to be analysed. Here the code:

## Loading and preprocessing the data

```r
        data <- read.csv("activity/activity.csv", colClasses = "character")
        data[, c(1,3)] <- sapply(data[, c(1,3)], as.numeric)
```
## What is mean total number of steps taken per day?
To answer this question the overall number of steps taken per day has been calculated . Here the analysis code:

1. A data set containing the sum of the number of steps taken each day is created.


```r
        dailyStepSum <- aggregate(data$steps, list(data$date), sum)
        colnames(dailyStepSum) <- c("Date", "Steps")
```
2. Then, to visualize the information a barplot is created:

```r
        with(dailyStepSum, {
        par(oma=c(2,0,0,0), mar=c(6.75,6.75,3,0), mgp=c(5.75,0.75,0), las=2)
        barplot(
        height=Steps,
        main="Graph of Total Steps taken per Day",
        xlab="Dates",
        ylab="Steps per Day",
        names.arg=Date,
        space=c(0)
        )
        })
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

3. Finally to answer the question the media and the median has been calculated ignoring the missing values (NA).

```r
        dailyStepMean <- mean(dailyStepSum$Steps, na.rm = TRUE)
        dailyStepMean
```

```
## [1] 10766.19
```

```r
        dailyStepMedian <- median(dailyStepSum$Steps, na.rm = TRUE)
        dailyStepMedian
```

```
## [1] 10765
```

## What is the average daily activity pattern?
The goal of this section is to calculate the mean of the steps taken every 5 minutes interval averaged over all the days in the observation periode.
Then the data sample has been aggregate in a slightly different way than before, by intervals. Here the code:


```r
         intervalSteps <- aggregate(data=data,steps~interval,FUN=mean,na.action=na.omit)
         colnames(intervalSteps) <- c("Interval", "AvgStepsAvgAcrossDay")
```
1. A time series plot is created with the new dataset:


```r
        plot(intervalSteps$Interval, intervalSteps$AvgStepsAvgAcrossDay,ylab ="averaged number of steps", xlab="5 minutes intervals", type="l", main = "Time series plot")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

2. The interval (bin) with the maximum number of steps has been calculated:

```r
        maxbin <- intervalSteps[intervalSteps$AvgStepsAvgAcrossDay == max(intervalSteps$AvgStepsAvgAcrossDay),]
        maxbin
```

```
##     Interval AvgStepsAvgAcrossDay
## 104      835             206.1698
```
        
## Imputing missing values
1. In order to study the impact of the missing values, we have to calculate how many missing observations are present in the original data.


```r
        NAnumber <- sum(!complete.cases(data))
        NAnumber
```

```
## [1] 2304
```

2. The better strategy to fill up the missing values is to use the mean of in the same interval averaged on the overall number of days. Here the code:


```r
        stepValues <- data.frame(data$steps)
        stepValues[is.na(stepValues),] <- tapply(X=data$steps,INDEX=data$interval,FUN=mean,na.rm=TRUE)
        newData <- cbind(stepValues, data[,2:3])
        colnames(newData) <- c("Steps", "Date", "Interval")        
```

3. The mean number of step taken by day is now calculated with the new dataset:


```r
        dailyStepSumNew <- aggregate(newData$Steps, list(newData$Date), sum)
        colnames(dailyStepSumNew) <- c("Date", "Steps")
```

4. The new plot of the mean number of steps per day is the following:


```r
        with(dailyStepSumNew, {
        par(oma=c(2,0,0,0), mar=c(6.75,6.75,3,0), mgp=c(5.75,0.75,0), las=2)
        barplot(height=Steps,main="Graph of Total Steps taken per Day",xlab      ="Dates",ylab="Steps per Day",names.arg=Date,space=c(0))})
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 

5. In order to calculate the mean and the median of the new data set we can apply same functions as before.


```r
        dailyStepMeanNew <- mean(dailyStepSumNew$Steps)
        dailyStepMeanNew
```

```
## [1] 10766.19
```

```r
        dailyStepMedianNew <- median(dailyStepSumNew$Steps)
        dailyStepMedianNew
```

```
## [1] 10766.19
```
6. It seams that the mean has been unchanged, but the median is higher in the new data set.

## Are there differences in activity patterns between weekdays and weekends?

1. In order to answer the last question a new data set with a new column to indicate if the day is during the week or in the weekend has been created.
Here the code:


```r
        dateDayType <- data.frame(sapply(X = newData$Date, FUN = function        (day) {if (weekdays(as.Date(day)) %in% c("Monday", "Tuesday", "Wednesday", "Thursday","Friday")) {day <- "weekday"} else {day <- "weekend"}}))
        
        newDataWithDayType <- cbind(newData, dateDayType)
        colnames(newDataWithDayType) <- c("Steps", "Date", "Interval", "DayType")
```
2. To compare the physical activity during the week and weekend days the new data set has been modified (as before) to calculate the mean number of steps taken per interval averaged over all the week or weekend days.
The code:


```r
        dayTypeIntervalSteps <- aggregate(data=newDataWithDayType,Steps ~ DayType + Interval,FUN=mean)
```
        
Finally the panel plot has been generated:


```r
        library("lattice")

        xyplot(type="l",data=dayTypeIntervalSteps,Steps ~ Interval | DayType,xlab="Interval",ylab="Number of steps",layout=c(1,2))
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png) 
        


