# CpG site methylation prediction project pipeline 
This repository contains all components of the CpG site methylation prediction pipeline, including constructing a experimental dataset for each trait with postive sites (AD associated) and negative sites (not AD assciated), processing features, selecting top 60 important features, model parameter tuning and selection and prediction of whole genome CpG sites. 

## Tools
* Python 3.5
* R 3.5

## Prerequites
The following input files are needed:
* CSV files with 450K sites CpG ID, F statistics and p-values for differential methylation between AD patients and normal subjects for each trait 
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

**1) Preparation for prediction beyond 450K array-based sites**
There are approximately 26 million CpG sites in human genome. To reduce the workload, we select CpG sites within 100kb up-/down-stream to all TSS sites in the whole genome, reducig the number of CpG sites to approximately 8 million. Then we assign all features to these CpG sites. 

The [WGBS_all_sites_feature_preprocess.py](https://github.com/xsun28/CpGMethylation/blob/master/code/features_preprocess/WGBS_all_sites_feature_preprocess.py) script available in  the [features_proprocess](https://github.com/xsun28/CpGMethylation/tree/master/code/features_preprocess) directory probacesses the CpG sites in the whole human genome in batches of 2 million sites. 

This step generates 8(?) HDF5 files corresponding 8 batches of CpG sites and their feature values:
> - 1. all_features_0_2000000
> - 2.
> - 3.
> - ....







```python
import pandas 
