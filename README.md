# AWS E-commerce Data Platform

> **AWS-native Data Engineering platform implementing a production-oriented e-commerce analytics architecture.**

This project demonstrates an end-to-end data platform built with **Amazon S3, AWS Glue, Apache Airflow (MWAA), Athena, and Amazon Redshift**, following a **Medallion Architecture** and modern Data Engineering practices.

The architecture is designed to demonstrate how a scalable data platform can be built using managed AWS services, from data ingestion and transformation to analytics and Data Warehousing.

---

## 🏗️ Architecture

```text
                         ┌─────────────────┐
                         │     GitHub      │
                         │ Source Control  │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ GitHub Actions  │
                         │   CI/CD + OIDC  │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │      MWAA       │
                         │ Apache Airflow  │
                         │  Orchestration  │
                         └────────┬────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
              Glue Bronze   Glue Silver    Glue Gold
                    │             │             │
                    └─────────────┼─────────────┘
                                  ▼
                         ┌─────────────────┐
                         │       S3        │
                         │   Data Lake     │
                         │    Parquet      │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ Glue Data       │
                         │    Catalog      │
                         └────────┬────────┘
                                  │
                         ┌────────┴────────┐
                         ▼                 ▼
                    ┌─────────┐      ┌───────────┐
                    │ Athena  │      │ Redshift  │
                    │ SQL on  │      │ Data      │
                    │  Lake   │      │ Warehouse │
                    └─────────┘      └───────────┘
```

## 🛠️ Tech Stack

| Category            | Technology                       |
| ------------------- | -------------------------------- |
| Cloud               | **AWS**                          |
| Data Lake           | **Amazon S3**                    |
| Processing          | **AWS Glue / PySpark**           |
| Orchestration       | **Amazon MWAA / Apache Airflow** |
| Metadata            | **AWS Glue Data Catalog**        |
| Data Lake Analytics | **Amazon Athena**                |
| Data Warehouse      | **Amazon Redshift Serverless**   |
| Storage Format      | **Apache Parquet**               |
| CI/CD               | **GitHub Actions**               |
| Authentication      | **AWS IAM + OIDC**               |
| Secrets             | **AWS Secrets Manager**          |
| Monitoring          | **Amazon CloudWatch**            |
| Testing             | **pytest**                       |

---

## 📊 Data Architecture

The platform follows a **Medallion Architecture**:

```text
Raw → Bronze → Silver → Gold
```

### Raw

Landing zone containing source data with minimal transformation.

### Bronze

Initial processing layer where schema handling and ingestion metadata are applied.

### Silver

Business-ready datasets with:

* Data type standardization
* Deduplication
* Null handling
* Business validation
* Data quality checks

### Gold

Analytics-ready datasets:

```text
gold/
├── daily_report/
├── monthly_report/
└── churn_report/
```

These datasets can be queried directly through **Athena** or loaded into **Redshift**.

---

## 🔄 Pipeline

```text
Source Data
     │
     ▼
   S3 Raw
     │
     ▼
 Glue Bronze
     │
     ▼
  S3 Bronze
     │
     ▼
 Glue Silver
     │
     ▼
  S3 Silver
     │
     ▼
  Glue Gold
     │
     ▼
   S3 Gold
    /    \
   ▼      ▼
Athena  Redshift
          │
          ▼
      Star Schema
```

**Airflow/MWAA orchestrates the workflow while Glue performs the actual data processing.**

This separation keeps orchestration and processing responsibilities independent.

---

## 🏢 Data Warehouse

Redshift uses a dimensional **Star Schema**:

```text
                    dim_customer
                         │
                         ▼
dim_date ───────► fact_transactions ◄────── dim_product
```

### Tables

```text
analytics
├── dim_customer
├── dim_product
├── dim_date
└── fact_transactions
```

The Gold layer provides the source data for the analytical warehouse.

---

## 🔎 Athena + Redshift

The project intentionally uses both services for different workloads:

| Athena               | Redshift                 |
| -------------------- | ------------------------ |
| Data Lake analytics  | Data Warehouse           |
| SQL directly over S3 | Dimensional model        |
| Ad-hoc analysis      | BI workloads             |
| Data exploration     | Structured analytics     |
| Parquet on S3        | Modeled warehouse tables |

