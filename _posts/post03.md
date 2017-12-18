---
layout: post
title: Credit Scoring Walk-through - Part Three
author: Stevan Caric
published: false
status: publish
tags: R RStudio Credit Scoring Model
---

In the previous two posts, we explored the data, applied the weight of evidence transformation to our predictors, and eventually built a credit scoring model with logistic regression. Part Three examines other possible methods that can be used to estimate the credit scoring model and demonstrates a **credit** **risk** **scorecard** development process.

-   [Data Set: Home Equity Loans](#data-set-home-equity-loans)
-   [Decision Trees](#decision-trees)
-   [Ensemble Methods](#ensemble-methods)
-   [Scorecard Development](#scorecard-development)
-   [End of Part Three](#end-of-part-three)

### Data Set: Home Equity Loans

The data set used in this exercise contains the following variables:

-   **BAD**: 1 = applicant defaulted on loan or seriously delinquent; 0 = applicant paid loan
-   **LOAN**: Amount of the loan request
-   **MORTDUE**: Amount due on existing mortgage
-   **VALUE**: Value of current property
-   **REASON**: DebtCon = debt consolidation; HomeImp = home improvement
-   **JOB**: Occupational categories
-   **YOJ**: Years at present job
-   **DEROG**: Number of major derogatory reports
-   **DELINQ**: Number of delinquent credit lines
-   **CLAGE**: Age of oldest credit line in months
-   **NINQ**: Number of recent credit inquiries
-   **CLNO**: Number of credit lines
-   **DEBTINC**: Debt-to-income ratio

### Decision Trees

[Decision trees](https://en.wikipedia.org/wiki/Decision_tree_learning) are commonly used as an alternative to logistic regression. Their main virtue is that they provide very straightforward predictive models, but this usually comes at a price, as a prediction quality deteriorates.

> "The key decisions to build a tree are:
> 1. **Splitting decision**: Which variable to split and at what value
> 2. **Stopping decision**: When to stop adding nodes to the tree
> 3. **Assignment decision**: What class (e.g. good or bad) to assign to a leaf node"
>
> B. Baesens, Analytics in a Big Data World, Wiley, 2014.

Different algorithms can be utilized for a decision tree construction. The most famous are C4.5, CART and CHAID. To estimate a decision tree model in **R** we will use `rpart()` function from the [rpart](https://www.rdocumentation.org/packages/rpart/versions/4.1-11) package. By default, `rpart()` constructs a CART model. However, many tuning parameters are available. In our case, we set the minimum number of observations in any terminal node ( *minbucket*) to 50. This is done to prevent a potential overfitting. The resulting plot (made by `fancyRpartPlot()` from [rattle](https://www.rdocumentation.org/packages/rattle/versions/5.1.0) package and with a little help of good old `text()` function) is given below:

![](post03_files/figure-markdown_github/tree-1.png)

Well, this plot is beautiful looking and simple enough at the same time. The final decision tree model makes all predictions using just three features: debt-to-income ratio (DEBTINC), number of delinquent credit lines (DELINQ) and age of oldest credit line in months (CLAGE). For instance, if an applicant has a debt-to-income ratio above 45% and at least one delinquent credit line, the model flags him as a bad customer. The relative importance of the variables is shown in the next plot.

![](post03_files/figure-markdown_github/tree_varimp-1.png)

Obviously, DEBTINC is by far the strongest predictor in the model. We can also see that among the variables that did not enter the final model, the value of the current property (VALUE), number of major derogatory reports (DEROG) and amount of the loan (LOAN) are the most significant.

Indeed, we got a really simple and logical model, but what about its predictive power? To answer this question, we need to compute the model's *AUC*.

![](post03_files/figure-markdown_github/tree_auc-1.png)

The red dashed line represents the ROC curve for the training sample, while multicolour full line represents the ROC curve for the test sample. Although the AUC on the test set is reasonably high ( **81.2%**), it is considerably less than the AUC for the logistic regression model ( **90.1%**). This is not surprising at all, and, in practice, decision trees are mostly used for variable selection or segmentation.

### Ensemble Methods

The idea behind ensemble methods is to build numerous models and then combine their predictions to make a final prediction. It has been shown that in many cases ensemble methods outperform the simple models, such as logistic regression.

Decision trees are quite often used in ensemble methods. In this section, we will present two extremely popular ensemble methods: **random forest** and **boosting**.

#### Random Forest

> "The steps for building a random forest are:
> 1. Take a bootstrap sample
> 2. Build a decision tree whereby for each node of the tree you randomly choose *m* inputs on which to base the splitting decision
> 3. Split on the best of this subset
> 4. Fully grow each tree without pruning"
>
> B. Baesens, Analytics in a Big Data World, Wiley, 2014.

We are going to estimate our [random forest](https://en.wikipedia.org/wiki/Random_forest) model in **R** using `randomForest()` function from (yes, you've guessed it) [randomForest](https://www.rdocumentation.org/packages/randomForest/versions/4.6-12) package. We set the number of variables randomly chosen as candidates at each split (mtry) to 3 and number of trees to grow (ntree) to 400. The relative importance of the predictors in the model are listed below:

![](post03_files/figure-markdown_github/forest_varimp-1.png)

Once again, the variable DEBTINC is the best predictor. However, this time it is not as dominant as it was in the simple decision tree model. Furthermore, due to the randomness in the splitting process, all variables take a part in the random forest model. But did we improve the predictive performance?

![](post03_files/figure-markdown_github/forest_auc-1.png)

Yes, we did! We improved it a lot. We got the AUC on the test set of fantastic **95.1%**! We now have a great predictive model. The only problem is that it is entirely a **black box**. We do not actually know how any particular variable is related to the model's outcome. This could be irrelevant if you try, for instance, to detect a credit card fraud, but, in the credit scoring field, this is a highly undesirable characteristic. Thus, the logistic regression remains the go-to method.

#### Boosting

> "Boosting works by estimating multiple models using a weighted sample of the data. Starting from uniform weights, boosting will iteratively reweight the data according to the classification error, whereby misclassified cases get higher weights." (B. Baesens, Analytics in a Big Data World, Wiley, 2014)

To perform boosting in **R** you can use `gbm()` from the [gbm](https://cran.r-project.org/web/packages/gbm/index.html) package. Several parameters can affect the predictive performance of the boosted models. Thus, we had experimented a bit with parameters' values before we chose our final model (`train()` from the [caret](https://topepo.github.io/caret/model-training-and-tuning.html) package makes this task effortless). In the end, we set the total number of trees to fit ( *n.trees*) to 500, shrinkage parameter or learning rate ( *shrinkage*) to 0.2 and number of cross-validation folds to perform ( *cv.folds*) to 5.

![](post03_files/figure-markdown_github/boost_varimp-1.png)

The variable importance plot is telling us a similar story as in the random forest case.

![](post03_files/figure-markdown_github/boost_auc-1.png)

The AUC on the test set is **91.2%**. This value is between the AUC for the logistic regression model and the AUC for the random forest model.

To conclude, we can certainly get somewhat better predictions by using ensemble methods, but if we are interested in building a credit risk scorecard, then we need to stick with our logistic regression model from Part Two.

### Scorecard Development

A **credit risk scorecard** is just another way of presenting a linear model. The idea is to transform the estimated model's coefficients into (more comprehensible) points.

We start with our logistic regression model:

![](eq_logit.png)

where *odds = P(bad) / P(good)*, *WOE<sub>DEBTINC</sub>* is the weight of evidence coded debt-to-income ratio, and so on.

We then express a score as a linear function of the *ln(odds)*:

![](eq_score1.png)

By combining last two equations we get:

![](eq_score2.png)

where *N* is the number of predictors.

> We can easily compute *offset* and *factor*. Let's say we want a score of 500 for odds of 1:1, and a score of 520 for odds of 1:2. This gives us the next two equations:
> 500 = *offset* + *factor* \* *ln( 1 / 1)*  
> 520 = *offset* + *factor* \* *ln( 1 / 2)*  
> We then solve this system of equations and get that *offset* = 500 and *factor* = -28.85.

In **R** the `scorecard()` function carries out all this calculations and builds a scorecard. The results for some of the variables are presented in the following tables. The points assigned to each bin are displayed in the last column.

<table style="text-align:center">
<caption>
<strong>Debt-to-income ratio</strong>
</caption>
<tr>
<td colspan="10" style="border-bottom: 1px solid black">
</td>
</tr>
<tr>
<td style="text-align:left">
bin
</td>
<td>
count
</td>
<td>
count\_distr
</td>
<td>
good
</td>
<td>
bad
</td>
<td>
badprob
</td>
<td>
woe
</td>
<td>
bin\_iv
</td>
<td>
total\_iv
</td>
<td>
points
</td>
</tr>
<tr>
<td colspan="10" style="border-bottom: 1px solid black">
</td>
</tr>
<tr>
<td style="text-align:left">
\[-Inf,30)
</td>
<td>
1,007
</td>
<td>
0.23
</td>
<td>
958
</td>
<td>
49
</td>
<td>
0.05
</td>
<td>
-1.58
</td>
<td>
0.34
</td>
<td>
2.20
</td>
<td>
43
</td>
</tr>
<tr>
<td style="text-align:left">
\[30,45)
</td>
<td>
2,445
</td>
<td>
0.55
</td>
<td>
2,262
</td>
<td>
183
</td>
<td>
0.07
</td>
<td>
-1.13
</td>
<td>
0.48
</td>
<td>
2.20
</td>
<td>
30
</td>
</tr>
<tr>
<td style="text-align:left">
\[45, Inf)
</td>
<td>
60
</td>
<td>
0.01
</td>
<td>
3
</td>
<td>
57
</td>
<td>
0.95
</td>
<td>
4.33
</td>
<td>
0.27
</td>
<td>
2.20
</td>
<td>
-116
</td>
</tr>
<tr>
<td style="text-align:left">
missing
</td>
<td>
959
</td>
<td>
0.21
</td>
<td>
356
</td>
<td>
603
</td>
<td>
0.63
</td>
<td>
1.92
</td>
<td>
1.10
</td>
<td>
2.20
</td>
<td>
-51
</td>
</tr>
<tr>
<td colspan="10" style="border-bottom: 1px solid black">
</td>
</tr>
</table>
<table style="text-align:center">
<caption>
<strong>Number of delinquent credit lines</strong>
</caption>
<tr>
<td colspan="10" style="border-bottom: 1px solid black">
</td>
</tr>
<tr>
<td style="text-align:left">
bin
</td>
<td>
count
</td>
<td>
count\_distr
</td>
<td>
good
</td>
<td>
bad
</td>
<td>
badprob
</td>
<td>
woe
</td>
<td>
bin\_iv
</td>
<td>
total\_iv
</td>
<td>
points
</td>
</tr>
<tr>
<td colspan="10" style="border-bottom: 1px solid black">
</td>
</tr>
<tr>
<td style="text-align:left">
\[-Inf,1)
</td>
<td>
3,122
</td>
<td>
0.70
</td>
<td>
2,672
</td>
<td>
450
</td>
<td>
0.14
</td>
<td>
-0.39
</td>
<td>
0.09
</td>
<td>
0.68
</td>
<td>
11
</td>
</tr>
<tr>
<td style="text-align:left">
\[1,2)
</td>
<td>
498
</td>
<td>
0.11
</td>
<td>
333
</td>
<td>
165
</td>
<td>
0.33
</td>
<td>
0.69
</td>
<td>
0.06
</td>
<td>
0.68
</td>
<td>
-18
</td>
</tr>
<tr>
<td style="text-align:left">
\[2,3)
</td>
<td>
184
</td>
<td>
0.04
</td>
<td>
105
</td>
<td>
79
</td>
<td>
0.43
</td>
<td>
1.10
</td>
<td>
0.07
</td>
<td>
0.68
</td>
<td>
-30
</td>
</tr>
<tr>
<td style="text-align:left">
\[3,5)
</td>
<td>
165
</td>
<td>
0.04
</td>
<td>
73
</td>
<td>
92
</td>
<td>
0.56
</td>
<td>
1.62
</td>
<td>
0.13
</td>
<td>
0.68
</td>
<td>
-43
</td>
</tr>
<tr>
<td style="text-align:left">
\[5, Inf)
</td>
<td>
61
</td>
<td>
0.01
</td>
<td>
3
</td>
<td>
58
</td>
<td>
0.95
</td>
<td>
4.35
</td>
<td>
0.28
</td>
<td>
0.68
</td>
<td>
-117
</td>
</tr>
<tr>
<td style="text-align:left">
missing
</td>
<td>
441
</td>
<td>
0.10
</td>
<td>
393
</td>
<td>
48
</td>
<td>
0.11
</td>
<td>
-0.71
</td>
<td>
0.04
</td>
<td>
0.68
</td>
<td>
19
</td>
</tr>
<tr>
<td colspan="10" style="border-bottom: 1px solid black">
</td>
</tr>
</table>
<table style="text-align:center">
<caption>
<strong>Age of oldest credit line in months</strong>
</caption>
<tr>
<td colspan="10" style="border-bottom: 1px solid black">
</td>
</tr>
<tr>
<td style="text-align:left">
bin
</td>
<td>
count
</td>
<td>
count\_distr
</td>
<td>
good
</td>
<td>
bad
</td>
<td>
badprob
</td>
<td>
woe
</td>
<td>
bin\_iv
</td>
<td>
total\_iv
</td>
<td>
points
</td>
</tr>
<tr>
<td colspan="10" style="border-bottom: 1px solid black">
</td>
</tr>
<tr>
<td style="text-align:left">
\[-Inf,60)
</td>
<td>
147
</td>
<td>
0.03
</td>
<td>
84
</td>
<td>
63
</td>
<td>
0.43
</td>
<td>
1.10
</td>
<td>
0.05
</td>
<td>
0.23
</td>
<td>
-26
</td>
</tr>
<tr>
<td style="text-align:left">
\[60,120)
</td>
<td>
1,051
</td>
<td>
0.24
</td>
<td>
769
</td>
<td>
282
</td>
<td>
0.27
</td>
<td>
0.39
</td>
<td>
0.04
</td>
<td>
0.23
</td>
<td>
-9
</td>
</tr>
<tr>
<td style="text-align:left">
\[120,180)
</td>
<td>
1,117
</td>
<td>
0.25
</td>
<td>
864
</td>
<td>
253
</td>
<td>
0.23
</td>
<td>
0.16
</td>
<td>
0.01
</td>
<td>
0.23
</td>
<td>
-4
</td>
</tr>
<tr>
<td style="text-align:left">
\[180,240)
</td>
<td>
1,000
</td>
<td>
0.22
</td>
<td>
851
</td>
<td>
149
</td>
<td>
0.15
</td>
<td>
-0.35
</td>
<td>
0.02
</td>
<td>
0.23
</td>
<td>
8
</td>
</tr>
<tr>
<td style="text-align:left">
\[240, Inf)
</td>
<td>
932
</td>
<td>
0.21
</td>
<td>
838
</td>
<td>
94
</td>
<td>
0.10
</td>
<td>
-0.80
</td>
<td>
0.10
</td>
<td>
0.23
</td>
<td>
19
</td>
</tr>
<tr>
<td style="text-align:left">
missing
</td>
<td>
224
</td>
<td>
0.05
</td>
<td>
173
</td>
<td>
51
</td>
<td>
0.23
</td>
<td>
0.17
</td>
<td>
0.001
</td>
<td>
0.23
</td>
<td>
-4
</td>
</tr>
<tr>
<td colspan="10" style="border-bottom: 1px solid black">
</td>
</tr>
</table>
We can now use this scorecard to score the applicants in our data set. This can be achieved with the function `scorecard_ply()`.

<table style="text-align:center">
<caption>
<strong>Home Equity Loans data set</strong>
</caption>
<tr>
<td colspan="12" style="border-bottom: 1px solid black">
</td>
</tr>
<tr>
<td style="text-align:left">
base\_points
</td>
<td>
YOJ\_points
</td>
<td>
CLNO\_points
</td>
<td>
JOB\_points
</td>
<td>
NINQ\_points
</td>
<td>
LOAN\_points
</td>
<td>
CLAGE\_points
</td>
<td>
VALUE\_points
</td>
<td>
DEROG\_points
</td>
<td>
DELINQ\_points
</td>
<td>
DEBTINC\_points
</td>
<td>
score
</td>
</tr>
<tr>
<td colspan="12" style="border-bottom: 1px solid black">
</td>
</tr>
<tr>
<td style="text-align:left">
520
</td>
<td>
1
</td>
<td>
-12
</td>
<td>
-6
</td>
<td>
0
</td>
<td>
-27
</td>
<td>
-9
</td>
<td>
-14
</td>
<td>
4
</td>
<td>
11
</td>
<td>
-51
</td>
<td>
417
</td>
</tr>
<tr>
<td style="text-align:left">
520
</td>
<td>
1
</td>
<td>
5
</td>
<td>
-6
</td>
<td>
2
</td>
<td>
-27
</td>
<td>
-4
</td>
<td>
3
</td>
<td>
4
</td>
<td>
-30
</td>
<td>
-51
</td>
<td>
417
</td>
</tr>
<tr>
<td style="text-align:left">
520
</td>
<td>
-6
</td>
<td>
5
</td>
<td>
-6
</td>
<td>
0
</td>
<td>
-27
</td>
<td>
-4
</td>
<td>
-32
</td>
<td>
4
</td>
<td>
11
</td>
<td>
-51
</td>
<td>
414
</td>
</tr>
<tr>
<td style="text-align:left">
520
</td>
<td>
13
</td>
<td>
0
</td>
<td>
33
</td>
<td>
5
</td>
<td>
-27
</td>
<td>
-4
</td>
<td>
-107
</td>
<td>
15
</td>
<td>
19
</td>
<td>
-51
</td>
<td>
416
</td>
</tr>
<tr>
<td style="text-align:left">
520
</td>
<td>
-6
</td>
<td>
5
</td>
<td>
15
</td>
<td>
2
</td>
<td>
-27
</td>
<td>
-9
</td>
<td>
8
</td>
<td>
4
</td>
<td>
11
</td>
<td>
-51
</td>
<td>
472
</td>
</tr>
<tr>
<td style="text-align:left">
520
</td>
<td>
1
</td>
<td>
5
</td>
<td>
-6
</td>
<td>
0
</td>
<td>
-27
</td>
<td>
-9
</td>
<td>
3
</td>
<td>
-55
</td>
<td>
-30
</td>
<td>
-51
</td>
<td>
351
</td>
</tr>
<tr>
<td colspan="12" style="border-bottom: 1px solid black">
</td>
</tr>
</table>
Next, we can plot and compare the score distributions for the training and test set.

![](post03_files/figure-markdown_github/card_plot-1.png)![](post03_files/figure-markdown_github/card_plot-2.png)

Clearly, the score distributions are rather close, so we can conclude that our scorecard makes reasonable predictions. This can also be confirmed by calculating a population stability index ( *PSI*). The PSI value of 0.01 indicates that actual distribution is almost the same as expected distribution.

### End of Part Three

Well, this has been a long and wonderful journey, but as the old proverb says "All good things must come to an end". The **credit scoring walk-through** is completed. It is finished and done! It is *kaputt*! It is no more! It has ceased to be! It has expired and gone to meet its maker! It is an ex-walk-through!
