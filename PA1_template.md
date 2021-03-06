Reproducible Research - Assignment 1
========================================================

## Part 1

First, we need to load the data


```r
data <- read.csv("activity.csv",stringsAsFactors=FALSE)
```

The loaded data look like this:


```r
head(data,10)
```

```
##    steps       date interval
## 1     NA 2012-10-01        0
## 2     NA 2012-10-01        5
## 3     NA 2012-10-01       10
## 4     NA 2012-10-01       15
## 5     NA 2012-10-01       20
## 6     NA 2012-10-01       25
## 7     NA 2012-10-01       30
## 8     NA 2012-10-01       35
## 9     NA 2012-10-01       40
## 10    NA 2012-10-01       45
```

## Part2 

Now make a histogram of the total number of steps taken each day and calculate the mean and median of the total number of steps taken per day. Ignore missing values.


```r
data.ex2 <- data[!is.na(data$steps),]
stepsByDay <- aggregate(steps ~ date, FUN=sum, data=data.ex2)
hist(stepsByDay$steps,breaks=10,main="Histogram of Number of Steps Taken Each Day", xlab="Number of Steps", ylab="Frequency")
```

![plot of chunk part2](figure/part2.png) 

The mean of the total number of steps taken per day is 1.0766 &times; 10<sup>4</sup>  

The median of the total number of steps taken per day is 10765

## Part3

In this section we are going to make a time series plot of the 5-minute interval and the average number of steps taken, averaged across all days.


```r
data.ex3 <- data
#posNA <- is.na(data.ex3$steps)
#data.ex3[posNA,"steps"] <- 0

sp <- split(data.ex3,data.ex3$interval)
y <- rep(0,length(sp))
x <- names(sp)

for(i in 1:length(sp))
{
    Z <- sp[[i]]$steps
    Z <- Z[!is.na(Z)]
    y[i] <- mean(Z)
}

plot(as.numeric(x),y,type="l",main="Interval vs. Average Number of Steps", xlab="Interval", ylab = "Averaged Number of Steps")
```

![plot of chunk part3](figure/part3.png) 

```r
maxStepsPos <- which.max(y)
```

The the fifth minute interval, on average across all the days in the dataset that contains the maximum number of steps is 835

## Part 4

Here we are to do the following tasks:

1. Calculate and report the total number of missing values in the dataset 

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

Bellow is the code that will help us to answer these questions.


```r
data.ex4 <- data
posStepsNA <- is.na(data.ex4$steps)
posDateNA <- is.na(data.ex4$date)
posIntervalNA <- is.na(data.ex4$interval)

amountOfNA <- sum(posStepsNA | posDateNA | posIntervalNA)

data.ex4$steps[posStepsNA] <- 0
data.ex4$date[posDateNA] <- "2012-01-01"
IntervalMeanForDate <- aggregate(interval ~ date, FUN=mean, data=data.ex4)
for(i in length(data.ex4$interval[posIntervalNA]))
{
    posDate <- IntervalMeanForDate$date == data.ex4$date[i]
    currentInterval <- IntervalMeanForDate$interval[posDate]
    data.ex4$interval[i] <- IntervalMeanForDate$interval[currentInterval[1]]
}

stepsByYear <- aggregate(steps ~ date, FUN=sum, data=data.ex4)
```

The total number of missing values in the dataset is 2304. 

To fill all the missing values in the data set we will follow the next extrategy:

* Missing Steps values will be filled with the value of 0.
* Missing Date values will be replaced with the date "2012-01-01"
* Missing Interval values will be replaced with the mean of the interval values of that day

The first elements of the new dataset looks as follows:


```r
head(data.ex4)
```

```
##   steps       date interval
## 1     0 2012-10-01        0
## 2     0 2012-10-01        5
## 3     0 2012-10-01       10
## 4     0 2012-10-01       15
## 5     0 2012-10-01       20
## 6     0 2012-10-01       25
```

Here is an histogram of the total number of steps by day. This time we can see that because the missing values were replaced by zero, the height of the bar comprising this value has a larger height than in the previous question.


```r
hist(stepsByYear$steps,breaks=10,main="Histogram of Number of Steps Taken Each Day", xlab="Number of Steps", ylab="Frequency")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

## Part 5

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

2. Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)

To complete this part of the assigment we used the following code.


```r
data.ex5 <- data

t <- as.Date(data.ex5$date,format="%Y-%m-%d")
wd <- weekdays(t,abbreviate=FALSE)

posWeekend <- wd == "Saturday" | wd == "Sunday"
posWeekday <- !posWeekend

data.ex5$day <- "weekday"
data.ex5$day[posWeekend] <- "weekend"

weekendData <- data.ex5[posWeekend,]
weekdayData <- data.ex5[posWeekday,]

spWeekend <- split(weekendData,weekendData$interval)
spWeekday <- split(weekdayData,weekdayData$interval)

yWeekend <- rep(0,length(spWeekend))
yWeekday <- rep(0,length(spWeekday))
xWeekend <- as.numeric(names(spWeekend))
xWeekday <- as.numeric(names(spWeekday))
weekendDay <- rep(0,length(spWeekend))
weekdayDay <- rep(0,length(spWeekday))

for(i in 1:length(spWeekend))
{
    Z <- spWeekend[[i]]$steps
    Z <- Z[!is.na(Z)]
    yWeekend [i] <- mean(Z)
    weekendDay[i] <- "weekend"
}

for(i in 1:length(spWeekday))
{
    Z <- spWeekday[[i]]$steps
    Z <- Z[!is.na(Z)]
    yWeekday [i] <- mean(Z)
    weekdayDay[i] <- "weekeday"
}

n <- length(spWeekend) + length(spWeekday)

finalData <- data.frame(x = numeric(n), y = numeric(n), day = character(n))
finalData$x <- append(xWeekday,xWeekend)
finalData$y <- append(yWeekday,yWeekend)
finalData$day <- append(weekdayDay, weekendDay)

#par(mfrow=c(2,1))
#plot(xWeekend,yWeekend,type="l", main="weekend", xlab="Interval", ylab="Average Number of Steps")
#plot(xWeekday,yWeekday,type="l", main="weekday", xlab="Interval",ylab="Average Number of Steps")
```

The new dataset with a new factor indicating if it is a weekend or a weekday looks like this:


```r
head(data.ex5,10)
```

```
##    steps       date interval     day
## 1     NA 2012-10-01        0 weekday
## 2     NA 2012-10-01        5 weekday
## 3     NA 2012-10-01       10 weekday
## 4     NA 2012-10-01       15 weekday
## 5     NA 2012-10-01       20 weekday
## 6     NA 2012-10-01       25 weekday
## 7     NA 2012-10-01       30 weekday
## 8     NA 2012-10-01       35 weekday
## 9     NA 2012-10-01       40 weekday
## 10    NA 2012-10-01       45 weekday
```

Following there is a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)


```r
library(lattice)
xyplot(y ~ x | day, data = finalData, layout = c(1,2), type="l", xlab="Interval", ylab="Number of Steps")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 
