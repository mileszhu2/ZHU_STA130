# STA130 Course Project

> See [here](https://github.com/pointOfive/stat130chat130/blob/main/CP/STA130F23_course_project_specification.ipynb) further details!!

The data we'll use for the Fall 2024 STA130 course project is based on the [Canadian Social Connection Survey](https://casch.org/cscs) (CSCS) run by [genwell](https://genwell.ca/) and the [Canadian Alliance for Social Connection and Health](https://casch.org/) (CASCH). This data is used to produce [Rapid Evidence Briefs](https://casch.org/publications) which scientifically analyze the importance of social connection and community engagement -- relationships with family, friends, coworkers, neighbours, and strangers -- for personal health and well-being. The purpose of our course project is to help raise awareness about and interest in this often under appreciated topic. To do so STA130 students will do "data blitz" to discover interesting information and identify key statistics hidden inside the CSCS data. The findings each project team produces will be shared with other UofT courses (in Digital Humanities, Writing, Sociology, etc.) and other genwell and CASCH associated teams specializing in communications and media relations, and they will transform the most interesting findings of our teams into content for social media campaigns. If our findings are important enough, they may even form the basis of future Rapid Evidence Briefs...

> In conjuction with this project, we are taking part in a reserach project "investigating the impact of community engaged learning on student happiness, satisfaction, and well-being" (alongside multiple courses in Pharmacology and Toxicology, Human Biology, and Psychology). We are curious to know if you find this project altruistic, and how this in turn might influece your own experience in the course, and beyond.

To use this data ethically, professionally, and appropriately we must take a quick look and abide by the [data use agreement](https://static1.squarespace.com/static/60283c2e174c122f8ebe0f39/t/6239c284d610f76fed5a2e69/1647952517436/Data+Use+Agreement+for+the+Canadian+Social+Connection+Survey.pdf). You should also take a quick look at the [list of available variables](https://drive.google.com/file/d/1ISVymGn-WR1lcRs4psIym2N3or5onNBi/view) and then check out the [data](https://drive.google.com/file/d/1mbUQlMTrNYA7Ly5eImVRBn16Ehy9Lggo/view) itself (available at the bottom of the [CSCS](https://casch.org/cscs) webpage).



```python
from IPython.display import YouTubeVideo
YouTubeVideo('rmuRoAf9-bo', width=800, height=500)
```





<iframe
    width="800"
    height="500"
    src="https://www.youtube.com/embed/rmuRoAf9-bo"
    frameborder="0"
    allowfullscreen
></iframe>





```python
# download these files from their original access point at the bottom of 
# the CSCS webpage (https://casch.org/cscs) or the course github and access them locally
# "https://raw.githubusercontent.com/pointOfive/stat130chat130/main/CP/var_names.csv"
# "https://raw.githubusercontent.com/pointOfive/stat130chat130/main/CP/CSCS_data_anon.csv"

import pandas as pd
cols = pd.read_csv("var_names.csv")
data = pd.read_csv("CSCS_data_anon.csv",
                   na_values=["9999", "", " ", "Presented but no response", "NA"])
empty = (data.isna().sum()==data.shape[0])
data = data[empty.index[~empty]] # keep non empty columns only

data.shape
```

    /tmp/ipykernel_86/3475480029.py:8: DtypeWarning: Columns (129,408,630,671,689,978,1001,1002,1006,1007,1008,1080,1113,1115,1116,1117,1118,1119,1120,1121,1124,1125,1126,1127,1128,1213,1214,1215,1216,1217,1218,1263,1266,1342,1343,1344,1345,1346,1347,1348,1349,1390,1391,1393,1439,1442,1463,1546,1549,1552,1555,1558,1561) have mixed types. Specify dtype option on import or set low_memory=False.
      data = pd.read_csv("CSCS_data_anon.csv",





    (11431, 1779)




```python
# This variable indicates the dataset that the observation belongs to
data.DATASET.value_counts()
```




    DATASET
    2022 Cross-Sectional    3916
    2021 Cross-Sectional    3589
    2023 Cross-Sectional    3058
    2022 Cohort              493
    2023 Cohort              375
    Name: count, dtype: int64




```python
# This variable indicates the year the observation was recruited
data.SURVEY_collection_year.value_counts()
```




    SURVEY_collection_year
    2022    4409
    2021    3589
    2023    3433
    Name: count, dtype: int64




```python
# This variable indicates whether the observation was part of the cohort survey or the cross-sectional survey 
data.SURVEY_collection_type.value_counts()
```




    SURVEY_collection_type
    cross     10563
    cohort      868
    Name: count, dtype: int64




```python
# This variable indicates whether the observation is from a cohort participant
data.SURVEY_cohort_participant.value_counts()
```




    SURVEY_cohort_participant
    False    10172
    True      1259
    Name: count, dtype: int64




```python
# This variable indicates the number of observations the participant has
data.SURVEY_num_responses.value_counts()
```




    SURVEY_num_responses
    1     9828
    3      691
    2      418
    4      407
    5       58
    6       19
    7        9
    11       1
    Name: count, dtype: int64




```python
# This variable indicates the unique ID of the participant
data.SURVEY_random_id.notna().value_counts() # few have IDs
```




    SURVEY_random_id
    False    10837
    True       594
    Name: count, dtype: int64




```python
# This variable indicates cases we recommend removing from an analysis due to fast completion times or possible fraudulent responses.
data.REMOVE_case.value_counts()
```




    REMOVE_case
    No       10018
    Yes       1153
    Maybe      260
    Name: count, dtype: int64




```python
# Let's just keep the recommended data
dataV2 = data[data.REMOVE_case=='No'].copy()
dataV2.shape
```




    (10018, 1779)




```python
# And select out participants who are part of the cohort data 
# (but may also be a part of the cross-sectional data)
dataV2_cohort = dataV2[dataV2.SURVEY_cohort_participant].copy()
dataV2_cohort.shape
```




    (865, 1779)




```python
# And remove year 2023 for which there's not yet much data collected
dataV2_cohortV2 = dataV2_cohort[dataV2_cohort.SURVEY_collection_year!=2023].copy()
dataV2_cohortV2.shape
```




    (850, 1779)




```python
# And remove columns that have less than some number of missing values
# We could consider no missing values using 850, or...?
missingness_limit = 100 # this retains 166 of 1024 columns that aren't fully empty
columns2keep = dataV2_cohortV2.isna().sum() < missingness_limit
columns2keep = columns2keep.index[columns2keep]
dataV2_cohortV3 = dataV2_cohortV2[columns2keep].copy()
dataV2_cohortV3.shape
```




    (850, 166)




```python
# The data itself then looks like this
pd.set_option('display.max_columns', dataV2_cohortV3.shape[1]) # Can cause jupyter notebooks to crash
# DO NOT USE in conjuection with pd.set_option('display.max_rows', 1000) 
dataV2_cohortV3
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>UNIQUE_id</th>
      <th>UNIQUE_num_records</th>
      <th>ELIGIBLE_consent</th>
      <th>COVID_prevention_distancing</th>
      <th>COVID_prevention_masks</th>
      <th>COVID_prevention_hand_washing</th>
      <th>COVID_prevention_reduce_people</th>
      <th>COVID_prevention_avoid_trips</th>
      <th>COVID_prevention_household</th>
      <th>COVID_vaccinated</th>
      <th>WELLNESS_life_satisfaction</th>
      <th>WELLNESS_malach_pines_burnout_measure_tired</th>
      <th>WELLNESS_malach_pines_burnout_measure_disappointed</th>
      <th>WELLNESS_malach_pines_burnout_measure_hopeless</th>
      <th>WELLNESS_malach_pines_burnout_measure_trapped</th>
      <th>WELLNESS_malach_pines_burnout_measure_helpless</th>
      <th>WELLNESS_malach_pines_burnout_measure_depressed</th>
      <th>WELLNESS_malach_pines_burnout_measure_sick</th>
      <th>WELLNESS_malach_pines_burnout_measure_worthless</th>
      <th>WELLNESS_malach_pines_burnout_measure_difficulty_sleeping</th>
      <th>WELLNESS_malach_pines_burnout_measure_had_it</th>
      <th>CONNECTION_activities_talked_day_p3m</th>
      <th>CONNECTION_activities_talked_family_p3m</th>
      <th>CONNECTION_activities_talked_job_p3m</th>
      <th>CONNECTION_activities_talked_hobbies_p3m</th>
      <th>CONNECTION_activities_phone_p3m</th>
      <th>CONNECTION_activities_letter_or_email_p3m</th>
      <th>CONNECTION_activities_checked_in_p3m</th>
      <th>CONNECTION_activities_text_or_messaged_p3m</th>
      <th>CONNECTION_activities_chat_p3m</th>
      <th>CONNECTION_activities_video_chat_p3m</th>
      <th>CONNECTION_activities_group_video_chat_p3m</th>
      <th>CONNECTION_activities_walk_p3m</th>
      <th>CONNECTION_activities_coffee_p3m</th>
      <th>CONNECTION_activities_board_games_p3m</th>
      <th>CONNECTION_activities_computer_games_p3m</th>
      <th>CONNECTION_activities_onlinegames_p3m</th>
      <th>CONNECTION_activities_visited_friends_p3m</th>
      <th>CONNECTION_activities_visited_family_p3m</th>
      <th>CONNECTION_activities_community_p3m</th>
      <th>CONNECTION_activities_helped_p3m</th>
      <th>CONNECTION_activities_meeting_work_p3m</th>
      <th>CONNECTION_activities_discussion_group_p3m</th>
      <th>CONNECTION_activities_group_exercise_p3m</th>
      <th>CONNECTION_activities_church_p3m</th>
      <th>CONNECTION_activities_new_friend_p3m</th>
      <th>CONNECTION_activities_hug_p3m</th>
      <th>CONNECTION_activities_kissed_p3m</th>
      <th>CONNECTION_activities_sex_p3m</th>
      <th>LONELY_ucla_loneliness_scale_companionship</th>
      <th>LONELY_ucla_loneliness_scale_left_out</th>
      <th>LONELY_ucla_loneliness_scale_isolated</th>
      <th>CONNECTION_social_num_close_friends_grouped</th>
      <th>CONNECTION_social_days_family_p7d_grouped</th>
      <th>CONNECTION_social_days_friends_p7d_grouped</th>
      <th>CONNECTION_social_days_coworkers_and_classmates_p7d_grouped</th>
      <th>CONNECTION_social_days_neighbours_p7d_grouped</th>
      <th>CONNECTION_social_time_family_p7d_grouped</th>
      <th>CONNECTION_social_time_friends_p7d_grouped</th>
      <th>CONNECTION_social_time_coworkers_and_classmates_p7d_grouped</th>
      <th>CONNECTION_social_time_neighbours_p7d_grouped</th>
      <th>CONNECTION_social_num_family_p7d_grouped</th>
      <th>CONNECTION_social_num_friends_p7d_grouped</th>
      <th>CONNECTION_social_num_coworkers_and_classmates_p7d_grouped</th>
      <th>CONNECTION_social_num_neighbours_p7d_grouped</th>
      <th>CONNECTION_preference_time_family_grouped</th>
      <th>CONNECTION_preference_time_friends_grouped</th>
      <th>CONNECTION_preference_time_coworkers_classmates_grouped</th>
      <th>CONNECTION_preference_time_neighbours_grouped</th>
      <th>LONELY_dejong_emotional_social_loneliness_scale_emptiness</th>
      <th>LONELY_dejong_emotional_social_loneliness_scale_rely</th>
      <th>LONELY_dejong_emotional_social_loneliness_scale_trust</th>
      <th>LONELY_dejong_emotional_social_loneliness_scale_close</th>
      <th>LONELY_dejong_emotional_social_loneliness_scale_miss</th>
      <th>LONELY_dejong_emotional_social_loneliness_scale_rejected</th>
      <th>LONELY_direct</th>
      <th>LONELY_change_pre_covid</th>
      <th>LONELY_others_aware</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_need</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_joys</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_positive_not_scored</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_gets_me_not_scored</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_family_helps</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_family_emotional</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_comfort</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_help</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_go_wrong</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_problems_family</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_problems_friends</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_sorrows</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_feelings</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_decisions</th>
      <th>WELLNESS_self_rated_physical_health</th>
      <th>WELLNESS_self_rated_mental_health</th>
      <th>WELLNESS_subjective_happiness_scale_happy</th>
      <th>WELLNESS_subjective_happiness_scale_peers</th>
      <th>WELLNESS_subjective_happiness_scale_always_happy</th>
      <th>WELLNESS_subjective_happiness_scale_never_happy</th>
      <th>WELLNESS_gad_anxious</th>
      <th>WELLNESS_gad_worry</th>
      <th>WELLNESS_phq_little_interest</th>
      <th>WELLNESS_phq_feeling_down</th>
      <th>SURVEY_collection_type</th>
      <th>SURVEY_collection_year</th>
      <th>DATASET</th>
      <th>SURVEY_recontact</th>
      <th>SURVEY_start_date</th>
      <th>SURVEY_end_date</th>
      <th>SURVEY_response_type</th>
      <th>SURVEY_duration_seconds</th>
      <th>SURVEY_progress</th>
      <th>SURVEY_finished</th>
      <th>SURVEY_recorded_date</th>
      <th>SURVEY_response_id</th>
      <th>SURVEY_distribution</th>
      <th>SURVEY_user_language</th>
      <th>SURVEY_cohort_participant</th>
      <th>SURVEY_num_responses</th>
      <th>SURVEY_orphan_cohort_response</th>
      <th>CONNECTION_activities_meeting_non_work_p3m</th>
      <th>CONNECTION_activities_meeting_non_work_pm</th>
      <th>CONNECTION_activities_church_pm</th>
      <th>CONNECTION_activities_greeted_neighbour_or_stranger_p3m</th>
      <th>CONNECTION_activities_greeted_neighbour_or_stranger_pm</th>
      <th>CONNECTION_activities_phone_pm</th>
      <th>CONNECTION_activities_group_video_chat_pm</th>
      <th>CONNECTION_activities_sex_pm</th>
      <th>CONNECTION_activities_helped_pm</th>
      <th>CONNECTION_activities_hug_pm</th>
      <th>CONNECTION_activities_kissed_pm</th>
      <th>CONNECTION_activities_new_friend_pm</th>
      <th>CONNECTION_activities_coffee_pm</th>
      <th>CONNECTION_activities_discussion_group_pm</th>
      <th>CONNECTION_activities_group_exercise_pm</th>
      <th>CONNECTION_activities_checked_in_pm</th>
      <th>CONNECTION_activities_visited_family_pm</th>
      <th>CONNECTION_activities_visited_friends_pm</th>
      <th>CONNECTION_activities_community_pm</th>
      <th>CONNECTION_activities_walk_pm</th>
      <th>CONNECTION_activities_games_p3m</th>
      <th>CONNECTION_activities_games_pm</th>
      <th>CONNECTION_activities_talked_p3m</th>
      <th>CONNECTION_activities_talked_pm</th>
      <th>SURVEY_num_surveys</th>
      <th>Num_answered</th>
      <th>Secs_per_q</th>
      <th>LONELY_ucla_loneliness_scale_score</th>
      <th>LONELY_ucla_loneliness_scale_score_y_n</th>
      <th>WELLNESS_malach_pines_burnout_measure_score</th>
      <th>WELLNESS_malach_pines_burnout_measure_score_y_n</th>
      <th>LONELY_dejong_emotional_social_loneliness_scale_score</th>
      <th>LONELY_dejong_emotional_loneliness_sub_scale_score</th>
      <th>LONELY_dejong_social_loneliness_sub_scale_score</th>
      <th>LONELY_dejong_emotional_social_loneliness_scale_score_y_n</th>
      <th>LONELY_dejong_emotional_loneliness_sub_scale_score_y_n</th>
      <th>LONELY_dejong_social_loneliness_sub_scale_score_y_n</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_score</th>
      <th>PSYCH_zimet_multidimensional_social_support_family_subscale_score</th>
      <th>PSYCH_zimet_multidimensional_social_support_significant_other_subscale_score</th>
      <th>PSYCH_zimet_multidimensional_social_support_friends_subscale_score</th>
      <th>WELLNESS_subjective_happiness_scale_score</th>
      <th>WELLNESS_phq_score</th>
      <th>WELLNESS_phq_score_y_n</th>
      <th>WELLNESS_gad_score</th>
      <th>WELLNESS_gad_score_y_n</th>
      <th>REMOVE_case</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>19</th>
      <td>cscs_00021</td>
      <td>3</td>
      <td>Yes</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Somewhat closely</td>
      <td>Yes, two doses</td>
      <td>4.0</td>
      <td>Sometimes</td>
      <td>Very Often</td>
      <td>Never</td>
      <td>Never</td>
      <td>Almost never</td>
      <td>Rarely</td>
      <td>Never</td>
      <td>Never</td>
      <td>Often</td>
      <td>Almost never</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Less than monthly</td>
      <td>A few times a week</td>
      <td>Less than monthly</td>
      <td>Daily or almost daily</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Less than monthly</td>
      <td>Less than monthly</td>
      <td>Daily or almost daily</td>
      <td>A few times a week</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>Less than monthly</td>
      <td>Monthly</td>
      <td>Monthly</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>A few times a month</td>
      <td>Some of the time</td>
      <td>Hardly Ever</td>
      <td>Often</td>
      <td>5 or more</td>
      <td>Most days (4 - 6 days)</td>
      <td>Most days (4 - 6 days)</td>
      <td>NaN</td>
      <td>Most days (4 - 6 days)</td>
      <td>1 to 4 hours</td>
      <td>5 or more hours</td>
      <td>NaN</td>
      <td>1 to 4 hours</td>
      <td>5 or more People</td>
      <td>5 or more People</td>
      <td>NaN</td>
      <td>1 - 2 People</td>
      <td>5 or more hours</td>
      <td>5 or more hours</td>
      <td>NaN</td>
      <td>1 to 4 hours</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Rarely (e.g. less than 1 day)</td>
      <td>Much more lonely</td>
      <td>Probably Yes</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Good</td>
      <td>Good</td>
      <td>5</td>
      <td>4</td>
      <td>4</td>
      <td>1 - Not at all</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>Several days</td>
      <td>cross</td>
      <td>2021</td>
      <td>2021 Cross-Sectional</td>
      <td>Yes</td>
      <td>2021-05-20 14:18</td>
      <td>2021-05-20 14:45</td>
      <td>IP Address</td>
      <td>1589</td>
      <td>100</td>
      <td>True</td>
      <td>2021-05-20 14:45</td>
      <td>R_2CploqdxsvHhmBg</td>
      <td>anonymous</td>
      <td>EN</td>
      <td>True</td>
      <td>3</td>
      <td>False</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>No</td>
      <td>Daily or almost daily</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>Weekly</td>
      <td>Yes</td>
      <td>3</td>
      <td>264</td>
      <td>6.018939</td>
      <td>6.0</td>
      <td>Yes (6-9)</td>
      <td>2.6</td>
      <td>No (1-3)</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>No (0-1)</td>
      <td>Yes (1-3)</td>
      <td>No (0)</td>
      <td>6.000000</td>
      <td>5.75</td>
      <td>7.00</td>
      <td>5.25</td>
      <td>5.00</td>
      <td>1.0</td>
      <td>No (0-2)</td>
      <td>0.0</td>
      <td>No (0-2)</td>
      <td>No</td>
    </tr>
    <tr>
      <th>21</th>
      <td>cscs_00021</td>
      <td>3</td>
      <td>Yes</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>Yes, three or more doses</td>
      <td>2.0</td>
      <td>Sometimes</td>
      <td>Sometimes</td>
      <td>Never</td>
      <td>Never</td>
      <td>Never</td>
      <td>Almost never</td>
      <td>Sometimes</td>
      <td>Never</td>
      <td>Often</td>
      <td>Never</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Less than monthly</td>
      <td>Daily or almost daily</td>
      <td>Less than monthly</td>
      <td>Less than monthly</td>
      <td>Weekly</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>A few times a week</td>
      <td>Less than monthly</td>
      <td>Less than monthly</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Monthly</td>
      <td>Monthly</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>A few times a month</td>
      <td>Some of the time</td>
      <td>Hardly Ever</td>
      <td>Some of the time</td>
      <td>5 or more</td>
      <td>None (0 Days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>None (0 Days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>1 to 4 hours</td>
      <td>5 or more hours</td>
      <td>No time</td>
      <td>Less than 1 hour</td>
      <td>1 - 2 People</td>
      <td>3 - 4 People</td>
      <td>None (0 People)</td>
      <td>3 - 4 People</td>
      <td>1 to 4 hours</td>
      <td>5 or more hours</td>
      <td>No time</td>
      <td>1 to 4 hours</td>
      <td>More or less</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>More or less</td>
      <td>No</td>
      <td>Some or a little of the time (e.g. 1-2 days)</td>
      <td>Somewhat more lonely</td>
      <td>Definitely Yes</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Very good</td>
      <td>Very good</td>
      <td>6</td>
      <td>4</td>
      <td>5</td>
      <td>2</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>Several days</td>
      <td>cohort</td>
      <td>2022</td>
      <td>2022 Cohort</td>
      <td>Yes</td>
      <td>2022-04-19 9:46</td>
      <td>2022-04-19 10:03</td>
      <td>IP Address</td>
      <td>1005</td>
      <td>100</td>
      <td>True</td>
      <td>2022-04-19 10:03</td>
      <td>R_3Dvy5jt24J1kCqZ</td>
      <td>gl</td>
      <td>EN</td>
      <td>True</td>
      <td>3</td>
      <td>False</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>No</td>
      <td>A few times a week</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Less than monthly</td>
      <td>No</td>
      <td>Weekly</td>
      <td>Yes</td>
      <td>3</td>
      <td>167</td>
      <td>6.017964</td>
      <td>5.0</td>
      <td>No (3-5)</td>
      <td>2.4</td>
      <td>No (1-3)</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>Yes (2-6)</td>
      <td>Yes (1-3)</td>
      <td>No (0)</td>
      <td>6.750000</td>
      <td>6.50</td>
      <td>7.00</td>
      <td>6.75</td>
      <td>5.25</td>
      <td>1.0</td>
      <td>No (0-2)</td>
      <td>0.0</td>
      <td>No (0-2)</td>
      <td>No</td>
    </tr>
    <tr>
      <th>71</th>
      <td>cscs_00074</td>
      <td>3</td>
      <td>Yes</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Not at all</td>
      <td>Somewhat closely</td>
      <td>Not at all</td>
      <td>Yes, three or more doses</td>
      <td>7.0</td>
      <td>Sometimes</td>
      <td>Rarely</td>
      <td>Never</td>
      <td>Never</td>
      <td>Never</td>
      <td>Almost never</td>
      <td>Sometimes</td>
      <td>Never</td>
      <td>Rarely</td>
      <td>Never</td>
      <td>A few times a week</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>NaN</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>A few times a week</td>
      <td>A few times a week</td>
      <td>Not in the past three months</td>
      <td>A few times a week</td>
      <td>Not in the past three months</td>
      <td>A few times a month</td>
      <td>Weekly</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>A few times a week</td>
      <td>A few times a week</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>A few times a week</td>
      <td>A few times a week</td>
      <td>Not in the past three months</td>
      <td>Hardly Ever</td>
      <td>Hardly Ever</td>
      <td>Some of the time</td>
      <td>5 or more</td>
      <td>Some days (1 - 3 days)</td>
      <td>Most days (4 - 6 days)</td>
      <td>None (0 Days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>1 to 4 hours</td>
      <td>5 or more hours</td>
      <td>No time</td>
      <td>Less than 1 hour</td>
      <td>1 - 2 People</td>
      <td>3 - 4 People</td>
      <td>None (0 People)</td>
      <td>None (0 People)</td>
      <td>1 to 4 hours</td>
      <td>5 or more hours</td>
      <td>Less than 1 hour</td>
      <td>Less than 1 hour</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>None of the time (e.g., 0 days)</td>
      <td>About the same</td>
      <td>Probably Yes</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Agree</td>
      <td>Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Agree</td>
      <td>Very Strongly Agree</td>
      <td>Good</td>
      <td>Excellent</td>
      <td>6</td>
      <td>5</td>
      <td>6</td>
      <td>1 - Not at all</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>cohort</td>
      <td>2022</td>
      <td>2022 Cohort</td>
      <td>Yes</td>
      <td>2022-04-19 9:24</td>
      <td>2022-04-19 9:39</td>
      <td>IP Address</td>
      <td>912</td>
      <td>100</td>
      <td>True</td>
      <td>2022-04-19 9:39</td>
      <td>R_1lrFgFjX7P2O6nx</td>
      <td>gl</td>
      <td>EN</td>
      <td>True</td>
      <td>5</td>
      <td>False</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>No</td>
      <td>Daily or almost daily</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>Daily or almost daily</td>
      <td>Yes</td>
      <td>3</td>
      <td>166</td>
      <td>5.493976</td>
      <td>4.0</td>
      <td>No (3-5)</td>
      <td>2.1</td>
      <td>No (1-3)</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>No (0-1)</td>
      <td>No (0)</td>
      <td>No (0)</td>
      <td>6.333333</td>
      <td>6.00</td>
      <td>6.50</td>
      <td>6.50</td>
      <td>6.00</td>
      <td>0.0</td>
      <td>No (0-2)</td>
      <td>0.0</td>
      <td>No (0-2)</td>
      <td>No</td>
    </tr>
    <tr>
      <th>77</th>
      <td>cscs_00080</td>
      <td>2</td>
      <td>Yes</td>
      <td>Somewhat closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Yes, three or more doses</td>
      <td>8.0</td>
      <td>Often</td>
      <td>Sometimes</td>
      <td>Almost never</td>
      <td>Almost never</td>
      <td>Almost never</td>
      <td>Rarely</td>
      <td>Sometimes</td>
      <td>Rarely</td>
      <td>Often</td>
      <td>Almost never</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Monthly</td>
      <td>A few times a week</td>
      <td>A few times a week</td>
      <td>Less than monthly</td>
      <td>A few times a month</td>
      <td>A few times a month</td>
      <td>A few times a month</td>
      <td>Less than monthly</td>
      <td>Not in the past three months</td>
      <td>Less than monthly</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Weekly</td>
      <td>A few times a month</td>
      <td>Monthly</td>
      <td>Not in the past three months</td>
      <td>Monthly</td>
      <td>Not in the past three months</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>A few times a month</td>
      <td>Daily or almost daily</td>
      <td>Monthly</td>
      <td>Some of the time</td>
      <td>Some of the time</td>
      <td>Some of the time</td>
      <td>5 or more</td>
      <td>Most days (4 - 6 days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>None (0 Days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>5 or more hours</td>
      <td>1 to 4 hours</td>
      <td>No time</td>
      <td>Less than 1 hour</td>
      <td>5 or more People</td>
      <td>3 - 4 People</td>
      <td>None (0 People)</td>
      <td>1 - 2 People</td>
      <td>5 or more hours</td>
      <td>1 to 4 hours</td>
      <td>No time</td>
      <td>Less than 1 hour</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>More or less</td>
      <td>More or less</td>
      <td>No</td>
      <td>Some or a little of the time (e.g. 1-2 days)</td>
      <td>About the same</td>
      <td>Probably No</td>
      <td>Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Disagree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Strongly Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Strongly Agree</td>
      <td>Agree</td>
      <td>Fair</td>
      <td>Good</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>2</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>cohort</td>
      <td>2022</td>
      <td>2022 Cohort</td>
      <td>Yes</td>
      <td>2022-04-29 2:08</td>
      <td>2022-04-29 2:29</td>
      <td>IP Address</td>
      <td>1252</td>
      <td>100</td>
      <td>True</td>
      <td>2022-04-29 2:29</td>
      <td>R_3E2tq0v88nWHa3T</td>
      <td>gl</td>
      <td>EN</td>
      <td>True</td>
      <td>2</td>
      <td>False</td>
      <td>Monthly</td>
      <td>Yes</td>
      <td>No</td>
      <td>A few times a week</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>Weekly</td>
      <td>Yes</td>
      <td>A few times a week</td>
      <td>Yes</td>
      <td>2</td>
      <td>168</td>
      <td>7.452381</td>
      <td>6.0</td>
      <td>Yes (6-9)</td>
      <td>3.2</td>
      <td>No (1-3)</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>Yes (2-6)</td>
      <td>Yes (1-3)</td>
      <td>Yes (1-3)</td>
      <td>4.916667</td>
      <td>4.00</td>
      <td>5.75</td>
      <td>5.00</td>
      <td>6.00</td>
      <td>0.0</td>
      <td>No (0-2)</td>
      <td>0.0</td>
      <td>No (0-2)</td>
      <td>No</td>
    </tr>
    <tr>
      <th>78</th>
      <td>cscs_00080</td>
      <td>2</td>
      <td>Yes</td>
      <td>Somewhat closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Yes, two doses</td>
      <td>9.0</td>
      <td>Often</td>
      <td>Rarely</td>
      <td>Never</td>
      <td>Almost never</td>
      <td>Never</td>
      <td>Almost never</td>
      <td>Rarely</td>
      <td>Almost never</td>
      <td>Sometimes</td>
      <td>Never</td>
      <td>A few times a week</td>
      <td>A few times a week</td>
      <td>Monthly</td>
      <td>A few times a week</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Less than monthly</td>
      <td>Not in the past three months</td>
      <td>Monthly</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Weekly</td>
      <td>A few times a month</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Less than monthly</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>Monthly</td>
      <td>Some of the time</td>
      <td>Some of the time</td>
      <td>Some of the time</td>
      <td>3–4</td>
      <td>Some days (1 - 3 days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>None (0 Days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>5 or more hours</td>
      <td>5 or more hours</td>
      <td>No time</td>
      <td>1 to 4 hours</td>
      <td>5 or more People</td>
      <td>5 or more People</td>
      <td>None (0 People)</td>
      <td>1 - 2 People</td>
      <td>1 to 4 hours</td>
      <td>5 or more hours</td>
      <td>No time</td>
      <td>1 to 4 hours</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>More or less</td>
      <td>No</td>
      <td>None of the time (e.g., 0 days)</td>
      <td>About the same</td>
      <td>Probably No</td>
      <td>Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Strongly Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Strongly Agree</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Agree</td>
      <td>Good</td>
      <td>Very good</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>2</td>
      <td>Several days</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>cross</td>
      <td>2021</td>
      <td>2021 Cross-Sectional</td>
      <td>Yes</td>
      <td>2021-07-13 23:08</td>
      <td>2021-07-14 0:37</td>
      <td>IP Address</td>
      <td>5344</td>
      <td>100</td>
      <td>True</td>
      <td>2021-07-14 0:37</td>
      <td>R_06uL36mkFDNwxLr</td>
      <td>anonymous</td>
      <td>EN</td>
      <td>True</td>
      <td>2</td>
      <td>False</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>No</td>
      <td>Daily or almost daily</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Weekly</td>
      <td>Yes</td>
      <td>A few times a week</td>
      <td>Yes</td>
      <td>2</td>
      <td>359</td>
      <td>14.885794</td>
      <td>6.0</td>
      <td>Yes (6-9)</td>
      <td>2.4</td>
      <td>No (1-3)</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>No (0-1)</td>
      <td>Yes (1-3)</td>
      <td>No (0)</td>
      <td>5.333333</td>
      <td>4.50</td>
      <td>6.25</td>
      <td>5.25</td>
      <td>6.00</td>
      <td>0.0</td>
      <td>No (0-2)</td>
      <td>1.0</td>
      <td>No (0-2)</td>
      <td>No</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>11379</th>
      <td>cscs_11760</td>
      <td>3</td>
      <td>Yes</td>
      <td>Somewhat closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Yes, one dose</td>
      <td>7.0</td>
      <td>Sometimes</td>
      <td>Sometimes</td>
      <td>Sometimes</td>
      <td>Often</td>
      <td>Often</td>
      <td>Very Often</td>
      <td>Almost never</td>
      <td>Sometimes</td>
      <td>Sometimes</td>
      <td>Sometimes</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Daily or almost daily</td>
      <td>A few times a month</td>
      <td>Monthly</td>
      <td>A few times a week</td>
      <td>Less than monthly</td>
      <td>Daily or almost daily</td>
      <td>A few times a week</td>
      <td>Monthly</td>
      <td>A few times a week</td>
      <td>Weekly</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>Weekly</td>
      <td>Monthly</td>
      <td>A few times a month</td>
      <td>Monthly</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Often</td>
      <td>Often</td>
      <td>Often</td>
      <td>3–4</td>
      <td>None (0 Days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>None (0 Days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>No time</td>
      <td>1 to 4 hours</td>
      <td>No time</td>
      <td>Less than 1 hour</td>
      <td>None (0 People)</td>
      <td>3 - 4 People</td>
      <td>None (0 People)</td>
      <td>3 - 4 People</td>
      <td>No time</td>
      <td>1 to 4 hours</td>
      <td>1 to 4 hours</td>
      <td>1 to 4 hours</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Occasionally or a moderate amount of time (e.g...</td>
      <td>Much more lonely</td>
      <td>Probably No</td>
      <td>Strongly Agree</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Very Strongly Disagree</td>
      <td>Very Strongly Disagree</td>
      <td>Strongly Agree</td>
      <td>Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Very Strongly Disagree</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Very good</td>
      <td>Fair</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
      <td>5</td>
      <td>More than half the days</td>
      <td>More than half the days</td>
      <td>Several days</td>
      <td>More than half the days</td>
      <td>cross</td>
      <td>2021</td>
      <td>2021 Cross-Sectional</td>
      <td>Yes</td>
      <td>2021-05-16 7:51</td>
      <td>2021-05-16 8:19</td>
      <td>IP Address</td>
      <td>1657</td>
      <td>100</td>
      <td>True</td>
      <td>2021-05-16 8:19</td>
      <td>R_2EavLKv6HLuoi00</td>
      <td>anonymous</td>
      <td>EN</td>
      <td>True</td>
      <td>3</td>
      <td>False</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>No</td>
      <td>Daily or almost daily</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>Daily or almost daily</td>
      <td>Yes</td>
      <td>3</td>
      <td>286</td>
      <td>5.793706</td>
      <td>9.0</td>
      <td>Yes (6-9)</td>
      <td>4.2</td>
      <td>Yes (4-7)</td>
      <td>6.0</td>
      <td>3.0</td>
      <td>3.0</td>
      <td>Yes (2-6)</td>
      <td>Yes (1-3)</td>
      <td>Yes (1-3)</td>
      <td>3.916667</td>
      <td>1.75</td>
      <td>5.25</td>
      <td>4.75</td>
      <td>3.00</td>
      <td>3.0</td>
      <td>Yes (3-6)</td>
      <td>4.0</td>
      <td>Yes (3-6)</td>
      <td>No</td>
    </tr>
    <tr>
      <th>11380</th>
      <td>cscs_11760</td>
      <td>3</td>
      <td>Yes</td>
      <td>Somewhat closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Very closely</td>
      <td>Yes, three or more doses</td>
      <td>7.0</td>
      <td>Sometimes</td>
      <td>Sometimes</td>
      <td>Often</td>
      <td>Often</td>
      <td>Often</td>
      <td>Always</td>
      <td>Rarely</td>
      <td>Very Often</td>
      <td>Rarely</td>
      <td>Sometimes</td>
      <td>A few times a week</td>
      <td>Weekly</td>
      <td>A few times a month</td>
      <td>A few times a week</td>
      <td>Less than monthly</td>
      <td>A few times a month</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>A few times a week</td>
      <td>A few times a month</td>
      <td>A few times a month</td>
      <td>A few times a week</td>
      <td>A few times a week</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Monthly</td>
      <td>Not in the past three months</td>
      <td>A few times a week</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>Monthly</td>
      <td>A few times a week</td>
      <td>Not in the past three months</td>
      <td>Less than monthly</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Often</td>
      <td>Some of the time</td>
      <td>Some of the time</td>
      <td>5 or more</td>
      <td>Every day (7 days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>1 to 4 hours</td>
      <td>Less than 1 hour</td>
      <td>Less than 1 hour</td>
      <td>Less than 1 hour</td>
      <td>1 - 2 People</td>
      <td>3 - 4 People</td>
      <td>5 or more People</td>
      <td>3 - 4 People</td>
      <td>1 to 4 hours</td>
      <td>1 to 4 hours</td>
      <td>1 to 4 hours</td>
      <td>1 to 4 hours</td>
      <td>More or less</td>
      <td>More or less</td>
      <td>More or less</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>More or less</td>
      <td>Some or a little of the time (e.g. 1-2 days)</td>
      <td>Much more lonely</td>
      <td>Probably No</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Disagree</td>
      <td>Very Strongly Disagree</td>
      <td>Agree</td>
      <td>Strongly Agree</td>
      <td>Agree</td>
      <td>Very Strongly Disagree</td>
      <td>Agree</td>
      <td>Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Excellent</td>
      <td>Fair</td>
      <td>4</td>
      <td>3</td>
      <td>4</td>
      <td>5</td>
      <td>Several days</td>
      <td>Several days</td>
      <td>Several days</td>
      <td>More than half the days</td>
      <td>cohort</td>
      <td>2022</td>
      <td>2022 Cohort</td>
      <td>Yes</td>
      <td>2022-04-18 22:47</td>
      <td>2022-04-18 22:59</td>
      <td>IP Address</td>
      <td>731</td>
      <td>100</td>
      <td>True</td>
      <td>2022-04-18 22:59</td>
      <td>R_1LiNhqSkx2n0WFL</td>
      <td>gl</td>
      <td>EN</td>
      <td>True</td>
      <td>3</td>
      <td>False</td>
      <td>Monthly</td>
      <td>Yes</td>
      <td>No</td>
      <td>Daily or almost daily</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>A few times a month</td>
      <td>Yes</td>
      <td>A few times a week</td>
      <td>Yes</td>
      <td>3</td>
      <td>169</td>
      <td>4.325444</td>
      <td>7.0</td>
      <td>Yes (6-9)</td>
      <td>4.6</td>
      <td>Yes (4-7)</td>
      <td>5.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>Yes (2-6)</td>
      <td>Yes (1-3)</td>
      <td>Yes (1-3)</td>
      <td>4.583333</td>
      <td>1.75</td>
      <td>6.50</td>
      <td>5.50</td>
      <td>3.50</td>
      <td>3.0</td>
      <td>Yes (3-6)</td>
      <td>2.0</td>
      <td>No (0-2)</td>
      <td>No</td>
    </tr>
    <tr>
      <th>11412</th>
      <td>cscs_11795</td>
      <td>3</td>
      <td>Yes</td>
      <td>Somewhat closely</td>
      <td>Very closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Not at all</td>
      <td>Yes, three or more doses</td>
      <td>7.0</td>
      <td>Sometimes</td>
      <td>Often</td>
      <td>Rarely</td>
      <td>Rarely</td>
      <td>Almost never</td>
      <td>Sometimes</td>
      <td>Sometimes</td>
      <td>Never</td>
      <td>Rarely</td>
      <td>Never</td>
      <td>A few times a week</td>
      <td>Daily or almost daily</td>
      <td>Monthly</td>
      <td>A few times a week</td>
      <td>Daily or almost daily</td>
      <td>Weekly</td>
      <td>A few times a week</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>Weekly</td>
      <td>Monthly</td>
      <td>A few times a week</td>
      <td>A few times a week</td>
      <td>Less than monthly</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>A few times a week</td>
      <td>Weekly</td>
      <td>Less than monthly</td>
      <td>Monthly</td>
      <td>Not in the past three months</td>
      <td>Weekly</td>
      <td>Monthly</td>
      <td>Not in the past three months</td>
      <td>Monthly</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>Weekly</td>
      <td>Often</td>
      <td>Some of the time</td>
      <td>Some of the time</td>
      <td>5 or more</td>
      <td>Most days (4 - 6 days)</td>
      <td>Every day (7 days)</td>
      <td>None (0 Days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>5 or more hours</td>
      <td>5 or more hours</td>
      <td>No time</td>
      <td>Less than 1 hour</td>
      <td>5 or more People</td>
      <td>5 or more People</td>
      <td>None (0 People)</td>
      <td>3 - 4 People</td>
      <td>5 or more hours</td>
      <td>5 or more hours</td>
      <td>No time</td>
      <td>1 to 4 hours</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>More or less</td>
      <td>Yes</td>
      <td>No</td>
      <td>All of the time (e.g. 5-7 days)]</td>
      <td>Somewhat more lonely</td>
      <td>Probably Yes</td>
      <td>Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Agree</td>
      <td>Strongly Disagree</td>
      <td>Agree</td>
      <td>Very Strongly Agree</td>
      <td>Agree</td>
      <td>Good</td>
      <td>Good</td>
      <td>6</td>
      <td>4</td>
      <td>6</td>
      <td>5</td>
      <td>Several days</td>
      <td>Several days</td>
      <td>Several days</td>
      <td>Not at all</td>
      <td>cohort</td>
      <td>2022</td>
      <td>2022 Cohort</td>
      <td>Yes</td>
      <td>2022-04-19 9:32</td>
      <td>2022-04-19 9:46</td>
      <td>IP Address</td>
      <td>838</td>
      <td>100</td>
      <td>True</td>
      <td>2022-04-19 9:46</td>
      <td>R_V56wRphCkuBRz8t</td>
      <td>gl</td>
      <td>EN</td>
      <td>True</td>
      <td>4</td>
      <td>False</td>
      <td>A few times a month</td>
      <td>Yes</td>
      <td>No</td>
      <td>A few times a week</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Less than monthly</td>
      <td>No</td>
      <td>Daily or almost daily</td>
      <td>Yes</td>
      <td>3</td>
      <td>168</td>
      <td>4.988095</td>
      <td>7.0</td>
      <td>Yes (6-9)</td>
      <td>3.0</td>
      <td>No (1-3)</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>Yes (2-6)</td>
      <td>Yes (1-3)</td>
      <td>Yes (1-3)</td>
      <td>5.416667</td>
      <td>5.75</td>
      <td>6.50</td>
      <td>4.00</td>
      <td>4.75</td>
      <td>1.0</td>
      <td>No (0-2)</td>
      <td>2.0</td>
      <td>No (0-2)</td>
      <td>No</td>
    </tr>
    <tr>
      <th>11428</th>
      <td>cscs_11812</td>
      <td>3</td>
      <td>Yes</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Very closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Somewhat closely</td>
      <td>Yes, two doses</td>
      <td>4.0</td>
      <td>Often</td>
      <td>Sometimes</td>
      <td>Sometimes</td>
      <td>Sometimes</td>
      <td>Often</td>
      <td>Often</td>
      <td>Often</td>
      <td>Sometimes</td>
      <td>Always</td>
      <td>Sometimes</td>
      <td>Weekly</td>
      <td>Weekly</td>
      <td>A few times a week</td>
      <td>Weekly</td>
      <td>Daily or almost daily</td>
      <td>Not in the past three months</td>
      <td>A few times a month</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>A few times a week</td>
      <td>Not in the past three months</td>
      <td>Less than monthly</td>
      <td>Less than monthly</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Monthly</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Less than monthly</td>
      <td>Not in the past three months</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Often</td>
      <td>Some of the time</td>
      <td>Some of the time</td>
      <td>1–2</td>
      <td>Some days (1 - 3 days)</td>
      <td>Every day (7 days)</td>
      <td>Most days (4 - 6 days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>1 to 4 hours</td>
      <td>5 or more hours</td>
      <td>1 to 4 hours</td>
      <td>Less than 1 hour</td>
      <td>3 - 4 People</td>
      <td>1 - 2 People</td>
      <td>1 - 2 People</td>
      <td>None (0 People)</td>
      <td>1 to 4 hours</td>
      <td>1 to 4 hours</td>
      <td>1 to 4 hours</td>
      <td>Less than 1 hour</td>
      <td>More or less</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Occasionally or a moderate amount of time (e.g...</td>
      <td>Somewhat more lonely</td>
      <td>Probably No</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Very Strongly Agree</td>
      <td>Disagree</td>
      <td>Disagree</td>
      <td>Very Strongly Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Very Strongly Agree</td>
      <td>Disagree</td>
      <td>Fair</td>
      <td>Poor</td>
      <td>5</td>
      <td>4</td>
      <td>2</td>
      <td>4</td>
      <td>Several days</td>
      <td>Several days</td>
      <td>More than half the days</td>
      <td>Several days</td>
      <td>cohort</td>
      <td>2022</td>
      <td>2022 Cohort</td>
      <td>Yes</td>
      <td>2022-04-19 11:54</td>
      <td>2022-04-19 12:08</td>
      <td>IP Address</td>
      <td>847</td>
      <td>100</td>
      <td>True</td>
      <td>2022-04-19 12:08</td>
      <td>R_OHY6lNe0aqV7gzL</td>
      <td>gl</td>
      <td>EN</td>
      <td>True</td>
      <td>3</td>
      <td>False</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>No</td>
      <td>A few times a week</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>A few times a week</td>
      <td>Yes</td>
      <td>3</td>
      <td>167</td>
      <td>5.071856</td>
      <td>7.0</td>
      <td>Yes (6-9)</td>
      <td>4.7</td>
      <td>Yes (4-7)</td>
      <td>4.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>Yes (2-6)</td>
      <td>Yes (1-3)</td>
      <td>Yes (1-3)</td>
      <td>4.833333</td>
      <td>3.25</td>
      <td>7.00</td>
      <td>4.25</td>
      <td>3.75</td>
      <td>3.0</td>
      <td>Yes (3-6)</td>
      <td>2.0</td>
      <td>No (0-2)</td>
      <td>No</td>
    </tr>
    <tr>
      <th>11430</th>
      <td>cscs_11812</td>
      <td>3</td>
      <td>Yes</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>Very closely</td>
      <td>No</td>
      <td>7.0</td>
      <td>Sometimes</td>
      <td>Rarely</td>
      <td>Almost never</td>
      <td>Never</td>
      <td>Never</td>
      <td>Rarely</td>
      <td>Rarely</td>
      <td>Never</td>
      <td>Sometimes</td>
      <td>Never</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>Less than monthly</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>Daily or almost daily</td>
      <td>Not in the past three months</td>
      <td>A few times a month</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Monthly</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>A few times a week</td>
      <td>Not in the past three months</td>
      <td>Not in the past three months</td>
      <td>Some of the time</td>
      <td>Hardly Ever</td>
      <td>Some of the time</td>
      <td>3–4</td>
      <td>Every day (7 days)</td>
      <td>Some days (1 - 3 days)</td>
      <td>None (0 Days)</td>
      <td>None (0 Days)</td>
      <td>5 or more hours</td>
      <td>No time</td>
      <td>No time</td>
      <td>No time</td>
      <td>1 - 2 People</td>
      <td>None (0 People)</td>
      <td>None (0 People)</td>
      <td>None (0 People)</td>
      <td>5 or more hours</td>
      <td>5 or more hours</td>
      <td>5 or more hours</td>
      <td>Less than 1 hour</td>
      <td>No</td>
      <td>More or less</td>
      <td>More or less</td>
      <td>More or less</td>
      <td>Yes</td>
      <td>No</td>
      <td>Some or a little of the time (e.g. 1-2 days)</td>
      <td>Somewhat more lonely</td>
      <td>Probably Yes</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Disagree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Agree</td>
      <td>Disagree</td>
      <td>Disagree</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Agree</td>
      <td>Neither Agree Nor Disagree</td>
      <td>Poor</td>
      <td>Fair</td>
      <td>5</td>
      <td>4</td>
      <td>5</td>
      <td>4</td>
      <td>Several days</td>
      <td>Not at all</td>
      <td>Not at all</td>
      <td>Several days</td>
      <td>cross</td>
      <td>2021</td>
      <td>2021 Cross-Sectional</td>
      <td>Yes</td>
      <td>2021-05-02 1:59</td>
      <td>2021-05-02 2:16</td>
      <td>IP Address</td>
      <td>1004</td>
      <td>100</td>
      <td>True</td>
      <td>2021-05-02 2:16</td>
      <td>R_1lntDjPA7BG7gyy</td>
      <td>anonymous</td>
      <td>EN</td>
      <td>True</td>
      <td>3</td>
      <td>False</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>No</td>
      <td>A few times a week</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>Yes</td>
      <td>Not in the past three months</td>
      <td>No</td>
      <td>Daily or almost daily</td>
      <td>Yes</td>
      <td>3</td>
      <td>272</td>
      <td>3.691176</td>
      <td>5.0</td>
      <td>No (3-5)</td>
      <td>2.3</td>
      <td>No (1-3)</td>
      <td>4.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>Yes (2-6)</td>
      <td>Yes (1-3)</td>
      <td>Yes (1-3)</td>
      <td>4.333333</td>
      <td>4.00</td>
      <td>5.00</td>
      <td>4.00</td>
      <td>4.50</td>
      <td>1.0</td>
      <td>No (0-2)</td>
      <td>1.0</td>
      <td>No (0-2)</td>
      <td>No</td>
    </tr>
  </tbody>
</table>
<p>850 rows × 166 columns</p>
</div>




```python
# Here's a high level summary of the data
for col in dataV2_cohortV3.columns:
    col_summary = dataV2_cohortV3[col].value_counts(dropna=False)
    if len(col_summary)<11:
        print(col_summary, "\n")
    else:
        print(col, dataV2_cohortV3[col].shape, "\n")
```

    UNIQUE_id (850,) 
    
    UNIQUE_num_records
    3    628
    2    199
    4     23
    Name: count, dtype: int64 
    
    ELIGIBLE_consent
    Yes    850
    Name: count, dtype: int64 
    
    COVID_prevention_distancing
    Very closely        431
    Somewhat closely    303
    Not at all           75
    NaN                  41
    Name: count, dtype: int64 
    
    COVID_prevention_masks
    Very closely        576
    Somewhat closely    142
    Not at all           90
    NaN                  42
    Name: count, dtype: int64 
    
    COVID_prevention_hand_washing
    Very closely        581
    Somewhat closely    189
    NaN                  41
    Not at all           39
    Name: count, dtype: int64 
    
    COVID_prevention_reduce_people
    Very closely        504
    Somewhat closely    205
    Not at all           96
    NaN                  45
    Name: count, dtype: int64 
    
    COVID_prevention_avoid_trips
    Very closely        410
    Somewhat closely    247
    Not at all          151
    NaN                  42
    Name: count, dtype: int64 
    
    COVID_prevention_household
    Very closely        387
    Somewhat closely    260
    Not at all          158
    NaN                  45
    Name: count, dtype: int64 
    
    COVID_vaccinated
    Yes, three or more doses    358
    Yes, one dose               207
    No                          129
    Yes, two doses              109
    NaN                          47
    Name: count, dtype: int64 
    
    WELLNESS_life_satisfaction (850,) 
    
    WELLNESS_malach_pines_burnout_measure_tired
    Sometimes       256
    Often           229
    Very Often      147
    Always           79
    Rarely           56
    NaN              39
    Almost never     36
    Never             8
    Name: count, dtype: int64 
    
    WELLNESS_malach_pines_burnout_measure_disappointed
    Sometimes       327
    Often           186
    Very Often      113
    Rarely           97
    Almost never     51
    NaN              38
    Always           25
    Never            13
    Name: count, dtype: int64 
    
    WELLNESS_malach_pines_burnout_measure_hopeless
    Sometimes       205
    Rarely          157
    Almost never    150
    Never           142
    Often            88
    Very Often       47
    NaN              38
    Always           23
    Name: count, dtype: int64 
    
    WELLNESS_malach_pines_burnout_measure_trapped
    Sometimes       211
    Never           184
    Almost never    115
    Rarely          115
    Often            92
    Very Often       60
    NaN              38
    Always           35
    Name: count, dtype: int64 
    
    WELLNESS_malach_pines_burnout_measure_helpless
    Sometimes       187
    Never           169
    Almost never    155
    Rarely          145
    Often            86
    Very Often       51
    NaN              38
    Always           19
    Name: count, dtype: int64 
    
    WELLNESS_malach_pines_burnout_measure_depressed
    Sometimes       247
    Rarely          135
    Almost never    122
    Often           113
    Never            78
    Very Often       72
    Always           44
    NaN              39
    Name: count, dtype: int64 
    
    WELLNESS_malach_pines_burnout_measure_sick
    Sometimes       220
    Rarely          162
    Almost never    161
    Never           101
    Often            88
    Very Often       47
    NaN              40
    Always           31
    Name: count, dtype: int64 
    
    WELLNESS_malach_pines_burnout_measure_worthless
    Never           222
    Almost never    157
    Sometimes       154
    Rarely          136
    Often            64
    Very Often       49
    NaN              38
    Always           30
    Name: count, dtype: int64 
    
    WELLNESS_malach_pines_burnout_measure_difficulty_sleeping
    Sometimes       216
    Often           133
    Almost never    117
    Rarely          112
    Very Often       93
    Always           71
    Never            70
    NaN              38
    Name: count, dtype: int64 
    
    WELLNESS_malach_pines_burnout_measure_had_it
    Sometimes       228
    Never           143
    Almost never    142
    Rarely          119
    Often            82
    Very Often       66
    NaN              41
    Always           29
    Name: count, dtype: int64 
    
    CONNECTION_activities_talked_day_p3m
    Daily or almost daily           254
    A few times a week              243
    Weekly                          135
    A few times a month              84
    Less than monthly                53
    Monthly                          38
    Not in the past three months     22
    NaN                              21
    Name: count, dtype: int64 
    
    CONNECTION_activities_talked_family_p3m
    A few times a week              201
    Weekly                          188
    Daily or almost daily           140
    A few times a month             129
    Less than monthly                76
    Monthly                          67
    Not in the past three months     29
    NaN                              20
    Name: count, dtype: int64 
    
    CONNECTION_activities_talked_job_p3m
    A few times a week              162
    Weekly                          148
    Daily or almost daily           131
    A few times a month             130
    Less than monthly                94
    Not in the past three months     90
    Monthly                          68
    NaN                              27
    Name: count, dtype: int64 
    
    CONNECTION_activities_talked_hobbies_p3m
    Weekly                          169
    A few times a week              166
    A few times a month             136
    Less than monthly               111
    Daily or almost daily           106
    Monthly                          71
    Not in the past three months     69
    NaN                              22
    Name: count, dtype: int64 
    
    CONNECTION_activities_phone_p3m
    Daily or almost daily           195
    Weekly                          164
    A few times a week              161
    A few times a month             136
    Less than monthly                70
    Monthly                          64
    Not in the past three months     40
    NaN                              20
    Name: count, dtype: int64 
    
    CONNECTION_activities_letter_or_email_p3m
    Not in the past three months    216
    Less than monthly               172
    A few times a month             140
    A few times a week               90
    Weekly                           81
    Monthly                          70
    Daily or almost daily            63
    NaN                              18
    Name: count, dtype: int64 
    
    CONNECTION_activities_checked_in_p3m
    Daily or almost daily           156
    A few times a week              155
    A few times a month             148
    Weekly                          144
    Less than monthly                81
    Monthly                          75
    Not in the past three months     70
    NaN                              21
    Name: count, dtype: int64 
    
    CONNECTION_activities_text_or_messaged_p3m
    A few times a month             153
    A few times a week              131
    Less than monthly               126
    Weekly                          125
    Not in the past three months    108
    Daily or almost daily           103
    Monthly                          88
    NaN                              16
    Name: count, dtype: int64 
    
    CONNECTION_activities_chat_p3m
    A few times a week              154
    Daily or almost daily           150
    A few times a month             143
    Weekly                          119
    Not in the past three months    103
    Less than monthly                85
    Monthly                          73
    NaN                              23
    Name: count, dtype: int64 
    
    CONNECTION_activities_video_chat_p3m
    Not in the past three months    259
    Less than monthly               130
    A few times a month             130
    Weekly                          111
    Monthly                          77
    A few times a week               75
    Daily or almost daily            44
    NaN                              24
    Name: count, dtype: int64 
    
    CONNECTION_activities_group_video_chat_p3m
    Not in the past three months    413
    Less than monthly               162
    Monthly                          85
    A few times a month              66
    Weekly                           65
    A few times a week               28
    NaN                              21
    Daily or almost daily            10
    Name: count, dtype: int64 
    
    CONNECTION_activities_walk_p3m
    Not in the past three months    190
    A few times a month             144
    Weekly                          124
    A few times a week              120
    Less than monthly               109
    Monthly                          78
    Daily or almost daily            66
    NaN                              19
    Name: count, dtype: int64 
    
    CONNECTION_activities_coffee_p3m
    Not in the past three months    281
    Less than monthly               163
    A few times a month             137
    Monthly                          95
    Weekly                           91
    A few times a week               47
    Daily or almost daily            19
    NaN                              17
    Name: count, dtype: int64 
    
    CONNECTION_activities_board_games_p3m
    Not in the past three months    535
    Less than monthly               138
    Monthly                          63
    A few times a month              43
    Weekly                           27
    NaN                              17
    Daily or almost daily            14
    A few times a week               13
    Name: count, dtype: int64 
    
    CONNECTION_activities_computer_games_p3m
    Not in the past three months    671
    Less than monthly                53
    Monthly                          30
    A few times a month              28
    NaN                              19
    Weekly                           17
    A few times a week               16
    Daily or almost daily            16
    Name: count, dtype: int64 
    
    CONNECTION_activities_onlinegames_p3m
    Not in the past three months    616
    Daily or almost daily            58
    Less than monthly                44
    A few times a month              36
    Weekly                           30
    A few times a week               27
    NaN                              20
    Monthly                          19
    Name: count, dtype: int64 
    
    CONNECTION_activities_visited_friends_p3m
    Not in the past three months    325
    Less than monthly               153
    A few times a month             121
    Monthly                          98
    Weekly                           97
    A few times a week               26
    NaN                              22
    Daily or almost daily             8
    Name: count, dtype: int64 
    
    CONNECTION_activities_visited_family_p3m
    Not in the past three months    296
    Less than monthly               186
    A few times a month             123
    Weekly                           86
    Monthly                          82
    A few times a week               33
    Daily or almost daily            23
    NaN                              21
    Name: count, dtype: int64 
    
    CONNECTION_activities_community_p3m
    Not in the past three months    560
    Weekly                           62
    Less than monthly                59
    A few times a month              50
    Monthly                          46
    A few times a week               37
    Daily or almost daily            18
    NaN                              18
    Name: count, dtype: int64 
    
    CONNECTION_activities_helped_p3m
    Not in the past three months    491
    Less than monthly               162
    Monthly                          74
    A few times a month              59
    Weekly                           29
    NaN                              18
    A few times a week               12
    Daily or almost daily             5
    Name: count, dtype: int64 
    
    CONNECTION_activities_meeting_work_p3m
    Not in the past three months    524
    Weekly                           63
    Less than monthly                61
    A few times a month              50
    Daily or almost daily            49
    Monthly                          41
    A few times a week               39
    NaN                              23
    Name: count, dtype: int64 
    
    CONNECTION_activities_discussion_group_p3m
    Not in the past three months    391
    Less than monthly               106
    Monthly                          87
    A few times a month              85
    Weekly                           74
    A few times a week               54
    Daily or almost daily            30
    NaN                              23
    Name: count, dtype: int64 
    
    CONNECTION_activities_group_exercise_p3m
    Not in the past three months    667
    Less than monthly                38
    Weekly                           36
    A few times a week               31
    A few times a month              25
    Monthly                          19
    NaN                              18
    Daily or almost daily            16
    Name: count, dtype: int64 
    
    CONNECTION_activities_church_p3m
    Not in the past three months    710
    Weekly                           51
    Less than monthly                27
    A few times a month              23
    NaN                              20
    Monthly                          12
    A few times a week                4
    Daily or almost daily             3
    Name: count, dtype: int64 
    
    CONNECTION_activities_new_friend_p3m
    Not in the past three months    554
    Less than monthly               171
    Monthly                          59
    A few times a month              26
    NaN                              18
    Weekly                           11
    A few times a week                7
    Daily or almost daily             4
    Name: count, dtype: int64 
    
    CONNECTION_activities_hug_p3m
    Daily or almost daily           239
    Not in the past three months    183
    A few times a month             108
    A few times a week               89
    Less than monthly                88
    Weekly                           72
    Monthly                          49
    NaN                              22
    Name: count, dtype: int64 
    
    CONNECTION_activities_kissed_p3m
    Not in the past three months    360
    Daily or almost daily           241
    A few times a week               81
    Less than monthly                56
    A few times a month              41
    Weekly                           36
    NaN                              24
    Monthly                          11
    Name: count, dtype: int64 
    
    CONNECTION_activities_sex_p3m
    Not in the past three months    491
    Weekly                           75
    Less than monthly                71
    A few times a week               69
    A few times a month              68
    Monthly                          31
    NaN                              31
    Daily or almost daily            14
    Name: count, dtype: int64 
    
    LONELY_ucla_loneliness_scale_companionship
    Some of the time    336
    Hardly Ever         245
    Often               235
    NaN                  34
    Name: count, dtype: int64 
    
    LONELY_ucla_loneliness_scale_left_out
    Some of the time    329
    Hardly Ever         298
    Often               188
    NaN                  35
    Name: count, dtype: int64 
    
    LONELY_ucla_loneliness_scale_isolated
    Some of the time    328
    Often               264
    Hardly Ever         223
    NaN                  35
    Name: count, dtype: int64 
    
    CONNECTION_social_num_close_friends_grouped
    3–4          285
    5 or more    280
    1–2          198
    NaN           87
    Name: count, dtype: int64 
    
    CONNECTION_social_days_family_p7d_grouped
    Some days (1 - 3 days)    307
    Every day (7 days)        264
    Most days (4 - 6 days)    121
    None (0 Days)             121
    NaN                        37
    Name: count, dtype: int64 
    
    CONNECTION_social_days_friends_p7d_grouped
    Some days (1 - 3 days)    402
    None (0 Days)             174
    Most days (4 - 6 days)    138
    Every day (7 days)         99
    NaN                        37
    Name: count, dtype: int64 
    
    CONNECTION_social_days_coworkers_and_classmates_p7d_grouped
    None (0 Days)             458
    Some days (1 - 3 days)    178
    Most days (4 - 6 days)    136
    NaN                        53
    Every day (7 days)         25
    Name: count, dtype: int64 
    
    CONNECTION_social_days_neighbours_p7d_grouped
    None (0 Days)             380
    Some days (1 - 3 days)    347
    Most days (4 - 6 days)     53
    NaN                        46
    Every day (7 days)         24
    Name: count, dtype: int64 
    
    CONNECTION_social_time_family_p7d_grouped
    5 or more hours     328
    1 to 4 hours        234
    Less than 1 hour    126
    No time             120
    NaN                  42
    Name: count, dtype: int64 
    
    CONNECTION_social_time_friends_p7d_grouped
    1 to 4 hours        245
    5 or more hours     234
    Less than 1 hour    168
    No time             161
    NaN                  42
    Name: count, dtype: int64 
    
    CONNECTION_social_time_coworkers_and_classmates_p7d_grouped
    No time             452
    Less than 1 hour    130
    1 to 4 hours        107
    5 or more hours     100
    NaN                  61
    Name: count, dtype: int64 
    
    CONNECTION_social_time_neighbours_p7d_grouped
    No time             360
    Less than 1 hour    322
    1 to 4 hours         89
    NaN                  50
    5 or more hours      29
    Name: count, dtype: int64 
    
    CONNECTION_social_num_family_p7d_grouped
    1 - 2 People        338
    3 - 4 People        198
    5 or more People    147
    None (0 People)     125
    NaN                  42
    Name: count, dtype: int64 
    
    CONNECTION_social_num_friends_p7d_grouped
    1 - 2 People        325
    3 - 4 People        174
    None (0 People)     166
    5 or more People    144
    NaN                  41
    Name: count, dtype: int64 
    
    CONNECTION_social_num_coworkers_and_classmates_p7d_grouped
    None (0 People)     463
    5 or more People    122
    1 - 2 People        114
    3 - 4 People         92
    NaN                  59
    Name: count, dtype: int64 
    
    CONNECTION_social_num_neighbours_p7d_grouped
    None (0 People)     380
    1 - 2 People        288
    3 - 4 People         92
    NaN                  51
    5 or more People     39
    Name: count, dtype: int64 
    
    CONNECTION_preference_time_family_grouped
    5 or more hours     357
    1 to 4 hours        306
    Less than 1 hour     96
    NaN                  49
    No time              42
    Name: count, dtype: int64 
    
    CONNECTION_preference_time_friends_grouped
    5 or more hours     364
    1 to 4 hours        332
    Less than 1 hour     78
    NaN                  45
    No time              31
    Name: count, dtype: int64 
    
    CONNECTION_preference_time_coworkers_classmates_grouped
    No time             324
    1 to 4 hours        186
    Less than 1 hour    181
    5 or more hours      92
    NaN                  67
    Name: count, dtype: int64 
    
    CONNECTION_preference_time_neighbours_grouped
    Less than 1 hour    368
    No time             197
    1 to 4 hours        184
    NaN                  52
    5 or more hours      49
    Name: count, dtype: int64 
    
    LONELY_dejong_emotional_social_loneliness_scale_emptiness
    No              383
    More or less    245
    Yes             186
    NaN              36
    Name: count, dtype: int64 
    
    LONELY_dejong_emotional_social_loneliness_scale_rely
    More or less    303
    Yes             298
    No              213
    NaN              36
    Name: count, dtype: int64 
    
    LONELY_dejong_emotional_social_loneliness_scale_trust
    Yes             283
    More or less    269
    No              262
    NaN              36
    Name: count, dtype: int64 
    
    LONELY_dejong_emotional_social_loneliness_scale_close
    Yes             348
    More or less    270
    No              195
    NaN              37
    Name: count, dtype: int64 
    
    LONELY_dejong_emotional_social_loneliness_scale_miss
    Yes             413
    More or less    224
    No              178
    NaN              35
    Name: count, dtype: int64 
    
    LONELY_dejong_emotional_social_loneliness_scale_rejected
    No              491
    More or less    188
    Yes             135
    NaN              36
    Name: count, dtype: int64 
    
    LONELY_direct
    Rarely (e.g. less than 1 day)                                199
    Some or a little of the time (e.g. 1-2 days)                 185
    None of the time (e.g., 0 days)                              185
    Occasionally or a moderate amount of time (e.g. 3-4 days)    164
    All of the time (e.g. 5-7 days)]                              83
    NaN                                                           34
    Name: count, dtype: int64 
    
    LONELY_change_pre_covid
    About the same          268
    Somewhat more lonely    263
    Much more lonely        207
    Somewhat less lonely     47
    NaN                      37
    Much less lonely         28
    Name: count, dtype: int64 
    
    LONELY_others_aware
    Probably No       398
    Probably Yes      196
    Definitely No     174
    Definitely Yes     43
    NaN                39
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_need
    Very Strongly Agree           279
    Agree                         166
    Strongly Agree                128
    Disagree                       89
    Neither Agree Nor Disagree     70
    Very Strongly Disagree         50
    NaN                            40
    Strongly Disagree              28
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_joys
    Very Strongly Agree           278
    Agree                         173
    Strongly Agree                132
    Neither Agree Nor Disagree     78
    Disagree                       77
    Very Strongly Disagree         44
    NaN                            40
    Strongly Disagree              28
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_positive_not_scored
    Very Strongly Agree           236
    Agree                         195
    Strongly Agree                137
    Neither Agree Nor Disagree     95
    Disagree                       87
    NaN                            43
    Very Strongly Disagree         32
    Strongly Disagree              25
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_gets_me_not_scored
    Very Strongly Agree           230
    Agree                         164
    Strongly Agree                137
    Neither Agree Nor Disagree    108
    Disagree                       92
    Very Strongly Disagree         44
    NaN                            40
    Strongly Disagree              35
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_family_helps
    Agree                         182
    Very Strongly Agree           171
    Neither Agree Nor Disagree    155
    Strongly Agree                120
    Disagree                       94
    Very Strongly Disagree         48
    NaN                            42
    Strongly Disagree              38
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_family_emotional
    Agree                         192
    Neither Agree Nor Disagree    155
    Disagree                      130
    Very Strongly Agree           118
    Strongly Agree                103
    Very Strongly Disagree         65
    Strongly Disagree              44
    NaN                            43
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_comfort
    Very Strongly Agree           258
    Agree                         176
    Strongly Agree                121
    Disagree                       93
    Neither Agree Nor Disagree     86
    Very Strongly Disagree         43
    NaN                            42
    Strongly Disagree              31
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_help
    Agree                         245
    Neither Agree Nor Disagree    210
    Strongly Agree                121
    Very Strongly Agree           100
    Disagree                       81
    NaN                            42
    Very Strongly Disagree         33
    Strongly Disagree              18
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_go_wrong
    Agree                         239
    Neither Agree Nor Disagree    183
    Strongly Agree                133
    Very Strongly Agree           116
    Disagree                       82
    NaN                            44
    Very Strongly Disagree         33
    Strongly Disagree              20
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_problems_family
    Agree                         204
    Neither Agree Nor Disagree    132
    Disagree                      123
    Strongly Agree                114
    Very Strongly Agree           102
    Very Strongly Disagree         68
    Strongly Disagree              64
    NaN                            43
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_problems_friends
    Agree                         291
    Strongly Agree                163
    Neither Agree Nor Disagree    144
    Very Strongly Agree            99
    Disagree                       66
    NaN                            43
    Very Strongly Disagree         26
    Strongly Disagree              18
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_sorrows
    Agree                         278
    Very Strongly Agree           180
    Strongly Agree                172
    Neither Agree Nor Disagree     98
    Disagree                       44
    NaN                            42
    Very Strongly Disagree         19
    Strongly Disagree              17
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_feelings
    Very Strongly Agree           284
    Agree                         154
    Strongly Agree                124
    Neither Agree Nor Disagree     92
    Disagree                       72
    Very Strongly Disagree         55
    NaN                            42
    Strongly Disagree              27
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_decisions
    Agree                         242
    Neither Agree Nor Disagree    168
    Strongly Agree                155
    Very Strongly Agree           121
    Disagree                       68
    NaN                            42
    Very Strongly Disagree         37
    Strongly Disagree              17
    Name: count, dtype: int64 
    
    WELLNESS_self_rated_physical_health
    Good         279
    Fair         200
    Very good    198
    Poor          75
    Excellent     58
    NaN           40
    Name: count, dtype: int64 
    
    WELLNESS_self_rated_mental_health
    Good         239
    Fair         225
    Very good    156
    Poor         118
    Excellent     72
    NaN           40
    Name: count, dtype: int64 
    
    WELLNESS_subjective_happiness_scale_happy
    5                          265
    6                          166
    4                          158
    7 - A very happy person     77
    3                           73
    NaN                         41
    1 - Not a happy person      39
    2                           31
    Name: count, dtype: int64 
    
    WELLNESS_subjective_happiness_scale_peers
    5                 199
    4                 194
    3                 109
    6                 108
    7 - More happy     89
    1 - Less happy     71
    2                  40
    NaN                40
    Name: count, dtype: int64 
    
    WELLNESS_subjective_happiness_scale_always_happy
    5                   178
    4                   171
    6                   125
    3                   105
    1 - Not at all       95
    7 - A great deal     80
    2                    57
    NaN                  39
    Name: count, dtype: int64 
    
    WELLNESS_subjective_happiness_scale_never_happy
    2                   170
    1 - Not at all      163
    4                   131
    3                   129
    5                   123
    6                    56
    NaN                  41
    7 - A great deal     37
    Name: count, dtype: int64 
    
    WELLNESS_gad_anxious
    Several days               316
    Not at all                 260
    More than half the days    122
    Nearly every day           112
    NaN                         40
    Name: count, dtype: int64 
    
    WELLNESS_gad_worry
    Not at all                 340
    Several days               244
    Nearly every day           116
    More than half the days    107
    NaN                         43
    Name: count, dtype: int64 
    
    WELLNESS_phq_little_interest
    Not at all                 320
    Several days               267
    More than half the days    128
    Nearly every day            93
    NaN                         42
    Name: count, dtype: int64 
    
    WELLNESS_phq_feeling_down
    Not at all                 331
    Several days               276
    More than half the days    112
    Nearly every day            91
    NaN                         40
    Name: count, dtype: int64 
    
    SURVEY_collection_type
    cohort    477
    cross     373
    Name: count, dtype: int64 
    
    SURVEY_collection_year
    2022    477
    2021    373
    Name: count, dtype: int64 
    
    DATASET
    2022 Cohort             477
    2021 Cross-Sectional    373
    Name: count, dtype: int64 
    
    SURVEY_recontact
    Yes    846
    NaN      3
    No       1
    Name: count, dtype: int64 
    
    SURVEY_start_date (850,) 
    
    SURVEY_end_date (850,) 
    
    SURVEY_response_type
    IP Address    850
    Name: count, dtype: int64 
    
    SURVEY_duration_seconds (850,) 
    
    SURVEY_progress
    100    811
    10      18
    5       14
    62       3
    29       1
    69       1
    28       1
    7        1
    Name: count, dtype: int64 
    
    SURVEY_finished
    True     811
    False     39
    Name: count, dtype: int64 
    
    SURVEY_recorded_date (850,) 
    
    SURVEY_response_id (850,) 
    
    SURVEY_distribution
    gl           476
    anonymous    373
    glà            1
    Name: count, dtype: int64 
    
    SURVEY_user_language
    EN       820
    FR-CA     30
    Name: count, dtype: int64 
    
    SURVEY_cohort_participant
    True    850
    Name: count, dtype: int64 
    
    SURVEY_num_responses
    3    452
    4    237
    2    118
    5     29
    6     11
    7      3
    Name: count, dtype: int64 
    
    SURVEY_orphan_cohort_response
    False    841
    True       9
    Name: count, dtype: int64 
    
    CONNECTION_activities_meeting_non_work_p3m
    Not in the past three months    533
    Monthly                          85
    Less than monthly                79
    A few times a month              57
    Weekly                           52
    NaN                              20
    A few times a week               17
    Daily or almost daily             7
    Name: count, dtype: int64 
    
    CONNECTION_activities_meeting_non_work_pm
    No     612
    Yes    218
    NaN     20
    Name: count, dtype: int64 
    
    CONNECTION_activities_church_pm
    No     737
    Yes     93
    NaN     20
    Name: count, dtype: int64 
    
    CONNECTION_activities_greeted_neighbour_or_stranger_p3m
    Daily or almost daily           272
    A few times a week              220
    A few times a month             120
    Weekly                          117
    Less than monthly                48
    Monthly                          38
    Not in the past three months     19
    NaN                              16
    Name: count, dtype: int64 
    
    CONNECTION_activities_greeted_neighbour_or_stranger_pm
    Yes    767
    No      67
    NaN     16
    Name: count, dtype: int64 
    
    CONNECTION_activities_phone_pm
    Yes    720
    No     110
    NaN     20
    Name: count, dtype: int64 
    
    CONNECTION_activities_group_video_chat_pm
    No     575
    Yes    254
    NaN     21
    Name: count, dtype: int64 
    
    CONNECTION_activities_sex_pm
    No     562
    Yes    257
    NaN     31
    Name: count, dtype: int64 
    
    CONNECTION_activities_helped_pm
    No     653
    Yes    179
    NaN     18
    Name: count, dtype: int64 
    
    CONNECTION_activities_hug_pm
    Yes    557
    No     271
    NaN     22
    Name: count, dtype: int64 
    
    CONNECTION_activities_kissed_pm
    No     416
    Yes    410
    NaN     24
    Name: count, dtype: int64 
    
    CONNECTION_activities_new_friend_pm
    No     725
    Yes    107
    NaN     18
    Name: count, dtype: int64 
    
    CONNECTION_activities_coffee_pm
    No     444
    Yes    389
    NaN     17
    Name: count, dtype: int64 
    
    CONNECTION_activities_discussion_group_pm
    No     497
    Yes    330
    NaN     23
    Name: count, dtype: int64 
    
    CONNECTION_activities_group_exercise_pm
    No     705
    Yes    127
    NaN     18
    Name: count, dtype: int64 
    
    CONNECTION_activities_checked_in_pm
    Yes    678
    No     151
    NaN     21
    Name: count, dtype: int64 
    
    CONNECTION_activities_visited_family_pm
    No     482
    Yes    347
    NaN     21
    Name: count, dtype: int64 
    
    CONNECTION_activities_visited_friends_pm
    No     478
    Yes    350
    NaN     22
    Name: count, dtype: int64 
    
    CONNECTION_activities_community_pm
    No     619
    Yes    213
    NaN     18
    Name: count, dtype: int64 
    
    CONNECTION_activities_walk_pm
    Yes    532
    No     299
    NaN     19
    Name: count, dtype: int64 
    
    CONNECTION_activities_games_p3m
    Not in the past three months    401
    Less than monthly               143
    A few times a month              73
    Monthly                          67
    NaN                              65
    Weekly                           56
    A few times a week               45
    Name: count, dtype: int64 
    
    CONNECTION_activities_games_pm
    No     544
    Yes    241
    NaN     65
    Name: count, dtype: int64 
    
    CONNECTION_activities_talked_p3m
    Daily or almost daily           317
    A few times a week              243
    Weekly                          121
    A few times a month              72
    Less than monthly                36
    Monthly                          34
    NaN                              16
    Not in the past three months     11
    Name: count, dtype: int64 
    
    CONNECTION_activities_talked_pm
    Yes    787
    No      47
    NaN     16
    Name: count, dtype: int64 
    
    SURVEY_num_surveys
    3    628
    2    199
    4     23
    Name: count, dtype: int64 
    
    Num_answered (850,) 
    
    Secs_per_q (850,) 
    
    LONELY_ucla_loneliness_scale_score
    6.0    166
    3.0    144
    9.0    142
    5.0    117
    7.0     96
    4.0     88
    8.0     62
    NaN     35
    Name: count, dtype: int64 
    
    LONELY_ucla_loneliness_scale_score_y_n
    Yes (6-9)    466
    No (3-5)     349
    NaN           35
    Name: count, dtype: int64 
    
    WELLNESS_malach_pines_burnout_measure_score (850,) 
    
    WELLNESS_malach_pines_burnout_measure_score_y_n
    No (1-3)     498
    Yes (4-7)    299
    NaN           53
    Name: count, dtype: int64 
    
    LONELY_dejong_emotional_social_loneliness_scale_score
    6.0    158
    5.0    146
    4.0    135
    3.0    115
    1.0    109
    2.0    107
    NaN     41
    0.0     39
    Name: count, dtype: int64 
    
    LONELY_dejong_emotional_loneliness_sub_scale_score
    1.0    247
    3.0    233
    2.0    222
    0.0    111
    NaN     37
    Name: count, dtype: int64 
    
    LONELY_dejong_social_loneliness_sub_scale_score
    3.0    368
    0.0    186
    2.0    147
    1.0    110
    NaN     39
    Name: count, dtype: int64 
    
    LONELY_dejong_emotional_social_loneliness_scale_score_y_n
    Yes (2-6)    661
    No (0-1)     148
    NaN           41
    Name: count, dtype: int64 
    
    LONELY_dejong_emotional_loneliness_sub_scale_score_y_n
    Yes (1-3)    702
    No (0)       111
    NaN           37
    Name: count, dtype: int64 
    
    LONELY_dejong_social_loneliness_sub_scale_score_y_n
    Yes (1-3)    625
    No (0)       186
    NaN           39
    Name: count, dtype: int64 
    
    PSYCH_zimet_multidimensional_social_support_scale_score (850,) 
    
    PSYCH_zimet_multidimensional_social_support_family_subscale_score (850,) 
    
    PSYCH_zimet_multidimensional_social_support_significant_other_subscale_score (850,) 
    
    PSYCH_zimet_multidimensional_social_support_friends_subscale_score (850,) 
    
    WELLNESS_subjective_happiness_scale_score (850,) 
    
    WELLNESS_phq_score
    0.0    247
    2.0    167
    1.0    140
    3.0     82
    4.0     68
    6.0     65
    NaN     43
    5.0     38
    Name: count, dtype: int64 
    
    WELLNESS_phq_score_y_n
    No (0-2)     554
    Yes (3-6)    253
    NaN           43
    Name: count, dtype: int64 
    
    WELLNESS_gad_score
    0.0    224
    2.0    175
    1.0    119
    3.0     98
    4.0     81
    6.0     67
    NaN     44
    5.0     42
    Name: count, dtype: int64 
    
    WELLNESS_gad_score_y_n
    No (0-2)     518
    Yes (3-6)    288
    NaN           44
    Name: count, dtype: int64 
    
    REMOVE_case
    No    850
    Name: count, dtype: int64 
    



```python
# And here are some explanations about the columns in the data
cols_dataV2_cohortV3 = cols.new_var.apply(lambda x: x in dataV2_cohortV3.columns)
cols_dataV2_cohortV3 = cols[cols_dataV2_cohortV3].iloc[:,-2:].drop_duplicates()
for i,row in cols_dataV2_cohortV3.sort_values('new_var').iterrows():
    print("\033[1m\033[4m",row.new_var, "\033[0m", sep="", end=": ")
    print(row.text, end="\n\n")
```

    [1m[4mCONNECTION_activities_board_games_p3m[0m: In the PAST THREE MONTH, how often have you... - played a board game with others?
    
    [1m[4mCONNECTION_activities_chat_p3m[0m: In the PAST THREE MONTH, how often have you... - had an extended conversation via text or a messaging app?
    
    [1m[4mCONNECTION_activities_chat_p3m[0m: In the PAST THREE MONTH, how often have you... - had an in-person, face-to-face conversation with someone?
    
    [1m[4mCONNECTION_activities_checked_in_p3m[0m: In the PAST THREE MONTH, how often have you... - sent a text/private message to someone just to check in?
    
    [1m[4mCONNECTION_activities_church_p3m[0m: In the PAST THREE MONTH, how often have you... - attended church, synagogue, temple, etc.?
    
    [1m[4mCONNECTION_activities_coffee_p3m[0m: In the PAST THREE MONTH, how often have you... - met someone for a meal, drink, dessert, or cup of coffee?
    
    [1m[4mCONNECTION_activities_community_p3m[0m: In the PAST THREE MONTH, how often have you... - volunteered in the community?
    
    [1m[4mCONNECTION_activities_computer_games_p3m[0m: In the PAST THREE MONTH, how often have you... - played a computer or console (e.g., Wii, Xbox, PlayStation) game with others?
    
    [1m[4mCONNECTION_activities_computer_games_p3m[0m: In the PAST THREE MONTH, how often have you... - played a video or board game with others?
    
    [1m[4mCONNECTION_activities_discussion_group_p3m[0m: In the PAST THREE MONTH, how often have you... - participated in an online discussion group?
    
    [1m[4mCONNECTION_activities_group_exercise_p3m[0m: In the PAST THREE MONTH, how often have you... - participated in group exercise (e.g., yoga classes, cycling)?
    
    [1m[4mCONNECTION_activities_group_video_chat_p3m[0m: In the PAST THREE MONTH, how often have you... - had a video chat with a GROUP of friends or family?
    
    [1m[4mCONNECTION_activities_helped_p3m[0m: In the PAST THREE MONTH, how often have you... - helped a neighbor or friend with a task or chore (e.g., yard work, moving)?
    
    [1m[4mCONNECTION_activities_hug_p3m[0m: In the PAST THREE MONTH, how often have you... - hugged someone?
    
    [1m[4mCONNECTION_activities_kissed_p3m[0m: In the PAST THREE MONTH, how often have you... - kissed someone?
    
    [1m[4mCONNECTION_activities_letter_or_email_p3m[0m: In the PAST THREE MONTH, how often have you... - wrote a letter or personal email to a friend or family member?
    
    [1m[4mCONNECTION_activities_letter_or_email_p3m[0m: In the PAST THREE MONTH, how often have you... - attended a meeting at work?
    
    [1m[4mCONNECTION_activities_meeting_work_p3m[0m: In the PAST THREE MONTH, how often have you... - attended a meeting at work?
    
    [1m[4mCONNECTION_activities_new_friend_p3m[0m: In the PAST THREE MONTH, how often have you... - made a new friend?
    
    [1m[4mCONNECTION_activities_onlinegames_p3m[0m: In the PAST THREE MONTH, how often have you... - played an online game with others?
    
    [1m[4mCONNECTION_activities_phone_p3m[0m: In the PAST THREE MONTH, how often have you... - had a phone conversation with a friend or family member?
    
    [1m[4mCONNECTION_activities_sex_p3m[0m: In the PAST THREE MONTH, how often have you... - had sex with someone?
    
    [1m[4mCONNECTION_activities_talked_day_p3m[0m: In the PAST THREE MONTH, how often have you... - talked to someone about how your / their day was going?
    
    [1m[4mCONNECTION_activities_talked_family_p3m[0m: In the PAST THREE MONTH, how often have you... - talked to someone about how your / their family was?
    
    [1m[4mCONNECTION_activities_talked_hobbies_p3m[0m: In the PAST THREE MONTH, how often have you... - talked to someone about your / their hobbies or interests?
    
    [1m[4mCONNECTION_activities_talked_job_p3m[0m: In the PAST THREE MONTH, how often have you... - talked to someone about your / their job?
    
    [1m[4mCONNECTION_activities_text_or_messaged_p3m[0m: In the PAST THREE MONTH, how often have you... - received a text/private message from someone who was checking in with you?
    
    [1m[4mCONNECTION_activities_text_or_messaged_p3m[0m: In the PAST THREE MONTH, how often have you... - sent a text or private message to someone just to check in?
    
    [1m[4mCONNECTION_activities_video_chat_p3m[0m: In the PAST THREE MONTH, how often have you... - had a video chat with a friend or family member?
    
    [1m[4mCONNECTION_activities_visited_family_p3m[0m: In the PAST THREE MONTH, how often have you... - visited with FAMILY at your / their home?
    
    [1m[4mCONNECTION_activities_visited_family_p3m[0m: In the PAST THREE MONTH, how often have you... - visited with FAMILY at your or their home?
    
    [1m[4mCONNECTION_activities_visited_friends_p3m[0m: In the PAST THREE MONTH, how often have you... - visited with FRIENDS at your or their home?
    
    [1m[4mCONNECTION_activities_visited_friends_p3m[0m: In the PAST THREE MONTH, how often have you... - visited with FRIENDS at your / their home?
    
    [1m[4mCONNECTION_activities_walk_p3m[0m: In the PAST THREE MONTH, how often have you... - went for a walk with someone?
    
    [1m[4mCONNECTION_preference_time_coworkers_classmates_grouped[0m: How much time per week would you like to spend socializing with others from the following groups? - Coworkers or Classmates
    
    [1m[4mCONNECTION_preference_time_family_grouped[0m: How much time per week would you like to spend socializing with others from the following groups? - Family Members
    
    [1m[4mCONNECTION_preference_time_friends_grouped[0m: How much time per week would you like to spend socializing with others from the following groups? - Friends
    
    [1m[4mCONNECTION_preference_time_neighbours_grouped[0m: How much time per week would you like to spend socializing with others from the following groups? - Neighbours
    
    [1m[4mCONNECTION_social_days_coworkers_and_classmates_p7d_grouped[0m: In the PAST WEEK, how many days did you spend at least 5 minutes socializing with people from the following groups? - Coworkers or Classmates
    
    [1m[4mCONNECTION_social_days_family_p7d_grouped[0m: In the PAST WEEK, how many days did you spend at least 5 minutes socializing with people from the following groups? - Family Members
    
    [1m[4mCONNECTION_social_days_friends_p7d_grouped[0m: In the PAST WEEK, how many days did you spend at least 5 minutes socializing with people from the following groups? - Friends
    
    [1m[4mCONNECTION_social_days_neighbours_p7d_grouped[0m: In the PAST WEEK, how many days did you spend at least 5 minutes socializing with people from the following groups? - Neighbours
    
    [1m[4mCONNECTION_social_num_close_friends_grouped[0m: How many close friends do you have?
    
    [1m[4mCONNECTION_social_num_coworkers_and_classmates_p7d_grouped[0m: In the PAST WEEK, how many people from each of the following groups did you spend at least 5 minutes socializing with? - Coworkers or Classmates
    
    [1m[4mCONNECTION_social_num_family_p7d_grouped[0m: In the PAST WEEK, how many people from each of the following groups did you spend at least 5 minutes socializing with? - Family Members
    
    [1m[4mCONNECTION_social_num_friends_p7d_grouped[0m: In the PAST WEEK, how many people from each of the following groups did you spend at least 5 minutes socializing with? - Friends
    
    [1m[4mCONNECTION_social_num_neighbours_p7d_grouped[0m: In the PAST WEEK, how many people from each of the following groups did you spend at least 5 minutes socializing with? - Neighbours
    
    [1m[4mCONNECTION_social_time_coworkers_and_classmates_p7d_grouped[0m: In the PAST WEEK, how many hours in total did you spend socializing with others from the following groups? - Coworkers or Classmates
    
    [1m[4mCONNECTION_social_time_family_p7d_grouped[0m: In the PAST WEEK, how many hours in total did you spend socializing with others from the following groups? - Family Members
    
    [1m[4mCONNECTION_social_time_friends_p7d_grouped[0m: In the PAST WEEK, how many hours in total did you spend socializing with others from the following groups? - Friends
    
    [1m[4mCONNECTION_social_time_neighbours_p7d_grouped[0m: In the PAST WEEK, how many hours in total did you spend socializing with others from the following groups? - Neighbours
    
    [1m[4mCOVID_prevention_avoid_trips[0m: To what extent are you currently following the COVID-19 prevention practices listed below? - Avoid non-essential trips in the community
    
    [1m[4mCOVID_prevention_distancing[0m: To what extent are you currently following the COVID-19 prevention practices listed below? - Physically distance yourself by 2 metres from others
    
    [1m[4mCOVID_prevention_hand_washing[0m: To what extent are you currently following the COVID-19 prevention practices listed below? - Wash your hands often
    
    [1m[4mCOVID_prevention_household[0m: To what extent are you currently following the COVID-19 prevention practices listed below? - Socialize indoors only with people in your immediate household
    
    [1m[4mCOVID_prevention_masks[0m: To what extent are you currently following the COVID-19 prevention practices listed below? - Wear a mask in public
    
    [1m[4mCOVID_prevention_reduce_people[0m: To what extent are you currently following the COVID-19 prevention practices listed below? - Reduce the number of people you interact with
    
    [1m[4mCOVID_vaccinated[0m: Have you received a COVID-19 Vaccine?
    
    [1m[4mELIGIBLE_consent[0m: Do you acknowledge and agree to the conditions outlined above?
    
    [1m[4mLONELY_change_pre_covid[0m: Comparing how you feel now to how you felt before the COVID-19 Pandemic, how would you describe the intensity of your loneliness?
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_close[0m: Please indicate for each of the statements, the extent to which they apply to your situation, the way you feel now. - There are enough people I feel close to
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_close[0m: For the statements below, please indicate the extent to which they apply to your situation and the way you feel now. - There are enough people I feel close to
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_emptiness[0m: For the statements below, please indicate the extent to which they apply to your situation and the way you feel now. - I experience a general sense of emptiness
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_emptiness[0m: Please indicate for each of the statements, the extent to which they apply to your situation, the way you feel now. - I experience a general sense of emptiness
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_miss[0m: Please indicate for each of the statements, the extent to which they apply to your situation, the way you feel now. - I miss having people around
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_miss[0m: For the statements below, please indicate the extent to which they apply to your situation and the way you feel now. - I miss having people around
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_rejected[0m: For the statements below, please indicate the extent to which they apply to your situation and the way you feel now. - I often feel rejected
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_rejected[0m: Please indicate for each of the statements, the extent to which they apply to your situation, the way you feel now. - I often feel rejected
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_rely[0m: Please indicate for each of the statements, the extent to which they apply to your situation, the way you feel now. - There are plenty of people I can rely on when I have problems
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_rely[0m: For the statements below, please indicate the extent to which they apply to your situation and the way you feel now. - There are plenty of people I can rely on when I have problems
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_trust[0m: Please indicate for each of the statements, the extent to which they apply to your situation, the way you feel now. - There are many people I can trust completely
    
    [1m[4mLONELY_dejong_emotional_social_loneliness_scale_trust[0m: For the statements below, please indicate the extent to which they apply to your situation and the way you feel now. - There are many people I can trust completely
    
    [1m[4mLONELY_direct[0m: During the PAST WEEK, have you felt lonely 
    
    [1m[4mLONELY_others_aware[0m: Generally speaking, do you think others are aware of the extent to which you feel lonely or connected?
    
    [1m[4mLONELY_ucla_loneliness_scale_companionship[0m: Indicate how often each of the statements below is descriptive of you. - How often do you feel that you lack companionship?
    
    [1m[4mLONELY_ucla_loneliness_scale_isolated[0m: Indicate how often each of the statements below is descriptive of you. - How often do you feel isolated from others?
    
    [1m[4mLONELY_ucla_loneliness_scale_left_out[0m: Indicate how often each of the statements below is descriptive of you. - How often do you feel left out?
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_comfort[0m: Please respond to the following items using the scale provided - I have a special person who is a real source of comfort to me
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_decisions[0m: Please respond to the following items using the scale provided - My friends and family are willing to help me make decisions
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_family_emotional[0m: Please respond to the following items using the scale provided - I get the emotional help and support I need from my family
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_family_helps[0m: Please respond to the following items using the scale provided - My family really tries to help me
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_feelings[0m: Please respond to the following items using the scale provided - There is a special person in my life who cares about my feelings
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_gets_me_not_scored[0m: Please respond to the following items using the scale provided - There is a person who really understands me on a deep level / gets me
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_go_wrong[0m: Please respond to the following items using the scale provided - I can count on my friends when things go wrong
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_help[0m: Please respond to the following items using the scale provided - My friends really try to help me.
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_joys[0m: Please respond to the following items using the scale provided - There is a special person with whom I can share my joys and sorrows
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_need[0m: Please respond to the following items using the scale provided - There is a special person who is around when I am in need.
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_positive_not_scored[0m: Please respond to the following items using the scale provided - There is a person who regularly makes me laugh, feel positive about myself
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_problems_family[0m: Please respond to the following items using the scale provided - I can talk about my problems with my family
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_problems_friends[0m: Please respond to the following items using the scale provided - I can talk about my problems with my friends
    
    [1m[4mPSYCH_zimet_multidimensional_social_support_scale_sorrows[0m: Please respond to the following items using the scale provided - I have friends or family members with whom I can share my joys and sorrows
    
    [1m[4mSURVEY_distribution[0m: Distribution Channel
    
    [1m[4mSURVEY_duration_seconds[0m: Duration (in seconds)
    
    [1m[4mSURVEY_end_date[0m: End Date
    
    [1m[4mSURVEY_finished[0m: Finished
    
    [1m[4mSURVEY_progress[0m: Progress
    
    [1m[4mSURVEY_recontact[0m: Is it okay for us to re-contact you to invite you to participate in next year's survey using the email address provided above? Note: Doing so will allow us to collect changes in response patterns over time to assess how your social life is changing in the post-pandemic period.
    
    [1m[4mSURVEY_recontact[0m: Is it okay for us to re-contact you about potentially participating in an additional follow-up survey using this email address? Note: Doing so will allow us to collect changes in response patterns over time to assess how your social life is changing in the post-pandemic period.
    
    [1m[4mSURVEY_recontact[0m: Is it okay for us to re-contact you about potentially participating in an additional follow-up survey using this email address?   Note: Doing so will allow us to collect changes in response patterns over time to assess how your social life is changing in the post-pandemic period.
    
    [1m[4mSURVEY_recorded_date[0m: Recorded Date
    
    [1m[4mSURVEY_response_id[0m: Response ID
    
    [1m[4mSURVEY_response_type[0m: Response Type
    
    [1m[4mSURVEY_start_date[0m: Start Date
    
    [1m[4mSURVEY_user_language[0m: User Language
    
    [1m[4mWELLNESS_gad_anxious[0m: Over the PAST TWO WEEKS, how often - have you felt nervous, anxious or on edge?
    
    [1m[4mWELLNESS_gad_worry[0m: Over the PAST TWO WEEKS, how often - were you not able to stop worrying or control your worries?
    
    [1m[4mWELLNESS_life_satisfaction[0m: On a scale of 1 to 10, How do you feel about your life as a whole right now?
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_depressed[0m: When you think about your work overall, how often do you feel the following? - Depressed
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_depressed[0m: When you think about your life overall, how often do you feel the following? - Depressed
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_difficulty_sleeping[0m: When you think about your work overall, how often do you feel the following? - Difficulties sleeping
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_difficulty_sleeping[0m: When you think about your life overall, how often do you feel the following? - Difficulties sleeping
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_disappointed[0m: When you think about your life overall, how often do you feel the following? - Disappointed with people
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_disappointed[0m: When you think about your work overall, how often do you feel the following? - Disappointed with people
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_had_it[0m: When you think about your life overall, how often do you feel the following? - I've had it
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_had_it[0m: When you think about your work overall, how often do you feel the following? - œIve had it
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_had_it[0m: When you think about your life overall, how often do you feel the following? - œIve had it
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_helpless[0m: When you think about your life overall, how often do you feel the following? - Helpless
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_helpless[0m: When you think about your work overall, how often do you feel the following? - Helpless
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_hopeless[0m: When you think about your life overall, how often do you feel the following? - Hopeless
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_hopeless[0m: When you think about your work overall, how often do you feel the following? - Hopeless
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_sick[0m: When you think about your life overall, how often do you feel the following? - Physically weak or sickly
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_sick[0m: When you think about your work overall, how often do you feel the following? - Physically weak or sickly
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_tired[0m: When you think about your life overall, how often do you feel the following? - Tired
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_tired[0m: When you think about your work overall, how often do you feel the following? - Tired
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_trapped[0m: When you think about your life overall, how often do you feel the following? - Trapped
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_trapped[0m: When you think about your work overall, how often do you feel the following? - Trapped
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_worthless[0m: When you think about your life overall, how often do you feel the following? - Worthless or like a failure
    
    [1m[4mWELLNESS_malach_pines_burnout_measure_worthless[0m: When you think about your work overall, how often do you feel the following? - Worthless or like a failure
    
    [1m[4mWELLNESS_phq_feeling_down[0m: Over the PAST TWO WEEKS, how often - were you feeling down, depressed or hopeless?
    
    [1m[4mWELLNESS_phq_little_interest[0m: Over the PAST TWO WEEKS, how often - have you had little interest or pleasure in doing things?
    
    [1m[4mWELLNESS_self_rated_mental_health[0m: At the present time, would you say your MENTAL HEALTH is:
    
    [1m[4mWELLNESS_self_rated_physical_health[0m: At the present time, would you say your PHYSICAL HEALTH is:
    
    [1m[4mWELLNESS_subjective_happiness_scale_always_happy[0m: Some people are generally very happy. They enjoy life regardless of what is going on, getting the most out of everything. To what extent does this characterization describe you?
    
    [1m[4mWELLNESS_subjective_happiness_scale_happy[0m: In general, I consider myself:
    
    [1m[4mWELLNESS_subjective_happiness_scale_never_happy[0m: Some people are generally not very happy. Although they are not depressed, they never seem as happy as they might be. To what extent does this characterization describe you?
    
    [1m[4mWELLNESS_subjective_happiness_scale_peers[0m: Compared with most of my peers, I consider myself:
    



```python
# Some columns are not included in the "list of available variables"
# but based on the available explanations their meaning is very clear
# for i,c in enumerate(dataV2_cohortV3.columns):
#    if not c in cols.new_var.values:
#        print(i,c)    
```

### Now for some comparisons...

Let's see if the number of seconds it takes to answer questions on average differs between the cross-sectional and cohort versions of the studies


```python
# REMINDER: our data includes participants who are part of the cohort data 
# But also includes cross-sectional data collected from them
dataV2_cohortV3.groupby('SURVEY_collection_type').size()
```




    SURVEY_collection_type
    cohort    477
    cross     373
    dtype: int64




```python
import plotly.express as px

fig = px.box(dataV2_cohortV3[dataV2_cohortV3.Secs_per_q<30], 
             x='SURVEY_collection_type', y='Secs_per_q')
fig.update_layout(xaxis_title='SURVEY_YEAR', yaxis_title='Secs_per_q',
                  title='Boxplot of Secs_per_q by SURVEY_YEAR').show()
```


        <script type="text/javascript">
        window.PlotlyConfig = {MathJaxConfig: 'local'};
        if (window.MathJax && window.MathJax.Hub && window.MathJax.Hub.Config) {window.MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}
        if (typeof require !== 'undefined') {
        require.undef("plotly");
        define('plotly', function(require, exports, module) {
            /**
* plotly.js v2.32.0
* Copyright 2012-2024, Plotly, Inc.
* All rights reserved.
* Licensed under the MIT license
*/
/*! For license information please see plotly.min.js.LICENSE.txt */
        });
        require(['plotly'], function(Plotly) {
            window._Plotly = Plotly;
        });
        }
        </script>




<div>                            <div id="852a0dfa-8eeb-4ac5-9a97-c129391b007f" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("852a0dfa-8eeb-4ac5-9a97-c129391b007f")) {                    Plotly.newPlot(                        "852a0dfa-8eeb-4ac5-9a97-c129391b007f",                        [{"alignmentgroup":"True","hovertemplate":"SURVEY_collection_type=%{x}\u003cbr\u003eSecs_per_q=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"","marker":{"color":"#636efa"},"name":"","notched":false,"offsetgroup":"","orientation":"v","showlegend":false,"x":["cross","cohort","cohort","cohort","cross","cross","cohort","cross","cohort","cross","cohort","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cohort","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cohort","cross","cross","cohort","cross","cohort","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cohort","cross","cohort","cross","cohort","cohort","cohort","cohort","cross","cross","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cohort","cross","cross","cross","cohort","cohort","cross","cohort","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cohort","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cohort","cross","cross","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cohort","cross","cross","cohort","cross","cohort","cohort","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cross","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cohort","cohort","cross","cross","cross","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cross","cross","cohort","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cohort","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cohort","cohort","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cross","cross","cohort","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cross","cross","cross","cohort","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cohort","cross","cohort","cross","cross","cohort","cohort","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cohort","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cohort","cohort","cohort","cross","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cohort","cross","cross","cross","cohort","cohort","cross","cross","cohort","cohort","cohort","cross","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cross","cross","cohort","cohort","cross","cohort","cohort","cohort","cohort","cross","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cohort","cross","cross","cross","cohort","cross","cohort","cross","cohort","cross","cross","cohort","cohort","cohort","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cross","cross","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cohort","cross","cross","cohort","cross","cohort","cohort","cohort","cross","cohort","cohort","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cohort","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cross","cohort","cross","cross","cohort","cohort","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cohort","cross","cross","cohort","cross","cohort","cross","cohort","cohort","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cohort","cross","cross","cohort","cross","cohort","cross","cross","cohort","cohort","cohort","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cross","cross","cohort","cross","cohort","cross","cohort","cross","cohort","cohort","cross","cross","cohort","cohort","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cross","cohort","cohort","cross","cohort","cross","cohort","cohort","cross","cohort","cohort","cohort","cross"],"x0":" ","xaxis":"x","y":[6.018939393939394,6.017964071856287,5.493975903614458,7.452380952380952,14.885793871866296,5.128919860627177,4.377483443708609,6.861940298507463,17.54054054054054,4.485294117647059,3.690476190476191,3.4107142857142856,4.954248366013072,4.451492537313433,3.562874251497006,4.585635359116022,5.727941176470588,7.670886075949367,11.90079365079365,7.125461254612546,5.232142857142857,4.209876543209877,5.144578313253012,6.595238095238095,7.662721893491124,4.335329341317365,5.734982332155477,4.669467787114846,4.815476190476191,4.480314960629921,3.4642857142857144,8.30952380952381,7.03485254691689,5.875739644970414,8.963636363636363,4.5054945054945055,4.395209580838324,6.398809523809524,5.068100358422939,2.8982035928143715,2.3902439024390243,5.311377245508982,4.421052631578948,5.221556886227545,6.428571428571429,5.464944649446495,4.8389261744966445,3.4790419161676644,5.50415512465374,4.892857142857143,6.08641975308642,18.38611111111112,7.671309192200557,6.9880239520958085,4.010489510489511,4.3532934131736525,2.766467065868264,5.404761904761905,8.562248995983936,5.910714285714286,5.75,6.8014440433213,15.625,4.494505494505495,4.466666666666667,4.622302158273381,7.355805243445693,7.2155688622754495,12.874251497005988,3.581881533101045,6.502994011976048,5.946327683615819,8.167272727272728,5.275449101796407,8.36,7.239520958083832,3.8854166666666665,28.281437125748504,6.353159851301116,4.706586826347305,8.126506024096386,9.428057553956837,4.111702127659575,8.590361445783133,9.938223938223938,3.813186813186813,3.502958579881657,4.470238095238095,5.442666666666667,4.862275449101796,7.737226277372263,5.897435897435898,6.738095238095238,3.5,6.794594594594595,4.005917159763314,6.226190476190476,6.9324324324324325,3.5207100591715976,8.005952380952381,17.861333333333334,5.402366863905326,5.074074074074074,5.289156626506024,6.794701986754967,8.753623188405797,6.531135531135531,7.514970059880239,3.934523809523809,5.188153310104529,4.662068965517242,5.315018315018315,6.297619047619048,9.308270676691729,5.517857142857143,6.185185185185185,3.029940119760479,3.335329341317365,4.431578947368421,5.630952380952381,6.003717472118959,8.27605633802817,9.326530612244898,10.214285714285714,3.7857142857142856,13.348754448398576,5.06508875739645,26.535714285714285,12.82638888888889,5.335766423357664,3.8632478632478633,4.079268292682927,2.814569536423841,4.52280701754386,6.712574850299402,6.643382352941177,2.8493975903614457,3.267857142857143,4.672619047619048,3.305389221556886,4.48314606741573,4.726775956284153,4.532934131736527,7.73090909090909,5.562874251497006,5.6,7.5212121212121215,9.42528735632184,7.494047619047619,3.4730538922155687,4.335329341317365,4.237037037037037,4.637037037037037,7.946236559139785,3.155688622754491,3.841059602649007,5.482758620689655,6.142011834319527,29.892857142857142,4.317343173431734,3.867595818815331,4.648809523809524,6.512820512820513,7.517857142857143,13.269461077844312,4.43501326259947,8.289855072463768,14.0188679245283,6.545454545454546,7.14859437751004,3.0119760479041915,7.014440433212997,3.4761904761904763,5.745205479452054,13.64,7.601190476190476,5.892215568862276,7.322344322344322,6.583333333333333,6.306273062730627,8.012121212121212,12.00657894736842,6.175824175824176,7.252032520325203,4.735099337748345,2.6508875739644973,4.974074074074074,6.339285714285714,4.770718232044199,4.446428571428571,13.858181818181816,3.7901234567901234,3.321428571428572,5.890510948905109,7.319148936170213,4.880952380952381,4.048484848484849,4.355421686746988,4.355311355311355,10.524691358024691,9.25263157894737,9.3,5.203592814371257,4.647058823529412,5.092696629213483,8.34,9.04191616766467,7.595505617977528,4.993055555555555,5.131736526946108,14.01418439716312,7.449101796407185,9.581314878892734,6.834558823529412,7.886904761904762,7.293447293447294,6.75,7.3053892215568865,4.526690391459074,5.706586826347305,4.76158940397351,4.620218579234972,7.389221556886228,8.919753086419753,8.958015267175572,5.029761904761905,5.01418439716312,5.391791044776119,7.7555555555555555,7.244047619047619,4.992700729927007,8.125748502994012,6.850299401197605,8.760479041916168,8.192028985507246,7.258741258741258,5.690476190476191,6.364912280701755,4.643617021276596,2.508982035928144,4.267857142857143,4.077821011673152,5.287272727272727,5.167664670658683,11.308510638297872,8.0,5.269461077844311,7.483394833948339,5.598930481283422,5.827814569536423,6.039735099337748,3.616766467065869,3.685920577617329,5.8273809523809526,4.212765957446808,8.081180811808117,20.36309523809524,8.244755244755245,12.267857142857142,7.0,10.235294117647058,5.757894736842105,4.4431137724550895,5.339285714285714,5.05614973262032,5.603550295857988,6.434523809523809,7.3527851458885936,4.508982035928144,3.4251497005988023,2.898809523809524,3.872262773722628,5.180505415162455,22.06159420289855,7.012121212121212,6.776515151515151,3.6867469879518073,4.568840579710145,4.913793103448276,11.653284671532846,5.088757396449704,10.63095238095238,11.944852941176473,4.48,5.8533333333333335,14.0,2.522727272727273,7.208333333333333,5.411764705882353,5.2155688622754495,5.976923076923077,4.150537634408602,2.820359281437126,4.287425149700598,9.439024390243905,14.012048192771084,5.9298245614035086,5.18562874251497,3.916955017301038,2.113207547169812,4.325443786982248,4.204301075268817,5.242603550295858,4.806451612903226,5.65934065934066,6.089285714285714,4.978260869565218,3.9820359281437128,8.297619047619047,9.6793893129771,6.321428571428571,6.779850746268656,11.136094674556212,3.7202380952380953,6.355029585798817,5.444444444444445,6.595238095238095,2.98224852071006,3.0422535211267605,4.892857142857143,5.1530054644808745,14.618705035971225,7.179640718562874,5.953642384105961,6.111111111111111,4.047337278106509,4.770609318996415,4.377245508982036,11.156133828996282,10.265060240963855,3.7142857142857135,4.332129963898917,6.916167664670659,6.031914893617022,5.412121212121212,7.114206128133705,4.91970802919708,5.466666666666667,5.791970802919708,3.179640718562874,4.454212454212454,4.114982578397212,4.686746987951807,6.047619047619048,6.789041095890411,8.479553903345725,3.42603550295858,11.710144927536232,10.727272727272728,2.696969696969697,3.9964028776978417,4.087193460490464,3.494047619047619,6.282208588957055,3.485207100591716,4.188405797101449,4.148809523809524,6.15625,5.255952380952381,3.1845238095238093,7.648979591836735,3.760479041916168,4.03971119133574,5.921686746987952,7.143243243243243,6.106761565836299,2.679245283018868,5.178832116788321,10.50595238095238,4.617647058823529,5.041916167664671,4.452830188679245,7.177935943060498,5.911347517730497,14.253164556962023,6.690476190476191,4.166666666666667,4.740524781341108,4.2155688622754495,6.07185628742515,4.051282051282051,4.535714285714286,5.136904761904762,3.4730538922155687,4.108433734939759,4.92962962962963,6.047619047619048,8.149732620320856,4.059880239520958,4.529745042492918,15.0,11.40828402366864,9.005952380952381,4.401197604790419,4.137724550898204,7.536144578313253,7.379204892966361,7.514792899408284,5.3017751479289945,5.090277777777778,9.100719424460433,8.530909090909091,7.0198675496688745,7.811267605633803,6.57396449704142,6.157706093189964,11.55072463768116,9.04191616766467,6.42603550295858,5.758620689655173,4.892857142857143,5.234657039711191,4.5212121212121215,9.089820359281438,4.602189781021898,5.167664670658683,5.826771653543307,8.570469798657719,9.545454545454543,5.430604982206406,3.803571428571429,9.478571428571428,6.591603053435114,5.281437125748503,9.274074074074074,5.8273809523809526,5.4523809523809526,6.872521246458923,7.634057971014493,5.303571428571429,2.808383233532934,4.4163701067615655,2.9047619047619047,4.101449275362318,5.174545454545455,5.622754491017964,3.15018315018315,3.005952380952381,3.7916666666666665,5.017421602787456,7.938028169014085,9.754966887417218,22.08333333333333,5.103703703703704,3.5748502994011977,3.729824561403509,3.712574850299401,15.0,2.765957446808511,5.173652694610778,6.232209737827716,8.390728476821192,8.569767441860465,5.061224489795919,8.643874643874645,8.172619047619047,7.391566265060241,7.988593155893536,6.257950530035336,6.705521472392638,4.656593406593407,3.575757575757576,7.797619047619047,5.802139037433155,11.984674329501916,6.468503937007874,6.668539325842697,6.875862068965517,5.391891891891892,6.502994011976048,5.786764705882353,11.24390243902439,8.538922155688622,3.862903225806452,3.730538922155689,3.9328358208955225,8.458333333333334,2.729946524064171,13.88095238095238,6.36013986013986,3.502994011976048,7.058608058608058,3.345454545454545,3.4285714285714284,2.40119760479042,4.863095238095238,7.011904761904762,20.848238482384826,5.527272727272727,5.800738007380073,4.927536231884058,4.455089820359281,3.157303370786517,3.0416666666666665,4.618181818181818,4.112426035502959,4.935251798561152,4.095238095238095,5.784431137724551,7.099290780141844,7.928571428571429,13.011904761904765,5.654761904761905,4.96309963099631,4.225201072386059,2.7976190476190474,4.628975265017668,3.9047619047619047,17.404761904761905,4.29757785467128,5.203592814371257,4.547619047619048,20.225,4.822222222222222,4.568862275449102,2.5828877005347595,3.1011904761904763,7.946996466431095,5.898809523809524,7.501930501930502,5.053571428571429,10.393939393939394,8.221428571428572,5.644444444444445,4.389221556886228,5.556213017751479,6.153333333333333,5.245508982035928,4.282442748091603,8.476190476190476,8.936170212765957,5.398809523809524,5.394052044609666,7.225092250922509,7.029761904761905,4.935064935064935,4.166666666666667,3.4918032786885247,3.9288389513108615,4.263473053892215,5.358490566037736,15.967857142857143,9.488095238095235,3.131736526946108,3.8865248226950353,4.359281437125748,5.050802139037433,7.52589641434263,7.850299401197605,5.723756906077348,7.880952380952381,5.630952380952381,7.291208791208791,8.594488188976378,5.766666666666667,5.969512195121951,3.75,3.658682634730539,5.285714285714286,4.128676470588236,6.922619047619048,5.314685314685315,7.860902255639098,4.595505617977528,13.577380952380953,8.806010928961749,4.680147058823529,6.601078167115903,5.072289156626506,9.450704225352112,3.9130434782608696,2.811320754716981,2.7142857142857144,2.4484848484848483,4.682634730538922,2.25,4.387267904509284,6.431137724550898,18.29341317365269,4.827683615819209,4.793333333333333,5.029761904761905,3.325925925925926,5.821428571428571,7.190114068441065,7.732782369146006,3.155688622754491,3.695035460992908,5.060975609756097,6.179389312977099,7.509933774834437,8.761538461538462,5.492957746478873,6.226190476190476,5.162878787878788,6.03030303030303,10.529411764705882,6.178571428571429,11.601503759398495,29.0,3.857142857142857,2.962264150943396,5.205882352941177,5.456989247311828,8.121771217712178,6.136094674556213,3.2111111111111112,3.167664670658682,3.5681818181818183,2.783132530120482,2.992857142857143,6.379679144385027,10.970238095238097,7.005952380952381,4.184523809523809,4.839285714285714,5.0,3.494047619047619,9.057971014492754,17.583333333333332,13.251497005988025,6.768939393939394,5.964497041420119,4.017964071856287,4.711484593837535,3.679245283018868,4.524079320113314,5.657718120805369,6.443636363636363,6.677536231884058,4.9523809523809526,7.934523809523809,4.591240875912408,7.745562130177515,9.46341463414634,4.104693140794224,4.666666666666667,5.574850299401198,5.003584229390681,4.556886227544911,5.056756756756757,4.76,4.383512544802867,5.173652694610778,11.450757575757576,7.503225806451613,3.894409937888199,5.277153558052435,5.389221556886228,6.67590027700831,4.196428571428571,3.5384615384615383,3.9192982456140353,2.636904761904762,5.666666666666667,5.162361623616236,5.751851851851852,4.5,8.041353383458647,7.754966887417218,3.950354609929078,5.718562874251497,7.795620437956204,5.598802395209581,2.25,4.451807228915663,6.087912087912088,6.264705882352941,4.784431137724551,7.612994350282486,7.191358024691358,3.2994011976047903,5.081081081081081,3.413173652694611,5.406666666666666,23.0,4.824817518248175,2.9880239520958085,3.458041958041958,4.681318681318682,6.885542168674699,3.021818181818182,2.3493975903614457,2.2806539509536785,5.431952662721893,7.338289962825279,5.333333333333333,14.946428571428571,18.377245508982035,5.607142857142857,5.605633802816901,6.976047904191617,4.251497005988024,5.681159420289855,8.105660377358491,7.735849056603773,10.060240963855422,4.532934131736527,6.003496503496503,5.081180811808118,9.467455621301776,4.970909090909091,3.862275449101797,5.784386617100372,8.433962264150944,4.386904761904762,10.893333333333333,11.430434782608696,6.369047619047619,5.419014084507042,6.976190476190476,8.113772455089821,7.155038759689923,5.724550898203593,5.062937062937063,5.152416356877324,3.6219512195121952,4.428571428571429,3.767857142857143,4.104529616724738,6.100671140939597,5.394957983193278,2.757575757575758,2.952380952380953,7.688715953307393,8.680473372781066,4.565476190476191,5.942652329749104,3.712574850299401,3.519434628975265,3.5502958579881656,5.67379679144385,2.386904761904762,3.646706586826348,4.7560975609756095,5.107142857142857,2.964071856287425,3.269333333333333,4.2075471698113205,5.836619718309859,11.25650557620818,3.9820359281437128,3.794776119402985,3.921686746987952,4.491017964071856,4.889705882352941,4.5190311418685125,4.445783132530121,3.452830188679245,4.031358885017422,3.9107142857142856,4.850909090909091,15.303571428571429,7.462686567164179,5.712328767123288,4.683333333333334,10.819787985865725,5.401197604790419,8.079545454545455,5.679045092838196,5.404761904761905,4.071428571428571,9.75,4.92,5.101190476190476,4.538922155688622,3.9480968858131487,4.802395209580839,3.928571428571429,4.772893772893773,10.0,5.363095238095238,4.238095238095238,3.7142857142857135,4.163498098859316,6.95910780669145,21.512048192771083,4.3293413173652695,13.110714285714286,3.6503496503496495,8.523985239852399,4.811320754716981,7.419475655430712,5.803921568627451,10.942084942084945,10.5,7.137546468401487,6.880239520958084,5.590361445783133,7.992700729927007,5.482573726541555,3.910179640718563,2.2710843373493974,3.4107142857142856,3.848591549295776,9.5,5.488095238095238,3.712574850299401,4.517786561264822,5.449704142011834,5.057142857142857,5.714673913043479,7.969512195121951,5.006024096385542,9.222641509433965,6.863095238095238,7.351449275362318,8.376543209876543,11.946107784431138,5.793706293706293,4.325443786982248,4.988095238095238,5.07185628742515,3.691176470588236],"y0":" ","yaxis":"y","type":"box"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"SURVEY_YEAR"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Secs_per_q"}},"legend":{"tracegroupgap":0},"margin":{"t":60},"boxmode":"group","title":{"text":"Boxplot of Secs_per_q by SURVEY_YEAR"}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('852a0dfa-8eeb-4ac5-9a97-c129391b007f');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


As seen from the code below (since "cross-sectional" is always in 2021 while "cohort" is always in 2022), it seems participants first did the cross-sectional survey and were then recruited into the cohort study the following year; so, the cohort study would have been the second time the participants completed the survey... so might we wonder if they were faster the second time around? 


```python
dataV2_cohortV4 = dataV2_cohortV3[dataV2_cohortV3.Secs_per_q<30].copy()
dataV2_cohortV4.loc[:, 'SURVEY_YEAR'] = (dataV2_cohortV4.SURVEY_collection_type+" "+dataV2_cohortV4.SURVEY_collection_year.astype(str)).values
# dataV2_cohortV4.SURVEY_YEAR.value_counts()
# SURVEY_YEAR
# cohort 2022    446
# cross 2021     368

# we'll focus on some of the numeric outcomes in the data set... including Secs_per_q
dataV2_cohortV4_wide = \
dataV2_cohortV4.melt(id_vars=['UNIQUE_id','SURVEY_YEAR'], 
                     value_vars=['Secs_per_q',
                                'LONELY_ucla_loneliness_scale_score',
                                'WELLNESS_life_satisfaction', 
                                'WELLNESS_malach_pines_burnout_measure_score',
                                'LONELY_dejong_emotional_social_loneliness_scale_score',
                                'LONELY_dejong_emotional_loneliness_sub_scale_score',
                                'LONELY_dejong_social_loneliness_sub_scale_score',
                                'PSYCH_zimet_multidimensional_social_support_scale_score',
                                'PSYCH_zimet_multidimensional_social_support_family_subscale_score',
                                'PSYCH_zimet_multidimensional_social_support_significant_other_subscale_score',
                                'PSYCH_zimet_multidimensional_social_support_friends_subscale_score',
                                'WELLNESS_subjective_happiness_scale_score',
                                'WELLNESS_phq_score', 'WELLNESS_gad_score'])
# and we'll put these in columns for "cross 2021" and "cohort 2022"
dataV2_cohortV4_wide['variable'] = dataV2_cohortV4_wide['variable'] + " ("+dataV2_cohortV4_wide['SURVEY_YEAR']+")" 
# on the basis of UNIQUE_id which links individuals across studies
dataV2_cohortV4_wide = dataV2_cohortV4_wide.pivot(index='UNIQUE_id', columns='variable', values='value')
# and consider fully observed data only
dataV2_cohortV4_wideV2 = dataV2_cohortV4_wide.dropna() 
dataV2_cohortV4_wideV2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>variable</th>
      <th>LONELY_dejong_emotional_loneliness_sub_scale_score (cohort 2022)</th>
      <th>LONELY_dejong_emotional_loneliness_sub_scale_score (cross 2021)</th>
      <th>LONELY_dejong_emotional_social_loneliness_scale_score (cohort 2022)</th>
      <th>LONELY_dejong_emotional_social_loneliness_scale_score (cross 2021)</th>
      <th>LONELY_dejong_social_loneliness_sub_scale_score (cohort 2022)</th>
      <th>LONELY_dejong_social_loneliness_sub_scale_score (cross 2021)</th>
      <th>LONELY_ucla_loneliness_scale_score (cohort 2022)</th>
      <th>LONELY_ucla_loneliness_scale_score (cross 2021)</th>
      <th>PSYCH_zimet_multidimensional_social_support_family_subscale_score (cohort 2022)</th>
      <th>PSYCH_zimet_multidimensional_social_support_family_subscale_score (cross 2021)</th>
      <th>PSYCH_zimet_multidimensional_social_support_friends_subscale_score (cohort 2022)</th>
      <th>PSYCH_zimet_multidimensional_social_support_friends_subscale_score (cross 2021)</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_score (cohort 2022)</th>
      <th>PSYCH_zimet_multidimensional_social_support_scale_score (cross 2021)</th>
      <th>PSYCH_zimet_multidimensional_social_support_significant_other_subscale_score (cohort 2022)</th>
      <th>PSYCH_zimet_multidimensional_social_support_significant_other_subscale_score (cross 2021)</th>
      <th>Secs_per_q (cohort 2022)</th>
      <th>Secs_per_q (cross 2021)</th>
      <th>WELLNESS_gad_score (cohort 2022)</th>
      <th>WELLNESS_gad_score (cross 2021)</th>
      <th>WELLNESS_life_satisfaction (cohort 2022)</th>
      <th>WELLNESS_life_satisfaction (cross 2021)</th>
      <th>WELLNESS_malach_pines_burnout_measure_score (cohort 2022)</th>
      <th>WELLNESS_malach_pines_burnout_measure_score (cross 2021)</th>
      <th>WELLNESS_phq_score (cohort 2022)</th>
      <th>WELLNESS_phq_score (cross 2021)</th>
      <th>WELLNESS_subjective_happiness_scale_score (cohort 2022)</th>
      <th>WELLNESS_subjective_happiness_scale_score (cross 2021)</th>
    </tr>
    <tr>
      <th>UNIQUE_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>cscs_00021</th>
      <td>2.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>6.50</td>
      <td>5.75</td>
      <td>6.75</td>
      <td>5.25</td>
      <td>6.750000</td>
      <td>6.000000</td>
      <td>7.00</td>
      <td>7.00</td>
      <td>6.017964</td>
      <td>6.018939</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>2.4</td>
      <td>2.6</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>5.25</td>
      <td>5.00</td>
    </tr>
    <tr>
      <th>cscs_00080</th>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>6.0</td>
      <td>6.0</td>
      <td>4.00</td>
      <td>4.50</td>
      <td>5.00</td>
      <td>5.25</td>
      <td>4.916667</td>
      <td>5.333333</td>
      <td>5.75</td>
      <td>6.25</td>
      <td>7.452381</td>
      <td>14.885794</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>8.0</td>
      <td>9.0</td>
      <td>3.2</td>
      <td>2.4</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>6.00</td>
      <td>6.00</td>
    </tr>
    <tr>
      <th>cscs_00081</th>
      <td>3.0</td>
      <td>3.0</td>
      <td>6.0</td>
      <td>6.0</td>
      <td>3.0</td>
      <td>3.0</td>
      <td>9.0</td>
      <td>9.0</td>
      <td>4.50</td>
      <td>3.50</td>
      <td>4.25</td>
      <td>2.25</td>
      <td>4.750000</td>
      <td>3.750000</td>
      <td>5.50</td>
      <td>5.50</td>
      <td>4.377483</td>
      <td>5.128920</td>
      <td>4.0</td>
      <td>4.0</td>
      <td>8.0</td>
      <td>2.0</td>
      <td>3.1</td>
      <td>4.9</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>4.50</td>
      <td>2.75</td>
    </tr>
    <tr>
      <th>cscs_00114</th>
      <td>1.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>8.0</td>
      <td>6.0</td>
      <td>6.00</td>
      <td>5.75</td>
      <td>5.75</td>
      <td>5.50</td>
      <td>5.583333</td>
      <td>5.416667</td>
      <td>5.00</td>
      <td>5.00</td>
      <td>3.690476</td>
      <td>4.485294</td>
      <td>0.0</td>
      <td>6.0</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>4.6</td>
      <td>3.7</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>4.50</td>
      <td>4.25</td>
    </tr>
    <tr>
      <th>cscs_00204</th>
      <td>3.0</td>
      <td>3.0</td>
      <td>6.0</td>
      <td>6.0</td>
      <td>3.0</td>
      <td>3.0</td>
      <td>8.0</td>
      <td>9.0</td>
      <td>1.25</td>
      <td>4.50</td>
      <td>6.25</td>
      <td>7.00</td>
      <td>3.916667</td>
      <td>5.666667</td>
      <td>4.25</td>
      <td>5.50</td>
      <td>4.954248</td>
      <td>4.451493</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>6.0</td>
      <td>2.0</td>
      <td>4.8</td>
      <td>3.6</td>
      <td>3.0</td>
      <td>4.0</td>
      <td>3.00</td>
      <td>5.00</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>cscs_11699</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>3.0</td>
      <td>5.50</td>
      <td>5.50</td>
      <td>5.75</td>
      <td>6.25</td>
      <td>6.083333</td>
      <td>6.250000</td>
      <td>7.00</td>
      <td>7.00</td>
      <td>7.969512</td>
      <td>5.714674</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>10.0</td>
      <td>3.0</td>
      <td>2.9</td>
      <td>3.5</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>5.25</td>
      <td>6.00</td>
    </tr>
    <tr>
      <th>cscs_11724</th>
      <td>2.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>5.0</td>
      <td>5.0</td>
      <td>6.25</td>
      <td>5.00</td>
      <td>5.00</td>
      <td>5.00</td>
      <td>6.083333</td>
      <td>5.666667</td>
      <td>7.00</td>
      <td>7.00</td>
      <td>6.863095</td>
      <td>9.222642</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>7.0</td>
      <td>8.0</td>
      <td>2.1</td>
      <td>3.3</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>5.25</td>
      <td>5.25</td>
    </tr>
    <tr>
      <th>cscs_11748</th>
      <td>3.0</td>
      <td>3.0</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>9.0</td>
      <td>9.0</td>
      <td>3.00</td>
      <td>1.50</td>
      <td>4.00</td>
      <td>3.00</td>
      <td>4.083333</td>
      <td>3.166667</td>
      <td>5.25</td>
      <td>5.00</td>
      <td>8.376543</td>
      <td>7.351449</td>
      <td>4.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>4.6</td>
      <td>5.4</td>
      <td>4.0</td>
      <td>1.0</td>
      <td>4.00</td>
      <td>3.00</td>
    </tr>
    <tr>
      <th>cscs_11760</th>
      <td>3.0</td>
      <td>3.0</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>7.0</td>
      <td>9.0</td>
      <td>1.75</td>
      <td>1.75</td>
      <td>5.50</td>
      <td>4.75</td>
      <td>4.583333</td>
      <td>3.916667</td>
      <td>6.50</td>
      <td>5.25</td>
      <td>4.325444</td>
      <td>5.793706</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>7.0</td>
      <td>7.0</td>
      <td>4.6</td>
      <td>4.2</td>
      <td>3.0</td>
      <td>3.0</td>
      <td>3.50</td>
      <td>3.00</td>
    </tr>
    <tr>
      <th>cscs_11812</th>
      <td>2.0</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>4.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>7.0</td>
      <td>5.0</td>
      <td>3.25</td>
      <td>4.00</td>
      <td>4.25</td>
      <td>4.00</td>
      <td>4.833333</td>
      <td>4.333333</td>
      <td>7.00</td>
      <td>5.00</td>
      <td>5.071856</td>
      <td>3.691176</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>7.0</td>
      <td>4.7</td>
      <td>2.3</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>3.75</td>
      <td>4.50</td>
    </tr>
  </tbody>
</table>
<p>283 rows × 28 columns</p>
</div>




```python
# This could be used for the proportion of a paired differences test...
(dataV2_cohortV4_wideV2['Secs_per_q (cohort 2022)']>
 dataV2_cohortV4_wideV2['Secs_per_q (cross 2021)']).sum(), dataV2_cohortV4_wideV2.shape[0]
```




    (104, 283)




```python
# This could be used for the proportion for a test of "random coin flipping" 
col = 'LONELY_dejong_emotional_loneliness_sub_scale_score (cohort 2022)'
(dataV2_cohortV4_wideV2[col]>1).sum(), dataV2_cohortV4_wideV2.shape[0]
```




    (153, 283)




```python
dataV2_cohortV4_wideV2['LONELY_dejong_emotional_loneliness_sub_scale_score (cohort 2022)'].value_counts()
```




    LONELY_dejong_emotional_loneliness_sub_scale_score (cohort 2022)
    2.0    84
    1.0    76
    3.0    69
    0.0    54
    Name: count, dtype: int64




```python
px.histogram(dataV2_cohortV4_wideV2, x='LONELY_dejong_emotional_loneliness_sub_scale_score (cohort 2022)')
```


<div>                            <div id="f138069c-c106-4eef-ab7c-f4ab367dad9e" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("f138069c-c106-4eef-ab7c-f4ab367dad9e")) {                    Plotly.newPlot(                        "f138069c-c106-4eef-ab7c-f4ab367dad9e",                        [{"alignmentgroup":"True","bingroup":"x","hovertemplate":"LONELY_dejong_emotional_loneliness_sub_scale_score (cohort 2022)=%{x}\u003cbr\u003ecount=%{y}\u003cextra\u003e\u003c\u002fextra\u003e","legendgroup":"","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"","offsetgroup":"","orientation":"v","showlegend":false,"x":[2.0,1.0,3.0,1.0,3.0,2.0,0.0,0.0,0.0,1.0,2.0,1.0,1.0,0.0,1.0,1.0,1.0,1.0,1.0,2.0,2.0,3.0,0.0,2.0,1.0,0.0,1.0,2.0,2.0,3.0,2.0,3.0,1.0,1.0,2.0,2.0,3.0,2.0,3.0,0.0,2.0,1.0,3.0,0.0,0.0,2.0,2.0,1.0,3.0,1.0,1.0,2.0,1.0,1.0,3.0,0.0,2.0,2.0,2.0,1.0,1.0,1.0,3.0,3.0,0.0,2.0,3.0,3.0,3.0,0.0,2.0,0.0,0.0,1.0,1.0,3.0,1.0,3.0,2.0,0.0,0.0,3.0,2.0,3.0,1.0,3.0,3.0,2.0,2.0,1.0,2.0,3.0,1.0,2.0,3.0,2.0,3.0,2.0,3.0,0.0,0.0,1.0,1.0,2.0,3.0,1.0,3.0,1.0,1.0,2.0,3.0,2.0,2.0,0.0,3.0,2.0,2.0,3.0,0.0,2.0,1.0,2.0,2.0,2.0,2.0,2.0,0.0,1.0,0.0,3.0,3.0,0.0,1.0,1.0,2.0,1.0,2.0,1.0,2.0,2.0,1.0,3.0,0.0,0.0,3.0,2.0,3.0,2.0,2.0,2.0,2.0,3.0,2.0,2.0,3.0,1.0,2.0,0.0,2.0,2.0,2.0,1.0,1.0,2.0,3.0,2.0,3.0,2.0,3.0,3.0,3.0,2.0,1.0,2.0,0.0,1.0,3.0,1.0,1.0,2.0,3.0,2.0,0.0,2.0,2.0,1.0,0.0,2.0,0.0,0.0,0.0,3.0,1.0,2.0,0.0,1.0,1.0,1.0,2.0,0.0,2.0,3.0,1.0,0.0,3.0,0.0,1.0,2.0,0.0,2.0,0.0,1.0,1.0,2.0,3.0,1.0,1.0,1.0,1.0,0.0,0.0,3.0,3.0,0.0,1.0,0.0,1.0,3.0,2.0,2.0,3.0,3.0,1.0,2.0,1.0,2.0,1.0,3.0,0.0,3.0,3.0,1.0,0.0,3.0,3.0,2.0,1.0,1.0,0.0,0.0,1.0,3.0,0.0,0.0,3.0,1.0,2.0,3.0,2.0,3.0,2.0,0.0,2.0,3.0,2.0,0.0,3.0,0.0,1.0,3.0,1.0,1.0,0.0,3.0,0.0,0.0,3.0,1.0,1.0,2.0,3.0,3.0,2.0],"xaxis":"x","yaxis":"y","type":"histogram"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"LONELY_dejong_emotional_loneliness_sub_scale_score (cohort 2022)"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"count"}},"legend":{"tracegroupgap":0},"margin":{"t":60},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('f138069c-c106-4eef-ab7c-f4ab367dad9e');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>
