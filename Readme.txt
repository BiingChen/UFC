

Overview of the process:


I. Started off by scraping data from a variety of sources:
1. V1 JSON API
2. V2 JSON API
3. UFC Fighter Page

- The APIs were FightMetric.com API used by the UFC website to populate their fight stats for each event.  
- The UFC fighter page is the overview page for each fighter on www.ufc.com


Description of data in each source

V1 JSON API:
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

- There were a lot of nulls and missing data that had to be cleaned up


V2 JSON API:
- The parameters for the V2 API is the EventID and FightID and follows the structure:
    http://liveapi.fightmetric.com/V2/{EventID}/{FightID}/Stats.json
- Each JSON contains fight stats for striking, grappling, and time in position data for the specified fight
- The fight stats were provided at an aggregated level as well as for each round
- The strikes were broken down into more granularity from various positions or based on the target of the strike as well as separate figures for the number "attempted" and "landed"
    - Examples:  body_significant_strikes_attempts, clinch_body_strikes_landed, head_total_strikes_landed
- The time in position stats were the number of minutes and seconds in a specific position
- Since many of these were null, I dropped these columns

UFC Fighter Page:
- Each fighter has their own Fighter Page which shows their stats (height, weight, reach, leg reach) as well as their fight history
- An example of a UFC fighter page:  http://www.ufc.com/fighter/robert-whittaker
- I scraped this page to get the fighters height, weight, reach and leg_reach because I believe these physical attributes can be used to help predict the winner of fights

II. Data Munging and Cleansing

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

- Summary of transformations done for each fight:
    - Calculated the success percentage for each fight statistic for a single fight:  
        - Example: Strikes_Landed / Strikes_Attempted => Strikes_Landed_Percent
        - Why this matters:  The percent landed is important as it gives us information on how accurate a fighter is
    - Calculate the fight statistics differential between F1 and F2.  This includes differentials for percentage landed
        - Example:  F1_Strikes_Landed - F2_Strikes_Landed = Strikes_Landed_Diff
        - Why this matters:  This allows us to capture how well the fighter did in relation to their opponent
        
- Calculating Historical Averages:
    - Once all of the transformations have been done on a fight by fight basis, then I calculated the historical average of each metric for each fight.
    - This allows the data to be put into the model since it avoids using the "in fight" data in the training and predictions.

- Calculating time in position (tip):
    - I originally wanted to also calculate the time in position ratio for each fight and get the average historical tip
    - This would allow us to get a sense of the style of each fighter:  striker vs. grappler
    - Unfortunately, much of the tip data was nulls so I had to drop them.
    - If I ever get access to a cleaner set of data, this is something I would build in

What features were used in the model

First, an overview of the data collected
- The data collected can be broken down into a few different categories.  At the most fundamental level fighters can be thought to have "in fight" data as well as "static" or "out of fight" data.  Static data is information about a fighter that doesn't change from fight to fight.  For example, this includes their height, weight, reach and birth place.  In fight data is information for a fighter for a specific fight, against a specific opponent, during a specific event.  Examples of "in fight" data are the number of strikes attempted and landed during the fight, the number of take downs attempted and landed, the number of minutes the fighter spent in a specific position.  

- The data collected all falls into one of the 2 categories outlined above.  However, in its raw form it cannot be used to train a model because when trying to predict a new fight, the in-fight data not available until after the fight has completed.  In order to deal with this, we instead calculate the historical average for each fight statistic for all the fights leading up to the fight being predicted.


- Before calculating the historical averages for each fight stat, I first did a number of additional data transformations.  First, the data includes "attempts" as well as "landed".  I first calculated the percentage landed, since this figure is can also be important relative to the absolute number attempted and landed.  This was done for both fighter 1 (f1) as well as fighter 2 (f2). 

- Next, I calculated the ratio of the metrics for each fight for each metric using the formula:  (f1 - f2) / (f1 + f2).  This was done because the ratio of each metric compared to their opponent is also important.  One would assume that landing a lot of strikes against an opponent is good.  But there are some fighters that tend to throw a lot strikes and also take a lot of strikes.  By calculating the ratio, it allows the model to get a better sense of how well the fighter was actually doing against their opponent.

- Finally, I calculate the historical average for every fight static including the additional transformations.  

Explaining the differences between f1 and f2.  
- At this point we have historical averages for each fight metric as well as percentage landed and difference for f1 and f2.  Intuitively, f1 represents fighter 1 and f2 represents fighter 2.  However, once we have calculated the historical averages, the f2 stats no longer represent the stats for a single fighter.  They are the average of the each of the opponents for f1's past fights.  In essense this tells us less about the opponents, and more about f1's defensive capabilities (or actually its inverse).  For example, if the f2 strikes landed percentage is high, it can mean that all the opponents have good striking accuracy.  But more likely it means that f1 has poor dodging ability. 

