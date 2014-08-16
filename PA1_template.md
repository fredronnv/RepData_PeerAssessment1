# Reproducible Research: Peer Assessment 1

## Loading and preprocessing the data


```r
library("data.table")
data <- fread("activity.csv", sep=",", na.strings="NA")
```

## What is mean total number of steps taken per day?

For this assignment we ignore the missing values in the dataset.

### Total number of steps taken (histogram)

```r
total_steps <- sapply(split(data$steps, data$date), sum, na.rm = TRUE)
hist(total_steps)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

### What is the mean and median of the total number of steps taken per day?

```r
mean(total_steps)
```

```
## [1] 9354
```

```r
median(total_steps)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

```r
activity_pattern <- sapply(split(data$steps, data$interval), mean, na.rm = TRUE)
```

### Average daily activity pattern time series plot

```r
plot(labels(activity_pattern), activity_pattern,
	main='Average number of steps taken, averaged across all days',
	type='l',
	ylab='Number of Steps',
	xlab='interval',

)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

### 5-minute intervals with most steps on average

```r
labels(last(sort(activity_pattern)))
```

```
## [1] "835"
```


## Imputing missing values


### Calculate the total number of missing values

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

### Devise a strategy for filling in all missing values in the dataset

I have chosen to use the average number of steps taken in the 5-minute interval to fill in missinga values. We can use the previous activity_pattern object to access these.

I'm using a simple transform to create a new dataset with the average put in place of NA.


```r
new_dataset <- transform(data, steps = ifelse( is.na(steps), activity_pattern, steps) )
```

### Histogram of the total number of steps taken

```r
total_steps_newdata <- sapply(split(new_dataset$steps, new_dataset$date), sum, na.rm = TRUE)
hist(total_steps_newdata)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 

### Mean and Median

```r
mean(total_steps_newdata)
```

```
## [1] 10766
```

```r
median(total_steps_newdata)
```

```
## [1] 10766
```


## Are there differences in activity patterns between weekdays and weekends?

We're going to first alter the dataset we've just recently created, new_dataset, and indicate if a certain measurement is taken on a weekend or a weekday, using the factor labels "Weekend" and "Weekday".


```r
new_dataset$day <- factor(ifelse(weekdays(as.POSIXct(new_dataset$date)) %in% c("Saturday","Sunday"), 1, 2), labels=c("Weekend","Weekday"))
```

We're going to produce a panel plot showing the average number of steps taken in a 5-minute interval averaged across all weekday days or weekend days.


```r
library("lattice")
avg_dataset <- aggregate( steps ~ interval + day, data=new_dataset, FUN=mean)
attach(avg_dataset)
xyplot(steps~interval|day, layout=c(1,2), type='l')
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12.png) 
