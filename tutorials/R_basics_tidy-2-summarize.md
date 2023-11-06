Tidyverse II: Data Summarization with group_by, summarize, and mutate
================
Kasper Welbers, Wouter van Atteveldt & Philipp Masur

- [Introduction: Grouping and
  Summarizing](#introduction-grouping-and-summarizing)
  - [The role of grouping](#the-role-of-grouping)
- [Before we start: Pipelines and the \|\>
  symbol](#before-we-start-pipelines-and-the--symbol)
  - [Pipelines: exercise](#pipelines-exercise)
- [Summarizing data in R](#summarizing-data-in-r)
  - [Data](#data)
  - [Grouping rows](#grouping-rows)
  - [Summarization functions](#summarization-functions)
  - [Summarizing data](#summarizing-data)
- [Using mutate on grouped data](#using-mutate-on-grouped-data)
- [Computing multiple summary
  values](#computing-multiple-summary-values)
  - [Exercise: Z-scores](#exercise-z-scores)
- [Grouping by multiple columns](#grouping-by-multiple-columns)
  - [Exercise: deviations from group
    means](#exercise-deviations-from-group-means)

## Introduction: Grouping and Summarizing

The functions used in the earlier tutorial on data preparation worked on
individual rows. In many cases, you need to compute properties of groups
of rows (cases).

For example, you might want to know the average age per group of
respondents, or the total number of votes for a party based on data per
region.

This is called aggregation (or summarization) and in tidyverse uses the
`group_by` function followed by either `summarize` or `mutate`.

This tutorial will explain how these functions work and how they can be
used for data summarization.

### The role of grouping

When we summarize data, we generally want to summarize some value per
group of rows. For this reason, the process in R consists of two steps.

For this example, assume we have individual-level survey data with
gender and age per respondent:

<figure>
<img
src="https://raw.githubusercontent.com/vanatteveldt/ccslearnr/master/data/summarize1.png"
alt="Data to summarize" />
<figcaption aria-hidden="true">Data to summarize</figcaption>
</figure>

First, you define the groups by which you want to summarize. In this
case, we want two groups (for male and female), but of course in other
cases there can be many more groups. The result of *grouping* is a very
similar data frame, but in this case the groups are explicitly marked:

<figure>
<img
src="https://raw.githubusercontent.com/vanatteveldt/ccslearnr/master/data/summarize2.png"
alt="Grouped data" />
<figcaption aria-hidden="true">Grouped data</figcaption>
</figure>

Now, the second step is to compute the summary statistics per group. For
example, we could compute the average age for each group:

<figure>
<img
src="https://raw.githubusercontent.com/vanatteveldt/ccslearnr/master/data/summarize3.png"
alt="Summarized data" />
<figcaption aria-hidden="true">Summarized data</figcaption>
</figure>

In R, summarization also follows the same steps, with `group_by` used
for creating the groups (without changing the data), and `summarize`
used for computing the summary statistics and creating the resulting
frame containing the summarized data.

In the next section, we will look at these commands in detail.

## Before we start: Pipelines and the \|\> symbol

In Tidyverse, often you run many data cleaning or transformation
commands in sequence. For example, you might want to read in data,
select some columns, filter some rows, mutate a new calue, and summarize
some other values.

If we would do this with single commands, we would get a very repetitive
script. For example,

``` r
library(tidyverse)
url <- "https://raw.githubusercontent.com/fivethirtyeight/data/master/poll-quiz-guns/guns-polls.csv"
polls <- read_csv(url)
polls <- filter(polls, Question == "age-21")
polls <- select(polls, Population, Pollster, Support)
polls <- mutate(polls, Percent = Support / 100)
```

The code above runs fine, but as you can see it repeats the name `polls`
an awful lot. Because this is such a common pattern, R has introduced a
shortcut to express such a sequence, or **pipeline**, of operations: The
pipeline operator `|>`.

This operator works by chaining together multiple functions, such that
the output of the first function is the input (first argument) of the
second functoin. To see this in action, the following snippet can be
rewritten as a pipeline like this:

``` r
library(tidyverse)
url <- "https://raw.githubusercontent.com/fivethirtyeight/data/master/poll-quiz-guns/guns-polls.csv"
polls <- read_csv(url) |>
  filter(Question == "age-21") |>
  select(Population, Pollster, Support) |>
  mutate(Percent = Support / 100)
```

The whole sequence of calls is now a single statement (‘sentence’),
where the result of `read_csv` is the input of `filter`, the result of
which is fed into `select`, etc., until the output of `mutate` is
assigned to the polls variable.

This technique is very convenient to group related calls together, and
can make code more readable by making it clear that such a group of
calls is a logical unit.

The biggest **common mistake** is that you no longer supply the data to
the functions in the pipeline (except the first), so instead of using
`filter(data, ...) |> select(data, ...)` make sure you use
`filter(data, ...) |> select( ...)`. Note that you can also feed the
data into the first function, like `data |> filter(...)`.

Whether you use pipelines or not yourself is a stylistic choice, but
it’s important to be able to read them as they are often used in scripts
(including this tutorial). Note that older code might use the `%>%`
symbol, which was around longer and has the same function.

### Pipelines: exercise

Can you rewrite the calls from `read_csv` to `mutate` into a single
pipeline?

``` r
library(tidyverse)
url <- "https://raw.githubusercontent.com/fivethirtyeight/data/master/poll-quiz-guns/guns-polls.csv"
gunpolls <- read_csv(url)
gunpolls <- select(gunpolls, Question, Support, 
                   rep=`Republican Support`, dem=`Democratic Support`)
gunpolls <- mutate(gunpolls, diff = rep - dem)
head(gunpolls)
```

    ## # A tibble: 6 × 5
    ##   Question Support   rep   dem  diff
    ##   <chr>      <dbl> <dbl> <dbl> <dbl>
    ## 1 age-21        72    61    86   -25
    ## 2 age-21        82    72    92   -20
    ## 3 age-21        67    59    76   -17
    ## 4 age-21        84    77    92   -15
    ## 5 age-21        78    63    93   -30
    ## 6 age-21        72    65    80   -15

**Solution:**

``` r
library(tidyverse)
url <- "https://raw.githubusercontent.com/fivethirtyeight/data/master/poll-quiz-guns/guns-polls.csv"
gunpolls <- read_csv(url) |>
 select(Question, Support, 
        rep=`Republican Support`, dem=`Democratic Support`) |>
  mutate(diff = rep - dem) 
head(gunpolls)
```

    ## # A tibble: 6 × 5
    ##   Question Support   rep   dem  diff
    ##   <chr>      <dbl> <dbl> <dbl> <dbl>
    ## 1 age-21        72    61    86   -25
    ## 2 age-21        82    72    92   -20
    ## 3 age-21        67    59    76   -17
    ## 4 age-21        84    77    92   -15
    ## 5 age-21        78    63    93   -30
    ## 6 age-21        72    65    80   -15

## Summarizing data in R

Let’s look at how we can we summarize data in R:

### Data

First, let’s fire up tidyverse and load the gun polls data used in the
earlier example:

``` r
library(tidyverse)
url <- "https://raw.githubusercontent.com/fivethirtyeight/data/master/poll-quiz-guns/guns-polls.csv"
polls <- read_csv(url) |>
  select(Question, Population, Pollster, Support)
polls
```

    # A tibble: 57 × 5
       Question     Pollster           Support   Rep   Dem
       <chr>        <chr>                <dbl> <dbl> <dbl>
     1 age-21       CNN/SSRS                72    61    86
     2 age-21       NPR/Ipsos               82    72    92
     3 age-21       Rasmussen               67    59    76

### Grouping rows

Now, we can use the group_by function to group by, for example,
Question:

``` r
polls |> group_by(Question)
```

    # A tibble: 57 × 5
    # Groups:   Question [8]
       Question     Pollster           Support   Rep   Dem
       <chr>        <chr>                <dbl> <dbl> <dbl>
     1 age-21       CNN/SSRS                72    61    86
     2 age-21       NPR/Ipsos               82    72    92
     3 age-21       Rasmussen               67    59    76
     4 age-21       Harris Interactive      84    77    92
     5 age-21       Quinnipiac              78    63    93
     [...]

As you can see, the data itself didn’t actually change yet, it merely
recorded (at the top) that we are now grouping by Question, and that
there are 8 groups (different questions) in total.

<div class="Info">

The grouping information created by `group_by` is recorded on the data
frame, and will stay intact until you `group_by` on a different
variable. You can also remove grouping information using the `ungroup()`
function.

</div>

### Summarization functions

An important consideration here is that the function used should be a
*summarization function*, that is, a function that summarizes a group of
values to a single value. An example summarization function is `mean`,
which computes the mean (average) of a group of values:

``` r
values = c(1,2,3)
mean(values)
```

    ## [1] 2

Other useful functions are for example `sum`, `max` and `min`.

Can you change the code above to compute the `sum` rather than the mean
of values?

### Summarizing data

Now, we are ready to use `summarize` to create the summarized data
frame.

This function is similar to `mutate` in the sense that we specify how to
compute the new columns using `name=function(column)` arguments.

``` r
polls |> group_by(Question) |> summarize(Support=mean(Support))
```

    ## # A tibble: 8 × 2
    ##   Question                    Support
    ##   <chr>                         <dbl>
    ## 1 age-21                         75.9
    ## 2 arm-teachers                   42  
    ## 3 background-checks              87.4
    ## 4 ban-assault-weapons            61.8
    ## 5 ban-high-capacity-magazines    67.3
    ## 6 mental-health-own-gun          85.8
    ## 7 repeal-2nd-amendment           10  
    ## 8 stricter-gun-laws              66.5

**Solution:**

``` r
polls |> group_by(Pollster) |> summarize(Support=max(Support))
```

    ## # A tibble: 14 × 2
    ##    Pollster            Support
    ##    <chr>                 <dbl>
    ##  1 ABC/Washington Post      51
    ##  2 CBS News                 75
    ##  3 CNN/SSRS                 72
    ##  4 Harris Interactive       84
    ##  5 Harvard/Harris           61
    ##  6 Marist                   73
    ##  7 Morning Consult          90
    ##  8 NPR/Ipsos                94
    ##  9 Quinnipiac               97
    ## 10 Rasmussen                67
    ## 11 Suffolk                  76
    ## 12 SurveyMonkey             43
    ## 13 YouGov                   87
    ## 14 YouGov/Huffpost          41

**Exercise**: Alter the code above to compute the highest level of
support (i.e. max support) per pollster

## Using mutate on grouped data

The previous section used `group_by(col1) |> summarize(name=fn(col2))`.
As you could see, this creates a new and often much smaller ‘summarized’
data frame, with one row per unique group and only keeps the grouping
column(s) and the calculated column(s).

Instead of `summarize`, you can also run `mutate` with summarization
functions on grouped data. In that case, the summary is computer per
group, but instead of replacing the data frame with only the summarized
data, the new column is added to the existing data frame, with the
values repeated within a group:

``` r
polls |> group_by(Question) |> mutate(Mean_Support=mean(Support))
```

    ## # A tibble: 57 × 5
    ## # Groups:   Question [8]
    ##    Question     Population        Pollster           Support Mean_Support
    ##    <chr>        <chr>             <chr>                <dbl>        <dbl>
    ##  1 age-21       Registered Voters CNN/SSRS                72         75.9
    ##  2 age-21       Adults            NPR/Ipsos               82         75.9
    ##  3 age-21       Adults            Rasmussen               67         75.9
    ##  4 age-21       Registered Voters Harris Interactive      84         75.9
    ##  5 age-21       Registered Voters Quinnipiac              78         75.9
    ##  6 age-21       Registered Voters YouGov                  72         75.9
    ##  7 age-21       Registered Voters Morning Consult         76         75.9
    ##  8 arm-teachers Registered Voters YouGov/Huffpost         41         42  
    ##  9 arm-teachers Adults            CBS News                44         42  
    ## 10 arm-teachers Adults            Rasmussen               43         42  
    ## # ℹ 47 more rows

This can be very useful to compute the relation between single values
and e.g. the group averages.

**Exercise:** Can you create a new column `difference`, that shows to
what extend an individual poll is higher or lower than the group
average? E.g. for the first row, the result would be (approximately)
`72 - 75.9 = -3.9`.

``` r
polls |> group_by(Question) |> mutate(Mean_Support=mean(Support), difference=____)
```

**Solution:**

``` r
polls |> group_by(Question) |> mutate(Mean_Support=mean(Support), difference=Support - Mean_Support)
```

    ## # A tibble: 57 × 6
    ## # Groups:   Question [8]
    ##    Question     Population        Pollster       Support Mean_Support difference
    ##    <chr>        <chr>             <chr>            <dbl>        <dbl>      <dbl>
    ##  1 age-21       Registered Voters CNN/SSRS            72         75.9     -3.86 
    ##  2 age-21       Adults            NPR/Ipsos           82         75.9      6.14 
    ##  3 age-21       Adults            Rasmussen           67         75.9     -8.86 
    ##  4 age-21       Registered Voters Harris Intera…      84         75.9      8.14 
    ##  5 age-21       Registered Voters Quinnipiac          78         75.9      2.14 
    ##  6 age-21       Registered Voters YouGov              72         75.9     -3.86 
    ##  7 age-21       Registered Voters Morning Consu…      76         75.9      0.143
    ##  8 arm-teachers Registered Voters YouGov/Huffpo…      41         42       -1    
    ##  9 arm-teachers Adults            CBS News            44         42        2    
    ## 10 arm-teachers Adults            Rasmussen           43         42        1    
    ## # ℹ 47 more rows

<div class="Info">

In `mutate`, if you compute two variables, you can already use the first
value in the second computation: R will do the computations one by one.

</div>

## Computing multiple summary values

The previous examples all grouped by a single variable and then computed
a single summary value. It is also possible to compute multiple values
in a single call.

The snippet below shows how you can compute the mean and standard
deviation (`sd`) of the Support per question. It also uses the function
`n()` (without argument) to compute the number of cases per question.

``` r
polls |> 
  group_by(Question) |> 
  summarize(mean=mean(Support), sd=sd(Support), n=n())
```

    ## # A tibble: 8 × 4
    ##   Question                     mean    sd     n
    ##   <chr>                       <dbl> <dbl> <int>
    ## 1 age-21                       75.9  6.01     7
    ## 2 arm-teachers                 42    1.55     6
    ## 3 background-checks            87.4  7.32     7
    ## 4 ban-assault-weapons          61.8  6.44    12
    ## 5 ban-high-capacity-magazines  67.3  3.86     7
    ## 6 mental-health-own-gun        85.8  5.46     6
    ## 7 repeal-2nd-amendment         10   NA        1
    ## 8 stricter-gun-laws            66.5  5.15    11

### Exercise: Z-scores

In statistics, the z-score or normalized value of a variable is it’s
number of standard deviations from the mean. For example, In the poll
from CNN support from age-21 was 72%. As you can see above, the mean
support for that question was about 76%, and the standard deviation was
about 6 percentage points. So, the CNN polls was about 0.67 standard
deviations lower than the average: it’s z-score is -0.67.

Can you compute the z-score for all polls? To do this, you can start
from the code above but use mutate rather than summarize to compute the
mean and standard deviation per question. Then, you compute the new
column by comparing the Support for that question to the computed mean.

``` r
polls |>
  group_by(Question) |>
  mutate(mean=mean(Support),  sd=sd(Support))
```

    ## # A tibble: 57 × 6
    ## # Groups:   Question [8]
    ##    Question     Population        Pollster           Support  mean    sd
    ##    <chr>        <chr>             <chr>                <dbl> <dbl> <dbl>
    ##  1 age-21       Registered Voters CNN/SSRS                72  75.9  6.01
    ##  2 age-21       Adults            NPR/Ipsos               82  75.9  6.01
    ##  3 age-21       Adults            Rasmussen               67  75.9  6.01
    ##  4 age-21       Registered Voters Harris Interactive      84  75.9  6.01
    ##  5 age-21       Registered Voters Quinnipiac              78  75.9  6.01
    ##  6 age-21       Registered Voters YouGov                  72  75.9  6.01
    ##  7 age-21       Registered Voters Morning Consult         76  75.9  6.01
    ##  8 arm-teachers Registered Voters YouGov/Huffpost         41  42    1.55
    ##  9 arm-teachers Adults            CBS News                44  42    1.55
    ## 10 arm-teachers Adults            Rasmussen               43  42    1.55
    ## # ℹ 47 more rows

**Solution:**

``` r
polls |>
  group_by(Question) |>
  mutate(mean=mean(Support),  sd=sd(Support)) |>
  mutate(zscore = (Support - mean) / sd)
```

    ## # A tibble: 57 × 7
    ## # Groups:   Question [8]
    ##    Question     Population        Pollster           Support  mean    sd  zscore
    ##    <chr>        <chr>             <chr>                <dbl> <dbl> <dbl>   <dbl>
    ##  1 age-21       Registered Voters CNN/SSRS                72  75.9  6.01 -0.642 
    ##  2 age-21       Adults            NPR/Ipsos               82  75.9  6.01  1.02  
    ##  3 age-21       Adults            Rasmussen               67  75.9  6.01 -1.47  
    ##  4 age-21       Registered Voters Harris Interactive      84  75.9  6.01  1.35  
    ##  5 age-21       Registered Voters Quinnipiac              78  75.9  6.01  0.356 
    ##  6 age-21       Registered Voters YouGov                  72  75.9  6.01 -0.642 
    ##  7 age-21       Registered Voters Morning Consult         76  75.9  6.01  0.0238
    ##  8 arm-teachers Registered Voters YouGov/Huffpost         41  42    1.55 -0.645 
    ##  9 arm-teachers Adults            CBS News                44  42    1.55  1.29  
    ## 10 arm-teachers Adults            Rasmussen               43  42    1.55  0.645 
    ## # ℹ 47 more rows

You can first just run the snippet above to see how the data looks.
Then, add a second `mutate` command to compute a new column called
`zscore`.

(Note that you can also include the new computation in the first mutate
call if desired, but it is a bit more readable to create a new mutate to
differentiate the group level and individual level computations.)\`  

## Grouping by multiple columns

In the `group_by` function, you can also specify multiple columns. For
example, the code below computes the average by question and by
populuation category.

``` r
polls |> group_by(Question, Population) |> summarize(Support=mean(Support))
```

    `summarise()` has grouped output by 'Question'. You can override using the `.groups` argument.
    # A tibble: 15 × 3
    # Groups:   Question [8]
       Question                    Population        Support
       <chr>                       <chr>               <dbl>
     1 age-21                      Adults               74.5
     2 age-21                      Registered Voters    76.4
     3 arm-teachers                Adults               42.7
     4 arm-teachers                Registered Voters    41.3
     [...]

As you can see, the result is a data set with one row per unique group,
that is, ther question - population combination. In the message above
the output, and in the output itself, you can also see that the result
it itself grouped by Question. When summarizing data with multiple
grouping value, R by default removes the last grouping column, so
instead of Question and Population it’s now just grouped by Question.

This behaviour can be quite useful if you want to compute new statistics
for the remaining groups. Note that you can remove this grouping either
by either adding `.groups=drop', or by calling the function`\|\>
ungroup()\` on the result.

### Exercise: deviations from group means

Starting from the code above, and taking into account that after the
summarize function the data is grouped by question, can you calculate
the deviation from the average support per population within each
question? So, the adults are `-0.95` point away from the average support
for age-21: the average support for age-21 is `75.4`, and
`74.5 - 75.4 = -0.95`.

``` r
polls |> 
  group_by(Question, Population) |> 
  summarize(Support=mean(Support)) |> 
  mutate(mean_support=____, deviation=____)
```

**Solution:**

``` r
polls |> 
  group_by(Question, Population) |> 
  summarize(Support=mean(Support)) |> 
  mutate(mean_support=mean(Support), deviation=Support - mean_support)
```

    ## # A tibble: 15 × 5
    ## # Groups:   Question [8]
    ##    Question                    Population        Support mean_support deviation
    ##    <chr>                       <chr>               <dbl>        <dbl>     <dbl>
    ##  1 age-21                      Adults               74.5         75.4    -0.950
    ##  2 age-21                      Registered Voters    76.4         75.4     0.950
    ##  3 arm-teachers                Adults               42.7         42       0.667
    ##  4 arm-teachers                Registered Voters    41.3         42      -0.667
    ##  5 background-checks           Adults               84.5         86.6    -2.05 
    ##  6 background-checks           Registered Voters    88.6         86.6     2.05 
    ##  7 ban-assault-weapons         Adults               62.5         62.0     0.450
    ##  8 ban-assault-weapons         Registered Voters    61.6         62.0    -0.450
    ##  9 ban-high-capacity-magazines Adults               73           69.7     3.33 
    ## 10 ban-high-capacity-magazines Registered Voters    66.3         69.7    -3.33 
    ## 11 mental-health-own-gun       Adults               92           88.3     3.70 
    ## 12 mental-health-own-gun       Registered Voters    84.6         88.3    -3.70 
    ## 13 repeal-2nd-amendment        Registered Voters    10           10       0    
    ## 14 stricter-gun-laws           Adults               70           67.8     2.17 
    ## 15 stricter-gun-laws           Registered Voters    65.7         67.8    -2.17
