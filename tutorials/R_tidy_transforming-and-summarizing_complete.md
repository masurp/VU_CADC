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

| resp | gender | height |
|-----:|:-------|-------:|
|    1 | M      |    176 |
|    2 | F      |    165 |
|    3 | F      |    172 |

**Exercise 1:** How would you create a subset that contains only female
participants?

``` r
# Alternative 1: Selecting the right rows and columns based on position (only doable if you know all positions!)
females <- data[2:3,]
females
```

| resp | gender | height |
|-----:|:-------|-------:|
|    2 | F      |    165 |
|    3 | F      |    172 |

``` r
# Alternative 2: Rule-based selection
females <- data[data$gender == "F", ]  
females
```

| resp | gender | height |
|-----:|:-------|-------:|
|    2 | F      |    165 |
|    3 | F      |    172 |

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

| Question                    | Start   | End     | Pollster            | Population        | Support | Republican Support | Democratic Support | URL                                                                                                                                                                  |
|:----------------------------|:--------|:--------|:--------------------|:------------------|--------:|-------------------:|-------------------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| age-21                      | 2/20/18 | 2/23/18 | CNN/SSRS            | Registered Voters |      72 |                 61 |                 86 | <http://cdn.cnn.com/cnn/2018/images/02/25/rel3a.-.trump>,.guns.pdf                                                                                                   |
| age-21                      | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      82 |                 72 |                 92 | <https://www.ipsos.com/en-us/npripsos-poll-majority-americans-support-policies-aimed-keep-guns-out-hands-dangerous-individuals>                                      |
| age-21                      | 3/1/18  | 3/4/18  | Rasmussen           | Adults            |      67 |                 59 |                 76 | <http://www.rasmussenreports.com/public_content/lifestyle/general_lifestyle/march_2018/americans_favor_raising_age_to_buy_gun_not_enlist_or_vote>                    |
| age-21                      | 2/22/18 | 2/26/18 | Harris Interactive  | Registered Voters |      84 |                 77 |                 92 | <http://thehill.com/opinion/civil-rights/375993-americans-support-no-gun-under-21>                                                                                   |
| age-21                      | 3/3/18  | 3/5/18  | Quinnipiac          | Registered Voters |      78 |                 63 |                 93 | <https://poll.qu.edu/national/release-detail?ReleaseID=2525>                                                                                                         |
| age-21                      | 3/4/18  | 3/6/18  | YouGov              | Registered Voters |      72 |                 65 |                 80 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/zq33h2ipcl/econTabReport.pdf>                                                                        |
| age-21                      | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      76 |                 72 |                 86 | <https://morningconsult.com/wp-content/uploads/2018/03/180303_crosstabs_POLITICO_v1_DK-1.pdf>                                                                        |
| arm-teachers                | 2/23/18 | 2/25/18 | YouGov/Huffpost     | Registered Voters |      41 |                 69 |                 20 | <http://big.assets.huffingtonpost.com/tabsHPSafeschools20180223.pdf>                                                                                                 |
| arm-teachers                | 2/20/18 | 2/23/18 | CBS News            | Adults            |      44 |                 68 |                 20 | <https://drive.google.com/file/d/0ByVu4fDHYJgVbjh5OW1Tb3FRdWxJQUpJSUdHUXdma1RKU0tV/view>                                                                             |
| arm-teachers                | 2/27/18 | 2/28/18 | Rasmussen           | Adults            |      43 |                 71 |                 24 | <http://www.rasmussenreports.com/public_content/politics/current_events/gun_control/most_adults_with_school_aged_kids_support_arming_teachers>                       |
| arm-teachers                | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      41 |                 68 |                 18 | <https://www.ipsos.com/en-us/npripsos-poll-majority-americans-support-policies-aimed-keep-guns-out-hands-dangerous-individuals>                                      |
| arm-teachers                | 3/3/18  | 3/5/18  | Quinnipiac          | Registered Voters |      40 |                 77 |                 10 | <https://poll.qu.edu/national/release-detail?ReleaseID=2525>                                                                                                         |
| arm-teachers                | 2/26/18 | 2/28/18 | SurveyMonkey        | Registered Voters |      43 |                 80 |                 11 | <https://www.nbcnews.com/politics/politics-news/poll-majority-americans-disagree-trump-s-proposal-arm-teachers-n854596>                                              |
| background-checks           | 2/16/18 | 2/19/18 | Quinnipiac          | Registered Voters |      97 |                 97 |                 99 | <https://poll.qu.edu/images/polling/us/us02202018_ugbw51.pdf/>                                                                                                       |
| background-checks           | 2/18/18 | 2/20/18 | YouGov              | Registered Voters |      85 |                 78 |                 87 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/57yxejgrmo/econTabReport.pdf>                                                                        |
| background-checks           | 2/25/18 | 2/27/18 | YouGov              | Registered Voters |      83 |                 76 |                 91 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/h8n9gvrqyj/econTabReport.pdf>                                                                        |
| background-checks           | 2/15/18 | 2/19/18 | Morning Consult     | Registered Voters |      88 |                 88 |                 90 | <https://morningconsult.com/wp-content/uploads/2018/02/180211_crosstabs_POLITICO_v1_DK-1-1-1.pdf>                                                                    |
| background-checks           | 2/20/18 | 2/23/18 | CBS News            | Adults            |      75 |                 66 |                 86 | <https://drive.google.com/file/d/0ByVu4fDHYJgVbjh5OW1Tb3FRdWxJQUpJSUdHUXdma1RKU0tV/view>                                                                             |
| background-checks           | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      94 |                 89 |                 99 | <https://www.ipsos.com/en-us/npripsos-poll-majority-americans-support-policies-aimed-keep-guns-out-hands-dangerous-individuals>                                      |
| background-checks           | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      90 |                 91 |                 91 | <https://morningconsult.com/wp-content/uploads/2018/03/180303_crosstabs_POLITICO_v1_DK-1.pdf>                                                                        |
| ban-assault-weapons         | 2/16/18 | 2/19/18 | Quinnipiac          | Registered Voters |      67 |                 43 |                 91 | <https://poll.qu.edu/images/polling/us/us02202018_ugbw51.pdf/>                                                                                                       |
| ban-assault-weapons         | 2/18/18 | 2/20/18 | YouGov              | Registered Voters |      61 |                 45 |                 79 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/57yxejgrmo/econTabReport.pdf>                                                                        |
| ban-assault-weapons         | 2/25/18 | 2/27/18 | YouGov              | Registered Voters |      58 |                 37 |                 82 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/h8n9gvrqyj/econTabReport.pdf>                                                                        |
| ban-assault-weapons         | 2/15/18 | 2/19/18 | Morning Consult     | Registered Voters |      69 |                 64 |                 81 | <https://morningconsult.com/wp-content/uploads/2018/02/180211_crosstabs_POLITICO_v1_DK-1-1-1.pdf>                                                                    |
| ban-assault-weapons         | 2/15/18 | 2/18/18 | ABC/Washington Post | Registered Voters |      51 |                 29 |                 71 | <https://www.washingtonpost.com/page/2010-2019/WashingtonPost/2018/02/20/National-Politics/Polling/release_513.xml?tid=a_mcntx>                                      |
| ban-assault-weapons         | 2/20/18 | 2/23/18 | CNN/SSRS            | Registered Voters |      57 |                 34 |                 80 | <http://cdn.cnn.com/cnn/2018/images/02/25/rel3a.-.trump>,.guns.pdf                                                                                                   |
| ban-assault-weapons         | 2/20/18 | 2/23/18 | CBS News            | Adults            |      53 |                 39 |                 66 | <https://drive.google.com/file/d/0ByVu4fDHYJgVbjh5OW1Tb3FRdWxJQUpJSUdHUXdma1RKU0tV/view>                                                                             |
| ban-assault-weapons         | 2/16/18 | 2/19/18 | Harvard/Harris      | Registered Voters |      61 |                 43 |                 76 | <http://harvardharrispoll.com/wp-content/uploads/2018/02/Final_HHP_20Feb2018_RegisteredVoters_Topline_Memo.pdf>                                                      |
| ban-assault-weapons         | 2/20/18 | 2/24/18 | Suffolk             | Registered Voters |      63 |                 40 |                 83 | <http://www.suffolk.edu/documents/SUPRC/3_1_2018_marginals.pdf>                                                                                                      |
| ban-assault-weapons         | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      72 |                 58 |                 88 | <https://www.ipsos.com/en-us/npripsos-poll-majority-americans-support-policies-aimed-keep-guns-out-hands-dangerous-individuals>                                      |
| ban-assault-weapons         | 3/3/18  | 3/5/18  | Quinnipiac          | Registered Voters |      61 |                 38 |                 88 | <https://poll.qu.edu/national/release-detail?ReleaseID=2525>                                                                                                         |
| ban-assault-weapons         | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      68 |                 41 |                 83 | <https://morningconsult.com/wp-content/uploads/2018/03/180303_crosstabs_POLITICO_v1_DK-1.pdf>                                                                        |
| ban-high-capacity-magazines | 2/18/18 | 2/20/18 | YouGov              | Registered Voters |      67 |                 52 |                 82 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/57yxejgrmo/econTabReport.pdf>                                                                        |
| ban-high-capacity-magazines | 2/25/18 | 2/27/18 | YouGov              | Registered Voters |      64 |                 44 |                 82 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/h8n9gvrqyj/econTabReport.pdf>                                                                        |
| ban-high-capacity-magazines | 2/15/18 | 2/19/18 | Morning Consult     | Registered Voters |      69 |                 63 |                 82 | <https://morningconsult.com/wp-content/uploads/2018/02/180211_crosstabs_POLITICO_v1_DK-1-1-1.pdf>                                                                    |
| ban-high-capacity-magazines | 2/20/18 | 2/23/18 | CNN/SSRS            | Registered Voters |      64 |                 48 |                 82 | <http://cdn.cnn.com/cnn/2018/images/02/25/rel3a.-.trump>,.guns.pdf                                                                                                   |
| ban-high-capacity-magazines | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      73 |                 59 |                 88 | <https://www.ipsos.com/en-us/npripsos-poll-majority-americans-support-policies-aimed-keep-guns-out-hands-dangerous-individuals>                                      |
| ban-high-capacity-magazines | 3/3/18  | 3/5/18  | Quinnipiac          | Registered Voters |      63 |                 39 |                 87 | <https://poll.qu.edu/national/release-detail?ReleaseID=2525>                                                                                                         |
| ban-high-capacity-magazines | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      71 |                 64 |                 84 | <https://morningconsult.com/wp-content/uploads/2018/03/180303_crosstabs_POLITICO_v1_DK-1.pdf>                                                                        |
| mental-health-own-gun       | 2/18/18 | 2/20/18 | YouGov              | Registered Voters |      87 |                 87 |                 84 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/57yxejgrmo/econTabReport.pdf>                                                                        |
| mental-health-own-gun       | 2/25/18 | 2/27/18 | YouGov              | Registered Voters |      84 |                 81 |                 86 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/h8n9gvrqyj/econTabReport.pdf>                                                                        |
| mental-health-own-gun       | 2/15/18 | 2/19/18 | Morning Consult     | Registered Voters |      88 |                 89 |                 90 | <https://morningconsult.com/wp-content/uploads/2018/02/180211_crosstabs_POLITICO_v1_DK-1-1-1.pdf>                                                                    |
| mental-health-own-gun       | 2/20/18 | 2/24/18 | Suffolk             | Registered Voters |      76 |                 84 |                 78 | <http://www.suffolk.edu/documents/SUPRC/3_1_2018_marginals.pdf>                                                                                                      |
| mental-health-own-gun       | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      92 |                 88 |                 96 | <https://www.ipsos.com/en-us/npripsos-poll-majority-americans-support-policies-aimed-keep-guns-out-hands-dangerous-individuals>                                      |
| mental-health-own-gun       | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      88 |                 91 |                 91 | <https://morningconsult.com/wp-content/uploads/2018/03/180303_crosstabs_POLITICO_v1_DK-1.pdf>                                                                        |
| repeal-2nd-amendment        | 2/20/18 | 2/23/18 | CNN/SSRS            | Registered Voters |      10 |                  5 |                 15 | <http://cdn.cnn.com/cnn/2018/images/02/25/rel3a.-.trump>,.guns.pdf                                                                                                   |
| stricter-gun-laws           | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      75 |                 59 |                 92 | <https://www.ipsos.com/en-us/npripsos-poll-majority-americans-support-policies-aimed-keep-guns-out-hands-dangerous-individuals>                                      |
| stricter-gun-laws           | 2/25/18 | 2/27/18 | YouGov              | Registered Voters |      63 |                 43 |                 87 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/h8n9gvrqyj/econTabReport.pdf>                                                                        |
| stricter-gun-laws           | 2/20/18 | 2/23/18 | CNN/SSRS            | Registered Voters |      70 |                 49 |                 93 | <http://cdn.cnn.com/cnn/2018/images/02/25/rel3a.-.trump>,.guns.pdf                                                                                                   |
| stricter-gun-laws           | 2/20/18 | 2/23/18 | CBS News            | Adults            |      65 |                 43 |                 85 | <https://drive.google.com/file/d/0ByVu4fDHYJgVbjh5OW1Tb3FRdWxJQUpJSUdHUXdma1RKU0tV/view>                                                                             |
| stricter-gun-laws           | 2/20/18 | 2/21/18 | Marist              | Registered Voters |      73 |                 55 |                 93 | <http://maristpoll.marist.edu/wp-content/misc/usapolls/us180220/Marist%20Poll_National%20Nature%20of%20the%20Sample%20and%20Tables_February%2023,%202018.pdf#page=3> |
| stricter-gun-laws           | 2/18/18 | 2/20/18 | YouGov              | Registered Voters |      58 |                 36 |                 76 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/57yxejgrmo/econTabReport.pdf>                                                                        |
| stricter-gun-laws           | 2/16/18 | 2/19/18 | Quinnipiac          | Registered Voters |      66 |                 34 |                 86 | <https://poll.qu.edu/images/polling/us/us02202018_ugbw51.pdf/>                                                                                                       |
| stricter-gun-laws           | 2/22/18 | 2/26/18 | Morning Consult     | Registered Voters |      68 |                 53 |                 89 | <https://morningconsult.com/wp-content/uploads/2018/02/180217_crosstabs_POLITICO_v1_DK-1.pdf>                                                                        |
| stricter-gun-laws           | 3/3/18  | 3/5/18  | Quinnipiac          | Registered Voters |      63 |                 39 |                 88 | <https://poll.qu.edu/national/release-detail?ReleaseID=2525>                                                                                                         |
| stricter-gun-laws           | 3/4/18  | 3/6/18  | YouGov              | Registered Voters |      61 |                 42 |                 82 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/zq33h2ipcl/econTabReport.pdf>                                                                        |
| stricter-gun-laws           | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      69 |                 57 |                 85 | <https://morningconsult.com/wp-content/uploads/2018/03/180303_crosstabs_POLITICO_v1_DK-1.pdf>                                                                        |

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

