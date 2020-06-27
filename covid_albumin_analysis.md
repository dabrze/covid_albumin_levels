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
library(lme4)
```


```r
read_data <- function(file) {
  df <- read_excel(file)
  df$PATIENT_ID <- na.locf(df$PATIENT_ID)
  df <- na.locf(df)
  
  df$AlbuminSampleId <- 1
  for (id in unique(df$PATIENT_ID)) {
    df[df$PATIENT_ID == id,"AlbuminSampleId"] <- seq.int(nrow(df[df$PATIENT_ID == id, ]))
  }
  
  df_all_samples <- df %>%
    mutate(gender = replace(gender, gender == 1, "Male"), 
           gender = replace(gender, gender == 2, "Female"), 
           outcome = replace(outcome, outcome == 0, "Survived"), 
           outcome = replace(outcome, outcome == 1, "Death"),
           Outcome = outcome)
  
  df_all_samples
}

get_last_samples <- function(df_all_samples) {
  df <- df_all_samples %>%
    group_by(PATIENT_ID) %>% 
    summarise(albumin=last(albumin), Outcome=last(outcome), glucose=last(glucose), n=n())
  
  df
}

get_first_samples <- function(df_all_samples) {
  df <- df_all_samples %>%
    group_by(PATIENT_ID) %>% 
    summarise(albumin=first(albumin), Outcome=first(outcome), n=n())
  
  df
}
```


```r
df_all_samples <- read_data("data/time_series_375_prerpocess_en.xlsx")
df <- get_last_samples(df_all_samples)
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
df_first <- get_first_samples(df_all_samples)
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
        axis.title.y = element_text(margin = margin(t = 0, r = 8, b = 0, l = 0))) + 
  xlab("") + 
  ylab ("Patient's albumin level (g/L)") +
  ylim(13, 56) +
  scale_color_manual(values=c("#EE6677", "#4477AA"))
```

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-1-1.png)<!-- -->




```r
ggplot(df, aes(x=glucose, y=albumin, color=Outcome)) + 
  geom_hline(aes(yintercept=35), colour="#555555", linetype="dashed") +
  geom_hline(aes(yintercept=55), colour="#555555", linetype="dashed") +
  geom_vline(aes(xintercept=3.9), colour="#555555", linetype="dashed") +
  geom_vline(aes(xintercept=7.1), colour="#555555", linetype="dashed") + 
  geom_point() + 
  theme_classic() + 
  theme(legend.position = c(0.88, 0.15), 
        text = element_text(size=18), 
        axis.text = element_text(colour = "black", size=18), 
        axis.title.y = element_text(margin = margin(t = 0, r = 18, b = 0, l = 0))) + 
  xlab("Patient's glucose level (mmol/L)") + 
  ylab ("Patient's albumin level (g/L)") +
  ylim(13, 56) +
  scale_color_manual(values=c("#EE6677", "#4477AA"))
```

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-3-1.png)<!-- -->



```r
ggplot(df, aes(x=Outcome, y=glucose, color=Outcome)) + 
  geom_hline(aes(yintercept=3.9), colour="#555555", linetype="dashed") +
  geom_hline(aes(yintercept=7.1), colour="#555555", linetype="dashed") + 
  geom_sina(seed=21) + 
  stat_summary(fun.data = "mean_sdl",  fun.args = list(mult = 1), 
               geom = "pointrange", color = "black") +
  theme_classic() + 
  theme(legend.position = "none", 
        text = element_text(size=18), 
        axis.text = element_text(colour = "black", size=18), 
        axis.title.y = element_text(margin = margin(t = 0, r = 18, b = 0, l = 0))) + 
  xlab("") + 
  ylab ("Patient's glucose level (mmol/L)") +
  scale_color_manual(values=c("#EE6677", "#4477AA"))
```

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
ggsave("plots/glucose_level_sina_plot_first.svg", width=6, height=6, units="in")
ggsave("plots/glucose_level_sina_plot_first.png", width=6, height=6, units="in")
```


