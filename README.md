# Data-Analytics-Case-Study
For this case study I analyzed data from the smart fitness device company, Bellabeat, out of San Francisco. I was able to produce action based solutions for the business after cleaning, organizing, and analyzing their data. 

## ASK
<ol> 
<li> What are some trends in smart device usage? </li>
<li> ﻿﻿﻿How could these trends apply to Bellabeat customers? </li>
<li> ﻿﻿﻿How could these trends help influence Bellabeat marketing strategy? </li>
</ol>

## PREPARE
The data is internal and only covers two months in 2016, from March 12 to May 12. From the data collected I used  dailyActivity_merged and sleepDay_Merged.

### Tools used & why:
<ul>
<li>__Google sheets__ - View and clean data</li>
<li>__My Sql__ - clean the larger data sets, create tables</li>
<li>__Tableau__ - Aggregate data for visualization</li>
</ul>

## PROCESS
To begin, I uploaded both dailyActivity_merged datasets into Google sheets, froze the first row to see all column names, then organized the sheet by Id, and lastly deleted all lines with 0 for every column. 
Next came formatting all dates in the date column to the standard ‘Year-Month-Day’ format.
Since the datasets were very large uploading them into BigQuery was the best option for cleaning and organining.

### In BigQuery
* Rounded decimals to 2 Decimal Places and sorted by ID then Date

```sql
SELECT  
  Id,
  ActivityDate,
  TotalSteps,
  ROUND(TotalDistance, 2) AS round_TotalDistance,
  ROUND (TrackerDistance, 2) AS round_TrackerDistance,
  ROUND(LoggedActivitiesDistance, 2) AS round_LoggedActiveDistance,
  ROUND(VeryActiveDistance) AS round_VeryActiveDistance,
  ROUND(ModeratelyActiveDistance, 2) AS round_ModeratelyActiveDistance,
  ROUND(LightActiveDistance, 2) AS round_LightActiveDistance,
  ROUND(SedentaryActiveDistance, 2) AS round_SedentaryActiveDistance,
  VeryActiveMinutes,
  FairlyActiveMinutes,
  LightlyActiveMinutes,
  SedentaryMinutes,
  Calories
FROM `capstone-436620.bellaBeat_March_April.dailyActivity1`
```

* I deleted all row with 0 calories as the meant all items in row were 0
```sql
	DELETE
	FROM `capstone-436620.bellaBeat_March_April.dailyActivity1`
	WHERE Calories < 1
```
* Saved as new Table: sorted_dailyActivity1 and  sorted_dailyActivity2
* Followed similar steps in sleepLog
* Removed All times from date (all times were set to 12:00 AM), then named DATE, Sorted By ID:
  
 ```sql
SELECT
Id,
SUBSTRING(date, 1, length(date)-11) AS Date,
TotalSleepRecords,
TotalMinutesAsleep,
TotalTimeInBed
FROM capstone-436620.bellaBeat_April_May.sleepDay2
ORDER BY
  Id
--Saved new Table
```
I followed similar steps for dailyIntensities, weightLogInfo, and dailyCalories sheets
  Deleted rows where all fields were 0

* Deleted all rows where SedentaryMinutes is 1440, because that is the entire day: 

```sql
DELETE
FROM `capstone-436620.bellaBeat_April_May.dailyIntensities2`
WHERE
  SedentaryMinutes = 1440;
```

* Rounded decimals to 2 places and sorted by id then date: 
```sql
SELECT
  Id,
  ActivityDay,
  SedentaryMinutes,
  LightlyActiveMinutes,
  FairlyActiveMinutes,
  VeryActiveMinutes,
  ROUND(SedentaryActiveDistance, 2) AS round_SedentaryActiveDistance,
  ROUND(LightActiveDistance, 2) AS round_LightActiveDistance,
  ROUND(ModeratelyActiveDistance, 2) AS round_ModeratelyActiveDistance,
  ROUND(VeryActiveDistance, 2) AS round_VeryActiveDistance
FROM `capstone-436620.bellaBeat_April_May.dailyIntensities2`
ORDER BY
  id,
  ActivityDay
--Created new table
```

 After rechecking that all the data was cleaned and sorted I went through and made sure all data homogenized in column names and formatting. A few of the sheets had a combined Date and time column that Isplit into 2 columns

```sql
SELECT  
  Id,
  split(Date, ' ')[offset(0)] as ActivityDate,
  split(Date, ' ')[offset(1)] as Time,
  ROUND(WeightKg, 2) AS round_WeightKg,
  ROUND(WeightPounds, 2) AS round_WeightPounds,
  ROUND(BMI, 2) AS round_BMI,
  IsManualReport,
  LogId
FROM `capstone-436620.bellaBeat_March_April.weightLogInfo1`
ORDER BY
 Id,
 ActivityDate,
 LogId;
```

## Data Validation checks
<ul>
  <li>Made sure column names where standard and changed those that werent</li>
  <li>Checked to make sure the length for Ids and logs where standard across all sheets</li>
  <li>Made sure the information added up to the total columns</li>
  <li>Sorted all by id then date </li>
  <li>Named clean_dataname</li>
 </ul>

