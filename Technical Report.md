# UFC Capstone Technical Report

## Executive Summary

### What is the goal of my project?
The goal of my project was to build a model that can predict the winner of a UFC fight based on current and past metrics for each fighter.

### Where did I get the data?
 1. V1 JSON API
 2. V2 JSON API
 3. UFC Fighter Page

The APIs were FightMetric.com API used by the UFC website to populate their fight stats for each fight. [Eample of UFC Fight Stats](http://www.ufc.com/event/UFC-225)

The [UFC fighter page](http://www.ufc.com/fighter/robert-whittaker) is the overview page for each fighter on www.ufc.com

### What were my metrics?
Since I am just trying to predict the winner of the fight, I am using accuracy as my metric.

### What were my findings?
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

### What risks/limitations/assumptions affect these findings?
- The biggest risk/limitation for my findings were that I was only able to get a limited number of fights that had enough data to model.
- The V1 and V2 jsons were not available for earlier fight data.  The earliest fight in my data was in 2013.  For many fighters, the fight data that I had only covered the most recent 2/3rds to 1/2 of their total fights.
- I started with 4700 fights with fight details, however since we need to calculate the historical average, the first fight for a fighter in the dataset had to be dropped.
- Any fighter with only 1 fight basically didn't have any training data included.
- After munging the data, I only had 3200 rows of data with each row representing a single fight
- Since the Jsons were limited, I was not able to get time in position data, leaving out a data that is significant for predictions.
- Since not all of the fights for each fighter was available, I was unable to calculate an accurate win/loss ratio which was likely a strong predictor.

## Summary of statistical analysis

#### Implementation
I trained logistic regression and random forest models using the raw dataset with about 700 features, a dataset with the 100 best features selected and a dataset with 100 best features put through PCA.  I ran both models through GridSearch.

#### Best model for logistic regression:
- Data:  100 best features selected using Select K Best
- Hyper Parameters: {'C': 0.01, 'max_iter': 2000, 'penalty': 'l2'}
- Test Score: 0.588

#### Best model for random forest:
- Data: Full dataset with ~700 features
- Hyper Parameters: {'max_depth': 5, 'min_samples_split': 2, 'n_estimators': 1000}
- Test Score: 0.607

#### Analysis
Logistic regression performed better on the dataset with the 100 best features since there was less noise in the data.  Grid Search chose to use Ridge Regularization which is not surprising since the features were already selected so Lasso would be somewhat redundant.  

Random forest performed actually about the same on the dataset with the full feature set and the dataset with the 100 best features.  This is because random forest can basically "select features" for me by just choosing to split on the best features anyway.  It means that the Select K Best did select the same features as random forest decided to split on when given the entire feature space.

With regularization and limiting the max depth and min sample split for the models, the final models chosen were not overfit.



# Project Sections

## Notebooks - 01_Data_Collection
This section contains all the notebooks used for scraping the data from the UFC website and fightMetric APIs.  The notebooks are split up by the target of the scrape with different notebooks for scraping the events list, the V1 API, the V2 API and the individual UFC fighter webpages.

## Notebooks - 02_Data_Munging
This section contains all the notebooks used for munging the data from the raw formats into dataframes and additional calculations and transformations to convert the data into a format that can be used to train a model.  

I've also included my "EDA Along The Way" notebook in this folder.  I aggregated my exploratory analysis into this notebook.  In some instances, other EDA is sprinkled in with the transformations and munging as necessary.

## Notebooks - 03_Data_Modeling
This section contains all the notebooks used for creating the predictive models.  One notebook includes all of my baseline models.  I created separate notebooks for all the iterations for the logistic regression and random forest models.


# In-depth explanation of data and necessary transformations

## Description of data in each source

**V1 JSON API:**
- The parameters for the V1 API is the EventID and follows the structure:  http://liveapi.fightmetric.com/V1/{EventID}/Fnt.json
- Each JSON contains data for each fight for the specified EventID
- Relevant fields that were scraped include:  
    - FightID for fight on the card
    - Max rounds possible and total rounds actually fought
    - Fight Outcome: win, loss, no contest, disqualification
    - Finish method:  knockout, submission, decision
  
    For each Fighter:
    - FighterID and Name
    - Birthdate
    - Height, Weight
    - Fighting Stance
    - Location fighter fights out of
    - Location fighter born
    - Total Wins, Losses, Ties

**V2 JSON API:**
- The parameters for the V2 API is the EventID and FightID and follows the structure:
    http://liveapi.fightmetric.com/V2/{EventID}/{FightID}/Stats.json
- Each JSON contains fight stats for striking, grappling, and time in position data for the specified fight
- The fight stats were provided at an aggregated level as well as for each round
- The strikes were broken down into more granularity from various positions or based on the target of the strike as well as separate figures for the number "attempted" and "landed"
    - Examples:  body_significant_strikes_attempts, clinch_body_strikes_landed, head_total_strikes_landed
- The time in position stats were the number of minutes and seconds in a specific position
- Since many of these were null, I dropped the time in position columns

**UFC Fighter Page:**
- Each fighter has their own Fighter Page which shows their stats (height, weight, reach, leg reach) as well as their fight history
- An example of a UFC fighter page:  http://www.ufc.com/fighter/robert-whittaker
- I scraped this page to get the fighters height, weight, reach and leg_reach because I believe these physical attributes can be used to help predict the winner of fights

## Data Munging,  Cleansing, Feature Engineering

The collected data was quite messy with many missing values and differing formats.  
- I flattened the JSONs and put the relevant data into a dataframe. 
- The V2 Jsons were "In Fight" statistics, which cannot be used to directly predict fight outcomes since the actual number of strikes thrown or time in certain positions is not avaialble before the fight actually occurs.
- In a separate notebook, I processed and transformed the V2 df into the historical averages for each fight
- Overall thought process and reasoning for the transformations:  (F1 = Fighter 1, F2 = Fighter 2)
    - Each row of data in the V2 dataframe is a fight from the perspective of the F1 fighter.
    - We can think of the F1 columns as the "offensive" statistics for the fighter
    - We can think of the F2 columns as the inverse "defensive" statistics for the fighter.  
        - A low striking percentage for F2 means that F1 was good at avoiding strikes
    - The data shows the absolute number attempted and landed.  It is also important to calcuate the percentage to get a sense of a fighters accuracy/precision.
    - In addition, we also want to know the differential between the fight stats for F1 and F2 since fights are relative.  A low striking fight may not be as bad if the opponent also threw very few strikes.  This could indicate that maybe most of the fight was on the ground, instead of indicating the fighters are poor strikers.

### Transformations done for each fight
- Calculated the success percentage for each fight statistic for a single fight:  
	- **Example:** Strikes_Landed / Strikes_Attempted => Strikes_Landed_Percent
	- **Why this matters:**  The percent landed is important as it gives us information on how accurate a fighter is
- Calculated the fight statistics differential between F1 and F2.  This includes differentials for percentage landed
	- **Example:**  F1_Strikes_Landed - F2_Strikes_Landed = Strikes_Landed_Diff
	- **Why this matters:**  This allows us to capture how well the fighter did in relation to their opponent
        
- Calculating Historical Averages:  Once all of the transformations have been done on a fight by fight basis, then I calculated the historical average of each metric for each fight.
	- The data up to this point represents single fights.  This data cannot be put into a model since this type of "in fight" data is not available before a fight occurs.
	- In order to overcome this, we calculate the historical average for each of these metrics in the fights leading up to the fight being predicted.


### Explaining the differences between f1 and f2
- At this point we have historical averages for each fight metric as well as percentage landed and difference for f1 and f2.  Intuitively, f1 represents fighter 1 and f2 represents fighter 2.  However, once we have calculated the historical averages, the f2 stats no longer represent the stats for a single fighter.  They are the average of the each of the opponents for f1's past fights.  This tells us less about the opponents, and more about f1's defensive capabilities (or actually its inverse).  For example, if the f2 strikes landed percentage is high, it can mean that all the opponents have good striking accuracy.  But more likely it means that f1 has poor dodging ability. 

### One more transformation to prepare the data for modeling
- Since the historical averages really only give us information about the f1 fighter, we need to do another transformation to bring all of the data for fighter 1 and fighter 2 into a single dataframe.  In notebook 9, I first use the V1 data and pull all the relevant keys (eventid, fightid, figherid for f1 and f2) and fight outcome into a "stub".  All other data will be joined to this stub.  Then I joined static fighter data and the transformed historical averages to the stub.  At this point we have for each fighter - their static information (height, reach, etc) as well as historical infight averages for both offensive ability (f1) and defensive ability (f2).  
- As the data is merged for fighter 1 and fighter 2, I appended an additional "f1" for fighter 1 and "f2" for fighter 2, thus when looking at the training data, it is important to keep the following in mind:
	- f1_f1 = fighter 1's offensive metrics
	- f1_f2 = fighter 1's defensive metrics
	- f2_f1 = fighter 2's offensive metrics
	- f2_f2 = fighter 2's defensive metrics

Since I also calculated differences in metrics (_diff), those did not have a remaining f1 or f2 tag, so will show up with only an f1 or f2.





