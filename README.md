# Where Are The Drugs ?
An analysis of prescription drugs.


# Background:

Medicare is a national health insurance program in the United States, begun in 1966 under the Social Security Administration and now administered by the Centers for Medicare and Medicaid Services (CMS).

The Centers for Medicare & Medicaid Services (CMS), is a federal agency within the United States Department of Health and Human Services (HHS) that administers the Medicare program and works in partnership with state governments to administer Medicaid, the Children's Health Insurance Program (CHIP), and health insurance portability standards.

In 2014 the CMS had 1.2 billion claims for medication prescriptions.


![]( https://boomerbenefits.com/wp-content/uploads/2019/04/Parts-of-Medicare.png )


# Medicare Part D

Part D is a federally created program to help you lower the cost of your retail prescription drugs. Unlike Medicare Part A & B, you will not enroll in Part D through the Social Security office. Instead, you will select one of the Part D plans available in your county from private insurance carriers.

By signing up for that plan, you will have enrolled in Part D.

Medicare drug plans are optional. You’ll have a monthly premium that you will pay to the insurance carrier. In return, they give you significantly lower copays on your medicines than you would pay if you had no Part D insurance.


# Why Research Drugs?

The majority of my work experience is in health care. I am facisnated with the complexity of the industry. This is a business of saving lives. Is there any more valuable service? I decided to focus on presciption drugs because I could view the treatments and diseases simultaneously. (ie the treatment indicates the disease)


# Goals

- Discover what effects drug prescription claims
- Discover if population demographics are a factor
 - Gender
 - Race
 - Age
- Discover if geographical location is a factor
 - Coastal vs South
 - Population density



# Sources

![]( https://github.com/Prvargas/Where_Are_The_Drugs/blob/master/Images/sources_pic.png )

### Google Cloud Platform: [<ins>Medications Table<ins>](https://console.cloud.google.com/bigquery?project=bold-lantern-272703&folder=&organizationId=&p=bigquery-public-data&d=cms_medicare&t=physicians_and_other_supplier_2015&page=table)
    
### Census.gov: [<ins>Population Table<ins>](https://www.census.gov/data/tables/time-series/demo/popest/2010s-state-total.html#par_textimage_1574439295)
    
### Census.gov: [<ins>Demographic Table<ins>](https://www.census.gov/data/tables/time-series/demo/popest/2010s-state-detail.html#par_textimage_785300169)


# Importing Packages and Accessing APIs



<!-- #region -->
```python
#Import all neccessary packages and modules
%matplotlib inline
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import scipy as sp
import scipy.stats as stats

plt.style.use('ggplot')

#Gives credentials for api access to GCP
import os
os.environ["GOOGLE_APPLICATION_CREDENTIALS"]='/Users/phillliprashaad/Google_API_Key.json' 

#Connects to medicare data set
import bq_helper
from bq_helper import BigQueryHelper
medicare = bq_helper.BigQueryHelper(active_project="bigquery-public-data",
                                   dataset_name="cms_medicare")
bq_assistant = BigQueryHelper("bigquery-public-data", "cms_medicare")
```
<!-- #endregion -->

# EDA

### Google Cloud Platform
- 23 Tables
- 200+ Features
- 9.5 million datapoints
- Features of interest: State, Drug Name, Total Claims

### Census.gov
- 2 tables
- 70+ Features
- 5k+ datapoints
- Features of interest: State, Estimated Population Size, Gender, Race


<!-- #region -->
# Pipeline
## SQL Query to get total claims by state
<p align='left'>
<img src="https://github.com/Prvargas/Where_Are_The_Drugs/blob/master/Images/SQL_Most_Presc.png" width="600" height="500">
</p>

## SQL Query to get top 5 drugs with highest number of claims
![](https://github.com/Prvargas/Where_Are_The_Drugs/blob/master/Images/SQL_Top5.png)

## Import csv files downloaded from Census
```python
#Import population data into dataframe
population_df = pd.read_csv('/Users/phillliprashaad/Desktop/Galvanzie_DSI/Capstone_1/Health Care/import_tables/region_pop_2.csv')

#Import demographics table
demographic_df = pd.read_csv('/Users/phillliprashaad/Desktop/Galvanzie_DSI/Capstone_1/Where_Are_The_Drugs/import_tables/demographic_by_state.csv')
```
## Create functions that helps merge needed tables on filtered row
```python
def left_df_merge(df1, df2, col1, col2 ):
    '''Helps left merge data frames
    
        df1:  Left Dataframe
        df2:  Right Dataframe
        col1: Left column to merge on
        col2: Right column to merge on
        
        Returns: Merged Dataframe
    '''
    df_merge = df1.merge(df2, how='left', 
                            left_on=col1, 
                            right_on=col2 )
    return df_merge ```


```python 
def filter_DF(df, colname, string):
    '''Searches a specific column to see if it contains a given value
        
        df:       The dataframe to search
        colname:  Name of column to search
        string:   String to search
        
        Returns: Filtered Dataframe
    '''
    mask = df[colname].str.contains(string)
    return df[mask]
    
  ```
  
# More EDA

## Demographic Correlations
<p align='left'>
<img src="https://github.com/Prvargas/Where_Are_The_Drugs/blob/master/Images/Population_Heatmap.jpg" width="600" height="500">
</p>

## Distributions
![](https://github.com/Prvargas/Where_Are_The_Drugs/blob/master/Images/Gender_Rank_Dist.png)


![](https://github.com/Prvargas/Where_Are_The_Drugs/blob/master/Images/LEVO_LISI_Dist.png)
<!-- #endregion -->

# Hypothesis Test

1. Define Null and Alternative Hypotheses
2. State Alpha
3. Calculate Degrees of Freedom
4. State Decision Rule
5. Calculate Test Statistic
6. State Results
7. State Conclusion

## Spearman Correlation r-test
1. There is NO relationship between gender and drug rank for drug LEVOTHYROXINE SODIUM
2. Alpha = .01
3. DOF = 47-2= 45
4. If r > .372, then REJECT Null
5. r(45) = .433
6. r(45) = .433 > .372; REJECT NULL 
7. There is a correlation with 99% CI

<p align='left'>
<img src="https://github.com/Prvargas/Where_Are_The_Drugs/blob/master/Images/Correlation_graph.png" width="600" height="500">
</p>

## Welch's t-test
1. There is NO significant difference in claims percentage between the drugs LEVOTHYROXINE SODIUM and LISINOPRIL
2. Alpha = .01
3. n/a
4. If pval < alpha, then REJECT Null
5. pval = .0007
6. pval = .0007 < .01; REJECT NULL 
7. With a 99% CI There is a significant difference

<p align='left'>
<img src="https://github.com/Prvargas/Where_Are_The_Drugs/blob/master/Images/LEVO_VS_LISI_Dist.png" width="600" height="500">
</p>

# IF I HAD MORE TIME!!

Due to time constraints I was unable to dive deeper into this data. Different people have different cultures and habits which lead lead to different health levels. Also, certain environments effect the human body differently. Not to mention certain groups tend to cluster in different areas for various reasons. If I had more time, then I would like to look at race and geographical location as variables for drug prescription type and use. 