| Question | Start   | End     | Pollster           | Population        | Support | Republican Support | Democratic Support | URL                                                                                                                                               |
|:---------|:--------|:--------|:-------------------|:------------------|--------:|-------------------:|-------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------|
| age-21   | 2/20/18 | 2/23/18 | CNN/SSRS           | Registered Voters |      72 |                 61 |                 86 | <http://cdn.cnn.com/cnn/2018/images/02/25/rel3a.-.trump>,.guns.pdf                                                                                |
| age-21   | 2/27/18 | 2/28/18 | NPR/Ipsos          | Adults            |      82 |                 72 |                 92 | <https://www.ipsos.com/en-us/npripsos-poll-majority-americans-support-policies-aimed-keep-guns-out-hands-dangerous-individuals>                   |
| age-21   | 3/1/18  | 3/4/18  | Rasmussen          | Adults            |      67 |                 59 |                 76 | <http://www.rasmussenreports.com/public_content/lifestyle/general_lifestyle/march_2018/americans_favor_raising_age_to_buy_gun_not_enlist_or_vote> |
| age-21   | 2/22/18 | 2/26/18 | Harris Interactive | Registered Voters |      84 |                 77 |                 92 | <http://thehill.com/opinion/civil-rights/375993-americans-support-no-gun-under-21>                                                                |
| age-21   | 3/3/18  | 3/5/18  | Quinnipiac         | Registered Voters |      78 |                 63 |                 93 | <https://poll.qu.edu/national/release-detail?ReleaseID=2525>                                                                                      |
| age-21   | 3/4/18  | 3/6/18  | YouGov             | Registered Voters |      72 |                 65 |                 80 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/zq33h2ipcl/econTabReport.pdf>                                                     |
| age-21   | 3/1/18  | 3/5/18  | Morning Consult    | Registered Voters |      76 |                 72 |                 86 | <https://morningconsult.com/wp-content/uploads/2018/03/180303_crosstabs_POLITICO_v1_DK-1.pdf>                                                     |

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

