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

brfss_data = brfss_raw %>% 
   filter(year %in% c(2011, 2012, 2013, 2014, 2015, 2016),
          break_out_category %in% c("Age Group", "Race/Ethnicity", "Gender")) %>% 
   spread(key = break_out_category, break_out)
```

The original dataset contains 1,386,855 rows and 27 column. For further use, we make a preliminary dataset. The preliminary dataset contains 8931 rows and 14 columns. The complete data dictionary is linked above.

Further cleaning
----------------

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

Additional analysis:
====================

If you undertake formal statistical analyses, describe these in detail

Discussion:
===========

What were your findings? Are they what you expect? What insights into the data can you make?
