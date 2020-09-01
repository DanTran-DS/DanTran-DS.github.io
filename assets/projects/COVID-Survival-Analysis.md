---
layout: single
author_profile: true
title: Survival Analysis - Days to lockdown
excerpt: "Investigate county-level factors that affect the days it take for counties to lockdown"
tags: [R, Survival Analysis, EDA, Statistics]
comments: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/teaser/code.png
---

# 1\. Data

## Install packages

``` r
# install.packages("readxl")
# install.packages("tidyverse")
# install.packages("ggrepel")
# install.packages("geosphere")
# install.packages("survminer")
# install.packages("table1")
# install.packages('countrycode')
# install.packages("ggstatsplot")
# install.packages("ggcorrplot")
# install.packages("kableExtra")

require(table1)
require(survival)
require(survminer)
require(geosphere)
require(readxl)
require(tidyverse)
require(ggplot2)
require(ggrepel)
require(zoo)
require(kableExtra)
require(ggstatsplot)
```

## Load final data

remove rail due to missing data

``` r
data <- read.csv("data-final.csv", header=T)

data <- data %>% select(-X) %>% #mutate(first_case = as.Date(first_case),lockdown = as.Date(lockdown)) %>%
  rename(test = test1_to_1death, trace = trace1_to_1death, test.trace = test_or_trace_to_1death)

head(data)
```

    ##               Country alpha3Code first_case   lockdown censor case_on_lockdown
    ## 1         Afghanistan        AFG 2020-02-24 2020-04-15      0              784
    ## 2             Albania        ALB 2020-03-09 2020-03-13      1               33
    ## 3             Algeria        DZA 2020-02-25 2020-03-23      1              230
    ## 4             Andorra        AND 2020-03-02 2020-04-15      0              673
    ## 5              Angola        AGO 2020-03-20 2020-04-15      0               19
    ## 6 Antigua and Barbuda        ATG 2020-03-13 2020-04-15      0               23
    ##   days_to_lockdown    area population   region gini      Lat     Long death
    ## 1               51  652230   27657145     Asia 27.8  33.0000  65.0000    25
    ## 2                4   28748    2886026   Europe 34.5  41.1533  20.1683     1
    ## 3               27 2381741   40400000   Africa 35.3  28.0339   1.6596    17
    ## 4               44     468      78014   Europe   NA  42.5063   1.5218    33
    ## 5               26 1246700   25868000   Africa 58.6 -11.2027  17.8739     2
    ## 6               33     442      86295 Americas   NA  17.0608 -61.7964     2
    ##   life median_age   GDP airports rail   road unemployment test trace test.trace
    ## 1 52.8       19.5  2000       43   NA  34903         23.9  -30   -29        -29
    ## 2 79.0       34.3 12500        4  677   3945         13.8   -4    -4         -4
    ## 3 77.5       28.9 15200      157 3973 104000         11.7  -17    50         50
    ## 4 83.0       46.2 49900       NA   NA    320          3.7  -25     0          0
    ## 5 61.3       15.9  6800      176 2852  26000          6.6  -14    50          4
    ## 6 77.3       32.7 26400        3   NA   1170         11.0   NA    NA         NA

## Descriptive statistics

``` r
# All vars
# table1::table1(~days_to_lockdown  + censor + region + population + death + case_on_lockdown, data = data)
# table1::table1(~ life + GDP + gini + unemployment +median_age , data = data)
# table1::table1(~trace + test.trace + road + rail + airports, data = data)
#table1::table1(~test + trace + test.trace, data = data)

# Important vars
# table1(~days_to_lockdown +case_on_lockdown+death+censor+ region+ gini +GDP+unemployment, data = data,)
```

Mising test and trace 40 countries.

# 2\. EDA

Specify countries in the top and bottom in term of days to lockdown.

``` r
countries_of_interest <- subset(data,days_to_lockdown > 60 | case_on_lockdown < 2)
```

## 2.1 Scatter Plots

### Cases vs days \*\*\*

This plot is reasonnable since the the cases should increase as time
goes by.

