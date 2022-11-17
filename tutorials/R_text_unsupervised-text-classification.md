Unsupervised Text Classification: Topic Modeling
================
Wouter van Atteveldt, Kasper Welbers &Philipp Masur
2022-11

-   <a href="#introduction" id="toc-introduction">Introduction</a>
    -   <a href="#text-preprocessing" id="toc-text-preprocessing">Text
        Preprocessing</a>
    -   <a href="#excluding-very-frequent-words"
        id="toc-excluding-very-frequent-words">Excluding very frequent words</a>
-   <a href="#estimating-the-topic-model"
    id="toc-estimating-the-topic-model">Estimating the topic model</a>
    -   <a href="#running-the-lda-model" id="toc-running-the-lda-model">Running
        the LDA model</a>
    -   <a href="#inspecting-the-resulting-topic-word-lists"
        id="toc-inspecting-the-resulting-topic-word-lists">Inspecting the
        resulting topic word lists</a>
    -   <a href="#visualizing-lda-with-ldavis"
        id="toc-visualizing-lda-with-ldavis">Visualizing LDA with LDAvis</a>
-   <a href="#what-are-the-tweets-about"
    id="toc-what-are-the-tweets-about">What are the tweets about?</a>
    -   <a href="#probabilities-of-topics-per-tweet"
        id="toc-probabilities-of-topics-per-tweet">Probabilities of topics per
        tweet</a>
    -   <a href="#assigning-topics-to-tweets"
        id="toc-assigning-topics-to-tweets">Assigning topics to tweets</a>

# Introduction

LDA, which stands for Latent Dirichlet Allocation, is one of the most
popular approaches for probabilistic topic modeling. The goal of topic
modeling is to automatically assign topics to documents without
requiring human supervision. Although the idea of an algorithm figuring
out topics might sound close to magical (mostly because people have too
high expectations of what these ‘topics’ are), and the mathematics might
be a bit challenging, it is actually really simple fit an LDA topic
model in R.

A good first step towards understanding what topic models are and how
they can be useful, is to simply play around with them, so that’s what
we’ll do here. First, let’s load some data. In this case, it is a data
set of tweets on new years resolutions.

``` r
library(quanteda)
library(quanteda.textplots)
library(quanteda.textstats)

# Load data
d <- read_csv2("new_year_resolutions_dataset.csv")
head(d)
```

As we can see, we get quite some information about the tweets. Although
we have already some “topics” in the data set, let’s assume we don’t
know yet about them and try to figure out what people write about.

Let’s first create a document term matrix from the tweets in quanteda.
Because it is tweets, we can already remove the “\#” and also get rid of
numbers and punctuations. For topic models, stopwords are really not
that useful, so we will get rid of them too:

## Text Preprocessing

``` r
corp <- d %>%
  mutate(text = str_remove_all(text, "#")) %>%
  corpus(text = "text")

dtm <- corp %>%
  tokens(remove_punct = T, remove_numbers = T) %>%
  tokens_remove(stopwords("en")) %>%
  dfm
dtm
```

Let’s quickly make a wordcloud of the most used words in the tweets.
This is often an important as it will tell as about potential words that
we might want to include before running the topic model.

``` r
textplot_wordcloud(dtm, max_words = 75)
textstat_frequency(dtm, n = 15)  
```

## Excluding very frequent words

As we can see, almost all tweets contain the words
“newyearsresolution,”new”, “resolution”, or “year(s)”. These words
should be excluded, as they do not tell us anything about the topic,
they were probably part of the search queries used to scrape these
tweets.

``` r
dtm <- corp %>%
  tokens(remove_punct = T, remove_numbers = T) %>%
  tokens_remove(stopwords("en")) %>%
  tokens_remove(c("newyearsresolution", "new", 
                  "year*", "newyear", "resolution*", 
                  "rt", "happynewyear", "@*")) %>%
  tokens_select(min_nchar = 2) %>%
  dfm

textplot_wordcloud(dtm, max_words = 75)
```

This looks a lot better and we can proceed with the actual topic
modeling.

# Estimating the topic model

## Running the LDA model

To run LDA from a dfm, first convert to the topicmodels format, and then
run LDA. Note the useof set.seed(.) to make sure that the analysis is
reproducible.

``` r
# Loading package
library(topicmodels)

# Convert dtm to topicmodels' specific format
dtm <- convert(dtm, to = "topicmodels") 

# Set seed to make it reproducible
set.seed(1)

# Fit topic model
m <- LDA(dtm, 
         method = "Gibbs", 
         k = 10,  
         control = list(alpha = 0.1))
m
```

