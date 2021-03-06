---
title: Setting yearly targets with Deep Work Supervisor
author: Will Bidstrup
date: '2018-12-24'
slug: setting-yearly-targets-with-deep-work-supervisor
categories:
  - R
tags:
  - R
---


The Deep Work Supervisor tells you how much deep work you have been doing.  

Today I'm going to play around with setting targets. 

A big TODO is to check out [packrat](https://rstudio.github.io/packrat/).  



```r
library(here)
```

```
## here() starts at /Users/williambidstrup/Documents/GitHub/data_science_explorer
```

```r
library(tidyverse)
```

```
## ── Attaching packages ────────────────────────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✔ ggplot2 3.0.0     ✔ purrr   0.2.5
## ✔ tibble  1.4.2     ✔ dplyr   0.7.6
## ✔ tidyr   0.8.1     ✔ stringr 1.3.1
## ✔ readr   1.1.1     ✔ forcats 0.3.0
```

```
## ── Conflicts ───────────────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```

```r
library(padr)
```


Here is the year of 2019.  

First I will create the year with padr. 


```r
dates <- as.Date(c("2019-01-01", "2019-01-02", "2019-12-30", "2019-12-31")) 
dates <- as.data.frame(dates)
dates  <- dates %>%
  pad()
```

```
## pad applied on the interval: day
```

# Treating workdays differently to weekends and vacation


```r
# Identify the weekends
weekends <- dates$dates[weekdays(dates$dates) %in% c("Saturday", "Sunday")]  

# Identify public holidays
public_holidays <- as.Date(c("2019-01-01", "2019-01-28", "2019-03-11", "2019-04-19",
                             "2019-04-20", "2019-04-22", "2019-04-25", "2019-06-10",
                             "2019-10-07", "2019-12-24", "2019-12-25", "2019-12-26", "2019-12-31"))

# Identify vacation days x 20
vacations <- as.Date(c("2019-03-04", "2019-03-05", "2019-03-06", "2019-03-07", "2019-03-08",
                      "2019-07-01", "2019-07-02", "2019-07-03", "2019-07-04", "2019-07-05",
                      "2019-09-16", "2019-09-17", "2019-09-18", "2019-09-19", "2019-09-20",
                      "2019-11-04", "2019-11-05", "2019-11-06", "2019-11-07", "2019-11-08"))

# Create tags
dates$day_type <- ifelse(dates$dates %in% vacations, "vacation",
                         ifelse(dates$dates %in% public_holidays, "public_holiday", 
                                ifelse(dates$dates %in% weekends, "weekend", "work")))

summary(as.factor(dates$day_type))
```

```
## public_holiday       vacation        weekend           work 
##             13             20            103            229
```


# Setting target number of intervals

I like to do my deep work intervals at 25 minutes.  I need to choose if I will set and manage targets based on number of intervals or time elapsed.  Number of intervals probably makes the most sense, from there I can always convert to hours if needed.  


```r
# Study
# Wealth
# Craft
# Work
```


```r
# Study target
dates$study_target <- ifelse(dates$dates %in% vacations, 0,
                         ifelse(dates$dates %in% public_holidays, 0, 
                                ifelse(dates$dates %in% weekends, 2, 4)))

study_total_sessions <- sum(dates$study_target)
study_total_minutes <- study_total_sessions * 25
study_total_hours <- round(study_total_minutes / 60)
```


```r
# Wealth target
dates$wealth_target <- ifelse(dates$dates %in% vacations, 0,
                         ifelse(dates$dates %in% public_holidays, 0, 
                                ifelse(dates$dates %in% weekends, 2, 2)))

wealth_total_sessions <- sum(dates$wealth_target)
wealth_total_minutes <- wealth_total_sessions * 25
wealth_total_hours <- round(wealth_total_minutes / 60)
```


```r
# Craft target
dates$craft_target <- ifelse(dates$dates %in% vacations, 4,
                         ifelse(dates$dates %in% public_holidays, 0, 
                                ifelse(dates$dates %in% weekends, 4, 2)))

craft_total_sessions <- sum(dates$craft_target)
craft_total_minutes <- craft_total_sessions * 25
craft_total_hours <- round(craft_total_minutes / 60)
```


```r
# Work target
dates$work_target <- ifelse(dates$dates %in% vacations, 0,
                         ifelse(dates$dates %in% public_holidays, 0, 
                                ifelse(dates$dates %in% weekends, 0, 12)))

work_total_sessions <- sum(dates$work_target)
work_total_minutes <- work_total_sessions * 25
work_total_hours <- work_total_minutes / 60

work_deepwork_efficiency <- round(work_total_hours / 2080, digits = 2)
```


```r
# Total target
dates$total_target <- dates$study_target + dates$wealth_target + dates$craft_target + dates$work_target

total_deepwork_sessions <- sum(dates$total_target)
total_deepwork_minutes <- total_deepwork_sessions * 25
total_deepwork_hours <- total_deepwork_minutes / 60

total_deepwork_efficiency <- round(total_deepwork_hours / 2920, digits = 2)
```


# Tidy


```r
names(dates)
```

```
## [1] "dates"         "day_type"      "study_target"  "wealth_target"
## [5] "craft_target"  "work_target"   "total_target"
```

```r
dates_tidy <- dates %>%
  select( dates, day_type, study_target, wealth_target, craft_target, work_target) %>%
  gather(session_type, session_count, 3:6) %>%
  mutate(session_hours = round((session_count * 25) / 60))
```

# The intentional year


```r
ggplot(data = dates_tidy, aes(x = dates, y = session_hours, fill = session_type)) +
  geom_bar(stat = "identity") +
  theme_minimal()
```

<img src="/post/2018-12-24-setting-yearly-targets-with-deep-work-supervisor_files/figure-html/unnamed-chunk-11-1.png" width="672" />


```r
ggplot(data = dates_tidy, aes(x = dates, y = session_hours, fill = session_type)) +
  geom_bar(stat = "identity") +
  facet_grid(. ~ session_type) +
  theme_minimal()
```

<img src="/post/2018-12-24-setting-yearly-targets-with-deep-work-supervisor_files/figure-html/unnamed-chunk-12-1.png" width="672" />


```r
ggplot(data = dates_tidy %>%
         group_by(session_type) %>%
         summarise(hours = sum(session_hours)), aes(x = "", y = hours, fill = session_type)) +
  geom_bar(stat = "identity", position = "fill") +
  theme_minimal()
```

<img src="/post/2018-12-24-setting-yearly-targets-with-deep-work-supervisor_files/figure-html/unnamed-chunk-13-1.png" width="672" />

