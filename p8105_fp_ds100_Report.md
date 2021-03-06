p8105\_fp\_ds100\_Report
================
Yue Gu (yg2625), Jianghui Lin (jl5172), Junyuan Zheng (jz3036), Jianyou Liu (jl5296), Zhiqian Fang (zf2212)
12/3/2018

Motivation
==========

Suicide is a leading cause of death in the US. Suicide rates increased in nearly every state from 1999 through 2016. Mental health conditions are often seen as the cause of suicide, but suicide is rarely caused by any single factor. In fact, many people who died by suicide are not known to have a diagnosed mental health condition at the time of death. We are interested in examining the variations of suicide death rates among different categorical variables such as gender, race, age group, and to test if the observed differences are statistically significant based on the analysis of suicide death rates and related confidence intervals.

Related work
============

Suicide has ranked as the 10th leading cause of death among Americans for many years. Here is the link for a report that inspired us. <https://afsp.org/about-suicide/suicide-statistics/> This report summarizes the suicide rates by race and age. In addition, they also include the most common suicide methods and data for suicide attempts.

Initial questions
=================

The initial thought of this project is to discover the relationship between suicide and people's mental status. We have found two separated datasets, one of them is the Injury Mortality data in the U.S., the other contains people's depression status in the US. Since both datasets include information for age, race, and gender, we decided to compare the trend of suicide death rates with depression prevalence under these three categories.

Data
====

BRFSS Prevalence Data (2011 to present)
---------------------------------------