``` r
ggplot(data, aes(days_to_lockdown, log(case_on_lockdown), 
                 color = region, shape=as.factor(censor))) +
  geom_point()+
  geom_text_repel(data=countries_of_interest, aes(label = Country))+
  theme_classic()
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

### Test vs days

``` r
countries_test.trace <- subset(data,(test&trace)>=50|days_to_lockdown >60)

ggplot(data, mapping = aes(days_to_lockdown,test, color = region,shape=as.factor(censor))) +
  geom_point()+
  geom_text_repel(data=countries_test.trace, aes(label = Country))+
  theme_classic()
```

    ## Warning: Removed 40 rows containing missing values (geom_point).

    ## Warning: Removed 1 rows containing missing values (geom_text_repel).

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

### Trace vs days

``` r
ggplot(data, mapping = aes(days_to_lockdown,trace, color = region,shape=as.factor(censor))) +
  geom_point()+
  geom_text_repel(data=countries_test.trace, aes(label = Country))+
  theme_classic()
```

    ## Warning: Removed 40 rows containing missing values (geom_point).

    ## Warning: Removed 1 rows containing missing values (geom_text_repel).

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

### Test or trace vs days

``` r
ggplot(data, mapping = aes(days_to_lockdown,test.trace, color = region,shape=as.factor(censor))) +
  geom_point()+
  geom_text_repel(data=countries_test.trace, aes(label = Country))+
  theme_classic()
```

    ## Warning: Removed 40 rows containing missing values (geom_point).

    ## Warning: Removed 1 rows containing missing values (geom_text_repel).

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

3 plots for trace, test and both are similar -\> use 1 of them

### Density vs days

``` r
ggplot(data, mapping = aes(days_to_lockdown,log(population/area), color = region,shape=as.factor(censor))) +
  geom_point()+
  geom_text_repel(data=countries_of_interest, aes(label = Country))+
  theme_classic()
```

    ## Warning: Removed 2 rows containing missing values (geom_point).

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

### Distance from China vs days

Countries close to china lockdown late? May due to other reasons.

``` r
# calculate distance from COVID origin (China)
get.distance <- function(x,y){
  dist = as.numeric(distm(c(x,y), c(112.2707,30.9756), fun = distHaversine))
  return(dist)
}

data$distance = mapply(get.distance, data$Long, data$Lat)

# Plot
ggplot(data, aes(days_to_lockdown, log(distance), color = region))+geom_point()+theme_classic()
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

### Death vs days \*\*\*

Let check death ratio.

Used death+1 since some countries has no death when lockdown Used
cases/death for easier interpretation

``` r
countries_death <- subset(data, censor == 1 &( death/case_on_lockdown >0.1 | 
                                days_to_lockdown > 55 |
                                days_to_lockdown < 2))

ggplot(data=data, mapping = aes(days_to_lockdown, log(death+1), color = region)) +
  geom_point()+
  geom_text_repel(data=countries_death, aes(label = Country))+
  theme_classic()
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

### Life vs days \*\*\* The plot indicates country with higher life exeptanacy tend to lockdown late.

Life expectancy may be a proxy for medical quality in those country. So
if life exp is low, the medical system is bad so they need to lock down
early since they know their hospital can’t handle it.

WE can see that Africa countries have low life exp.

``` r
countries_life <- subset(data,  (censor = 1) & (life >85 | life <60 | 
                                days_to_lockdown > 55 |
                                days_to_lockdown < 2))

ggplot(data, mapping = aes(days_to_lockdown,life, color = region)) +
  geom_point()+
  geom_text_repel(data=countries_life, aes(label = Country))+
  theme_classic()
```

    ## Warning: Removed 1 rows containing missing values (geom_point).

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

### GDP vs days \*\*\*

Here, high GDP countries are late to response. Maybe they are more
confident on their ability to deal with the disease or just don’t want
to lockdown since it affects their economy.

``` r
countries_gdp <- subset(data, (censor = 1) &(log(GDP) >11.5 | log(GDP) <7 | 
                                days_to_lockdown > 60 |
                                days_to_lockdown < 2))


ggplot(data, mapping = aes(days_to_lockdown,log(GDP), color = region)) +
  geom_point()+
  geom_text_repel(data=countries_gdp, aes(label = Country))+
  theme_classic()
