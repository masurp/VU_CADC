Homework 1: Basic Data Wrangling (Solutions)
================
Philipp Masur

-   [Formalities](#formalities)
-   [Instructions](#instructions)
-   [Exercises](#exercises)
    -   [Task 1: Provide a description of the
        sample](#task-1-provide-a-description-of-the-sample)
    -   [Task 2: Explore social media
        use](#task-2-explore-social-media-use)
    -   [Task 3: Norm perceptions on
        Instagram](#task-3-norm-perceptions-on-instagram)

# Formalities

-   Name: \[ENTER YOUR NAME HERE\]
-   Student ID: \[ENTER YOUR STUDENT ID HERE\]

In the end, klick on “knit” and upload the respective html-output file
to Canvas. Please add your name and lastname to the output file name:
e.g., 01\_homework\_assignment\_NAME-LASTNAME.html

# Instructions

In this homework assignment, you are going to practice what you have
learned in the past two practical sessions. To do soe, we need to get
some data into R. On Canvas, you will find a module called “Data Sets”.
Within this module, there are several data sets that we will use
throughout the course. For this assignment, download the data set
**norm\_data.csv**. This data set stems from an online survey on social
media use and norm perceptions.

If you have set the working directory to the location of this RMarkdown
Script and you put the csv-file in the same folder, you can load that
data set without specifying a specific path.

``` r
library(tidyverse)
d <- read_csv("norm_data.csv")
head(d)
```

    ## # A tibble: 6 x 144
    ##   id    duration MLI_REF_01 MLI_REF_02 MLI_REF_03 MLI_REF_04 MLI_REF_05
    ##   <chr>    <dbl>      <dbl>      <dbl>      <dbl>      <dbl>      <dbl>
    ## 1 R_Un…      206          5          7          5          3          5
    ## 2 R_2Y…      340          5          3          2          2          3
    ## 3 R_BS…      282          4          4          3          4          4
    ## 4 R_3s…      428          6          3          6          6          7
    ## 5 R_1k…      551          5          4          6          7          5
    ## 6 R_2p…      408          5          7          6          1          6
    ## # … with 137 more variables: MLI_REF_06 <dbl>, MLI_REF_07 <dbl>,
    ## #   MLI_CRI_01 <dbl>, MLI_CRI_02 <dbl>, MLI_CRI_03 <dbl>, MLI_CRI_04 <dbl>,
    ## #   MLI_CRI_05 <dbl>, MLI_CRI_06 <dbl>, MLI_CRI_07 <dbl>, INC_SIZ_01 <dbl>,
    ## #   FAC_SIZE_01 <chr>, INC_NET_01 <dbl>, INC_NET_02 <dbl>, INC_NET_03 <dbl>,
    ## #   INC_NET_04 <dbl>, INC_NET_05 <dbl>, INC_NET_06 <dbl>, INC_NET_07 <dbl>,
    ## #   INC_NET_08 <dbl>, INC_NET_09 <dbl>, INC_NET_10 <dbl>, INC_NET_11 <dbl>,
    ## #   FAC_NET_01 <dbl>, FAC_NET_02 <dbl>, FAC_NET_03 <dbl>, FAC_NET_04 <dbl>,
    ## #   FAC_NET_05 <dbl>, FAC_NET_06 <dbl>, FAC_NET_07 <dbl>, FAC_NET_08 <dbl>,
    ## #   FAC_NET_09 <dbl>, FAC_NET_10 <dbl>, FAC_NET_11 <dbl>, INV_NOR_01 <dbl>,
    ## #   INV_NOR_02 <dbl>, INV_NOR_03 <dbl>, INV_NOR_04 <dbl>, INV_NOR_05 <dbl>,
    ## #   INV_NOR_06 <dbl>, INV_NOR_07 <dbl>, INV_NOR_08 <dbl>, INV_NOR_09 <dbl>,
    ## #   INV_NOR_10 <dbl>, INV_NOR_11 <dbl>, INV_NOR_12 <dbl>, IND_NOR_01 <dbl>,
    ## #   IND_NOR_02 <dbl>, IND_NOR_03 <dbl>, IND_NOR_04 <dbl>, IND_NOR_05 <dbl>,
    ## #   IND_NOR_06 <dbl>, IND_NOR_07 <dbl>, IND_NOR_08 <dbl>, IND_NOR_09 <dbl>,
    ## #   IND_NOR_10 <dbl>, IND_NOR_11 <dbl>, IND_NOR_12 <dbl>, FAV_NOR_01 <dbl>,
    ## #   FAV_NOR_02 <dbl>, FAV_NOR_03 <dbl>, FAV_NOR_04 <dbl>, FAV_NOR_05 <dbl>,
    ## #   FAV_NOR_06 <dbl>, FAV_NOR_07 <dbl>, FAV_NOR_08 <dbl>, FAV_NOR_09 <dbl>,
    ## #   FAV_NOR_10 <dbl>, FAV_NOR_11 <dbl>, FAV_NOR_12 <dbl>, FAD_NOR_01 <dbl>,
    ## #   FAD_NOR_02 <dbl>, FAD_NOR_03 <dbl>, FAD_NOR_04 <dbl>, FAD_NOR_05 <dbl>,
    ## #   FAD_NOR_06 <dbl>, FAD_NOR_07 <dbl>, FAD_NOR_08 <dbl>, FAD_NOR_09 <dbl>,
    ## #   FAD_NOR_10 <dbl>, FAD_NOR_11 <dbl>, FAD_NOR_12 <dbl>, INV_BEH_01 <dbl>,
    ## #   INV_BEH_02 <dbl>, INV_BEH_03 <dbl>, INV_BEH_04 <dbl>, INV_BEH_05 <dbl>,
    ## #   INV_BEH_06 <dbl>, IND_BEH_01 <dbl>, IND_BEH_02 <dbl>, IND_BEH_03 <dbl>,
    ## #   IND_BEH_04 <dbl>, IND_BEH_05 <dbl>, IND_BEH_06 <dbl>, IND_BEH_07 <dbl>,
    ## #   FAV_BEH_01 <dbl>, FAV_BEH_02 <dbl>, FAV_BEH_03 <dbl>, FAV_BEH_04 <dbl>,
    ## #   FAV_BEH_05 <dbl>, FAV_BEH_06 <dbl>, …

# Exercises

## Task 1: Provide a description of the sample

Let’s explore the data set. The data set contains some variables that
assess socio-demographic characteristics of the participants (age,
gender, edu, race…). Use the tidyverse functions that we discussed on
the last practical session,

-   Create a subset that only contains socio-demographic variables.
-   How old are participants on average?
-   How many female and male participants are in the data set (0 = male,
    1 = female, 2 = other)?
-   How old are participants from different educational levels?

``` r
# Create subset
d_soc <- d %>%   ## don't forget to save the subset to be able to work with it afterwards!
  select(age, gender, edu, race, employment, relationship)

# Recode one person who provided birth year instead of age (This is a good reminder to always check your data!)
d_soc <- d_soc %>%
  mutate(age = ifelse(age > 100, 2021-age, age))

# Alternative 1: Using base functions
mean(d_soc$age, na.rm = T)
```

    ## [1] 36.50807

``` r
sd(d_soc$age, na.rm = T)
```

    ## [1] 11.25631

``` r
# Alternative 2: Using the summarize function in the tidyvere
d_soc %>%
  summarize(m = mean(age, na.rm = T),
            sd = sd(age, na.rm = T))
```

    ## # A tibble: 1 x 2
    ##       m    sd
    ##   <dbl> <dbl>
    ## 1  36.5  11.3

``` r
# Alternative 3: Using the psych package and its function describe
library(psych)  # you have to install this first!
describe(d_soc$age) 
```

    ##    vars    n  mean    sd median trimmed   mad min max range skew kurtosis   se
    ## X1    1 1053 36.51 11.26     34   35.25 10.38  18  80    62 0.99     0.51 0.35

**Answer:** Participants are on average M = 36.5 years old (SD = 11.3).

``` r
# Alternative 1: Frequency table of gender using summarize
d_soc %>%
  group_by(gender) %>%
  summarize(n = n()) %>%
  mutate(prop = prop.table(n))  ## adding percentages to the table
```

    ## # A tibble: 4 x 3
    ##   gender     n    prop
    ## *  <dbl> <int>   <dbl>
    ## 1      0   411 0.359  
    ## 2      1   638 0.557  
    ## 3      2     5 0.00436
    ## 4     NA    92 0.0803

``` r
# Alternative 2: Using the count function
d_soc %>%
  group_by(gender) %>%
  count %>%                     ## Does the same as the summarizing command above
  mutate(prop = prop.table(n))  
```

    ## # A tibble: 4 x 3
    ## # Groups:   gender [4]
    ##   gender     n  prop
    ##    <dbl> <int> <dbl>
    ## 1      0   411     1
    ## 2      1   638     1
    ## 3      2     5     1
    ## 4     NA    92     1

**Answer:** There are 411 males (35.9%) and 638 females (55.7%) in the
sample. Five participants did not indicate their gender and 92 did not
indicate their gender at all.

``` r
# Age per educational level
d_soc %>%
  group_by(edu) %>%
  summarize(m = mean(age, na.rm = T))
```

    ## # A tibble: 7 x 2
    ##     edu     m
    ## * <dbl> <dbl>
    ## 1     1  34.8
    ## 2     2  34.2
    ## 3     3  37.4
    ## 4     4  35.7
    ## 5     5  36.6
    ## 6     6  37.9
    ## 7    NA NaN

**Answer:** Age hardly differs between educational levels (range = 34.2
- 37.9).

## Task 2: Explore social media use

The following code adds a mean indices that represent the latent
dimension *critical media literacy* (ranging from 1 to 7).

-   Create a subset that contains the variables *age*, *edu*, and
    *MLI\_CRI*.
-   Remove all missing values using the function `na.omit()`.
-   Does critical media literacy differ depending on the educational
    level?
-   Filter out people below 40 and again assess whether critical media
    literacy differs across educational levels.

``` r
d <- d %>%
  mutate(MLI_CRI = rowMeans(d %>% select(contains("MLI_CRI")), na.rm = T))

# Select variabke, remove all missings and create the subset
d2 <- d %>%                        ## again, don't forget to save your steps in a new object!
  select(age, edu, MLI_CRI) %>%
  na.omit

# Summarize critical media literacy by educational level
d2 %>%
  group_by(edu) %>%
  summarise(mean = mean(MLI_CRI, na.rm = T),
            sd = sd(MLI_CRI, na.rm = T))
```

    ## # A tibble: 6 x 3
    ##     edu  mean    sd
    ## * <dbl> <dbl> <dbl>
    ## 1     1  3.61  2.36
    ## 2     2  4.55  1.45
    ## 3     3  4.72  1.24
    ## 4     4  4.96  1.15
    ## 5     5  4.88  1.23
    ## 6     6  5.03  1.21

``` r
# Bonus: Significant test using an analysis of variance (ANOVA)
m <- aov(MLI_CRI ~ edu, d2)
summary(m)
```

    ##               Df Sum Sq Mean Sq F value   Pr(>F)    
    ## edu            1   20.9  20.907   13.96 0.000197 ***
    ## Residuals   1051 1574.2   1.498                     
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

**Answer:** Based on a simple mean comparison, critical media literacy
does differ between educational level: The higher the educational level,
the more literate are the participants. This difference is significant,
F(1, 1051) = 13.96, p &lt; .001.

``` r
# Create subset
d2_old <- d2 %>%
  filter(age >= 40)

# Summarize data in subset
d2_old %>%
  group_by(edu) %>%
  summarize(mean = mean(MLI_CRI, na.rm = T),
            sd = sd(MLI_CRI, na.rm = T))
```

    ## # A tibble: 6 x 3
    ##     edu  mean    sd
    ## * <dbl> <dbl> <dbl>
    ## 1     1  4.14 NA   
    ## 2     2  4.42  1.25
    ## 3     3  4.71  1.34
    ## 4     4  4.92  1.05
    ## 5     5  4.97  1.23
    ## 6     6  5.01  1.33

``` r
# Bonus: Significant test using an analysis of variance (ANOVA)
m <- aov(MLI_CRI ~ edu, d2_old)
summary(m)
```

    ##              Df Sum Sq Mean Sq F value Pr(>F)  
    ## edu           1    7.0   6.999   4.647 0.0318 *
    ## Residuals   334  503.1   1.506                 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

**Answer:** If we only inspect older participants (&gt;= 40 years), this
results changes slightly: the differences are seemingly smaller. Yet,
the difference is still statistically significant: F(1,334) = 4.65, p =
.032.

## Task 3: Norm perceptions on Instagram

The data set contains some variables that measure norm perceptions in
relation to visual self-disclosure on Instagram (12 items starting with
“INV\_NOR\_”).

-   Create the mean across all these item per person and add it as a new
    column to the data set.
-   Do norm perceptions differ depending on gender (only looking at 0 =
    males and 1 = females)?
-   How could you test whether this difference is statistically
    significant (tip: ?t.test)?

``` r
d <- d %>%
  mutate(NORM = rowMeans(d %>% select(contains("INV_NOR")), na.rm = T))

# Filter 
d_gen <- d %>%
  filter(gender == 0 | gender == 1)

# Means
d_gen %>%
  group_by(gender) %>%
  summarize(average_norm = mean(NORM))
```

    ## # A tibble: 2 x 2
    ##   gender average_norm
    ## *  <dbl>        <dbl>
    ## 1      0         4.90
    ## 2      1         5.13

``` r
# Significant test
t.test(NORM ~ gender, d_gen)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  NORM by gender
    ## t = -3.9973, df = 846.76, p-value = 6.965e-05
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.3401149 -0.1161021
    ## sample estimates:
    ## mean in group 0 mean in group 1 
    ##        4.901421        5.129530

**Answer:** Males perceive norms related to visual self-disclosure to be
slightly less strong (M = 4.90) compared to females (M = 5.1). This
difference is statistically significant, t(846.76) = -4.00, p &lt; .001.