They are **complementary rather than redundant**.

---

## 🧪 Data Quality

Data quality validation is integrated into the Silver processing layer.

Examples:

* Required fields validation
* Primary key uniqueness
* Schema validation
* Transaction amount validation
* Business rule validation

Pipeline failures propagate through Airflow:

```text
Glue Job FAILED
      ↓
Airflow Task FAILED
      ↓
Downstream tasks blocked
```

This prevents invalid data from continuing through the pipeline.

---

## 🔐 Security & DevOps

The project follows AWS security practices including:

* IAM least-privilege access
* AWS Secrets Manager
* CloudWatch monitoring
* GitHub Actions CI/CD
* GitHub → AWS authentication through OIDC
* No long-lived AWS credentials stored in GitHub

CI/CD flow:

```text
Git Push
   │
   ▼
GitHub Actions
   │
   ├── Tests
   ├── Linting
   ├── Validation
   └── OIDC Authentication
           │
           ▼
          AWS
```

---

## 📁 Repository Structure

```text
aws_ecommerce_data_platform/
│
├── dags/
│   └── ecommerce_pipeline.py
│
├── glue/
│   ├── common/
│   ├── bronze/
│   ├── silver/
│   └── gold/
│
├── redshift/
│   ├── ddl/
│   └── load/
│
├── athena/
├── tests/
├── infrastructure/
│
├── .github/
│   └── workflows/
│
├── docs/
│
├── requirements.txt
└── README.md
```

---

## 💡 Key Engineering Decisions

### Managed Services

The architecture prioritizes managed AWS services:

```text
MWAA       → Managed Airflow
Glue       → Managed Spark
S3         → Data Lake
Athena     → Serverless SQL
Redshift   → Serverless Data Warehouse
```

This reduces infrastructure management and keeps the platform focused on Data Engineering rather than server administration.

### Separation of Responsibilities

```text
MWAA / Airflow → Orchestration
Glue           → Processing
S3             → Storage
Glue Catalog   → Metadata
Athena         → Data Lake Analytics
Redshift       → Data Warehouse
```

This modular design makes the platform easier to maintain and evolve.

---

## 🔄 Databricks → AWS Mapping

The project also demonstrates how common Data Engineering concepts can be implemented across different cloud platforms:

| Databricks      | AWS                     |
| --------------- | ----------------------- |
| Spark           | Glue / PySpark          |
| Delta Lake      | S3 / Parquet            |
| Unity Catalog   | Glue Data Catalog       |
| Workflows       | MWAA / Airflow          |
| Databricks SQL  | Athena                  |
| Databricks Jobs | Glue Jobs               |
| Lakehouse       | S3 Data Lake + Redshift |

This demonstrates the ability to work with **data platform concepts rather than being tied to a single technology stack**.

---

## 🎯 What This Project Demonstrates

**Data Engineering**

* Data Lake architecture
* Medallion Architecture
* ETL / ELT
* PySpark
* Apache Airflow
* Data Quality
* Data Modeling
* Dimensional Modeling
* SQL Analytics

**AWS**

* Amazon S3
* AWS Glue
* Glue Data Catalog
* Amazon MWAA
* Amazon Athena
* Amazon Redshift
* IAM
* Secrets Manager
* CloudWatch

**DevOps**

* GitHub Actions
* CI/CD
* OIDC authentication
* Automated testing
* Observability

---

## 🚧 Project Status

**In development**

Planned implementation:

* [ ] S3 Data Lake
* [ ] Glue Bronze/Silver/Gold pipelines
* [ ] Glue Data Catalog
* [ ] Athena analytics
* [ ] Redshift Star Schema
* [ ] MWAA orchestration
* [ ] Data quality validation
* [ ] Automated tests
* [ ] GitHub Actions CI/CD
* [ ] CloudWatch observability
* [ ] Architecture and data lineage documentation

---

## 👨‍💻 Portfolio

This project was designed to demonstrate practical knowledge of **AWS Data Engineering, distributed processing, orchestration, Data Lakes, Data Warehousing, analytics, security, and CI/CD** through an end-to-end cloud architecture.