## ANALYZE AND SHARE

* Sort users into groups by activity levels
```sql
(SELECT
 Id,
 COUNT(Id) AS Logged_Use,
CASE
 WHEN COUNT (Id) BETWEEN 45 AND 63 THEN 'Active User'
 WHEN COUNT (Id) BETWEEN 35 AND 44 THEN 'Moderate User'
 WHEN COUNT (Id) BETWEEN 0 AND 34 THEN 'Light User'
END as Usage_Type
FROM `capstone-436620.bellaBeat__AllCleanData.totals_ById`
GROUP BY
 Id
ORDER BY
 Logged_Use);
--Created new table: users_Category
```
### Finindings
 <ul>
   <li>9 Light Users </li>
  <li>20 Moderate Users </li>
  <li>6 Active Users</li>
 </ul>


* I then Downloaded the cleaned data into Google Sheets 
* I combined sleep logs and daily activity sheets into one. 

** In google Sheets I separated daily activity into 3 sheet: 
All daily activity for both months
The total of all column for each user
The Monthly  totals of all columns for each user

In the usersCategorized sheet I created a chart showing number of user in each category
(hover to see the title text):
![UserChart](https://github.com/user-attachments/assets/93929bf6-8224-4bfc-ae2f-30f96b10236c)

* I then  uploaded  totalsById, monthlyById, UsersCatorgorized, and sleepLogged sheets into tableau.

One of the first things I noticed is that not all users logged sleep info. Of the users that logged sleep, none of them consistently  logged their sleep. Due to this lack of information, the data does not give an accurate representation of how sleep affects and activity level. From the data that was available, moderate user logged significantly more minutes of sleep than both Light and Active. As shown here (This graph does not account for the fact the majority of users are moderate):

![Sleep Chart](https://github.com/user-attachments/assets/29999501-25b0-46ca-8692-f7f5f12c001c)

### CALORIES, STEPS, AND ACTIVITY LEVEL
To understand how Lightly, Moderate, and Active Users use their devices I Start by looking at Calories Burned. This graph showed that while Lightly Active users have the lowest amount of calories burned and A Very Active User has the highest amount of calories burned, every type of user is somewhat evenly portrayed. 

![Calorie 1](https://github.com/user-attachments/assets/2f1eb5cb-a6dc-4395-83c6-5c614b11c2b1)

The calories burned compared each groups to itself to see if there was any significant change between the first and second months. (see calories_cycle_Usertype tab in linked Tableau Viz)

I then compared the number of steps for the users to the amount of calories burned. The difference between the user type was easier to see when comparing both their steps and their calories burned. 
![2](https://github.com/user-attachments/assets/4c8d3709-819a-4b7f-8ffb-7d2cf1834f26)


As expected when mapping the total number of steps for all users,The Majority Very Active User were higher on the chart. 

![4](https://github.com/user-attachments/assets/1472ae77-14eb-4e6b-83ee-659f9cfc7f44)

Because Steps,Calories, and sleep alone does not determine what makes a user Light, Moderate, or Very Active 
I went Back to big Query to get the Average of all three.
```sql
SELECT
 Id,
 avg(VeryActiveMinutes) AS very_Minutes,
 avg(FairlyActiveMinutes) AS fairly_Minutes,
 avg(LightlyActiveMinutes) AS lightly_Minutes,
 avg(SedentaryMinutes) AS sedentary_Minutes,


FROM `capstone-436620.bellaBeat__AllCleanData.daily_Acticity`

And then rounded those to 2 decimal places.
SELECT 
 Id,
 round(Very_Minutes, 2) AS very_ActiveMinutes,
 round(Fairly_Minutes, 2) AS fairly_ActiveMinutes,
 round(Lightly_Minutes, 2) AS lightly_ActiveMinutes,
 round(Sedentary_Minutes, 2) AS fairly_ActiveMinutes,
FROM `capstone-436620.bellaBeat__AllCleanData.average minutes`

-created new table
```

This shows that the majority of minutes for all user is sedentary minutes. 

Then, Removed Sedentary Minutes to see how many combined minuted each group spends being active. I found that the Moderate users are the most active followed by Light User with Active users coming in last. 

![5](https://github.com/user-attachments/assets/ed55e64c-f1a6-460a-8f19-10f4cf0ef708)


## SHARE
[This can be viewed in my Tableau board BellaBeat](https://public.tableau.com/views/BellabeatCaseStudy_17281487985690/Average_ActivityMinutes?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)


## ACT
<ol>
<li>Bellabeat would benefit from continuous syncing of data between the wearable devices and the app. With continuous syncing there would be little to no  gaps in the data. Therefore the health and wellness of users would be more accurate. 
</li>
<li>Instead of relying on users to log the minutes of their own activity and sleep, Bellabeat should instead log the heart rate, and distance for a measure of time, for example when there is a spike in heart rate. Then allow users to log what activity they were doing, by choosing from a list of possible activities. </li>
</ol>




```

