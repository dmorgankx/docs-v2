---
title: Automated machine learning user guide | Machine Learning | Documentation for kdb+ and q
description: Top-level user-callable functions within the automated machine learning platform
author: Deanna Morgan
date: December 2020
keywords: keywords: machine learning, automated, ml, automl, fit, predict, persisted models
---

# :fontawesome-solid-share-alt: Top-level user-callable functions

:fontawesome-brands-github:
[KxSystems/automl](https://github.com/kxsystems/automl)

The top-level functions in the repository are:

<div markdown="1" class="typewriter">
.automl   **Top-level functions**
  fit                   Apply AutoML to provided features and associated targets
  getModel              Retrieve a previously fit AutoML model and use for predictions
  newConfig             Generate a new JSON parameter file for use with .automl.fit
  updateIgnoreWarnings  Update print warning severity level
  updateLogging         Update logging state
  updatePrinting        Update printing state
</div>

`.automl.fit` can be modified by a user to suit specific use cases. Where possible, the functions listed above have been designed to cover a wide range of functional options and to be extensible to a users needs. Details regarding all available modifications which can be made are outlined in the [advanced section](options.md).

The following examples and function descriptions outline the most basic vanilla implementations of AutoML specific to each supported use case. Namely, non-time series specific machine learning examples, along with time series examples which make use of the [FRESH algorithm](../../toolkit/fresh) and [NLP Library](../../nlp/index.md).


## `.automl.fit`

_Apply AutoML to provided features and associated targets_

Syntax: `.automl.fit[features;target;ftype;ptype;params]`

Where

-   `features` is an unkeyed tabular feature data or a dictionary outlining how to retrieve the data in accordance with `.ml.i.loaddset`
-   `target` is target vector of any type or a dictionary outlining how to retrieve the target vector in accordance with `.ml.i.loaddset`
-   `ftype` is the feature extraction type as a symbol (``` `nlp/`normal/`fresh ```)
-   `ptype` is the problem type being solved as a symbol (``` `reg/`class ```)
-   `params` is one of the following:
      1. Path relative to `.automl.path` pointing to a user defined JSON file for modifying default parameters
      2. Dictionary containing the default behaviours to be overwritten
      3. Null (::) indicating to run AutoML using default parameters 

returns the configuration produced within the current run of AutoML along with a prediction function which can be used to make predictions using the best model produced.

The default setup saves the following items from an individual run:

1. The best model, saved as a HDF5 file, or ‘pickled’ byte object.
2. A saved report indicating the procedure taken and scores achieved.
3. A saved binary encoded dictionary denoting the procedure to be taken for reproducing results, running on new data and outlining all important information relating to a run.
4. Results from each step of the pipeline saved to the generated report.
5. On application NLP techniques a word2vec model is saved outlining the text to numerical mapping for a specific run.

The following examples demonstrate how to apply data in various use cases to `.automl.fit`. Note that while only one example is shown for each feature extraction type, datasets with binary-classification, multi-classification and regression targets can all be used in each case. Additionally, the terminal output has only been displayed for the last example.

```q
// Non-time series (normal) regression example table
q)table:([]asc 100?0t;100?1f;desc 100?0b;100?1f;asc 100?1f)
// Regression target
q)regTarget:asc 100?1f
// Feature extraction type
q)featExtractType:`normal
// Problem type
q)problemType:`reg
// Use default system parameters
q)params:(::)
// Run example
q).automl.fit[table;regTarget;featExtractType;problemType;params]

// Non-time series (normal) multi-classification example table
q)table:([]100?1f;100?1f)
// Multi-classification target
q)classTarget:100?5
// Feature extraction type
q)featExtractType:`normal
// Problem type
q)problemType:`class
// Use default system parameters
q)params:(::)
// Run example
q).automl.fit[table;classTarget;featExtractType;problemType;params]

// NLP binary-classification example table
q)table:([]100?1f;asc 100?("Testing the application of nlp";"With different characters"))
// Binary-classification target
q)classTarget:asc 100?0b
// Feature extraction type
q)featExtractType:`nlp
// Problem type
q)ptype:`class
// Use default system parameters
q)params:(::)
// Run example
q).automl.fit[table;classTarget;featExtractType;ptype;params]

// FRESH regression example table
q)table:([]5000?100?0p;asc 5000?1f;5000?1f;desc 5000?10f;5000?0b)
// Regression target
q)regTarget:desc 100?1f
// Feature extraction type
q)featExtractType:`fresh
// Problem type
q)problemType:`reg
// Use default system parameters
q)params:(::)
// Run example
q)outputs:.automl.fit[table;regTarget;featExtractType;problemType;params]
Executing node: automlConfig
Executing node: configuration
Executing node: targetDataConfig
Executing node: targetData
Executing node: featureDataConfig
Executing node: featureData
Executing node: dataCheck
Executing node: featureDescription

The following is a breakdown of information for each of the relevant columns in the dataset

  | count unique mean      std       min          max       type
