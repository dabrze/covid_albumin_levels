---
title: "Analysis of albumin levels in patients from Tongji Hospital, Wuhan, China"
output:
  html_document: 
    keep_md: yes
---


```r
library(readxl)
library(Hmisc)
library(ggplot2)
library(ggforce)
library(dplyr)
library(zoo)
library(ggpubr)
```


```r
read_data <- function(file) {
  df <- read_excel(file)
  df$PATIENT_ID <- na.locf(df$PATIENT_ID)
  df <- na.locf(df)
  df <- df %>%
    mutate(gender = replace(gender, gender == 1, "Male"), 
           gender = replace(gender, gender == 2, "Female"), 
           outcome = replace(outcome, outcome == 0, "Survived"), 
           outcome = replace(outcome, outcome == 1, "Death")) %>%
    group_by(PATIENT_ID) %>% 
    summarise(albumin=last(albumin), Outcome=last(outcome), n=n())
  
  df
}
```


```r
df <- read_data("data/time_series_375_prerpocess_en.xlsx")
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
ggplot(df, aes(x=Outcome, y=albumin, color=Outcome)) + 
  geom_hline(aes(yintercept=35), colour="#555555", linetype="dashed") +
  geom_hline(aes(yintercept=55), colour="#555555", linetype="dashed") + 
  geom_sina(seed=21) + 
  stat_summary(fun.data = "mean_sdl",  fun.args = list(mult = 1), 
               geom = "pointrange", color = "black") +
  theme_classic() + 
  theme(legend.position = "none", 
        text = element_text(size=18), 
        axis.text = element_text(colour = "black", size=18), 
        axis.title.y = element_text(margin = margin(t = 0, r = 18, b = 0, l = 0))) + 
  xlab("") + 
  ylab ("Patient's albumin level (g/L)") +
  scale_color_manual(values=c("#EE6677", "#4477AA"))
```

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-1-1.png)<!-- -->




```r
print(tapply(df$albumin, df$Outcome, length))
```

```
##    Death Survived 
##      174      199
```

```r
print(tapply(df$albumin, df$Outcome, median))
```

```
##    Death Survived 
##     27.6     37.0
```

```r
print(tapply(df$albumin, df$Outcome, quantile))
```

```
## $Death
##     0%    25%    50%    75%   100% 
## 13.600 24.275 27.600 31.000 39.500 
## 
## $Survived
##    0%   25%   50%   75%  100% 
## 26.90 33.85 37.00 39.30 47.60
```


```r
ggqqplot(df, x="albumin", color="Outcome", palette = c("#EE6677", "#4477AA")) + 
  theme(legend.position = c(0.88, 0.18), 
        axis.text = element_text(colour = "black", size=18), 
        text = element_text(size=18))
```

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-4-1.png)<!-- -->




```r
shapiro.test(df[df$Outcome=="Death",]$albumin)
```

```
## 
## 	Shapiro-Wilk normality test
## 
## data:  df[df$Outcome == "Death", ]$albumin
## W = 0.99544, p-value = 0.876
```

```r
shapiro.test(df[df$Outcome=="Survived",]$albumin)
```

```
## 
## 	Shapiro-Wilk normality test
## 
## data:  df[df$Outcome == "Survived", ]$albumin
## W = 0.99193, p-value = 0.3387
```

```r
t.test(df[df$Outcome=="Death",]$albumin, df[df$Outcome=="Survived",]$albumin)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  df[df$Outcome == "Death", ]$albumin and df[df$Outcome == "Survived", ]$albumin
## t = -19.188, df = 338.06, p-value < 2.2e-16
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -9.919379 -8.074770
## sample estimates:
## mean of x mean of y 
##  27.77529  36.77236
```
