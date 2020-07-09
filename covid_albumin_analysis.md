---
title: "Analysis of albumin levels in patients from Tongji Hospital, Wuhan, China"
author: "Dariusz Brzezinski"
date: "2020-07-09"
output: 
  html_document: 
    number_sections: yes
    toc: yes
    toc_float: yes
    keep_md: yes
---





This notebook presents the plot scripts and statistical analyses presented in the paper "Molecular determinants of dexamethasone vascular transport in COVID-19 therapy" by Shabalin *et al.* Plots presented herein are interactive versions of those presented in the manuscript.

# Data characteristics



The data for the study was taken from the supplementary materials of "An interpretable mortality prediction model for COVID-19 patients" Nat. Mach. Intell. 2, 283–288 (2020) by *Yan et al.* and describes COVID-19 patients admitted to Tongji Hospital, Wuhan, China between January 10 and February 18, 2020. The raw dataset contained information about 375 patients, however only 373 patients that had their albumin levels measured at least once during their hospital stay, and those were of interest to the current study. The table below presents the basic statistics of clinical variables analyzed in this study. These and other statistics, unless stated otherwise, are calculated based on the last available blood sample of given patient.


+----+---------------------+--------------------------+---------------------+----------------------+---------+
| No | Variable            | Stats / Values           | Freqs (% of Valid)  | Graph                | Missing |
+====+=====================+==========================+=====================+======================+=========+
| 1  | Outcome\            | 1\. Died\                | 174 (46.7%)\        | ![](/tmp/ds0109.png) | 0\      |
|    | [factor]            | 2\. Survived             | 199 (53.3%)         |                      | (0%)    |
+----+---------------------+--------------------------+---------------------+----------------------+---------+
| 2  | Gender\             | 1\. Male\                | 222 (59.5%)\        | ![](/tmp/ds0110.png) | 0\      |
|    | [factor]            | 2\. Female               | 151 (40.5%)         |                      | (0%)    |
+----+---------------------+--------------------------+---------------------+----------------------+---------+
| 3  | Age\                | Mean (sd) : 58.8 (16.5)\ | 71 distinct values  | ![](/tmp/ds0111.png) | 0\      |
|    | [numeric]           | min < med < max:\        |                     |                      | (0%)    |
|    |                     | 18 < 62 < 95\            |                     |                      |         |
|    |                     | IQR (CV) : 24 (0.3)      |                     |                      |         |
+----+---------------------+--------------------------+---------------------+----------------------+---------+
| 4  | TimeSinceAdmission\ | Mean (sd) : 7.7 (5.9)\   | 342 distinct values | ![](/tmp/ds0112.png) | 0\      |
|    | [numeric]           | min < med < max:\        |                     |                      | (0%)    |
|    |                     | 0 < 7.3 < 21.8\          |                     |                      |         |
|    |                     | IQR (CV) : 10.5 (0.8)    |                     |                      |         |
+----+---------------------+--------------------------+---------------------+----------------------+---------+
| 5  | Albumin\            | Mean (sd) : 32.6 (6.3)\  | 183 distinct values | ![](/tmp/ds0113.png) | 0\      |
|    | [numeric]           | min < med < max:\        |                     |                      | (0%)    |
|    |                     | 13.6 < 33 < 47.6\        |                     |                      |         |
|    |                     | IQR (CV) : 9.7 (0.2)     |                     |                      |         |
+----+---------------------+--------------------------+---------------------+----------------------+---------+
| 6  | Glucose\            | Mean (sd) : 8.5 (5.2)\   | 284 distinct values | ![](/tmp/ds0114.png) | 0\      |
|    | [numeric]           | min < med < max:\        |                     |                      | (0%)    |
|    |                     | 1 < 6.5 < 38.8\          |                     |                      |         |
|    |                     | IQR (CV) : 4.7 (0.6)     |                     |                      |         |
+----+---------------------+--------------------------+---------------------+----------------------+---------+

The overall mortality rate was 46.65%, whereas the mortality rate among male and female patients was 56.76% and 31.79%, respectively.

# Albumin levels

## Distributions and normality tests



The sina plot below presents the distribution of albumin levels among the two outcome groups of patients (Died, Survived), with the mean and standard deviation overlaid.

![](covid_albumin_analysis_files/figure-html/albumin_sina_plot-1.png)<!-- -->

