---
title: "Understanding the Airflow Metadata Database"
description: "An structural walkthrough of Apache Airflow's metadata database, with a full ERD."
date: 2020-11-21T00:00:00.000Z
slug: "airflow-database"
tags: ["Database", "SQL", "Components"]
---

## Overview

> Note: This guide has been written and tested with Airflow Airflow 1.10.10. New tables may be introduced in newer versions. 

We at Astronomer often get asked about the structure of the underlying Airflow metadata database. The Airflow metadata database stores configurations, such as variables and connections, user information, roles, and policies. It is also the Airflow Scheduler's source of truth for all metadata regarding DAGs, schedule intervals, statistics from each run, and tasks. 

Airflow uses SQLAlchemy and Object Relational Mapping (ORM) in Python to connect and interact with the underlying metadata database from the application layer. Thus, any database supported by [SQLAlchemy](https://www.sqlalchemy.org/) can theoretically be configured to host Airflow's metadata. On Astronomer, each Airflow deployment is equipped with PostgreSQL database for this purpose. The following guide details all of the tables available in the database repository including dependencies and the complete ERD. 

Even though we don't recommend modifying the values directly on the database, as doing so might affect dependencies, understanding the underlying structure can be very useful when it comes to building your own reports or querying directly against the database. You can find some useful queries that go directly against these tables in our [Useful SQL queries for Apache Airflow](https://www.astronomer.io/guides/airflow-queries/) guide. 


## ERD Diagram

The following diagram displays all the tables from the Airflow database. 

![Airflow DB](https://assets2.astronomer.io/main/guides/airflow_db.png)

------------------------------------------------------------------------

## Tables

The Airflow metadata database has a total of 30 tables  tables are stored on the public schema by default. The following describe the table structure and reference for the Airflow metadata tables. 

### public.ab_permission
 
**Table Structure:**

------------------------------------------------------------------------

|  F-Key   | Name  |Type | Description |
|----------|-------|--------|------------------|
|          | id    |integer | *PRIMARY KEY*    |
|          | name  |character varying(100)  | *UNIQUE NOT NULL* |

**Tables referencing this one via Foreign Key Constraints:**

-   [public.ab\_permission\_view](#public.ab\_permission\_view)

------------------------------------------------------------------------

### public.ab\_permission\_view

**Table Structure:**

------------------------------------------------------------------------

|F-Key                                                 |Name            |Type      |Description   |
|-------------------|---------------|---------|-------------|
|                                                      |id              |integer   |*PRIMARY KEY*|
|  [public.ab\_permission.id](#public.ab\_permission)   |permission\_id   |integer   |*UNIQUE\#1*|
|  [public.ab\_view\_menu.id](#public.ab\_view\_menu)    |view\_menu\_id    |integer   |*UNIQUE\#1*|

**Tables referencing this one via Foreign Key Constraints:**

-   [public.ab\_permission\_view\_role](#public.ab\_permission\_view\_role)

------------------------------------------------------------------------

### public.ab\_permission\_view\_role

**Table Structure:**

------------------------------------------------------------------------

|F-Key  |Name |Type |Description|
|------|----|----|----------|
|       |id   |integer |   *PRIMARY KEY*|
|[public.ab\_permission\_view.id](#public.ab\_permission\_view)   |permission\_view\_id   |integer   |*UNIQUE\#1*|
|[public.ab\_role.id](#public.ab\_role)                         |role\_id              |integer   |*UNIQUE\#1*|

------------------------------------------------------------------------

### public.ab\_register\_user

**Table Structure:**

------------------------------------------------------------------------

|  F-Key  |Name                |Type                          |Description|
|------- |-------------------|-----------------------------|------------------|
||          id                  |integer                       |*PRIMARY KEY*
||          first_name          |character varying(64)         |*NOT NULL*
||          last_name           |character varying(64)         |*NOT NULL*
||          username            |character varying(64)         |*UNIQUE NOT NULL*
||          password            |character varying(256)        |
||          email               |character varying(64)         |*NOT NULL*
||          registration_date   |timestamp without time zone   |
||          registration_hash   |character varying(256)        |

------------------------------------------------------------------------

### public.ab_role

**Table Structure:**

------------------------------------------------------------------------

|F-Key     |Name   |Type                    |Description|
|-------  |------|-----------------------|-------------------|
|          |id     |integer                 |*PRIMARY KEY*|
|          |name   |character varying(64)   |*UNIQUE NOT NULL*|

**Tables referencing this one via Foreign Key Constraints:**

-   [public.ab\_permission\_view\_role](#public.ab-permission\_view\_role)
-   [public.ab\_user\_role](#public.ab\_user\_role)

------------------------------------------------------------------------

### public.ab_user

**Table Structure:**

------------------------------------------------------------------------

|F-Key                                        |Name                 |Type                          |Description
|--------------------------------------------|--------------------|-----------------------------|-------------------
|                                             |  id                 |integer                       |*PRIMARY KEY*
|                                             |  first_name         |character varying(64)         |*NOT NULL*
|                                             |  last_name          |character varying(64)         |*NOT NULL*
|                                             |  username           |character varying(64)         |*UNIQUE NOT NULL*
|                                             |  password           |character varying(256)        |
|                                             |  active             |boolean                       |
|                                             |  email              |character varying(64)         |*UNIQUE NOT NULL*
|                                             |  last_login         |timestamp without time zone   |
|                                             |  login_count        |integer                       |
|                                             |  fail\_login\_count   |integer                       |
|                                             |  created_on         |timestamp without time zone   |
|                                             |  changed_on         |timestamp without time zone   |
|  [public.ab\_user.id](#public.ab\_user) |  created\_by\_fk      |integer                       |
|  [public.ab\_user.id](#public.ab\_user) |  changed\_by\_fk      |integer                       |

**Tables referencing this one via Foreign Key Constraints:**

-   [public.ab\_user](#public.ab\_user)
-   [public.ab\_user\_role](#public.ab\_user\_role)

------------------------------------------------------------------------

### public.ab\_user\_role

**Table Structure:**

------------------------------------------------------------------------

|F-Key                                        |Name      |Type      |Description
|------------------------------------------- |-------- |-------- |--------------
|                                             |id        |integer   |*PRIMARY KEY*
|[public.ab\_user.id](#public.ab\_user)   |user\_id   |integer   |*UNIQUE\#1*
|[public.ab\_role.id](#public.ab\_role)   |role\_id   |integer   |*UNIQUE\#1*

------------------------------------------------------------------------

### public.ab\_view\_menu
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name   |Type                     |Description
|------ |----- |----------------------- |-------------------
|        |id     |integer                  |*PRIMARY KEY*
|        |name   |character varying(100)   |*UNIQUE NOT NULL*

**Tables referencing this one via Foreign Key Constraints:**

-   [public.ab\_permission\_view](#public.ab\_permission\_view)

------------------------------------------------------------------------

### public.alembic_version
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name          |Type                    |Description
|------- |------------ |---------------------- |--------------
|        |version_num   |character varying(32)   |*PRIMARY KEY*

------------------------------------------------------------------------

### public.chart
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key                                    |Name              |Type                       |Description
|---------------------------------------- |---------------- |------------------------- |--------------
|                                         |id                |serial                     |*PRIMARY KEY*
|                                         |label             |character varying(200)     |
|                                         |conn_id           |character varying(250)     |*NOT NULL*
|[public.users.id](#public.users)   |user_id           |integer                    |
|                                         |chart_type        |character varying(100)     |
|                                         |sql_layout        |character varying(50)      |
|                                         |sql               |text                       |
|                                         |y\_log\_scale       |boolean                    |
|                                         |show_datatable    |boolean                    |
|                                         |show_sql          |boolean                    |
|                                         |height            |integer                    |
|                                         |default_params    |character varying(5000)    |
|                                         |x\_is\_date         |boolean                    |
|                                         |iteration_no      |integer                    |
|                                         |last_modified     |timestamp with time zone   |

------------------------------------------------------------------------

### public.connection

**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name                 |Type                      |Description
|------ |------------------- |------------------------ |--------------
|        |id                   |serial                    |*PRIMARY KEY*
|        |conn_id              |character varying(250)    |
|        |conn_type            |character varying(500)    |
|        |host                 |character varying(500)    |
|        |schema               |character varying(500)    |
|        |login                |character varying(500)    |
|        |password             |character varying(500)    |
|        |port                 |integer                   |
|        |extra                |character varying(5000)   |
|        |is_encrypted         |boolean                   |
|        |is\_extra\_encrypted   |boolean                   |

------------------------------------------------------------------------

### public.dag

**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name                 |Type                       |Description
|------- |------------------- |------------------------- |--------------
|        |dag_id               |character varying(250)     |*PRIMARY KEY*
|        |is_paused            |boolean                    |
|        |is_subdag            |boolean                    |
|        |is_active            |boolean                    |
|        |last\_scheduler\_run   |timestamp with time zone   |
|        |last_pickled         |timestamp with time zone   |
|        |last_expired         |timestamp with time zone   |
|        |scheduler_lock       |boolean                    |
|        |pickle_id            |integer                    |
|        |fileloc              |character varying(2000)    |
|        |owners               |character varying(2000)    |
|        |description          |text                       |
|        |default_view         |character varying(25)      |
|        |schedule_interval    |text                       |
|        |root\_dag\_id          |character varying(250)     |

**Indexes:**

-   **idx\_root\_dag\_id** root\_dag\_id

------------------------------------------------------------------------

### public.dag_pickle
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name           |Type                       |Description
|------ |------------- |------------------------- |--------------
|        |id             |serial                     |*PRIMARY KEY*
|        |pickle         |bytea                      |
|        |created_dttm   |timestamp with time zone   |
|        |pickle_hash    |bigint                     |

------------------------------------------------------------------------

### public.dag_run
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name               |Type                       |Description
|------ |----------------- |------------------------- |----------------------
|        |id                 |serial                     |*PRIMARY KEY*
|        |dag_id             |character varying(250)     |*UNIQUE\#2 UNIQUE\#1*
|        |execution_date     |timestamp with time zone   |*UNIQUE\#2*
|        |state              |character varying(50)      |
|        |run_id             |character varying(250)     |*UNIQUE\#1*
|        |external_trigger   |boolean                    |
|        |conf               |bytea                      |
|        |end_date           |timestamp with time zone   |
|        |start_date         |timestamp with time zone   |

**Indexes:**

-   **dag\_id\_state** dag_id, state

------------------------------------------------------------------------

### public.import_error

**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name         |Type                       |Description
|------- |----------- |------------------------- |--------------
|        |id           |serial                     |*PRIMARY KEY*
|        |timestamp    |timestamp with time zone   |
|        |filename     |character varying(1024)    |
|        |stacktrace   |text                       |

------------------------------------------------------------------------

### public.job
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name               |Type                       |Description
|------ |----------------- |------------------------- |---------------
|        |id                 |serial                     |*PRIMARY KEY*
|        |dag_id             |character varying(250)     |
|        |state              |character varying(20)      |
|        |job_type           |character varying(30)      |
|        |start_date         |timestamp with time zone   |
|        |end_date           |timestamp with time zone   |
|        |latest_heartbeat   |timestamp with time zone   |
|        |executor_class     |character varying(500)     |
|        |hostname           |character varying(500)     |
|        |unixname           |character varying(1000)    |

**Indexes:**

-   **idx\_job\_state\_heartbeat** state, latest\_heartbeat
-   **job\_type\_heart** job\_type, latest\_heartbeat

------------------------------------------------------------------------

### public.known_event

**Table Structure:**

------------------------------------------------------------------------

|F-Key                                                          |Name                  |Type                          |Description
|------------------------------------------------------------- |-------------------- |---------------------------- |--------------
|                                                               |id                    |serial                        |*PRIMARY KEY*
|                                                               |label                 |character varying(200)        |
|                                                               |start_date            |timestamp without time zone   |
|                                                               |end_date              |timestamp without time zone   |
|[public.users.id](#public.users)                         |user_id               |integer                       |
|[public.known\_event\_type.id](#public.known-event-type)   |known\_event\_type\_id   |integer                       |
|                                                               |description           |text                          |

------------------------------------------------------------------------

### public.known\_event\_type

**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name              |Type                     |Description
|------ |---------------- |----------------------- |--------------
|        |id                |serial                   |*PRIMARY KEY*
|        |know\_event\_type   |character varying(200)   |

**Tables referencing this one via Foreign Key Constraints:**

-   [public.known_event](#public.known-event)

------------------------------------------------------------------------

### public.kube\_resource\_version

**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name               |Type                     |Description
|------ |----------------- |----------------------- |---------------------------
|        |one\_row\_id         |boolean                  |*PRIMARY KEY DEFAULT true*
|        |resource_version   |character varying(255)   |

**Constraints:**

|Name                                |Constraint
|---------------------------------- |--------------------
|kube\_resource\_version\_one\_row\_id    |CHECK (one\_row\_id)

------------------------------------------------------------------------

### public.kube\_worker\_uuid
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name          |Type                     |Description
|------ |------------ |----------------------- |---------------------------
|        |one\_row\_id    |boolean                  |*PRIMARY KEY DEFAULT true*
|        |worker_uuid   |character varying(255)   |

**Constraints:**

|Name                     |Constraint
|----------------------- |-------------------
|kube\_worker\_one\_row\_id   |CHECK (one\_row\_id)

------------------------------------------------------------------------

### public.log
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name             |Type                       |Description
|------ |--------------- |------------------------- |--------------
|        |id               |serial                     |*PRIMARY KEY*
|        |dttm             |timestamp with time zone   |
|        |dag_id           |character varying(250)     |
|        |task_id          |character varying(250)     |
|        |event            |character varying(30)      |
|        |execution_date   |timestamp with time zone   |
|        |owner            |character varying(500)     |
|        |extra            |text                       |

**Indexes:**

-   **idx\_log\_dag** dag\_id

------------------------------------------------------------------------

### public.serialized_dag

**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name           |Type                       |Description
|------ |------------- |------------------------- |--------------
|        |dag_id         |character varying(250)     |*PRIMARY KEY*
|        |fileloc        |character varying(2000)    |*NOT NULL*
|        |fileloc_hash   |integer                    |*NOT NULL*
|        |data           |json                       |*NOT NULL*
|        |last_updated   |timestamp with time zone   |*NOT NULL*

**Indexes:**

-   **idx\_filelo\_hash** fileloc_hash

------------------------------------------------------------------------

### public.sla_miss 

**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name                |Type                       |Description
|------ |------------------ |------------------------- |--------------
|        |task_id             |character varying(250)     |*PRIMARY KEY*
|        |dag_id              |character varying(250)     |*PRIMARY KEY*
|        |execution_date      |timestamp with time zone   |*PRIMARY KEY*
|        |email_sent          |boolean                    |
|        |timestamp           |timestamp with time zone   |
|        |description         |text                       |
|        |notification_sent   |boolean                    |

**Indexes:**

-   **sm_dag** dag_id

------------------------------------------------------------------------

### public.slot_pool
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name          |Type                    |Description
|------- |------------- |----------------------- |---------------
|        |id            |serial                  |*PRIMARY KEY*
|        |pool          |character varying(50)   |*UNIQUE*
|        |slots         |integer                 |
|        |description   |text                    |

------------------------------------------------------------------------

### public.task_fail
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name             |Type                       |Description
|------- |---------------- |-------------------------- |---------------
|        |id               |serial                     |*PRIMARY KEY*
|        |task_id          |character varying(250)     |*NOT NULL*
|        |dag_id           |character varying(250)     |*NOT NULL*
|        |execution_date   |timestamp with time zone   |*NOT NULL*
|        |start_date       |timestamp with time zone   |
|        |end_date         |timestamp with time zone   |
|        |duration         |integer                    |

**Indexes:**

-   **idx\_task\_fail\_dag\_task\_date** dag\_id, task\_id, execution\_date

------------------------------------------------------------------------

### public.task_instance 

**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name              |Type                       |Description
|------- |----------------- |-------------------------- |---------------------------
|        |task_id           |character varying(250)     |*PRIMARY KEY*
|        |dag_id            |character varying(250)     |*PRIMARY KEY*
|        |execution_date    |timestamp with time zone   |*PRIMARY KEY*
|        |start_date        |timestamp with time zone   |
|        |end_date          |timestamp with time zone   |
|        |duration          |double precision           |
|        |state             |character varying(20)      |
|        |try_number        |integer                    |
|        |hostname          |character varying(1000)    |
|        |unixname          |character varying(1000)    |
|        |job_id            |integer                    |
|        |pool              |character varying(50)      |*NOT NULL*
|        |queue             |character varying(256)     |
|        |priority_weight   |integer                    |
|        |operator          |character varying(1000)    |
|        |queued_dttm       |timestamp with time zone   |
|        |pid               |integer                    |
|        |max_tries         |integer                    |*DEFAULT \'-1\'::integer*
|        |executor_config   |bytea                      |

**Tables referencing this one via Foreign Key Constraints:**

-   [public.task_reschedule](#table-public.task-reschedule)

**Indexes:**

-   **ti\_dag\_date** dag\_id, execution\_date
-   **ti\_dag\_state** dag\_id, state
-   **ti\_job\_id** job\_id
-   **ti\_pool** pool, state, priority\_weight
-   **ti_state** state
-   **ti\_state\_lkp** dag\_id, task\_id, execution\_date, state

------------------------------------------------------------------------

### Table public.task_reschedule
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key                                                                   |Name              |Type                       |Description
|----------------------------------------------------------------------- |----------------- |-------------------------- |---------------
|                                                                        |id                |serial                     |*PRIMARY KEY*
|[public.task\_instance.task\_id\#1](#public.task-instance)          |task_id           |character varying(250)     |*NOT NULL*
|[public.task\_instance.dag\_id\#1](#public.task-instance)           |dag_id            |character varying(250)     |*NOT NULL*
|[public.task\_instance.execution\_date\#1](#public.task-instance)   |execution_date    |timestamp with time zone   |*NOT NULL*
|                                                                        |try_number        |integer                    |*NOT NULL*
|                                                                        |start_date        |timestamp with time zone   |*NOT NULL*
|                                                                        |end_date          |timestamp with time zone   |*NOT NULL*
|                                                                        |duration          |integer                    |*NOT NULL*
|                                                                        |reschedule_date   |timestamp with time zone   |*NOT NULL*

**Indexes:**

-   **idx\_task\_reschedule\_dag\_task\_date** dag\_id, task\_id, execution\_date

------------------------------------------------------------------------

### public.users

**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name        |Type                     |Description
|-------|-----------|------------------------|---------------
|        |id          |serial                   |*PRIMARY KEY*
|        |username    |character varying(250)   |*UNIQUE*
|        |email       |character varying(500)   |
|        |password    |character varying(255)   |
|        |superuser   |boolean                  |

**Tables referencing this one via Foreign Key Constraints:**

-   [public.chart](#public.chart)
-   [public.known_event](#public.known-event)

------------------------------------------------------------------------

### public.variable
 
**Table Structure:**

------------------------------------------------------------------------

|F-Key   |Name           |Type                     |Description
|------- |--------------|------------------------|---------------
|        |id             |serial                   |*PRIMARY KEY*
|        |key            |character varying(250)   |*UNIQUE*
|        |val            |text                     |
|        |is_encrypted   |boolean                  |

------------------------------------------------------------------------

### public.xcom

**Table Structure:**

------------------------------------------------------------------------

|F-Key  | Name            | Type                      |Description|
|------|----------------|--------------------------|---|
|       |id               |serial                     |*PRIMARY KEY*|
|       |key              |character varying(512)     ||
|       |value            |bytea                      ||
|       |timestamp        |timestamp with time zone   |*NOT NULL*|
|       |execution_date   |timestamp with time zone   |*NOT NULL*|
|       |task_id          |character varying(250)     |*NOT NULL*|
|       |dag_id           |character varying(250)     |*NOT NULL*|

**Indexes:**

-   **idx\_xcom\_dag\_task\_date** dag\_id, task\_id, execution\_date

------------------------------------------------------------------------
