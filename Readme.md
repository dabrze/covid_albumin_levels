# Reproducible analysis of albumin levels in patients from Tongji Hospital, Wuhan, China

The main folder contains the R source code (`covid_albumin_analysis.Rmd`) in the form of a knitr notebook and the resulting report in Markdown (`covid_albumin_analysis.md`) and HTML (`covid_albumin_analysis.html`) format. If you want to take a look at the report open the markdown file in the browser or download the html file to your computer. The data for this analysis were taken from: https://github.com/HAIRLAB/Pre_Surv_COVID_19 and were describe in "An interpretable mortality prediction model for COVID-19 patients" by Li Yan *et al.* ([Nature Machine Intelligence](https://www.nature.com/articles/s42256-020-0180-7)).

The rest of the repository is organized as follows:

- the `data` folder contains the source data for this analysis;
- the `plots` folder contains data plots that were used in the accompanying publication.

# Usage

To reproduce the analysis:

1. Clone or download the repository
2. Open RStudio and run (*knit*) `covid_albumin_analysis.Rmd`
3. The svg and png versions of the resulting plots will be saved into the `plots` folder


## Contact
Dariusz Brzezinski: dariusz (dot) brzezinski (at) cs (dot) put (dot) poznan (dot) pl