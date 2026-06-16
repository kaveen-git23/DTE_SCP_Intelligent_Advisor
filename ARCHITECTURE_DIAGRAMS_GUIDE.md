# Architecture Diagrams Guide
## Intelligent Advisor Application - Diagram Specifications

**Owner:** Sai Nikhars  
**Document Version:** 1.0  
**Last Updated:** 2026-06-09  

---

## Overview

This guide provides specifications for creating architecture diagrams for the three proposed approaches:
1. **Fully Databricks Approach**
2. **Hybrid Approach**
3. **Fully Azure Approach**

---

## Diagram Requirements

### General Guidelines
- **Tool Recommendations:** Draw.io, Lucidchart, Microsoft Visio, or Azure Architecture Designer
- **Format:** PNG or SVG (high resolution, minimum 1920x1080)
- **Style:** Use official Azure and Databricks icons/logos
- **Color Coding:**
  - 🔵 Blue: Azure services
  - 🟠 Orange: Databricks components
  - 🟢 Green: Data flow
  - 🔴 Red: External integrations (Maximo SAP)
  - ⚪ Gray: Storage components

### Required Diagram Types (for each approach)
1. **High-Level Architecture Diagram**
2. **Data Flow Diagram**
3. **Component Interaction Diagram**
4. **Deployment Architecture Diagram**

---

## 1. Fully Databricks Approach

### 1.1 High-Level Architecture Diagram

**Components to Include:**
```
┌───────────────────────────────────────────────────┐
│                  MAXIMO SAP                           │
│              (External System)                        │
└───────────────────────┬──────────────────────────┘
                         │
                         │ REST API / JDBC
                         │
                         │
┌────────────────────┴──────────────────────────┐
│                                                       │
│              AZURE APP PLATFORM                      │
│          (Hosting Environment)                       │
│                                                       │
│   ┌───────────────────────────────────────┐   │
│   │                                         │   │
│   │        DATABRICKS WORKSPACE            │   │
│   │                                         │   │
│   │   ┌────────────────────────────┐   │   │
│   │   │  Databricks SQL Endpoints  │   │   │
│   │   │    (API Gateway)          │   │   │
│   │   └────────────┬───────────────┘   │   │
│   │                │                   │   │
│   │   ┌────────────┴───────────────┐   │   │
│   │   │   Databricks Clusters      │   │   │
│   │   │   - ETL Processing         │   │   │
│   │   │   - ML Runtime             │   │   │
│   │   │   - Embedding Generation   │   │   │
│   │   └────────────┬───────────────┘   │   │
│   │                │                   │   │
│   │   ┌────────────┴───────────────┐   │   │
│   │   │  Databricks Vector Search │   │   │
│   │   │   (Vector Database)        │   │   │
│   │   └────────────┬───────────────┘   │   │
│   │                │                   │   │
│   │   ┌────────────┴───────────────┐   │   │
│   │   │    Delta Lake Storage      │   │   │
│   │   │   - Raw Data               │   │   │
│   │   │   - Processed Data         │   │   │
│   │   │   - Vector Embeddings      │   │   │
│   │   └────────────────────────────┘   │   │
│   │                                         │   │
│   └───────────────────────────────────────┘   │
│                                                       │
└───────────────────────────────────────────────────┘
```

**Key Elements:**
- Maximo SAP (external system)
- Azure App Platform (hosting)
- Databricks Workspace (core platform)
- Databricks SQL Endpoints (API layer)
- Databricks Clusters (compute)
- Databricks Vector Search (vector database)
- Delta Lake Storage (data storage)

**Annotations:**
- Show data flow with arrows
- Label API protocols (REST, JDBC)
- Indicate security boundaries
- Mark auto-scaling components

### 1.2 Data Flow Diagram