To verify whether the albumin levels in both outcome groups are normally distributed, we first plotted Q-Q plots, as presented below. The shaded area represents the 95% confidence intervals.

![](covid_albumin_analysis_files/figure-html/qq_plot-1.png)<!-- -->

Upon a positive verification of the Q-Q plots, the normality of the the albumin distributions was additionally confirmed by two Shapiro-Wilk tests:


<table class="table table-condensed">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> value </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> statistic.W </td>
   <td style="text-align:right;"> 0.995438624983042 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> p.value </td>
   <td style="text-align:right;"> 0.876037297780011 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> method </td>
   <td style="text-align:right;"> Shapiro-Wilk normality test </td>
  </tr>
  <tr>
   <td style="text-align:left;"> data.name </td>
   <td style="text-align:right;"> Outcome == Died </td>
  </tr>
</tbody>
</table>

<table class="table table-condensed">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> value </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> statistic.W </td>
   <td style="text-align:right;"> 0.991932747653017 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> p.value </td>
   <td style="text-align:right;"> 0.338673164623577 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> method </td>
   <td style="text-align:right;"> Shapiro-Wilk normality test </td>
  </tr>
  <tr>
   <td style="text-align:left;"> data.name </td>
   <td style="text-align:right;"> Outcome == Survived </td>
  </tr>
</tbody>
</table>


Since the p-values of the Shapiro-Wilk tests are above 0.05, we cannot reject the null hypothesis that the albumin levels are normally distributed. Therefore, we assume the null hypothesis is true and **the albumin levels for both outcome groups are normally distributed**. Having verified the normality of the distributions, we can perform a two-tailed Welch's t-test to check if the differences in albumin levels are statistically significant.


<table class="table table-condensed">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> value </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> statistic.t </td>
   <td style="text-align:right;"> -19.1881248432455 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parameter.df </td>
   <td style="text-align:right;"> 338.060954028682 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> p.value </td>
   <td style="text-align:right;"> 4.94695446316735e-56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> conf.int1 </td>
   <td style="text-align:right;"> -9.91937925851427 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> conf.int2 </td>
   <td style="text-align:right;"> -8.07476964693251 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> estimate.mean of x </td>
   <td style="text-align:right;"> 27.7752873563218 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> estimate.mean of y </td>
   <td style="text-align:right;"> 36.7723618090452 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> null.value.difference in means </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stderr </td>
   <td style="text-align:right;"> 0.468887633691339 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alternative </td>
   <td style="text-align:right;"> two.sided </td>
  </tr>
  <tr>
   <td style="text-align:left;"> method </td>
   <td style="text-align:right;"> Welch Two Sample t-test </td>
  </tr>
  <tr>
   <td style="text-align:left;"> data.name </td>
   <td style="text-align:right;"> Outcome == Died vs Outcome == Survived </td>
  </tr>
</tbody>
</table>


With a p-value < 0.001 we can reject the null hypothesis that the means are equal, and state that **the differences in mean albumin levels between the patients that died and survived are statistically significant**. The mean albumin level for those that sirvived was 36.7723618 g/L, whereas for those that died it was 27.7752874 g/L.

## Gender

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-1-1.png)<!-- -->


```
## 
## 	Welch Two Sample t-test
## 
## data:  df_last[df_last$Outcome == "Died" & df_last$Gender == "Male", ]$Albumin and df_last[df_last$Outcome == "Survived" & df_last$Gender == "Male", ]$Albumin
## t = -13.739, df = 212.27, p-value < 2.2e-16
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -9.750720 -7.303844
## sample estimates:
## mean of x mean of y 
##  28.00397  36.53125
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  df_last[df_last$Outcome == "Died" & df_last$Gender == "Female", ]$Albumin and df_last[df_last$Outcome == "Survived" & df_last$Gender == "Female", ]$Albumin
## t = -11.956, df = 71.305, p-value < 2.2e-16
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -11.460025  -8.184149
## sample estimates:
## mean of x mean of y 
##  27.17500  36.99709
```

## Time series and trends




```
## [1] "Died =  -0.08 Time +  27.65"
```

```
## [1] "Survived =  0.02 Time +  37.2"
```


```
## [1] 2.728242
```

```
## [1] 0.645511
```

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-6-1.png)<!-- -->


