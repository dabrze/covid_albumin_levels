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
library(cowplot)
```


```r
PLOT_SIZE=5
FONT_SIZE=18
PLOT_UNITS="in"
DEATH_COLOR <- "#EE6677"
SURVIVED_COLOR <- "#4477AA"
NORMAL_RANGE_COLOR <- "#000000"
OUTCOME_PALETTE <- c(DEATH_COLOR, SURVIVED_COLOR)

read_data <- function(file) {
  df <- read_excel(file)
  df$PATIENT_ID <- na.locf(df$PATIENT_ID)
  df <- na.locf(df)
  
  for (id in unique(df$PATIENT_ID)) {
    df[df$PATIENT_ID == id,"First_RE_DATE"] <- df[df$PATIENT_ID == id, "RE_DATE"][[1]][1]
  }
  
  df_all_samples <- df %>%
    mutate(gender = replace(gender, gender == 1, "Male"), 
           gender = replace(gender, gender == 2, "Female"), 
           outcome = replace(outcome, outcome == 0, "Survived"), 
           outcome = replace(outcome, outcome == 1, "Died"),
           Outcome = outcome,
           TimeSinceSubmission = as.numeric(difftime(RE_DATE, First_RE_DATE), units="days"))
  
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

save_plot <- function(p, name) {
  ggsave(paste0("plots/", name, ".svg"), p, width=PLOT_SIZE, height=PLOT_SIZE, units=PLOT_UNITS)
  ggsave(paste0("plots/", name, ".png"), p, width=PLOT_SIZE, height=PLOT_SIZE, units=PLOT_UNITS)
}

draw_albumin_sina_plot <-function(df, plot_name) {
  p <- ggplot(df, aes(x=Outcome, y=albumin, color=Outcome)) + 
  geom_hline(aes(yintercept=35), colour=NORMAL_RANGE_COLOR, linetype="dashed") +
  geom_hline(aes(yintercept=55), colour=NORMAL_RANGE_COLOR, linetype="dashed") + 
    
  geom_sina(seed=21) + 
  stat_summary(fun.data = "mean_sdl",  fun.args = list(mult = 1), 
               geom = "pointrange", color = "black") +
  theme_classic() + 
  theme(legend.position = "none", 
        text = element_text(size=FONT_SIZE), 
        axis.text = element_text(colour = "black", size=FONT_SIZE), 
        axis.title.y = element_text(margin = margin(t = 0, r = 8, b = 0, l = 0))) + 
  xlab("") + 
  ylab ("Patient's albumin level (g/L)") +
  ylim(13, 56) +
  scale_color_manual(values=OUTCOME_PALETTE)
  
  save_plot(p, plot_name)
  p
}

draw_glucose_sina_plot <- function(df, plot_name, width=PLOT_SIZE, height=PLOT_SIZE, units=PLOT_UNITS) {
  p <- ggplot(df, aes(x=Outcome, y=glucose, color=Outcome)) + 
  geom_hline(aes(yintercept=3.9), colour=NORMAL_RANGE_COLOR, linetype="dashed") +
  geom_hline(aes(yintercept=7.1), colour=NORMAL_RANGE_COLOR, linetype="dashed") + 
  geom_sina(seed=21) + 
  stat_summary(fun.data = "mean_sdl",  fun.args = list(mult = 1), 
               geom = "pointrange", color = "black") +
  theme_classic() + 
  theme(legend.position = "none", 
        text = element_text(size=FONT_SIZE), 
        axis.text = element_text(colour = "black", size=FONT_SIZE), 
        axis.title.y = element_text(margin = margin(t = 0, r = 8, b = 0, l = 0))) + 
  xlab("") + 
  ylab ("Patient's glucose level (mmol/L)") +
  scale_color_manual(values=OUTCOME_PALETTE)
  
  save_plot(p, plot_name)
  p
}

draw_albumin_glucose_plot <-function(df, plot_name, width=6, height=6) {
  p <- ggplot(df, aes(x=glucose, y=albumin, color=Outcome)) + 
  geom_hline(aes(yintercept=35), colour=NORMAL_RANGE_COLOR, linetype="dashed") +
  geom_hline(aes(yintercept=55), colour=NORMAL_RANGE_COLOR, linetype="dashed") +
  geom_vline(aes(xintercept=3.9), colour=NORMAL_RANGE_COLOR, linetype="dashed") +
  geom_vline(aes(xintercept=7.1), colour=NORMAL_RANGE_COLOR, linetype="dashed") + 
  geom_point() + 
  theme_classic() + 
  theme(legend.position = "none", #c(0.85, 0.88), 
        text = element_text(size=FONT_SIZE), 
        axis.text = element_text(colour = "black", size=FONT_SIZE), 
        axis.text.y = element_blank()) + 
  xlab("Patient's glucose level (mmol/L)") + 
  ylab ("") +
  ylim(13, 56) +
  scale_color_manual(values=OUTCOME_PALETTE)
  
  save_plot(p, plot_name)
  p
}

draw_time_series_plot <- function(plot_name, aDeath, bDeath, aSurvived, bSurvived) {
  p <- ggplot(df_all, aes(x=TimeSinceSubmission, y=albumin, color=Outcome)) + 
    geom_hline(aes(yintercept=35), colour=NORMAL_RANGE_COLOR, linetype="dashed") +
    geom_hline(aes(yintercept=55), colour=NORMAL_RANGE_COLOR, linetype="dashed") + 
    geom_line(mapping=aes(group=PATIENT_ID), alpha=0.35) +
    geom_segment(aes(x=0, xend=22, y=bDeath + aDeath, 
                     yend = bDeath + aDeath*22), 
                 color=DEATH_COLOR, size=1.25) +
    geom_segment(aes(x=0, xend=22, y=bSurvived + aSurvived, 
                     yend = bSurvived + aSurvived*22),
                 color=SURVIVED_COLOR, size=1.25) +
    theme_classic() + 
    theme(legend.position = "none", 
          text = element_text(size=FONT_SIZE), 
          axis.text = element_text(colour = "black", size=FONT_SIZE), 
          axis.text.y = element_blank()) + 
    # geom_text(mapping=aes(label=as.character(labDeath), x=10, y=bDeath), 
    #           label.size=FONT_SIZE*0.8, vjust=-0.5, hjust=0, inherit.aes = FALSE, 
    #           color = NORMAL_RANGE_COLOR, parse = T) +
    #   geom_text(mapping=aes(label=as.character(labSurvived), x=12, y=bSurvived), 
    #         label.size=FONT_SIZE*0.8, vjust=-0.5, hjust=0, inherit.aes = FALSE, 
    #         color = NORMAL_RANGE_COLOR, parse = T) +
    xlab("Days since admission") + 
    ylab ("") +
    ylim(13, 56) +
    scale_color_manual(values=OUTCOME_PALETTE)
  
  save_plot(p, plot_name)
  p
}
```


```r
df_all <- read_data("data/time_series_375_prerpocess_en.xlsx")
df_last <- get_last_samples(df_all)
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```


```r
a_plot <- draw_albumin_sina_plot(df_last, "albumin_level_sina_plot")
```


```r
c_plot <- draw_albumin_glucose_plot(df_last, "albumin_glucose_plot")
```


```r
supplement_plot <- draw_glucose_sina_plot(df_last, "glucose_level_sina_plot")
```


```r
d <- df_all[, c("PATIENT_ID", "albumin", "TimeSinceSubmission")]

fits <- lmList(albumin ~ TimeSinceSubmission | PATIENT_ID, data=d)
coefs <- coef(fits)
coefs <- cbind(coefs, df_last)

aDeath <- median(coefs[coefs$Outcome == "Died", "TimeSinceSubmission"], na.rm=T)
bDeath <- median(coefs[coefs$Outcome == "Died", "(Intercept)"], na.rm=T)
aSurvived <- median(coefs[coefs$Outcome == "Survived", "TimeSinceSubmission"], na.rm=T)
bSurvived <- median(coefs[coefs$Outcome == "Survived", "(Intercept)"], na.rm=T)

b_plot <- draw_time_series_plot("albumin_level_series", aDeath, bDeath, aSurvived, bSurvived)
```


```r
plot_grid(a_plot, b_plot, c_plot, labels=c("A", "B", "C"), ncol = 3, nrow = 1, label_size=24)
```

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
ggsave(paste0("plots/", "combined", ".svg"), width=PLOT_SIZE*3, height=PLOT_SIZE)
ggsave(paste0("plots/", "combined", ".png"), width=PLOT_SIZE*3, height=PLOT_SIZE)
```




```r
ggqqplot(df_last, x="albumin", color="Outcome", palette = c("#EE6677", "#4477AA")) + 
  theme(legend.position = c(0.88, 0.18), 
        axis.text = element_text(colour = "black", size=18), 
        text = element_text(size=18)) + xlab("Theoretical quantiles") + ylab("Sample quantiles")
```

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-8-1.png)<!-- -->




