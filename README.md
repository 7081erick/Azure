# Azure Project

All the steps I took while building this project using data from Tokyo Olympics
The purpose of this project is to learn more about Azure technologies and services, I know that everything can be done inside Azure Synapse Analytics but I wanted to test and learn the services in their own

# Creating a Free Trial Account on Azure Cloud

## Creating a Storage account with Gen 2
    - Creating a folder for raw-data
    - Creating a folder for processed-data
    - Configuring Access Control (IAM)

## Creating a Data Factory and launching Data Factory Studio
    - Creating a flow to copy data from a GitHub repository 
    - In total 1 new pipeline with 5 Copy functions was created pulling the data raw link from / https://github.com/darshilparmar/tokyo-olympic-azure-data-engineering-project/tree/main/data
    - GitHub was the source and sink Azure Data Lake Storage Gen2 was storing the data

### Creating an App in App registration
    - Make sure to create an app and copy the following information: Client ID, Directory (Tenant) ID, and Create, and Secret Key Value (Under Certificates & Secrets create one)
    - Also, give the appropriate permissions for the app to navigate inside Data Storage
    - Under Access Control (IAM) / Under role assignments / Create a new role and add Storage Blob Data Distributor
    
 
## Creating Azure Databricks and Launch DataBricks
    - Create a notebook
    - Create a compute
    - I used the following to connect, extract, transform, and show data
    - Make sure to create an app 

from pyspark.sql.functions import col
from pyspark.sql.types import IntegerType, DoubleType, BooleanType, DataType

configs = {
  "fs.azure.account.auth.type": "OAuth",
  "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
  "fs.azure.account.oauth2.client.id": "Client ID",
  "fs.azure.account.oauth2.client.secret": "App Secret",
  "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/TenentID/oauth2/token"
}

dbutils.fs.mount(
  source="abfss://container@storagename.dfs.core.windows.net",
  mount_point="/mnt/tokyoolymic",
  extra_configs=configs
)

- Test code is pulling storage correctly

%fs
ls "/mnt/tokyoolymic"

- To create object Dataframes with Pyspark

athletes = spark.read.format("csv").option("header", "true").option("inferSchema","true").load("/mnt/tokyoolymic/raw-data/athletes.csv")
coaches = spark.read.format("csv").option("header", "true").option("inferSchema","true").load("/mnt/tokyoolymic/raw-data/coaches.csv")
entriesgender = spark.read.format("csv").option("header", "true").option("inferSchema","true").load("/mnt/tokyoolymic/raw-data/entriesgender.csv")
medals = spark.read.format("csv").option("header", "true").option("inferSchema","true").load("/mnt/tokyoolymic/raw-data/medals.csv")
teams = spark.read.format("csv").option("header", "true").option("inferSchema","true").load("/mnt/tokyoolymic/raw-data/teams.csv")

athletes:pyspark.sql.dataframe.DataFrame = [PersonName: string, Country: string ... 1 more field]
coaches:pyspark.sql.dataframe.DataFrame = [Name: string, Country: string ... 2 more fields]
entriesgender:pyspark.sql.dataframe.DataFrame = [Discipline: string, Female: integer ... 2 more fields]
medals:pyspark.sql.dataframe.DataFrame = [Rank: integer, Team_Country: string ... 5 more fields]
teams:pyspark.sql.dataframe.DataFrame = [TeamName: string, Discipline: string ... 2 more fields]

- Check Schema and see if the type of data is correctly being pulled

athletes.printSchema()
coaches.printSchema()
entriesgender.printSchema()
medals.printSchema()
teams.printSchema()

root
 |-- PersonName: string (nullable = true)
 |-- Country: string (nullable = true)
 |-- Discipline: string (nullable = true)

root
 |-- Name: string (nullable = true)
 |-- Country: string (nullable = true)
 |-- Discipline: string (nullable = true)
 |-- Event: string (nullable = true)

root
 |-- Discipline: string (nullable = true)
 |-- Female: integer (nullable = true)
 |-- Male: integer (nullable = true)
 |-- Total: integer (nullable = true)

root
 |-- Rank: integer (nullable = true)
 |-- Team_Country: string (nullable = true)
 |-- Gold: integer (nullable = true)
 |-- Silver: integer (nullable = true)
 |-- Bronze: integer (nullable = true)
 |-- Total: integer (nullable = true)
 |-- Rank by Total: integer (nullable = true)

root
 |-- TeamName: string (nullable = true)
 |-- Discipline: string (nullable = true)
 |-- Country: string (nullable = true)
 |-- Event: string (nullable = true)



