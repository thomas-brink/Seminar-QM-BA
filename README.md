# Seminar-QM-BA
Code for Seminar in QM &amp; BA.

The files in this repository belong to the paper "A Certainty-Based Approach for Dynamic Hierarchical Classification of Product Order Satisfaction". This paper models the classification of customer satisfaction belonging to product orders at an online retailer in the Netherlands using hierarchical classification over time with a certainty-based blocking approach. The main components of this paper are the Hierarchical Classification Over Time (HCOT) and Certainty-based Automated Thresholds (CAT) procedures. These two components are outlined in corresponding code files below. In addition, we provide code for data cleaning and preparation, dynamic data generation, training and validation, and baseline comparison. The code for implementing hierarchical classifiers is based on https://github.com/davidwarshaw/hmc.

The steps for running our code are as follows.
1. Run the `Data_Cleaning_Preparation' file. This file loads in the data received from the retailer and creates a .csv file that contains the cleaned and prepared data.
2. Load in the 'dynamic_input_data' file. This file creates dynamic data variables and the input data to be used for the CAT and HCOT procedures.
3. Run the functions in the 'validation' file to obtain the optimal classifiers belonging to CAT-HCOT and the flat CAT-HCOT baseline.
4. Load in the function from the 'CAT' file to implement the certainty-based automated thresholds (CAT) algorithm.
5. Load in the functions from the 'HCOT' file and run the 'dynamicHierarchicalClassifier' function (in our paper, we use START=0 and END=10 days) to run the classifier belonging to the CAT-HCOT framework.
6. Compare the performance of CAT-HCOT with the two baseline approaches by running the functions included in the 'baseline' file (in our paper, we use START=0 and END=10 days).

Instead of steps 3-6, we also provide the 'seminar_hierarchical_classifiers' file, which can be used for validation, CAT, HCOT and the baselines as well. Note that this code does rely on the cleaned and prepared data as well as the dynamic input data functions from steps 1 and 2.  

Please note that some of the code requires the user to select a path from which to read input or to which to write output. These paths are currently kept general and should be filled in by the user. 

## PaperCAT-HCOT.pdf

File with our respective research paper. 

**Abstract**: Online retailers collaborate more and more with partners to sell products via their platform, making it increasingly important to preserve platform quality. In this paper, we predict the quality of matches between sellers and customers associated with product orders, where we face a trade-off between the accuracy and timeliness of our predictions. To deal with this trade-off, we introduce the Hierarchical Classification Over Time (HCOT) algorithm, which dynamically classifies product orders using top-down, non-mandatory leaf-node prediction. To enforce a blocking approach with respect to the tree-based class hierarchy, we introduce the Certainty-based Automated Thresholds (CAT) algorithm, which automatically computes optimal thresholds at each node in the hierarchy. The resulting CAT-HCOT algorithm has the ability to provide both accurate and timely predictions while specifically tailoring the hierarchical classification approach to the domain of customer satisfaction in online retailing. CAT-HCOT obtains a predictive accuracy of 94%. In terms of timeliness, CAT-HCOT classifies 40% of product orders on the order date itself, 80% of product orders within five days after the order date, and 100% of product orders after 10 days. In contrast, a static classification approach is unable to capture the accuracy vs. timeliness trade-off. Also, CAT-HCOT outperforms a flat, one-level hierarchy baseline method in terms of hierarchical precision and recall scores.

## Data_Cleaning_Preparation.ipynb

This file includes the code which is used to clean and prepare the data sets received from the online retailer. Firstly, we rename some of the columns in this data set. Secondly, we remove noisy observations from the data (as is explained in Appendix B in the research paper). Thirdly, we provide code for variables that we created ourselves. Fourth and last, code for creating Figure 1 in the research paper (determinant availability) is provided. Note that the paths where the data files from the retailer originate and to which the cleaned and prepared data should be written should be specified by the user.

## dynamic_input_data.py

This file includes functions which are used to create the input data (belonging to dynamic variables) for the HCOT algorithm. 

- addKnownColumns(df,X): add the dynamic variables accociated with the match determinants.
- addProductColumns(df,X): add dynamic variables accociated with products. This function makes use of two supporting functions: 'addProductColumns0' and 'addProductColumnsX' based on the number of days after the order date that the dynamic variables should be computed for.
- addSellerColumns(df,X): add dynamic variables accociated with sellers.  This function makes use of two supporting functions: 'addSellerColumns0' and 'addSellerColumnsX' based on the number of days after the order date that the dynamic variables should be computed for.
- addHistoricPerformance(df, variable = 'transporterCode', X = 0): add dynamic variables associated with historic performance. This function creates 3 columns with the historic fraction of orders belonging to Happy, Unhappy and Unknown matches.
- dataX(df, DATE, X_col, y_col, historic_variable, days): main function for creating input data _X_ (explanatory) and _y_ (dependent), computed _days_ days after the date of ordering.
- initialiseData(): return a data frame with data as well as feature names, historic variables and date information. This function is used to initialise the data and create lists of variable names that will be used in subsequent code files.

## validation.py

This file includes the code for applying the validation procedure that we use for HCOT and flat CAT-HCOT (as well as the baseline approaches).

- HCOT_optimisation(): optimisation of a single hierarchy with pre-defined set of possible combination which includes the functions:
  - get_hyperspace(combination): create hyperspace for optimisation (currently supports DT, RF, NN and LR)
  - clf_hypers(params): function which distributes the hyperparameters over the 3 specific parent nodes in the hierarchy
  - objective_function(params): objective for optimising daily hierarchical classifier combination in HCOT
  - hyperopt(param_space, X_train, y_train, X_val, y_val, num_eval): actual optimisation function for a single hierarchy with pre-set classifier combination
- flat_HCOT_optimisation(): optimisation of single 'one-level hierarchy' of the flat HCOT basline which includes similar functions (supporting LR + RF):
  - flat_get_hyperspace(combination): create hyperspace for optimisation 
  - flat_objective_function(params): objective for optimising either LR or RF
  - flat_hyperopt(param_space, X_train, y_train, X_val, y_val, num_eval): actual optimisation function

## CAT.py

This file includes the CAT algorithm for both CAT-HCOT and flat CAT-HCOT
- opt_threshold(probs, node, day, certainty, option = 2, steps = 100): CAT for HCOT, using the hierarchy from our report
- flat_thresholds(probs, node, day, certainty, steps = 100): CAT for flat HCOT, using the flat hierarchy from our report

## HCOT.py

This file includes the complete HCOT algoritm together with function to test and evaluate the performance.
- ClassHierarchy(root): Class with all functions to construct a hierarchy which is thereafter used for HierarchicalClassifier, main function:
  - add_node(children, parent): add a node with corresponding childnodes to the hierachy
- HierarchicalClassifier(class_hierarchy): Class with all functions to train and test a hierarchical classifier, the main functions are:
  - fit_classifiers(classifiers): function to fit the classifier per parent node, the input is a dictionary with the parent nodes as keys and values the classifiers
  - fit(X, y): train the hierarchy with X and Y
  - predict(X): predict the values of X with the fitted hierarchy
  - predict_proba(X, threshold): predict the values of X by making use of a pre-defined threshold for each node, if threshold not reached then blocked
  - predict_proba2(X, THRESHOLDS): predict the values of X by making use of node-specific thresholds (THRESHOLDS), if threshold not reached then blocked
  - get_probabilities(X, y): get the class probabilities for each instance, this is used for training the thresholds with CAT
- initialiseData(): function to load data and retrieve features for prediction (included in dynamicHierarchicalClassifier)
- dynamicHierarchicalClassifier(START, END): CAT-HCOT algorithm to test instances over a period from START till END
- get_performance(...): function included in dynamicHierarchicalClassifier to compute daily/global performance measures while running HCOT
- precision_score_ancestors(class_hierarchy, y_true, y_pred): hierarchical precision score, input is the class_hierarchy from ClassHierarchy
- recall_score_ancestors(class_hierarchy, y_true, y_pred): hierarchical recall score, input is the class_hierarchy from ClassHierarchy
- f1_score_ancestors(class_hierarchy, y_true, y_pred, beta): hierarchical f1 score, input is the class_hierarchy from ClassHierarchy

## baseline.py

This file includes the baseline methods: static HCOT and flat CAT-HCOT
- staticHierarchicalClassifier(START, END): static HCOT (no blocking), hence predicting all instances on a single point in time
- dynamicFlatClassifier(START, END): flat CAT-HCOT (one-level hierarchy), using similar blokking approach as in CAT-HCOT

## Seminar_Hierarchical_Classifiers.ipynb

This file includes code that can be used as a replacement for the separate 'validation', 'CAT', 'HCOT' and 'baseline' code files. Note that the functions contained in this file are the same ones that are included in the previously discussed separate files. Running all parts in this single file allows the user to perform validation and testing for both CAT-HCOT and the two baselines. To do so, the following steps should be followed.

1. Import packages: import the necessary packages.
2. Load Data: load the data as created in the 'data_cleaning_preparation' code by running the initialiseData() function
3. Functions: run the functions provided at the bottom of the code.
4. Validation (flat): run the code in this section to perform validation for the flat CAT-HCOT baseline.
5. Validation (HCOT): run the code in this section to perform validation for CAT-HCOT and the static HCOT baseline. Make sure to first run the functions contained in this folder before running the output.
6. CAT-HCOT: run the code in this folder to perform testing following CAT-HCOT.
7. Base Case 1 (Static): run the code in this folder to perform testing following the static HCOT baseline.
8. Base Case 2 (Flat): run the code in this folder to perform testing following the flat CAT-HCOT baseline.