```
## 
## 	Pearson's product-moment correlation
## 
## data:  d[d$Outcome == "Died", ]$TimeSinceAdmission and d[d$Outcome == "Died", ]$Albumin
## t = -4.1374, df = 346, p-value = 4.415e-05
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.3150624 -0.1146082
## sample estimates:
##        cor 
## -0.2171231
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  d[d$Outcome == "Survived", ]$TimeSinceAdmission and d[d$Outcome == "Survived", ]$Albumin
## t = -1.3569, df = 396, p-value = 0.1756
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.1652221  0.0304729
## sample estimates:
##         cor 
## -0.06802892
```

## Relation between albumin and glucose levels

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

## Relation between albumin and glucose levels

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

# Regression analysis





## Unadjusted models

<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">OutcomeNumber</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Odds Ratios</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">(Intercept)</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00&nbsp;&ndash;&nbsp;0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Albumin</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.56</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.44&nbsp;&ndash;&nbsp;1.71</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">373</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">R<sup>2</sup> Tjur</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.551</td>
</tr>

</table>

<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">OutcomeNumber</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Odds Ratios</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">(Intercept)</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.76</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.58&nbsp;&ndash;&nbsp;0.99</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.045</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Gender [Female]</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">2.82</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.83&nbsp;&ndash;&nbsp;4.37</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">373</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">R<sup>2</sup> Tjur</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.060</td>
</tr>

</table>
<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">OutcomeNumber</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Odds Ratios</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">(Intercept)</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">479.80</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">133.55&nbsp;&ndash;&nbsp;2023.14</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Age</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.90</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.88&nbsp;&ndash;&nbsp;0.92</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">373</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">R<sup>2</sup> Tjur</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.332</td>
</tr>

</table>
<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">OutcomeNumber</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Odds Ratios</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">(Intercept)</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">10.30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">5.76&nbsp;&ndash;&nbsp;19.34</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Glucose</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.76</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.70&nbsp;&ndash;&nbsp;0.81</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">373</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">R<sup>2</sup> Tjur</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.227</td>
</tr>

</table>

## Adjusted model

<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">OutcomeNumber</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Odds Ratios</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">(Intercept)</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00&nbsp;&ndash;&nbsp;0.02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Albumin</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.51</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.37&nbsp;&ndash;&nbsp;1.69</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Glucose</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.89</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.81&nbsp;&ndash;&nbsp;0.96</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.006</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Age</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.92</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.89&nbsp;&ndash;&nbsp;0.94</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Gender [Female]</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">2.19</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.04&nbsp;&ndash;&nbsp;4.72</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.041</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">373</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">R<sup>2</sup> Tjur</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.676</td>
</tr>

</table>

## Testing for effect modification

<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">OutcomeNumber</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Odds Ratios</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">(Intercept)</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00&nbsp;&ndash;&nbsp;0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Albumin</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.48</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.35&nbsp;&ndash;&nbsp;1.66</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Gender [Female]</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00&nbsp;&ndash;&nbsp;13.43</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.285</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Albumin * Gender [Female]</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.15</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.95&nbsp;&ndash;&nbsp;1.42</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.173</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">373</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">R<sup>2</sup> Tjur</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.570</td>
</tr>

</table>
<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">OutcomeNumber</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Odds Ratios</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">(Intercept)</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00&nbsp;&ndash;&nbsp;71.37</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.173</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Albumin</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.68</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.03&nbsp;&ndash;&nbsp;2.89</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.049</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Age</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.95</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.74&nbsp;&ndash;&nbsp;1.24</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.722</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Albumin * Age</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.99&nbsp;&ndash;&nbsp;1.01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.772</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">373</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">R<sup>2</sup> Tjur</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.658</td>
</tr>

</table>
<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">OutcomeNumber</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Odds Ratios</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">(Intercept)</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00&nbsp;&ndash;&nbsp;0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Albumin</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.73</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.42&nbsp;&ndash;&nbsp;2.09</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Glucose</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.44</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.70&nbsp;&ndash;&nbsp;2.39</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.242</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Albumin * Glucose</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.98</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.97&nbsp;&ndash;&nbsp;1.01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.096</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">373</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">R<sup>2</sup> Tjur</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.590</td>
</tr>

</table>

## Difference in outcome~albumin regression based on gender

![](covid_albumin_analysis_files/figure-html/unnamed-chunk-16-1.png)<!-- -->

# Citing