| Question | Start   | End     | Pollster           | Population        | Support | Republican Support | Democratic Support | URL                                                                                                                             |
|:---------|:--------|:--------|:-------------------|:------------------|--------:|-------------------:|-------------------:|:--------------------------------------------------------------------------------------------------------------------------------|
| age-21   | 2/27/18 | 2/28/18 | NPR/Ipsos          | Adults            |      82 |                 72 |                 92 | <https://www.ipsos.com/en-us/npripsos-poll-majority-americans-support-policies-aimed-keep-guns-out-hands-dangerous-individuals> |
| age-21   | 2/22/18 | 2/26/18 | Harris Interactive | Registered Voters |      84 |                 77 |                 92 | <http://thehill.com/opinion/civil-rights/375993-americans-support-no-gun-under-21>                                              |

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

| Population        | Support | Pollster           |
|:------------------|--------:|:-------------------|
| Registered Voters |      72 | CNN/SSRS           |
| Adults            |      82 | NPR/Ipsos          |
| Adults            |      67 | Rasmussen          |
| Registered Voters |      84 | Harris Interactive |
| Registered Voters |      78 | Quinnipiac         |
| Registered Voters |      72 | YouGov             |
| Registered Voters |      76 | Morning Consult    |

You can also specify a range of columns, for example all columns from
Support to Democratic Support:

``` r
select(age21, Support:`Democratic Support`)
```

| Support | Republican Support | Democratic Support |
|--------:|-------------------:|-------------------:|
|      72 |                 61 |                 86 |
|      82 |                 72 |                 92 |
|      67 |                 59 |                 76 |
|      84 |                 77 |                 92 |
|      78 |                 63 |                 93 |
|      72 |                 65 |                 80 |
|      76 |                 72 |                 86 |

Note the use of ‘backticks’ (reverse quotes) to specify the column name,
as R does not normally allow spaces in names.

You can also use some more versatile functions such as `contains()` or
`starts_with()` within a `select()` command:

``` r
select(age21, contains("Supp")) # Selects all variables that contain the stem "Supp" in their name
```

| Support | Republican Support | Democratic Support |
|--------:|-------------------:|-------------------:|
|      72 |                 61 |                 86 |
|      82 |                 72 |                 92 |
|      67 |                 59 |                 76 |
|      84 |                 77 |                 92 |
|      78 |                 63 |                 93 |
|      72 |                 65 |                 80 |
|      76 |                 72 |                 86 |

Select can also be used to rename columns when selecting them, for
example to get rid of the spaces:

``` r
select(age21, Pollster, rep = `Republican Support`, dem = `Democratic Support`)
```

| Pollster           | rep | dem |
|:-------------------|----:|----:|
| CNN/SSRS           |  61 |  86 |
| NPR/Ipsos          |  72 |  92 |
| Rasmussen          |  59 |  76 |
| Harris Interactive |  77 |  92 |
| Quinnipiac         |  63 |  93 |
| YouGov             |  65 |  80 |
| Morning Consult    |  72 |  86 |

