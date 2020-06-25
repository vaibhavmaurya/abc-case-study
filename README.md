
# Case Study: ABC International

## Table of contents
- [Case Study: ABC International](#case-study)
  - [Supplier Financial Data Analysis](#about-tickets-issued-dataset)
    - [Financial Data ETL](#background-of-organization)
    - [Financial Data Search](#financial-data-search)
	    - [Proposal 1: Financial Data API](#financial-data-api)
	    - [Proposal 2: Web Crawler ](#web-crawler)
	- [Suppliers Profiling and Scoring](#profiling-and-scoring)
  - [Sentiment Analysis for ABC International](#sentiment-analysis)
    - [News Feed API](#news-feed-api)
    - [News Text Data ETL](#news-text-data-etl)
    - [Sentiment Analysis](#sentiment-analysis)
      - [Entity Recognition](#entity-recognition)
      - [Detect Sentiment](#detect-sentiment)
      - [Intent Matchine](#intent-matching)
   - [Roadmap](#roadmap)
	    - [Supplier Assignement Optimization](#supplier-assignment-optimization)
	    - [Risk Assesment](#risk-assesment)
  - [References](#references)

## About Tickets Issued Dataset <a name="AboutTicketData"></a>

This dataset is issued by organization KSRTC. It is a State Road Transport Corporation.

### BackGround of Organization <a name="OrgBackground"></a>

The Karnataka State Road Transport Corporation (KSRTC) is a state-owned road
transportation company catering to the different cities, towns and villages within and outside Karnataka. The corporation has the largest fleet of Volvo buses among the different state-owned transport companies in India. It is wholly owned by the Government of Karnataka.

### Services Provided by Organization <a name="OrgServices"></a>
Services covers 92% villages in Karnataka. KSRTC operates with a total fleet of 23829 buses (KSRTC-8348, NEKRTC-4343, NWKRTC-4716, and BMTC–6422). It transports, on an average, 74.57 lakh passengers per day. It also operates to the neighbouring states of Maharashtra, Andhra Pradesh, Telangana, Tamil Nadu, Puducherry, Goa and Kerala. KSRTC was the first state transport corporation to introduce Volvo B7RLE low floor city buses in India in 2005. At present, KSRTC operates TATA, Ashok Leyland, Eicher Motors are More, Also Volvo, Mercedes Benz, Scania buses under the A/C (Airavat) services (Airavat means the mythical white elephant in Kannada).


### Dataset Info. <a name="DatasetInfo"></a>

- Dataset issued by KSRTC contains ticket issued to the passenger.
- Each row is the ticket issued to passenger. Though some observations contains route change of the bus and route direction change of the bus also.

### Structure of Dataset <a name="DataSetStructure"></a>

|  Column Name | Description  |
| ------------ | ------------ |
| ETD_WAYBILL_NO   | Indetifier for Bus  |
| ETD_ROUTE_NO   | Identifier for the current route of the bus  |
|  ETD_ROUTE_TYPE  | Type of route  |
| ETD_TRIP_NO   | Indetifier for the current trip of a bus for agiven route  |
|  ETD_TICKET_TYPE  | Type of ticket sold  |
| ETD_ADULTS   | No of adults boarded the bus  |
| ETD_CHILD   |  No of kids boarded the bus |
|  ETD_AMOUNT  | Expense beared for a ticket sold  |
| ETD_DEPOT_CODE   | Depot for which this belongs  |
| ETD_BATTERY_VOLT   | Battery voltage  |
| ETD_CUR_STOP_NAME   | Name of the current stop from where passenger has boarded  |
| ETD_DST_STOP_NAME   | Name of the current stop from where passenger has hop off  |
| ETD_KMS   | Distance between source bus stop and destination bus stop  |
| ETD_TICKET_TYPE_DESCR   | Description of the type of the ticket sold  |
| ETD_TRIP_DIRECTION   | Direction of a route  |
| ETD_TICKET_NO   | Identfier for the ticket sold  |
| ETD_TICKET_SUBNO   |  Sub Identfier for the ticket sold |
| ETD_CUR_STOP_NO   |  No assigned to the current bus stop for a given route |
| ETD_CUR_STOP_CODE   | Code assigned to the current bus stop for a given route |
| ETD_DST_STOP_NO   | No assigned to the destination bus stop for a given route  |
| ETD_DST_STOP_CODE   | Code assigned to the destination bus stop for a given route  |
| ETD_CUR_SUB_STAGE   | Sub stage assigned to the current bus stop for a given route  |
| ETD_DST_SUB_STAGE   | Sub stage assigned to the destination bus stop for a given route  |
| ETD_DATE  | Date of the ticket issued  |
| ETD_TD_TIME  | Time of the ticket issued  |


## Data Wrangling <a name="datawrangling"></a>

Dataset url : [full_data.zip](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/data/full_data.zip)

This archived file contains a text file named `aug18-dec18.txt`.

This dataset is paginated and contains unnecessary strings as follows.
![](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/images/datapreview.png)

#### Challenges in the dataset
This dataset has following issues:

- Dataset is in a flat text file. It is paginated, 21 rows per page, though all pages are in the same text file seperated by new line.
- Columns in the dataset are not fully qualified names. Each column is seperated by space.
- Size of a column is difficult to determine.
- The text files contains some additional strings in the begining and in the end also.

Fortunately this dataset has following pros.

- Dataset columns positions are fixed throughout the dataset.
- Each page is uniformly seperated by newline throughout the dataset.

### Gathering Data <a name="gatherdata"></a>

#### Challenges

- Data is scattered accross multiple Bus Depots.
- Each Depot maintain their on set of current bus stop codes and destination bus stop codes.
- There is no centralized data organization for tickets.
- It is required to go each Depot and request for data, which again comes out in the form of inconsistent xlsx, csv or text files.


To parse dataset into csv, following is the strategy opted:

- Created a [json file](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/src/datastructure.json) to parse text dataset. This json file contains column name as key and value is the object which contains start of the column, size of the column and position of the column.
- Created a [library](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/src/DataProcessingModule.py) to clean the dataset.

The program follows the logic as below.

![](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/images/DataCleaning.png)

Data is gathered in the [notebook](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/notebooks/Data_Wrangling.ipynb).


## Data Exploration Results <a name="exploration"></a>

Data is explored in step by step basis as follows.

### Data Walkthrough <a name="exploration_walkthrough"></a>

Overall picture of dataset is as below.
- There are 577850 rows and 24 columns.
- There are 4 columns of datatype string, rest all are numeric.
- There is one column ETD_WAYBILL_NO which identifies a bus uniquely.
- There is one column ETD_ROUTE_NO which identifies a route of the bus uniquely.
- There are 6 categorical columns even though they are numeric. Those are "ETD_ROUTE_TYPE", "ETD_TICKET_TYPE", "ETD_DEPOT_CODE", "ETD_CUR_STOP_NAME", "ETD_DST_STOP_NAME" and "ETD_TRIP_DIRECTION".
- There are 5 quantitative columns. those are "ETD_ADULTS", "ETD_CHILD", "ETD_AMOUNT", "ETD_BATTERY_VOLT" and "ETD_KMS"
- ETD_CUR_STOP_NO is the indetifier of bus stop and ETD_CUR_STOP_NAME is the name of the bus stop. The same follows with ETD_DST_STOP_NO and ETD_DST_STOP_NAME
- ETD_CUR_STOP_CODE is assumed to be code of the bus stop though there is already a field ETD_CUR_STOP_NO to uniquely identify a bus stop. Field ETD_CUR_STOP_CODE, purpose is unknown. Same follows with ETD_DST_STOP_CODE.

### Defining scope of the data analysis, so adjust and clean the data accordingly <a name="scopeandclean"></a>

#### Scope of the Data Exploration
Idea is to 
- Explore distribution of tickets issued to passengers. It's relationship to ticket type, date time, bus stops etc.
- Explore the fare recieved from passengers.

#### Certain Data changes

Following changes are needed in the dataset.

- ETD_ADULTS and ETD_CHILD columns depict no of adults and children boarded the bus. These columns can be combined to one single column PASSENGERS.
- Some rows contains ETD_ADULTS and ETD_CHILD as 0. These rows are specific to certain purposes defined by [KSRTC](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/README.md#OrgBackground). These rows does not contain information about ticket issued and so these are out of the scope.
- Columns ETD_CUR_SUB_STAGE, ETD_DST_SUB_STAGE and ETD_TICKET_SUBNO are always 0. It is better to remove them.
- ETD_BATTERY_VOLT column is battery voltage of the bus when ticket is issued. This column is out of scope.
- ETD_DST_STOP_CODE and ETD_CUR_STOP_CODE, both fields the purpose is not known.
- It would be good to have fate and time decomposed to month and week. So that monthly and weekly behavior can be analysed further.
- Decompose date and time column to month, week and weekday columns. These columns will be used to find passengers congestion based on various time frames.

### Analysis of various characteristics of dataset <a name="characteristicsanalysis"></a>

- Dataset timeline starts from 1st Aug 2018 and ends on 31 Dec 2018.
- As per the above bar chart "Distribution of tickets issued Monthly", maximum tickets sold in December and least in September
- As per the chart "Distribution of ticktes issued week of the year", data is almost consistent from week 31 to 51 but the week 30 and 52 shows least tickets sold. This is because there is no full data for week 30 and 52. The timeline starts from the mid of the week 30 and ends at mid of the week 52.
- There is only one route in the dataset.
- There are total 2805 buses in the dataset.
- There is only one depot in the dataset.
- There are 17 bus stops in the dataset.
- Maximum passengers boarded a bus is 32, which seems unrealistic or very rare.
- Minimum, median and mean are almost same, which means that in more than 75% of the data one ticket sold contains only one passenger.
- More than 75% of the tickets sold for fare less than and equal to 145.
- There is a maximum fare taken which is 2820, which is because the ticket sold may have more than one passengers. As already analysed column PASSENGERS, throughout the data maximum one ticket sold to one passenger.
- Average distance is 87, which is different from median 52. Standard deviation is considerably high.
- Though in the distribution chart it seems the maximum tickets sold for distance between more than 0 to less than 55.
- 75% of tickets sold for distance less than or equal to 128.
- There is a ticket for minimum distance 0, which seems unlikely.
- There is a strong positive correlation between ETD_AMOUNT and ETD_KMS, which evdident though. Though the fare recieved depends on no of passengers per ticket also.
- Rest combinations does not show any correlation.
- For trip no 1 and 2, maximum buses are allocated.
- Since there are only 2805 buses in the dataset. It means buses are running for both trips 1 and 2.
- In trips 6, 7 and 8, buses passes through 14 bus stops, in trip 5 bus passes through 15 bus stops and , trips 1,2,3, and 4 bus passes through all bus stops.

- For ticket type 11 average passengers more than 4, which is maximum.
- For ticket types 12 to 19 and 32 to 57, average passenger is one.
- On average close to 1.6, passengers boards from bus stop 6.
- Boxplot for passengers with respect to boarding bus stop does not give clear information. Though it seems from bus top 1, once more than 30 passengers boarded which according to dataset is highly unlikely, can be considered as outlier.


### Analysis of impact of various characteristics of dataset on number tickets sale <a name="ticketsaleanalysis"></a>

- Dataset has 466956 tickets sold, starting from 1st Aug 2018 till on 31 Dec 2018.
- Date time column is decomposed to analyse no of passengers in different time spans.
- To condsider bus stops, columns ETD_CUR_STOP_NO and ETD_DST_STOP_NO should be used since these columns contains number uniquely assigned to each bus stops.

- After observing average passengers monthly, it looks like average demand increases from August to September. Then November and December average remain almost same.
- Week of the year wise average passengers observation, it is consistent with month wise average passengers. But 52th week, which is the last week, there is a sudden surge in average demand. It may be because of winter vacation.
- Considering the weekday wise average passengers distribution. Thursday looks like more passengers board in comparison to other days of the week.

- There are few source and destination bus stops combination which shows higher demand in comparison to other source and destination bus stops. Routes like GUBBI-NITTUR, NITTUR-KB CROSS, BASAV.BUS.S-BHADRAVATHI.
- It requires more data from other depots to come up with more variation in the data.
- Ticket type 11 and 21 have covered most of the passengers. Where ticket type 11 shows the highest sale, which is actually the student pass.


## How to run this project <a name="run"></a>

1. Download this project to local.
2. Extract data [zip file](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/data/full_data.zip)
3. Create 4 folders inside the [data](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/tree/master/data) folder: `clean_data`, `db`, `processed`, and `raw`.
4. Copy file aug18-dec18.txt from extracted folder `full_data` to folder `raw`
5. Run jupyter notebook [Data_Wrangling.ipynb](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/notebooks/Data_Wrangling.ipynb).
6. For data analysis run jupyter notebook [PassengersDistributionAnalysis.ipynb](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/notebooks/PassengersDistributionAnalysis.ipynb)

Following are the jupyter notebooks html version.

- [Data Wrangling and Data exploration](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/notebooks/Data_Wrangling.slides.html)
- [Data Analysis with visulaization](https://github.com/vaibhavmaurya/BusAndTicketsOptimization/blob/master/notebooks/PassengersDistributionAnalysis.slides.html)