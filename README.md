# Cyclistic_Tripdata
A capstone project for the [Google Data Analytics certificate](https://www.coursera.org/professional-certificates/google-data-analytics). This Certificate is part of 8 mandatory courses, which included this project.

## Scenario

In this project, I am acting as a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company based in Chicago. The company has a bike-share program that provides more than 5,800 bicycles with over 600 docking stations. The company offers different types of bikes from reclining bikes to cargo bikes. Cyclistic bikes can be unlocked and acquired at one dock station, then returned to a different station in the system anytime. To unlock the bikes, customers would have to have a single-ride pass, full-day pass, and annual memberships. The customers who purchase annual memberships are Cyclistic members, while customers who purchase single-ride and full-day passes are casual riders.

Cyclistic's finance analysts determined that annual members are much more profitable than casual riders. The marketing strategy of Cyclistic relied on building general awareness of the brand and appealing to the broad consumer segments. Cyclistic is looking for ways to grow and the director of marketing, Lily Moreno, believes the company’s future rests on the company maximizing the number of annual memberships. Our team wants to understand how casual riders and annual members use Cyclistic bikes differently. From here our team will design a new marketing strategy to convert casual riders into annual members for Cyclistic to succeed.


Rather than creating a marketing campaign that targets all-new customers, Moreno believes there is a very good chance to convert casual riders into members. She notes that casual riders are already aware of the Cyclistic program and have chosen Cyclistic for their mobility needs. The data analysis team has the mission of working with past data and finding insights that can be used to convert casual riders to annual members.

My assignment is to answer the following question: How do annual members and casual riders use Cyclistic bikes differently?

## Deliverables

Our analysis must follow the list below to be considered successful.

1) A clear statement of the business task
2) A description of all data sources used
3) Documentation of any cleaning or manipulation of data
4) A summary of the analysis
5) Supporting visualizations and key findings
6) Your top three recommendations based on your analysis

To succeed in the deliverables, we are going to follow the data analysis process: ask, prepare, process, analyze, share and act.

## Ask Phase

Our task is to identify the differences between members and casual riders over the past 12 months.

As we analyze the difference between both members and casual riders, we should give at least three recommendations that can help the company in achieving its goals.

Our main stakeholders are:

1) Cyclistic executive team. They are very detailed-oriented and they will decide to approve our recommended marketing strategy.
2) Lily Moreno, our manager and the director of marketing. She designed this goal of maximizing on annual members and assigned us this specific task.

## Prepare Phase

