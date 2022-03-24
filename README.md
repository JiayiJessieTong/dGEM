dGEM: Decentralized algorithm for Generalized mixed Effect Models
==============================================


## Outline

1. dGEM workflow
2. Package requirements
3. Instructions for installing and running pda package


## dGEM Workflow
<img width="1170" alt="image" src="https://user-images.githubusercontent.com/38872447/159751836-2509fe6a-a7fa-45ec-bf50-35a403d45f89.png">

## Package Requirements
- A database with clear and consistent variable names
- On Windows: download and install [RTools](http://cran.r-project.org/bin/windows/Rtools/) 


## Instructions for Installing and Running pda Package

Below are the instructions for installing and then running the package.

### How to install the pda package
There are several ways in which one could install the `pda` package. 

1. In RStudio, create a new project: File -> New Project... -> New Directory -> New Project. 

2. Execute the following R code: 

```r
# Install the latest version of PDA in R:
install.packages("pda")
library(pda)

# Or you can install via github:
install.packages("devtools")
library(devtools)
devtools::install_github("penncil/pda")
library(pda)
```

### How to run pda examples

Below are two ways to run the pda examples. 

In the toy example below we aim to analyze the association of lung status with age and sex using logistic regression, data(lung) from 'survival', we randomly assign to 3 sites: 'site1', 'site2', 'site3'. We run the example in local directory. In actual collaboration, our pda-ota (pda over the air) platform can be used to coordinate the project using a cloud-based server. 

You can either 

#### *Run example with demo()*

```r
demo(dGEM)
``` 
or
####  *Run example with code*

Step 0: load related R packages and prepare sample data

```r
# load packages
require(survival)
require(data.table)
require(pda)

# sample data, lung, from "survival" package
data(lung)

# create 3 sites, split the lung data amongst them
sites = c('site1', 'site2', 'site3')
set.seed(42)
lung2 <- lung[,c('time', 'status', 'age', 'sex')]
lung2$sex <- lung2$sex-1
lung2$status <- ifelse(lung2$status == 2, 1, 0)
lung_split<-split(lung2, sample(1:length(sites), nrow(lung), replace=TRUE))

## fit logistic reg using pooled data
fit.pool <- glm(status ~ age + sex, family = 'binomial', data = lung2)

``` 
Step 1: Initialization

```r
# ############################  STEP 1: initialize  ###############################
## lead site1: please review and enter "1" to allow putting the control file to the server
control <- list(project_name = 'Lung cancer study',
                step = 'initialize',
                sites = sites,
                heterogeneity = TRUE,
                model = 'dGEM',
                family = 'binomial',
                outcome = "status",
                variables = c('age', 'sex'),
                variables_site_level = c('volume'),
                optim_maxit = 100,
                lead_site = 'site1',
                upload_date = as.character(Sys.time()) )

##' specify control file (to be shared with others)
pda(site_id = 'site1', control = control, dir = getwd())

##' run dGEM step 1 under site 3
pda(site_id = 'site3', ipdata = lung_split[[3]], dir=getwd())

##' run dGEM step 1 under site 2
pda(site_id = 'site2', ipdata = lung_split[[2]], dir=getwd())

## run dGEM step 1 under site 1
##' control.json is also automatically updated
pda(site_id = 'site1', ipdata = lung_split[[1]], dir=getwd())


``` 
Step 2: Calculate hospital effects

```r
#' ############################'  STEP 2: derive  ###############################
##' run dGEM step 2 under site 3
pda(site_id = 'site3', ipdata = lung_split[[3]], hosdata = c('volume' = 3000), dir=getwd())

##' run dGEM step 2 under site 2
pda(site_id = 'site2', ipdata = lung_split[[2]], hosdata = c('volume' = 5000), dir=getwd())

##' run dGEM step 2 under site 1
##' control.json is also automatically updated
pda(site_id = 'site1', ipdata = lung_split[[1]], hosdata = c('volume' = 10000), dir=getwd())
``` 
Step 3: Calculate counterfactural rates

```r
#' ############################'  STEP 3: estimate  ###############################
##' run dGEM step 3 under site 3
pda(site_id = 'site3', ipdata = lung_split[[3]], dir=getwd())

##' run dGEM step 3 under site 2
pda(site_id = 'site2', ipdata = lung_split[[2]], dir=getwd())

##' run dGEM step 3 under site 1
##' control.json is also automatically updated
pda(site_id = 'site1', ipdata = lung_split[[1]], dir=getwd())


``` 

Step 4: Calculate standardized event rates

```r
#' ############################'  STEP 4: synthesize  ###############################
##' run final step in the lead site or coordinating center
pda(site_id = 'site1', ipdata = lung_split[[1]], dir=getwd())

##' the PDA dGEM is now completed!
``` 


