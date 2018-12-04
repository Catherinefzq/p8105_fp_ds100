p8105\_fp\_ds100\_Report
================
Yue Gu, Jianghui Lin, Junyuan Zheng, Jianyou Liu, Zhiqian Fang
12/3/2018

Motivation:
===========

Suicide is a leading cause of death in the US. Suicide rates increased in nearly every state from 1999 through 2016. Mental health conditions are often seen as the cause of suicide, but suicide is rarely caused by any single factor. In fact, many people who die by suicide are not known to have a diagnosed mental health condition at the time of death. We are interested in examining the variations of suicide death rates among different categorical variables such as gender, race, age group, and to test if the observed differences are statistically significant combined with the analysis to age-specific rate and related confidence interval.

Related work:
=============

Anything that inspired you, such as a paper, a web site, or something we discussed in class.

Suicide has ranked as the 10th leading cause of death among Americans for many years. <https://www.verywellmind.com/suicide-rates-overstated-in-people-with-depression-2330503>

Initial questions:
==================

The initial thought of this project is to discover the relationship between suicide and people's mental status. We have found two separated datasets, in which one is the Injury Mortality data in the US, the other contains people's depression status in the US. Since both of the two datasets contains information for age, race, and gender, we decided to compare the trend of suicide death rates with depression prevalence under these three categories.

Data:
=====

Source, scraping method, cleaning, etc.

BRFSS Prevalence Data (2011 to present)
---------------------------------------