```r
 d <- df_all_samples[, c("PATIENT_ID", "albumin", "AlbuminSampleId")]

 fits <- lmList(albumin ~ AlbuminSampleId | PATIENT_ID, data=d)
 coefs <- coef(fits)
 coefs <- cbind(coefs, df)
 
 
aDeath <- median(coefs[coefs$Outcome == "Death", "AlbuminSampleId"], na.rm=T)
aQ1Death <- quantile(coefs[coefs$Outcome == "Death", "AlbuminSampleId"], 0.25, na.rm=T)
aQ3Death <- quantile(coefs[coefs$Outcome == "Death", "AlbuminSampleId"], 0.75, na.rm=T)
bDeath <- median(coefs[coefs$Outcome == "Death", "(Intercept)"], na.rm=T)
aSurvived <- median(coefs[coefs$Outcome == "Survived", "AlbuminSampleId"], na.rm=T)
aQ1Survived <- quantile(coefs[coefs$Outcome == "Survived", "AlbuminSampleId"], 0.25, na.rm=T)
aQ3Survived <- quantile(coefs[coefs$Outcome == "Survived", "AlbuminSampleId"], 0.75, na.rm=T)
bSurvived <- median(coefs[coefs$Outcome == "Survived", "(Intercept)"], na.rm=T)
```


```r
ggplot(df_all_samples, aes(x=AlbuminSampleId, y=albumin, color=Outcome)) + 
  geom_hline(aes(yintercept=35), colour="#555555", linetype="dashed") +
  geom_hline(aes(yintercept=55), colour="#555555", linetype="dashed") + 
  geom_line(mapping=aes(group=PATIENT_ID), alpha=0.35) +
  geom_segment(aes(x=1, xend=60, y=bDeath + aDeath, 
                   yend = bDeath + aDeath*60), 
               color="#EE6677", size=1) +
    # geom_segment(aes(x=1, xend=60, y=bDeath + aQ1Death, 
    #                yend = bDeath + aQ1Death*60), 
    #            color="#EE6677", size=0.8, linetype = 2) +
    # geom_segment(aes(x=1, xend=60, y=bDeath + aQ3Death, 
    #                yend = bDeath + aQ3Death*60), 
    #            color="#EE6677", size=0.8, linetype = 2) +
  geom_segment(aes(x=1, xend=60, 
                   y=bSurvived + aSurvived, yend = bSurvived + aSurvived*60),
               color="#4477AA", size=1) +
    # geom_segment(aes(x=1, xend=60, 
    #                y=bSurvived + aQ1Survived, yend = bSurvived + aQ1Survived*60),
    #            color="#4477AA", size=0.8, linetype = 2) +
    # geom_segment(aes(x=1, xend=60, 
    #                y=bSurvived + aQ3Survived, yend = bSurvived + aQ3Survived*60),
    #            color="#4477AA", size=0.8, linetype = 2) +
  theme_classic() + 
  theme(legend.position = c(0.88, 0.12), 
        text = element_text(size=18), 
        axis.text = element_text(colour = "black", size=18), 
        axis.title.y = element_text(margin = margin(t = 0, r = 18, b = 0, l = 0))) + 
  xlab("Blood sample number") + 
  ylab ("Patient's albumin level (g/L)") +
  ylim(13, 56) +
  scale_color_manual(values=c("#EE6677", "#4477AA"))
```

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-7-1.png)<!-- -->




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
        text = element_text(size=18)) + xlab("Theoretical quantiles") + ylab("Sample quantiles")
```

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-10-1.png)<!-- -->




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


```r
mean(df[df$Outcome=="Death",]$albumin)
```

```
## [1] 27.77529
```

```r
sd(df[df$Outcome=="Death",]$albumin)
```

```
## [1] 4.878064
```

```r
mean(df[df$Outcome=="Survived",]$albumin)
```

```
## [1] 36.77236
```

```r
sd(df[df$Outcome=="Survived",]$albumin)
```

```
## [1] 4.066554
```
