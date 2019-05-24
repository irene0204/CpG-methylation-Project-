# CpG site methylation prediction project pipeline 
This repository contains all components of the CpG site methylation prediction pipeline, including constructing a experimental dataset for each trait with postive sites (AD associated) and negative sites (not AD assciated), processing features, selecting top 60 important features, model parameter tuning and selection and prediction of whole genome CpG sites. 

## Tools
* Python 3.5
* R 3.5

## Prerequites
The following input files are needed:
* CSV files with 450K sites CpG ID, F statistics and p-values for differential methylation between AD patients and normal subjects for each trait from the ROSMAP study
* a TXT file with the whole genome spread across 200 base-pair intervals 
* BED files with sites window ID and feature values for 1806 epigenomic features related to histone modification, TF binding, open chromatin and RNA Pol II/III binding
* TSV.GZ files with site coordinates and CADD scores 
* TSV.BGZ files with site coordinates and DANN scores 
* TAB.BGZ files with sites coordinates and EIGEN scores
* BED files with sites window ID and RNA-sequencing read counts data
* BED files with sites window ID and ATAC-sequencing read counts data
* BED files with site coordinates and WGBS read counts from various tissue/cell types 

## Running the pipeline 

The genomic coordinates in this pipeline are in hg19. The original WGBS datasets are in hg38 and need to be converted to hg19. A CSV file containing all WGBS sites with their genomic location in hg19 can be generated using the [WGBS_allsites_preprocess.py](https://github.com/xsun28/CpGMethylation/blob/master/code/prediction/WGBS_allsites_preprocess.py) script available in the [prediction](https://github.com/xsun28/CpGMethylation/tree/master/code/prediction) directory. In the code below, it is assumed that this conversion is completed and `all_wgbs_sites_winid.csv` was generated, which contains the genomic location in both hg38 and hg19 and window ID for all WGBS sites. 

**1) Preparation for prediction beyond 450K array-based sites-asssign feaatures to WGBS sites**

There are approximately 26 million CpG sites in human genome. To reduce the workload, we select CpG sites within 100kb up-/down-stream to all TSS sites in the whole genome, reducig the number of CpG sites to approximately 8 million. Then we assign all features to these CpG sites. 

The [WGBS_all_sites_feature_preprocess.py](https://github.com/xsun28/CpGMethylation/blob/master/code/features_preprocess/WGBS_all_sites_feature_preprocess.py) script available in  the [features_proprocess](https://github.com/xsun28/CpGMethylation/tree/master/code/features_preprocess) directory, which processes the CpG sites in the whole human genome in batches of 2 million sites. 

This step generates 8(?) HDF5 files corresponding 8 batches of CpG sites and their feature values:
``` 
1. all_features_0_2000000
2.
3.
....
```

**2) Assign features to all 450K sites**



**3) Experimental dataset construction for each trait**

For each trait, we pick out positive sites and negative sites from the ROSMAP CSV files to build a experimental set for future feature selection and model training processes. We set up p-value thresholds for positive site and negative sites so that for each trait, the experimental dataset contains appproximately 140 positives sites abd for each of them, we pick out 10 matching negative sites with the closest beta values. 

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



















