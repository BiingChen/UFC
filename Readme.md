# UFC Capstone


### Goal of the project:
The goal of my project was to build a model that can predict the winner of a UFC fight based on current and past metrics for each fighter.

### Data sources:
 1. V1 JSON API
 2. V2 JSON API
 3. UFC Fighter Page


### Findings:
I trained 2 types of model: Logistic Regression and Random Forest.  Logistic Regression was able to achieve a 0.58 accuracy and Random Forest 0.60 accuracy prediction.

**For the logistic regression, the most important metrics were:**
|                   Feature                  | Coefficient |
|:------------------------------------------:|-------------|
| f1_reach_adv                               | 0.044867    |
| f1_grappling_submissions_attempts_avg_diff | 0.044572    |
| f1_knock_down_landed_avg_diff              | 0.041376    |
| f2_total_strikes_landed_avg_diff           | -0.022935   |
| f2_head_total_strikes_landed_avg_diff      | -0.023089   |
| f1_f2_clinch_head_strikes_landed_avg       | -0.038542   |


**For random forest, the top 10 important features were:**
| Feature                                       | Importance |
|-----------------------------------------------|------------|
| f2_head_significant_strikes_landed_diff_avg   | 0.031720   |
| f2_head_significant_strikes_percent_avg_diff  | 0.030295   |
| f1_head_significant_strikes_landed_avg_diff   | 0.029473   |
| f1_significant_strikes_landed_diff_avg        | 0.028200   |
| f1_f2_clinch_head_strikes_percent_avg         | 0.024601   |
| f1_significant_strikes_attempts_diff_avg      | 0.023133   |
| f1_head_significant_strikes_landed_diff_avg   | 0.023019   |
| f2_significant_strikes_landed_diff_avg        | 0.022822   |
| f1_distance_head_strikes_landed_avg_diff      | 0.021123   |
| f2_head_significant_strikes_attempts_diff_avg | 0.020115   |



# Project Sections

## Notebooks - 01_Data_Collection
This section contains all the notebooks used for scraping the data from the UFC website and fightMetric APIs.  The notebooks are split up by the target of the scrape with different notebooks for scraping the events list, the V1 API, the V2 API and the individual UFC fighter webpages.

## Notebooks - 02_Data_Munging
This section contains all the notebooks used for munging the data from the raw formats into dataframes and additional calculations and transformations to convert the data into a format that can be used to train a model.  

I've also included my "EDA Along The Way" notebook in this folder.  I aggregated my exploratory analysis into this notebook.  In some instances, other EDA is sprinkled in with the transformations and munging as necessary.

## Notebooks - 03_Data_Modeling
This section contains all the notebooks used for creating the predictive models.  One notebook includes all of my baseline models.  I created separate notebooks for all the iterations for the logistic regression and random forest models.


**For more detail, please see the Technical Report**