```

    ## Warning: Removed 3 rows containing missing values (geom_point).

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

### Unemployment vs days

No pattern

``` r
ggplot(data, mapping = aes(days_to_lockdown,log(unemployment+1), color = region)) +
  geom_point()+
  geom_text_repel(data=countries_of_interest, aes(label = Country))+
  theme_classic()
```

    ## Warning: Removed 10 rows containing missing values (geom_point).

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

## 2.2 Boxplots

### days by region

``` r
ggplot(data, aes( y=days_to_lockdown,  fill = region)) + 
  geom_boxplot()+
  theme_classic()
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->

This one same as above but with more information

``` r
# ggstatsplot::ggbetweenstats(data = data, 
#                x = region,
#                y = days_to_lockdown,
#                outlier.tagging = TRUE,
#                outlier.label = Country,plot.type = "box",results.subtitle = F,)
```

Asia has a higher median than the rest. 2 outliers Egypt and Australia

## 2.3 Histogram

### Days

``` r
ggplot(data, aes(x=days_to_lockdown)) + geom_histogram(colour="black", fill="steelblue4") + theme_bw()
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->

### Population

``` r
ggplot(data, aes(x=log(population))) + geom_histogram(binwidth=1, color="black",fill="springgreen4")+ theme_bw()
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->

## 2.4 Correlation Heatmap

``` r
#install.packages("ggcorrplot")

data.X <- data %>% select( population ,death,
                      life , GDP , gini , unemployment , 
                       median_age ,test ,trace ,test.trace ,road,rail,airports)

correlation.table <- round(cor(data.X,use = "complete.obs"), 1)

ggcorrplot::ggcorrplot(correlation.table, method = "circle", type = "lower",
     outline.col = "white")
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->

We can see that rail, road and airports are highly correlated \_\>
should only use 1 in a model. -\> use airports since it is less corr
with other vars.

Test and trace also correlated -\> pick 1

Life, median\_age, GDP also -\> pick 1

Values for appendix

``` r
correlation.table
```

    ##              population death life  GDP gini unemployment median_age test trace
    ## population          1.0   0.0 -0.1 -0.1  0.0         -0.1       -0.1 -0.1  -0.1
    ## death               0.0   1.0  0.1  0.1 -0.1          0.0        0.1  0.1   0.0
    ## life               -0.1   0.1  1.0  0.7 -0.5         -0.4        0.8 -0.2  -0.2
    ## GDP                -0.1   0.1  0.7  1.0 -0.4         -0.3        0.7 -0.1  -0.2
    ## gini                0.0  -0.1 -0.5 -0.4  1.0          0.2       -0.5  0.1   0.1
    ## unemployment       -0.1   0.0 -0.4 -0.3  0.2          1.0       -0.4  0.1   0.1
    ## median_age         -0.1   0.1  0.8  0.7 -0.5         -0.4        1.0 -0.2  -0.1
    ## test               -0.1   0.1 -0.2 -0.1  0.1          0.1       -0.2  1.0   0.5
    ## trace              -0.1   0.0 -0.2 -0.2  0.1          0.1       -0.1  0.5   1.0
    ## test.trace         -0.1   0.0 -0.2 -0.1  0.1          0.1       -0.1  0.5   0.9
    ## road                0.7   0.0  0.1  0.2  0.0         -0.1        0.1 -0.1  -0.2
    ## rail                0.4   0.1  0.2  0.3  0.0         -0.1        0.2  0.0  -0.2
    ## airports            0.3   0.0  0.1  0.2  0.2         -0.1        0.1  0.0  -0.1
    ##              test.trace road rail airports
    ## population         -0.1  0.7  0.4      0.3
    ## death               0.0  0.0  0.1      0.0
    ## life               -0.2  0.1  0.2      0.1
    ## GDP                -0.1  0.2  0.3      0.2
    ## gini                0.1  0.0  0.0      0.2
    ## unemployment        0.1 -0.1 -0.1     -0.1
    ## median_age         -0.1  0.1  0.2      0.1
    ## test                0.5 -0.1  0.0      0.0
    ## trace               0.9 -0.2 -0.2     -0.1
    ## test.trace          1.0 -0.1 -0.2     -0.1
    ## road               -0.1  1.0  0.9      0.8
    ## rail               -0.2  0.9  1.0      0.9
    ## airports           -0.1  0.8  0.9      1.0

# 3\. Survival analysis

We delete rail due to missing data and high correlation with road and
airport. We remove observations missing data -\> have 110 countries
left. ALots of missing (40) due too test.trace variable.

R packages:

survival for computing survival analyses survminer for visualizing
survival analysis results

## Censoring

Survival analysis focuses on the expected duration of time until
occurrence of an event of interest. However, the event may not be
observed for some individuals within the study time period.

We have a dataset for lockdown date, we assume that countries that are
not in the dataset is not locked down. Thus is right censored.

## 3.1 Univariate Models

### region as predictor

``` r
# Modify data
#data$censor = 1
data$region <- factor(data$region)
data <- na.omit(data %>% select(-rail))

