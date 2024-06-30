## User session analysis - project for marketing
# Project overview
## 1. Defining business problem
Identify overall trends of all marketing campaigns on ecommerce site of the company. Find out if users tend to spend more time on the website on certain weekdays and how that behavior differs across campaigns. 
Present dynamic weekday duration, focusing on differences between marketing campaigns. Analyze whether there are interesting data points, that can give more insights. Provide visuals and presentation for business.

### Dataset description

Use  **raw_events table** hosted in **BigQuery** project. Data in raw_events table captures a lot of events from users based on their timestamps.

The dataset contains 31 columns and 4.295.584 rows.

**Important note:** there are no session identifiers in the dataset, therefore it is necessary to come up with logic for how to model sessions, calculate session duration and convert it to human readable format.

<details>

**<summary>Click to expand raw_events table schema</summary>**

### raw_events schema

| Field name | Type | Mode |
|---------------|-----------|-----------|
| event_date | STRING | NULLABLE |
| event_timestamp | INTEGER | NULLABLE |
| event_name |	 STRING | NULLABLE |
| event_value_in_usd| FLOAT| NULLABLE|
| user_id | STRING| NULLABLE|
| user_pseudo_id| STRING| NULLABLE|
| user_first_touch_timestamp| INTEGER| NULLABLE|
| category| STRING| NULLABLE |
| mobile_model_name | STRING| NULLABLE |
| mobile_brand_name |STRING | NULLABLE |
| operating_system | STRING | NULLABLE |
| language | STRING | NULLABLE |
| is_limited_ad_tracking| STRING | NULLABLE |
| browser | STRING | NULLABLE |
| browser_version | STRING | NULLABLE |
| country | STRING | NULLABLE |
| medium | STRING | NULLABLE |
| name | STRING | NULLABLE |
| traffic_source | STRING | NULLABLE |
| platform | STRING | NULLABLE |
| total_item_quantity | INTEGER | NULLABLE |
| purchase_revenue_in_usd | FLOAT | NULLABLE |
| refund_value_in_usd | FLOAT | NULLABLE |
| shipping_value_in_usd | FLOAT | NULLABLE |
| tax_value_in_usd | FLOAT | NULLABLE |
| transaction_id	 | STRING | NULLABLE |
| page_title | STRING | NULLABLE |
| page_location	 | STRING | NULLABLE |
| source | STRING | NULLABLE |
| page_referrer	 | STRING | NULLABLE |
| campaign | STRING | NULLABLE |

</details>


## 2.	Defining user session model
User session typically represents a continuous period of user activity on a website or application. We’ll consider a session as a sequence of events by the same user with relatively short gaps between them. 
A single user can come to a website on multiple days and using multiple devices. User’s activity on different devices should be considered as separate sessions. If the time difference between two events is less than a certain threshold, 
we consider them as part of the same session. If the time difference exceeds the threshold, we start a new session. As a benchmark, Google Analytics defaults to 30 minutes as a threshold for session gap.

For this project User session has been defined as a sequence of events per user per category (desktop, mobile, tablet). Since one user can come to a website using different devices, events on different devices are 
considered as parts of separate sessions. Gap in session aggregating events to separate sessions = 30 minutes.

For sequence of events per user per category:

- if the difference between events is less than or equals 30 minutes, we consider them as part of the same session,

- if the difference between events is longer than 30 minutes, this will be considered as a gap in session, starting a new session.

Defined user session model implies, that one user can have multiple sessions on multiple devices.

## 3.	Preparing and processing data

