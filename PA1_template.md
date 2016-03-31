# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

#### 1. Load the data (i.e. read.csv())
#### 2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
data <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

#### 1. Calculate the total number of steps taken per day
#### 2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
#### 3. Calculate and report the mean and median of the total number of steps taken per day

```r
total.steps <- tapply(data$steps, data$date, FUN=sum, na.rm=TRUE)

hist(total.steps, xlab="Total # of Steps Per Day", ylab="Frequency", main="Total Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)

```r
mean(total.steps, na.rm=TRUE)
```

```
## [1] 9354.23
```

```r
median(total.steps, na.rm=TRUE)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

#### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
#### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.2.4
```

```r
averages <- aggregate(x=list(steps=data$steps), by=list(interval=data$interval), FUN=mean, na.rm=TRUE)

ggplot(data=averages, aes(x=interval, y=steps)) + geom_line() + xlab("5 Minute Interval") + ylab("Average # of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)

```r
averages[which.max(averages$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```

## Imputing missing values

#### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
#### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
#### 3. Create a new dataset that is equal to the original dataset but with the missing data filled_data in.
#### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
missing <- is.na(data$steps)
table(missing)
```

```
## missing
## FALSE  TRUE 
## 15264  2304
```

```r
filling.value <- function(steps, interval) {
    filled_data <- NA
    if (!is.na(steps))
        filled_data <- c(steps)
    else
        filled_data <- (averages[averages$interval==interval, "steps"])
    return(filled_data)
}

filled_data.data <- data
filled_data.data$steps <- mapply(filling.value, filled_data.data$steps, filled_data.data$interval)

total.steps <- tapply(filled_data.data$steps, filled_data.data$date, FUN=sum)
hist(total.steps, xlab="Total # of Steps Per Day", ylab="Frequency", main="Total Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)

```r
mean(total.steps)
```

```
## [1] 10766.19
```

```r
median(total.steps)
```

```
## [1] 10766.19
```

Imputing missing data changes various values in terms of calculations, graphs, etc.

## Are there differences in activity patterns between weekdays and weekends?

#### 1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
#### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
weekday.or.weekend <- function(date) {
    day <- weekdays(date)
    if (day %in% c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday"))
        return("weekday")
    else if (day %in% c("Saturday", "Sunday"))
        return("weekend")
    else
        stop("Error")
}

filled_data.data$date <- as.Date(filled_data.data$date)
filled_data.data$day <- sapply(filled_data.data$date, FUN=weekday.or.weekend)

averages <- aggregate(steps ~ interval + day, data=filled_data.data, mean)
ggplot(averages, aes(interval, steps)) + geom_line() + xlab("5 Minute Interval") + ylab("# of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)
