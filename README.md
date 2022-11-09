# BigQuery-GitHub-Public-Data-Analysis
## Introduction
BigQuery Github dataset is an open dataset released by Google, in collaboration with GitHub, Google BigQuery. It makes it possible to analyse open source software using SQL.\
Google BigQuery Public Datasets is a full snapshot of the content of more than 2.8 million open source GitHub repositories in BigQuery
## Objective 
- Know popularly used programming languague on open source project
- Investigate the common indenting method (space or tab) used by programmers \
## Prerequisites
A key knowledge required for this task is SQL, and an intermiediate level is recommended. If you need to learn SQL, or fresh your memory on it, check out the link below, they are all free:
- [Udacity SQL for data analysis](https://www.udacity.com/course/sql-for-data-analysis--ud198)
- [DataCamp introduction to SQL](https://www.datacamp.com/courses/introduction-to-sql)
- [Coursera SQL for data scientist](https://www.coursera.org/learn/sql-for-data-science)
- [FreeCodeCamp SQL for Beginners](https://www.youtube.com/watch?v=HXV3zeQKqGY&t=5606s)

Practice they say make prefer, I would recommend the using of [HackerRank SQL](https://www.hackerrank.com/domains/sql) for practise to get confortable using SQL.\
## Tools
For this analysis, you will be needing;
- Google cloud account (Qlik lab)
- BigQuery access
## Analysis Breakdown
__COPYWRITE:__ The majority of this code where made availabel on the lab instructure. However, I made few changes to understand BigQuery and make additional analysis. \
Before I dive into the analysis, BigQuery is a fully-managed, serverless data warehouse that enables scalable analysis over petabytes of data. I would say is google SQL but it does more than the conventional SQL, few of which I will highlight in this project.
### Finding The Data
Google on her BigQuery platform has a lot of public dataset which she pays for the storage and you pay for just the query. As a learner, you can used them to build awesome project, and as an expert you may need it to get new dataset or enhance existing one. For this project the Github dataset for open source projects was used. \
![png](https://github.com/uchiharon/BigQuery-GitHub-Public-Data-Analysis/blob/main/data1.png) \
To access public dataset on Bigquery, you can either search for the name on the search bar or click add data, follow by explore public dataset, as shown in the above image./
### Understanding The Data
To understand the metadata of a dataset on BigQuery, you simple hold down cltr key and click on the table name. This would provide you the following details:
- Schema: The columns and datatypes present in the table
- Details: How large is the table, size, rows, date created and modified, and data location
- Preview: To get an overview of the data without running any SQL 

|column           |datatype| descriptions |
| -------         | ----   |  --------------------------- |
|id               |STRING  |  the identification number of each repo |
|size             |INTEGER |  size of code |
|content          |STRING  |  code content of the repo |
|binary           |BOOLEAN |   |
|copies           |INTEGER |  number of forked copies|
|sample_repo_name |STRING  |  repo name |
|sample_path      |STRING  |  repo github path |

table 1: _Schema of the GitHub DataSet_
### Preview of Data
On the SQL query edition, the first 10 rows was previed using;

```
SELECT 
  * 
FROM 
  `bigquery-public-data.github_repos.sample_contents` 
LIMIT 
  10
```
### Split code by lines
To acheive the second objective, the content column was split by line.
```
WITH
  lines_of_code AS (
  SELECT
    SPLIT(content,'\n') AS line, # BREAK code INTO an ARRAY OF individual line sample_repo_name,
    sample_path
  FROM  `bigquery-public-data.github_repos.sample_contents`)
```
This is where BigQuery becomes interesting. On like the regular SQL database, BigQuery stores the result as an array over creating new role for each line of code as shown below. The concept of array implies that a list would be created in the content column instead of duplicating the other columns to form new rows for each line. I was amazed when this covered this, because array data structure was initially avaliable to just NoSQL like MongoDB, but now google made a stop that by including it in BigQuery. \
You might wonder why this interest me, and I would say its a game changer because this would reduce the size of the table, makes it query faster, and ultimately create room for making cost effective database architecture.

![png3](https://github.com/uchiharon/BigQuery-GitHub-Public-Data-Analysis/blob/main/Split%20code%20by%20line.png)
As you see in the above image, row 1 of line column has multiple values (an array)


### Flatten the line column
While store data in array makes its consume less space, fast and cheaper, arrays can not be interacted with directly and would need to be flattened to the convensional sql format.

```
flattened_code AS (
  SELECT
    sample_repo_name,
    sample_path,
    flattened_line
  FROM
    lines_of_code, UNNEST(line) AS flattened_line)
```
After flattening the code, it became;
![png4](https://github.com/uchiharon/BigQuery-GitHub-Public-Data-Analysis/blob/main/Flattening%20array.png)
