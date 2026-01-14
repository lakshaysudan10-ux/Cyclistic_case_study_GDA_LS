# Cyclistic_case_study_GDA_LS
Analysis of the Cyclistic case study 


The case study chosen is “Cyclistic,” a bike-share program which features 5824 bikes and 692 docking stations across Chicago. They offer reclining bikes, hand tricycles and cargo bikes, making it inclusive for people who are unable to use a standard two-wheel bike. Customers who purchase a single-ride or full-day pass are referred to as casual riders. Customers who purchased annual memberships are ‘Cyclistic’ members. The majority of riders opt for traditional bikes, and about 8% of riders use assistive options. Cyclistic users are more likely to ride for leisure, but about 30% use bikes to commute to work every day.
Cyclistic’s finance analysts have concluded that annual members are much more profitable than casual riders. Although the pricing flexibility helps Cyclistic attract more customers, Moreno believes that maximizing the number of annual members will be key to future growth. Rather than creating a marketing campaign that targets all-new customers, Moreno believes there is a solid opportunity to convert casual riders into members. She notes that casual riders are already aware of the Cyclistic program and have chosen Cyclistic for their mobility needs.

# ASK

Three questions will guide the future marketing program:
1. How do annual members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become members?

# Identifying the stakeholders

Lily Moreno
The director of marketing and my manager are responsible for the development of campaigns and initiatives to promote the bike-share program. 

Cyclistic Marketing Team:
A team of analysts who are responsible for collecting, analyzing, and reporting data that helps guide Cyclistic's marketing strategy. I am a part of this team. I joined 6 months ago and have spent time learning about Cyclistic’s mission and business goals. 

Cyclistic Executive Team:
The notoriously detail-oriented team that will decide whether the recommendation for the program will be approved or not. 

Team Goal:
Design marketing strategies aimed at converting casual riders into annual members based on the trends of bike usage between annual members and the casual riders of the 'Cyclistic' bike sharing app, and make recommendations accordingly.

They are the stakeholders of this project, as part of my job is answering one of the three questions. The marketing team is relying on my analysis to gain insights into customer patterns and preferences, which will help answer the other two questions and provide an appropriate marketing plan for Lily to put into place and execute.

# Deliverable 
1. To identify the business task.
2. A description of all data sources used.
3. Documentation of any cleaning or manipulation of data.
4. A summary of your analysis.
5. Supporting visualizations and key findings.
6. Your top three recommendations based on your analysis. 


# PREPARE 

# Data source 
  https://divvy-tripdata.s3.amazonaws.com/index.html
   is provided by Divvy/Lyft alongside the 'Chicago Department of Transportation' under this license 
   https://ride.divvybikes.com/data-license-agreement
  Data is stored in .zip and .csv formats. Subsequently, the ride data is anonymized. 
# ROCCC 
  Reliable - Data is accurate and shows no sign of bias; it does, however, have missing values, which will have to be deleted during the cleaning process. 
  Original - Data is first-party collected data from the 'Divvy' Database licensed by Lyft andthe  Chicago Dept of Transportation.
  Comprehensive - Data is moderately comprehensive because privacy issues prevent us from using the rider's personal identification number to learn if the rider has bought multiple single   passes or if they live in the Cyclistic service area. 
  Current - Data is updated regularly on the 'Divvy' website, monthly and yearly. 
  Cited - Data is provided by Divvy and owned by the City of Chicago. 
# Data Integrity 
Data integrity will be verified by performing initial data cleaning processes such as data inspection, duplicate removal, filling in missing and null data, ensuring data type consistency, fixing and handling structural errors, and removing invalid values. The data requires merging into one dataset; therefore, the data temporarily has different column names and membership types.
 
The table contains useful information about the dates, times, durations, locations, start and stop stations for each ride, along with the type of membership held by the people who used Cyclistic. This can help measure insightful statistics, which in turn later will help us determine the type of bike usage by each type of rider the marketing team will be able to propose strategies based on our findings. 

# PROCESS  
I am using R to process, clean and analyze the data, and Tableau for visualization. 

