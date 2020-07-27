# Project: Data Modeling with Apache Cassandra
A startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. The analysis team is particularly interested in understanding what songs users are listening to. Currently, there is no easy way to query the data to generate the results, since the data reside in a directory of CSV files on user activity on the app.

They'd like a data engineer to create an Apache Cassandra database which can create queries on song play data to answer the questions, and wish to bring you on the project. Your role is to create a database for this analysis. You'll be able to test your database by running queries given to you by the analytics team from Sparkify to create the results.

# Project Overview
In this project, you'll apply what you've learned on data modeling with Apache Cassandra and complete an ETL pipeline using Python. To complete the project, you will need to model your data by creating tables in Apache Cassandra to run queries. You are provided with part of the ETL pipeline that transfers data from a set of CSV files within a directory to create a streamlined CSV file to model and insert data into Apache Cassandra tables.

We have provided you with a project template that takes care of all the imports and provides a structure for ETL pipeline you'd need to process this data.

## Datasets
For this project, you'll be working with one dataset: event_data. The directory of CSV files partitioned by date. Here are examples of filepaths to two files in the dataset:

event_data/2018-11-08-events.csv
event_data/2018-11-09-events.csv

## Project Template
To get started with the project, go to the workspace on the next page, where you'll find the project template (a Jupyter notebook file). You can work on your project and submit your work through this workspace.

The project template includes one Jupyter Notebook file, in which:

you will process the event_datafile_new.csv dataset to create a denormalized dataset
you will model the data tables keeping in mind the queries you need to run
you have been provided queries that you will need to model your data tables for
you will load the data into tables you create in Apache Cassandra and run your queries

## Project Steps
Below are steps you can follow to complete each component of this project.

Modeling your NoSQL database or Apache Cassandra database
Design tables to answer the queries outlined in the project template
Write Apache Cassandra CREATE KEYSPACE and SET KEYSPACE statements
Develop your CREATE statement for each of the tables to address each question
Load the data with INSERT statement for each of the tables
Include IF NOT EXISTS clauses in your CREATE statements to create tables only if the tables do not already exist. We recommend you also include DROP TABLE statement for each table, this way you can run drop and create tables whenever you want to reset your database and test your ETL pipeline
Test by running the proper select statements with the correct WHERE clause

## Build ETL Pipeline
Implement the logic in section Part I of the notebook template to iterate through each event file in event_data to process and create a new CSV file in Python
Make necessary edits to Part II of the notebook template to include Apache Cassandra CREATE and INSERT statements to load processed records into relevant tables in your data model
Test by running SELECT statements after running the queries on your database

## File Overview
event_data/ - contains our data for user sessions in the streaming app in csv format. files are partitioned by day. (e.g. 2018-11-02-events.csv)
images - contains an image of what our data looks like after an ETL process
data_modeling_with_cassandra.ipynb - Has two parts:
ETL pipeline for processing the event files and compiling them into a single csv with the desired columns
Data modeling examples that show three queries and the creation of three tables to serve those queries
Note: the data is not included in this repo, but can be shared upon request.

To run this, you simply need to open the Notebook and run all cells.

The Data Model: 1 Query, 1 Table
Subset of our data after the ETL pipeline:

Final Data

Queries to model for
Song plays by session

**Query 1:**

Give me the artist, song title and song's length in the music app history that was heard during sessionId = 338, and itemInSession = 4

Required fields: artist, song, length, sessionid, itemInSession

Partition Key: should be sessionId, because this is the primary way our data is being filtered in the query. We are also filtering on itemInSession, but that field is less specific and would result in a less logical distribution across nodes.

Primary Key: sessionId is not sufficient for a PK, because it is not unique. However, adding itemInSession as a clustering column with give us uniqueness.

Song plays by User's session

**Query 2:**

Give me only the following: name of artist, song (sorted by itemInSession) and user (first and last name) for userid = 10, sessionid = 182

Required fields: artist, song, itemInSession, firstName, lastName, userId, sessionId

Partition Key: should be userId in this case, because this is the primary way our data is being filtered in the query. We are also filtering on sessionId, but again that field is less specific and would result in a less logical distribution across nodes.

Primary Key: In this table userId would not be unique, and adding sessionId would still not guarantee uniqueness. However, the query also asks for sorting by itemInSession, which means we need it as a clustering column anyway. So a PK that is both unique and sorts the way we would like is (userId, sessionId, itemInSession).

Users by Song listens

**Query 3:**

Give me every user name (first and last) in my music app history who listened to the song 'All Hands Against His Own'

Required fields: firstName, lastName, song, userId

Partition Key: we're filtering by song in this case, so we'll use that as our partition key

Primary Key: It's not necessarily required given our query, but I've chosen to include the userId field so that we can add it to our PK. I've done this for two reasons, the first being that we need a unique PK and (song) or (song, firstName) don't suffice. Another reason would be to give us some sorting by userId, which feels like a logical way to order. If instead we wanted to sort alphabetically we could do (song, firstName, userId).

## **Why was a NoSQL Database chosen?**
Well, the real answer is, to get experience with a NoSQL database. But if think about this as if it were a real production system, we might ask:

## **When would we want NoSQL over a relational DB?**
Perhaps you have LARGE amounts of data. Because a relational database can only be scaled by adding machine memory, its very possible to have a table too large for a single machine. So if you want distributed tables, NoSQL is what you want. Some other potential reasons are: need for high throughput that isn't slowed down by ACID transactions or high availability. In our case Apache Cassandra is an AP tolerant system, so we're optimizing for availability and partition tolerance.

## **What are the caveats of a NoSQL database like Apache Cassandra?**
So first off, based on CAP Theorem, we know that what's being sacrificed by our AP tolerant system is consistency. Which means it's possible that a read from our DB might not give us the most up to date information, instead we settle for eventual consistency. Another thing we must take into account is the lack of JOINs. One of the nice aspects of a relational database is the query flexibility and capability to do aggregations. With Cassandra, we don't have that ability, so instead we must know about the queries our Data Scientists would like before hand so we can model our tables to fit them. And if our queries change, so must our tables.

## **Which of these combinations is desirable for a production system - Consistency and Availability, Consistency and Partition Tolerance, or Availability and Partition Tolerance?**
As the CAP Theorem Wikipedia entry says, "The CAP theorem implies that in the presence of a network partition, one has to choose between consistency and availability." So there is no such thing as Consistency and Availability in a distributed database since it must always tolerate network issues. You can only have Consistency and Partition Tolerance (CP) or Availability and Partition Tolerance (AP). Remember, relational and non-relational databases do different things, and that's why most companies have both types of database systems.

## **Is Eventual Consistency the opposite of what is promised by SQL database per the ACID principle?**
Much has been written about how Consistency is interpreted in the ACID principle and the CAP theorem. Consistency in the ACID principle refers to the requirement that only transactions that abide by constraints and database rules are written into the database, otherwise the database keeps previous state. In other words, the data should be correct across all rows and tables. However, consistency in the CAP theorem refers to every read from the database getting the latest piece of data or an error.
