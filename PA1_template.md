---
title: 'Peer-graded Assignment: Course Project 1'
author: "Rown Parola"
date: "7/10/2020"
output: 
  html_document: 
    keep_md: yes
---



This is the first assignment in the Coursera Reproducible Research course.

My first task is to load and do preprocessing of the date.

The following code downloads the data, unzips it, and reads it into R


```r
myurl<- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(url = myurl, destfile = "Zipped_Data")
unzip(zipfile = "Zipped_Data")
activity<-read.csv(file="activity.csv")
```

The following code process/transforms the data into a format suitable for my
analysis.


```r
Date<-with(activity,as.Date(date))
```

The following code begins by summing the total number of steps in each day, then
creating a histogram of the frequency of the total number of steps in a day.


```r
TotStepsInDay<-aggregate(activity$steps,by=list(Date=activity$date),FUN=sum)
hist(TotStepsInDay$x,xlab="Steps",main="Histogram of Steps in a Day",
     breaks = c(0,5000,10000,15000,20000,25000))
```

![](PA1_template_files/figure-html/Histogram_of_steps_taken_each_day-1.png)<!-- -->

The following code calculate the mean and median number of steps taken each day.


```r
MeanSteps<-mean(TotStepsInDay$x,na.rm = TRUE)
MedianSteps<-median(TotStepsInDay$x, na.rm = TRUE)

paste("The mean and median total number of steps taken per day are",MeanSteps,
      "and", MedianSteps,"respectively.")
```

```
## [1] "The mean and median total number of steps taken per day are 10766.1886792453 and 10765 respectively."
```

The following code describes and shows two strategies for imputing missing data. 
It begins by calculating and reporting the total number of missing values in the
dataset. 

```r
IntervalsMissingSteps<-sum(!complete.cases(activity))
DailyMissingSteps<-sum(!complete.cases(TotStepsInDay))
TotalMissingSteps<-IntervalsMissingSteps+DailyMissingSteps
paste("The number of intervals and total days missing step values is",
      IntervalsMissingSteps,"and", DailyMissingSteps,"respectively")
```

```
## [1] "The number of intervals and total days missing step values is 2304 and 8 respectively"
```

The missing interval values are replaced by the mean interval value for that
interval. The missing daily values are replaced by the mean daily value for 
that day of the week. Next a histogram of the total number of steps taken each 
day with the imputed values of interval and daily total steps per day are 
displayed.


```r
MeanInterval <- activity %>%
  group_by(interval) %>%
  summarise(steps = mean(steps,na.rm=T))

MissingSteps<-is.na(activity$steps)
MeanStepsPerInterval<-rep(MeanInterval$steps,61)
ImputedInterval<-data.frame(activity,MeanStepsPerInterval)
ImputedInterval$steps[MissingSteps]<-
  ImputedInterval$MeanStepsPerInterval[MissingSteps]

TotStepsInDay$DayOfWeek<-weekdays(as.Date(TotStepsInDay$Date))
ImputedTotStepsInDay <- TotStepsInDay %>%
  group_by(DayOfWeek) %>%
  mutate(x = ifelse(is.na(x),mean(x,na.rm=T),x))

ImputedIntervalPerDay<-aggregate(ImputedInterval$steps,
                                 by=list(Date=ImputedInterval$date),FUN=sum)
hist(ImputedIntervalPerDay$x,xlab="Steps",
     main="Histogram of Steps in a Day with Interval Imputing",
     breaks = c(0,5000,10000,15000,20000,25000))
```

![](PA1_template_files/figure-html/Imputing_missing_values-1.png)<!-- -->

```r
hist(ImputedTotStepsInDay$x,xlab="Steps",
     main="Histogram of Steps in a Day with Daily Imputing",
     breaks = c(0,5000,10000,15000,20000,25000))
```

![](PA1_template_files/figure-html/Imputing_missing_values-2.png)<!-- -->

The histograms show that while the shape of the histogram is largely retained, 
my imputing methods using mean values causes a higher peak between 10,000 and
15,000 steps in a day. This peak is more pronounced with interval imputing.
This can be further explored by calculating the mean and median 
total number of steps taken per day for each method in the code below.


```r
ImputedIntervalPerDayMean<-mean(ImputedIntervalPerDay$x)
ImputedIntervalPerDayMedian<-median(ImputedIntervalPerDay$x)

paste("The mean and median total number of steps taken per day with the interval imputing method are"
      ,ImputedIntervalPerDayMean,"and", 
      ImputedIntervalPerDayMedian,"respectively.")
```

```
## [1] "The mean and median total number of steps taken per day with the interval imputing method are 10766.1886792453 and 10766.1886792453 respectively."
```

```r
ImputedTotStepsInDayMean<-mean(ImputedTotStepsInDay$x)
ImputedTotStepsInDayMedian<-median(ImputedTotStepsInDay$x)

paste("The mean and median total number of steps taken per day with the daily imputing method are"
      ,ImputedTotStepsInDayMean,"and", 
      ImputedTotStepsInDayMedian,"respectively.")
```

```
## [1] "The mean and median total number of steps taken per day with the daily imputing method are 10821.2096018735 and 11015 respectively."
```

This shows that imputing the data caused the mean and median to stay pretty
similar, although the median is now the same as the mean in the case of
interval imputing.

The following block of code will create a new factor variable in the dataset 
with two levels – “Weekday” and “Weekend” indicating whether a given date is a 
weekday or weekend day.


```r
WeekDayFactor <- rep("Weekday", dim(ImputedInterval)[1])
ImputedInterval$DayOfWeek<-weekdays(as.Date(ImputedInterval$date))
FirstLetter<-substr(ImputedInterval$DayOfWeek,1, 1)
WeekendIndex<-FirstLetter=="S"
WeekDayFactor[WeekendIndex]<-"Weekend"
WeekDayFactor<-as.factor(WeekDayFactor)
ImputedInterval$EndOrDay<-WeekDayFactor
```

The next block of code will create a panel plot containing a time series plot of
the 5-minute interval (x-axis) and the average number of steps taken, averaged
across all weekday days or weekend days (y-axis).


```r
EndorDayTable <- ImputedInterval %>%
  group_by(EndOrDay,interval) %>%
  summarise(ave = mean(steps))

colnames(EndorDayTable)<-c("Weekend_or_Weekday", "Interval","Number_of_Steps")
x<-split(EndorDayTable,EndorDayTable$Weekend_or_Weekday)
Weekday<-x[[1]]
Weekend<-x[[2]]
par(mfrow = c(2,1))
with(Weekday,{
  plot(Interval,Number_of_Steps,type = "l",
       main = "Weekday average steps per 5 minute time interval")
})
with(Weekend,{
  plot(Interval,Number_of_Steps,type = "l",
       main = "Weekend average steps per 5 minute time interval")
})
```

![](PA1_template_files/figure-html/Creating_a_plot_showing_differences_in_activity_patterns-1.png)<!-- -->