Note that `select` drops all columns not selected. If you only want to
rename columns, you can use the `rename` function:

``` r
rename(age21, start_date = Start, end_date = End)
```

| Question | start\_date | end\_date | Pollster           | Population        | Support | Republican Support | Democratic Support | URL                                                                                                                                               |
|:---------|:------------|:----------|:-------------------|:------------------|--------:|-------------------:|-------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------|
| age-21   | 2/20/18     | 2/23/18   | CNN/SSRS           | Registered Voters |      72 |                 61 |                 86 | <http://cdn.cnn.com/cnn/2018/images/02/25/rel3a.-.trump>,.guns.pdf                                                                                |
| age-21   | 2/27/18     | 2/28/18   | NPR/Ipsos          | Adults            |      82 |                 72 |                 92 | <https://www.ipsos.com/en-us/npripsos-poll-majority-americans-support-policies-aimed-keep-guns-out-hands-dangerous-individuals>                   |
| age-21   | 3/1/18      | 3/4/18    | Rasmussen          | Adults            |      67 |                 59 |                 76 | <http://www.rasmussenreports.com/public_content/lifestyle/general_lifestyle/march_2018/americans_favor_raising_age_to_buy_gun_not_enlist_or_vote> |
| age-21   | 2/22/18     | 2/26/18   | Harris Interactive | Registered Voters |      84 |                 77 |                 92 | <http://thehill.com/opinion/civil-rights/375993-americans-support-no-gun-under-21>                                                                |
| age-21   | 3/3/18      | 3/5/18    | Quinnipiac         | Registered Voters |      78 |                 63 |                 93 | <https://poll.qu.edu/national/release-detail?ReleaseID=2525>                                                                                      |
| age-21   | 3/4/18      | 3/6/18    | YouGov             | Registered Voters |      72 |                 65 |                 80 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/zq33h2ipcl/econTabReport.pdf>                                                     |
| age-21   | 3/1/18      | 3/5/18    | Morning Consult    | Registered Voters |      76 |                 72 |                 86 | <https://morningconsult.com/wp-content/uploads/2018/03/180303_crosstabs_POLITICO_v1_DK-1.pdf>                                                     |

Finally, you can drop a variable by adding a minus sign in front of a
name:

``` r
select(age21, -Question, -URL)
```

| Start   | End     | Pollster           | Population        | Support | Republican Support | Democratic Support |
|:--------|:--------|:-------------------|:------------------|--------:|-------------------:|-------------------:|
| 2/20/18 | 2/23/18 | CNN/SSRS           | Registered Voters |      72 |                 61 |                 86 |
| 2/27/18 | 2/28/18 | NPR/Ipsos          | Adults            |      82 |                 72 |                 92 |
| 3/1/18  | 3/4/18  | Rasmussen          | Adults            |      67 |                 59 |                 76 |
| 2/22/18 | 2/26/18 | Harris Interactive | Registered Voters |      84 |                 77 |                 92 |
| 3/3/18  | 3/5/18  | Quinnipiac         | Registered Voters |      78 |                 63 |                 93 |
| 3/4/18  | 3/6/18  | YouGov             | Registered Voters |      72 |                 65 |                 80 |
| 3/1/18  | 3/5/18  | Morning Consult    | Registered Voters |      76 |                 72 |                 86 |

## Sorting with arrange()

You can easily sort a data set with `arrange`: you first specify the
data, and then the column(s) to sort on. To sort in descending order,
put a minus in front of a variable. For example, the following orders by
population and then by support (descending):

``` r
age21 <- arrange(age21, Population, -Support)
age21
```

| Question | Start   | End     | Pollster           | Population        | Support | Republican Support | Democratic Support | URL                                                                                                                                               |
|:---------|:--------|:--------|:-------------------|:------------------|--------:|-------------------:|-------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------|
| age-21   | 2/27/18 | 2/28/18 | NPR/Ipsos          | Adults            |      82 |                 72 |                 92 | <https://www.ipsos.com/en-us/npripsos-poll-majority-americans-support-policies-aimed-keep-guns-out-hands-dangerous-individuals>                   |
| age-21   | 3/1/18  | 3/4/18  | Rasmussen          | Adults            |      67 |                 59 |                 76 | <http://www.rasmussenreports.com/public_content/lifestyle/general_lifestyle/march_2018/americans_favor_raising_age_to_buy_gun_not_enlist_or_vote> |
| age-21   | 2/22/18 | 2/26/18 | Harris Interactive | Registered Voters |      84 |                 77 |                 92 | <http://thehill.com/opinion/civil-rights/375993-americans-support-no-gun-under-21>                                                                |
| age-21   | 3/3/18  | 3/5/18  | Quinnipiac         | Registered Voters |      78 |                 63 |                 93 | <https://poll.qu.edu/national/release-detail?ReleaseID=2525>                                                                                      |
| age-21   | 3/1/18  | 3/5/18  | Morning Consult    | Registered Voters |      76 |                 72 |                 86 | <https://morningconsult.com/wp-content/uploads/2018/03/180303_crosstabs_POLITICO_v1_DK-1.pdf>                                                     |
| age-21   | 2/20/18 | 2/23/18 | CNN/SSRS           | Registered Voters |      72 |                 61 |                 86 | <http://cdn.cnn.com/cnn/2018/images/02/25/rel3a.-.trump>,.guns.pdf                                                                                |
| age-21   | 3/4/18  | 3/6/18  | YouGov             | Registered Voters |      72 |                 65 |                 80 | <https://d25d2506sfb94s.cloudfront.net/cumulus_uploads/document/zq33h2ipcl/econTabReport.pdf>                                                     |

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
```

| Question | Pollster           | party\_diff |
|:---------|:-------------------|------------:|
| age-21   | NPR/Ipsos          |          20 |
| age-21   | Rasmussen          |          17 |
| age-21   | Harris Interactive |          15 |
| age-21   | Quinnipiac         |          30 |
| age-21   | Morning Consult    |          14 |
| age-21   | CNN/SSRS           |          25 |
| age-21   | YouGov             |          15 |

``` r
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

| Question | Pollster           | party\_diff |
|:---------|:-------------------|------------:|
| age-21   | Quinnipiac         |          30 |
| age-21   | CNN/SSRS           |          25 |
| age-21   | NPR/Ipsos          |          20 |
| age-21   | Rasmussen          |          17 |
| age-21   | Harris Interactive |          15 |
| age-21   | YouGov             |          15 |
| age-21   | Morning Consult    |          14 |

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

