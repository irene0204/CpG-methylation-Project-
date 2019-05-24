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

T

```python
import pandas 