- Changing a schema type to the desired
  
entriesgender = entriesgender.withColumn("Female",col("Female").cast(IntegerType()))\
    .withColumn("Male",col("Male").cast(IntegerType()))\
    .withColumn("Total",col("Total").cast(IntegerType()))

- Create Top Gold medals per Country

top_gold_medal_countries = medals.orderBy("Gold", ascending=False).select("Team_Country", "Gold")
top_gold_medal_countries.show()

+--------------------+----+
|        Team_Country|Gold|
+--------------------+----+
|United States of ...|  39|
|People's Republic...|  38|
|               Japan|  27|
|       Great Britain|  22|
|                 ROC|  20|
|           Australia|  17|
|         Netherlands|  10|
|              France|  10|
|             Germany|  10|
|               Italy|  10|
|              Canada|   7|
|              Brazil|   7|
|         New Zealand|   7|
|                Cuba|   7|
|             Hungary|   6|
|   Republic of Korea|   6|
|              Poland|   4|
|      Czech Republic|   4|
|               Kenya|   4|
|              Norway|   4|
+--------------------+----+
only showing top 20 rows


- Calculate the average number of entries by gender for each discipline

average_gender = entriesgender.withColumn(
    'Avg_Female', entriesgender['Female'] / entriesgender['Total'] * 100
).withColumn(
    'Avg_Male', entriesgender['Male'] / entriesgender['Total'] * 100
)
average_gender.show()

+--------------------+------+----+-----+------------------+------------------+
|          Discipline|Female|Male|Total|        Avg_Female|          Avg_Male|
+--------------------+------+----+-----+------------------+------------------+
|      3x3 Basketball|    32|  32|   64|              50.0|              50.0|
|             Archery|    64|  64|  128|              50.0|              50.0|
| Artistic Gymnastics|    98|  98|  196|              50.0|              50.0|
|   Artistic Swimming|   105|   0|  105|             100.0|               0.0|
|           Athletics|   969|1072| 2041| 47.47672709456149| 52.52327290543851|
|           Badminton|    86|  87|  173| 49.71098265895954| 50.28901734104046|
|   Baseball/Softball|    90| 144|  234| 38.46153846153847| 61.53846153846154|
|          Basketball|   144| 144|  288|              50.0|              50.0|
|    Beach Volleyball|    48|  48|   96|              50.0|              50.0|
|              Boxing|   102| 187|  289|35.294117647058826| 64.70588235294117|
|        Canoe Slalom|    41|  41|   82|              50.0|              50.0|
|        Canoe Sprint|   123| 126|  249| 49.39759036144578|50.602409638554214|
|Cycling BMX Frees...|    10|   9|   19| 52.63157894736842|47.368421052631575|
|  Cycling BMX Racing|    24|  24|   48|              50.0|              50.0|
|Cycling Mountain ...|    38|  38|   76|              50.0|              50.0|
|        Cycling Road|    70| 131|  201| 34.82587064676617| 65.17412935323384|
|       Cycling Track|    90|  99|  189| 47.61904761904761| 52.38095238095239|
|              Diving|    72|  71|  143|50.349650349650354| 49.65034965034965|
|          Equestrian|    73| 125|  198|36.868686868686865| 63.13131313131313|
|             Fencing|   107| 108|  215| 49.76744186046512| 50.23255813953489|
+--------------------+------+----+-----+------------------+------------------+
only showing top 20 rows


- Save transformed data into transformed-data on data storage gen2

athletes.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolymic/transformed-data/athletes")
coaches.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolymic/transformed-data/coaches")
entriesgender.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolymic/transformed-data/entriesgender")
medals.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolymic/transformed-data/medals")
teams.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolymic/transformed-data/teams")

## Creating and launching Azure Synapse Analytics
- Under Data / I created a Lake Database 
- I created tables using data we previously stored and manipulated from Databricks
- Now we can analyze using SQL and even integrate with PowerBI

## Some SQL used to breathly analyze the data

-- Count the number of athletes per country

SELECT Country, COUNT(*) as TotalAthetlets 
from athletes 
group BY Country 
order by TotalAthetlets desc;

-- Calculate the total medals won by each country
SELECT team_country, sum(gold) as Gold, sum(silver) as Silver, SUM(bronze) as Bronze
from medals
GROUP by team_country
ORDER by gold desc;

-- Calculate the average number of entries by gender
SELECT Discipline, AVG(Female) Avg_Female, AVG(Male) Avg_Male 
from entriesgender
group by Discipline
order by Avg_Female desc;