Although LDA will figure out the topics, we do need to decide ourselves
how many topics we want. Also, there are certain hyperparameters (alpha)
that we can tinker with to have some control over the topic
distributions. For now, we won’t go into details, but do note that we
could also have asked for 100 topics, and our results would have been
much different.

## Inspecting the resulting topic word lists

We can use `terms()` to look at the top terms per topic:

``` r
terms(m, 15)
```

**Question:** Any ideas what these topics stand for?

The posterior function gives the posterior distribution of words and
documents to topics, which can be used to plot a word cloud of terms
proportional to their occurrence:

``` r
topic <- 9
words <- posterior(m)$terms[topic, ]
topwords <- head(sort(words, decreasing = T), n = 20)
head(topwords, 10) 
```

Now we can plot these words:

``` r
library(wordcloud)
wordcloud(names(topwords), topwords)
```

**Exercise:** Pick another topic and check out the most used words. What
is it about?

``` r
# Solution here
```

Of course, it would be interesting to plot all topwords of all topics in
one go. We can do that with a little data wrangling:

``` r
tibble(topic = 1:10) %>%
  apply(1, function(x) posterior(m)$terms[x,]) %>%
  as.data.frame() %>%
  rownames_to_column("terms") %>%
  as_tibble %>%
  gather(key, value, -terms) %>%
  group_by(key) %>%
  arrange(key, -value) %>%
  slice(1:10) %>%
  ggplot(aes(x = fct_reorder(terms, value), 
                y = value,
             fill = key)) +
  geom_col() +
  facet_wrap(~key, scales = "free", nrow = 2) +
  coord_flip() +
  theme(legend.position = "none") +
  labs(y = "Frequency", x = "",
       title = "Topwords per Topic")
```

We can also look at the topics per document, to find the top documents
per topic:

``` r
topic.docs <- posterior(m)$topics[, 9] 
topic.docs <- sort(topic.docs, decreasing=T)
head(topic.docs)
```

We see for example that the tweets 1979 has a 93% probability of being
about topic 9 (stop smoking/drinking). Given the document ids of the top
documents, we can look up the text in the corp corpus

``` r
topdoc <- names(topic.docs)[1]
topdoc_corp <- corp[docnames(corp) == topdoc]
texts(topdoc_corp)
```

## Visualizing LDA with LDAvis

`LDAvis` is a nice interactive visualization of LDA results. It needs
the LDA and DTM information in a slightly different format than what’s
readily available, but you can use the code below to create that format
from the lda model `m` and the `dtm`. If you don’t have it yet, you’ll
have to install the `LDAvis` package, and you might also have to install
the `servr` package.

``` r
library(LDAvis)   

dtm <- dtm[slam::row_sums(dtm) > 0, ]
phi <- as.matrix(posterior(m)$terms)
theta <- as.matrix(posterior(m)$topics)
vocab <- colnames(phi)
doc.length <- slam::row_sums(dtm)
term.freq <- slam::col_sums(dtm)[match(vocab, colnames(dtm))]

json <- createJSON(phi = phi, theta = theta, vocab = vocab,
     doc.length = doc.length, term.frequency = term.freq)
serVis(json)
```

# What are the tweets about?

## Probabilities of topics per tweet

The posterior function also gives the posterior distribution documents
to topics. This is interesting to understand which tweet is about what:

``` r
topic_prob <- posterior(m)$topics
head(topic_prob) %>%
  round(4)
```

We can of course also plot this as a bar plot:

``` r
topic_prob_long <- topic_prob %>%
  as.data.frame %>%
  rownames_to_column("docs") %>%
  gather(topic, value, -docs) %>%
  as_tibble %>%
  arrange(docs, value) 

topic_prob_long %>%
  filter(docs == "text1" | docs == "text2" | docs == "text3") %>%
  ggplot(aes(x = fct_reorder(topic, value), y = value, fill = docs)) +
  geom_col() +
  coord_flip() +
  facet_wrap(~docs) +
  labs(y = "Probability of a tweet being about a certain topic", 
       x = "Topic",
       title = "Probabilites of a tweet being about a certain topic") +
  theme(legend.position = "none")
```

## Assigning topics to tweets

If we group by documents and filter out the max probabiliy, we assign
each tweet the most probable topic (in some cases, this can mean that
two topics with the same probabilities are both assigned to a text):

``` r
(topic_table <- topic_prob_long %>%
  group_by(docs) %>%
  filter(value == max(value)))
```

Let’s have a look how often each of these topic occur in the data set

``` r
topic_table %>%
  group_by(topic) %>%
  count %>%
  mutate(prob = n/nrow(topic_table)) %>%
  arrange(-prob)
```

Bear in mind, we arbitrarily chose 10 topics for our LDA topic model. In
the next session, we will explore ways to identify the best number of
topics.