**Flow Sequence:**
```
1. Vendor Data Ingestion
   Maximo SAP → Databricks SQL Endpoint → Delta Lake (Bronze)

2. Data Processing & Transformation
   Delta Lake (Bronze) → Databricks Cluster (ETL) → Delta Lake (Silver)

3. Embedding Generation
   Delta Lake (Silver) → Databricks ML Runtime → Vector Embeddings → Delta Lake (Gold)

4. Vector Index Creation
   Delta Lake (Gold) → Databricks Vector Search → Vector Index

5. Query Processing
   Maximo SAP → Databricks SQL Endpoint → Databricks Vector Search → Results → Maximo SAP
```

**Data Layers (Medallion Architecture):**
- **Bronze:** Raw vendor data
- **Silver:** Cleaned and validated data
- **Gold:** Enriched data with embeddings

### 1.3 Component Interaction Diagram

**Interactions to Show:**
- API request/response flow
- Authentication and authorization (Azure AD)
- Monitoring and logging (Databricks Monitoring)
- Error handling and retry logic
- Caching mechanisms

### 1.4 Deployment Architecture

**Infrastructure Components:**
- Azure Virtual Network
- Databricks Workspace (Premium Tier)
- Azure Key Vault (secrets management)
- Azure Monitor (logging and metrics)
- Azure Private Link (secure connectivity)

---

## 2. Hybrid Approach

### 2.1 High-Level Architecture Diagram

**Components to Include:**
```
┌───────────────────────────────────────────────────┐
│                  MAXIMO SAP                           │
│              (External System)                        │
└───────────────────────┬───────────────────────────┘
                         │
                         │ HTTPS / REST API
                         │
┌────────────────────────┴─────────────────────────┐
│                                                       │
│              AZURE APP PLATFORM                      │
│                                                       │
│   ┌───────────────────────────────────────┐   │
│   │      Azure Functions / App Service      │   │
│   │           (API Gateway)                │   │
│   └────────────────┬───────────────────────┘   │
│                    │                               │
│        ┌───────────┴────────────┐                │
│        │                        │                │
│        │                        │                │
│   ┌────┴──────────────────────┴───────────┐   │
│   │  DATABRICKS WORKSPACE (ETL)          │   │
│   │                                      │   │
│   │  ┌────────────────────────────┐  │   │
│   │  │  Databricks Clusters      │  │   │
│   │  │  - ETL Processing         │  │   │
│   │  │  - Data Transformation    │  │   │
│   │  └────────────┬───────────────┘  │   │
│   │               │                   │   │
│   │  ┌────────────┴───────────────┐  │   │
│   │  │  Delta Lake (Staging)     │  │   │
│   │  └────────────────────────────┘  │   │
│   │                                      │   │
│   └─────────────────┬──────────────────┘   │
│                    │                               │
│   ┌────────────────┴───────────────────────┐   │
│   │   AZURE VECTOR DATABASE               │   │
│   │                                       │   │
│   │   Option A: Cosmos DB (MongoDB vCore) │   │
│   │   Option B: PostgreSQL + pgvector     │   │
│   │                                       │   │
│   └───────────────────────────────────────┘   │
│                                                       │
│   ┌───────────────────────────────────────┐   │
│   │   Azure OpenAI Service (Optional)     │   │
│   │   - Embedding Generation              │   │
│   └───────────────────────────────────────┘   │
│                                                       │
└───────────────────────────────────────────────────┘
```

**Key Elements:**
- Maximo SAP (external system)
- Azure Functions/App Service (API layer)
- Databricks Workspace (ETL only)
- Delta Lake (staging storage)
- Azure Vector Database (Cosmos DB or PostgreSQL)
- Azure OpenAI Service (optional for embeddings)

**Annotations:**
- Show data synchronization flow
- Highlight integration points
- Mark cost optimization areas

### 2.2 Data Flow Diagram

**Flow Sequence:**
```
1. Vendor Data Ingestion
   Maximo SAP → Azure Functions → Databricks → Delta Lake

2. Data Processing & Transformation
   Delta Lake → Databricks Cluster (ETL) → Processed Data

3. Embedding Generation (Option A: Databricks)
   Processed Data → Databricks ML Runtime → Vector Embeddings

   Embedding Generation (Option B: Azure OpenAI)
   Processed Data → Azure Functions → Azure OpenAI → Vector Embeddings

4. Vector Storage
   Vector Embeddings → Azure Data Factory → Azure Vector DB

5. Query Processing
   Maximo SAP → Azure Functions → Azure Vector DB → Results → Maximo SAP
```