```{R}
#loading packages 
library(tidyverse)
library(conflicted)

#setting dplyr::filter and dplyr::lag as the dafault choices

conflict_prefer("filter", "dplyr")
conflict_prefer("lag", "dplyr")

## Collecting Data from the two .csv files uploaded from the data source, then combining them into one single file.  

q1_2019 <- read_csv("Divvy_Trips_2019_Q1.csv")
q1_2020 <- read_csv("Divvy_Trips_2020_Q1.csv")

#compare Column names for each dataset and then, combine the two based on the 2020 Q1 
colnames(q1_2019)
colnames(q1_2020)

(q1_2019 <- rename(q1_2019
                   ,ride_id = trip_id
                   ,rideable_type = bikeid
                   ,started_at = start_time
                   ,ended_at = end_time
                   ,start_station_name = from_station_name
                   ,start_station_id = from_station_id
                   ,end_station_name = to_station_name
                   ,end_station_id = to_station_id
                   ,member_casual = usertype
                   ))
#inspect and convert ride_id and rideable_type character to be able to stack correctly
str(q1_2019)
str(q1_2020)

q1_2019 <-  mutate(q1_2019, ride_id = as.character(ride_id)
                   ,rideable_type = as.character(rideable_type))

#removed 2019 columns that were not required as part of the dataset merged

all_trips <- all_trips %>%  
  select(-c(start_lat, start_lng, end_lat, end_lng, birthyear, gender,  "tripduration"))

#inspect the new dataset

colnames(all_trips)
nrow(all_trips
dim(all_trips)
head(all_trips)
str(all_trips)
summary(all_trips)

#fixing 4 values of riders to only two for the type of members.

table(all_trips$member_casual)

all_trips <- all_trips %>%
  mutate(member_casual = recode(member_casual
                 ,"Subscriber" = "member"
                 ,"Customer" = "casual"))  
table(all_trips_member_casual)

#adding new columns for each day, month, year, and day of the week.
all_trips$date <- as.Date(all_trips$started_at)

all_trips$month <- format(as.Date(all_trips$date), "%m")

all_trips$day <- format(as.Date(all_trips$date), "%d")

all_trips$year <- format(as.Date(all_trips$date), "%Y")

all_trips$day_of_week <- format(as.Date(all_trips$date), "%A")

#added a ride_length column
all_trips$ride_length <- difftime(all_trips$ended_at,all_trips$started_at)
str(all_trips)
#removing the negative trip lengths that were caused by bikes taken off the stations for quality assurance.
#also created a second new dataset to keep the original rows and columns untouched.

all_trips_v2 <- all_trips[!(all_trips$start_station_name == "HQ QR" | all_trips$ride_length<0),]

#Analysis on column ride_length
summary(all_trips_v2$ride_length)
## Aggregate your data so it’s useful and accessible.
#comparing members and casual users 
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = mean)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = median)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = max)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = min)

# calculating average ride time by day of the week, members vs casuals

aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)

#correcting the order for the week to start on Sunday
all_trips_v2$day_of_week <- ordered(all_trips_v2$day_of_week, levels=c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))

## visualizing number_of_rides

all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge")

#Visualization for average duration

all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = average_duration, fill = member_casual)) +
  geom_col(position = "dodge")


#creating a counts table with aggregate data for the number of rides and average duration

counts <- aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)

#exporting summary file onto the desktop for further visualization
write.csv(counts, file = 'avg_ride_length.csv')

downloading the above file from R onto the desktop, then uploading
```
 
# Deliverable 
Documentation of any cleaning or manipulation of data

# ANALYZE 
Aggregated data 
member_casual  weekday   number_of_rides  average_duration
   <chr>         <ord>             <int> <drtn>          
 1 casual        Sun               18,652 5061.3044 secs  
 2 casual        Mon                5,591 4752.0504 secs  
 3 casual        Tue                7,311 4561.8039 secs  
 4 casual        Wed                7,690 4480.3724 secs  
 5 casual        Thu                7,147 8451.6669 secs  
 6 casual        Fri                8,013 6090.7373 secs  
 7 casual        Sat               13,473 4950.7708 secs  
 8 member        Sun               60,197  972.9383 secs  
 9 member        Mon              110,430  822.3112 secs  