# Fit using the Kaplan-Meier method
surv.obj <- Surv(time = data$days_to_lockdown, event = as.numeric(data$censor));
region.mod  <- survfit( surv.obj ~ region, data = data);
#summary(region.mod)
#summary(region.mod)$table # consine version

ggsurvplot(region.mod, data = data, pval = TRUE,xlab='Days', ylab='Lockdown Probability')
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

#### Log Rank Test

``` r
# Log Rank Test of Equality of Survival Distributions over groups
survdiff(surv.obj ~ region, data = data)
```

    ## Call:
    ## survdiff(formula = surv.obj ~ region, data = data)
    ## 
    ##                  N Observed Expected (O-E)^2/E (O-E)^2/V
    ## region=Africa   28        8    14.04    2.5961     3.511
    ## region=Americas 21       15     9.31    3.4770     4.216
    ## region=Asia     26       17    25.05    2.5850     4.540
    ## region=Europe   32       28    19.11    4.1390     5.951
    ## region=Oceania   3        2     2.50    0.0998     0.108
    ## 
    ##  Chisq= 13.6  on 4 degrees of freedom, p= 0.009

The log-rank p-value of 0.035 indicates a significant result. The region
affects the time to lockdown. Asia countries are less likely to lockdown
early.

### population as predictor

``` r
# Based on histogram -> split into 3 groups
data$pop.size <- ifelse(log(data$population)>18,"large",
                        ifelse(log(data$population)<15,'small','medium'))
# Fit on population size
pop.mod  <- survfit( surv.obj ~ pop.size, data = data);
#summary(pop.mod)

ggsurvplot(pop.mod, data = data, pval = TRUE,xlab='Days', ylab='Lockdown Probability') 
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-23-1.png)<!-- -->

Not Significant.

### Use Strata

``` r
strata.mod  <- survfit( surv.obj ~ region + strata(pop.size), data = data);
ggsurvplot(strata.mod, data = data, pval = TRUE,xlab='Days', ylab='Lockdown Probability')
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-24-1.png)<!-- -->

``` r
survdiff(surv.obj ~ region + strata(pop.size), data = data) 
```

    ## Call:
    ## survdiff(formula = surv.obj ~ region + strata(pop.size), data = data)
    ## 
    ##                  N Observed Expected (O-E)^2/E (O-E)^2/V
    ## region=Africa   28        8    13.55      2.27     3.179
    ## region=Americas 21       15     9.28      3.52     4.365
    ## region=Asia     26       17    24.38      2.23     4.021
    ## region=Europe   32       28    20.16      3.05     4.546
    ## region=Oceania   3        2     2.63      0.15     0.168
    ## 
    ##  Chisq= 11.9  on 4 degrees of freedom, p= 0.02

Still good.Region is important.

### gini as predictor

``` r
gini.mod <- coxph(surv.obj ~ gini, data = data);
summary(gini.mod)
```

    ## Call:
    ## coxph(formula = surv.obj ~ gini, data = data)
    ## 
    ##   n= 110, number of events= 70 
    ## 
    ##         coef exp(coef) se(coef)     z Pr(>|z|)
    ## gini 0.00859   1.00863  0.01377 0.624    0.533
    ## 
    ##      exp(coef) exp(-coef) lower .95 upper .95
    ## gini     1.009     0.9914    0.9818     1.036
    ## 
    ## Concordance= 0.506  (se = 0.045 )
    ## Likelihood ratio test= 0.38  on 1 df,   p=0.5
    ## Wald test            = 0.39  on 1 df,   p=0.5
    ## Score (logrank) test = 0.39  on 1 df,   p=0.5

