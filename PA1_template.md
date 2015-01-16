# Reproducible Research: Peer Assessment 1
Robert Ross - 2015.01.18


## Loading and preprocessing the data


```r
  ## setup
  setwd("~/Coursera/5-Reproducible Research/Assignment1")
  library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
## 
## The following object is masked from 'package:stats':
## 
##     filter
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
  library(ggplot2)
  
  ## read the data files from the activity Dataset
  moves <- read.csv("./activity.csv")
  
  ## get a subset of rows without the NA values for steps
  moves.noNA <- moves[complete.cases(moves),]
  
  ## sum up total steps taken each day
  moves.daily <- moves.noNA %>% group_by(date) %>% summarise(sum(steps))
```

### What does the data look like?


```r
  ## show first 6 rows of data
  head(moves)  
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

## What is mean total number of steps taken per day?


```r
  ## make histogram of total number of steps per day
  names(moves.daily)[2]="steps"
  hist(moves.daily$steps, 
       main="Histogram of total number of steps per day", 
       xlab="Total steps per day",
       breaks=10)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

```r
  ## calculate mean and median for daily steps
  moves.daily.mean <- mean(moves.daily$steps)
  moves.daily.median <- median(moves.daily$steps)
```

#### The mean of the total number of steps taken per day is 1.0766189\times 10^{4}.


#### The median of the total number of steps taken per day is 10765.





## What is the average daily activity pattern?


```r
  ## figure out and plot the daily activity pattern
  moves.daily.activity <- aggregate(steps ~ interval, data=moves.noNA, mean)
  moves.daily.activity$interval <- as.character.Date(moves.daily.activity$interval)

  plot(moves.daily.activity$interval, moves.daily.activity$steps, 
       type="l", 
       main="Average number of steps for each 5-minute interval",
       xlab="5-minute interval during day", 
       ylab="Steps", 
       col="blue", 
       axes=F)
       axis(side=1, at=moves.daily.activity$interval)
       axis(side=2, at=round(moves.daily.activity$steps,0))
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

### Most active 5-minute interval



```r
  ## figure out the most active average 5-minute period in the day
  most.steps <- round(moves.daily.activity[moves.daily.activity$steps == max(moves.daily.activity$steps), 2],0)
  most.steps.time <- moves.daily.activity[moves.daily.activity$steps == max(moves.daily.activity$steps), 1]
```

#### The 5-minute interval with the most activity (average of 206)     is at the  835 interval.




## Imputing missing values


```r
  ## calculate number of rows in raw data with NA
  moves.NA <- nrow(moves) - nrow(moves.noNA)
```

#### The total number of missing values (NAs) in the dataset is 2304.



```r
  ## calculate average number of steps per interval for non-NA rows
  ave.steps.per.interval <- sum(moves.noNA$steps) / nrow(moves.noNA)

  ## make a copy of original data and replace NAs with ave steps/interval
  moves.replaceNA <- moves
  moves.replaceNA[is.na(moves.replaceNA)] <- ave.steps.per.interval

  ## redo the histogram, mean and median for new dataset with NAs replaced
  ## sum up total steps taken each day
  moves.daily.replaceNA <- moves.replaceNA %>% group_by(date) %>% summarise(sum(steps))

  ## make histogram of total number of steps per day with NAs replace
  names(moves.daily.replaceNA)[2]="steps"
  hist(moves.daily.replaceNA$steps, 
       main="Histogram of total number of steps per day", 
       xlab="Total steps per day when NAs replaced",
       breaks=10)
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png) 

```r
  ## calculate mean and median for daily steps
  moves.daily.replaceNA.mean <- mean(moves.daily.replaceNA$steps)
  moves.daily.replaceNA.median <- median(moves.daily.replaceNA$steps)

  ## calculate difference between this mean and median and the no-NA values
  mean.diff <- moves.daily.mean - moves.daily.replaceNA.mean
  median.diff <- moves.daily.median - moves.daily.replaceNA.median
```

#### The mean of the total number of steps taken per day is 1.0766189\times 10^{4}.


#### The median of the total number of steps taken per day is 1.0766189\times 10^{4}.


#### The difference between the mean of the total number of steps taken per day with no NA values vs. replacing the NAs with the average steps per interval is 0 (which is kind of expected:-).


#### The difference between the median of the total number of steps taken per day with no NA values vs. replacing the NAs with the average steps per interval is -1.1886792. This happens because several days have no values (all NAs), and therefore every interval gets replaced with the same average, causing all of these days to have the same total number of steps - so lots of the same "median" values to pick from.




## Are there differences in activity patterns between weekdays and weekends?