```r
shapiro.test(df_last[df_last$Outcome=="Died",]$albumin)
```

```
## 
## 	Shapiro-Wilk normality test
## 
## data:  df_last[df_last$Outcome == "Died", ]$albumin
## W = 0.99544, p-value = 0.876
```

```r
shapiro.test(df_last[df_last$Outcome=="Survived",]$albumin)
```

```
## 
## 	Shapiro-Wilk normality test
## 
## data:  df_last[df_last$Outcome == "Survived", ]$albumin
## W = 0.99193, p-value = 0.3387
```

```r
t.test(df_last[df_last$Outcome=="Died",]$albumin, df_last[df_last$Outcome=="Survived",]$albumin)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  df_last[df_last$Outcome == "Died", ]$albumin and df_last[df_last$Outcome == "Survived", ]$albumin
## t = -19.188, df = 338.06, p-value < 2.2e-16
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -9.919379 -8.074770
## sample estimates:
## mean of x mean of y 
##  27.77529  36.77236
```


```r
mean(df_last[df_last$Outcome=="Died",]$albumin)
```

```
## [1] 27.77529
```

```r
sd(df_last[df_last$Outcome=="Died",]$albumin)
```

```
## [1] 4.878064
```


```r
mean(df_last[df_last$Outcome=="Survived",]$albumin)
```

```
## [1] 36.77236
```

```r
sd(df_last[df_last$Outcome=="Survived",]$albumin)
```

```
## [1] 4.066554
```
