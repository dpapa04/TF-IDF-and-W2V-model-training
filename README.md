Response Clarity Classification in Political QA Pairs

1. Introduction
The task is focused on creating an agent who has the ability to classify political Question-Answer pairs into three distinct clarity labels (Clear Reply, Ambivalent, Clear Non-Reply) using two representations: 

TF-IDF
Word2Vec

,both paired with Logistic Regression. 

2. Eliminating other factors
Before starting with the data analysis we first need to pick a random seed so that the model’s training does not get affected by random factors that lie beyond our control. This ensures that the results we get are a true representation of the abilities of the model.

3. Data Analysis
An initial analysis of the training dataset was performed to understand label distribution, text length characteristics, and vocabulary patterns across classes.

3.1 Label Distribution
The dataset shows a class imbalance. The Ambivalent class constitutes the majority of samples, while Clear Non-Reply is at a significantly lower percentage with approximately 10% of total samples. This imbalance provoked the idea to use a modified weight instead of the default “balanced” in the Logistic Regression classifier.

3.2 Answer Length Analysis
A key finding is that answer length varies meaningfully across clarity labels. Ambivalent answers are the longest on average, consistent with the belief that politically evasive speakers tend to provide lengthy responses that touch on the topic without directly addressing the question. Clear Non-Reply answers are by far the shortest, suggesting that speakers who outright avoid a question tend to do so briefly. Clear Reply answers fall between the two extremes.

This inspired the making and use of a scaled answer length feature that when implemented affected the model negatively so it was removed from both TF-IDF and Word2Vec representations.

Text Preprocessing

In order for the data to be handled with more easy and flexibility, the questions and answers were combined to one. Both the train and test set were prepared in this way, so that the clarity assessment could be made.

5. Experimental Results
5.1 Hyperparameter Tuning
Logistic Regression regularisation strength C was tuned trying to input values [0.01, 0.1, 0.5, 1, 5, 10], optimizing for macro F1. The results did show that the optimal value is the default value C=1, even though it is showing characteristics of overfitting. Values that reduced the overfitting (e.g. 5) resulted in the the prediction model performing noticeably worse in the validation tests.

5.2 Learning Curves
TF-IDF Model
10% training data — Train F1: 0.96 | Val F1: 0.38
20% training data — Train F1: 0.94 | Val F1: 0.43
30% training data — Train F1: 0.92 | Val F1: 0.43
40% training data — Train F1: 0.91 | Val F1: 0.45
50% training data — Train F1: 0.89 | Val F1: 0.47
60% training data — Train F1: 0.88 | Val F1: 0.49
70% training data — Train F1: 0.88 | Val F1: 0.52
80% training data — Train F1: 0.87 | Val F1: 0.55
90% training data — Train F1: 0.86 | Val F1: 0.55
100% training data — Train F1: 0.85 | Val F1: 0.57

The TF-IDF learning curves reveal significant overfitting. Training F1 starts at 0.96 with 10% of data and decreases steadily to 0.85 at 100%, while validation F1 steadily rises until it reaches 0.57. To boost the perfomance the max features of the Vectorizer were set at 100000 to make sure we save all of them and the range set to 5 as in these type of speeches by politicians words that are close together give more information. Also the custom weights were utilized here in order to compensate for the difference in the number of each clarity label in the data. These weights were decided first by observing the split of the data and secondly by testing them with manual inputs to document the changes that occur. 


Word2Vec Model
10% training data — Train F1: 0.64 | Val F1: 0.44
20% training data — Train F1: 0.56 | Val F1: 0.40
30% training data — Train F1: 0.54 | Val F1: 0.44
40% training data — Train F1: 0.54 | Val F1: 0.43
50% training data — Train F1: 0.55 | Val F1: 0.44
60% training data — Train F1: 0.53 | Val F1: 0.44
70% training data — Train F1: 0.53 | Val F1: 0.45
80% training data — Train F1: 0.52 | Val F1: 0.45
90% training data — Train F1: 0.51 | Val F1: 0.45
100% training data — Train F1: 0.52 | Val F1: 0.48


In the case of the Word2Vec model the learning curves appear to be significantly worse than the ones in the TF-IDF model. Training F1 starts at 0.64 with 10% of data and decreases steadily to 0.52 at 100%, while validation F1 pivots around 0.40-0.48. Despite the model showing no sign of overfitting compared to the earlier one the clear drop in performance makes it unsuitable for selection. Custom weights were utilized although they were ineffective since the worsened the performance further.

5.3 Final Validation Performance
=== TF-IDF ===
                 precision    recall  f1-score   support

    Ambivalent       0.70      0.65      0.67       306
    Clear Non-Reply       0.48      0.63      0.54        54
    Clear Reply       0.48      0.49      0.48       158

       accuracy                           0.60       518
      macro avg       0.55      0.59      0.57       518
    weighted avg       0.61      0.60      0.60       518

=== W2V ===
                 precision    recall  f1-score   support

     Ambivalent       0.68      0.67      0.67       306
    Clear Non-Reply       0.32      0.56      0.41        54
    Clear Reply       0.43      0.33      0.37       158

       accuracy                           0.55       518
      macro avg       0.47      0.52      0.48       518
     weighted avg       0.56      0.55      0.55       518

TF-IDF with Logistic Regression outperformed the Word2Vec-based approach across all evaluation metrics, achieving a macro F1 of 0.57 compared to 0.48. Despite exhibiting overfitting, TF-IDF is the clearly the better model to use for the agent. The performance loss is most likely due to the avaraging function that takes place in the Word2Vec model as small words that completely change the meaning of a sentence sometimes get lost.

Clear Non-Reply is the most challenging class with only 54 validation samples (approximately 10% of the validation set). The model has seen very few examples of this class during training, making it difficult to learn reliable decision boundaries. Despite applying class weighting and experimenting with custom class weights, performance improvements were minimal.

6.Summary Of Experiments That Did Not Improve Performance
The following approaches were explored but did not bring significant improvements:

C=[0.01, 0.1, 0.5, 1, 5, 10] regularisation was tested but even after the extensive research the default option of C=1 was choosen as the superior one based on performance.
Custom class weights {Ambivalent: 1, Clear Non-Reply: 2, Clear Reply: 2.5}: Slightly improved Clear Non-Reply recall but in a non significant amount.
Word count for every class of answers, that when implemented either did not changes or in some cases made the performance worse.