--| ---------------------------------------------------------------
x1| 5000  5000   0.5004232 0.2908372 0.0001313207 0.999641  numeric
x2| 5000  5000   0.4967023 0.2897377 0.0007908894 0.9998165 numeric
x3| 5000  5000   5.036043  2.904289  0.002741043  9.998293  numeric
x | 5000  100    ::        ::        ::           ::        time
x4| 5000  2      ::        ::        ::           ::        boolean

Executing node: dataPreprocessing

Data preprocessing complete, starting feature creation

Executing node: featureCreation
Executing node: labelEncode
Executing node: featureSignificance

Total number of significant features being passed to the models = 214

Executing node: trainTestSplit
Executing node: modelGeneration
Executing node: selectModels

Starting initial model selection - allow ample time for large datasets

Executing node: runModels

Scores for all models using .ml.mse

AdaBoostRegressor        | 0.04068134
RandomForestRegressor    | 0.04209565
Lasso                    | 0.04291184
GradientBoostingRegressor| 0.04677987
KNeighborsRegressor      | 0.05348689
LinearRegression         | 0.3415033
MLPRegressor             | 1624.88

Best scoring model = AdaBoostRegressor

Executing node: optimizeModels

Continuing to hyperparameter search and final model fitting on testing set

Best model fitting now complete - final score on testing set = 0.1974598

Executing node: predictParams
Executing node: preprocParams
Executing node: pathConstruct
Executing node: saveGraph

Saving down graphs to automl/outputs/2020.12.15/run_16.58.46.400/images/

Executing node: saveReport

Saving down procedure report to automl/outputs/2020.12.15/run_16.58.46.400/report/

Executing node: saveMeta

Saving down model parameters to automl/outputs/2020.12.15/run_16.58.46.400/config/

Executing node: saveModels

Saving down model to automl/outputs/2020.12.15/run_16.58.46.400/models/
```

!!! note Predict on new data
    Predictions can be made using the model output by `.automl.fit`. Users simply need to pass in the new feature data as shown below:
    
    ```q
    // Non-time series (normal) regression example table - training features
    q)xtrain:([]asc 100?0t;100?1f;desc 100?0b;100?1f;asc 100?1f)
    // Regression target - training target
    q)ytrain:asc 100?1f
    // Generate model
    q)outputModel:.automl.fit[xtrain;ytrain;`normal;`reg;(::)]
    // Testing features
    q)xtest:([]asc 10?0t;10?1f;desc 10?0b;10?1f;asc 10?1f)
    // Make predicts on generated model
    q)outputModel.predict[xtest]
    0.02526887 0.2412723 0.2432483 0.3049373 0.330132 0.3727415 0.4536485..
    ```

## `.automl.getModel`

_Retrieve a previously fit AutoML model and use for predictions_

Syntax: `.automl.getModel[modelDetails]`

Where

-   `modelDetails` a dictionary with information regarding the location of the model and metadata within the outputs directory

returns the predict function (generated using `.automl.utils.generatePredict`) and all relevant metadata for the model.

```q
// Persisted model details
q)modelDetails:`startDate`startTime!(2020.12.15;16:58:46.400)
// Retrieve model
q).automl.getModel[modelDetails]
modelInfo| `modelLib`modelFunc`startDate`startTime`featureExtractionType`prob..
predict  | {[config;features]
  original_print:utils.printing;
  utils.printi..
```

!!! note Predict on new data
    The model retrieved using `.automl.getModel` can be used to make predictions on new data using the same method detailed above for `.automl.fit`.

## `.automl.newConfig`

_Generate a new JSON parameter file for use with .automl.fit_

Syntax: `.automl.newConfig[fileName]`

Where

-   `fileName` is the name to call the newly generated JSON configuration file as a string, symbol or symbolic file handle. This file is stored in 'code/customization/configuration/customConfig'.

returns generic null on successful invocation and saves a copy of the file 'code/customization/configuration/default.json' to the appropriately named file.

```q
// Path where new JSON configuration file will be saved
q)configPath:hsym`$.automl.path,"/code/customization/configuration/customConfig/"
// Check files present in directory at present
q)key configPath
`symbol$()
// Generate new configuration file called "newConfigFile"
q).automl.newConfig[`newConfigFile]
// Check files present in directory - new configuration file has been generated
q)key configPath
,`newConfigFile
```

## `.automl.updateIgnoreWarnings`

_Update print wanring severity level_

Syntax: `.automl.updateIgnoreWarnings[warningLevel]`

Where

-   warningLevel is 0, 1 or 2 long denoting how severely warnings are to be handled, where:
      0. Ignore warnings completely and continue evaluation
      1. Highlight to a user that a warning was being flagged but continue
      2. Exit evaluation of AutoML highlighting to the user why this happened

returns null on success, with `.automl.utils.ignoreWarnings` updated to new level.

```q

```

## `.automl.updateLogging`

_Update logging state_

Syntax: `.automl.updateLogging[]`

Function takes no parameters and returns null on success when the boolean representating `.automl.utils.logging` has been inverted.

```q

```

## `.automl.updatePrinting`

_Update printing state_

Syntax: `.automl.updatePrinting[]`

Function takes no parameters and returns null on success when the boolean representating `.automl.utils.printing` has been inverted.

```q

```