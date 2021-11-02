R Tidyverse: Data Transformation & Summarization
================
Kasper Welbers, Wouter van Atteveldt & Philipp Masur
2021-10

-   [Introduction](#introduction)
    -   [Installing tidyverse](#installing-tidyverse)
-   [Tidyverse basics](#tidyverse-basics)
    -   [Reading data](#reading-data)
    -   [Subsetting with filter()](#subsetting-with-filter)
    -   [Selecting certain columns](#selecting-certain-columns)
    -   [Sorting with arrange()](#sorting-with-arrange)
    -   [Adding or transforming variables with
        mutate()](#adding-or-transforming-variables-with-mutate)
-   [Working with Pipes](#working-with-pipes)
-   [Data Summarization](#data-summarization)
    -   [Grouping rows](#grouping-rows)
    -   [Summarizing](#summarizing)
    -   [Using mutate with group\_by](#using-mutate-with-group_by)
    -   [Ungrouping](#ungrouping)
-   [Multiple grouping variables](#multiple-grouping-variables)

# Introduction

The goal of this practical session is to get you acquainted with the
[Tidyverse](https://www.tidyverse.org/). Tidyverse is a collection of
packages that have been designed around a singular and clearly defined
set of principles about what data should look like and how we should
work with it. It comes with a nice introduction in the [R for Data
Science](http://r4ds.had.co.nz/) book, for which the digital version is
available for free. This tutorial deals with most of the material in
chapter 5 of that book.

In this part of the tutorial, we’ll focus on working with data using the
`tidyverse` package. This package includes the `dplyr` (data-pliers)
packages, which contains most of the tools we’re using below, but it
also contains functions for reading, analysing and visualising data that
will be explained later.

## Installing tidyverse

As before, `install.packages()` is used to download and install the
package (you only need to do this once on your computer) and `library()`
is used to make the functions from this package available for use
(required each session that you use the package).

``` r
install.packages('tidyverse') # only needed once
```

``` r
library(tidyverse)
```

Note: don’t be scared if you see a red message after calling `library`.
RStudio doesn’t see the difference between messages, warnings, and
errors, so it displays all three in red. You need to read the message,
and it will contain the word ‘error’ if there is an error, such as a
misspelled package:

``` r
library(tidyvers) # this will cause an error!
```

# Tidyverse basics

As in most packages, the functionality in dplyr is offered through
functions. In general, a function can be seen as a command or
instruction to the computer to do something and (generally) return the
result. In the tidverse package `dplyr`, almost all `functions`
primarily operate on data sets, for example for filtering and sorting
data.

With a data set we mean a rectangular data frame consisting of rows
(often items or respondents) and columns (often measurements of or data
about these items). These data sets can be R `data.frames`, but
tidyverse has its own version of data frames called `tibble`, which is
functionally (almost) equivalent to a data frame but is more efficient
and somewhat easier to use.

As a very simply example, the following code creates a tibble containing
respondents, their gender, and their height:

``` r
data <- tibble(resp = c(1,2,3), 
               gender = c("M","F","F"), 
               height = c(176, 165, 172))
data
```

**Exercise 1:** How would you create a subset that contains only female
participants?

``` r
# Code here
```

## Reading data

The example above manually created a data set, but in most cases you
will start with data that you get from elsewhere, such as a csv file
(e.g. downloaded from an online dataset or exported from excel) or an
SPSS or Stata data file.

Tidyverse contains a function `read_csv` that allows you to read a csv
file directly into a tibble. You specify the location of the file,
either on your local drive (as we did in the last practical session) or
directly from the Internet!

The example below downloads an overview of gun polls from the [data
analytics site 538](https://fivethirtyeight.com/), and reads it into a
tibble using the read\_csv function:

``` r
url <- "https://raw.githubusercontent.com/fivethirtyeight/data/master/poll-quiz-guns/guns-polls.csv"
d <- read_csv(url)
d
```

(Note that you can safely ignore the (red) message, they simply tell you
how each column was parsed)

The shows the first ten rows of the data set, and if the columns don’t
fit they are not printed. The remaining rows and columns are printed at
the bottom. For each column the data type is also mentioned (<int>
stands for integer, which is a *numeric* value; <chr> is textual or
*character* data). If you want to browse through your data, you can also
click on the name of the data.frame (d) in the top-right window
“Environment” tab or call `View(d)`.

## Subsetting with filter()

The `filter` function can be used to select a subset of rows. In the
guns data, the `Question` column specifies which question was asked. We
can select only those rows (polls) that asked whether the minimum
purchage age for guns should be raised to 21:

``` r
age21 <- filter(d, Question == 'age-21')
age21
```

This call is typical for a tidyverse function: the first argument is the
data to be used (`d`), and the remaining argument(s) contain information
on what should be done to the data.

Note the use of `==` for comparison: In R, `=` means assingment and `==`
means equals. Other comparisons are e.g. `>` (greather than), `<=` (less
than or equal) and `!=` (not equal). You can also combine multiple
conditions with logical (boolean) operators: `&` (and), `|` or, and `!`
(not), and you can use parentheses like in mathematics.

So, we can find all surveys where support for raising the gun age was at
least 80%:

``` r
filter(d, Question == 'age-21' & Support >= 80)
```

Note that this command did not assign the result to an object, so the
result is only displayed on the screen but not remembered. This can be a
great way to quickly inspect your data, but if you want to continue
analysing this subset you need to assign it to an object as above.

## Selecting certain columns

Where `filter` selects specific rows, `select` allows you to select
specific columns. Most simply, we can simply name the columns that we
want to retrieve them in that particular order.

``` r
select(age21, Population, Support, Pollster)
```

You can also specify a range of columns, for example all columns from
Support to Democratic Support:

``` r
select(age21, Support:`Democratic Support`)
```

Note the use of ‘backticks’ (reverse quotes) to specify the column name,
as R does not normally allow spaces in names.

Select can also be used to rename columns when selecting them, for
example to get rid of the spaces:

``` r
select(age21, Pollster, rep = `Republican Support`, dem = `Democratic Support`)
```

Note that `select` drops all columns not selected. If you only want to
rename columns, you can use the `rename` function:

``` r
rename(age21, start_date = Start, end_date = End)
```

Finally, you can drop a variable by adding a minus sign in front of a
name:

``` r
select(age21, -Question, -URL)
```

## Sorting with arrange()

You can easily sort a data set with `arrange`: you first specify the
data, and then the column(s) to sort on. To sort in descending order,
put a minus in front of a variable. For example, the following orders by
population and then by support (descending):

``` r
age21 <- arrange(age21, Population, -Support)
age21
```

Note that I assigned the result of arranging to the `age21` object
again, i.e. I replace the object by its sorted version. If I wouldn’t
assign it to anything, it would display it on screen but not remember
the sorting. Assigning a result to the same name means I don’t create a
new object, preventing the environment from being cluttered (and saving
me from the bother of thinking up yet another object name). For sorting,
this should generally be fine as the sorted data should contain the same
data as before. For subsetting, this means that the rows or columns are
actually deleted from the dataset (in memory), so you will have to read
the file again (or start from an earlier object) if you need those rows
or columns later.

## Adding or transforming variables with mutate()

The `mutate` function makes it easy to create new variables or to modify
existing ones. For those more familiar with SPSS, this is what you would
do with compute and recode.

If you look at the documentation page, you see that mutate works
similarly to `filter()` and `select()`, in the sense that the first
argument is the *tibble*, and then any number of additional arguments
can be given to perform mutations. The mutations themselves are named
arguments, in which you can provide any calculations using the existing
columns.

Here we’ll first create some variables and then look at the variables
(using the `select` function to focus on the changes). Specifically,
we’ll make a column for the absolute difference between the support
scores for republicans and democrats, as a measure of how much they
disagree.

``` r
age21 <- mutate(age21, party_diff = abs(`Republican Support` - `Democratic Support`))
select(age21, Question, Pollster, party_diff)
age21 <- arrange(age21, Population, -Support)
```

To transform (recode) a variable in the same column, you can simply use
an existing name in `mutate()` to overwrite it.

# Working with Pipes

If you look at the code above, you notice that the result of each
function is stored as an object, and that this object is used as the
first argument for the next function. Moreover, we don’t really care
about this temporary object, we only care about the final summary table.

This is a very common usage pattern, and it can be seen as a *pipeline*
of functions, where the output of each function is the input for the
next function. Because this is so common, tidyverse offers a more
convenient way of writing the code above using the pipeline operator
`%>%`. In sort, whenever you write `f(a, x)` you can replace it by
`a %>% f(x)`. If you then want to use the output of `f(a, x)` for a
second function, you can just add it to the pipe: `a %>% f(x) %>% f2(y)`
is equivalent to `f2(f(a,x), y)`, or more readable, `b=f(a,x); f2(b, y)`

Put simply, pipes take the output of a function, and directly use that
output as the input for the `.data` argument in the next function. As
you have seen, all the `dplyr` functions that we discussed have in
common that the first argument is a *tibble*, and all functions return a
*tibble*. This is intentional, and allows us to pipe all the functions
together.

This seems a bit abstract, but consider the code below, which is a
collection of statements from above:

``` r
d <- read_csv(url)
age21 <- filter(d, Question == 'age-21')
age21 <- mutate(age21, party_diff = abs(`Republican Support` - `Democratic Support`))
age21 <- select(age21, Question, Pollster, party_diff)
arrange(age21, -party_diff)
```

To recap, this reads the csv, filters by question, computes the
difference, drops other variables, and sorts. Since the output of each
function is the input of the next, we can also write this as a single
pipeline:

``` r
read_csv(url) %>% 
  filter(Question == 'age-21') %>% 
  mutate(party_diff = abs(`Republican Support` - `Democratic Support`)) %>%
  select(Question, Pollster, party_diff) %>% 
  arrange(-party_diff)
```

The nice thing about pipes is that it makes it really clear what you are
doing. Also, it doesn’t require making many intermediate objects (such
as `ds`). If applied right, piping allows you to make nicely contained
pieces of code to perform specific parts of your analysis from raw input
straight to results, including statistical modeling or visualization. It
usually makes sense to have each “step” in the pipeline in its own line.
This way, we can easily read the code line by line

Of course, you probably don’t want to replace your whole script with a
single pipe, and often it is nice to store intermediate values. For
example, you might want to download, clean, and subset a data set before
doing multiple analyses with it. In that case, you probably want to
store the result of downloading, cleaning, and subsetting as a variable,
and use that in your analyses.

**Exercise 2:** Create a subset of the tibble `d` in which only polls
with the question “arm-teacher” are included. Select only the variables
*Pollster*, *Population*, and *Support* (feel free to rename them to
more shorter abbreviations at the same time). Recode the variable
Support so that it ranges from 0 to 1. Recode the variable *Population*
so that the values reader *reg* and *adu* (tip: use the function
`recode()` within a `mutate()` command. If you don’t know how the
`recode` function works, use the help function `?`!)

``` r
# Code here
```

# Data Summarization

The functions used in the earlier part on data preparation worked on
individual rows. Sometimes, you need to compute properties of groups of
rows (cases). This is called aggregation (or summarization) and in
tidyverse uses the `group_by` function followed by either `summarize` or
`mutate`.

Let’s again work with the gun-poll data, remove the URL and rename some
variables.

``` r
d <- d %>% 
  select(-URL) %>% 
  rename(Rep = `Republican Support`, Dem = `Democratic Support`)
d
```

## Grouping rows

Now, we can use the group\_by function to group by, for example,
pollster:

``` r
d %>% 
  group_by(Question)
```

As you can see, the data itself didn’t actually change yet, it merely
recorded (at the top) that we are now grouping by Question, and that
there are 8 groups (different questions) in total.

## Summarizing

To summarize, you follow the group\_by with a call to `summarize`.
Summarize has a syntax that is similar to mutate:
`summarize(column = calculation, ...)`. The crucial difference, however,
is that you always need to use a function in the calculation, and that
function needs to compute a single summary value given a vector of
values. Very common summarization functions are sum, mean, and sd
(standard deviation).

For example, the following computes the average support per question
(and sorts by descending support):

``` r
d %>% 
  group_by(Question) %>%                    ## group by "Questions"
  summarize(Support = mean(Support)) %>%    ## average "Support" per group
  arrange(-Support)                         ## sort based on "Support"
```

As you can see, summarize drastically changes the shape of the data.
There are now rows equal to the number of groups (8), and the only
columns left are the grouping variables and the summarized values.

You can also compute summaries of multiple values, and even do ad hoc
calculations:

``` r
d %>% 
  group_by(Question) %>% 
  summarize(Dem = mean(Dem), 
            Rep = mean(Rep), 
            Diff = mean(Dem-Rep)) %>% 
  arrange(-Diff)
```

So, Democrats are more in favor of all proposed gun laws except arming
teachers.

You can also compute multiple summaries of a single value. Another
useful function is `n()` (without arguments), which simply counts the
values in each group. For example, the following gives the count, mean,
and standard deviation of the support:

``` r
d %>% 
  group_by(Question) %>% 
  summarize(n = n(),
            mean = mean(Support), 
            sd = sd(Support))
```

Note: As you can see, one of the values has a missing value (NA) for
standard deviation. Why?

## Using mutate with group\_by

The examples above all reduce the number of cases to the number of
groups. Another option is to use mutate after a group\_by, which allows
you to add summary values to the rows themselves.

For example, suppose we wish to see whether a certain poll has a
different prediction from the average polling of that question. We can
group\_by question and then use mutate to calculate the average support:

``` r
d2 <- d %>% 
  group_by(Question) %>%
  mutate(avg_support = mean(Support), 
         diff = Support - avg_support)
d2
```

As you can see, where summarize reduces the rows and columns to the
groups and summaries, mutate adds a new column which is identical for
all rows within a group.

## Ungrouping

Finally, you can use `ungroup` to get rid of any groupings.

For example, the data produced by the example above is still grouped by
Question as mutate does not remove grouping information. So, if we want
to compute the overall standard deviation of the difference we could
ungroup and then summarize:

``` r
d2 %>% 
  ungroup() %>% 
  summarize(diff = sd(diff))
```

(of course, running `sd(d2$diff))` would yield the same result.)

If you run the same command without the ungroup, what would the result
be? Why?

# Multiple grouping variables

The above examples all used a single grouping variable, but you can also
group by multiple columns. For example, I could compute average support
per question and per population:

``` r
d %>% 
  group_by(Question, Population) %>% 
  summarize(Support = mean(Support))
```

This results in a data set with one row per unique group,
i.e. combination of Question and Population, and with separate columns
for each grouping column and the summary values.

*Exercise 2:* The following data set stems from an experiment conducted
by [Masur, DiFranzo and Bazarova
(2021)](https://doi.org/10.1371/journal.pone.0254670) in which
participant were exposed to social media feeds that differed with regard
to how many of the (simulated) users showed themselves visually in the
posts and profile pictures. The experiement a 3 (norm: 0%, 20%, 80% of
the posts contained faces) x 3 (profile: 0%, 20%, 80% of the profile
pictures contained faces) between-subject design. The authors were
interested whether these manipulations led to different norm perceptions
and different intentions to disclose onself. The full data set can be
downloaded here: <https://osf.io/kfm6x/>. This subset contains the
following variables:

-   **id**: participants’ unique identifiers
-   **condition**: the condition they were randomly assigned.
-   **norm**: The first manipulated factor, i.e., number of posts that
    contained faces.
-   **profile**: The second manipulated factor, i.e., the number of
    profile pictures that contained faces
-   **age**: Age of the participants -**gender**: Gender of the
    participants -**norm\_perc**: A scale measuring how strongly
    participants perceived the social norm to disclose oneself on the
    platform. -**disclosure**: Their intention to disclose themselves on
    the platform

Using the functions of the tidyverse, try to answer the following
questions:

1.  On average, does age differ across the nine conditions?

2.  How did the two manipulations (*norm* and *profile*) affect
    subsequent *norm perceptions* and *disclosure intentions*? Think
    about how you can compute the difference across conditions for both
    factors independently as well as at the same time.

3.  Bonus question: How strongly are norm perceptions and intentions
    correlated (tip: try `?cor.test` to learn how to compute
    correlations)?

``` r
d <- read_csv("https://raw.githubusercontent.com/masurp/VU_CADC/main/tutorials/data/masur-et-al_data.csv")

# Solution here
```