Not significant

### test.trace as predictor

``` r
test.mod <- coxph(surv.obj ~ test.trace, data = data);
summary(test.mod)
```

    ## Call:
    ## coxph(formula = surv.obj ~ test.trace, data = data)
    ## 
    ##   n= 110, number of events= 70 
    ## 
    ##                coef exp(coef) se(coef)     z Pr(>|z|)
    ## test.trace 0.006962  1.006986 0.004430 1.571    0.116
    ## 
    ##            exp(coef) exp(-coef) lower .95 upper .95
    ## test.trace     1.007     0.9931    0.9983     1.016
    ## 
    ## Concordance= 0.553  (se = 0.039 )
    ## Likelihood ratio test= 2.41  on 1 df,   p=0.1
    ## Wald test            = 2.47  on 1 df,   p=0.1
    ## Score (logrank) test = 2.47  on 1 df,   p=0.1

Not significant

## 3.2 Multivariate Models

### Check linear assumption

Compare continous variables to martingale residual loess line

``` r
loessline <- function(y, x) {
  x.list <- min(x) + ((0:100)/100)*(max(x) - min(x))
  y.x <- predict(loess(y ~ x), se=T, newdata=(x=x.list))
  lines(y.x$fit ~ x.list, lwd=2, col="red")
}
```

Here is untransformed

``` r
coxph.null <- coxph(surv.obj ~ 1, data=data)
resid.m.null <- residuals(coxph.null, type="martingale")

par(mfrow=c(2,2))
plot(resid.m.null ~ gini,data, ylab="Martingale Residual")
loessline(resid.m.null, data$gini)

plot(resid.m.null ~ life,data, ylab="Martingale Residual")
loessline(resid.m.null, data$life)

plot(resid.m.null ~ median_age,data, ylab="Martingale Residual")
loessline(resid.m.null, data$median_age)

plot(resid.m.null ~ GDP,data, ylab="Martingale Residual")
loessline(resid.m.null, data$GDP)
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-28-1.png)<!-- -->

``` r
plot(resid.m.null ~ population,data, ylab="Martingale Residual")
loessline(resid.m.null, data$population)

plot(resid.m.null ~ test.trace,data, ylab="Martingale Residual")
loessline(resid.m.null, data$test.trace)

plot(resid.m.null ~ airports,data, ylab="Martingale Residual")
loessline(resid.m.null, data$airports)
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-28-2.png)<!-- -->

Issues with life, population, airports, GDP.

DO log transform. New plot.

``` r
par(mfrow=c(2,2))

plot(resid.m.null ~ log(population),data)
loessline(resid.m.null, log(data$population))

plot(resid.m.null ~ log(GDP),data)
loessline(resid.m.null, log(data$GDP))

plot(resid.m.null ~ log(airports),data)
loessline(resid.m.null, log(data$airports))

plot(resid.m.null ~ log(life),data)
loessline(resid.m.null, log(data$life))
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-29-1.png)<!-- -->

GDP and Population look better.

Big issue with life. Tried some other transform -\> didn’t work. Use it
as categorical.

``` r
data$life.grp <- as.factor(ifelse(data$life<70, "Low",ifelse(data$life>80, "High","Medium")))

ggplot(data, aes( y=days_to_lockdown,  fill = life.grp)) + 
  geom_boxplot()+
  theme_classic()
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-30-1.png)<!-- -->

``` r
# ggstatsplot::ggbetweenstats(data = data, 
#                x = life.grp,
#                y = days_to_lockdown,
#                outlier.tagging = TRUE,
#                outlier.label = Country,plot.type = "box",results.subtitle = F,)
```

### Edit data

