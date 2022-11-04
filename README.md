# MAPDB_Project
# Analysis of Covid-19 papers

This distributed computing project will be focused on the analysis of 1000 papers about
COVID-19, SARS-CoV-2, and related coronaviruses. The dataset is a sub-sample of 1000
items taken from the original dataset that is composed of more than 75000 (and still
growing) papers. This dataset is a part of real-world research on COVID-19 named
COVID-19 Open Research Dataset Challenge (CORD-19). The research and related challenges are available on the dedicated page on Kaggle: https://www.kaggle.com/alleninstitute-for-ai/CORD-19-research-challenge

Using Dask to link 3 virtual machines (SSH protocol) in Cloud-Veneto for distributed computation


### Task 1 : Word counter distributed algorithm:


* We implement a distributed algorithm to count the occurrences of all the words inside a list of documents using Bag data-structure of DASK. Here, we explain the algorithm step by step and in the end we wrap all the steps inside a single function in order to test the computational time required by the algorithm as a function of the number of cluster workers and the number of data partitions.

### Task 2: Chek computation time with different partitions and workers:


* Even in this case do multiple runs by changing the number of partitions and workers and then describe the behavior of the timings.

### Task3:  Get the embedding for the title of the papers:


* In NLP a common technique for performing analysis over a set of texts is to transform the text into a set of vectors each one representing a word inside a document. At the end of the pre-processing the document will be transformed into a list of vectors or a matrix of n × m where n is the number of words in the document and m is the size of the vector that represents the word.


