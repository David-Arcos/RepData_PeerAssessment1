---
html_document:
  keep_md: yes
author: "David Arcos"
date: "2023-03-20"
title: 'Reproducible Investigation: Week 2 Course Project'
---

# Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the following site:

- **Dataset:** [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

- **steps:** Number of steps taking in a 5-minute interval (missing values are coded as NA)

- **date:** The date on which the measurement was taken in YYYY-MM-DD format

- **interval:** Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.


```r
library(ggplot2)
library(tidyverse)
```

## 1. Loading and preprocessing the data
First, we download the dataset available for this assingment from the link provided. For this, we create an empty folder to save the zip file, then we proceed to unzip this file, and load the csv file into the R environment. 

```r
if(!file.exists("./data")){dir.create("./data")}
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileUrl,destfile="./data/activity.zip",method="curl")
unzip(zipfile="./data/activity.zip",exdir="./data")
activity <- read.csv("./data/activity.csv")
activity$date <- as.Date(activity$date)
```

## 2. What is mean total number of steps taken per day?

#### 2.1. Calculate the total number of steps taken per day

For this part of the assignment, we ignore the missing values in the dataset by using `na.rm = TRUE`. From the data we can observe that the number of steps were recorded during 61 days, starting from October 1st to November 30 of 2012.

```r
daily_steps <- activity %>%
  group_by(date) %>%
  summarize(Sumsteps = sum(steps, na.rm = TRUE)) 
```

#### 2.2. Make a histogram of the total number of steps taken each day

```r
hist(daily_steps$Sumsteps, 
     main = "Histogram of steps taken per day",
     xlab="Steps", ylim = c(0,30))
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)

#### 2.3. Calculate and report the mean and median of the total number of steps taken per day

```r
# Mean
paste("The mean of the total steps taken per day is",
          round(mean(daily_steps$Sumsteps),digits = 2))
```

```
## [1] "The mean of the total steps taken per day is 9354.23"
```

```r
# Median
paste("The median of the total steps taken per day is",
            round(median(daily_steps$Sumsteps),digits = 2))
```

```
## [1] "The median of the total steps taken per day is 10395"
```

## 3. What is the average daily activity pattern?

#### 3.1. Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
steps_per_interval <- activity %>%
  group_by(interval) %>%
  summarize(Meansteps = mean(steps, na.rm = TRUE))
plot(x = steps_per_interval$interval,
     y = steps_per_interval$Meansteps,
     col="red", type="l", 
     xlab = "5 Minute Intervals", ylab = "Average Number of Steps",
     main = "Steps By Time Interval")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

#### 3.2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
paste("5-Minute Interval containing the most steps on average:",
          steps_per_interval$interval[which.max(steps_per_interval$Meansteps)]
)
```

```
## [1] "5-Minute Interval containing the most steps on average: 835"
```

```r
paste("Average steps for that interval:",
            round(max(steps_per_interval$Meansteps)))
```

```
## [1] "Average steps for that interval: 206"
```

## 4. Imputing missing values

#### 4.1. Calculate and report the total number of missing values in the dataset
The total number of NA by column. There are 2.304 NA that belong entirely to the `steps` column.

```r
colSums(is.na(activity))
```

```
##    steps     date interval 
##     2304        0        0
```

#### 4.2. Devise a strategy for filling in all of the missing values in the dataset.
    The strategy for imputing missing values consists of replacing each NA with the mean value for the corresponding interval. For this purpose, we will use a `for` loop to replace each NA in `activity$step` with the associated interval mean. Also, we will create a new data frame to replace the NA, which is a copy of the original `activity` data frame.


```r
activityNoNA <- activity 
for (i in 1:nrow(activity)){
  if(is.na(activity$steps[i])){
    activityNoNA$steps[i]<- steps_per_interval$Meansteps[activityNoNA$interval[i] == steps_per_interval$interval]
  }
}
```

#### 4.3. Make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
daily_steps_NoNA <- activityNoNA %>%
        group_by(date) %>%
        summarize(Sumsteps = sum(steps, na.rm = TRUE)) 

hist(daily_steps_NoNA$Sumsteps, main = "Histogram of Daily Steps", 
     col="lightblue", xlab="Steps")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png)

Let's compare the mean and median between the data frames with and without NA. As expected, the mean of steps is greater when we replace the NA, as we replaced them with the average of steps for each interval. This increases the median as well.

It's important to note that, when we replace the NA with the associated interval average, the mean and the median are the same value, meaning that the distribution is unbiased.

```r
mean_NA <- round(mean(daily_steps$Sumsteps), 2)
median_NA <- round(median(daily_steps$Sumsteps), 2)
mean_NoNA <- round(mean(daily_steps_NoNA$Sumsteps),2)
median_NoNA <- round(median(daily_steps_NoNA$Sumsteps),2)

NACompare <- data.frame(mean = c(mean_NA,mean_NoNA),median = c(median_NA,median_NoNA))
rownames(NACompare) <- c("With NA's", "Without NA's")
print(NACompare)
```

```
##                  mean   median
## With NA's     9354.23 10395.00
## Without NA's 10766.19 10766.19
```

## 5. Are there differences in activity patterns between weekdays and weekends?

#### 5.1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
activityNoNA$date <- as.Date(activityNoNA$date)
activityNoNA$day <- ifelse(weekdays(activityNoNA$date) %in% c("sábado", "domingo"), "weekend", "weekday")
activityNoNA$day <- as.factor(activityNoNA$day)
```

#### 5.2. Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
activityWeekday <- activityNoNA %>%
        filter(day == "weekday") %>%
        group_by(interval) %>%
        summarize(steps = mean(steps)) 
activityWeekday$day <- "weekday"

activityWeekend <- activityNoNA %>%
        filter(day == "weekend") %>%
        group_by(interval) %>%
        summarize(steps = mean(steps)) 
activityWeekend$day <- "weekend"

activity_week <- rbind(activityWeekday, activityWeekend)
activity_week$day <- as.factor(activity_week$day)

g <- ggplot (activity_week, aes (interval, steps))
g + geom_line() + facet_grid (day~.) + 
        labs(x = "Interval", 
             y = "Number of Steps",
             title  = "Average number of steps: Weekdays vs. Weekends") 
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13-1.png)

As we can observe, there is a slight difference between the steps taken during weekdays and weekends. For the weekdays, the average of the steps taken is approximately 37 steps, meanwhile the average of steps taken during weekends is approximately 42 steps per interval.

This could mean that during weekdays, fewer steps are taken as most people spend their time if an office/school, which can also mean that most of those steps are taken during the morining, which is reflected in that large peak observed in the upper part of the graph. On the other side, steps taken during the weekend are observed to be more consistent throughout the day.