### 3.1.	Creating a table of unique sessions from raw_events table using SQL
Table from this query has been used for analysis in the next steps of the project. To view the code go to the uploaded SQL file [3.1. Sessions_data_query](https://github.com/PatrycjaDanilczuk/User-session-analysis-marketing-project/blob/main/3.1.%20Sessions_data_query)

<details>

**<summary>Click to expand the SQL code logic details</summary>**

### SQL code logic explanation:

1. Identify columns for the analysis: user_pseudo_id, category, campaign, country, event_name, purchase_revenue_in_usd, event_timestamp
2. Converting event_timestamp into datetime in microseconds using TIMESTAMP_MICROS()
3. Creating unique user_session_id (user can come to website from different devices, events on different devices should be considered as separate sessions)
4. Calculating difference between time ordered events in seconds per user_session_id  using LAG() and TIMESTAMP_DIFF()
5. Assigning gaps in sessions - if the difference between time ordered events is longer than 30 minutes, this will be considered as a break in session, the event assigned as break in session will be considered as a start of a new session
6. Assigning session number per user_session_id (user can have multiple sessions) using SUM() OVER
7. Creating session_id  from user_session_id and session_number - each session will be assigned with individual id
8. Getting session start time and session end time for each session_id using MIN() OVER and MAX()OVER
9. Calculating session duration in seconds
10. Creating additional fields for analysis:
      
  -	purchase_flag  (if the session ended up with purchase then 1)
  -	campaign (session will be assigned to a campaign, if there was at least one event in the session assigned to a campaign, other sessions assigned as no_campaign)
  -	session_date (session start gets credit as session day)
  -	weekday (session start gets credit as session weekday)
  -	purchase_revenue
  -	bounce_5_flag (sessions with duration <= 5 seconds)
  -	bounce_10_flag (sessions with duration <= 60 seconds)
  - user_engagement_segment –  based on time spent on site:  "up_to_5_SEC”,  "up_to_1_MIN",  "up_to_5_MIN", "up_to_15_MIN", "up_to_30_MIN", "over_30_MIN"
    
11. Selecting table of distinct sessions with relevant data for each session
</details>
  
### 3.2.	Extra step: “human readable format”: HH:MM:SS in SQL
Creating SQL table of overall trends by marketing campaign in “human readable format”: HH:MM:SS. To view the code go to the uploaded SQL file [3.2. Sessions_agg_query](https://github.com/PatrycjaDanilczuk/User-session-analysis-marketing-project/blob/main/3.2.%20Sessions_agg_query)

## 4.	Analyzing data

### 4.1. Using Python for descriptive statistics and other calculations
Using Python for getting descriptive statistics, analyzing data distribution, impact of possible outliers, selecting most suitable measure of central tendency, 
that should be used for further analysis of session duration variable across different dimensions, converting duration in seconds into “human readable format” HH:MM:SS, 
conducting additional calculations not supported by Tableau. 

The analysis and conclusions has been provided in 2 Jupyter Notebook files:

[4.1.1. Python_session_duration_distribution](https://github.com/PatrycjaDanilczuk/User-session-analysis-marketing-project/blob/main/4.1.1.%20Python_session_duration_distribution_2024.ipynb)

[4.1.2. Python_sessions_calculations](https://github.com/PatrycjaDanilczuk/User-session-analysis-marketing-project/blob/main/4.1.2.%20Python_sessions_calculations_2024.ipynb)

### 4.2.	Using Tableau for further analysis
Preparing Table Dashboard with relevant visuals. The link to the dashboard can be found here: [Usser session analysis dashboard](https://public.tableau.com/views/Marketing_session_duration_dashboard_2024_automatic/DOverview?:language=en-US&publish=yes&:sid=&:display_count=n&:origin=viz_share_link)

### 4.3. A/B Testing
Using A/B testing online calculator for testing statistical significance of certain findings.  
Results are presented in the PDF file [4.3. AB Testing results](https://github.com/PatrycjaDanilczuk/User-session-analysis-marketing-project/blob/main/4.3.%20AB%20Testing%20results_session_duration_2024.pdf)

## 5.	Communicating analysis results and providing actionable insights
Preparing presentation for business. The presentation has been uploaded in PDF file [5. Presentation](https://github.com/PatrycjaDanilczuk/User-session-analysis-marketing-project/blob/main/5.%20Presentation_session_duration_analysis_2024.pdf)

Use Tableau [Usser session analysis dashboard](https://public.tableau.com/views/Marketing_session_duration_dashboard_2024_automatic/DOverview?:language=en-US&publish=yes&:sid=&:display_count=n&:origin=viz_share_link) for more details.