| Question | Pollster           | party\_diff |
|:---------|:-------------------|------------:|
| age-21   | Quinnipiac         |          30 |
| age-21   | CNN/SSRS           |          25 |
| age-21   | NPR/Ipsos          |          20 |
| age-21   | Rasmussen          |          17 |
| age-21   | Harris Interactive |          15 |
| age-21   | YouGov             |          15 |
| age-21   | Morning Consult    |          14 |

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
# Check out the help function first
?recode
```

    ## Recode values
    ## 
    ## Description:
    ## 
    ##      This is a vectorised version of 'switch()': you can replace
    ##      numeric values based on their position or their name, and
    ##      character or factor values only by their name. This is an S3
    ##      generic: dplyr provides methods for numeric, character, and
    ##      factors. For logical vectors, use 'if_else()'. For more
    ##      complicated criteria, use 'case_when()'.
    ## 
    ##      You can use 'recode()' directly with factors; it will preserve the
    ##      existing order of levels while changing the values. Alternatively,
    ##      you can use 'recode_factor()', which will change the order of
    ##      levels to match the order of replacements. See the forcats package
    ##      for more tools for working with factors and their levels.
    ## 
    ##      *[Questioning]* 'recode()' is questioning because the arguments
    ##      are in the wrong order. We have 'new <- old', 'mutate(df, new =
    ##      old)', and 'rename(df, new = old)' but 'recode(x, old = new)'. We
    ##      don't yet know how to fix this problem, but it's likely to involve
    ##      creating a new function then retiring or deprecating 'recode()'.
    ## 
    ## Usage:
    ## 
    ##      recode(.x, ..., .default = NULL, .missing = NULL)
    ##      
    ##      recode_factor(.x, ..., .default = NULL, .missing = NULL, .ordered = FALSE)
    ##      
    ## Arguments:
    ## 
    ##       .x: A vector to modify
    ## 
    ##      ...: <'dynamic-dots'> Replacements. For character and factor '.x',
    ##           these should be named and replacement is based only on their
    ##           name. For numeric '.x', these can be named or not. If not
    ##           named, the replacement is done based on position i.e. '.x'
    ##           represents positions to look for in replacements. See
    ##           examples.
    ## 
    ##           When named, the argument names should be the current values
    ##           to be replaced, and the argument values should be the new
    ##           (replacement) values.
    ## 
    ##           All replacements must be the same type, and must have either
    ##           length one or the same length as '.x'.
    ## 
    ## .default: If supplied, all values not otherwise matched will be given
    ##           this value. If not supplied and if the replacements are the
    ##           same type as the original values in '.x', unmatched values
    ##           are not changed. If not supplied and if the replacements are
    ##           not compatible, unmatched values are replaced with 'NA'.
    ## 
    ##           '.default' must be either length 1 or the same length as
    ##           '.x'.
    ## 
    ## .missing: If supplied, any missing values in '.x' will be replaced by
    ##           this value. Must be either length 1 or the same length as
    ##           '.x'.
    ## 
    ## .ordered: If 'TRUE', 'recode_factor()' creates an ordered factor.
    ## 
    ## Value:
    ## 
    ##      A vector the same length as '.x', and the same type as the first
    ##      of '...', '.default', or '.missing'. 'recode_factor()' returns a
    ##      factor whose levels are in the same order as in '...'. The levels
    ##      in '.default' and '.missing' come last.
    ## 
    ## See Also:
    ## 
    ##      'na_if()' to replace specified values with a 'NA'.
    ## 
    ##      'coalesce()' to replace missing values with a specified value.
    ## 
    ##      'tidyr::replace_na()' to replace 'NA' with a value.
    ## 
    ## Examples:
    ## 
    ##      # For character values, recode values with named arguments only. Unmatched
    ##      # values are unchanged.
    ##      char_vec <- sample(c("a", "b", "c"), 10, replace = TRUE)
    ##      recode(char_vec, a = "Apple")
    ##      recode(char_vec, a = "Apple", b = "Banana")
    ##      
    ##      # Use .default as replacement for unmatched values. Note that NA and
    ##      # replacement values need to be of the same type. For more information, see
    ##      # https://adv-r.hadley.nz/vectors-chap.html#missing-values
    ##      recode(char_vec, a = "Apple", b = "Banana", .default = NA_character_)
    ##      
    ##      # Throws an error as NA is logical, not character.
    ##      ## Not run:
    ##      
    ##      recode(char_vec, a = "Apple", b = "Banana", .default = NA)
    ##      ## End(Not run)
    ##      
    ##      
    ##      # Use a named character vector for unquote splicing with !!!
    ##      level_key <- c(a = "apple", b = "banana", c = "carrot")
    ##      recode(char_vec, !!!level_key)
    ##      
    ##      # For numeric values, named arguments can also be used
    ##      num_vec <- c(1:4, NA)
    ##      recode(num_vec, `2` = 20L, `4` = 40L)
    ##      
    ##      # Or if you don't name the arguments, recode() matches by position.
    ##      # (Only works for numeric vector)
    ##      recode(num_vec, "a", "b", "c", "d")
    ##      # .x (position given) looks in (...), then grabs (... value at position)
    ##      # so if nothing at position (here 5), it uses .default or NA.
    ##      recode(c(1,5,3), "a", "b", "c", "d", .default = "nothing")
    ##      
    ##      # Note that if the replacements are not compatible with .x,
    ##      # unmatched values are replaced by NA and a warning is issued.
    ##      recode(num_vec, `2` = "b", `4` = "d")
    ##      # use .default to change the replacement value
    ##      recode(num_vec, "a", "b", "c", .default = "other")
    ##      # use .missing to replace missing values in .x
    ##      recode(num_vec, "a", "b", "c", .default = "other", .missing = "missing")
    ##      
    ##      # For factor values, use only named replacements
    ##      # and supply default with levels()
    ##      factor_vec <- factor(c("a", "b", "c"))
    ##      recode(factor_vec, a = "Apple", .default = levels(factor_vec))
    ##      
    ##      # Use recode_factor() to create factors with levels ordered as they
    ##      # appear in the recode call. The levels in .default and .missing
    ##      # come last.
    ##      recode_factor(num_vec, `1` = "z", `2` = "y", `3` = "x")
    ##      recode_factor(num_vec, `1` = "z", `2` = "y", `3` = "x",
    ##                    .default = "D")
    ##      recode_factor(num_vec, `1` = "z", `2` = "y", `3` = "x",
    ##                    .default = "D", .missing = "M")
    ##      
    ##      # When the input vector is a compatible vector (character vector or
    ##      # factor), it is reused as default.
    ##      recode_factor(letters[1:3], b = "z", c = "y")
    ##      recode_factor(factor(letters[1:3]), b = "z", c = "y")
    ##      
    ##      # Use a named character vector to recode factors with unquote splicing.
    ##      level_key <- c(a = "apple", b = "banana", c = "carrot")
    ##      recode_factor(char_vec, !!!level_key)

``` r
# Solution
d %>%
  filter(Question == "arm-teachers") %>%     ## Filter based on question
  select(poll = Pollster,                    ## Selecting and renaming
         pop = Population,
         sup = Support) %>%
  mutate(sup2 = sup/100,                     ## Transforming the support variable
         pop = recode(pop, `Registered Voters` = "reg", Adults = "adu"))  ## Redoding the population variable
