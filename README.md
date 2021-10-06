# CASE STUDY: Bellabeat Fitness Data Analysis - EDITING...

##### Author: Emily Liang

##### Date: October 5, 2021

#

_The case study follows the six step data analysis process:_

### ❓ [Ask](#1-ask)
### 💻 [Prepare](#2-prepare)
### 🛠 [Process](#3-process)
### 📊 [Analyze](#4-analyze)
### 📋 [Share](#5-share)
### 🚲 [Act](#6-act)

## Scenario
Since it was founded in 2013, Bellabeat has grown rapidly and quickly positioned itself as a tech-driven wellness company for women. The company has 5 focus products: bellabeat app, leaf, time, spring and bellabeat membership. Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market. Our team have been asked to analyze smart device data to gain insight into how consumers are using their smart devices. The insights we discover will then help guide marketing strategy for the company. 

## 1. Ask
💡 **BUSINESS TASK: Analyze Fitbit data to gain insight and help guide marketing strategy for Bellabeat to grow as a global player. **

Primary stakeholders: Urška Sršen and Sando Mur, executive team members.

Secondary stakeholders: Bellabeat marketing analytics team.

## 2. Prepare 
Data Source: 30 participants FitBit Fitness Tracker Data from Mobius: https://www.kaggle.com/arashnic/fitbit

The dataset has 18 CSV. The data also follow a ROCCC approach:

- Reliability: The data is from 30 FitBit users who consented to the submission of personal tracker data and generated by from a distributed survey via Amazon Mechanical Turk. 
- Original: The data is from 30 FitBit users who consented to the submission of personal tracker data via Amazon  Mechanical Turk.
- Comprehensive: Data minute-level output for physical activity, heart rate, and sleep monitoring. While the data tracks many factors in the user activity and sleep, but the sample size is small and most data is recorded during certain days of the week. 
- Current: Data is from March 2016 to May 2016. Data is not current so the users habit may be different now. 
- Cited: Unknown. 

⛔ The dataset has limitations:

- Only 30 user data is available. The central limit theorem general rule of n≥30 applies and we can use the t test for statstic reference. However, a larger sample size is preferred for the analysis.
- Upon further investigation with ```n_distinct()``` to check for unique user Id, the set has 33 user data from daily activity, 24 from sleep and only 8 from weight. There are 3 extra users and some users did not record their data for tracking daily activity and sleep. 
- For the 8 user data for weight, 5 users manually entered their weight and 3 recorded via a connected wifi device (eg: wifi scale).
- Most data is recorded from Tuesday to Thursday, which may not be comprehensive enough to form an accurate analysis. 

## 3. Process

Examine the data, check for NA, and remove duplicates for three main tables: daily_activity, sleep_day and weight:
```
dim(sleep_day)
sum(is.na(sleep_day))
sum(duplicated(sleep_day))
sleep_day <- sleep_day[!duplicated(sleep_day), ]
```
Check to see if we have 30 users. If there is a discrepency such as in the weight table, check to see how the data are recorded. The way the user record the data may give you insight on why there is missing data. 
```
n_distinct(weight$Id)

weight %>% 
  filter(IsManualReport == "True") %>% 
  group_by(Id) %>% 
  summarise("Manual Weight Report"=n()) %>%
  distinct()
 ```
