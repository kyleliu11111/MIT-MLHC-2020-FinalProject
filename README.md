# Early Clinical Diagnosis & Lab Test Prediction in Pediatrics

In this project, we apply several predictive models to the tasks of early Sepsis diagnosis and abnormalitiy detection in WBC count, hemoglobin, and procalcitonin amounts as measured by lab tests. We additionally train and evaluate each of the models on the well-studied baseline tasks of ICU length of stay and in-ICU mortality prediction. The [*PIC database*](https://www.nature.com/articles/s41597-020-0355-4) is the source of all training and test data. 

Our main contributions are:
  1. The exploration and analysis of the relatively new PIC dataset with traditional time series analysis methods
  2. A modified Transformer-based architecture centered on continuous-timestep positional embeddings to allow the processing of PIC's time series data in full resolution.

In order to reproduce our results, one should request access to the PIC database on [*Physionet*](https://physionet.org/content/picdb/1.0.0/).

See `final_paper.pdf` for a detailed report on the details and motivations behind the prediction models we used, as well as some analysis for our results.

# Preliminaries 

Because there are no gold labels for positive diagnosis of Sepsis in PIC, we must generate silver labels via a rule-based approach for training and testing. Here, we follow the method of [*Predicting Severse Sepsis using Machine Learning*](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6798083/), defining a Septic patient as one with a SIRS score of at least 2, and a suspicion of infection. For an indication of this, we look for certain antibiotics in the patient's prescription history. See [*Sepsis-3 in MIMIC*](https://github.com/alistairewj/sepsis3-mimic).

- `ExtractSeptic.ipynb` contains the code used to extract a list of the ID's of Septic patients

# Baseline Models 

For our prediction models, we focus on utilizing the time-series of vital sign data provided in PIC to diagnose Sepsis, and predict in-ICU mortality, length of stay, and abnormality in WBC count, hemoglobin, and procalcitonin. See the exact features selected in `final_paper.pdf`. 

As with nearly all time-series of clinical data, the PIC vital-sign data has a very high (~80%) missingness rate. Moreover, measurements are taken at irregular intervals. This introduces two main challenges to overcome in order to apply traditional architectures: (1.) Imputation of missing values, and (2.) Regularization of time steps

For our baseline, we follow work done in MIMIC, and solve (1.) via simple forward imputation, and (2.) via hourly bucketing. After performing these two steps, we end up with a rough approximation of the information available in the original time series, but this apporoximation is easily processed with traditional NLP architectures. 

- `BasicBaselines_LOS.ipynb` and `BasicBaselines_Mortality.ipynb` contain details on these two steps. They then train and evaluate Random Forest and LSTM models on the baseline tasks of ICU length of stay, and in-ICU mortality respectively. 

Motivated by existing clinical practice, for our more specific tasks of early Sepsis diagnosis and abnormalitiy detection in WBC count, hemoglobin, and procalcitonin amounts, we modify our models to output a rolling assessment of Sepsis/abnormality risk. Specifically, we allow our models to ingest 24-hour windows of data, and ask them to predict risk in a specified prediction window (usually a 12-hour window after a gap time of 6 hours to prevent label leakage).

- See `Rolling_Baselines_LabTests.ipynb` and `Rolling_Baselines_Sepsis.ipynb` for details.

# Transformer-based Architecture 

We introduce an architecture modified from [*Attend and Diagnose: Clinical Time Series Analysis using Attention Models*](https://arxiv.org/abs/1711.03905) that utilizes a sinusoidal positional encoding for irregular measurement timesteps to bypass the need for imputation or bucketing. We implemented this new architecture in PyTorch, training and evaluating it on the ICU length-of-stay and in-ICU mortality tasks. We adopted the transformer backbone from [*The Annotated Transformer*](http://nlp.seas.harvard.edu/2018/04/03/attention.html).

- See `Transformer_Model_Mortality_LOS.ipynb` for details

## Acknowledgements

We would like to thank Dr.  Chase Parsons for his clinical insights in the project.  We wouldalso  like  to  thank  Matthew  McDermott  and  Professor  Peter  Szolovits  for  their  valuable feedback and help in framing the project.