``` r
#data <- na.omit(data %>% select(-rail))
#surv.obj <- Surv(time = data$days_to_lockdown, event = data$censor);
#hist(log(data$death))
#data$death.count <- as.factor(ifelse(log(data$death)<2, "Low",ifelse(log(data$death)>5, "High","Medium")))
```

### 3.2.1 Full Model

This model includes all variables.

``` r
# Fit a Cox proportional hazards model on death, region and population
#data$death.rate = data$death / data$case_on_lockdown
full.mod <- coxph(surv.obj ~ region + log(population)  +
                      life.grp + log(GDP) + gini + unemployment + 
                       median_age +test +trace +test.trace +road+ log(airports), 
                   data = data);
ggforest(full.mod, data = data)
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-33-1.png)<!-- -->

Region is significant. May have collinearity.

### 3.2.2 Reduced Model

pop.size was unsignificant in univariate models -\> use continuous var :
log(population)

we include those and add some more. Based on corelation table, try to
avoid some combination (ie airports + road)

We take all variables that are signifiant in the full model.

``` r
reduced.mod <- coxph(surv.obj ~ region + log(population) 
                       + log(GDP) + life.grp 
                        +test.trace + log(airports), 
                   data = data);
ggforest(reduced.mod, data = data)
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-34-1.png)<!-- -->

### Stepwise

Use to pick best AIC model.

``` r
step.mod <- step(full.mod, scope=list(upper=~ region + log(population)  +
                      life.grp + log(GDP) + gini + unemployment + 
                       median_age +test +trace +test.trace +road+ log(airports), lower=~region), trace = F)

step.mod
```

    ## Call:
    ## coxph(formula = surv.obj ~ region + log(population) + life.grp + 
    ##     log(GDP) + test.trace + log(airports), data = data)
    ## 
    ##                      coef exp(coef)  se(coef)      z       p
    ## regionAmericas   1.380978  3.978790  0.583152  2.368 0.01788
    ## regionAsia      -0.280642  0.755299  0.531877 -0.528 0.59775
    ## regionEurope     0.866320  2.378144  0.539836  1.605 0.10854
    ## regionOceania    1.042787  2.837114  0.909701  1.146 0.25167
    ## log(population)  0.277754  1.320161  0.137890  2.014 0.04398
    ## life.grpLow      1.312353  3.714905  0.763491  1.719 0.08564
    ## life.grpMedium   1.044216  2.841169  0.405012  2.578 0.00993
    ## log(GDP)         0.695530  2.004770  0.270734  2.569 0.01020
    ## test.trace       0.010198  1.010250  0.004892  2.085 0.03711
    ## log(airports)   -0.331641  0.717745  0.143560 -2.310 0.02088
    ## 
    ## Likelihood ratio test=28.32  on 10 df, p=0.001607
    ## n= 110, number of events= 70

``` r
AIC(step.mod)
```

    ## [1] 532.2634

Model from step function agree with our choice.

### 3.3.3 Final Model

``` r
final.mod <- step.mod
# final.mod <- coxph(surv.obj ~ region + test.trace  + pop.size, 
#                    data = data);

summary(final.mod)
```

    ## Call:
    ## coxph(formula = surv.obj ~ region + log(population) + life.grp + 
    ##     log(GDP) + test.trace + log(airports), data = data)
    ## 
    ##   n= 110, number of events= 70 
    ## 
    ##                      coef exp(coef)  se(coef)      z Pr(>|z|)   
    ## regionAmericas   1.380978  3.978790  0.583152  2.368  0.01788 * 
    ## regionAsia      -0.280642  0.755299  0.531877 -0.528  0.59775   
    ## regionEurope     0.866320  2.378144  0.539836  1.605  0.10854   
    ## regionOceania    1.042787  2.837114  0.909701  1.146  0.25167   
    ## log(population)  0.277754  1.320161  0.137890  2.014  0.04398 * 
    ## life.grpLow      1.312353  3.714905  0.763491  1.719  0.08564 . 
    ## life.grpMedium   1.044216  2.841169  0.405012  2.578  0.00993 **
    ## log(GDP)         0.695530  2.004770  0.270734  2.569  0.01020 * 
    ## test.trace       0.010198  1.010250  0.004892  2.085  0.03711 * 
    ## log(airports)   -0.331641  0.717745  0.143560 -2.310  0.02088 * 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ##                 exp(coef) exp(-coef) lower .95 upper .95
    ## regionAmericas     3.9788     0.2513    1.2687    12.478
    ## regionAsia         0.7553     1.3240    0.2663     2.142
    ## regionEurope       2.3781     0.4205    0.8255     6.851
    ## regionOceania      2.8371     0.3525    0.4770    16.874
    ## log(population)    1.3202     0.7575    1.0075     1.730
    ## life.grpLow        3.7149     0.2692    0.8319    16.589
    ## life.grpMedium     2.8412     0.3520    1.2845     6.284
    ## log(GDP)           2.0048     0.4988    1.1793     3.408
    ## test.trace         1.0102     0.9899    1.0006     1.020
    ## log(airports)      0.7177     1.3933    0.5417     0.951
    ## 
    ## Concordance= 0.693  (se = 0.038 )
    ## Likelihood ratio test= 28.32  on 10 df,   p=0.002
    ## Wald test            = 28.38  on 10 df,   p=0.002
    ## Score (logrank) test = 30.25  on 10 df,   p=8e-04