Additional insight to be awared of is how often user record their data. We can see from the ```ggplot()``` bar graph that the data are greatest from Tuesday to Thursday. We need to investigate the data recording distribution. Monday and Friday are both weekdays, why isn't the data recordings as much as the other weekdays? 
```
ggplot(data=merged_data, aes(x=Weekday))+
  geom_bar(fill="steelblue")
```
![image](https://user-images.githubusercontent.com/62857660/136111719-f210057f-9ebc-45e9-a16f-e8349e30becc.png)

⛔ From weekday's total asleep minutes, we can see the graph look very similiar to the graph above. We can confirmed that most sleep data is also recorded during Tuesday to Thursday. This raised a question "how comprehensive are the data to form an accurate analysis?"

![image](https://user-images.githubusercontent.com/62857660/136238301-d63dda57-028c-41f4-b7f4-c02db46c512f.png)

Merge the three tables:
```
merged_data <- merge(merged_activity_sleep, weight, by = c("Id"), all=TRUE)
```
Convert ActivityDate into date format and add a column for day of the week:
```
daily_activity <- daily_activity %>% mutate( Weekday = weekdays(as.Date(ActivityDate, "%m/%d/%Y")))
```
Optional: convert sleep time to hours:
```
sleep_day$TotalMinutesAsleep <- sleep_day$TotalMinutesAsleep/60
sleep_day$TotalTimeInBed <- sleep_day$TotalTimeInBed/60
```

Clean the data to prepare for analysis in 4. Analyze!

## 4. Analyze

**SUMMARY:** Check min, max, mean, median and any outliers. 
```
merged_data %>%
  dplyr::select(Weekday,
         TotalSteps,
         TotalDistance,
         VeryActiveMinutes,
         FairlyActiveMinutes,
         LightlyActiveMinutes,
         SedentaryMinutes,
         Calories,
         TotalMinutesAsleep,
         TotalTimeInBed,
         WeightPounds,
         BMI
         ) %>%
  summary()
```
![summary](https://user-images.githubusercontent.com/62857660/136262678-18377ce4-3443-48a4-b108-eba6a273f963.PNG)

**ACTIVE MINUTES:** Percentage of active minutes in the four categories: very active, fairly active, lightly active and sedentary. From the pie chart, we can see that most users spent 81.3% of their daily activity in sedentary minutes and only 1.74% in very active minutes. 
```
percentage <- data.frame(
  level=c("Sedentary", "Lightly", "Fairly", "Very Active"),
  minutes=c(sedentary_percentage,lightly_percentage,fairly_percentage,active_percentage)
)

plot_ly(percentage, labels = ~level, values = ~minutes, type = 'pie',textposition = 'outside',textinfo = 'label+percent') %>%
  layout(title = 'Activity Level Minutes',
         xaxis = list(showgrid = FALSE, zeroline = FALSE, showticklabels = FALSE),
         yaxis = list(showgrid = FALSE, zeroline = FALSE, showticklabels = FALSE))
```

![newplot](https://user-images.githubusercontent.com/62857660/136252582-96e1f52a-dfe0-4247-a882-82d179d9b2b9.png)


The American Heart Association and World Health Organization recommend at least 150 minutes of moderate-intensity activity or 75 minutes of vigorous activity, or a combination of both, each week. That means it needs an daily goal of 21.4 minutes of FairlyActiveMinutes or 10.7 minutes of VeryActiveMinutes.

In our dataset, 30 users met fairly active minutes or very active minutes.
```
active_users <- daily_activity %>%
  filter(FairlyActiveMinutes >= 21.4 | VeryActiveMinutes>=10.7) %>% 
  group_by(Id) %>% 
  count(Id) 
  
active_users #30 
```


**TOTAL STEPS:** let's look at how active the users are per hourly in total steps. From 5PM to 7PM the users take the most steps. 
```
ggplot(data=hourly_step, aes(x=Hour, y=StepTotal, fill=Hour))+
  geom_bar(stat="identity")+
  labs(title="Hourly Steps")
```
![image](https://user-images.githubusercontent.com/62857660/136235391-bb22c15d-93aa-494d-bce2-76a984274fb7.png)


How active the users are weekly in total steps. Tuesday and Saturdays the users take the most steps. 
```
ggplot(data=merged_data, aes(x=Weekday, y=TotalSteps, fill=Weekday))+ 
  geom_bar(stat="identity")+
  ylab("Total Steps")
```
![image](https://user-images.githubusercontent.com/62857660/136252217-53d355de-2c25-4185-8e6d-27ba087573ae.png)




**INTERESTING FINDS:** The more active that you're, the more steps you take, and the more calories you will burn. This is an obvious fact, but we can still look into the data to find any interesting. Here we see that some users who are sedentary, take minimal steps, but still able to burn over 1500 to 2500 calories compare to users who are more active, take more steps, but still burn similar calories.

```
ggplot(data=daily_activity, aes(x=TotalSteps, y = Calories, color=SedentaryMinutes))+ 
  geom_point()+ 
  stat_smooth(method=lm)+
  scale_color_gradient(low="steelblue", high="orange")

```
![image](https://user-images.githubusercontent.com/62857660/136260311-a379b303-76ac-426c-9c30-ea2695569632.png)

Comparing the four active levels to the total steps, we see most data is concentrated on users who take about 5000 to 15000 steps a day. These users spent an average between 8 to 13 hours in sedentary, 5 hours in lightly active, and 1 to 2 hour for fairly and very active. 

![image](https://user-images.githubusercontent.com/62857660/136269396-7019cf93-6e0c-4216-9944-5e58e017f593.png)

According to [this healthline.com article](https://www.healthline.com/nutrition/how-many-calories-per-day#average-calorie-needs), moderately active woman between the ages of 26–50 needs to eat about 2,000 calories per day and moderately active man between the ages of 26–45 needs 2,600 calories per day to maintain his weight. Comparing the four active levels to the calories, we see most data is concentrated on users who burn 2000 to 3000 calories a day. These users also spent an average between 8 to 13 hours in sedentary, 5 hours in lightly active, and 1 to 2 hour for fairly and very active. Additionally, we see that the sedentary line is leveling off toward the end while fairly + very active line is curing back up. This indicate that the users who burn more calories spend less time in sedentary, more time in fairly + active. 

![image](https://user-images.githubusercontent.com/62857660/136263632-ac5c1958-23db-4374-b810-df6f322b047b.png)



**REGRESSION ANALYSIS**: We can use regression analysis look at the R-squared, slope, p-value and intercept for different combinations of the variables. For R-squared, 0% indicates that the model explains none of the variability of the response data around its mean. Higher % indicates that the model explains more of the variability of the response data around its mean. Postive slope means variables increase/decrease with each other, and negative means one variable go up and the other go down. 

Postive slope with 27.88% R-squared:
```
calories_vs_steps.mod <- lm(Calories ~ TotalSteps, data = merged_data)
summary(calories_vs_steps.mod)
```
![calvssteps](https://user-images.githubusercontent.com/62857660/136107751-e1aba885-9475-4430-94cc-e56e39d0c380.PNG)




**SLEEP**: According to article: [Fitbit Sleep Study](https://blog.fitbit.com/sleep-study/#:~:text=The%20average%20Fitbit%20user%20is,is%20spent%20restless%20or%20awake.&text=People%20who%20sleep%205%20hours,the%20beginning%20of%20the%20night.), 55 minutes are spent awake in bed before going to sleep. We have 13 users in our dataset spend 55 minutes awake before alseep. 

```
awake_in_bed <- mutate(sleep_day, AwakeTime = TotalTimeInBed - TotalMinutesAsleep)
awake_in_bed <- awake_in_bed %>% 
  filter(AwakeTime >= 55) %>% 
  group_by(Id) %>% 
  arrange(AwakeTime) 
```

Next, we want to look at if users who spend more time in sedentary minutes spend more time sleeping as well. We can use regression analysis ```lm()``` to check for R-squared and the slope. We can see that it has a negative slope with an very small R-squared. How many minutes an user asleep have an very weak correlation with their sedentary minutes.  
```
sedentary_vs_sleep.mod <- lm(SedentaryMinutes ~ TotalMinutesAsleep, data = merged_data)
summary(sedentary_vs_sleep.mod)
```
![calvssteps2](https://user-images.githubusercontent.com/62857660/136107919-65c86392-4f12-4038-b3d3-09166d8d5381.PNG)




Editing...


## 5. Share 
Editing...

## 6. Act
Editing...









