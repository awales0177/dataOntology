
---

## Core Principles

- **Datasets are raw and ungoverned**
- **Priority belongs to the Dataset**
- **The Dataset Priority Queue is the canonical intake list**
- **Runs produce Data Products (not Datasets)**
- **Data Products are governed by Data Contracts**
- **Transformations explain how data changes**
- **Products encapsulate schemas and tables**
- **Validation is defined on the Product and evaluated per Run**
- **Jobs orchestrate execution; Tasks perform work**

---

# Data Ontology Overview

This repository defines a **practical, opinionated data ontology** that models how raw datasets move through **discovery, prioritization, transformation, governance, execution, and delivery** as governed data assets.

The ontology is intentionally designed for **real-world data platforms**, not abstract academic modeling.

---

## What This Ontology Is (and Is Not)

### ✅ This is
- A **lifecycle-oriented** data ontology
- Explicit separation of **raw data**, **governed value**, and **execution**
- Designed to map cleanly to:
  - Airflow / Dagster / Prefect
  - Spark / dbt
  - Dremio / Iceberg / Lakehouse stacks
  - Metadata catalogs and data governance systems

### ❌ This is not
- A generic “everything is a dataset” model
- A UI-specific schema
- A replacement for physical storage schemas

---

## Core Separations (Intentional Design)

The model explicitly separates:

| Concept | Responsibility |
|------|---------------|
| **Dataset** | Raw, ungoverned input data |
| **Priority Queue** | Intake ordering & decision-making |
| **Data Product** | Governed, consumable output |
| **Pipeline / Job / Task** | Execution mechanics |
| **Run** | Execution instance |
| **Contract & Validation** | Data Quality |

This separation avoids the most common failure mode of data catalogs:  
**blending execution metadata with business value.**

---


## Ontology Diagram

```mermaid
classDiagram
direction LR

class DatasetPriorityQueue {
  +name: string
  +policy: string
}

class Dataset {
  +name: string
  +source: string
  +owner: string
  +priority: int
  +queueStatus: string
}

class DataProfile {
  +profiledAt: datetime
  +rowCount: int
  +nullRate: decimal
  +freshnessSeconds: int
}

class DataAnalytics {
  +analysisType: string
  +metrics: list
  +insights: string
}

class DataDomain {
  +name: string
  +steward: string
}

class DataContract {
  +name: string
  +version: string
  +owner: string
}

class DataPlatform {
  +name: string
  +type: string
  +environment: string
}

class DataPipeline {
  +name: string
  +version: string
  +owner: string
}

class Job {
  +name: string
  +schedule: string
}

class Task {
  +name: string
  +type: string
  +order: int
  +logicRef: string
}

class DataTransformation {
  +name: string
  +type: string
  +logicRef: string
}

class Run {
  +runId: string
  +status: string
  +startedAt: datetime
  +endedAt: datetime
}

class DataProduct {
  +productType: string
  +releaseTag: string
}

class Table {
  +name: string
  +storageRef: string
}

class DataValidation {
  +name: string
  +scope: string
  +severity: string
  +logic: string
  +status: string
  +checkedAt: datetime
  +details: string
}

class DataLineage {
  +sourceRef: string
  +targetRef: string
}

DatasetPriorityQueue "1" --> "0..*" Dataset : orders

Dataset "1" --> "0..*" DataProfile : hasProfile
Dataset "1" --> "0..*" DataAnalytics : analyzedBy
DataAnalytics "0..*" --> "0..1" DataDomain : informsDomain
Dataset "1" --> "0..1" DataDomain : inDomain

DataProduct "1" --> "1" DataContract : governedBy
DataContract "1" --> "1" DataDomain : appliesToDomain

DataContract "1" --> "1" DataPlatform : targetPlatform
DataPipeline "0..*" --> "1" DataPlatform : runsOn
DataPipeline "1" --> "1..*" Job : hasJob
Job "1" --> "1..*" Task : hasTask

Task "1" --> "1..*" DataTransformation : performs

Job "1" --> "0..*" Run : hasRun
Run "1" --> "1..*" Task : executes

DataPipeline "0..*" --> "0..*" Dataset : consumes
Run "1..*" --> "0..*" DataProduct : produces

DataProduct "1" --> "1..*" Table : hasSchema

DataProduct "1" --> "0..*" DataValidation : defines
Run "0..*" --> "0..*" DataValidation : evaluates

Dataset "0..*" --> "0..*" DataLineage : upstream
DataTransformation "1" --> "0..*" DataLineage : creates
Table "1" --> "0..*" DataLineage : downstream
DataProduct "1" --> "0..*" DataLineage : materializedFrom
Run "0..*" --> "0..*" DataLineage : emmits