### 2.3 Component Interaction Diagram

**Interactions to Show:**
- Databricks to Azure Vector DB synchronization
- Azure Data Factory orchestration
- API authentication flow (Azure AD)
- Error handling and retry mechanisms
- Monitoring across platforms

### 2.4 Deployment Architecture

**Infrastructure Components:**
- Azure Virtual Network (VNet)
- Azure Functions (consumption or premium plan)
- Databricks Workspace (Standard Tier)
- Azure Cosmos DB or PostgreSQL (with private endpoint)
- Azure Data Factory (orchestration)
- Azure Key Vault (secrets)
- Azure Monitor (unified monitoring)

---

## 3. Fully Azure Approach

### 3.1 High-Level Architecture Diagram

**Components to Include:**
```
┌───────────────────────────────────────────────────┐
│                  MAXIMO SAP                           │
│              (External System)                        │
└───────────────────────┬───────────────────────────┘
                         │
                         │ HTTPS / REST API
                         │
┌────────────────────────┴─────────────────────────┐
│                                                       │
│              AZURE APP PLATFORM                      │
│                                                       │
│   ┌───────────────────────────────────────┐   │
│   │   Azure API Management (Optional)     │   │
│   │        (API Gateway & Security)        │   │
│   └────────────────┬───────────────────────┘   │
│                    │                               │
│   ┌────────────────┴───────────────────────┐   │
│   │  Azure Kubernetes Service (AKS)      │   │
│   │                                       │   │
│   │  ┌────────────────────────────┐  │   │
│   │  │  API Service Pods         │  │   │
│   │  │  - Query Handler          │  │   │
│   │  │  - Embedding Service      │  │   │
│   │  └────────────────────────────┘  │   │
│   │                                       │   │
│   └───────────────────────────────────────┘   │
│                                                       │
│   ┌───────────────────────────────────────┐   │
│   │   Azure Synapse Analytics / ADF       │   │
│   │   - Data Pipelines                    │   │
│   │   - ETL Orchestration                 │   │
│   └────────────────┬───────────────────────┘   │
│                    │                               │
│   ┌────────────────┴───────────────────────┐   │
│   │   Azure Data Lake Storage Gen2        │   │
│   │   - Raw Data                          │   │
│   │   - Processed Data                    │   │
│   └───────────────────────────────────────┘   │
│                                                       │
│   ┌───────────────────────────────────────┐   │
│   │   Azure OpenAI Service                │   │
│   │   - text-embedding-3-small/large      │   │
│   └───────────────────────────────────────┘   │
│                                                       │
│   ┌───────────────────────────────────────┐   │
│   │   AZURE VECTOR DATABASE               │   │
│   │                                       │   │
│   │   Option A: Cosmos DB (MongoDB vCore) │   │
│   │   Option B: PostgreSQL + pgvector     │   │
│   │   Option C: Azure AI Search           │   │
│   │                                       │   │
│   └───────────────────────────────────────┘   │
│                                                       │
│   ┌───────────────────────────────────────┐   │
│   │   Azure Container Registry            │   │
│   │   - Docker Images                     │   │
│   └───────────────────────────────────────┘   │
│                                                       │
└───────────────────────────────────────────────────┘
```

**Key Elements:**
- Maximo SAP (external system)
- Azure API Management (optional gateway)
- Azure Kubernetes Service (containerized services)
- Azure Synapse Analytics / Data Factory (ETL)
- Azure Data Lake Storage Gen2 (data storage)
- Azure OpenAI Service (embeddings)
- Azure Vector Database (Cosmos DB / PostgreSQL / AI Search)
- Azure Container Registry (container images)

**Annotations:**
- Show CI/CD pipeline integration
- Highlight auto-scaling components
- Mark security zones (VNet, NSG)

### 3.2 Data Flow Diagram

