# AMATH 582 Project -- Imbalanced Data, Fraud

## Git workflow:
1. `git commit -m "fix typos"`
2. `git pull --rebase`
3. Resolve any conflicts
4. `git push`

## Work to do:
1. Lee -- Research/implement best evaluation metrics
2. Taylor -- implement 1 or 2 approaches to improve classification of imbalanced data (see next steps below)
4. Add results to paper
5. Transfer notes to paper

## Ideas on next steps:
Focus on performance metrics in the case of imbalanced data.

Try 1 or 2 methods to deal with imbalanced data for comparison. Sounds like synthetic data generation performs better that over/under sampling. I'd prefer to try that or cost function methods. Over/under sampling seem hacky and kinda stupid, although the easiest route.
For LDA do:
 - default
 - train with ADASYN (like SMOTE, see paper in DB)
 - alternate cost function (large penalty for FNs)

The top kernel on Kaggle takes this approach:
> We are not going to perform feature engineering in first instance. The dataset has been downgraded in order to contain 30 features (28 anonymized + time + amount).
> We will then compare what happens when using resampling and when not using it. We will test this approach using a simple logistic regression classifier.
> We will evaluate the models by using some of the performance metrics mentioned above.
> We will repeat the best resampling/not resampling method, by tuning the parameters in the logistic regression classifier.
> We will finally perform classifications model using other classification algorithms.


## Notes:

### Background
Machine learning algorithms have trouble with unbalanced data. Algorithms are typically designed to minimize the error rate or something very similar. This inherently favors the majority class.

Our dataset is a great example of imbalanced data. It is real credit card data and we are attempting to identify fraudulent transactions. Our dataset has only 492 of 284807 data points that are positive. This means that the positive class represents a mere 0.17% of the total. In this light it's actually extremely easy to build a classifier with 99.83% accuracy--just always predict negative!

Accuracy, and therefore error, are not the metrics we care most about in this case. We want to be able to identify nearly all the fraudulent transactions. What we care about most is reducing false negatives (FNs). Unless the data is extremely well separated in, which ours is not, if we optimize for low numbers of FNs we will receive lower accuracy. This is because if the class distributions are overlapping then decreasing FPs by will increase false positives (FPs) by a much larger number. In light of this consideration it's immediately clear that we need a better way to measure the success of our algorithm besides accuracy.

Because our dataset necessarily includes sensitive information the features have been obscured by an SVD decomposition. All features except some arbitrary time and transactions amount have been anonymized in this way, so we will not be performing any feature analysis in this work. We will instead concentrate on methods to work with imbalanced data. We focus specifically only how to evaluate the performance of a classifier in this scenario because, as covered more thoroughly below, common performance metrics do not work well. We will investigate performance metrics by applying methods known to help with imbalanced and comparing this to classification without those methods.



#### Performance metric

It's not just accuracy that serves as a poor metric in this task. Precision and Recall are two common measures. Plotting the former on the y-axis and the latter on the x-axis gives you Precision-Recall (PR) curves, which are often used to evaluate algorithms.
Recall is the fraction of positive entries that you correctly identified as positive: tp/(tp+fn).
Precision is the fraction of entries that were actually positive out of all the ones you guessed were positive: tp/(tp+fp).

In our case we want to optimize for recall over precision. That is, when we are given an input that is actually positive we want to correctly predict as much as possible that it is positive, even if that means we end up with false positives. If we get false positives that lowers our precision, but since false negatives are more costly than false positives here we can accept potentially many more false positives.  

For the reason above precision is not a good metric for this task. For the functionality we want to optimize we will likely see precision increase over a simple majority class classifier, but we can't really be sure of the exact relationship. A better metric would be one that is guaranteed to decrease monotonically as we approach our ideal of perfect recall, regardless of what happens to precision.

A metric that does work well is the Area Under the Precision-Recall Curve (AUPRC)...

#### Methods for imbalanced data
1. Under-sampling -- Removing data from the larger class to balance the class sizes. May lead to a poorer choice of decision line due to losing data at the border of the classes.
2. Oversampling -- Adding extra observations on top of existing minority class observations to balance out the class sizes. May lead to overfitting with some classification models.
3. Synthetic data generation -- Generating artificial data from your existing data to balance classes. Generated data generally stays within the n-dimensional volume that minimally encloses the existing data.
4. Cost functions-- Classification algorithms use cost functions (decision functions) to define their decision boundaries. With imbalanced data you would set the misclassification of the minority class to be much more costly, to encourage the algorithm to classify them correctly more often than the majority class.

Using LDA as our classification algorithm we compared the synthetic data generation and alternate cost function approaches to the default algorithm. For synthetic data generation we chose to use ADASYN (Adaptive Synthetic Sampling Approach for Imbalanced Learning) a generation algorithm similar to the well known SMOTE (Synthetic Minority Over-Sampling Technique) algorithm, which is ADASYN's precursor. The goal of both is to improve the class balance by increasing the number of minority class members. SMOTE places synthetic data between existing data points randomly (linear interpolation), with no preference shown to any specific points. ADASYN does the same thing but places more synthetic data points close to the boundary between classes because those are the original data points that are more difficult to learn.
(Does this favor decision trees or SVMs or something? I imagine that all this would do for an LDA is to move the mean of the minority gaussian close to the boundary... SMOTE might be better here...)

We also tried changing the cost function of our LDA model to discourage FNs. Since LDA works by creating probability distributions the cost comes into play only when making predictions. Each observation that it is trying to predict has a calculated posterior probability. It tries to minimize the "classification cost". Unfortunately if an observation has an extremely small posterior probability then even with a vary large cost for miscalculating that observation it may not change the classification cost by much, meaning there will be little change to the resulting prediction.

### Classification algorithms
Linear discriminant analysis..

In MATLAB:
> For linear discriminant analysis, the model has the same covariance matrix for each class; only the means vary.
> For quadratic discriminant analysis, both means and covariances of each class vary.

 *We may want to just stick to LDA for this project!!*

Binary decision trees are a common and effective choice for classification tasks. The MATLAB command to build a binary classification decision tree model, `fitctree`, is a CART algorithm. At each node in the tree you can a binary decision and you work your way down the tree until you hit a terminal leaf--this is your determination. You can think about this process as splitting up m-dimensional space into blocks, where m is the number of features. You first divide the entire space into two, then one of those subspaces in two, and onwards again and again until you have made a decision based on all the features. Each of the resulting (m+1?) blocks represents a subspace that corresponds to s specific determination: a class.

The question then becomes how to build the tree. What are the split points chosen to create the best classifier? This is accomplished by minimizing a cost function for training points in a given block you are trying to split. The algorithm is greedy, meaning it always chooses the best prediction that each split point; it does not consider error globally. For regression predictive modeling CART (likely) minimizes the squared error. For classification it uses the Gini cost function.




### References and usefulness
https://www3.nd.edu/~dial/publications/dalpozzolo2015calibrating.pdf
https://en.wikipedia.org/wiki/Receiver_operating_characteristic#Area_under_the_curve
http://machinelearningmastery.com/classification-and-regression-trees-for-machine-learning/
