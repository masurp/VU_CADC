Word Embeddings and Transformers
================
Philipp K. Masur
2023-11

- [Introduction](#introduction)
  - [Setting up a hugging face
    account](#setting-up-a-hugging-face-account)
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
- [Classification using a BERT model via hugging
  face](#classification-using-a-bert-model-via-hugging-face)
  - [A small example](#a-small-example)
  - [A more elaborate example](#a-more-elaborate-example)

# Introduction

In this tutorial, we are going to engage with a) word-embeddings and b)
zero-shot learning using a BERT model from hugging face.

## Setting up a hugging face account

Although we can assess the hugging face account directly, it is better
to link our analyis here with an account as this will increase the rate
limit for API prompts. As a first step, I hence would like to ask you to
create a hugging face account. For this, please follow these steps:

1.  Go to <https://www.huggingface.co>

2.  Click on “Sign Up” in the upper right corner and follow the steps.

3.  Once you have an account, click on your account picture in the upper
    right corner and click on settings:

![](hf_1.png) 4. Click on “Access Tokens” on the left and create a new
one.

![](hf_2.png)

5.  Copy the code. We will use it in the tutorial shortly.

# Preparation

In this tutorial, we are going to use the package collections
`tidyverse` and `tidytext`. Now, please enter the Access Token from your
hugging face account in the code below (replace `XXXX`).

``` r
library(tidyverse)
library(tidytext)

# Provide access to hugging face account
Sys.setenv(
  HF_API_TOKEN = "XXXX" # <-- enter your hugging face token here
)
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

# Classification using a BERT model via hugging face

Now, we are going to use a BERT model to classify the topic of the
tweets. We are not going to fine-tune the model and hence rely on
zero-shot learning. The figure below shows the zero-shot classification
pipeline.

![](Slide06.png)

## A small example

Let’s start small and only classify the first 10 tweets. We can do this
with a specific function from the package `ccsamsterdamR`, which
contains function that we can use to very easily access the hugging face
API and thus a variety of BERT models (and other models too!). Please
install the pacakge now and then load it.

The package contains the function `hf_zeroshot()`, which provide a very
simple syntax for zero-shot text classification. You only have to
provide the text you want to have classified as a vector. Next, you need
to provide the labels from which the model should choose to predict the
outcome class. Finally, you have to provide the URL to the model that
you want to use. In this case, we are using a smaller version of BERT
called “DEBERTA”, which our colleage Moritz Laurer pretrained on various
classification tasks to improve its zero-shot learning ability.

To find other models, you can go to <https://huggingface.co/models> and
check out what else you can use. To provide an URL to the function,
simple add follow this syntax:
`"https://api-inference.huggingface.co/models/[author]/[modelname]"`.
Depending on the traffic on the site, this will take a moment to produce
the results.

``` r
# Install the package from github
remotes::install_github("ccs-amsterdam/ccsamsterdamR/")

# Load course-specific package
library(ccsamsterdamR)

# Simple zero-shot analysis of the first 10 tweets
results <- hf_zeroshot(txt = tweets$text[1:10],
                       labels = c("health", "personal growth", "education", "career", "family", "relationship", "hobby"),
                       url = "https://api-inference.huggingface.co/models/MoritzLaurer/deberta-v3-large-zeroshot-v1")
results
```

To see the results, we need to unnest the columns `labels` and `scores`.
This creates a table in which each tweet receives a probability score
with which it fits into any of the topic categories.

While we’re at it, let’s further boil this down to a format that we are
already used to. By transforming this set to a long format, an filtering
based on the maximum probability, we get the most likely topic per
tweet!

``` r
# Data wrangling to get a nice output
result_table <- results |> 
  unnest(cols = c("labels", "scores"))  |> 
  as_tibble() |> 
  spread(labels, scores) |> 
  mutate_if(is.numeric, round, 5)
result_table

# Get topic per tweet
result_table |> 
  pivot_longer(career:relationship) |> 
  group_by(sequence) |> 
  filter(value == max(value)) 
```

## A more elaborate example

Via the API, we can only classify a few tweets in one go before that
will stop the processing. It is not designed to allow every single user
to classify infinite amount of texts. We can circumvent this by
splitting our data in smaller chunks and gradually classify these chunks
one after another. If the traffic is not to high on the site, this
should (we need to lower the amount of text per chunk and increase the
dealy between prompts, if it breaks!)

But first, let’s create a simpler topic column against which we can
evaluate the BERT model’s performance. Here, I am generating a simple
topic vector based on the already existing resolution category column.

``` r
tweets2 <- tweets |> 
  mutate(topic = case_when(grepl("Health", resolution_category) ~ "health",
                           grepl("Personal Growth", resolution_category) ~ "growth",
                           grepl("Philanthropic", resolution_category) ~ "philanthropic",
                           grepl("Career", resolution_category) ~ "career",
                           grepl("Leisure", resolution_category) ~ "leisure",
                           grepl("Relationships", resolution_category) ~ "relationships")) |> 
  filter(!is.na(topic))
```

Now, we need to split the data. The `ccsamsterdamR` package includes a
function for this. To not go crazy, we only going to classify the first
60 tweets and hence only split those into groups of 15. Then, we create
a loop with the function `map()`, which iteratively sends the prompt to
the API and extracts the results. We can add a delay between prompts to
make sure that we do not send to many prompts per minute (this of course
makes the process longer).

``` r
# Split data sets in reasonable chunks (a bit of trial and error in light of what works with the API)
splits <- gpt_split_data(tweets2[1:60,], n_per_group = 15)

# MAp across the chunks. 
map_results <- map_df(splits, function(x) {
  output <- hf_zeroshot(
    txt = x$text,
    labels = c("health", "growth", "philanthropic", "leisure", "relationships", "career"),
    url = "https://api-inference.huggingface.co/models/MoritzLaurer/deberta-v3-large-zeroshot-v1")
  Sys.sleep(15) # Delay between prompts
  output
})

# Check
map_results
```

Now we can wrangle the data to get a “predict” object like we are used
to from earlier practical session and that we can use to compute
confusion matrix and performance scores.

``` r
# Unnest columns
map_results_table <- map_results |> 
  unnest(cols = c("labels", "scores"))  |> 
  as_tibble() |> 
  spread(labels, scores) |> 
  mutate_if(is.numeric, round, 4)
map_results_table |> 
  head()

# Select most probable topic per Tweet
final_results <- map_results_table |> 
  pivot_longer(career:relationships) |> 
  group_by(sequence) |> 
  filter(value == max(value)) |> 
  unique() |> 
  select(-error)

# Bind prediction to actual topic (gold standard) in the data set 
predict_hf <- final_results |> 
  rename(text = sequence,
         predicted = name) |>
  inner_join(tweets2[1:100,], by = "text") |> 
  select(topic, predicted) |> 
  mutate(topic = factor(topic, levels = c("growth", "health","relationships", "leisure", "career", "philanthropic")),
         predicted = factor(predicted, levels = c("growth", "health","relationships", "leisure", "career", "philanthropic"))) |> 
  ungroup()

predict_hf |> 
  head()
```

**Exercise 1:** Now, we have a “predict”-object similar to the ones we
had before, can you create the confusion matrix and estimate relevant
performance scores?

``` r
library(tidymodels)

# Code here
```

**Exercise 2:** Can you classify the next bunch of tweets and add them
to the already classified tweets to recompute the performance?

``` r
# Code here
```
