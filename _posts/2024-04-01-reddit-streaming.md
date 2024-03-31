---
title: Streaming Reddit Data with Confluence Kafka & Pyspark on fastAPI
author: kato
date: 2024-04-01 00:00:00 +0800
categories: [Big Data, Streaming]
tags: [Kafka, fastAPI, Cassandra, Pyspark, Spicy, ]
render_with_liquid: false
published: true
image:
  path: /assets/post/reddit-streaming-banner.png
---

## Introduction
The project is a robust streaming pipeline built with FastAPI, Kafka, Pyspark, Cassandra, Spacy, Redshift, and Grafana. It provides a scalable and efficient architecture for processing and analyzing data from Reddit in real-time.

**Git repo:** &nbsp; [<img src="https://git-scm.com/images/logos/1color-lightbg@2x.png" width="45">](https://github.com/Katoo2706/redditStreaming)

## Project Architecture

![reddit-streaming.png](/assets/post/reddit-streaming.png)

**FastAPI** serves as the core of the API layer, providing robust data validation using Pydantic models. The API allows users to start and stop streaming threads for specific subreddits, enabling real-time data ingestion.

**Kafka and Zookeeper** form the backbone of the messaging system, ensuring fault tolerance and high throughput. With three brokers, Kafka provides a resilient and scalable platform for handling large volumes of data. The integration with Confluent Schema Registry ensures data consistency and compatibility across different services.

**Kafdrop** serves as a monitoring tool for Kafka, allowing users to track topics and messages across brokers. This facilitates easy monitoring and debugging of the streaming pipeline.

**Pyspark** is used for distributed data processing, enabling efficient data transformation and analysis at scale.

**Cassandra** acts as the backend database for storing structured data, providing high availability and scalability for storing Reddit submissions and related information.

**Spacy** is leveraged for natural language processing tasks, enabling advanced text analytics and sentiment analysis on Reddit submissions.

**Redshift** serves as a data warehouse for storing and analyzing large volumes of structured data, providing powerful querying capabilities for deriving insights from Reddit data.

**Grafana** offers a rich visualization platform for monitoring and analyzing data metrics, providing real-time dashboards for monitoring the performance of the streaming pipeline and analyzing key metrics.

Overall, the project offers a comprehensive solution for processing and analyzing Reddit data in real-time, enabling users to derive valuable insights and make data-driven decisions.
