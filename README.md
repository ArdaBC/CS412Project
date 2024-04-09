# CS Project

**Group members:** Arda Berk Çetin (31077), Ece Güler (29107)

# Project Goal:

The goal of our project is to find correlations between student interactions with ChatGPT and their assignment grades.

# Approaches:

By analyzing dialogue patterns, we aim to uncover insights that correlate with student grades. To achieve this, we used techniques involving data preprocessing, feature engineering, sentiment analysis, and machine learning. We applied a multitude of approaches and made sometimes quite minor, and sometimes, major changes to the document. Here's a detailed overview of our approaches:

One of the minor changes we did is that we removed markdown from the question prompts. Since markdown is not copied, this might facilitate prompt matching with questions. Another minor change we did is that we added some more keywords to keywords2search. Description for why we added these are as follows:

"code here", "pts": This is a part of the question that is not really needed to be copied since it gives no additional info to ChatGPT. We thought this might indicate sloppiness on user's end and hence, a lower grade.

"instead", "try", "why", "hint": This might signal the user is dissatisfied with the result and confused about ChatGPT's response. 

"Traceback", "most recent call last": This indicates that the user encountered and error and directly copy pasted the error message into ChatGPT.

"google", "http": This might indicate the user googled something or posted a link after it was dissatisfied with the answer.

"driver": We added this to see if the user asked about driver features specifically mentioned in the homework document.

These keyword analyses are critical for understanding the students' interactions and their level of engagement with ChatGPT.

In addition to these keyword-based insights, we analyzed the emotional context of the user's messages by applying sentiment analysis on every prompt of the user to possibly find a relation with grades. This seemed logical to us since the user might react negatively if ChatGPT is consistently returning useless answers. Conversely, if ChatGPT is returning useful answers, the user might react positively. The model we used also had neutral sentiment which might still be useful in comparison to its balance with negative and positive sentiments. We downloaded a pretrained model from Huggingface, namely the famous RoBERTa. Then, we calculated the average negative, neutral and positive sentiment for each user using RoBERTa. 

Another important approach we took is that we scraped the date on every document. This was a challenging task because the date information is generated by javascript and therefore, it cannot be scraped with BeautifulSoup. This forced us to use Selenium for this task. Furthermore, since the latest Chromedriver is for Chrome version 114 and we were using Chrome version 120, we had to delete and redownload Chrome which made the task more challenging. Then, when we finally manage to scrape the date information, it was added as a feature after representing it as the difference from the latest date. This aspect of data collection was key in adding a time-related perspective to our analysis.

Then we applied another minor change. We multiplied the probabilities from quesion-prompt matching with the points of the questions to take their different weights. Afterall, not every question was of equal importance. This method made sure our model understood the importance of different types of interactions, leading to more accurate predictions.

Finally, the last important approach we took is that we trained two neural networks. The first neural network does binary classification. Its goal is to correctly classify grades that are in the bottom 25th percentile of grades. This is done separately due to the high right-skewed nature of our data. Then, the predictions of this specialized neural network is fed to the second one in combination with the all other features we were already using. The second neural network than outputs a grade prediction between 0 and 100. This is achieved by having an ultimate layer that multiplies the outcome of the penultimate sigmoid layer by 100. We are in the hopes that our two-model strategy will handle the uneven nature of our data, enabling our model to make accurate predictions for both low and high grades. Note that we made sure to not accidentally train our first neural network (classifier) with grade information and our second neural network (grade predictor) with the actual low grade information. 

To make sure the train/test split was done accurately, we made sure the ratio of low grades in both the train and test set was close to each other by repeatedly splitting until we got a good result. We would have used something like StratifiedShuffleSplit or simply add "stratify=y" to train_test_split. However, because we had so little data, we didn't have enough different y values to make this work. Nevertheless, we managed to split our data evenly by comparing the aforementioned ratios. Achieving a balanced division between training and testing data was key for accurately training and assessing our model.

Lastly, alongside our neural network models, we also experimented with a random forest classifier. We used GridSearchCV to find the best hyperparameters inorder to improve the predictive power of our model.

# Results:

Long story short, we didn't manage to find a relation.

Our binary classifier has the following confusion matrix:

![Confusion Matrix](https://github.com/ArdaBC/FearlessPianoPancake/blob/main/confusion_matrix.png?raw=true)

As you can see, this is not that impressive, but still, not horrible. It has an accuracy of 72% on the test set. Now, we feed the predictions from this outlier classifier to the neural network and we get the following result:

![NN Results](https://github.com/ArdaBC/FearlessPianoPancake/blob/main/nnresults.png?raw=true)

This seems okay, however, when you take a close look at the predictions of our neural network, the allure is lost:

Index | Actual | Predicted
--- | --- |---
0 | 97.0 | 99.966873 
1 | 96.0 | 99.985054
2 | 93.0 | 99.999931
3 | 90.0 | 97.460190
4 | 98.0 | 98.051086
5 | 99.0 | 99.998169
6 | 100.0 | 100.000000
7 | 100.0 | 99.999817
8 | 100.0 | 99.999947
9 | 88.0 | 100.000000
10 | 92.0 | 100.000000
11 | 98.0 | 99.999695
12 | 99.0 | 100.000000
13 | 89.0 | 99.999878
14 | 94.0 | 100.000000
15 | 96.0 | 99.999985
16 | 100.0 | 100.000000
17 | 97.0 | 100.000000
18 | 99.0 | 100.000000
19 | 97.0 | 99.995987
20 | 84.0 | 100.000000
21 | 90.0 | 99.989761
22 | 15.0 | 100.000000
23 | 78.0 | 99.993286
24 | 94.0 | 99.999969

Our model has low error because it predicts high grades for everyone, and it gets away with this because the data is extremely right skewed. Furthermore, as can be seen on the 22nd line, our model predicted that someone who got 15 got 100. This shows that our approach of using two neural networks was not useful.

If we look at the results of our random forest classifier, we also see nothing useful unfortunately. With a max depth of 5, minimum samples per leaf 2, minimum samples per split 5 and 10 estimators, we got the following results:

MSE Train: 45.45360824742268

R2 Train: 0.628097499043445

MSE Test: 339.12

R2 Test: -0.24650074101954877

This tells us our model did a poor job.

Lastly, let's take a look at the decision tree results:

MSE Train: 7.168113225486601

MSE TEST: 389.7955068818447

R2 Train: 0.941350327543042

R2 TEST: -0.4327683067183006

This is clearly a case of overfitting. R2 and MSE on training data is good, but we are doing horribly on test data. Therefore, this is not useful either.

# Conclusion:

The project highlighted various challenges, particularly in the feature extraction process and model building phase. Maintaining model generalization without overfitting was a crucial aspect. Our project represented a comprehensive effort to predict student performance based on interactions with ChatGPT. Despite facing difficulties and getting mixed results, we learned important things about machine learning itself and working on projects.
