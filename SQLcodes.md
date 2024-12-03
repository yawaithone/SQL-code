# SQL-code
Projects executed with SQL

# installing required libraries
!pip install ipython-sql prettytable
import prettytable
prettytable.DEFAULT = 'DEFAULT'

# Load the SQL magic extension
%load_ext sql

# importing necessary libraries
import sqlite3, csv
import pandas as pd

# create database and read csv files using pandas and stored in dataframe
con = sqlite3.connect("FinalDB.db")
df1 = pd.read_csv('/content/ChicagoCensusData.csv')
df2 = pd.read_csv('/content/ChicagoCrimeData.csv')
df3 = pd.read_csv('/content/ChicagoPublicSchools.csv')
df1.to_sql("CENSUS_DATA",con, if_exists = "replace",index = False)
df2.to_sql("CHICAGO_CRIME_DATA",con, if_exists = "replace",index = False)
df3.to_sql("CHICAGO_PUBLIC_SCHOOLS",con, if_exists = "replace",index = False)

# Connect to the SQLite database
%sql sqlite:///FinalDB.db

# Query a few rows from each table
%sql select * from CENSUS_DATA limit 5;
%sql select * from CHICAGO_CRIME_DATA limit 5;
%sql select * from CHICAGO_PUBLIC_SCHOOLS limit 5;

# Find the total number of crimes recorded in the CRIME table.
%sql select count(*) from CHICAGO_CRIME_DATA;

# List community area names and numbers with per capita income less than 11000.
%sql select COMMUNITY_AREA_NAME,COMMUNITY_AREA_NUMBER from CENSUS_DATA where PER_CAPITA_INCOME < 11000;

# top 5 areas with highest per capita income
%sql select COMMUNITY_AREA_NAME, PER_CAPITA_INCOME from CENSUS_DATA order by PER_CAPITA_INCOME desc limit 5;

# top 5 areas with highest % of aged 25 without high school diploma
%sql select COMMUNITY_AREA_NAME, PERCENT_AGED_25__WITHOUT_HIGH_SCHOOL_DIPLOMA from CENSUS_DATA order by PERCENT_AGED_25__WITHOUT_HIGH_SCHOOL_DIPLOMA desc limit 5;

# 10 areas with highest percent of dependents (aged below 18 and above 64)
%%sql
select COMMUNITY_AREA_NAME, PERCENT_AGED_UNDER_18_OR_OVER_64
from CENSUS_DATA
order by PERCENT_AGED_UNDER_18_OR_OVER_64 desc limit 10;

# list all case numbers for crimes involving minors?
#(children are not considered minors for the purposes of crime analysis)
%%sql
SELECT CASE_NUMBER
FROM CHICAGO_CRIME_DATA
WHERE DESCRIPTION LIKE '%minor%'
  AND DESCRIPTION NOT LIKE '%child%';

# locations that crimes mostly took place
%sql select COUNT(CASE_NUMBER) as Crime_Cases,LOCATION_DESCRIPTION from CHICAGO_CRIME_DATA group by LOCATION_DESCRIPTION order by Crime_cases desc limit 10;

# crime cases in each year
%%sql
select strftime("%Y",DATE) as Year, Count(CASE_NUMBER) as No_of_Crime_Cases
from CHICAGO_CRIME_DATA
group by strftime("%Y",DATE)
order by strftime("%Y",DATE);

# List all kidnapping crimes involving a child?
%sql select * from CHICAGO_CRIME_DATA where PRIMARY_TYPE = "KIDNAPPING" AND DESCRIPTION LIKE '%child%';

# List the kind of crimes that were recorded at schools. (No repetitions)
%sql select COUNT(DISTINCT(CASE_NUMBER)) from CHICAGO_CRIME_DATA where LOCATION_DESCRIPTION = "SCHOOL";

# List the type of schools along with the average safety score for each type.
%sql select "Elementary, Middle, or High School" as type_of_sch, AVG(SAFETY_SCORE) from CHICAGO_PUBLIC_SCHOOLS group by "Elementary, Middle, or High School";

# List 5 community areas with highest % of households below poverty line
%sql select COMMUNITY_AREA_NAME, PERCENT_HOUSEHOLDS_BELOW_POVERTY from CENSUS_DATA order by PERCENT_HOUSEHOLDS_BELOW_POVERTY desc limit 5;

# Which community area is most crime prone? Display the coumminty area number only.
%sql select COMMUNITY_AREA_NUMBER, COUNT(ID) from CHICAGO_CRIME_DATA group by COMMUNITY_AREA_NUMBER order by COUNT(ID) desc limit 5;

# Use a sub-query to find the name of the community area with highest hardship index
%sql select COMMUNITY_AREA_NAME from CENSUS_DATA where HARDSHIP_INDEX = (select MAX(hardship_index)from CENSUS_DATA);

# Use a sub-query to determine the Community Area Name with most number of crimes?
%%sql
SELECT COMMUNITY_AREA_NAME
FROM CENSUS_DATA
WHERE COMMUNITY_AREA_NUMBER = (
    SELECT COMMUNITY_AREA_NUMBER
    FROM CHICAGO_CRIME_DATA
    GROUP BY COMMUNITY_AREA_NUMBER
    ORDER BY COUNT(ID) DESC
    LIMIT 1
);