```

| poll            | pop | sup | sup2 |
|:----------------|:----|----:|-----:|
| YouGov/Huffpost | reg |  41 | 0.41 |
| CBS News        | adu |  44 | 0.44 |
| Rasmussen       | adu |  43 | 0.43 |
| NPR/Ipsos       | adu |  41 | 0.41 |
| Quinnipiac      | reg |  40 | 0.40 |
| SurveyMonkey    | reg |  43 | 0.43 |

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

| Question                    | Start   | End     | Pollster            | Population        | Support | Rep | Dem |
|:----------------------------|:--------|:--------|:--------------------|:------------------|--------:|----:|----:|
| age-21                      | 2/20/18 | 2/23/18 | CNN/SSRS            | Registered Voters |      72 |  61 |  86 |
| age-21                      | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      82 |  72 |  92 |
| age-21                      | 3/1/18  | 3/4/18  | Rasmussen           | Adults            |      67 |  59 |  76 |
| age-21                      | 2/22/18 | 2/26/18 | Harris Interactive  | Registered Voters |      84 |  77 |  92 |
| age-21                      | 3/3/18  | 3/5/18  | Quinnipiac          | Registered Voters |      78 |  63 |  93 |
| age-21                      | 3/4/18  | 3/6/18  | YouGov              | Registered Voters |      72 |  65 |  80 |
| age-21                      | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      76 |  72 |  86 |
| arm-teachers                | 2/23/18 | 2/25/18 | YouGov/Huffpost     | Registered Voters |      41 |  69 |  20 |
| arm-teachers                | 2/20/18 | 2/23/18 | CBS News            | Adults            |      44 |  68 |  20 |
| arm-teachers                | 2/27/18 | 2/28/18 | Rasmussen           | Adults            |      43 |  71 |  24 |
| arm-teachers                | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      41 |  68 |  18 |
| arm-teachers                | 3/3/18  | 3/5/18  | Quinnipiac          | Registered Voters |      40 |  77 |  10 |
| arm-teachers                | 2/26/18 | 2/28/18 | SurveyMonkey        | Registered Voters |      43 |  80 |  11 |
| background-checks           | 2/16/18 | 2/19/18 | Quinnipiac          | Registered Voters |      97 |  97 |  99 |
| background-checks           | 2/18/18 | 2/20/18 | YouGov              | Registered Voters |      85 |  78 |  87 |
| background-checks           | 2/25/18 | 2/27/18 | YouGov              | Registered Voters |      83 |  76 |  91 |
| background-checks           | 2/15/18 | 2/19/18 | Morning Consult     | Registered Voters |      88 |  88 |  90 |
| background-checks           | 2/20/18 | 2/23/18 | CBS News            | Adults            |      75 |  66 |  86 |
| background-checks           | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      94 |  89 |  99 |
| background-checks           | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      90 |  91 |  91 |
| ban-assault-weapons         | 2/16/18 | 2/19/18 | Quinnipiac          | Registered Voters |      67 |  43 |  91 |
| ban-assault-weapons         | 2/18/18 | 2/20/18 | YouGov              | Registered Voters |      61 |  45 |  79 |
| ban-assault-weapons         | 2/25/18 | 2/27/18 | YouGov              | Registered Voters |      58 |  37 |  82 |
| ban-assault-weapons         | 2/15/18 | 2/19/18 | Morning Consult     | Registered Voters |      69 |  64 |  81 |
| ban-assault-weapons         | 2/15/18 | 2/18/18 | ABC/Washington Post | Registered Voters |      51 |  29 |  71 |
| ban-assault-weapons         | 2/20/18 | 2/23/18 | CNN/SSRS            | Registered Voters |      57 |  34 |  80 |
| ban-assault-weapons         | 2/20/18 | 2/23/18 | CBS News            | Adults            |      53 |  39 |  66 |
| ban-assault-weapons         | 2/16/18 | 2/19/18 | Harvard/Harris      | Registered Voters |      61 |  43 |  76 |
| ban-assault-weapons         | 2/20/18 | 2/24/18 | Suffolk             | Registered Voters |      63 |  40 |  83 |
| ban-assault-weapons         | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      72 |  58 |  88 |
| ban-assault-weapons         | 3/3/18  | 3/5/18  | Quinnipiac          | Registered Voters |      61 |  38 |  88 |
| ban-assault-weapons         | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      68 |  41 |  83 |
| ban-high-capacity-magazines | 2/18/18 | 2/20/18 | YouGov              | Registered Voters |      67 |  52 |  82 |
| ban-high-capacity-magazines | 2/25/18 | 2/27/18 | YouGov              | Registered Voters |      64 |  44 |  82 |
| ban-high-capacity-magazines | 2/15/18 | 2/19/18 | Morning Consult     | Registered Voters |      69 |  63 |  82 |
| ban-high-capacity-magazines | 2/20/18 | 2/23/18 | CNN/SSRS            | Registered Voters |      64 |  48 |  82 |
| ban-high-capacity-magazines | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      73 |  59 |  88 |
| ban-high-capacity-magazines | 3/3/18  | 3/5/18  | Quinnipiac          | Registered Voters |      63 |  39 |  87 |
| ban-high-capacity-magazines | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      71 |  64 |  84 |
| mental-health-own-gun       | 2/18/18 | 2/20/18 | YouGov              | Registered Voters |      87 |  87 |  84 |
| mental-health-own-gun       | 2/25/18 | 2/27/18 | YouGov              | Registered Voters |      84 |  81 |  86 |
| mental-health-own-gun       | 2/15/18 | 2/19/18 | Morning Consult     | Registered Voters |      88 |  89 |  90 |
| mental-health-own-gun       | 2/20/18 | 2/24/18 | Suffolk             | Registered Voters |      76 |  84 |  78 |
| mental-health-own-gun       | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      92 |  88 |  96 |
| mental-health-own-gun       | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      88 |  91 |  91 |
| repeal-2nd-amendment        | 2/20/18 | 2/23/18 | CNN/SSRS            | Registered Voters |      10 |   5 |  15 |
| stricter-gun-laws           | 2/27/18 | 2/28/18 | NPR/Ipsos           | Adults            |      75 |  59 |  92 |
| stricter-gun-laws           | 2/25/18 | 2/27/18 | YouGov              | Registered Voters |      63 |  43 |  87 |
| stricter-gun-laws           | 2/20/18 | 2/23/18 | CNN/SSRS            | Registered Voters |      70 |  49 |  93 |
| stricter-gun-laws           | 2/20/18 | 2/23/18 | CBS News            | Adults            |      65 |  43 |  85 |
| stricter-gun-laws           | 2/20/18 | 2/21/18 | Marist              | Registered Voters |      73 |  55 |  93 |
| stricter-gun-laws           | 2/18/18 | 2/20/18 | YouGov              | Registered Voters |      58 |  36 |  76 |
| stricter-gun-laws           | 2/16/18 | 2/19/18 | Quinnipiac          | Registered Voters |      66 |  34 |  86 |
| stricter-gun-laws           | 2/22/18 | 2/26/18 | Morning Consult     | Registered Voters |      68 |  53 |  89 |
| stricter-gun-laws           | 3/3/18  | 3/5/18  | Quinnipiac          | Registered Voters |      63 |  39 |  88 |
| stricter-gun-laws           | 3/4/18  | 3/6/18  | YouGov              | Registered Voters |      61 |  42 |  82 |
| stricter-gun-laws           | 3/1/18  | 3/5/18  | Morning Consult     | Registered Voters |      69 |  57 |  85 |

## Grouping rows

Now, we can use the group\_by function to group by, for example,
pollster:

``` r
d %>% 
  group_by(Question)