Data from the Behavioral Risk Factor Surveillance System (BRFSS) Prevalence Data (2011 to present) were accessed from cdc.gov.(<https://chronicdata.cdc.gov/Behavioral-Risk-Factors/Behavioral-Risk-Factor-Surveillance-System-BRFSS-P/dttw-5yxu>) The version of the data that we will use in this analysis can be found in our Github.

Methodology: <http://www.cdc.gov/brfss/factsheets/pdf/DBS_BRFSS_survey.pdf>

Glossary: <http://apps.nccd.cdc.gov/BRFSSQuest/index.asp>

### Data acquisition and description

Since the original dataset is too large, we download the dataset and acquire the data we need. The specific data to be used in this project was accessed in December 2018 using the code below.

``` r
library(tidyverse)

data_BRFSS = 
  read_csv(file='./data/Behavioral_Risk_Factor_Surveillance_System__BRFSS__Prevalence_Data__2011_to_present_.csv')

data_BRFSS %>% 
  select(., Question) %>%
  distinct(.,)

brfss_data = brfss_raw %>% 
   filter(year %in% c(2011, 2012, 2013, 2014, 2015, 2016),
          break_out_category %in% c("Age Group", "Race/Ethnicity", "Gender")) %>% 
   spread(key = break_out_category, break_out)
```

The original dataset contains 1,386,855 rows and 27 columns. For further use, we make a preliminary dataset. The preliminary dataset contains 8931 rows and 14 columns. We saved the preliminary dataset in our data file. The link for complete data dictionary is attached above.

### Further cleaning

``` r
# load the preliminary dataset
brfss_data = read.csv("./data/brfss_data.csv") %>% janitor::clean_names()
# create age dataset
brfss_age = brfss_data %>% 
  select(year, locationabbr, locationdesc, response, sample_size, data_value, age_group) %>% 
  filter(!is.na(age_group)) 
# create race dataset
brfss_race = brfss_data %>% 
  select(year, locationabbr, locationdesc, response, sample_size, data_value, gender) %>% 
  filter(!is.na(gender)) 
# create gender dataset
brfss_gender = brfss_data %>% 
  select(year, locationabbr, locationdesc, response, sample_size, data_value, race_ethnicity) %>% 
  filter(!is.na(race_ethnicity)) 
```

As we will do our following analysis by age, race, and gender, we created three tidy subsets by age, race, and gender.

NCHS - Injury Mortality: United States
--------------------------------------

Data from the NCHS - Injury Mortality: United States were accessed from data.cdc.gov. This dataset describes injury mortality in the United States beginning in 1999. Two concepts are included in the circumstances of an injury death: intent of injury and mechanism of injury. In our project, we focus on intent of injury, specifically suicide. Data are based on information from all resident death certificates filed in the 50 states and the District of Columbia.

### Data Cleaning

``` r
injury_data = read_csv("./data/NCHS_-_Injury_Mortality__United_States.csv") %>% 
  janitor::clean_names()    # tidy the variable names
injury_tidy = injury_data %>% 
  filter(injury_intent == "Suicide", year %in% c(2011, 2012, 2013, 2014, 2015, 2016),
          sex != 'Both sexes',age_group_years != 'All Ages',race != 'All races',injury_mechanism == 'All Mechanisms') %>% 
  arrange(year) # filtering out data for suicide 
                # filtering only data from 2011-2016 for further comparison analysis with another dataset.
```

The original dataset contains 98280 rows and 17 columns. For further use, we make a preliminary dataset. The preliminary dataset contains 216 rows and 7 columns. Each row includes the information of the mortality rate for a specific age, sex, gender group who attempted suicide during a specific year.

Exploratory analysis:
=====================

Create a new data from the original data just for this section, so that it won't affect other parts of the analysis: For BRFSS dataset, select some variables (year, locationabbr, locationdesc, response, sample\_size, age\_group, gender, race\_ethnicity) that might be useful for later analysis.

For the Injury Mortality dataset, we filter out 'Suicide' as our focus and get rid of summarized rows for age, sex, and race. Since the cases and total population do not differ by 'injury\_mechanism', here we use 'All Mechanisms' to prevent over counting for the population.

Exploring by 'Age'
------------------

![](p8105_fp_ds100_Report_files/figure-markdown_github/BRFSS_IM_year_age-1.png)

The first thing we did is to combine ages to make the two dataset comparable. One defect of this analysis is that the BRFSS data only includes 18-24 age group compared to '&lt; 25' age group in the mortality dataset. If we just look at the other three age groups, for people from 25-64, a high prevalence of depression seems to be consistent with the death rate. However, regardless of the age group '65+' having a relative low depression prevalence, their suicide rate remains relatively high.
When using the 'str\_replace' function for '75+', the result kept giving me an extra '+' at the end of the string. So I set '65+' for the '45–64' group so that they can combine to become the '65+' group.
For both Prevalence and Death Rate, 5% CI were calculated. Usually, large CI stands for a relatively small sample size.
Another thing worth noticing is that in the raw dataset, the '–' in between '45–64' is not the usual '-' in the keyboard, thus I had to copy and paste the symbol to my code.

Exploring by 'Race':
--------------------

![](p8105_fp_ds100_Report_files/figure-markdown_github/BRFSS_IM_year_race-1.png)

Race categories other than White, Black, and Hispanic were combined into 'Other'. Here we see White category has both high depression prevalence as well as suicide rate.

Exploring by 'Gender'
---------------------

![](p8105_fp_ds100_Report_files/figure-markdown_github/BRFSS_IM_year_gender-1.png)

Although female have higher depression prevalence, their suicide rate is much lower than that of the male. Could this give a hint that women are more endurable to depression? We do some researches and find out that women are more likely than men to report suicidal ideation and attempts and to be hospitalized for suicide attempts whereas male tends to choose more lethal methods to commit suicide than female (Vijayakumar,2015).

Regression Model Analysis
-------------------------

To maintain comparability between injury mechanism and depression(brfss) data, we only generate statistic results for:

-   race:
    -   Hispanic*(baseline)*
    -   Non-Hispanic Black
    -   Non-Hispanic White
-   age groups(in years):
    -   &lt; 25*(baseline)*
    -   25-44
    -   45-64
    -   65+
-   sex/gender:
    -   = 0 if male*(baseline)*
    -   = 1 if female
-   year:
    -   2011*(baseline)*
    -   2012
    -   2013
    -   2014
    -   2015
    -   2016

### Suicide rate model

| term                     |  estimate| p.value    |
|:-------------------------|---------:|:-----------|
| (Intercept)              |     7.840| &lt; 0.001 |
| sex: 1                   |   -12.565| &lt; 0.001 |
| race: Non-Hispanic black |    -0.912| 0.399      |
| race: Non-Hispanic white |    10.313| &lt; 0.001 |
| age group years: 25–44   |     7.475| &lt; 0.001 |
| age group years: 45–64   |     7.531| &lt; 0.001 |
| age group years: 65+     |     6.362| &lt; 0.001 |
| year: 2012               |     0.212| 0.89       |
| year: 2013               |     0.215| 0.888      |
| year: 2014               |     0.707| 0.644      |
| year: 2015               |     0.778| 0.611      |
| year: 2016               |     1.050| 0.492      |

### Suicide Model Analysis

Using injury data, we first calculate **suicide death rate = (Deaths  Population) \* 100000** which represents the number of deaths caused by suicide per 100,000 units population. Then, we transform sex, race, age, year into factor variables for future regression model building.

Then, we construct regression for suicide death rate as response, sex, race, age(in years), year as predictor. By observing the coefficients estimates and p-values, there is several interesting findings for suicide death rates:

**Suicide Death Rate = 7.84 - 12.565 I{sex = female} - 0.912 I{race = Non-Hispanic black} + 10.313 I{race = Non-Hispanic white} + 7.745 I{25 &lt; age &lt; 44} + 7.531 I{45 &lt; age &lt; 64} + 6.362 I{age &gt;= 65} + 0.212 I{year = 2012} + 0.215 I{year = 2013} + 0.707 I{year = 2014} + 0.778 I{year = 2015} + 1.05 I{year = 2016}**

1.  Sex has **significant** p-value &lt; 0.0001: There is a significant difference in suicide rate between the male and female group and female tends to have lower suicide death rate.
2.  Race of Non-Hispanic white has **significant** p-value: There is a significant difference of suicide rate between Non-Hispanic Whites and Hispanic and Non-Hispanic Black has a non-significant difference with Hispanic. Whites tend to have higher suicide death rate compared to Hispanic and Blacks.
3.  All groups of age(in years) have **significant** p-value: There is a significant difference between all age groups with baseline age group(age &lt; 25).
4.  All years have **non-significant** p-value: There is not a significant difference between all years with year 2011.
5.  The model produced an adjusted *R*<sup>2</sup> = 0.6405, which represents there are 64% of the variability of the suicide rate are explained by the fitted model and data after adjusted and it's an acceptable proportion for the model.

#### Pairwise comparison

    ##   Tukey multiple comparisons of means
    ##     95% family-wise confidence level
    ## 
    ## Fit: aov(formula = .)
    ## 
    ## $sex
    ##          diff       lwr       upr p adj
    ## 1-0 -12.56459 -14.30119 -10.82799     0
    ## 
    ## $race
    ##                                             diff       lwr       upr
    ## Non-Hispanic black-Hispanic           -0.9120358 -3.458863  1.634791
    ## Non-Hispanic white-Hispanic           10.3133176  7.766490 12.860145
    ## Non-Hispanic white-Non-Hispanic black 11.2253534  8.678526 13.772180
    ##                                           p adj
    ## Non-Hispanic black-Hispanic           0.6751966
    ## Non-Hispanic white-Hispanic           0.0000000
    ## Non-Hispanic white-Non-Hispanic black 0.0000000
    ## 
    ## $age_group_years
    ##                    diff       lwr       upr     p adj
    ## 25–44-<25    7.47502573  4.052757 10.897295 0.0000003
    ## 45–64-<25    7.53077512  4.108506 10.953044 0.0000002
    ## 65+-<25      6.36248653  3.568216  9.156757 0.0000001
    ## 45–64-25–44  0.05574939 -3.895946  4.007445 0.9999824
    ## 65+-25–44   -1.11253920 -4.534808  2.309730 0.8342908
    ## 65+-45–64   -1.16828858 -4.590557  2.253980 0.8130266
    ## 
    ## $year
    ##                  diff       lwr      upr     p adj
    ## 2012-2011 0.212133477 -4.176897 4.601164 0.9999927
    ## 2013-2011 0.215043989 -4.173987 4.604074 0.9999922
    ## 2014-2011 0.706522705 -3.682508 5.095553 0.9973082
    ## 2015-2011 0.778233496 -3.610797 5.167264 0.9957454
    ## 2016-2011 1.049947338 -3.339083 5.438978 0.9831005
    ## 2013-2012 0.002910512 -4.386120 4.391941 1.0000000
    ## 2014-2012 0.494389228 -3.894641 4.883420 0.9995197
    ## 2015-2012 0.566100019 -3.822930 4.955131 0.9990719
    ## 2016-2012 0.837813860 -3.551217 5.226844 0.9939877
    ## 2014-2013 0.491478716 -3.897552 4.880509 0.9995334
    ## 2015-2013 0.563189507 -3.825841 4.952220 0.9990948
    ## 2016-2013 0.834903348 -3.554127 5.223934 0.9940845
    ## 2015-2014 0.071710791 -4.317320 4.460741 1.0000000
    ## 2016-2014 0.343424632 -4.045606 4.732455 0.9999199
    ## 2016-2015 0.271713841 -4.117317 4.660744 0.9999749

Then, we make a pairwise comparison with Bonferroni and Tukey for race, age and year groups. Findings:
1. **Race**: Non-Hispanic White and Hispanic, Non-Hispanic White and Black have significant different suicide death rate. The white has the highest suicide death rate among 3 race groups. And Blacks have lower suicide death rate compared to Hispanic and Whites.
2. **Age**: 25-44 and &lt;25, 45-64 and &lt;25, 65+ and &lt;25 have significant different suicide death rate. Age groups that are &gt;25 all have higher suicide death rate compared to age &lt;25. And age 65+ have lower suicide death rate compared to 25-44, 45-64 groups.
3. **Year**: All pairwise comparison for years don't generate a significant result, meaning there is no significant different suicide death rate in different years.

### Depression model

#### Age model

| term             |   estimate|     p.value|
|:-----------------|----------:|-----------:|
| (Intercept)      |  16.485710|  &lt; 0.001|
| age group: 25-44 |   2.310023|  &lt; 0.001|
| age group: 45-64 |   4.528117|  &lt; 0.001|
| age group: 65+   |  -2.125364|  &lt; 0.001|

##### Pairwise comparison

    ##   Tukey multiple comparisons of means
    ##     95% family-wise confidence level
    ## 
    ## Fit: aov(formula = .)
    ## 
    ## $age_group
    ##                  diff       lwr       upr p adj
    ## 25-44-<25    2.310023  1.598922  3.021124     0
    ## 45-64-<25    4.528117  3.817016  5.239218     0
    ## 65+-<25     -2.125364 -2.944523 -1.306204     0
    ## 45-64-25-44  2.218094  1.642109  2.794080     0
    ## 65+-25-44   -4.435386 -5.140451 -3.730321     0
    ## 65+-45-64   -6.653481 -7.358546 -5.948416     0

#### Model Analysis

Because of the data structure of brfss data, the age, gender, race are independent characteristics of the participants to the study, we have to build 3 seperate model for independent analysis. And the data\_value(in %) represents the proportion of people have depression.

**Age Model**:
**Depression Rate = 16.486 + 2.31 I{25 &lt; age &lt; 44} + 4.528 I{45 &lt; age &lt; 64} - 2.125 I{age &gt;= 65}**

1.  There is significant difference in the depression propotion between age group &lt;25 and each other age group including 25-44, 45-64, 65+.

2.  All pairwise comparison showed a significant p-value between each age groups while 25-44, 45-64 ages showed an increased depression proportion and 65+ showed a decreased depression proportion relative to the reference level being &lt;25 years old.

3.  The adjusted *R*<sup>2</sup> = 0.2617 indicates that 26.17% of variability of the depression proportion is explained by the model only including age groups as its predictor.

#### Gender model

| term        |   estimate|     p.value|
|:------------|----------:|-----------:|
| (Intercept) |  13.660439|  &lt; 0.001|
| gender: 1   |   9.028339|  &lt; 0.001|

    ##   Tukey multiple comparisons of means
    ##     95% family-wise confidence level
    ## 
    ## Fit: aov(formula = .)
    ## 
    ## $gender
    ##         diff      lwr      upr p adj
    ## 1-0 9.028339 8.472382 9.584295     0

#### Model Analysis

**Gender Model**:
**Depression Rate = 13.66 + 9.028 I{gender = female}**

1.  There is significant difference in the depression propotion between male and female. And female indicates a higher depression proportion than male.
2.  The adjusted *R*<sup>2</sup> = 0.6146 indicates that 61.46% of variability of the depression proportion is explained by the model only includes gender as predictor.

#### Race model

    ## Warning: Outer names are only allowed for unnamed scalar atomic inputs

| term                      |   estimate|     p.value|
|:--------------------------|----------:|-----------:|
| (Intercept)               |  17.511454|  &lt; 0.001|
| race: Black, non-Hispanic |  -1.715593|  &lt; 0.001|
| race: White, non-Hispanic |   1.900893|  &lt; 0.001|

    ##   Tukey multiple comparisons of means
    ##     95% family-wise confidence level
    ## 
    ## Fit: aov(formula = .)
    ## 
    ## $race_ethnicity
    ##                                              diff       lwr        upr
    ## Black, non-Hispanic-Hispanic            -1.715592 -2.589486 -0.8416988
    ## White, non-Hispanic-Hispanic             1.900893  1.089433  2.7123529
    ## White, non-Hispanic-Black, non-Hispanic  3.616485  2.766051  4.4669193
    ##                                           p adj
    ## Black, non-Hispanic-Hispanic            1.4e-05
    ## White, non-Hispanic-Hispanic            2.0e-07
    ## White, non-Hispanic-Black, non-Hispanic 0.0e+00

#### Model Analysis

**Race Model**:
**Depression Rate = 17.511 - 1.716 I{race = Black, non-Hispanic} + 1.901 I{race = White, non-Hispanic}**

1.  There is a significant difference in the depression proportion among Hispanic and Non-Hispanic Black, Hispanic and Non-Hispanic White.
2.  All pairwise comparison showed a significant p-value between each race groups while Whites have higher depression proportion than Hispanics and Blacks and Blacks have lower depression proportion than Hispanics.
3.  The adjusted *R*<sup>2</sup> = 0.1081 indicates that 10.81% of the variability of the depression proportion is explained by the model only includes race groups as the predictor.

### Additional Analysis for Location

![](p8105_fp_ds100_Report_files/figure-markdown_github/location%20analysis-1.png)

In addition to investigating the trend of depression prevalence under the category of *"Age"*, *"Race"*, and *"Gender"*, we are also interested in examining if there is a difference in depression proportion across states of the U.S. Based on the graph, a remarkable finding is that the Virgin Islands has the lowest proportion of depression compared to other locations and the difference is quite significant. According to our research, the state ethnicity group consists of over 70% Black race(<https://en.wikipedia.org/wiki/United_States_Virgin_Islands>), and linking back to our visualizations and regression model results, they are consistent with each other since Blacks tend to have the lowest depression prevalence relative the others race groups included in our analysis.

Discussion
==========

From 2011 through 2016, the general trend for depression prevalence in the US increases from 2011 to 2013, then decreases thereafter. While on the other hand, the suicide death rate appears to be increasing each year. Even if we know that depression is just one causal factor for suicide, we expected to see a similar trend for the three groups (age, race, gender).

People older than 65 have a relatively low prevalence of depression but a high suicide rate. On the contrary, Black people have both low depression prevalence and low suicide rate. White people have high depression prevalence and high suicide rate. Compared to men, women are reported to have a higher prevalence of depression while keeping a relatively low death rate of the suicide rate. Any apparent difference of trend on the two plot might suggest other causal factors in the 'causal pies'.

Our results obtained from the regression models agree mostly with the patterns found in the visualization plots by looking at the estimated coefficients for each model parameter and comparing their magnitude and signs with the reference factor level. However, by looking at the fitted regression for suicide death rate, it can be observed that the term 'year' is not a significant predictor and thus should be removed from the model. This means that although suicide death rates appear to be gradually increasing from 2011 to 2016, there is not enough statistical evidence to show that the change is significant. Overall, we recommend the final model to be **Suicide Death Rate ~ Age + Race + Gender**

| term                     |     estimate|     p.value|
|:-------------------------|------------:|-----------:|
| (Intercept)              |    8.3336988|  &lt; 0.001|
| sex: 1                   |  -12.5645905|  &lt; 0.001|
| race: Non-Hispanic black |   -0.9120358|       0.394|
| race: Non-Hispanic white |   10.3133176|  &lt; 0.001|
| age group: 25–44         |    7.4750257|  &lt; 0.001|
| age group: 45–64         |    7.5307751|  &lt; 0.001|
| age group: 65+           |    6.3624865|  &lt; 0.001|

**Suicide Death Rate = 8.334 - 12.565 I{sex = female} - 0.912 I{race = Non-Hispanic black} + 10.313 I{race = Non-Hispanic white} + 7.475 I{25 &lt; age &lt; 44} + 7.531 I{45 &lt; age &lt; 64} + 6.362 I{age &gt;= 65}**

On the other hand, for the simple linear regressions to predict depression prevalence, *"Age"*, *"Race"*, and *"Gender"* are all significant covariates. This supports our initial hypothesis that suicide death rate and depression prevalence do vary among these proposed factors. However, the direct association between depression and suicide may be difficult to test. For rare cases like suicide, a case-control study should be a suitable way to conduct the research by carefully selecting the cases and controls and examining their odds ratio of being exposed to depression. The challenge is that psychological status such as depression is hardly detected accurately, and people may not be willing to report authentic information which potentially leads to bias when analyzing the results. Generally speaking, our project was successful in exploring the relationship between suicide death rate and depression prevalence among *"Age"*, *"Race"*, and *"Gender"*.

Reference
=========

\[1\] American Foundation for Suicide Prevention (2018). Suicide Statistics Retrieved from <https://afsp.org/about-suicide/suicide-statistics/>

\[2\] Vijayakumar L. (2015). Suicide in women. Indian journal of psychiatry, 57(Suppl 2), S233-8.

\[3\] Centers for disease control and prevention. (2018). Behavioral Risk Factor Surveillance System (BRFSS) Prevalence Data (2011 to present) | Chronic Disease and Health Promotion Data & Indicators. Retrieved from <https://chronicdata.cdc.gov/Behavioral-Risk-Factors/Behavioral-Risk-Factor-Surveillance-System-BRFSS-P/dttw-5yxu>

\[4\] National Center for Health Statistics. (2018). NCHS - Injury Mortality: United States | Data. Retrieved from <https://data.cdc.gov/NCHS/NCHS-Injury-Mortality-United-States/nt65-c7a7>