10 member        Tue              127,974  769.4416 secs  
11 member        Wed              121,902  711.9838 secs  
12 member        Thu              125,228  707.2093 secs  
13 member        Fri              115,168  796.7338 secs  
14 member        Sat               59,413  974.0730 secs 

# Trends or relationships in the data 
The relationship between riders, casual and members distinguished by the day of the week is partially due to members using Cyclistic for commuting to work and day-to-day tasks during the week; the number of rides is significantly higher for members. While casual members were more frequent on weekends and had 4 times as long rides compared to a member on any day of the week, this implies casual customers are taking long leisure rides and sightseeing in Chicago during the week and especially on the weekends. These insights help to understand the user relationship with Cyclistic. The difference in usage will be key in understanding how the marketing team can target the casual riders to promote yearly memberships. 


# Deliverable 
A summary of your analysis -

The Summary concludes that there is a high demand for Cyclistic bikes from both Members and Casual riders, and the number of trips is very high throughout the week for annual members, the number of trips is 1/10th for casual riders, and they double in average duration, implying long rides for leisure and sightseeing through Chicago. On the weekend, however, the number of casual rides increases three times its weekly average and maintains a high average duration on Saturdays and Sundays. The annual members' rides decrease to about half their weekly rides, and a slight surge in average duration on Saturdays and Sundays. There is potential for Casual riders to convert into annual members, seeing their average duration and usage, it is definitely worth exploring for the marketing team. 


# SHARE 

Guiding Questions: 
1. Were you able to answer the questions of how annual members and casual riders use Cyclistic bikes?
Yes, we obtained answers on how Cylcistic is used by riders who are riding on an annual membership and casual basis. 

2. What story does your data tell?
Casual riders' data displays low trip counts and very high trip durations, contrary to annual membership riders, who show high trip counts and low trip durations. Ultimately, reveals an opportunity for Cyclistic to improve and target casual riders to promote yearly memberships.

3. How do your findings relate to your original question?
The findings reveal that there is a way for casual members to be converted into members, which may be a monthly or even a weekend membership. 


4. Who is your audience? What is the best way to communicate with them? 
 My audience is the Cyclistic marketing team, Lily Moreno, and the Cyclistiv executive team. The best way to communicate with them is through a meeting which contains a presentation as well as visualizations about my findings. 

5. Can data visualization help you share your findings?
Yes Data visualization will help draw focus to my part of the analysis through a clear and concise visualization, which will help to make data understandable and deliver a clear message of my recommendations.

6. Is your presentation accessible to your audience? 


Key Tasks:
Determine the best way to share your findings.
Create effective data visualizations. 
Present your findings. 
Ensure your work is accessible. 

Deliverables
Supporting visualizations and key findings 

# ACT 

Guiding Questions: 
What is your conclusion based on your analysis? 
How could your team and business apply your insights?
What next steps would you or your stakeholder take based onyour findings? 
Is there additional data you could use to support your findings? 

KEY TASKS 
Create a portfolio. 
Add your case study. 
Practice presenting your case study to a family member or friend. 

Deliverables 
Your top three recommendations based on your analysis


Recommendations 

1. Cyclistic could introduce weekend passes/memberships eligible starting Friday evening and Valid until Sunday night/Monday morning, for casual riders who are visiting the city for a short period of time. It can be a good incentive to purchase the ‘weekend memberships.’It would include 15-20 trips and guided navigation map tours for the city’s favourite tourist spots.  

2. Cyclistic marketing team could pursue Casual riders with incentives to buy monthly memberships that are based on usage, such as distance travelled and time on the bike. Cyclistic could offer different tiers of membership, such as ‘Silver’, ‘Gold’, and ‘Platinum.’ 
Usage and time on the bike categorized: 
- 50 trips a month (Silver) 
- 75 trips a month (Gold) 
- 90+ trips a month (Platinum) 