Data from the Behavioral Risk Factor Surveillance System (BRFSS) Prevalence Data (2011 to present) were accessed from cdc.gov.(<https://chronicdata.cdc.gov/Behavioral-Risk-Factors/Behavioral-Risk-Factor-Surveillance-System-BRFSS-P/dttw-5yxu>) The version of the data that we will use in this analysis can be found in our Github (./data).

Methodology: <http://www.cdc.gov/brfss/factsheets/pdf/DBS_BRFSS_survey.pdf>

Glossary: <http://apps.nccd.cdc.gov/BRFSSQuest/index.asp>

### Data acquisition and description

As the original dataset is too large, we download the dataset and acquire the data we need. The specific data to be used in this project was accessed in December 2018 using the code below.

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

The original dataset contains 1,386,855 rows and 27 column. For further use, we make a preliminary dataset. The preliminary dataset contains 8931 rows and 14 columns. We save the preliminary dataset in our data file. The complete data dictionary is linked above.

Further cleaning
----------------

``` r
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.0.0     ✔ purrr   0.2.5
    ## ✔ tibble  1.4.2     ✔ dplyr   0.7.6
    ## ✔ tidyr   0.8.1     ✔ stringr 1.3.1
    ## ✔ readr   1.1.1     ✔ forcats 0.3.0

    ## ── Conflicts ───────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
# load the preliminary dataset
brfss_data = read.csv("./data/brfss_data.csv") 
# method 1 - select the columns we need
brfss_tidy = brfss_data %>% 
  select(year, locationabbr, locationdesc, response, sample_size, data_value, Age.Group, Gender, Race.Ethnicity, geo_location) 
# method 2 - create age dataset
brfss_age = brfss_data %>% 
  select(year, locationabbr, locationdesc, response, sample_size, data_value, Age.Group) %>% 
  filter(!is.na(Age.Group)) %>% 
  janitor::clean_names()
# create race dataset
brfss_race = brfss_data %>% 
  select(year, locationabbr, locationdesc, response, sample_size, data_value, Gender) %>% 
  filter(!is.na(Gender)) %>% 
  janitor::clean_names()
# create gender dataset
brfss_gender = brfss_data %>% 
  select(year, locationabbr, locationdesc, response, sample_size, data_value, Race.Ethnicity) %>% 
  filter(!is.na(Race.Ethnicity)) %>% 
  janitor::clean_names()
```

As we will do our followin analysis by age, race, and gender, we created three tidy subsets by age, race, and gender.

``` r
 injury_data = read_csv("./data/NCHS_-_Injury_Mortality__United_States.csv") %>% 
  janitor::clean_names() %>%   # tidy the variable names
  #distinct(year, sex,race,age_group_years, injury_intent, .keep_all = TRUE) %>% 
  filter(injury_intent == "Suicide", year %in% c(2011, 2012, 2013, 2014, 2015, 2016),
          sex != 'Both sexes',age_group_years != 'All Ages',race != 'All races',injury_mechanism == 'All Mechanisms') %>% 
  arrange(year) # filtering out data for suicide 
```

    ## Parsed with column specification:
    ## cols(
    ##   Year = col_integer(),
    ##   Sex = col_character(),
    ##   `Age group (years)` = col_character(),
    ##   Race = col_character(),
    ##   `Injury mechanism` = col_character(),
    ##   `Injury intent` = col_character(),
    ##   Deaths = col_integer(),
    ##   Population = col_integer(),
    ##   `Age Specific Rate` = col_double(),
    ##   `Age Specific Rate Standard Error` = col_double(),
    ##   `Age Specific Rate Lower Confidence Limit` = col_double(),
    ##   `Age Specific Rate Upper Confidence Limit` = col_double(),
    ##   `Age Adjusted Rate` = col_double(),
    ##   `Age Adjusted Rate Standard Error` = col_double(),
    ##   `Age Adjusted Rate Lower Confidence Limit` = col_double(),
    ##   `Age Adjusted Rate Upper Confidence Limit` = col_double(),
    ##   Unit = col_character()
    ## )

``` r
                # filtering only data from 2011-2016 for further comparison analysis with another dataset.
```

The original dataset contains 98280 rows and 17 columns. For further use, we make a preliminary dataset. The preliminary dataset contains 504 rows and 7 columns. Each row includes the information of the mortality rate for specific age,sex,gender group who attempted suicide during specific year.

Exploratory analysis:
=====================

Visualizations, summaries, and exploratory statistical analyses. Justify the steps you took, and show any major changes to your ideas.

``` r
library(tidyverse)
library(plotly)
library(patchwork)
data_BRFSS = 
  read_csv(file = './data/brfss_data.csv')
data_IM = 
  read_csv(file = './data/NCHS_-_Injury_Mortality__United_States.csv')
```

Creat a new data from the original data just for this section, so that it won't affect other part of the analysis: For BRFSS dataset, select some variables that might be usful for later analysis:

``` r
data_BRFSS_JZ = data_BRFSS %>% 
  janitor::clean_names(.) %>% 
  select(., year, locationabbr, locationdesc, response, sample_size, age_group, gender, race_ethnicity)
```

For the Injury Mortality dataset, filter out 'Suicide' as our focus. Get rid of summarized rows for age, sex and race. Since the cases and total population do not differ by 'injury\_mechanism', here I use 'All Mechanisms' to prevend over counting for the population.

``` r
data_IM_JZ = data_IM %>% 
  janitor::clean_names(.) %>% 
  filter(., injury_intent == 'Suicide',
    year == 2011 | year == 2012 | year == 2013 | year == 2014 | year == 2015 | year == 2016,
    sex != 'Both sexes',
    age_group_years != 'All Ages',
    race != 'All races',
    injury_mechanism == 'All Mechanisms')
```

**Exploring by 'Age':**

``` r
BRFSS_age_plot = 
data_BRFSS_JZ %>%
  mutate(., age_group = str_replace(age_group, '25-34', '25-44'),
            age_group = str_replace(age_group, '35-44', '25-44'),
            age_group = str_replace(age_group, '45-54', '45-64'),
            age_group = str_replace(age_group, '55-64', '45-64')) %>% 
  filter(., age_group != 'NA') %>% 
  group_by(., year, response, age_group) %>% 
  summarize(., sum_sample_size = sum(sample_size)) %>% 
  spread(., key = response, value = sum_sample_size) %>% 
  mutate(., prevalence = (Yes / (Yes + No)),
    ci_low = prevalence - qnorm(.975) * sqrt(prevalence * (1 - prevalence) / (Yes + No)),
    ci_high = prevalence + qnorm(.975) * sqrt(prevalence * (1 - prevalence) / (Yes + No))) %>% 
  ggplot(., aes(x = year, y = prevalence, color = age_group)) +
  geom_point() +
  geom_line() +
  geom_errorbar(aes(ymax = ci_high, ymin = ci_low), width = 0.3, alpha = 0.5) + 
  theme(legend.position = 'bottom', legend.text = element_text(size=6), legend.box = 'vertical', legend.key.size = unit(0.5, 'cm')) +
  ggtitle('Depression Prevalence by Age Groups')

IM_age_plot = 
data_IM_JZ %>% 
  mutate(., age_group_years = str_replace(age_group_years, '< 15', '< 25'),
            age_group_years = str_replace(age_group_years, '15–24', '< 25'),
            age_group_years = str_replace(age_group_years, '65–74', '65+'),
            age_group_years = str_replace(age_group_years, '75+', '65'),
         age_group = age_group_years) %>% 
  group_by(., year, age_group) %>% 
  summarize(., deaths = sum(deaths), population = sum(population)) %>% 
  mutate(., death_rate = (deaths / population),
    ci_low = death_rate - qnorm(.975) * sqrt(death_rate * (1 - death_rate) / population),
    ci_high = death_rate + qnorm(.975) * sqrt(death_rate * (1 - death_rate) / population)) %>%
  mutate(., age = forcats::fct_relevel(age_group, c('< 25', '25–44', '45–64', '65+'))) %>% 
  ggplot(., aes(x = year, y = death_rate, color = age_group)) +
  geom_point() +
  geom_line() +
  geom_errorbar(aes(ymax = ci_high, ymin = ci_low), width = 0.3, alpha = 0.8) + 
  theme(legend.position = 'bottom', legend.text = element_text(size=6), legend.box = 'vertical', legend.key.size = unit(0.5, 'cm')) +
  ggtitle('Suicide Death Rate by Age')

BRFSS_age_plot + IM_age_plot
```

![](p8105_fp_ds100_Report_files/figure-markdown_github/BRFSS_IM_year_age-1.png) First thing I did is to combine ages to make the two dataset compariable. One defect of this analysis is that the BRFSS data only includes 18-24 age group compared to '&lt; 25' age group in the mortality dataset. If we just look at the other three age groups, for people from 25-64, high prevalence of depression seems to be consistant with suicide death rate. However, regardless of age group '65+' having a relative low depression prevalence, their suicide rate remains relatively high.
When using the 'str\_replace' function for '75+', the result kept giving me an extra '+' at the end of the string. So I set '65+' for the '45–64' group so that they can combine to become the '65+' group.
For both Prevalence and Death Rate, 5% CI were calculated. Usually large CI stands for a relatively small sample size.
Another thing that is worth noticing is that in the raw dataset, the '–' in between '45–64' is not the usual '-' in the keyboard, So I had to copy and paste the symble to my code.
**Exploring by 'Race':**

``` r
BRFSS_race_plot =
data_BRFSS_JZ %>% 
  mutate(., race_ethnicity = str_replace(race_ethnicity, 'American Indian or Alaskan Native, non-Hispanic', 'Other'),
            race_ethnicity = str_replace(race_ethnicity, 'Asian, non-Hispanic', 'Other'),
            race_ethnicity = str_replace(race_ethnicity, 'Multiracial, non-Hispanic', 'Other'),
            race_ethnicity = str_replace(race_ethnicity, 'Native Hawaiian or other Pacific Islander, non-Hispanic', 'Other'),
            race_ethnicity = str_replace(race_ethnicity, 'Other, non-Hispanic', 'Other')) %>%  
  filter(., race_ethnicity != 'NA') %>% 
  group_by(., year, response, race_ethnicity) %>% 
  summarize(., sum_sample_size = sum(sample_size)) %>% 
  spread(., key = response, value = sum_sample_size) %>% 
  mutate(., prevalence = (Yes / (Yes + No)),
    ci_low = prevalence - qnorm(.975) * sqrt(prevalence * (1 - prevalence) / (Yes + No)),
    ci_high = prevalence + qnorm(.975) * sqrt(prevalence * (1 - prevalence) / (Yes + No))) %>%
  mutate(., race_ethnicity = forcats::fct_relevel(race_ethnicity,
                     c('Hispanic', 'Black, non-Hispanic', 'White, non-Hispanic', 'Other'))) %>%
  ggplot(., aes(x = year, y = prevalence, color = race_ethnicity)) +
  geom_point() +
  geom_line() +
  geom_errorbar(aes(ymax = ci_high, ymin = ci_low), width = 0.1, alpha = 0.8) + 
  theme(legend.position = 'bottom', legend.text = element_text(size=5), legend.box = 'vertical', legend.key.size = unit(0.4, 'cm')) +
  ggtitle('Depression Prevalence by Race')

IM_race_plot =
data_IM_JZ %>% 
  group_by(., year, race) %>%
  summarize(., sum_deaths = sum(deaths), population = sum(population)) %>% 
  mutate(., death_rate = (sum_deaths / population),
    ci_low = death_rate - qnorm(.975) * sqrt(death_rate * (1 - death_rate) / population),
    ci_high = death_rate + qnorm(.975) * sqrt(death_rate * (1 - death_rate) / population)) %>% 
  ggplot(., aes(x = year, y = death_rate, color = race)) +
  geom_point() +
  geom_line() +
  geom_errorbar(aes(ymax = ci_high, ymin = ci_low), width = 0.5, alpha = 0.5) + 
  theme(legend.position = 'bottom', legend.text = element_text(size=5), legend.box = 'vertical', legend.key.size = unit(0.4, 'cm')) +
  ggtitle('Suicide Death Rate by Age')

BRFSS_race_plot + IM_race_plot
```

![](p8105_fp_ds100_Report_files/figure-markdown_github/BRFSS_IM_year_race-1.png) Race categories other than White, Black, and Hispanic were combined into 'Other'. Here we see White category has both high depression prevalence as well as suicide rate.

**Exploring by 'Gender':**

``` r
BRFSS_gender_plot = 
data_BRFSS_JZ %>% 
  filter(., gender != 'NA') %>% 
  group_by(., year, gender, response) %>% 
  summarize(., sum_sample_size = sum(sample_size)) %>% 
  spread(., key = response, value = sum_sample_size) %>% 
  mutate(., prevalence = (Yes / (Yes + No)),
    ci_low = prevalence - qnorm(.975) * sqrt(prevalence * (1 - prevalence) / (Yes + No)),
    ci_high = prevalence + qnorm(.975) * sqrt(prevalence * (1 - prevalence) / (Yes + No))) %>% 
  ggplot(., aes(x = year, y = prevalence, color = gender)) +
  geom_point() +
  geom_line() +
  geom_errorbar(aes(ymax = ci_high, ymin = ci_low), width = 0.5, alpha = 1) +
  theme(legend.position = 'bottom', legend.text = element_text(size=6), legend.box = 'vertical', legend.key.size = unit(1, 'cm')) +
  ggtitle('Depression Prevalence by Genders')

IM_gender_plot = 
data_IM_JZ %>% 
  group_by(., year, sex) %>% 
  summarize(., sum_deaths = sum(deaths), population = sum(population)) %>% 
  mutate(., death_rate = (sum_deaths / population),
    ci_low = death_rate - qnorm(.975) * sqrt(death_rate * (1 - death_rate) / population),
    ci_high = death_rate + qnorm(.975) * sqrt(death_rate * (1 - death_rate) / population)) %>% 
  ggplot(., aes(x = year, y = death_rate, color = sex)) +
  geom_point() +
  geom_line() +
  geom_errorbar(aes(ymax = ci_high, ymin = ci_low), width = 0.5, alpha = 0.8) + 
  theme(legend.position = 'bottom', legend.text = element_text(size=6), legend.box = 'vertical', legend.key.size = unit(1, 'cm')) +
  ggtitle('Suicide Death Rate by Age')

BRFSS_gender_plot + IM_gender_plot
```

![](p8105_fp_ds100_Report_files/figure-markdown_github/BRFSS_IM_year_gender-1.png) Surprisingly, even female appears to have more depression, their suicide rate is much lower than male. Could this give a hint that women are more endurable to depression?

Additional analysis:
====================

If you undertake formal statistical analyses, describe these in detail

Discussion:
===========

What were your findings? Are they what you expect? What insights into the data can you make?

From 2011 through 2016, the general depression prevalence in US goes up then goes down again. On the other hand, the suicide rate has been increasing each year. Even we know that depression is just one causal factor to suicide, we expected seeing similar trend for the three groups (age, race, gender). The result are generally as expected while there are some noticable ones.
People older than 65 have a relatively low prevalence of depression while still keep a high suicide rate. In contrary to Black people who have both low depression prevalence and suicide rate, White people are high in both of the statistics. Women are reported to have higher prevalence of depression compared to men, while keep a relatively low death rate of suicide rate. Any apparent difference of trend on the two plot might suggest other causal factors in the 'causal pies'. However, the association between depression and suicide is kind of hard to test. For rare cases like suicide, case-control study should be a suitable way to conduct the research, while the psychological status like depression are really complicated to detect, especially for suicide cases.
