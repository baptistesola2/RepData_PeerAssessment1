# Reproducible reserch : Peer Assessment 1
Baptiste Sola  
14 août 2017  

## Loading and preprocessing the data

We download the data from [Activity Data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

```r
# Download the zip file
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip","Activity monitoring data.zip")
#unzip the downloaded file 
unzip("Activity monitoring data.zip")
#read data
ActivityRawData <- read.csv("activity.csv")
ActivityRawData$date <- as.Date(ActivityRawData$date)
```

## What is mean total number of steps taken per day?
Here is the histogram of the number of total steps per day 

```r
StepDate <- aggregate(steps ~ date, ActivityRawData, sum )
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.3.3
```

```r
hist(StepDate$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

The mean number of step per day is 

```r
mean(StepDate$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

The median numbber of step per day is 

```r
median(StepDate$steps, na.rm = TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
Here's a look at the average daily pattern for number of steps :

```r
StepInterval <- aggregate(steps ~ interval, ActivityRawData, mean)

p2 <- ggplot(data  = StepInterval, aes(x= interval, y= steps))
p2+ geom_line (stat = "identity")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

The maximum average number of steps occurs at the inerval :

```r
StepInterval[StepInterval$steps ==max(StepInterval$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```

## Imputing missing values
We have the following number of missing values : 

```r
nrow(ActivityRawData[is.na(ActivityRawData$steps),])
```

```
## [1] 2304
```

Given that the number of step is more volatile per interval than days, we decide to fill the missing values with the average number of steps for the interval they're in.

Here's the histogram of number of step per day plus the mean and median number of step per days, based on the new data 

```r
ActivityPlusIntervalMean <- merge(ActivityRawData, StepInterval, by = "interval")
ActivityPlusIntervalMean [is.na(ActivityPlusIntervalMean$steps.x),]$steps.x <- ActivityPlusIntervalMean [is.na(ActivityPlusIntervalMean$steps.x),]$steps.y

StepDateMV <- aggregate(steps.x ~ date, ActivityPlusIntervalMean, sum )

hist(StepDateMV$steps.x)
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
mean(StepDateMV$steps.x, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(StepDateMV$steps.x, na.rm = TRUE)
```

```
## [1] 10766.19
```
As shown this transformation has little impact on the mean, but does have an impact on the histogram, as there is no "empty"" days with no steps anymore

## Are there differences in activity patterns between weekdays and weekends?

Here is the comparison between nb of step per day between weekdays and weekends :


```r
ActivityPlusIntervalMean$wk <- weekdays( ActivityPlusIntervalMean$date)
ActivityPlusIntervalMean[ActivityPlusIntervalMean$wk == "samedi"| ActivityPlusIntervalMean$wk == "dimanche",]$wk <- "Weekend"
ActivityPlusIntervalMean[! ActivityPlusIntervalMean$wk == "Weekend",]$wk <- "weekday"

StepIntervalwk <- aggregate(steps.x ~interval + wk, ActivityPlusIntervalMean, mean)
p4 <- ggplot(data  = StepIntervalwk, aes(x= interval, y= steps.x))
p4 + geom_line(stat = "identity")+ facet_grid(~wk)
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

The main difference we observe is a pic of #steps around interval 800 on weekdays that is completely absent on weekends