```

    ## # A tibble: 57 x 8
    ## # Groups:   Question [8]
    ##    Question    Start   End     Pollster       Population     Support   Rep   Dem
    ##    <chr>       <chr>   <chr>   <chr>          <chr>            <dbl> <dbl> <dbl>
    ##  1 age-21      2/20/18 2/23/18 CNN/SSRS       Registered Vo…      72    61    86
    ##  2 age-21      2/27/18 2/28/18 NPR/Ipsos      Adults              82    72    92
    ##  3 age-21      3/1/18  3/4/18  Rasmussen      Adults              67    59    76
    ##  4 age-21      2/22/18 2/26/18 Harris Intera… Registered Vo…      84    77    92
    ##  5 age-21      3/3/18  3/5/18  Quinnipiac     Registered Vo…      78    63    93
    ##  6 age-21      3/4/18  3/6/18  YouGov         Registered Vo…      72    65    80
    ##  7 age-21      3/1/18  3/5/18  Morning Consu… Registered Vo…      76    72    86
    ##  8 arm-teache… 2/23/18 2/25/18 YouGov/Huffpo… Registered Vo…      41    69    20
    ##  9 arm-teache… 2/20/18 2/23/18 CBS News       Adults              44    68    20
    ## 10 arm-teache… 2/27/18 2/28/18 Rasmussen      Adults              43    71    24
    ## # … with 47 more rows

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

| Question                    |  Support |
|:----------------------------|---------:|
| background-checks           | 87.42857 |
| mental-health-own-gun       | 85.83333 |
| age-21                      | 75.85714 |
| ban-high-capacity-magazines | 67.28571 |
| stricter-gun-laws           | 66.45455 |
| ban-assault-weapons         | 61.75000 |
| arm-teachers                | 42.00000 |
| repeal-2nd-amendment        | 10.00000 |

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

| Question                    |      Dem |      Rep |        Diff |
|:----------------------------|---------:|---------:|------------:|
| stricter-gun-laws           | 86.90909 | 46.36364 |  40.5454545 |
| ban-assault-weapons         | 80.66667 | 42.58333 |  38.0833333 |
| ban-high-capacity-magazines | 83.85714 | 52.71429 |  31.1428571 |
| age-21                      | 86.42857 | 67.00000 |  19.4285714 |
| repeal-2nd-amendment        | 15.00000 |  5.00000 |  10.0000000 |
| background-checks           | 91.85714 | 83.57143 |   8.2857143 |
| mental-health-own-gun       | 87.50000 | 86.66667 |   0.8333333 |
| arm-teachers                | 17.16667 | 72.16667 | -55.0000000 |

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

| Question                    |   n |     mean |       sd |
|:----------------------------|----:|---------:|---------:|
| age-21                      |   7 | 75.85714 | 6.011893 |
| arm-teachers                |   6 | 42.00000 | 1.549193 |
| background-checks           |   7 | 87.42857 | 7.322503 |
| ban-assault-weapons         |  12 | 61.75000 | 6.440285 |
| ban-high-capacity-magazines |   7 | 67.28571 | 3.860669 |
| mental-health-own-gun       |   6 | 85.83333 | 5.455884 |
| repeal-2nd-amendment        |   1 | 10.00000 |       NA |
| stricter-gun-laws           |  11 | 66.45455 | 5.145165 |

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

    ## # A tibble: 57 x 10
    ## # Groups:   Question [8]
    ##    Question Start End   Pollster Population Support   Rep   Dem avg_support
    ##    <chr>    <chr> <chr> <chr>    <chr>        <dbl> <dbl> <dbl>       <dbl>
    ##  1 age-21   2/20… 2/23… CNN/SSRS Registere…      72    61    86        75.9
    ##  2 age-21   2/27… 2/28… NPR/Ips… Adults          82    72    92        75.9
    ##  3 age-21   3/1/… 3/4/… Rasmuss… Adults          67    59    76        75.9
    ##  4 age-21   2/22… 2/26… Harris … Registere…      84    77    92        75.9
    ##  5 age-21   3/3/… 3/5/… Quinnip… Registere…      78    63    93        75.9
    ##  6 age-21   3/4/… 3/6/… YouGov   Registere…      72    65    80        75.9
    ##  7 age-21   3/1/… 3/5/… Morning… Registere…      76    72    86        75.9
    ##  8 arm-tea… 2/23… 2/25… YouGov/… Registere…      41    69    20        42  
    ##  9 arm-tea… 2/20… 2/23… CBS News Adults          44    68    20        42  
    ## 10 arm-tea… 2/27… 2/28… Rasmuss… Adults          43    71    24        42  
    ## # … with 47 more rows, and 1 more variable: diff <dbl>

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

|     diff |
|---------:|
| 5.192389 |

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

    ## # A tibble: 15 x 3
    ## # Groups:   Question [8]
    ##    Question                    Population        Support
    ##    <chr>                       <chr>               <dbl>
    ##  1 age-21                      Adults               74.5
    ##  2 age-21                      Registered Voters    76.4
    ##  3 arm-teachers                Adults               42.7
    ##  4 arm-teachers                Registered Voters    41.3
    ##  5 background-checks           Adults               84.5
    ##  6 background-checks           Registered Voters    88.6
    ##  7 ban-assault-weapons         Adults               62.5
    ##  8 ban-assault-weapons         Registered Voters    61.6
    ##  9 ban-high-capacity-magazines Adults               73  
    ## 10 ban-high-capacity-magazines Registered Voters    66.3
    ## 11 mental-health-own-gun       Adults               92  
    ## 12 mental-health-own-gun       Registered Voters    84.6
    ## 13 repeal-2nd-amendment        Registered Voters    10  
    ## 14 stricter-gun-laws           Adults               70  
    ## 15 stricter-gun-laws           Registered Voters    65.7

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

# Check out data set
head(d)
```

