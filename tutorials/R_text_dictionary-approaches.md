Dictionary Approaches: Brand Analysis in R
================
Wouter van Atteveldt, Kasper Welbers & Philipp Masur
2021-11

-   [Introduction](#introduction)
-   [Getting some data](#getting-some-data)
-   [Getting a DTM](#getting-a-dtm)
-   [Dictionary-based analysis](#dictionary-based-analysis)
    -   [Obtaining a dictionary](#obtaining-a-dictionary)
    -   [Applying a quanteda
        dictionary](#applying-a-quanteda-dictionary)
    -   [Analyzing the results](#analyzing-the-results)
    -   [Validating a dictionary](#validating-a-dictionary)
-   [Sentiment analysis](#sentiment-analysis)
    -   [Creating a quanteda dictionary from a word
        list](#creating-a-quanteda-dictionary-from-a-word-list)
    -   [Improving a dictionary](#improving-a-dictionary)

# Introduction

In this tutorial, we are going to learn how to use existing dictionaries
to analyze larger text corpora. Dictionaries are often used to conduct a
so-called sentiment analysis (and we will run a sentiment analysis at
the end of this tutorial), but they can also be used to characterize
brands/companies.

For example, [Nadeau, Rutter and
Lettice](https://doi.org/10.1080/0267257X.2020.1764080) (2020) used the
[Brand Personality
Dictionary](https://provalisresearch.com/products/content-analysis-software/wordstat-dictionary/brand-personality-dictionary/)
to understand the process of attitudinal changes towards a brand in
crisis and the brand’s communication around the crisis. They collected
data (tweets) from brands’ Twitter platforms and use this to conduct
extensive text analyses.

In this tutorial, we will do something very similar using the same
dictionary. However, as we will see, there are a lot of problems with
these approaches. Particularly, the validity of such dictionaries is -
despite them being used in actual research - often questionable. So
overall, the main lessons of today’s tutorial is that you should always
validate to make sure your results are valid for your task and domain.
For example, it is always a good idea to adapt a lexicon (dictionary) to
your domain/task by inspecting (the context of) the most common words.
Depending on resources, using crowd coding and/or machine learning can
also be a better option than a purely lexical (dictionary-based)
approach.

In this tutorial, we again are going to use functions from the
`tidyverse` and the collection of packages around `quanteda`.

``` r
library(tidyverse)
library(quanteda)
library(quanteda.textplots)
library(quanteda.textstats)
```

# Getting some data

For this example, we will use the latest \~3200 tweets of four different
mobile phone companies (Google, Huawei, Samsung, and Sony). If you want,
you can scrape their newest tweets using the package `rtweet` and the
following code. Note: You will need a Twitter Account to actually scrape
the tweets.

``` r
library(rtweet)

# Get timelines of major mobile phone companies
tmls1 <- get_timeline(c("Google", "Huawei"), n = 3200)         ## we can only scrape two twitter timelines at a time
tmls2 <- get_timelines(c("SamsungMobile", "Sony"), n = 3200)

# Bind them together
tml <- bind_rows(tmls1, tmls2)  ## merging results from both scraping processes

# Save data frame
write_csv(tml, file = "phone_tweets.csv")
```

For the time being, let’s simply use the following data set (which I
uploaded on Canvas), which contains the latest tweets of these companies
from last Monday.

``` r
tweets <- read_csv("phone_tweets.csv")
tweets
```

In a first step, we engage in some simple data wrangling. We first
create a unique post id and then select specific variables from the
comparatively large data set. As a final, but very important step when
we use Twitter data, we remove the `#` from the tweets as R will
otherwise not recognize e.g., “\#awesome” as the same word as “awesome”

``` r
tweets <- tweets %>%
  group_by(screen_name) %>%
  mutate(post_id = paste(user_id, 1:n(), sep = "_")) %>%
  select(user_id, post_id, created_at, screen_name, description, text, favorite_count, retweet_count) %>%
  mutate(text = str_remove_all(text, "#")) %>%       ## removing '#' from the tweets
  ungroup
tweets
```

When we look at the data, we see that we have a variable that tells us
when the tweet was created (created\_at), how created the tweet
(screen\_name), the content of the tweet (text) and some information
about whether it was liked (favorite\_count) or retweeted
(retweet\_count).

Because the format is still a simple data set (or tibble), we can
quickly check how many tweets we have per company and check how much
each company tweeted over time:

``` r
# Number of tweets per company
tweets %>%
  group_by(screen_name) %>%
  count

# Tweets over time
tweets %>%
  mutate(date = as.Date(created_at)) %>%
  group_by(screen_name, date) %>%
  summarize(n = n()) %>%
  ggplot(aes(x = date, y = n, color = screen_name)) +
  geom_line() +
  theme_bw() +
  labs(color = "Smartphone Provider", x = "Time", y = "Number of tweets")
```

We can clearly see that the last \~3000 tweets of each company cover
vastly different time spans. Furthermore, we see that “SamsungMobile”
tweets a lot on particular dates (spikes!), but not as much otherwise.
Google, of course, which do not only tweet about their phones, tweets a
lot more than the others.

# Getting a DTM

As we learn in the last practical session, the main primitive for any
dictionary analysis is the document-term matrix (DTM). For more
information on creating a DTM from a vector (column) of text, see again
the [tutorial on basic text analysis with
quanteda](https://github.com/masurp/VU_CADC/blob/main/tutorials/R_text_basics-of-text-analysis.md).
For this tutorial, it is important to make sure that the preprocessing
options (especially stemming) match those in the dictionary: if the
dictionary entries are not stemmed, but the text is, they will not
match. In case of doubt, it’s probably best to skip stemming altogether.

For our text analysis, we first need to transform this data set into a
corpus format. We can do so with the function `corpus()` which only asks
us which variable serves as document id (in our case the newly created
“post\_id”) and which contains the actual text (“text”). Note again that
all other variables are added as “docvars”!

``` r
corp <- corpus(tweets, docid_field = 'post_id', text_field = 'text')
corp
```

Now, we can create a document-term matrix as usual. In a first step, we
won’t apply any preprocessing steps (other than lowercasing, the default
in `dfm()`), because dictionaries tend to have full (i.e. not-stemmed)
words, and sometimes contain punctuation such as emoticons.

``` r
dtm <- corp %>% 
  tokens %>%
  dfm
dtm
```

And when we have a dtm, we might as well make a word cloud to make sure
that the results make some sense:

``` r
textplot_wordcloud(dtm, max_words=200, color = c("lightblue", "orange", "red", "black"))
```

There are a lot of unncessary words in the dfm (e.g., search terms, stop
words, symbols…). Although this is not necessarily problematic for a
dictionary analysis, we can nonetheless - for the moment - remove them
to get a better word cloud.

``` r
dtm2 <- corp %>% 
  tokens(remove_punct = T, remove_numbers = T, remove_symbols = T) %>% 
  tokens_remove(stopwords("en")) %>%
  tokens_remove("https*") %>%     ## Removing any links
  dfm %>%
  dfm_select(min_nchar = 2) %>%   ## at least two characters
  dfm_trim(min_docfreq=10)        ## word should be in the data set at least 10 times
textplot_wordcloud(dtm2, max_words=100, color = c("lightblue", "orange", "red", "black"))
```

# Dictionary-based analysis

## Obtaining a dictionary

To do a dictionary-based analysis, we can simply use the `dfm_lookup`
method to apply an existing dictionary to a dfm (similar to last
session). There are many existing dictionaries that can be downloaded
from the Internet.

Here, we are going to use the [Brand Personality
Dictionary](https://provalisresearch.com/products/content-analysis-software/wordstat-dictionary/brand-personality-dictionary/)
(Opoku, Abratt, and Pitt, 2006) that has been used by Nadeau et
al. (2020) for classyifing brands with regard to different latent
variables:

-   Competence
-   Excitement
-   Ruggedness
-   Sincerity
-   Sophistication

As it stored in a zip-file on the website, we can use R to download it,
unzip and directly transform it into a quanteda dictionary using the
function `dictionary()`

``` r
download.file("https://provalisresearch.com/Download/BrandPersonality.zip", "BrandPersonality.zip")
unzip("BrandPersonality.zip", 'Brand Personality.cat')
brandPers <- dictionary(file="Brand Personality.cat")
brandPers
```

As we can see, the dictionary includes extensive (&gt; 120 words per
category) for each of the five brand personality dimensions. Note: Some
of the words are actually not unigrams (i.e., more than one word such as
“award\_winning”). In this form, they are more or less useless to try to
code text that is tokenized into single words. This is a potential
drawback of this particular dictionary. We could try to recode the
dictionary or try to tokenize the text differently. For the time being,
we will just go ahead with this limitation.

## Applying a quanteda dictionary

Now that we have a dtm and a dictionary, applying it is relatively
simple by using the `dfm_lookup` function. To use the result in further
analysis, we convert to a data frame and change it in to a tibble. The
last step is purely optional, but it makes working with it within
tidyverse slightly easier:

``` r
dtm_brand <- dtm %>% 
  dfm_lookup(brandPers)
dtm_brand

result <- dtm_brand %>%
  convert(to = "data.frame") %>% 
  as_tibble
result
```

We know have a data frame that includes information about how often any
of the words related to one of the five brand personality dimensions
occurred in the tweet.

In a final step, we know would probably like to combine this results
with the initial data set so that we know which text belonged to which
company.

``` r
tweets2 <- tweets %>% 
  rename(doc_id = post_id) %>%  ## renaming the idea variable to match the result data frame
  inner_join(result) 
tweets2
```

## Analyzing the results

Usually, coding our tweets manually is not really the end goal of our
analysis. Instead, we now can use this information to further analyze
our text. A first very simple analysis could be to summarize the amount
of times each brand personality dimension was mentioned by the different
companies.

``` r
tweets2 %>%
  select(company = screen_name,
         COMPETENCE:SOPHISTICATION) %>%
  pivot_longer(-company, names_to = "brand_pers") %>%
  group_by(company, brand_pers) %>%
  summarize(number_of_mentions = sum(value)) %>%
  pivot_wider(names_from = "brand_pers", values_from = "number_of_mentions")
```

Of course, instead of a table, we could also plot the differences using
a barplot:

``` r
tweets2 %>%
  group_by(screen_name) %>%
  summarize(competence = sum(COMPETENCE)/n(),
            excitement = sum(EXCITEMENT)/n(),
            ruggedness = sum(RUGGEDNESS)/n(),
            sincerity = sum(SINCERITY)/n(),
            sophistication = sum(SOPHISTICATION)/n()) %>%
  pivot_longer(-screen_name) %>%
  ggplot(aes(x = screen_name, y = value, fill = name)) +
  geom_bar(stat = "identity", position = "dodge") +
  scale_fill_brewer(palette = "Set2") +
  theme_replace() +
  labs(x = "Smartphone Company", y = "Number of mentions in tweets", fill = "Brand Personality")
```

And to introduce some statistical approaches and how you can do them in
R, here is an analysis of variance (ANOVA) testing the differences in
“COMPETENCE” across the four companies.

``` r
library(emmeans) # don't forget to install the package first!
m_comp <- aov(COMPETENCE ~ screen_name, tweets2)

# Model summary
summary(m_comp)

# Means and confidence intervals
tab <- emmeans(m_comp, specs = c("screen_name"))
tab

# Pairwise comparisons using Tukey's HSD
pairs(tab, adjust = "tukey")
plot(tab, comparisons = T)
```

## Validating a dictionary

To get an overall quantitative measure of the validity of a dictionary,
we should manually code a random sample of documents and compare the
coding with the dictionary results. This can (and should) be reported in
the methods section of a paper using a sentiment dictionary.

You can create a random sample from the original data frame using the
sample function:

``` r
sample_ids <- sample(docnames(dtm), size=50)
```

``` r
## convert quanteda corpus to data.frame
docs <- docvars(corp)
docs$doc_id <- docnames(corp)
docs$text <- as.character(corp)

# Create a csv-file of the sub sample
docs %>% 
  filter(doc_id %in% sample_ids) %>% 
  mutate(manual_competence = "",
         manual_excitement = "",
         manual_ruggedness = "",
         manual_sincerity = "",
         manual_sophistication = "") %>% 
  write_csv("to_code.csv")
```

Then, you can open the result in excel, code the documents by filling in
the sentiment column, and read the result back in and combine with your
results above.

Note that I rename the columns and turn the document identifier into a
character column to facilitate matching it:

``` r
validation <- read_csv("to_code.csv") %>% 
  mutate(doc_id=as.character(doc_id)) %>% inner_join(result)
```

In the following code, I simply create a “random” manual coding to
demonstrate some of the typical validation steps (we will engage in more
meaningful validation practices in later practical sessions).

``` r
# Just creates just a column with "random" manual codings
validation <- tibble(doc_id=sample_ids, manual_competence = rep(c(1,1, 1, 0, 0), 10)) %>% 
  inner_join(result)
```

Now let’s see if my (admittedly completely random) manual coding matches
the sentiment score. We can do a correlation:

``` r
cor.test(validation$manual_competence, validation$COMPETENCE)
```

We can also get a ‘confusion matrix’ if we create a nominal value from
the variable “COMPETENCE” (1 = coded as “competence”, 0 = not coded as
“competence”, thereby neglecting when a tweets mentions several words of
a category)

``` r
validation <- validation %>%
  mutate(competence = ifelse(COMPETENCE > 1, 1, COMPETENCE))
cm <- table(manual = validation$manual_competence, dictionary = validation$competence)
cm
```

This shows the amount of errors in each category. Total accuracy is the
sum of the diagonal of this matrix divided by total sample size.

``` r
sum(diag(cm)) / sum(cm)
```

# Sentiment analysis

In this second part of this tutorial, we will engage in a special case
of a dictionary analysis: a so-called sentiment analysis. Some of the
most important questions about text have to do with *sentiment* (or
*tone*): Is a text in general positive or negate? Are actors described
as likable and successful? Is the economy doing well or poorly? Is an
issue framed as good or bad? Is an actor in favor of or against a
certain policy proposal?

*Caveat*: This method is very successful for some tasks such as deciding
whether a review is positive or negative. In other cases, however, one
should be more careful about assuming dictionary analyses are valid.
Especially in political communication, sentiment can mean one of
multiple things, and many texts contain multiple statements with
opposing sentiment.

For more information and critical perspectives on dictionary based
sentiment analysis, see e.g. the references below:

-   Soroka, S., Young, L., & Balmas, M. (2015). Bad news or mad news?
    sentiment scoring of negativity, fear, and anger in news content.
    The ANNALS of the American Academy of Political and Social Science,
    659 (1), 108–121.
-   González-Bailón, S., & Paltoglou, G. (2015). Signals of public
    opinion in online communication: A comparison of methods and data
    sources. The ANNALS of the American Academy of Political and Social
    Science, 659(1), 95-107.
-   Barberá, P., Boydstun, A., Linn, S., McMahon, R., & Nagler, J.
    (2016, August). Methodological challenges in estimating tone:
    Application to news coverage of the US economy. In Meeting of the
    Midwest Political Science Association, Chicago, IL.

For easy use, the package `SentimentAnalysis` contains 3 dictionaries:
`DictionaryGI` is a general sentiment dictionary based on The General
Inquirer, and `DictionaryHE` and `DictionaryLM` are dictionaries of
finance-specific words presented by Henry (2008) and Loughran & McDonald
(2011) respectively. The package `qdapDictionaries` also contains a
number of interesting dictionaries, including a list of `positive.words`
and `negative.words` from the sentiment dictionary of Hu & Liu (2004),
and specific lists for `strong.words`, `weak.words`, `power.words` and
`submit.words` from the Harvard IV dictionary.

You can inspect the lists and their origins from each package by loading
it and getting the help:

``` r
library(SentimentAnalysis)
?DictionaryGI
names(DictionaryGI)
head(DictionaryGI$negative, 27)

library(qdapDictionaries)
?weak.words
head(weak.words, 27)
```

You can also download many dictionaries as CSV. For example, the VADER
lexicon is specifically made for analysing social media and includes
important words such as “lol” and “meh” that are not present in most
standard dictionaries:

``` r
url <- "https://raw.githubusercontent.com/cjhutto/vaderSentiment/master/vaderSentiment/vader_lexicon.txt"
vader <- read_delim(url, col_names=c("word","sentiment", "details"),  col_types="cdc",  delim="\t")
vader
```

Which dictionary you should use depends on your task and research
question. Sometimes, it may even be best to use several ones and compare
their validity. Again, make sure that your choices for preprocessing
match your analysis strategy: if you want to use e.g. the emoticons in
the VADER dictionary, you should not strip them from your data.

## Creating a quanteda dictionary from a word list

You can apply the dictionaries listed above again using dfm\_lookup. The
dictionaries from `SentimentAnalysis` can be directly turned into a
quanteda dictionary:

``` r
GI_dict <- dictionary(DictionaryGI)
```

For the word lists, you can compose a dictionary e.g. of positive and
negative terms: (and similarly e.g. for weak words and strong words, or
any list of words you find online)

``` r
HL_dict <- dictionary(list(positive=positive.words, negative=negation.words))
```

To run the actual sentiment analysis, we can use the same dtm that we
created earlier and again use the `dfm_lookup()` function. Note that we
are adding a variable “dict” that tells us which dictionary we have used
and already produce a length variable (i.e., a variable that contains
the length of each tweet) that we will use later to compute a sentiment
score.

``` r
sentiment1 <- dtm %>%
  dfm_lookup(GI_dict) %>%
  convert("data.frame") %>%
  as_tibble %>%
  mutate(dict = "GI_dict") %>%
  mutate(length = ntoken(dtm))

sentiment2 <- dtm %>%
  dfm_lookup(HL_dict) %>%
  convert("data.frame") %>%
  as_tibble %>%
  mutate(dict = "HL_dict") %>%
  mutate(length = ntoken(dtm))

sentiment <- bind_rows(sentiment1, sentiment2)
sentiment
```

Now, we probably want to compute some sort of overall sentiment score.
We can make various choices here, but a common one is to subtract the
negative count from the positive count and divide by either the total
number of words or by the number of sentiment words. We can also compute
a measure of subjectivity to get an idea of how much sentiment is
expressed in total:

``` r
sentiment <- sentiment %>% 
  mutate(sentiment1 = (positive - negative) / (positive + negative),
         sentiment2 = (positive - negative) / length,
         subjectivity = (positive + negative) / length)
sentiment
```

As a final step, we can again add this data.frame to our original data
frame that contains the tweets and companies

``` r
tweets3 <- tweets %>% 
  rename(doc_id = post_id) %>%  ## renaming the idea variable to match the result data frame
  inner_join(sentiment) 
tweets3
```

For now, let us simply investigate differences in subjectivity across
the four companies.

``` r
tweets3 %>%
  select(screen_name, subjectivity, dict) %>%
  group_by(dict, screen_name) %>%
  summarize(sub = mean(subjectivity)) %>%
  ggplot(aes(x = dict, y = sub, fill = screen_name)) +
  geom_bar(stat = "identity", position = "dodge") +
  labs(x = "Type of Dictionary", y = "Subjectivity Score", fill = "Company")
```

## Improving a dictionary

To improve a sentiment dictionary, it is important to see which words in
the dictionary are driving the results. The easiest way to do this is to
use `textstat_frequency` function and then using the tidyverse filter
function together with the `%in%` operator to select only rows where the
feature is in the dictionary:

``` r
freqs <- textstat_frequency(dtm)
freqs %>% 
  as_tibble %>% 
  filter(feature %in% HL_dict$positive)
```

As you can see, the most frequent ‘positive’ words found are ‘better’
and ‘thank’. Now, it’s possible that these are actually used in a
positive sense (“We have become better at…”, “We are thankful…”), but it
is equally possible that they are used negative, especially the word
“better” (e.g., “We should do better…”).

To find out, the easiest method is to get a keyword-in-context list for
the term:

``` r
head(kwic(tokens(corp), "better"))
```

From this, it seems that the word `better` here is not used as a
positive verb, but rather as a way to indicate the the providers can
make the app or phone even better. To remove it from the list of
positive words, we can use the `setdiff` (difference between two sets)
function:

``` r
positive.cleaned <- setdiff(positive.words, c("better"))
HL_dict2 <- dictionary(list(positive = positive.cleaned, negative = negation.words))
```

To check, look at the top positive words that are now found:

``` r
freqs %>% 
  filter(feature %in% HL_dict2$positive) %>%
  as_tibble
```

This seems like a lot of work for each word, but even just checking the
top 25 words can have a very strong effect on validity since these words
often drive a large part of the outcomes.

Similarly, you can check for missing words by inspecting the top words
not matched by any of the terms (using `!` to negate the condition)

``` r
sent.words <- c(HL_dict$positive, HL_dict$negative)
freqs %>% 
  filter(!feature %in% sent.words) %>% 
  View
```

By piping the result to View, it is easy to scroll through the results
in rstudio. Note that this does not work in a Rmd file since View cannot
be used in a static document!

Scroll through the most frequent words, and if you find a word that
might be positive or negative check using `kwic` whether it is indeed
(generally) used that way, and then add it to the dictionary similar to
above, but using the combination function `c` rather than `setdiff`.
