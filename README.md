# *ProGENI* - Prioritization of Genes Enhanced with Network Information
#### Amin Emad (email: emad2 (at) illinois (dot) edu)
#### KnowEnG BD2K Center of Excellence
#### University of Illinois Urbana-Champaign

# Motivation
Identification of genes whose basal mRNA expression can predict a phenotype (e.g. sensitivity of tumor cells to different treatments) in an important task in bioinformatics. For example, in the context of genes predictors of drug sensitivity, screening the expression of these genes in the tumor tissue may suggest the best course of chemotherapy or suggest a combination of drugs to overcome chemoresistance. We present ProGENI (Prioritization of Genes Enhanced with Network Information) a novel computational method to identify such genes by leveraging their basal expressions and prior knowledge in the form of an interaction network. ProGENI is based on identifying a small set of genes where a combination of their expression and the activity level of the network module surrounding them shows a high correlation with drug response, followed by the ranking of the genes based on their relevance to this set using random walk techniques.

The figure below depcits the method overview in the context of drug response. (a) shows an overview of the ProGENI method, while (b) shows how bootstrap sampling can be used in order to obtain robust ranking. 

![Method Overview](/images/Pipeline_ProGENI.png)

# Requirements

In order to run the code, you need to have Python 3.5 installed. In addition, the code uses the following python modules/libraries which need to be installed:
- [Numpy](http://www.numpy.org/)
- [Scipy](https://www.scipy.org/)
- [Pandas](http://pandas.pydata.org/)
- [Sklearn (scikit-learn)](http://scikit-learn.org/stable/)

Instead of installing all these libraries independently, you can use prebulit Python distributions such as [Anaconda](https://www.continuum.io/downloads), which provides a free academic subscription.

# Input files

### Description of required inputs:
#### Gene expression (features) file:
This is a genes x samples csv file where the first column contains name of genes and the first row contains name/IDs of the samples. ProGENI assumes that the expression of each gene (across all samples) follows a normal distribution. As a result, we recommend you perform proper transformation on your expression data (e.g. log2 transform on microarray data) to satsify this condition for best results. NAs are not allowed in this file. 

Example Gene expression file:

|  | sample_1 | sample_2 | sample_3 |
| :--- | :--- | :--- | :--- |
| G1 | 0.24 | 0.67 | 2.12 |  
| G2 | 0.34 | -1.34 | 0.45 |
| G3 | 1.51 | 0.05 | -0.22 |
| G4 | 0.03 | 0.55 | 1.15 |
| G5 | -0.23 | 0.23 | 0.55 |
| G6 | 0.94 | 0.33 | 1.12 |


#### Phenotype (response) file:
This is a phenotype x samples csv file where the first column contains name of different phenotypes (e.g. different drugs) and the first row contains name/IDs of the samples. Make sure that the samples are ordered the same as the gene expression file (i.e. the first row of both files hould be identical). NAs are allowed in this file. 

Example phenotype file:

|  | sample_1 | sample_2 | sample_3 |
| :--- | :--- | :--- | :--- |
| drug_1 | 0.65 | 0.12 | 1.45 |  
| drug_2 | 1.67 | NA | 2.45 |
| drug_3 | 2.51 | 0.56 | 0.34 |


#### Network edge file:
This is a csv file which contains information on gene-gene interactions. The first should be the header of the file. The network should be represented as a three-column format where each edge in the network is represented as a row in the file: the first two columns contain name of genes and the third column shows the (positive) weight (e.g. representing confidence) corresponding to this relationship. If the set of genes in the network is slightly different from the set of genes in the gene expression data, ProGENI will focus on the intersection of the genes.  

Example network edge file:

| node_1 | node_2 | weight |
| :--- | :--- | :--- |
| G1 | G4 | 777 |
| G1 | G6 | 232 |
| G2 | G4 | 999 |
| G2 | G7 | 131 |
| G4 | G5 | 444 |
 

# Running ProGENI
### With default settings
There are only 3 required (positional) arguments that needs to be specified by the user:
- input_expression: name of the csv file containing the gene expression data
- input_response: name of the csv file containing the phenotype data
- input_network: name of the csv file containing the network edges
By default, ProGENI assumes that all these files are located in the current directory. Given these arguments, one can run ProGENI with default settings. The results will be saved in a file called "results.csv" in the current directory, which contains the ranked list of genes for each response. Only genes shared between the network and gene expression data will be included in the results. The following line shows how to run ProGENI:
```
python3 ProGENI.py gene_expr.csv phenotype.csv network.csv
```

### With advanced settings
In addition to the positional arguemtns, one can use the following optional arguments to change the default settings.
- -o, --output (string, default='results.csv'): name of the file containg the results
- -do, --directory_out (string, default='./'): directory for the results
- -de, --directory_expression (string, default='./'): directory containing the gene expression file
- -dr, --directory_response (string, default='./'): directory containing the response file
- -dn --directory_network (string, default='./'): directory containing the network file
- -s, --seed (integer, default=1011): the seed for the pseudo random generator used in bootstrap sampling
- -nr, --num_RCG (integer, default=100): number of genes in the response-correlated gene (RCG) set
- -pt, --prob_restart_trans (float, default=0.5): restart probability of RWR to network-transform gene expression
- -pr, --prob_restart_rank (float, default=0.5): restart probability for RWR used to rank nodes w.r.t. RCG
- -t, --tolerance (float, default=1e-8): residual tolerance used to determine convergence of RWR
- -mi, --max_iteration (integer, default=100): maximum number of iterations used in RWR
- -nb, --num_bootstrap (integer, default=1): number of bootstrap samplings
- -pb, --percent_bootstrap (integer, default=100): percent of samples for bootstrap sampling (between 0-100)

For example, to run Robust-ProGENI with 80% bootstrap sampling and 50 times repeat and save the results in a file called "results_80_50.csv" one can use the following line:
```
python3 ProGENI.py gene_expr.csv phenotype.csv network.csv -o results_80_50.csv -nb 50 -pb 80
```

### Sample inputs and outputs:
To test whether ProGENI runs as expected on your machine, you can use the sample inputs in the folder "sample_data" and run ProGENI with num_RCG=2 and other arguments set with default values. The results should match the file "results_sample.csv".
```
python3 ProGENI.py gene_expr_sample.csv response_sample.csv network_sample.csv -nr 2
```

Please note that the network used in the manuscript can be obtained from "9606.protein.links.detailed.v10.txt.gz" and downloaded from the [STRING database](http://string-db.org/cgi/download.pl?UserId=jQZ9rmKZ7UaW&sessionId=scBzK1XjdBol&species_text=Homo+sapiens). The gene expression and drug response datasets used in the mansucript are also publically available (see the accompanying paper for more details).
