---
title: Project - Human Resources
author: kato
date: 2023-06-20 14:10:00 +0800
categories: [Projects, Human Resources]
tags: [MongoDB, Docker, Streamlit, Cloud Deta, Postgres, Apache Airflow]
render_with_liquid: false
published: true
---

## Great purposes:
- Developed a robust data collector using Streamlit application and a Cloud Database.
- Designed and managed a non-relational staging database to ingest data from `Jira` (Track recruitment activities), `Humi`, `1Office`, and performed web scraping from the Breadstack career site.
- Successfully tested and integrated data from additional sources (`Trello`, `Snipe-it`, `LinkedIn Jobs`)
- Implemented a Postgres data warehouse for the HR project to make unified Human resources data model.
- Developed extract, transform, and load (ETL) pipelines to move data from various sources to staging, and from staging (including Streamlit app) to the data warehouse.

## Architecture
![bs-hr-architecture.png](/assets/post/bs-hr-architecture.png)

## Data warehouse model
<iframe
  src="https://dbdocs.io/katoo2706/human_resources_management?view=relationships"
  style="width:100%; height:600px;"></iframe>
[Reference](https://dbdocs.io/katoo2706/human_resources_management?view=relationships)

## Data Collector Application
> `Streamlit`, `plotly`, `deta space`

In this project, I use `Streamlit` to develop a Data Collector Application to gather metrics data from Business user.\
Data from Data Collector will be stored in Cloud Database (`Mongo Atlas`)

## Staging Area
> NoSQL Database: `MongoDB`

Using non-structured database for staging area will take the advantages of data retrieval from different data sources and we don't need to take time to ensure the data source structure.
On the other hand, It can make the extract and load process will be fault tolerance and easy to scale.



## Learn More

For more knowledge about my posts, reach me via [katoo@gmail.com](mailto:katoo@gmail.com)
