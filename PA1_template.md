---
title: 'Reproducible Research: Peer Assessment 1'
author: "Charles Smith"
date: "April 18, 2015"
output: html_document
---


## Introduction

This document responds to questions regarding data related to physical activity (measured in steps) over a two month period collected for an anonymous individual. The data used for this report was collected from a personal activity monitoring device that was designed to collect data at 5 minute intervals on a daily basis.

This document presents the results in a report using a R markdown document that can be processed by the knitr package and be transformed into an HTML file for detailed viewing. Please note, the report is designed (i.e., **echo = TRUE**) so that viewers will be able to read the related code as well.  

The report can be read as follows:  

## Prepare the R environment   

```r
library(knitr)
opts_chunk$set(echo = TRUE, results = 'hold')
```

## Load required libraries

```r
library(data.table)
library(lattice) 
library(ggplot2)
```


## Loading and preprocessing the data    

 


## Load the required data

The following statement is used to load the data using `read.csv()`.

**Please note**: It is assumed that the file activity.csv is in the current working directory. The corresponding data file can be downloaded from [here](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

1. Load and check the data 

```r
activity <- read.csv("activity.csv", colClasses = c("numeric", "character", "numeric"))
names(activity)

head(activity)
```

2. Process/transform the data into desired format for analysis

Convert the **date** field to `Date` class and **interval** field to `Factor` class.

```r
activity$date <- as.Date(activity$date, "%Y-%m-%d")
activity$interval <- as.factor(activity$interval)
```

Check the data using `str()` method:

```r
str(activity)
```

```
##data.frame':        17568 obs. of  3 variables:
## $ steps   : num  NA NA NA NA NA NA NA NA NA NA ...
## $ date    : Date, format: "2012-10-01" "2012-10-01" "2012-10-01" ...
## $ interval: Factor w/ 288 levels "0","5","10","15",..: 1 2 3 4 5 6 7 8 9 10 ...
```

## What is mean total number of steps taken per day(ignore missing values?

Calculating the total steps per day.

```r
avg_steps <- aggregate(steps ~ date, data=activity, sum, na.rm=TRUE)
colnames(avg_steps) <- c("date","steps")
head(avg_steps)
```

```
##         date steps
## 1 2012-10-02   126
## 2 2012-10-03 11352
## 3 2012-10-04 12116
## 4 2012-10-05 13294
## 5 2012-10-06 15420
## 6 2012-10-07 11015
```

Generate a histogram of the total number of steps taken per day, plotted with appropriate bin interval.

```r
ggplot(steps_per_day, aes(x = steps)) + 
       geom_histogram(fill = "green", binwidth = 5000) + 
        labs(title="Histogram of Steps Taken per Day", 
             x = "Number of Steps per Day", y = "Number of times in a day(Count)") + theme_bw() 
```

![plot of chunk histo](/figures/AvgStepsPerDay.png) 


Calculate the mean and median of the number of steps taken per day.

```r
mean_steps <- mean(avg_steps$steps, na.rm=TRUE)
mean_steps
median_steps <- (avg_steps$steps, na.rm=TRUE)
median_steps
```

The mean is **10766.19** and median is **10765**.

## What is the average daily activity pattern?

Prepare data for time series plot.

```r
steps_time_series <- tapply(activity$steps, activity$interval, mean, na.rm=TRUE)
steps_time_series$interval <- as.integer(levels(steps_time_series$interval)[steps_time_series$interval])
colnames(steps_time_series) <- c("interval", "steps")
```


Generate a time series plot of the average number of steps taken 

```r
ggplot(steps_time_series, aes(x=interval, y=steps)) +   
        geom_line(color="orange", size=1) +  
        labs(title="Average Daily Activity", x="Interval", y="Number of Steps") +  
        theme_bw()
```

![plot of chunk plot_time_series](/figures/AvgDailyActtivityPattern.png) 


Determine which interval contains the maximum number of steps.

```r
max_steps_interval <- steps_time_series[which.max(steps_time_series$steps),] 
max_steps_interval
```

Interval **835<sup>th</sup>** has the maximum in the amount of **206** steps.

## Imputing missing values:

Determine the total number of missing values in the data set.

```r
naTotal <- sum(is.na(activity))
naTotal
```

There are **2304** **missing values** from the data set.

## Develop a strategy for filling in all of the missing values in the dataset

Develop the code to update all missing values in the data set.

```r
avg_steps_int <- aggregate(steps ~ interval, data=activity, FUN=mean)
NA_update <- numeric()
for (i in 1:nrow(activity)) {
        cells <- activity[i, ]
        if (is.na(cells$steps)) {
                steps <- subset(avg_steps_int, interval == cells$interval)$steps
        } else {
                steps <- cells$steps
        }
        NA_update <- c(NA_update, steps)
}

activity2 <- activity
activity2$steps <- NA_update
str(activity2)
```

```
##'data.frame':        17568 obs. of  3 variables:
## $ steps   : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
## $ date    : Date, format: "2012-10-01" "2012-10-01" "2012-10-01" ...
## $ interval: Factor w/ 288 levels "0","5","10","15",..: 1 2 3 4 5 6 7 8 9 10 ...
```

Confirm that all missing values have been updated in the data set.

```r
sum(is.na(activity2$steps))
```

```
## [1] 0
```

All missing values have been updated resulting in no more missing values.


## Generate a histogram of the total number of steps taken each day

Generate a histogram based on the new data set (with no missing values).

```r
newTotSteps <- aggregate(steps ~ date, data = activity2, sum, na.rm = TRUE)
colnames(newTotSteps) <- c("date","steps")
hist(newTotSteps$steps, main = "Total Steps by Day", xlab = "Day", col = "green") 
```

![plot of chunk histo_fill](/figures/New Total Steps per Day.png) 

## Calculate and report the mean and median steps taken per day.

Calculate the mean and median steps taken per day.

```r
avg_steps2 <- mean(newTotSteps$steps, na.rm=TRUE)
avg_steps2
median_steps <- median(newTotSteps$steps, na.rm=TRUE)
median_steps
```

The mean is **10766.19** and median is **10766.19**.

##Determine if these values differ from the original values previously calculated

Do these values differ from the estimates from the first part of the assignment?

While the mean remained unchanged, the median did increase slightly (as shown below).

- **With missing values**          
    1. Mean  : **10766.19**           
    2. Median: **10765**                

-

- **With missing values updated**
    1. Mean  : **10766.19**
    2. Median: **10766.19** 
 

As stated above, the mean did not change after the missing values were update, while the median did increase slightly and is now equal to the mean.

## Detemrine if there are any differences in trends/patterns in the data between weekdays and weekends. 

Create new variable for weekday (i.e., day of the week) and weekday type (weekday vs weekend). 

```r
wkday <- weekdays(activity2$date)
wkday_type <- vector()
for (i in 1:nrow(activity2)) {
        if (wkday[i] == "Saturday") {
                wkday_type[i] <- "weekend"
        } else if (wkday[i] == "Sunday") {
                wkday[i] = "weekend"  
        } else {
                wkday_type[i] <- "weekday"
        }
}

Add the new weekday type variable to the new data set.

activity2$wkday_type <- wkday_type
activity2$date <- factor(activity2$wkday_type)
steps_per_day <- aggregate(steps ~ interval + wkday_type, data = activity2, mean)
names(steps_per_day) <- c("interval", "wkday_type", "steps")
```

Make a panel plot containing the time series plot of the five minute interval and the
average number of steps taken (across all weekday days or weekdend days).

```r
xyplot(steps ~ interval | wkday_type, steps_per_day, type = "l", layout = c(1, 2), 
       xlab = "5-min Interval", ylab = "Number of Steps", col="red")
```

![plot of chunk plot_weekdays](/figures/Wkday vs Wkend plot.png) 

Based on the above graph comparison, weekend days appear to show the greatest peak from all step intervals. Also, weekends activities appera to show more peaks than weekday days.