One more transformation to prepare the data for modeling
- Since the historical averages really only give us information about the f1 fighter, we need to do another transformation to bring all of the data for fighter 1 and fighter 2 into a single dataframe.  In notebook 9, I first use the V1 data and pull all the relevant keys (eventid, fightid, figherid for f1 and f2) and fight outcome into a "stub".  All other data will be joined to this stub.  Then I joined static fighter data and the transformed historical averages to the stub.  At this point we have for each fighter - their static information (height, reach, etc) as well as historical infight averages for both offensive ability (f1) and defensive ability (f2).  I appended an additional "f1" for fighter 1 and "f2" for fighter 2, thus when looking at the training data, it is important to keep the following in mind:
- f1_f1 = fighter 1's offensive metrics
- f1_f2 = fighter 1's defensive metrics
- f2_f1 = fighter 2's offensive metrics
- f2_f2 = fighter 2's defensive metrics

Since I also calculated differences in metrics (_diff), those did not have a remaining f1 or f2 tag, so will show up with only an f1 or f2.

At this point, we have the absolute metrics for each fighter and their historical averages for both offense and defense.  As before, what is important is also the relative difference, so once again for each metric I calculated the difference in ratio.



What were my findings?
- My best model was able to achieve a 60% accuracy for predicting fights. 
- The best predictors for winning and losing from fighter 1's perspective:
    - Significant Head Strikes Landed - Historical Average Difference
    - Significant Strikes Landed - etc etc.
    
All of this is pretty much inline with general fight intuition.  Landing strikes to the head is the most common way to try to end a fight.  Performing submission is also a very common way to end a fight.  The number of submissions attempted is correlated with the number of submissions landed.


What risks/limitations/assumptions affect these findings?
- The biggest risk/limitation for my findings were that I was only able to get a limited number of fights that had enough data to model.
- The V1 and V2 jsons were mostly only available for more recent fights, with the most latest fight being in 2011.  For many fighters, the fight data that I had only covered the most recent 2/3rds to 1/2 of their total fights.
- I started with 4700 fights with fight details, however since we need to calculate the historical average, the first fight for a fighter in the dataset had to be dropped.
- Any fighter with only 1 fight basically didn't have any training data included.
- Since the Jsons were limited, I was not able to get time in position data, leaving out a data that is significant for predictions.


Summarize your statistical analysis, including:
implementation

- I implemented 2 types of models:  Logistic Regression and Random Forest.  I tried a number of different models including ADAboost, GradientBoost, but ultimately decided to drop them since they did not provide better predictions and were more "black box".


evaluation

- Both models ultimately performed relatively similarly topping out close to 60% accuracy.  

inference

- I chose model X since based on the analysis of its coefficients - its greatest predictors lined up with correlation with the outcome and also aligned with subject matter expert expectations - that reach is important, ability to land strikes to your opponents head is important (for strikers) and creating opportunities to attempt submissions is important (for grapplers).




### Description of data in each source

**V1 JSON API:**
The parameters for the V1 API is the EventID and follows the structure:  
    http://liveapi.fightmetric.com/V1/{EventID}/Fnt.json

- Each JSON contains data for each fight for the specified EventID
- Relevant fields that were scraped include:  
    - FightID for fight on the card
    - Max rounds possible and total rounds actually fought
    - Fight Outcome: win, loss, no contest, disqualification
    - Finish method:  knockout, submission, decision
    
 - For each Fighter:
    - FighterID and Name
    - Birthdate
    - Height, Weight
    - Fighting Stance
    - Location fighter fights out of
    - Location fighter born
    - Total Wins, Losses, Ties

**V2 JSON API:**
- The parameters for the V2 API is the EventID and FightID and follows the structure:  http://liveapi.fightmetric.com/V2/{EventID}/{FightID}/Stats.json
- Each JSON contains fight stats for striking, grappling, and time in position data for the specified fight
- The fight stats were provided at an aggregated level as well as for each round
- The strikes were broken down into more granularity from various positions or based on the target of the strike as well as separate figures for the number "attempted" and "landed"
    - Examples:  body_significant_strikes_attempts, clinch_body_strikes_landed, head_total_strikes_landed
- The time in position stats were the number of minutes and seconds in a specific position
- Since many of these were null, I dropped these columns

**UFC Fighter Page:**
- Each fighter has their own Fighter Page which shows their stats (height, weight, reach, leg reach) as well as their fight history
- An example of a UFC fighter page:  http://www.ufc.com/fighter/robert-whittaker
- I scraped this page to get the fighters height, weight, reach and leg_reach because I believe these physical attributes can be used to help predict the winner of fights


