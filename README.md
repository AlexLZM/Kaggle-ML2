# Kaggle Competetion - Cancer prediction based on microRNA
<a href="https://www.kaggle.com/c/ml2-usf-2021/">https://www.kaggle.com/c/ml2-usf-2021/</a>
## ABSTRACT
Effective early non-invasive screening of breast cancer is crucial to strategize treatment of breast tumors. Circulating microRNAs (miRNAs) have been recognized as potential biomarkers that can lead to clinical applications.

The goal of this project is to construct models to predict the target value for each observation for different kinds of cancer, using the given profile of microRNA expression extracted from the serum of the subject. The project was wrapped in the form of Kaggle competition, where we were given pre-processed microRNA expression training data with target labels and we predicted the accuracy for the test dataset.

To find the best model that gives the highest accuracy, we developed and tuned hyperparameters for different models including XGBoost and LightGBM. For hyperparameter tuning we combined the data from different problem ids and used cross validation. Once we had the best hyper parameters, we created LightGBM models with Dart mode, for each of the dataset. So at the end we had 21 different LightGBM models for 21 different datasets and we used these to make the final predictions on the test data.

## TEAM DETAILS
Team Name : Gradient Ascent

Team Members : Hashneet Kaur / Zhimin (Alex) Lyu Kaggle ID’s : hashneetkaur / finslcx

## DATASET DESCRIPTION
The training data consists of data from 21 different datasets, where each dataset corresponds to a different kind of cancer. We can segregate the dataset into 21 different subsets based on the column ‘problem_id’ where observations with the same problem_id belong to the same subset. The size of each subset varies from 200 - 550 observations. So in total the combined dataset has 8302 observations. There are 978 continuous miRNA expression features across all subsets. Each subset’s target variable has 4 - 5 unique categorical values meaning they are multi-class classification problems.

The miRNA expression features are pre-processed by the instructor with values ranging from 0 to 1. There are no missing values. Most of the observations have target value as 0 or 1.

## EDA
Next, we will look at how the data is distributed through value count plots for the target variable and problem_id.
1. Problem ID: Each problem_id is indicating a type of cancer.
The value count plot for ‘problem_id’ shows that the data has been taken from 21 different sources. The data source corresponding to problem_id 13 and 16 contributes for the maximum data and source with problem_id 10 has the least data.
![Clustering of ring shape data](./images/WX20210615-183407@2x.png?raw=true)

2. Target : Another important thing to look at is how are the target classes distributed for each problem_id. The target variable is a categorical variable with values 0,1,2,3, and 4.
![Clustering of ring shape data](./images/WX20210615-183426@2x.png?raw=true)
The class distribution for the target column shows that we have maximum data with label 0 and least data with label as 4. There are 4124 observations with target label 0 and 3175 observations with target label as 1, whereas we have only 19 observations with label 4. 11 problem ids have only 2 classes and label 4 only exists in problem 17.
## MACHINE LEARNING MODELS AND EXPERIMENTAL RESULTS
We mainly used LightGBM for this classification task. For 21 subsets, we trained 21 LightGBM models. As a gradient boosting machine, LightGBM has 2 major virtues over XGBoost from our experience:
1. Faster training: LightGBM took almost 1/10th the time to train with similar hyperparameter compared to XGBoost. It also has the option to utilize GPU. This makes the training-tuning iteration really short and thus we can tune its hyperparameters more finely, achieving higher cross validation performance. Furthemore, Dart mode is viable here as the shorter training time permits.
  
2. More choices of hyperparameters: Beyond hyperparameters of XGBoost, LightGBM has more choices in each hyperparameters category and more categories available than XGBoost, with possibilities to achieve better cross validation performance.
### Hyperparameter tuning for LightGBM:
For each subset, we used Optuna as a bayesian optimization tool combined with manual tuning to obtain best hyperparameters in terms of cross validation performance. We combined group-wise hyperparameter tuning, manual adjustment and global hyperparameter tuning in the process.
5-fold cross validation is used in all model tuning, with CV validation log_loss as the main tuning metric.
As we have a high number of features relative to sample size, overfitting alleviation hyperparameters were focused in the tuning process. To implement group-wise hyperparameter tuning, we categorized LightGBM’s main hyperparameters into 4 groups:
1. Learning rate and number of boost rounds: We first set eta in a relatively high value with acceptable test loss, to achieve a relatively quick training-tuning iteration. Then early_stopping_round is set to 10 to avoid extra rounds after minimum validation loss is achieved.
2. Tree related hyper parameters: max_depth , min_hessian and num_leaves. These 3 are tuned together till optimal validation loss is achieved. Reducing the max depth/num_leaves and increasing the min_hessian can reduce overfitting.
3. Random sampling parameters: subsample and colsample_bytree. Reducing these parameters to less than 1 can also have regularization effects.
4. L1/L2 Regularization parameters: lastly alpha and lambda are tuned to achieve best validation performance.
After group-wise tuning (mainly by optuna), we feed a global tuning task, combining select ranges of all hyperparameters we mentioned above, to Optuna for an overnight run. From this run we can get a relatively optimal set of hyperparameters.
After all steps above, we re-tune eta to smaller values to avoid boosting overshoot the local loss minima and achieve best cross validation performance for LightGBM models. Finally, after an optimal set of hyperparameters is obtained in normal ‘gbdt’ mode, we change the mode to Dart directly and tune the drop_rate and num_boost_round afterwards (as early_stopping_rounds does not work in Dart mode).
 
### Hyperparemeter tuning results for LightGBM
This sequence of approaches helped us to achieve a good score in the public leaderboard. We chose the hyperparameter with public Leaderboard score 0.75122 accuracy although it is not the highest one, but it has better CV performance:
## Final results
The final private leaderboard score is 0.71755. It is lower than the public one, probably because we overfitted to the small size public test data (800 samples).
