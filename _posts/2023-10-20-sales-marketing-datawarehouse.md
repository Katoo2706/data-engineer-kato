---
title: Sales & Marketing Data warehouse project
author: kato
date: 2023-06-20 14:10:00 +0800
categories: [Projects, Sales & Marketing Data warehouse]
tags: [Apache Airflow, Apache Spark, Polars, Apache Arrow, Pandas, Kubernetes, Helm chart, DBT]
render_with_liquid: false
published: true
---

## Introduction

## Project Architecture
Data Architecture Design (by @Kato)
![Data architecture](/assets/post/data-architect-design.png)

Core Warehouse Design
![Warehouse architecture](/assets/post/warehouse-architect.png)

## 1. Data Ingestion layer
![Polars](/assets/post/polars.png){: width="300" height="589" .w-30 .left}
<br>
**Ingestion framwork:**`Polars` with Python to process data on a single machine. [Polars](https://pola.rs/) is a scalable and efficient framework for handling data and is considered as an alternative to the very popular pandas library.

Compared to pandas, it can achieve more than 30x performance gains.

<div style="clear:both;"></div> 


> Updating...
{: .prompt-info }

## Learn More

For more knowledge about my posts, reach me via [katoo2706@gmail.com](mailto:katoo2706@gmail.com)