``` r
ggforest(final.mod, data = data)
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-36-1.png)<!-- -->

# 4\. Model Diagnostic

We checked 3 things

  - Plot Schoenfeld residuals for each covariate, vs Time or log(Time)
    to check proportional hazards (PH)

  - Plot deviance residuals vs Observation \# to check for outliers

  - Plot martingale residuals vs continuous covariates to check
    functional form of covariates

## Proportional hazards assumption

The Schoenfeld residuals should be independent of time. A plot that
shows a non-random pattern against time is evidence of violation of the
PH assumption.

### Test

We focuss in the Harrell (1986), a variation of a test originally
proposed by Schoenfeld (1982). This is a test of correlation between the
Schoenfeld residuals and survival time. A correlation of zero indicates
that the model met the proportional hazards assumption (the null
hypothesis). Goodness-of-fit test

The function cox.zph() test for independence between residuals and time
for each covariate, and for the model as a whole.

``` r
(test <- cox.zph(final.mod))
```

    ##                  chisq df    p
    ## region          2.8187  4 0.59
    ## log(population) 1.2938  1 0.26
    ## life.grp        1.6973  2 0.43
    ## log(GDP)        2.1900  1 0.14
    ## test.trace      0.0835  1 0.77
    ## log(airports)   1.3543  1 0.24
    ## GLOBAL          7.4654 10 0.68

The test is not statistically significant so we can assume the
proportional hazards.

Re check with log time

``` r
cox.zph(final.mod, transform="log")
```

    ##                  chisq df     p
    ## region           5.299  4 0.258
    ## log(population)  1.451  1 0.228
    ## life.grp         3.718  2 0.156
    ## log(GDP)         5.292  1 0.021
    ## test.trace       0.109  1 0.741
    ## log(airports)    0.994  1 0.319
    ## GLOBAL          10.901 10 0.365

OK

### Graphical Test

We graph the scaled Schoenfeld residuals against the transformed time
for each covariate.

``` r
ggcoxzph(test,);
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-39-1.png)<!-- -->

the solid line is a smoothing spline fit to the plot, with the dashed
lines representing a +/- 2-standard-error band around the fit.

There is no pattern with time so the assumption of proportional hazards
holds.

## Influential obs

we visualize the deviance residuals and the dfbeta values

``` r
ggcoxdiagnostics(final.mod, type = "dfbeta", linear.predictions = F)
```

    ## `geom_smooth()` using formula 'y ~ x'

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-40-1.png)<!-- -->

OK. DFBETAS \< 1

``` r
# dfb <- residuals(final.mod,type="dfbeta")
# 
# which(abs(dfb[,7]) > 1)
# data[c(which(abs(dfb[,7]) > 1)),c("Country","death.rate","death")]
```

Some points need to be investigated.

## Outlier

``` r
ggcoxdiagnostics(final.mod, type = "deviance",
                 linear.predictions = FALSE, ggtheme = theme_bw())
```

    ## `geom_smooth()` using formula 'y ~ x'

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-42-1.png)<!-- -->

