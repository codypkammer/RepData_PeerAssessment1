---
title: "Reproducible Research PA1"
author: "Cody"
date: "12/21/2021"
output: html_document
---



## Global settings, working directory, and needed package(s)


```r
knitr::opts_chunk$set(warning=FALSE)

setwd("G:/R/reproducible_research/RepData_PeerAssessment1")

library(ggplot2)
```

## Loading and processing the data

```r
PA1_data <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"

download.file(PA1_data, "./activity.zip")

unzip("activity.zip")

act <- read.csv("./activity.csv")

act$date <- as.POSIXct(act$date, "%Y-%m-%d")

weekday <- weekdays(act$date)

act <- cbind(act, weekday)
```

## What is the mean total number of steps taken per day?

```r
act_steps_total <- with(act, aggregate(steps, by = list(date),
                                       FUN = sum, na.rm = TRUE))

names(act_steps_total) <- c("date", "steps")
hist(act_steps_total$steps, main = "Total Number of Steps Taken Per Day",
     xlab = "Total Steps Taken Per Day", col = "green", ylim = c(0,20),
     breaks = seq(0,25000, by=2500))
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)


#### Calculate and report the mean and median of the total number of steps taken per day.

```r
mean(act_steps_total$steps)
```

```
## [1] 9354.23
```


```r
median(act_steps_total$steps)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

#### Make a time series plot (i.e. \color{red}{\verb|type = "l"|}type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
avg_act_daily <- aggregate(act$steps, by=list(act$interval),
                                    FUN=mean, na.rm=TRUE)

names(avg_act_daily) <- c("interval", "mean")
plot(avg_act_daily$interval, avg_act_daily$mean, type = "l",
     col="orange", lwd = 2, xlab="Time Period", ylab="Average Number of Steps",
     main="Average Number of Steps Per Time Period")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)


#### Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
avg_act_daily[which.max(avg_act_daily$mean), ]$interval
```

```
## [1] 835
```

## Imputing missing values
#### Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with \color{red}{\verb|NA|}NAs)

```r
sum(is.na(act$steps))
```

```
## [1] 2304
```

#### Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
imputed_steps <- avg_act_daily$mean[match(act$interval, avg_act_daily$interval)]
```

#### Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
act_imputed <- transform(act, steps = ifelse(is.na(act$steps),
                                             yes = imputed_steps,
                                             no = act$steps))
imputed_steps_total <- aggregate(steps ~ date, act_imputed, sum)
names(imputed_steps_total) <- c("date", "daily_steps")
```

#### Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
hist(imputed_steps_total$daily_steps, col = "green",
     xlab = "Total Steps Per Day", ylim = c(0,30),
     main = "Total Number of Steps Taken Each Day", breaks = seq(0,25000,by=2500))
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png)


```r
mean(imputed_steps_total$daily_steps)
```

```
## [1] 10766.19
```


```r
median(imputed_steps_total$daily_steps)
```

```
## [1] 10766.19
```

## Are there differences in activity patterns between weekdays and weekends?
#### Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
act$date <- as.Date(strptime(act$date, format="%Y-%m-%d"))
act$datetype <- sapply(act$date, function(x) {
  if (weekdays(x) == "Saturday" | weekdays(x) =="Sunday") 
  {y <- "Weekend"} else 
  {y <- "Weekday"}
  y
})
```

#### Make a panel plot containing a time series plot (i.e. \color{red}{\verb|type = "l"|}type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
act_by_day <- aggregate(steps~interval + datetype, act, mean, na.rm = TRUE)
ggplot(act_by_day, aes(x = interval , y = steps, color = datetype)) +
       geom_line() +
       labs(title = "Average Daily Steps by Type of Day", 
            x = "Time Period",
            y = "Average Number of Steps") +
  facet_wrap(~datetype, ncol = 1, nrow=2)
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15-1.png)
