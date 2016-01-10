---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

Download into 'dataset.zip / extract into 'activity.csv'

```r
fileurl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileurl, destfile = "dataset.zip", method = "curl")
unzip("dataset.zip")
activity <- read.csv("activity.csv")
```

Add a column to specify the day of the week

```r
activity$day <- weekdays(as.Date(activity$date))
```


## What is mean total number of steps taken per day?
Get the total amount of steps for each day:

```r
totalStepsByDay <- aggregate(activity$steps, by=list(Category=activity$date), FUN=sum)
```
Histogram:

```r
qplot(date, steps, data=activity, stat="summary", fun.y="sum", geom="bar")
```

```
## Error in eval(expr, envir, enclos): could not find function "qplot"
```
![plot](https://raw.githubusercontent.com/Linz17/RepData_PeerAssessment1/master/activityTotalStepsByDay.png)

Get the mean an median of steps by day:

```r
summary(totalStepsByDay$x)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##      41    8841   10760   10770   13290   21190       8
```
The **mean** total number of steps taken per day is **10770**
The **median** total number of steps taken per day is **10760**


## What is the average daily activity pattern?
Get the mean amount of steps for each interval:

```r
totalStepsByInterval <- aggregate(activity$steps, by=list(Category=activity$interval), FUN=mean, na.rm=TRUE)
```
Get the interval with the most steps

```r
totalStepsByInterval[which.max(totalStepsByInterval$x),]
```

```
##     Category        x
## 104      835 206.1698
```
The 5-minute interval, on average across all the days in the dataset, that contains the maximum number of steps is:
**835** with **206.1698** steps on average

Plot:

```r
plot(meanStepsByInterval, type="l", xlab="Interval", ylab="Steps")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 
![plot](https://raw.githubusercontent.com/Linz17/RepData_PeerAssessment1/master/meanStepsByInterval.png)


## Imputing missing values
Total number of missing values in the dataset: **2304**

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```
New dataset, filling gaps using mean:

```r
noNAactivity <- activity
for (i in 1:17568) {
  if (is.na(noNAactivity$steps[i])== TRUE) {
    noNAactivity$steps[i] <- round(meanStepsByInterval[meanStepsByInterval[,1] == noNAactivity$interval[i],2])
  }
}
```
Old dataset: Mean = 10770 // Median = 10760

```r
summary(totalStepsByDay$x)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##      41    8841   10760   10770   13290   21190       8
```
New dataset: Mean = 10770 // Median = 10760

```r
summary(totalStepsByDayFILLED$x)
```

```
## Error in summary(totalStepsByDayFILLED$x): object 'totalStepsByDayFILLED' not found
```

Histogram:

```r
qplot(date, steps, data=noNAactivity, stat="summary", fun.y="sum", geom="bar")
```

```
## Error in eval(expr, envir, enclos): could not find function "qplot"
```
![Plot](https://raw.githubusercontent.com/Linz17/RepData_PeerAssessment1/master/noNAactivityTotalStepsByDay.png)


## Are there differences in activity patterns between weekdays and weekends?

Subsetting into 2 datasets: weekdaysActivity and weekendActivity.

```r
weekdaysActivity <- noNAactivity[noNAactivity$day %in% c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday"),]
weekendActivity <- noNAactivity[noNAactivity$day %in% c("Saturday", "Sunday"),]
meanStepsByIntervalWeekdays <- aggregate(weekdaysActivity$steps, by=list(Category=weekdaysActivity$interval), FUN=mean, na.rm=TRUE)
meanStepsByIntervalWeekends <- aggregate(weekendActivity$steps, by=list(Category=weekendActivity$interval), FUN=mean, na.rm=TRUE)
```

Plot:

```r
par(mfcol = c(2,1))
with(meanStepsByIntervalWeekdays, {
  plot(meanStepsByIntervalWeekdays, type = "l", xlab="Interval", ylab="Steps")
  plot(meanStepsByIntervalWeekends, type = "l", xlab="Interval", ylab="Steps")
})
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15-1.png) 
![Plot](https://raw.githubusercontent.com/Linz17/RepData_PeerAssessment1/master/diffWeekdayWeekend.png)