|  id | condition | norm | profile | age | gender | norm\_perc | disclosure |
|----:|:----------|:-----|:--------|----:|-------:|-----------:|-----------:|
|   1 | n0\_p20   | n0   | p20     |  25 |      1 |         NA |        5.0 |
|   2 | n20\_p20  | n20  | p20     |  29 |      1 |   6.750000 |        6.0 |
|   3 | n0\_p0    | n0   | p0      |  30 |      0 |   6.000000 |        6.5 |
|   4 | n20\_p80  | n20  | p80     |  28 |      1 |   6.083333 |        6.0 |
|   5 | n20\_p20  | n20  | p20     |  28 |      0 |   7.000000 |        7.0 |
|   6 | n0\_p80   | n0   | p80     |  36 |      1 |   3.454546 |        3.5 |

``` r
## 1: Does age differ across the nine conditions

# Summarizing means across the nine conditions
d %>%
  group_by(condition) %>%
  summarize(mean = mean(age, na.rm = T),
            sd = sd(age, na.rm = T))
```

| condition |     mean |       sd |
|:----------|---------:|---------:|
| n0\_p0    | 37.37805 | 11.31113 |
| n0\_p20   | 34.67606 | 10.35067 |
| n0\_p80   | 38.20000 | 11.18249 |
| n20\_p0   | 38.30137 | 12.56614 |
| n20\_p20  | 34.86111 | 11.13339 |
| n20\_p80  | 35.30864 | 10.96659 |
| n80\_p0   | 36.45205 | 11.51815 |
| n80\_p20  | 38.55556 | 13.59773 |
| n80\_p80  | 37.69863 | 11.72330 |

``` r
# Alternative: Grouping by both factors (same result, a bit more organized)
d %>%
  group_by(norm, profile) %>%
  summarize(mean = mean(age, na.rm = T),
            sd = sd(age, na.rm = T))
```

    ## # A tibble: 9 x 4
    ## # Groups:   norm [3]
    ##   norm  profile  mean    sd
    ##   <chr> <chr>   <dbl> <dbl>
    ## 1 n0    p0       37.4  11.3
    ## 2 n0    p20      34.7  10.4
    ## 3 n0    p80      38.2  11.2
    ## 4 n20   p0       38.3  12.6
    ## 5 n20   p20      34.9  11.1
    ## 6 n20   p80      35.3  11.0
    ## 7 n80   p0       36.5  11.5
    ## 8 n80   p20      38.6  13.6
    ## 9 n80   p80      37.7  11.7

``` r
# Bonus: Testing the significance
m <- aov(age ~ norm + profile, d)
summary(m)   # -> There is no significant difference in age across conditions. 
```

    ##              Df Sum Sq Mean Sq F value Pr(>F)
    ## norm          2    227   113.7   0.839  0.433
    ## profile       2    217   108.3   0.799  0.450
    ## Residuals   672  91073   135.5

``` r
## 2. How did the two norm manipulations affect subsequent norm perceptions and disclosure intentions

# Summarizing across nine conditions
d %>%
  group_by(norm, profile) %>%
  summarize(mean_norms = mean(norm_perc, na.rm = T),
            mean_disc = mean(disclosure, na.rm = T))
```

    ## # A tibble: 9 x 4
    ## # Groups:   norm [3]
    ##   norm  profile mean_norms mean_disc
    ##   <chr> <chr>        <dbl>     <dbl>
    ## 1 n0    p0            3.07      3.01
    ## 2 n0    p20           2.31      2.56
    ## 3 n0    p80           2.93      2.90
    ## 4 n20   p0            4.29      3.70
    ## 5 n20   p20           4.71      3.87
    ## 6 n20   p80           4.72      4.03
    ## 7 n80   p0            5.56      4.15
    ## 8 n80   p20           5.76      4.23
    ## 9 n80   p80           5.80      4.36

``` r
# Per post manipulation
d %>%
  group_by(norm) %>%
  summarize(mean_norms = mean(norm_perc, na.rm = T),
            mean_disc = mean(disclosure, na.rm = T))
```

| norm | mean\_norms | mean\_disc |
|:-----|------------:|-----------:|
| n0   |    2.793684 |   2.838197 |
| n20  |    4.578305 |   3.872714 |
| n80  |    5.706804 |   4.247706 |

``` r
# -> Both norm perceptions and disclosure seem to increase if more faces are shown in the feed!

# Per profile manipulation
d %>%
  group_by(profile) %>%
  summarize(mean_norms = mean(norm_perc, na.rm = T),
            mean_disc = mean(disclosure, na.rm = T))
```

| profile | mean\_norms | mean\_disc |
|:--------|------------:|-----------:|
| p0      |    4.256679 |   3.596637 |
| p20     |    4.287879 |   3.558139 |
| p80     |    4.445642 |   3.750000 |

``` r
# -> Showing more faces did not seem to increase the norm perception or disclosure. 

# Bonus: Significance test (analysis of variance)
summary(aov(norm_perc ~ norm + profile, d))
```

    ##              Df Sum Sq Mean Sq F value Pr(>F)    
    ## norm          2  972.4   486.2 334.066 <2e-16 ***
    ## profile       2    5.9     3.0   2.041  0.131    
    ## Residuals   670  975.1     1.5                   
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 2 observations deleted due to missingness

``` r
summary(aov(disclosure ~ norm + profile, d))
```

    ##              Df Sum Sq Mean Sq F value Pr(>F)    
    ## norm          2  242.5  121.27   47.46 <2e-16 ***
    ## profile       2    5.3    2.63    1.03  0.358    
    ## Residuals   672 1716.9    2.55                   
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
# -> Indeed, only the post (norm) manipulation had a significant effect on both variables. 


# 3. How strongly are norm perceptions and intentions correlated?

cor.test(d$norm_perc, d$disclosure)
```

    ## 
    ##  Pearson's product-moment correlation
    ## 
    ## data:  d$norm_perc and d$disclosure
    ## t = 16.966, df = 673, p-value < 2.2e-16
    ## alternative hypothesis: true correlation is not equal to 0
    ## 95 percent confidence interval:
    ##  0.4922072 0.5980999
    ## sample estimates:
    ##       cor 
    ## 0.5473405

``` r
# -> The correlation is r = .55 (p < .001). The correlation is hence comparatively strong!
```
