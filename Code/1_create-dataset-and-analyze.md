Sample size calculations for maternal RSV immunization and development of childhood recurrent wheeze
================
Corinne Riddell
2/2/2018

``` r
library(Hmisc) #for the bsamsize function for sample size calculation under unequal randomization schemes
```

    ## Loading required package: lattice

    ## Loading required package: survival

    ## Loading required package: Formula

    ## Loading required package: ggplot2

    ## 
    ## Attaching package: 'Hmisc'

    ## The following objects are masked from 'package:base':
    ## 
    ##     format.pval, units

``` r
library(plyr)
```

    ## 
    ## Attaching package: 'plyr'

    ## The following objects are masked from 'package:Hmisc':
    ## 
    ##     is.discrete, summarize

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:plyr':
    ## 
    ##     arrange, count, desc, failwith, id, mutate, rename, summarise,
    ##     summarize

    ## The following objects are masked from 'package:Hmisc':
    ## 
    ##     src, summarize

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(ggplot2)
library(DT)
library(scales)
```

### Create data frame and calculate sample size and NNT

Puts every combination of parameter values into a data frame:

``` r
scenarios <- expand.grid(vaccine.efficacy     = c(0.5, 0.7, 0.9), 
                         rsv.attack.rate      = c(27/1000, 60/1000, 170/1000),
                         RR.wheeze.rsvh       = c(1.6, 2.6, 4), 
                         baseline.risk.wheeze = c(0.049, 0.095, 0.2)
                         )
```

For each scenario, we calculate the risk of wheeze among vaccinated and unvaccinated mother-child pairs and take the ratio. We use these risks to calculate the sample size under 1:1 and 2:1 randomization schemes.

``` r
scenarios <- scenarios %>% 
               mutate(percent.altered    = vaccine.efficacy * rsv.attack.rate,
                      risk.wheeze.unvacc = (baseline.risk.wheeze * (1 - rsv.attack.rate)) +  # baseline risk among non-RSV infants
                                           (baseline.risk.wheeze * RR.wheeze.rsvh * rsv.attack.rate),  # increased risk among RSV infants
                      risk.wheeze.vacc   = (baseline.risk.wheeze * (1 - rsv.attack.rate)) +  # baseline risk among non-RSV infants
                                           (baseline.risk.wheeze * percent.altered) +  # baseline risk among RSV *prevented* infants
                                           (baseline.risk.wheeze * RR.wheeze.rsvh * (rsv.attack.rate - percent.altered)), # increased risk among RSV *doomed* infants
                      RR.wheeze.vacc     = risk.wheeze.vacc / risk.wheeze.unvacc)
```

The R functions to calculate sample size are `power.prop.test` from base and `bsamsize` from Hmisc for unequal randomization.

``` r
scenarios$size.one.arm.equal <- NA  # will store sample size *per arm* under equal randomization
scenarios$size.unvacc.2.1 <- NA  # will store sample size *unvaccinated* under 2:1 randomization
scenarios$size.vacc.2.1 <- NA

for (i in 1:dim(scenarios)[1]) {
  scenarios$size.one.arm.equal[i] <- power.prop.test(p1 = scenarios$risk.wheeze.unvacc[i],
                                                     p2 = scenarios$risk.wheeze.vacc[i],
                                                     power = 0.8, sig.level = 0.05, 
                                                     alternative = "two.sided")$n  
  
  unequal <- bsamsize(p1 = scenarios$risk.wheeze.unvacc[i], 
                      p2 = scenarios$risk.wheeze.vacc[i], 
                      fraction = 0.333333, 
                      alpha    = 0.05, power = 0.8)
  
  scenarios$size.unvacc.2.1[i] <- unequal[1]
  scenarios$size.vacc.2.1[i] <- unequal[2]
}

rm(i, unequal)

scenarios <- scenarios %>% 
  mutate(sample.total.2.1 = ceiling(scenarios$size.unvacc.2.1 + scenarios$size.vacc.2.1), #round up
         sample.total.1.1 = ceiling(scenarios$size.one.arm.equal * 2))
```

NNT is the "number needed to treat" or the number needed to be vaccinated to prevent one case of recurrent wheeze during childhood. It is the inverse of the risk difference.

``` r
scenarios <- scenarios %>% 
  mutate(RD.wheeze.vacc = risk.wheeze.unvacc - risk.wheeze.vacc,
         NNT = 1/RD.wheeze.vacc)
```

We create a new variable for the vaccine efficacy information that will be used to label the panels in the figure. We also add a variable `plausible` that classifies the plausbility of the scenarios as described in the paper.

``` r
scenarios$`Vaccine efficacy` <- NA
scenarios$`Vaccine efficacy`[scenarios$vaccine.efficacy == 0.5] <- "50%"
scenarios$`Vaccine efficacy`[scenarios$vaccine.efficacy == 0.7] <- "70%"
scenarios$`Vaccine efficacy`[scenarios$vaccine.efficacy == 0.9] <- "90%"

scenarios <- scenarios %>% arrange(- size.one.arm.equal)

scenarios$plausible <- NA
scenarios$plausible[scenarios$RR.wheeze.rsvh == 4 & scenarios$rsv.attack.rate == 0.17] <- "Implausible"
scenarios$plausible[scenarios$RR.wheeze.rsvh == 2.6 & scenarios$rsv.attack.rate == 0.17] <- "Less likely"
scenarios$plausible[scenarios$RR.wheeze.rsvh == 4 & scenarios$baseline.risk.wheeze == 0.20] <- "Implausible"
```

We also create a wide format of the data, in case it is useful later.

``` r
scenarios <- scenarios %>% arrange(rsv.attack.rate, vaccine.efficacy, percent.altered, RR.wheeze.rsvh)

table2 <- scenarios %>% 
  filter(baseline.risk.wheeze == 0.049) %>% select(vaccine.efficacy, rsv.attack.rate,
                                                        RR.wheeze.rsvh, baseline.risk.wheeze, percent.altered,
                                                        RR.wheeze.vacc) %>% 
  tidyr::spread(key = RR.wheeze.rsvh, value = RR.wheeze.vacc, sep = ".")

table3 <- scenarios %>% 
  filter(baseline.risk.wheeze == 0.095) %>% select(vaccine.efficacy, rsv.attack.rate,
                                                        RR.wheeze.rsvh, baseline.risk.wheeze, percent.altered,
                                                        RR.wheeze.vacc) %>% 
  tidyr::spread(key = RR.wheeze.rsvh, value = RR.wheeze.vacc, sep = ".")

table4 <- scenarios %>% 
  filter(baseline.risk.wheeze == 0.20) %>% select(vaccine.efficacy, rsv.attack.rate,
                                                        RR.wheeze.rsvh, baseline.risk.wheeze, percent.altered,
                                                        RR.wheeze.vacc) %>% 
  tidyr::spread(key = RR.wheeze.rsvh, value = RR.wheeze.vacc, sep = ".")

scenarios.wide <- rbind(table2, table3, table4)
```

We save the two dataset.

``` r
readr::write_csv(x = scenarios, path = "../Data/scenarios.csv")
readr::write_csv(x = scenarios.wide, path = "../Data/scenarios-wide-format.csv")
```
