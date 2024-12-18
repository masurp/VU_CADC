Word Embeddings
================
Philipp K. Masur
2024-11

- [Introduction](#introduction)
- [Preparation](#preparation)
  - [Loading data](#loading-data)
  - [Visualize word frequency per new year’s resolution
    topic](#visualize-word-frequency-per-new-years-resolution-topic)
- [Analysis with Word-Embeddings](#analysis-with-word-embeddings)
  - [Downloading and understanding
    word-embeddings](#downloading-and-understanding-word-embeddings)
  - [Using these word embeddings for our
    tweets](#using-these-word-embeddings-for-our-tweets)
  - [Computations with
    word-embeddings](#computations-with-word-embeddings)
- [Text Classification with
  Word-Embeddings](#text-classification-with-word-embeddings)
  - [Preparing the data](#preparing-the-data)

# Introduction

In this tutorial, we are going to engage with word-embeddings as better
ways to represent text as numbers. We are also going to fit a neural
network with word-embeddings.

# Preparation

We are going to use the package collections `tidyverse` and `tidytext`.

``` r
library(tidyverse)
library(tidytext)
```

## Loading data

Next, we are going to load the data set that we will analyze in this
tutorial (download it from Canvas and put into the working direction of
your choice). It is a corpus of 5,002 tweets that contained the hashtag
“\#newyearsresolution” and thus contain resolutions by people (scraped
in 2015). Next to a `text` column which contains the actual tweet
content, the data set also contains the `name` of the author, time and
date, as well as a first categorization of the tweets into “topics” or
“categories.

``` r
# Tweets on "New Year's Resolution
tweets <- read_csv2("new_year_resolutions_dataset.csv")
tweets |> 
  head()
```

Let’s do some standard text analysis with this data set. We create a id
variable (often helpful!), select only a few columns, and - as always -
we tokenize the tweets into words. This gets already rid of all symbols
(e.g., `#`), which is usually a good choice (but depends on your
research goals, of course. )

``` r
# Data wrangling
tidy_tweets <- tweets |> 
  mutate(id = 1:n()) |> 
  select(name, text, topic = resolution_category) |> 
  unnest_tokens(word, text)
tidy_tweets |> 
  head()
```

## Visualize word frequency per new year’s resolution topic

How about we do a first descriptive analysis of word frequencies per
topic. As you know, for visualizations it is usually fruitful to remove
some words. Next to standard stopwords, it can be useful to create
additional words that should be removed. Particularly in tweets, there
are often words, that do not help in understanding the tweets. E.g. the
hashtag word “newyearsresolution” is literally in all tweets, so not
helpful to differentiate topics. Below, I added a number of
Twitter-specific terms (e.g., “rt” = retweet, “t.co” = typical URL
abbreviation on Twitter, etc.)

With this preprocessed data, we can visualize the top 10 words in each
topic!

``` r
# Removing some stopwords
add_stopwords <- c("newyearsresolution", "resolution", 
                   "rt", "http", "t.co", "2015", "2014", 
                   "1", "2", "4")

# Visualizing top 10 words per topic
tidy_tweets |> 
  anti_join(stop_words) |> 
  filter(!word %in% add_stopwords) |> 
  group_by(topic, word) |> 
  summarize(n = n()) |> 
  slice_max(n, n = 10) |> 
  ggplot(aes(x = fct_reorder(word, n), y = n, 
             fill = topic)) +
  geom_col() +
  facet_wrap(~topic, scales = "free") +
  coord_flip() +
  theme(legend.position = "none") +
  labs(x = "", y = "Absolute frequency")
```

**Exercise:** What do you see? Do the words make sense? Discuss in
class!

# Analysis with Word-Embeddings

As mentioned in the lecture on Monday, using word embeddings instead of
a document-feature matrix is a more powerful and informative ways to
represent words in a multidimensional vector space (see image below).

![Figure 1: Basic idea of word embeddings](word_embeddings1.png) To get
the word embeddings for our tweet corpus, we have to either train a
shallow neural network to find the weights that represent values on
these dimensions for each word (would be slow and the quality would be
questionable given the small size of the data set and the short format
of tweets) or we use pretrained word-embeddings (e.g., the GloVe word
embeddings).

## Downloading and understanding word-embeddings

We can get the full list of word embeddings trough the package
`textdata`. We can decide for ourselves, how many dimensions we want to
have for each word. Yet, we **are not going to run the code bwlow** as
the word-embeddings would take up almost 1 GB on your harddrive. This is
just to show how we could obtain high quality word-embeddings!

``` r
# Do not run!!!
library(textdata)

glove6b <- embedding_glove6b(dimensions = 100)
glove6b
```

Instead, we use a small subset of 10,000 words with embedding 50
dimensions that we can assess through the supplementary material of the
book by Van Atteveldt et al. We have to wrangle the resulting data type
a bit to get a tidy format.

``` r
# Download data
glove_fn <- "glove.6B.50d.10k.w2v.txt"
url <- glue::glue("https://cssbook.net/d/glove.6B.50d.10k.w2v.txt")
if (!file.exists(glove_fn)) 
    download.file(url, glove_fn)

# Data wrangling
word_embeddings <- read_delim(glove_fn, skip=1, delim=" ", quote="", 
    col_names = c("word", paste0("d", 1:50)))
```

Let’s quickly check out this data set. If we e.g., arrange after the
first dimension, we see that it seems to represent something related to
“aviation” or planes/rockets more generally.

``` r
# 10 highest scoring words on dimension 1
word_embeddings |> 
  arrange(-d1) |> 
  select(1:10)
```

## Using these word embeddings for our tweets

To embed our tweets, we simply “join” the word embeddings with our
tokenized and tidy tweet data set. As both data sets contain a columb
called “word”, it will naturally attach the dimensions to the tidy tweet
data set. We use `inner_join()` as this will join only those that are
actually present in the data set. We also create another version of this
in which we only have unique words, i.e., each row represent a different
word. In the object `embedded_tweets`, we attached dimensions for a word
several times, if it was in the data set more than once!

``` r
# Join embeddings with our tidy tweet data farme
embedded_tweets <- tidy_tweets |> 
  inner_join(word_embeddings) 
embedded_tweets |> 
  head()

# Get unique word embeddings
unique_embeddings <- embedded_tweets |> 
  select(-name, -topic) |> 
  unique()
```

We can now investigate similarities between words from different tweets.
For this, we need to transform our `unique_embeddings` data set into a
matrix (technically not necessary, but speeds up computation).

We further create two function that allow us to find a relevant word and
its dimensions (`wvector()`) and to find similar words in the corpus.

``` r
# Create a matrix from the word embeddings -> fast computation
tweet_matrix <- as.matrix(unique_embeddings[-1])
rownames(tweet_matrix) <- unique_embeddings$word
tweet_matrix <- tweet_matrix / sqrt(rowSums(tweet_matrix^2))

# Function to extract a word vector
wvector <- function(wv, word) wv[word,,drop=F]

# Functionl to compute similarities between word vector and n other words
wv_similar <- function(wv, target, n=5) {
  similarities = wv %*% t(target)
  similarities |>  
    as_tibble(rownames = "word") |> 
    rename(similarity=2) |> 
    arrange(-similarity) |>  
    head(n=n)  
}
```

Let’s this out. Here, we search for the 10 most similar words to
“smoking”, “guitar”, and “happy” in our tweet corpus.

``` r
wv_similar(tweet_matrix, wvector(tweet_matrix, "smoking"), n = 10)
wv_similar(tweet_matrix, wvector(tweet_matrix, "guitar"), n = 10)
wv_similar(tweet_matrix, wvector(tweet_matrix, "happy"), n = 10)
```

## Computations with word-embeddings

Now, we can do some computations with it. For example, we can extract
the word embeddings for the words “college” and “drinking”, then
subtract the latter from the former and investigate which word in the
corpus the result is most similar to:

``` r
# Extract word embeddings
college <- wvector(tweet_matrix, "college")
drinking <- wvector(tweet_matrix, "drinking")

# Compute the result
whatisthis <- college - drinking 

# Compare result to other words in the corpus
wv_similar(tweet_matrix, whatisthis, n = 20)
```

Well, it is reasonable that the result of this subtraction is close to
the word “college” (it is still part of it), but funny enough, we also
have many words like “graduate”, “graduates”, and “graduation”. So
apparently, if you substract “drinking” from “college”, it results in
“graduation”. Perhaps worth remembering in you final year of the
masters! :D

**Exercise:** What other words could you combine by e.g., subtraction or
addition and what does it result in? Play around with these vector
computations!

# Text Classification with Word-Embeddings

Our goal is now to predict the new year’s resolution category from the
tweet. This is a fairly difficult task because the tweets is often short
and cryptic. We are going to do so by building on the GloVe
word-embeddings. Remember, the pipeline looks as follows:

![](img/Slide04.png)

## Preparing the data

As always, we need to prepare the data set. To make the task a bit
easier, we do not use all categories, but just ask the algorithm to find
a ways to classify the tweets in either the category “Personal Growth”
or “Other”. Then, we split the data into a training and a testing set.

``` r
tweets <- tweets |> 
  mutate(label = factor(ifelse(resolution_category == "Personal Growth", "Personal Growth", "Other")))

# Sample 
split <- initial_split(tweets, prop = .95)
train_data <- training(split)
test_data <- testing(split)
```

Now, we have to create the “recipe” for the preprocessing (see last
tutorial!). Here, we only tokenize and add the word-embeddings. Then, we
define a neural network structure, create a workflow and fit a model.

``` r
# Set up the recipe for text preprocessing with GloVe embeddings
text_rec <- recipe(label ~ text, data = tweets) |> 
  step_tokenize(text) |> 
  step_stopwords(text, 
                 custom_stopword_source = c("newyearsresolution", "resolution", "rt",            # <- remove some words that won't help
                                            "http", "t.co", "2015", "2014", "1", "2", "4")) |> 
  step_word_embeddings(text, embeddings = word_embeddings, aggregation = "mean")                 # <-- Here, I am adding the word-embeddings

# Define the MLP model specification
mlp_spec <-
  mlp(epochs = 600,          # <- times that algorithm will work through train set
      hidden_units = c(6),   # <- nodes in hidden units
      penalty = 0.01,        # <- regularization
      learn_rate = 0.2) |>   # <- shrinkage
  set_engine("brulee") |>    # <-- engine = R package
  set_mode("classification")

# Create workflow
mlp_wflow <- workflow()  |> 
  add_recipe(text_rec) |> 
  add_model(mlp_spec)

# Fit the workflow on the training data
mlp_fit <- mlp_wflow |> 
  fit(data = train_data)
```

**Exercise:** Think back about the last tutorial: How can we assess the
performance of this neural network?

``` r
# Solution
```

**Exercise:** Can you find a better architecture for this neural network
and improve the performance?

``` r
# Solution
```