**Flow Sequence:**
```
1. Vendor Data Ingestion
   Maximo SAP → Azure API Management → AKS API Service → Azure Data Lake Gen2

2. Data Processing & Transformation
   Azure Data Lake Gen2 → Azure Synapse / ADF → Processed Data → Azure Data Lake Gen2

3. Embedding Generation
   Processed Data → AKS Embedding Service → Azure OpenAI → Vector Embeddings

4. Vector Storage
   Vector Embeddings → Azure Vector DB

5. Query Processing
   Maximo SAP → Azure APIM → AKS Query Handler → Azure Vector DB → Results → Maximo SAP
```

### 3.3 Component Interaction Diagram

**Interactions to Show:**
- AKS pod communication
- Azure OpenAI API calls
- Azure Data Factory pipeline triggers
- Azure Service Bus messaging (if used)
- Azure Monitor telemetry collection
- Azure AD authentication flow

### 3.4 Deployment Architecture

**Infrastructure Components:**
- Azure Virtual Network (VNet with subnets)
- Azure Kubernetes Service (with node pools)
- Azure Container Registry (geo-replication)
- Azure Data Lake Storage Gen2 (with lifecycle policies)
- Azure Synapse Analytics / Data Factory
- Azure OpenAI Service (with rate limiting)
- Azure Vector Database (with private endpoint)
- Azure API Management (with policies)
- Azure Key Vault (with RBAC)
- Azure Monitor + Application Insights
- Azure DevOps (CI/CD pipelines)
- Azure Load Balancer (for AKS ingress)

**CI/CD Pipeline:**
```
Code Commit → Azure DevOps → Build Docker Image → Push to ACR → Deploy to AKS (Blue-Green)
```

---

## Diagram Deliverables Checklist

### For Each Approach:
- [ ] High-Level Architecture Diagram (PNG/SVG)
- [ ] Data Flow Diagram (PNG/SVG)
- [ ] Component Interaction Diagram (PNG/SVG)
- [ ] Deployment Architecture Diagram (PNG/SVG)
- [ ] Diagram source files (Draw.io, Visio, etc.)
- [ ] Architecture Decision Record (ADR) document

### Additional Diagrams (Optional):
- [ ] Security Architecture Diagram
- [ ] Network Topology Diagram
- [ ] Disaster Recovery Architecture
- [ ] Monitoring and Observability Diagram
- [ ] Cost Breakdown Diagram (visual representation)

---

## Tools and Resources

### Recommended Diagramming Tools:
1. **Draw.io (diagrams.net)** - Free, web-based, supports Azure/Databricks icons
2. **Lucidchart** - Professional, collaborative, extensive shape libraries
3. **Microsoft Visio** - Enterprise standard, Azure stencils available
4. **Azure Architecture Designer** - Azure-native, limited to Azure services
5. **Mermaid** - Code-based diagrams, good for version control

### Icon Libraries:
- **Azure Architecture Icons:** https://learn.microsoft.com/en-us/azure/architecture/icons/
- **Databricks Icons:** Available in Databricks documentation
- **General Cloud Icons:** AWS Architecture Icons (for generic cloud concepts)

### Reference Architectures:
- **Azure AI Search with RAG:** https://learn.microsoft.com/en-us/azure/search/
- **Databricks Vector Search:** https://docs.databricks.com/en/generative-ai/vector-search.html
- **Azure OpenAI Reference Architectures:** https://learn.microsoft.com/en-us/azure/architecture/ai-ml/

---

## Next Steps

1. **Week 1:** Create high-level architecture diagrams for all three approaches
2. **Week 2:** Develop detailed data flow diagrams
3. **Week 3:** Complete component interaction diagrams
4. **Week 4:** Finalize deployment architecture diagrams
5. **Week 5:** Review and refine diagrams with stakeholders
6. **Week 6:** Incorporate feedback and create final versions

---

**Document Owner:** Sai Nikhars  
**Reviewers:** Architecture Team, Technical Leadership  
**Approval Required:** Yes  
**Target Completion Date:** [To be determined]  