Look good.

## Linear test

Nonlinearity is not an issue for categorical variables,

Plotting the Martingale residuals against continuous covariates

This might help to properly choose the functional form of continuous
variable in the Cox model. Fitted lines with lowess function should be
linear to satisfy the Cox proportional hazards model assumptions.

``` r
coxph.null <- coxph(surv.obj ~ 1, data=data)
resid.m.null <- residuals(coxph.null, type="martingale")

par(mfrow=c(2,2))

plot(resid.m.null ~ log(GDP),data, ylab="Martingale Residual")
loessline(resid.m.null, log(data$GDP))

plot(resid.m.null ~ log(population),data, ylab="Martingale Residual")
loessline(resid.m.null, log(data$population))

plot(resid.m.null ~ test.trace,data, ylab="Martingale Residual")
loessline(resid.m.null, data$test.trace)

plot(resid.m.null ~ log(airports),data, ylab="Martingale Residual")
loessline(resid.m.null, log(data$airports))
```

![](days-to-lockdown-ver4_files/figure-gfm/unnamed-chunk-43-1.png)<!-- -->

All vars look ok.

# 5\. Model Inference

``` r
summary(final.mod)
```

    ## Call:
    ## coxph(formula = surv.obj ~ region + log(population) + life.grp + 
    ##     log(GDP) + test.trace + log(airports), data = data)
    ## 
    ##   n= 110, number of events= 70 
    ## 
    ##                      coef exp(coef)  se(coef)      z Pr(>|z|)   
    ## regionAmericas   1.380978  3.978790  0.583152  2.368  0.01788 * 
    ## regionAsia      -0.280642  0.755299  0.531877 -0.528  0.59775   
    ## regionEurope     0.866320  2.378144  0.539836  1.605  0.10854   
    ## regionOceania    1.042787  2.837114  0.909701  1.146  0.25167   
    ## log(population)  0.277754  1.320161  0.137890  2.014  0.04398 * 
    ## life.grpLow      1.312353  3.714905  0.763491  1.719  0.08564 . 
    ## life.grpMedium   1.044216  2.841169  0.405012  2.578  0.00993 **
    ## log(GDP)         0.695530  2.004770  0.270734  2.569  0.01020 * 
    ## test.trace       0.010198  1.010250  0.004892  2.085  0.03711 * 
    ## log(airports)   -0.331641  0.717745  0.143560 -2.310  0.02088 * 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ##                 exp(coef) exp(-coef) lower .95 upper .95
    ## regionAmericas     3.9788     0.2513    1.2687    12.478
    ## regionAsia         0.7553     1.3240    0.2663     2.142
    ## regionEurope       2.3781     0.4205    0.8255     6.851
    ## regionOceania      2.8371     0.3525    0.4770    16.874
    ## log(population)    1.3202     0.7575    1.0075     1.730
    ## life.grpLow        3.7149     0.2692    0.8319    16.589
    ## life.grpMedium     2.8412     0.3520    1.2845     6.284
    ## log(GDP)           2.0048     0.4988    1.1793     3.408
    ## test.trace         1.0102     0.9899    1.0006     1.020
    ## log(airports)      0.7177     1.3933    0.5417     0.951
    ## 
    ## Concordance= 0.693  (se = 0.038 )
    ## Likelihood ratio test= 28.32  on 10 df,   p=0.002
    ## Wald test            = 28.38  on 10 df,   p=0.002
    ## Score (logrank) test = 30.25  on 10 df,   p=8e-04

Beased on final model,

  - Countries in Americas has nearly 4 times the HR, exp(coef)=3.97879,
    than countries in Africa
  - If population increase by 1% then the HR increase by 32%. -\> large
    countries close early.
  - Medium Life expectancy increase HR 2.8 times compare to high life.
    If we assume life expectancy as proxy for health services then
    countries have worse health survices lockdown earlier. -\> make
    sense.
  - 1 point increase in log(GDP) increase HR by (2.0047 - 1) = 100%.
  - Countries with small number of Airports close late -\> lockdown late
    if there is less travel to the country
  - Test trace increase HR by 1%.

Over all: reasonable.
