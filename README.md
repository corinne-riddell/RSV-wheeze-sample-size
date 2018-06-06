# Informing randomized clinical trials of respiratory syncytial virus vaccination during pregnancy to prevent recurrent childhood wheezing: a sample size analysis
### [Corinne A Riddell](corinneriddell.com), Bhat N, Bont LJ, Dupont WD, Feikin DR, Fell DB, Gebretsadik T, Hartert TV, Hutcheon JA, Karron RA, Nair H, Reiner RC, Shi T, Sly PD, Stein RT, Wu P, Zar HJ, Ortiz JR for the WHO Technical Working Group on Respiratory Syncytial Virus Vaccination During Pregnancy to Prevent Recurrent Childhood Wheezing

# Project overview

This repository contains all of the data and code that goes alongside the manuscript "Informing randomized clinical trials of respiratory syncytial virus vaccination during pregnancy to prevent recurrent childhood wheezing: a sample size analysis" by Riddell et al. The manuscript is currently under review at the *American Journal of Respiratory and Critical Care Medicine* (AJRCCM).

The objective of this paper was to estimate sample size requirements for studies of maternal RSV vaccination during pregnancy, under plausible scenarios in which maternal vaccination prevents early infant RSV. Under a variety of scenarios, we assume that the prevention of early infant RSV prevents development of childhood wheezing illness and estimate the sample size required to observe such a causal effect.

# Organization of this Github repository

### Code/ directory

The Code folder contains the two analysis files: Code/1_create-dataset-and-analyze creates the analytic dataset defined by the combination of parameter scenarios. For each scenario, we calculate the require sample size and the number of mother-infant pairs who need to be vaccinated to prevent one case of childhood wheezing illness.

Code/2_manuscript-and-figures contains the manuscript text, and the codes to summarize our findings and produce the figures and tables. It also contains the supplementary tables. The .Rmd version shows the code and the .md version contains the rendered tables that are formatted for online viewing.

### Data/ directory

The Data directory stores the analytic datasets that are created and saved as outputs from Code/1_create-dataset-and-analyze.

### Plots/ directory and Images/ directory

The Plots/ directory stores the manuscript figures 2-4. Figure 1 is a conceptual causal diagram that is store as an image in the Images/ directory. 

# How to replicate these analyses

If your intention is to replicate our analysis and you are familiar with GitHub, please clone this repository. All of the analysis is contained within the files "Code/1_create-dataset-and-analyze" and "Code/2_manuscript-and-figures", and you can run these files within RStudio by running all of the code chunks within these R markdown documents. Remember first to open the .Rproj file within RStudio so that all of the relative pathways are correct.

If you are unfamiliar with GitHub, but familiar with R and RStudio, you may wish to download the .Rproj file and analysis files by navigating to them and downloading the raw versions and open them locally within RStudio. You will also need to download the data file that is referenced in the code chunk `load-libraries-and-data` in Code/1_create-dataset-and-analyze. This dataset can be found in the Data/ folder. You can then run the code chunks within RStudio to replicate the analysis. To replicate the paper, you can use the "Knit" button to knit to word or html.

Happy Replicating! Reproducible Research FTW!
