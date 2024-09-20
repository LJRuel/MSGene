## MSGene
* Multistate modeling

Welcome to our implementation for estimating lifetime probabilities iterated over a multistate model!! 

This repository contains codes for implementing MSGene, a framework for dynamic risk modelling of coronary artery disease using genetic risk and the electronic health record with multistate modelling. Detailed description of MSGene development can be found in the manuscript [MSGene: Derivation and validation of a multistate model for lifetime risk of coronary artery disease using genetic risk and the electronic health record](https://www.medrxiv.org/content/10.1101/2023.11.08.23298229v1)

To install:

```{r}
devtools::install_github("surbut/MSGene");
library(MSGene)
```

To run this code with minimal example and compute the model fit, we create a sample data frame with the following columns. Note that cases are written as 2, controls as 1. If an individual does not have a condition, his censor age is the age of last follow up or death.

Please see [ukbpheno](https://github.com/niekverw/ukbpheno/tree/master) for directions on deriving case control status from your data. Here statin and antihtn refer to the use of statins or antithn, and age (or NA) of prescription. This is to produce the training and computation listed in Urbut et al. 

We include a simulated data set with the appropriate column names. Here, field f.31.0.0 represents the sex indicator from the UKB. UK Biobank is open to all researchers with appropriate credentials and IRB approval, [UK Biobank](https://www.ukbiobank.ac.uk).

#### Column names

The column names are described in [ukbpheno](https://github.com/niekverw/ukbpheno/tree/master) and below the as follows:

* **cad.prs** : scaled N(0,1) coronary artery disease PRS, as available in ukbshowcase field 26202 through 26289 and described [here](https://biobank.ndph.ox.ac.uk/showcase/label.cgi?id=301).
* **f.31.0.0**: sex (male/female, 1/0)
* **Trait_0_Any**: binary indicator for case (2) or control status (1)
* **Trait_0_censor_age**: minimum age of first diagnosis, last censored, or death (whichever comes first)
* **statin/antihtn**: binary indicator of medication use
* **statin_age/htn_age**: NA (age) if never using/age of first prescription
* **smoke**git : smoking status at baseline

# Load the sample data

```{r}
df2 <- readRDS(system.file("data", "msgene_sampledf.rds", package = "MSGene"))
head(df2)
```


# set a training set

Now, we select a portion of the data for training, and convert to a data table
```{r}
train=data.table(df2[1:(nrow(df2)*0.80),])
```

# Parameter choice

Here we choose the ages we will consider and the states we will move through.

```{r}
ages=c(40:80)
nstates=c("Health", "Ht","HyperLip","Dm","Cad","death","Ht&HyperLip","HyperLip&Dm","Ht&Dm","Ht&HyperLip&Dm")
```

## Fit our model

Now we're ready to fit our model! Sit tight, this step takes between 60-90 seconds  ... but it's worth it.

```{r}
modelfit = fitfunc2(
  df_frame=data.table::data.table(train),
  ages = ages,
  nstates = nstates,
  mode = "binomial",
  covariates = "cad.prs+f.31.0.0+smoke+antihtn_now+statin_now")
```

Here you can see all the we produce:

```{r}
print(attributes(modelfit))
```

This produces the model fit. For more details, including state to state coefficient extraction, lifetime probabilities of disease under treated and untreated strategy, please see our test vignette at 
https://surbut.github.io/MSGene/vignette.html 


For more flexible states inputting an array of state positions and possible transitions with any names, please see:
https://surbut.github.io/MSGene/usingMarginal.html


