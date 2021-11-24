Supervised Text Classification II
================
Philipp Masur
2021-11

-   [Introduction](#introduction)
-   [Preprocessing](#preprocessing)
    -   [Getting some Twitter data](#getting-some-twitter-data)
    -   [Creating train and test data
        sets](#creating-train-and-test-data-sets)
    -   [Text preprocessing and creating the
        DTM](#text-preprocessing-and-creating-the-dtm)
-   [Machine Learning Process](#machine-learning-process)
    -   [Training different algorithm](#training-different-algorithm)
    -   [Testing the algorithms on the held-out
        data](#testing-the-algorithms-on-the-held-out-data)
-   [Using the classifier on a new data
    set](#using-the-classifier-on-a-new-data-set)
    -   [Getting some new twitter data](#getting-some-new-twitter-data)
    -   [Using the trained algorithm to predict whether these tweets
        contain “aggressive”
        language](#using-the-trained-algorithm-to-predict-whether-these-tweets-contain-aggressive-language)
    -   [Some simple analyses](#some-simple-analyses)
-   [Where to go from here?](#where-to-go-from-here)

# Introduction

In this tutorial, we are going to further practice using different
supervised learning approaches to classify/code text.

A small reminder: In supervised text classification, we train a
statistical model on the *features* of our data (e.g. the word
frequencies) to predict the *class* of our texts (e.g. the sentiment,
topic,…). We thus need annotated data that we can use to train and test
the algorithm.

In the following example, we will use and existing, annotated data set
to train an algorithm and check out whether this machine learning
algorithm can then be used to classify a new data set.

We will again use functions from the `quanteda.textmodels` package as
well as some functions from the `caret` package.

``` r
library(tidyverse)
library(quanteda)
library(quanteda.textplots)
library(quanteda.textmodels)
library(caret)
```

# Preprocessing

## Getting some Twitter data

In this tutorial, our goal is to use an existing, annotated data sets of
Twitter tweets to train an algorithm to detect “uncivil” language (or
even cybertrolls) in tweets. This [Data
Set](https://www.kaggle.com/dataturks/dataset-for-detection-of-cybertrolls)
includes about 20,000 tweets that are labeled as being aggressive (1) or
not aggressive (0). The data set comes as a json-file, which I uploaded
to our Canvas page.

Let’s download the data, use the function `stream_in()` from the
`jsonlite` package and do some basic data wrangling.

``` r
# Importing the json data base
tweets <- jsonlite::stream_in(file("trolls_data.json"))

# Some data wrangling to get a tidy data set
tweets <- tweets %>%
  as_tibble %>%
  mutate(label = unlist(annotation$label)) %>%   ## The label column was stored as a list
  select(label, text = content) %>%
  mutate(text = trimws(text))                    ## Let's remove some white space
head(tweets)
```

Let’s try to better understand our data and check how many tweets are
labeled as aggressive vs. not aggressive.

``` r
tweets %>%
  group_by(label) %>%
  count %>%                      ## Absolute frequencies
  mutate(prop = n/nrow(tweets))  ## Relative frequencies
```

To better understand how the data was labeled, let’s look at the most
used words in the tweets that were coded as “aggressive”. We can do so
by first filtering our data set to contain only aggressive tweets. We
then go through some text pre-processing and create a document-term
matrix. We can then create a simply word cloud.

``` r
tweets %>%
  mutate(text = str_remove_all(text, "#")) %>%
  filter(label == 1) %>%
  corpus %>%
  tokens(remove_punct = T, remove_symbols = T) %>%
  tokens_remove(stopwords("en")) %>%
  dfm %>%
  textplot_wordcloud(max_words = 75, color = c("orange", "red"))
```

Not very nice, but it seems that the tweets have been labeled in a
decent way. Bear in mind, we use a data set that was simply published on
some website. We do not now how well these tweets were actually coded.
The quality of our following analyses, however, rest on the assumption
that this manual coding was done well and has high validity!

## Creating train and test data sets

Similar to our last practical session, we need to split the model into
training and text data. We again do this with regular R and tidyverse
function. We sample from the row indices and use `slice` to select the
appropriate rows:

``` r
# To ensure replicability
set.seed(42)

# Sample 
trainset <- sample(nrow(tweets), size=round(nrow(tweets) * 0.8))
tweets_train <- tweets %>% slice(trainset)
tweets_test <- tweets %>% slice(-trainset)
```

## Text preprocessing and creating the DTM

The next step again consists of engaging in various text pre-processing
steps. We use `str_remove_all()` to remove the `#` from the tweets. This
is quite useful as some words are otherwise not identified as being
similar (e.g., ‘\#hate’ and ‘hate’). Because supervised machine learning
algorithm otherwise do well even if we do not remove any further noise,
we only tokenize and create the dtm.

``` r
dfm_train <- tweets_train %>% 
  mutate(text = str_remove_all(text, "#")) %>%
  corpus %>% 
  tokens %>%
  dfm
dfm_train
```

# Machine Learning Process

## Training different algorithm

Based on this document-term matrix, we can now start to train
algorithms. This time, we are actually going to use two different
algorithms (both Naive Bayes and Support Vector Machines) and test which
one performs better.

``` r
# Naive Bayes
nb_model <- textmodel_nb(dfm_train, dfm_train$label)

# Support Vector Machines
svm_model <- textmodel_svm(dfm_train, dfm_train$label)
```

## Testing the algorithms on the held-out data

To validate our algorithms, we again need to preprocess our test data in
the exact same way as we did our training data. Bear in mind that we
need to “match” both document-feature matrices for the prediction to
work.

``` r
# Pre-processing test data
dfm_test <- tweets_test %>% 
  mutate(text = str_remove_all(text, "#")) %>%
  corpus %>% 
  tokens %>%
  dfm %>%
  dfm_match(features = featnames(dfm_train))

# Predicting 
nb_predictions <- predict(nb_model, dfm_test)
svm_predictions <- predict(svm_model, dfm_test)

# Accuracy
mean(nb_predictions == dfm_test$label)
mean(svm_predictions == dfm_test$label) #
```

As we can see, support vector machines have a much higher accuracy!But
we should always investigate the difference in a bit more detail. To do
so, we again built the confusion matrix using the `caret` package. This
time, we save both in a new variable, so that we can extract whatever
information (or validation score) we are interested in.

``` r
# Create confusion matrices
nb_cm <- confusionMatrix(table(predictions = nb_predictions, actual = dfm_test$label), mode = 'prec_recall')
svm_cm <- confusionMatrix(table(predictions = svm_predictions, actual = dfm_test$label), mode = 'prec_recall')

# Check results
nb_cm
svm_cm
```

A nice aspect of using R is that we can customize our results in various
ways. Because we created variables for the results from the function
`confusionMatrix()`, we can create a table that puts the results from
both algorithms next to each other.

``` r
# Extracting accuracy scores
accuracy <- bind_rows(nb_cm$overa, svm_cm$overall) %>%
  mutate(algorithm = c("Naive Bayes", "Support Vector Machines")) %>%
  select(algorithm, Accuracy, Kappa)

# Extracting all other scores and adding the accuracy scores
validation <- bind_rows(nb_cm$byClass, svm_cm$byClass) %>%
  bind_cols(accuracy) %>%
  select(algorithm, Accuracy, `Balanced Accuracy`, Kappa, Precision, Recall, F1)

# Results
validation
```

Perhaps it would be nice, if we could assess this visually:

``` r
validation %>%
  pivot_longer(-algorithm) %>%
  ggplot(aes(x = algorithm, y = value, fill = algorithm)) +
  geom_bar(stat = "identity", position = "dodge") +
  scale_fill_brewer(palette = "Pastel2") +
  ylim(0, 1) +
  guides(fill = F) +
  facet_wrap(~name) +
  coord_flip() +
  theme_bw() +
  labs(x = "", y = "Value",
       title = "Naive Bayes vs. Support Vector Machines")
```

We can clearly see that support vector machines have better values on
all scores. That said, Naive Bayes doesn’t perform that much worse. So
both could potentially be useful in classify new data.

# Using the classifier on a new data set

## Getting some new twitter data

Now, we have learned that particularly the SVM algorithm performs well
based on our test data set. So perhaps we can now use it to classify a
new set of tweets? Let’s get some new twitter data, i.e., \~30,000
english-speaking tweets using the search term “lockdown”. If you have a
Twitter account, you can run the following code to scrape some 18,000
tweets yourself. Note: this can take quite a while…

``` r
library(rtweet)
tweets_new <- search_tweets("lockdown", n = 18000, lang = "en")
write_csv(tweets_new, file = "tweets_new.csv")
```

Alternatively, simply download the data set from Canvas and load it into
R using the standard procedure. We directly select only those variables
that are of interest to us:

``` r
tweets_new <- read_csv("tweets_new.csv") %>%
  select(user_id, status_id, created_at, user = screen_name, text, 
         is_retweet, favorite_count, retweet_count, quote_count, reply_count)
head(tweets_new)
```

## Using the trained algorithm to predict whether these tweets contain “aggressive” language

To classify our new tweets automatically, we need to preprocess our new
data in the exact same way as our train data set and also match (i.e.,
align) the document-feature matrix using `dfm_match()`.

``` r
dfm_tweets <- tweets_new %>%
  mutate(text = str_remove_all(text, "#")) %>%
  corpus %>% 
  tokens %>%
  dfm %>%
  dfm_match(features = featnames(dfm_train))
dfm_tweets
```

Now, we use our algorithm to predict the classes and we add them
directly to our data set. For this example, we are going to predict the
class in three different ways: 1) We simply predict whether a tweet is
aggressive or not using the Naive Bayes classifier, 2) we predict the
probability with which a tweet is aggressive using the Naive Bayes
classifier (an added benefit of this algorithm!), and 3) we predict
whether or not a tweet is aggressive using the support vector machine
algorithm.

We directly add the resulting column to our original data set so that we
can explore relationship between the codes and other variables.
Attention: When we predict the probability (alternative 2) we get both
the probability for a tweet being aggressive vs. not aggressive. So we
only add the second (aggressive) to the data set!

``` r
# Predict aggressiveness in tweets
pred_nb <- predict(nb_model, dfm_tweets)
pred_nb_prop <- predict(nb_model, dfm_tweets, type = "probability")  
pred_svm <- predict(svm_model, dfm_tweets)

# Add prediction to data set
tweets_new$pred_nb = pred_nb
tweets_new$pred_nb_prop = pred_nb_prop[,2]  ## Only the probability of the tweet being aggressive
tweets_new$pred_svm = pred_svm

# Check
tweets_new %>%
  select(contains("pred"))
```

As we can see, the second column includes not 1s and 0s. Instead, it
includes the probability for each tweet to be “aggressive”. A numeric
variable that can tell as a bit more than simply “yes” or “no”.

Of course, we cannot really tell whether any of the algorithms did
“well” on this new data set. After all, we valdiated it on parts of the
data set that we just found online. We would need to manually annotate a
subsample of our new data and compare the automated coding against this
gold standard. For the time being, let’s have a look at some of the
tweets that have been classified as containing “aggressive” language.

``` r
# Filtering based on support vector machines algorithm
tweets_new %>%
  filter(!is_retweet) %>%
  filter(pred_svm == 1) %>%   ## All tweets that are coded as "aggressive"
  select(user, text) %>%
  print(n = 25)

# Filtering based on probability
tweets_new %>%
  filter(!is_retweet) %>%
  filter(pred_nb_prop > .95) %>%  # All tweets that are highly likely to be "aggressive"
  select(user, text) %>%
  print(n = 25)
```

What words do they use? To create a meaningful word cloud, we have to
engage in some more elaborate text pre-processing. We may want to remove
punctuation, symbols, numbers, words that are clearly not related to
aggressiveness, but highly likely to appear in these tweets, stop words,
etc.

``` r
tweets_new %>%
  mutate(text = str_remove_all(text, "#")) %>%
  filter(!is_retweet) %>%
  filter(pred_svm == 1) %>%   ## only those that have been coded as "aggressive"
  corpus %>%
  tokens(remove_punct = T, remove_symbols = T, remove_numbers = T) %>%
  tokens_remove(c("lockdown", "https", "t.co", "covid*", "corona*")) %>% ## removing common, but non-interesting words
  tokens_remove(stopwords("en")) %>%   ## removing stopwords
  dfm %>%
  textplot_wordcloud(max_words = 75, color = c("orange", "red"))
```

Although we already saw that the algorithms don’t do too well in
classifying tweets as aggressive in this data set, the words nonetheless
make a bit sense. We see words such as “anti”, “unvaccinated”, “fuck”.
That at least may indicate some emotional content…

## Some simple analyses

With these results, we can now do some analyses, e.g., how many tweets
overall were classified as aggressive?

``` r
# Naive Bayes
tweets_new %>%
  group_by(pred_nb) %>%
  count %>%
  mutate(prop = n/nrow(tweets_new))

# Support Vector Machines
tweets_new %>%
  group_by(pred_svm) %>%
  count %>%
  mutate(prop = n/nrow(tweets_new))
```

Support Vector Machines were a lot stricter in classifying a tweet as
“aggressive”. Only 3.2% of all 30,000 tweets were labeled as such.

We can also investigate, whether aggressiveness leads to more or less
liking or retweeting:

``` r
# Favorite
tweets_new %>%
  filter(!is_retweet) %>%
  glm(favorite_count > 0 ~ pred_nb_prop, data = ., family = binomial()) %>%
  summary

# Retweet
tweets_new %>%
  filter(!is_retweet) %>%
  glm(retweet_count > 0 ~ pred_nb_prop, data = ., family = binomial()) %>%
  summary
```

Or let’s plot how aggressiveness changes over the course of the day in
tweets on the lockdown…

``` r
library(lubridate)
tweets_new %>%
  mutate(date = round_date(created_at, "hours")) %>%
  group_by(date) %>%
  summarize(aggro = mean(pred_nb_prop, na.rm = T)) %>%  # We are using the probability from the NB classifier here!
  ggplot(aes(x = date, y = aggro)) +
  geom_line(color = "red") +
  theme_classic() +
  labs(x = "Hours", y = "Amount of aggressiveness in tweets",
       title = "Aggressiveness on Twitter across 2 days")
```

**Bonus:** We actually can put both the number of tweets and the level
of aggressiveness in one plot. A second y-axis can be specified using
the `scale_y_continuous` command. The difficult part is to get the
scaling right…. Here, I just roughly normalized by multiplying it with
5000…

``` r
tweets_new %>%
  mutate(date = round_date(created_at, "hours")) %>%
  group_by(date) %>%
  summarize(aggro = mean(pred_nb_prop, na.rm = T), # We are using the probability from the NB classifier here!
            n = n()) %>%
  ggplot() +
  geom_line(aes(x = date, y = aggro), color = "red") +
  geom_line(aes(x = date, y = n/5000), color = "blue", linetype = "dotted") +
  scale_y_continuous(sec.axis = sec_axis(~.*5000, name = "Number of tweets")) +
  theme_classic() +
  labs(x = "Hours", y = "Level of Aggressiveness",
       title = "Aggressiveness on Twitter across 2 days")
```

# Where to go from here?

Machine learning is a broad topic and a lot of new methods are developed
on a daily basis. Within R, the package `tidymodels` may become a new
go-to collection of interesting functions to do machine learning. For
more information, check out this book: [Supervised Machine Learning for
Text Analysis in R](https://smltar.com/) by Emil Hvitfeldt and Julia
Silge. For more advanced “deep learning” methods, check out this online
book: [Deep Learning](https://srdas.github.io/DLBook/) by Subir Varma
and Sanjiv Das.