The data used in this project was downloaded from the [Divvy Trip Data](https://divvy-tripdata.s3.amazonaws.com/index.html). The data we used were the last 12 months from Cyclistic historical trip data, which is stored in .zip files.

The data is current and has been updated monthly for the past few years. Since data cleaning is needed, since it is collected by our system the data seems reliable. This reliability gives our team certain confidence that the data has everything that it should have. The data is relevant but I do have some concerns about our task. Since we have to analyze the data over the past 12 months, I'm afraid we might miss out on some key trends that could be available with more data over a longer timeframe.

With that said, I do believe we can still utilize the data from the past 12 months to come up with a useful analysis for the company.

## Process Phase

The data, which includes the 12 months of September 2020 through August 2021 is downloaded. The .zip files convert to CSV text files, and we import these files to tables housed in an Azure Data Studio database, which runs Microsoft SQL Server (MSSQL). The SQL tables are then cleaned and copied to a larger table.

### The following steps are repeated for each monthly table.

#### Add Ride Length Column

It would help to know the average time casual riders and members use Cyclistic bikes on a per weekly and monthly basis. This may give us some insights we need to identify key trends. Therefore, we need to add a column to all the tables called "ride_length_sec." 


```
ALTER TABLE dbo.tripdata202009
    ADD ride_length [int] NULL;
```


Once the field has been added to our table, we run the following SQL statement to populate the ride_length_sec field:


```
UPDATE dbo.tripdata202009
    SET ride_length = DATEDIFF(SS, started_at, ended_at);
```


This statement calculates the number of seconds between the start and end times of a particular trip.

#### Add Day Of The Week Column

We need to add a column called "day_of_week" and add this column to each table.


```
ALTER TABLE dbo.tripdata202009
    ADD day_of_week NVARCHAR(10) NULL;
```


Once the column has been added, we run the following SQL statement to populate the day_of_week:


```
UPDATE dbo.tripdata202009
    SET day_of_week = DATENAME(WEEKDAY, started_at);
```


#### Start Time Is Greater Than The End Time

There were a few instances where the start time occured after the end time. Therefore, we need to remove the recorded bike trips where this occurs. 


```
DELETE FROM dbo.tripdata202009
    WHERE started_at > ended_at;
```


#### Ride Lengths Less Than A Minute

Some ride lengths were less than a minute due to the possibility of false starts or users trying to re-dock a bike to ensure it was secure. So this needs to be removed as well.


```
DELETE FROM dbo.tripdata202009
    WHERE ride_length < 60;
```


#### Remove Test Fields

Certain trips in the tables indicated the station ID were test stations. We'll remove these test trips as they will not add to our analysis.


```
DELETE FROM dbo.tripdata202009
    WHERE start_station_id LIKE '%TEST%' OR end_station_id LIKE '%TEST%';
```


#### Trips With No Coordindates Nor Station Identifiers

Let's remove the trips that are missing any coordinates or docking station identifiers.


```
DELETE FROM dbo.tripdata202009
    WHERE start_lat IS NULL OR start_lng IS NULL OR end_lat IS NULL OR end_lng IS NULL;

DELETE FROM dbo.tripdata202009
    WHERE start_station_id IS NULL OR end_station_id IS NULL;
```


### Create Table To Combine Data

The following SQL statement will create the table that will contain all our data from the other tables for our analysis.


```
CREATE TABLE dbo.[cyclistic.tripdata](
    [ride_id] [nvarchar](50) NOT NULL,
    [rideable_type] [nvarchar](50) NOT NULL,
    [started_at] [datetime2](7) NOT NULL,
    [ended_at] [datetime2](7) NOT NULL,
    [start_station_name] [nvarchar](60) NOT NULL,
    [start_station_id] [nvarchar](50) NOT NULL,
    [end_station_name] [nvarchar](60) NOT NULL,
    [end_station_id] [nvarchar](50) NOT NULL,
    [start_lat] [float] NOT NULL,
    [start_lng] [float] NOT NULL,
    [end_lat] [float] NOT NULL,
    [end_lng] [float] NOT NULL,
    [member_casual] [nvarchar](50) NOT NULL,
    [ride_length_sec] [int] NOT NULL,
    [day_of_week_text] [nvarchar](10) NOT NULL,
    [month]  AS (DATENAME(month,[started_at])),
    CONSTRAINT PK_cyclistictripdata_rideid PRIMARY KEY (ride_id)
)
```


Now it is time to import the monthly data tables into the newly created data table. The following SQL statement will be used for each monthly table.


```
INSERT INTO dbo.[cyclistic.tripdata] (
    ride_id,
    rideable_type,
    started_at,
    ended_at,
    start_station_name,
    start_station_id,
    end_station_name,
    end_station_id,
    start_lat,
    start_lng,
    end_lat,
    end_lng,
    member_casual,
    ride_length_sec,
    day_of_week_text
)
SELECT
    ride_id,
    rideable_type,
    started_at,
    ended_at,
    start_station_name,
    start_station_id,
    end_station_name,
    end_station_id,
    start_lat,
    start_lng,
    end_lat,
    end_lng,
    member_casual,
    ride_length,
    day_of_week
FROM dbo.tripdata202009;
```


Now it is time to begin our analysis.

## Analysis

We will be using pivot tables and charts from Excel to get a summary of our findings. The results from the following SQL statements will create our data viz in Excel.

### Average Monthly Ride Length Chart

To analyze and visualize the average bike trip length per month for casual and member riders, we used the results from this SQL statement to create the visualization in Excel:


```
SELECT * FROM (
    SELECT
        [month],
        member_casual,
        ride_length_sec
    FROM dbo.[cyclistic.tripdata]
) AS t
PIVOT (
    AVG(ride_length_sec)
    FOR [month] IN (
        [September],[October],[November],
        [December],[January],[February],[March],
        [April],[May],[June],[July],[August]
    )
) AS pvt_tbl;
```



![AVG Monthly Ride Length](https://user-images.githubusercontent.com/92260713/139891709-80038894-4d97-491d-ad74-2b25f4bb7949.png)



Based on the chart, we see that members tend to remain steady in their use of our bikes. The casual rider is a lot more unpredictable in their usage and tends to take longer trips than members.



### Average Weekly Ride Length Chart


Analyzing the average bike trip every week utilizing the Information from the monthly tables, we used the results from this SQL statement to create or data viz:


```
SELECT * FROM (
    SELECT
        day_of_week_text,
        member_casual,
        ride_length_sec
    FROM dbo.[cyclistic.tripdata]
) AS t
PIVOT (
    AVG(ride_length_sec)
    FOR day_of_week_text IN (
        [Sunday],[Monday],[Tuesday],[Wednesday],
        [Thursday],[Friday],[Saturday]
    )
) AS pvt_tbl;
```


![AVG Weekly Ride Length](https://user-images.githubusercontent.com/92260713/139893611-53faa0c0-8dba-452e-b0d1-c8df3cd390ed.png)


This data viz reiterates the trend from the average monthly analysis casual riders tends to take longer trips than members. However, members are more consistent with their rides throughout the week.


### Total Monthly Ridership


The analysis of the total number of members and casual riders' bike trips for each month. The results of this SQL statement will be used to create our data viz:


```
SELECT * FROM (
    SELECT DISTINCT 
        ride_id,
        [month],
        member_casual
    FROM dbo.[cyclistic.tripdata]
) AS t
PIVOT (
    COUNT(ride_id)
    FOR [month] IN (
        [September],[October],[November],
        [December],[January],[February],[March],
        [April],[May],[June],[July],[August]
        )
) AS pvt_tbl;
```



![Total Monthly Ridership](https://user-images.githubusercontent.com/92260713/139893928-898ae5a5-db93-46ca-91bd-86f654264f1c.png)



The chart above illustrates a significant drop in bike trips during the winter months and then bike trips increase as the weather gets warmer.


### Total Weekly Ridership


The analysis of the total number of members and casual risers' bike trips for each day of the week. We will use the results of this SQL statement to create or visualization:


```
SELECT * FROM (
    SELECT DISTINCT
        ride_id,
        day_of_week_text,
        member_casual
    FROM dbo.[cyclistic.tripdata]
) AS t
PIVOT (
    COUNT(ride_id)
    FOR day_of_week_text IN (
        [Sunday],[Monday],[Tuesday],[Wednesday],
        [Thursday],[Friday],[Saturday]
    )
) AS pvt_tbl;
```


![Total Weekly Ridership](https://user-images.githubusercontent.com/92260713/139894142-a02caf38-cc4e-42b9-8b6e-69593cfe1240.png)


Once again, our members are more consistent with their utilization throughout the week. Casual riders are consistent from Monday to Thursday, Friday through Sunday is more popular, and they have longer bike trips than our members on the weekends.


## Analysis Summary

Our business task was to search for differences between casual riders and members. The most relevant differences we found are:

1) Annual members take shorter rides.
2) Members are the main users of our bikes on the weekdays. But on the weekends, the casual riders surpass members when riding our bikes.
3) The pattern of use during the weekdays differs between the two groups. On weekdays, there is a jump in usage around 8 am. However, the same behavior is not observed in casual riders or on the weekends.


## Recommendations

1) Based on our analysis, the average length of a single trip per week and month, casual riders tend to take longer bike trips than members. Therefore, I recommend that we offer an incentive program for members. This program might entice the casual rider to upgrade to an annual membership.

2) Our analysis illustrates a rise in the number of casual rides during the weekend. In regards to our products, we should create a weekend-only membership. It's a possibility that some of the casual riders could be interested in this type of membership.

3) The number of casual riders increasing their bike trips on Friday, Saturday, and Sunday, we could hold a special membership event on the weekends. We meet the casual riders where they already are, and this is a great opportunity to employ marketing campaigns to maximize the chance of a conversion. 

4) Instead of Cyclistic relying heavily on brand recognition to gain members, we can utilize a social media campaign to convince casual riders to become members. We could show the benefits being a member has to offer on all our social media platforms. 

