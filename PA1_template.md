---
title: 'Reproducible Research: Peer Assessment 1'
  html_document:
    keep_md: yes
---


## Loading and preprocessing the data

```r
# 0. Installing and loading necessary libraries
packs <- c("ggplot2", "dplyr", "lubridate", "scales", "timeDate")
InstIfNec<-function (pack) {   # function for library installation if necessary
    if (!do.call(require,as.list(pack))) {
        do.call(install.packages,as.list(pack))
        do.call(require,as.list(pack)) 
        } else { do.call(require,as.list(pack)) } }
lapply(packs, InstIfNec)
```

```
## [[1]]
## [1] TRUE
## 
## [[2]]
## [1] TRUE
## 
## [[3]]
## [1] TRUE
## 
## [[4]]
## [1] TRUE
## 
## [[5]]
## [1] TRUE
```

```r
# 1. Load the data
unzip("./RepData_PeerAssessment1/activity.zip") # Unzipping the file
```

```
## Warning in unzip("./RepData_PeerAssessment1/activity.zip"): error 1 in
## extracting from zip file
```

```r
df <- read.csv2 ("./activity.csv", header=TRUE, sep=",")

# 2. Process the data into a suitable format  
## Converting date variable from factor to date class
df$date<-as.Date(as.character(df$date), "%Y-%m-%d")
##??? Converting intervals into day hours
df$hour <- sprintf("%04d", df$interval)
df$hour <- format(strptime(df$hour, format="%H%M"), format = "%H:%M")
```




## What is mean total number of steps taken per day?

```r
# 1. Calculate the total number of steps taken per day
dfsd<-dplyr::summarize(group_by(df, date), totSteps_day=sum(steps, na.rm=T))

# 2. Make a histogram of the total number of steps taken each day
ggplot(dfsd, aes(totSteps_day))+
    geom_histogram()+
    theme_bw()+
    ylab("Count")+ xlab("Total Steps Per Day")+
    ggtitle("Histogram of total number of steps per day")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk mean_step](figure/mean_step-1.png) 

```r
# 3. Calculate and report the mean and median of the total number of steps taken per day
meanSteps_day<-mean(mean(dfsd$totSteps_day, na.rm=T))
medianSteps_day<-mean(median(dfsd$totSteps_day, na.rm=T))
print(paste("The mean and the median number of steps per day are respectively:", 
            round(meanSteps_day, 0), "and", round(medianSteps_day, 0), sep=" "))
```

```
## [1] "The mean and the median number of steps per day are respectively: 9354 and 10395"
```




## What is the average daily activity pattern?

```r
#??? 1.Make a time series plot of the 5-minute interval and the average number of steps taken
## Sumarizing data by interval
dfsi<-dplyr::summarize(group_by(df, hour), meanSteps_5=mean(steps, na.rm=T)) 
dfsi$hour<-as.integer(as.duration((hm(dfsi$hour))))/3600
```

```
## estimate only: convert periods to intervals for accuracy
```

```r
## Plotting the results
ggplot(data=dfsi, aes(x=hour, y=meanSteps_5))+
    geom_line()+
    theme_bw()+xlim(0,24)+
    labs(x="Hour of the day", y="Daily activity (steps number in 5 minutes")+
    ggtitle("Daily activity pattern")
```

![plot of chunk daily_activity_pattern](figure/daily_activity_pattern-1.png) 

```r
# 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
time_max<-dfsi[which.max(dfsi$meanSteps_5),"hour"]
hm_max<- paste(floor(time_max), round((time_max-floor(time_max))*60), sep=":")
print(paste("The 5-minute interval which on average contains the maximum number of steps start at", hm_max))
```

```
## [1] "The 5-minute interval which on average contains the maximum number of steps start at 8:35"
```




## Imputing missing values

```r
# 1. Calculate and report the total number of missing values in the dataset 
missings<-sum(complete.cases(df))
print(paste("There is", missings, "missing values in the dataset"))
```

```
## [1] "There is 15264 missing values in the dataset"
```

```r
# 2. Devise a strategy for filling in all of the missing values in the dataset
## The easiest way to impute missing value is to fill missing value with the mean number of steps for that interval averaged over all days available

# 3. Create a new dataset that is equal to the original dataset but with the missing data filled in
df2<-df
df2$steps<-ifelse(is.na(df2$steps),dfsi$meanSteps_5,df2$steps)

# 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day
dfsd2<-dplyr::summarize(group_by(df2, date), totSteps_day=sum(steps, na.rm=T))
ggplot(dfsd2, aes(totSteps_day))+
    geom_histogram()+
    theme_bw()+
    ylab("Count")+ xlab("Total Steps Per Day")+
    ggtitle("Histogram of total number of steps per day with missing values imputation")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk imputing_missing_values](figure/imputing_missing_values-1.png) 

```r
meanSteps_day2<-mean(mean(dfsd2$totSteps_day, na.rm=T))
medianSteps_day2<-mean(median(dfsd2$totSteps_day, na.rm=T))
print(paste("The mean and the median number of steps per day with missing values imputation are respectively:", round(meanSteps_day2, 0), "and", 
            round(medianSteps_day2, 0), "which corresponds respectively to",
            round(meanSteps_day2-meanSteps_day,0), "and",
            round(medianSteps_day2-medianSteps_day,0),
            "steps of difference between the two dataset", sep=" "))
```

```
## [1] "The mean and the median number of steps per day with missing values imputation are respectively: 10766 and 10766 which corresponds respectively to 1412 and 371 steps of difference between the two dataset"
```




## Are there differences in activity patterns between weekdays and weekends?

```r
# 1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend"
df$daytype<-ifelse(isWeekday(df$date, wday=1:5)=="TRUE","Weekday","Weekend")

#??? 2.Make a panel plot containing a time series plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days
dfsi<-dplyr::summarize(group_by(df, daytype, hour), 
                       meanSteps_5=mean(steps, na.rm=T)) 
dfsi$hour<-as.integer(as.duration((hm(dfsi$hour))))/3600
```

```
## estimate only: convert periods to intervals for accuracy
```

```r
ggplot(data=dfsi, aes(x=hour, y=meanSteps_5))+
    geom_line()+
    theme_bw()+xlim(0,24)+
    labs(x="Hour of the day", y="Daily activity (steps number in 5 minutes)")+
    facet_grid(daytype~.)+
    ggtitle("Daily activity pattern during weekday and weekend")
```

![plot of chunk activity_pattern_dif](figure/activity_pattern_dif-1.png) 



