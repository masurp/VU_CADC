One-Shot and Few-Shot Classification with LLMs
================
Philipp K. Masur
2023-11

- [Introduction](#introduction)
- [Preparation](#preparation)
  - [Getting data](#getting-data)
  - [Again a Small Example with either llama or
    GPT](#again-a-small-example-with-either-llama-or-gpt)
- [Classifying more than a couple of
  texts](#classifying-more-than-a-couple-of-texts)
  - [Zero-Shot Classification](#zero-shot-classification)
  - [Validation](#validation)
- [Classifying with some examples](#classifying-with-some-examples)
  - [One-Shot Classification](#one-shot-classification)
  - [Validation](#validation-1)
  - [Few-Shot Classification?](#few-shot-classification)

# Introduction

In the last tutorial, we have worked with llama and/or GPT-3.5 to
perform some zero-shot classifications of sentiment of yelp reviews. In
this tutorial, we will dive deeper into this logic and explore ways to
provide these models with more examples that may increase their
performance even more.

Again, the pipeline is as follows:

![](Slide06.png)

# Preparation

For this tutorial, we are again going to load the `tidyverse`,
`tidytext`, `tidymodels`, `tidyllm`, and the package `glue`.

``` r
library(tidyverse)
library(tidytext)
library(tidymodels)
library(tidyllm)
library(glue)
```

## Getting data

We are going to explore a data set that includes movie descriptions and
the respective genre of the movie. The goal will be to predict the genre
from the movie description.

``` r
# Load data
movies <- read_csv("tutorials/data/wiki_movie_plots_deduped.csv")
movies |> 
  head()
```

Let’s also directly set the OpenAI API key so that we can use the GPT
later on.

``` r
# Provide access to OPEN AI account
Sys.setenv(
  OPENAI_API_KEY = 'XXX' # <- enter token here
)
```

Let’s explore the film genres in this data set:

``` r
movies |>  
  group_by(Genre) |> 
  tally() |> 
  filter(Genre != "unknown") |> 
  slice_max(n, n = 25) |> 
  ggplot(aes(x = fct_reorder(Genre, n), y = n, fill = Genre)) +
  geom_col() +
  geom_label(aes(label = n), fill = "white", size = 2.5) +
  coord_flip() +
  labs(x = "", y = "Number of movies in data set") +
  theme_minimal() +
  theme(legend.position = "none")
```

As we can see, most movies are classified as “drama”, but we also have a
lot of comedies. For the purpose of this tutorial, let’s break this down
a bit more. Having any language model trying to predict this many
different genres will probably not result in great performance.

So for the purpose of this tutorial, we are going to sample 200 movies
that were release after 2010 (so that we hopefully all now them!). We
filter so that we have only action, comedy, and horror movies in the
data set.

``` r
set.seed(123)
movies_small <- movies |> 
  mutate(id = 1:n()) |> 
  rename(genre = Genre) |> 
  filter(genre %in% c("action", "comedy", "horror")) |> 
  filter(`Origin/Ethnicity` == "American") |> 
  select(id, year = `Release Year`, title = Title, director = Director, genre, text = Plot) |> 
  filter(year > 2010) |> 
  sample_n(size = 200)

# Number of movies in subset
movies_small |> 
  group_by(genre) |> 
  tally() 
```

## Again a Small Example with either llama or GPT

Now that the access token is set, we can again use the package `tidyllm`
to conduct text classification. Let’s adapt the code a bit more and
provide the model with more instructions on how to structure the
response. What I am trying to achieve here is that the model, next to
the code, also provides a justification for its choice.

``` r
glue("What is genre of this movie description: {description}
      
      Pick only one of the following codes from this list:
      
      Comedy, Horror, or Action.
     
      Structure your answer in this way:
     
      'Genre; Justification (but only about 10 words)'",
      description = movies_small$text[1]) |> 
  llm_message() |> 
  chat(ollama(.model = "llama3.2:1b", .temperature = 0))
  #chat(openai(.model = "gpt-3.5-turbo", .temperature = 0)) # You can use GPT as well if you dont have a local model
```

This seems to work. And the justification is nice to have. We will drop
this justification later, but for initial test, such an extra output can
be nice to have. Let’s try to classify some more movies.

# Classifying more than a couple of texts

Classifying many text via the any API requires a bit of thought:

For example, in the current OpenAI subscription, we can do 200,000
tokens per minute and 500 requests per minute and 10,000 requests per
day with `gpt-3.5-turbo`. If we would use a better model, if would have
even less. At the same time, each model has a “context window”, i.e. the
amount of input you can provide as tokens. If we use `gpt-3.5-turbo`, we
have a window of 16,385 tokens. Max output tokens is at the same time
about 4,000 tokens.

This of course heavily restricts our use and makes the procedure of
classifying many texts much longer. It thus makes sense to first look at
how many tokens movie plots usually have. We can count the number of
words and multiply by 1.6, which results roughly in the token type that
GPT uses (it breaks text down into something smaller than words).

``` r
movies_small |> 
  unnest_tokens(word, text) |> 
  group_by(title) |> 
  summarize(n = n()) |> 
  arrange(-n) |> 
  mutate(gpt_tokens = n*1.6)
```

As you can see, we have movie plots that contain about 4,000 tokens but
after the ten longest ones, most of them have less that 2,000. We
nonetheless should perhaps truncate the movie plots to only 1,000
characters which covers roughly the first 3-5 sentences, assuming that
this will be enough for GPT to figure out the genre. This way, we save
tokens without (hopefully) sacrificing performance. We then use
`gpt_split_data` to again separate our data into small chunks of 4
movies. Then, we again create a loop with the function `map()`, which
iteratively sends the prompt to the API and extracts the results. We can
should add a small delay between prompts to make sure that we do not
send to many prompts per minute (this of course makes the process
longer). Let’s start with 3 subsets of 4 movies each for now and then
see whether we can increase the number of splits that should be coded.
Bear in mind, all of you are using the same API TOKEN, so we need to be
careful not to overuse it, otherwise it won’t work for anyone and we
have to wait for minute.

``` r
# Truncate plot description
movies_small2 <- movies_small |> 
  mutate(text = stringr::str_trunc(text, 1000)) 
```

## Zero-Shot Classification

Now, we can create our task as in the last tutorial:

``` r
codebook <- glue("What is genre of this movie description: {description}
      
                 Pick only one of the following codes from this list:
                 
                 Comedy, Horror, or Action.
                
                 Structure your answer in this way:
                
                 'Genre; Justification (but only about 10 words)'",
                 description = movies_small2$text[1:20])

classification_task <- map(codebook, llm_message)

# Function to classify all 50 texts
classify_sequential_gpt <- function(texts, message){
    raw_code <- message |>
        chat(openai(.model = "gpt-3.5-turbo",   
                    .temperature = .0)) |>
        #chat(ollama(.model = "llama3.2:1b",   # Try with local model as well...
        #            .temperature = .0)) |>
        get_reply()
    tibble(text = texts, label = raw_code) 
}

# Run the classification by sequentially prompting LLama3
results_gpt <- tibble(texts = movies_small2[1:20,]$text, 
                        message = classification_task) |>  
  pmap_dfr(classify_sequential_gpt, .progress = T) 


# Combine them
predict_gpt <- results_gpt |> 
  separate(label, c("label", "justification"), sep = ";") |> 
  mutate(label = tolower(trimws(label)))
```

Let’s check out some of the justifications:

``` r
# Justifications
head(predict_gpt) |> 
  select(label, justification, text)
```

Looks good!

## Validation

As always, let’s validate our result based on the gold standard in the
data. Don’t forget to specify the class_metrics!

``` r
predict_gpt <- predict_gpt |> 
  select(label) |> 
  bind_cols(movies_small[1:20,]) |> 
  mutate(label = factor(label, levels = c("action", "comedy", "horror")),
         genre = factor(genre))

# Set performance scores
class_metrics <- metric_set(accuracy, precision, recall, f_meas)

# Performance
predict_gpt |> 
  class_metrics(truth = genre, estimate = label)
```

Pretty good performance overall already. But, can we perhaps further
improve this?

# Classifying with some examples

As discussed in the lecture, we can sometimes boost performance by
adding one (or more) example. We can achieve this by “stacking”
`llm_message` prompts. Below, I first define the task and glue the first
text to it. Then, instead of passing to immediately to the `chat()`
function, I just pass it to another `llm_message`, in which I provide
the resonse “action” (the gold standard for the first text). Now, note
that I have specified the role for this answer as “assistant”. It is
thus as if the LLM would have given that response. Then I pass this
altogether to another `llm_message`, which contains the next task (for
now just one more movie). If we then pass this to GPT, we see that the
entire “simulated” conversation becomes the message history that
influence the next response. We thus provide a “better” prompt for the
classification.

``` r
llm_message(
  glue("What is genre of this movie description: {description}
      
        Pick only one of the following codes from the following list. 
        Only provide the code, nothing else.
        
        comedy, horror, or action", 
        description = movies_small2$text[1])) |> 
  llm_message(
    "action", 
    .role = "assistant") |>  # note the specific role for this answer!
  llm_message(
  glue("Now the genre of the following movie descriptions in the same way: {description}
                    
       Again, pick only one of the following codes from the following list. 
       Only provide the code, nothing else.
           
       comedy, horror, or action", 
      description = movies_small2$text[2])) |> 
   chat(openai(.model = "gpt-3.5-turbo",   
                    .temperature = .0))
```

## One-Shot Classification

If we want to set this up for a whole classification task (i.e.,
classifying several movies), we better create the example part, then the
actual classification and then we again `map` them together.

``` r
# Example coding 
example <- glue("What is genre of this movie description: {description}
      
                Pick only one of the following codes from the following list. 
                Only provide the code, nothing else.
                    
                comedy, horror, or action", 
                description = movies_small2$text[1]) |> 
  llm_message() |> 
  llm_message("action", .role = "assistant")

# The actual task (this should be familiar from zero-shotting)
classification_tasks <- glue("Now the genre of the following movie descriptions in the same way: {description}
                             
                             Pick only one of the following codes from the following list. 
                             Only provide the code, nothing else.
                                 
                             comedy, horror, or action", 
                   description = movies_small2$text[2:20])

# Map tasks together
oneshot_tasks <- map(classification_tasks, function(x) example |> llm_message(x))


# Check
oneshot_tasks
```

As we can see, the entire history is added to each individual prompt.

``` r
# Function to classify all 50 texts
classify_sequential_gpt <- function(texts, message){
    raw_code <- message |>
        chat(openai(.model = "gpt-3.5-turbo",   
                    .temperature = .0)) |>
        #chat(ollama(.model = "llama3.2:1b",   # Try with local model as well...
        #            .temperature = .0)) |>
        get_reply()
    tibble(text = texts, label = raw_code) 
}


# Run classificatios
results_gpt_oneshot <- tibble(texts = movies_small2[2:20,]$text, 
                              message = oneshot_tasks) |>  
  pmap_dfr(classify_sequential_gpt, .progress = T)
```

## Validation

We check the performance as always

``` r
# Bin prediction to gold standard
predict_gpt_oneshot <- results_gpt_oneshot  |> 
  select(label) |> 
  mutate(label = tolower(label)) |> 
  bind_cols(movies_small2[2:20,]) |> 
  mutate(label = factor(label, levels = c("action", "comedy", "horror")),
         genre = factor(genre, levels = c("action", "comedy", "horror")))

# Performance
predict_gpt_oneshot |> 
  class_metrics(truth = genre, estimate = label)
```

And voila, we have improved the performance!

``` r
bind_rows(
  predict_gpt |> 
  class_metrics(truth = genre, estimate = label) |> 
  mutate(type = "zero-shot"),
  predict_gpt_oneshot |> 
  class_metrics(truth = genre, estimate = label) |> 
  mutate(type = "one-shot")
) |> 
  ggplot(aes(x = .metric, y = .estimate, fill = type)) +
  geom_col(position = position_dodge(width = 1)) +
  geom_text(aes(label = round(.estimate, 2)), position = position_dodge(width = 1)) +
  coord_flip() +
  ylim(0, 1) +
  theme_minimal() +
  labs(x = "", y = "Performance Score")
```

## Few-Shot Classification?

Can you extend this logic to a few-shot classification framework? This
involves stacking even more llm_message on top of each other. Also: If
you have a local model, does its performance also improve with more
examples?

``` r
# Solution
```
