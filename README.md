# CpG site methylation prediction project pipeline 
This repository contains all components of the pipeline for predicting novel Alzheimer's Diseasr (AD)-associated CpG sites across the human genome, including experimental set construction, features collection/processing, features selection and ensemble learning, for each of the AD-associated trait of interest. 

## Tools
* Python 3.5
* R 3.5

## Prerequites
The following input files are needed:
* CSV files with summary level data from the ROSMAP study. For each trait, the file includes CpG ID, F statistics and p-values (null hypothesis: AD samples have the same methylation leve as control samples) for *CpG sites whose methylation level was measured using Illumina 450K array (i.e. 450K sites)*.
* a TXT file with the whole human genome spread across 200 base-pair intervals `wins.txt`
* BED files with window IDs of *all CpG sites across the whole human genome (i.e. WGBS sites)* and values of the 1806 features used in our previously published work on DIVAN.  `1806.bed`
* TSV.GZ files with genomic locations of WGBS sites and CADD scores `CADD.tsv.gz`
* TSV.BGZ files with genomic locations of WGBS sites and DANN scores `DANN.tsv.bgz`
* TAB.BGZ files with genomic locations of WGBS sites and EIGEN scores `EIGEN.tab.bgz`
* BED.GZ files with genomic locations of WGBS sites and GWAVA scores `GWAVA.bed.gz`
* BED files with window IDs of WGBS sites and RNA-sequencing read counts data `RNASEQ.bed`
* BED files with window IDs of WGBS sites and ATAC-sequencing read counts data `ATACSEQ.bed`
* BED files with genomic locations of WGBS sites and WGBS read counts data `wgbs_readcounts.bed`


## Running the pipeline 

**0) Preliminary step: hg38 to hg19 conversion**

The genomic coordinates in this pipeline are 1-indexed/in hg19. The original WGBS datasets are 0-indexed/in hg38 and therefore need to be converted. This conversion can be completed by running the[WGBS_allsites_preprocess.py](https://github.com/xsun28/CpGMethylation/blob/master/code/prediction/WGBS_allsites_preprocess.py) script available in the [prediction](https://github.com/xsun28/CpGMethylation/tree/master/code/prediction) directory. The number of studied WGBS sites is reduced from `approximatly 28 million` to `approximatly 26 million` after this conversion due to using LiftOver (inconsistent chromosome, multiple conversion results, etc). 

``` 
WGBS_allsites_preprocess.py ${wgbs_readcounts.bed} ${wins.txt}
```
The file `${wgbs_readcounts.bed}`contains the 0-indexed/hg38 genomic locations of WGBS sites.

This step generates `all_wgbs_sites_winid.csv`,which contains the genomic locations in both hg38 and hg19 and window IDs for WGBS sites. 




**1) Asssigning feature values to WGBS sites**

To prepare for future prediction of AD-associated WGBS sites, we first assign all (2256) feature values to WGBS sites. 

By running the [WGBS_all_sites_feature_preprocess.py](https://github.com/xsun28/CpGMethylation/blob/master/code/features_preprocess/WGBS_all_sites_feature_preprocess.py) script in the [features_proprocess](https://github.com/xsun28/CpGMethylation/tree/master/code/features_preprocess) directory, we can processes features of WGBS sites in batches of 2 million for the consideration of memory limit. 

``` 
WGBS_all_sites_feature_preprocess.py ${all_wgbs_sites_winid.csv} ${1806.bed} ${CADD.tsv.gz} \
    ${DANN.tsv.bgz} ${EIGEN.tab.bgz} ${GWAVA.bed.gz} ${RNASEQ.bed} ${ATACSEQ.bed} ${wgbs_readcounts.bed}

```


This step generates HDF5 files for all batches of WGBS sites and their feature values:
``` 
all_features_0_2000000
....
```

**2) Assign features to all 450K sites**



**3) Experimental set construction for each trait**

For furture model training purpose, we constructed a experimental set for each trait. In each set, we include positive sites (signficantly associated with AD) and negative sites (not significantly associated with AD). The inclusion criteria are as follows:

a) Select positive sites whose p-values are below trait-specific threshold
b) For each selected positive site, select 10 negative sites that:

* have p-values greater than 0.4
* have the same methylation status (either hyper- and hypo) as the positive site
* have the closest β-values as the positive site 

The above process is achieved by running the [AD_sites_selection.py](https://github.com/xsun28/CpGMethylation/blob/master/code/sites_selection/AD_sites_selection.py) script available in the [site_selection](https://github.com/xsun28/CpGMethylation/tree/master/code/sites_selection) directory. 

This step outputs 7 CSV files under 7 trait folders and 1 CSV file for all 450K sites: 
``` 
all_sites_winid.csv
```
which contains the selected postive and negative sites with their CpG ID, chromosome, coordinate, p-value, beta value and label, negative sites are labeled as 0 and positive sites are labeled as 1. 

and 1 CSV file for all 450K sites:
``` 
all_450k_sites_winid.csv
```
which contains all 450k sites with their CpG ID, chromosome, coordinate, p-value, beta value 


**4) Assign feature values to the experimental dataset for each trait**

Using the [all_features_preprocess.py](https://github.com/xsun28/CpGMethylation/blob/master/code/features_preprocess/all_features_preprocess.py) script available in the [features_proprocess](https://github.com/xsun28/CpGMethylation/tree/master/code/features_preprocess) directory, we obtain feature values for the selected experimental dataset for each trait.

This step generates 7 HDF5 files under 7 trait folders:
``` 
all_features
```
which contains all feature values for the complete dataset of each trait.


**5) Feature selection for each trait**

Considering we have 2069 features in total and experimental dataset of limited size, we want to reduce the number of features to avoid the "small N large P" problem. 

The feature selection process is achived by running the 

[feature_selection.py](https://github.com/xsun28/CpGMethylation/blob/master/code/features_selection/feature_selection.py) script available in the [features_selection](https://github.com/xsun28/CpGMethylation/tree/master/code/features_selection) directory, in which we:

* split train/test data on 9:1 ratio and scaled the train data (?????) 
* used random forest, xgboost, logistic regression and linear_SVC to fit the train data and pick out top 100 significant features 
* rank selected features by the number of classifiers that select it (n)
* calculate the p-values which indicates the differences of the selected features between positive sites and negative sites 
* rank selected features first by desceding n and then by ascending p-value 
* select top 60 ranked features for each trait 

This step outputs a CSV file:
``` 
feature_stats.csv 
```
which contains information of the selected feaures, including feature name, p-value, and n

and a HDF5 file:
``` 
selected_features 
```
which contains the train and test dataframe with only the selected features, and the sample weights for train and test data. 


**6) Model parameter tuning and model selection for each trait**

We use 4 base classifiers, random forest, xgboost, logistic regression and linear_SVC. The paramters for each classifier and best combination of base classifiers are selected using the 

[ModelSelectionTuning.py](https://github.com/xsun28/CpGMethylation/blob/master/code/models/ModelSelectionTuning.py) script 

available in the [models](https://github.com/xsun28/CpGMethylation/tree/master/code/models) directory, which:

* uses the train data (generated from 5) with 3-fold cross-validation to select the best parameters
* uses the train/test data (generated from 5) with 10-fold cross-validation to evaluate all possible combination of classifiers
* calculates average AUC, F1-score, etc across all folds 
* select the best combination of classifiers 
* save selected models (??????)

**7) Predict WGBS sites for each trait**



















