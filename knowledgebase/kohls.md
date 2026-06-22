# Kohl's — Project Manager and Data Analyst, Supply Chain Transit Time Forecasting

> Index: [00_INDEX.md](00_INDEX.md) · Roles: Data Scientist, Data Analyst, ML/Analytics Eng, BA, Supply Chain/Operations Analyst, Technical Consultant
> One of Roy's strongest, most technically mature experiences.

## Metadata
- Organization: Kohl's
- Location: Los Angeles, California / client-facing DSU consulting context
- Role: Project Manager and Data Analyst
- Timeframe: June 2025 – March 2026
- Project type: Client-facing data science consulting project through UCLA Data Science Union
- Primary tools: SQL, Python, Google BigQuery, Google Vertex AI Workbench, pandas, NumPy, scikit-learn, XGBoost, visualization libraries
- Primary domains: Supply Chain Analytics, Logistics Forecasting, Data Science, Machine Learning, Business Analytics, Data Engineering, Consulting

## Experience Scope
Client-facing data science consulting project for Kohl's, forecasting shipment arrival times within the domestic portion of Kohl's supply chain — the movement of shipments after U.S. port arrival and before reaching distribution centers. Kohl's had extensive internal logistics data (operational timestamps, containers, purchase orders, ports of entry, carriers, shipment dimensions, transportation modes, product-level contents, DC destinations). Roy worked with ~5–8.5M+ logistics records in BigQuery and Vertex AI Workbench, contributing to data extraction, EDA, cleaning, validation, aggregation, feature engineering, modeling, evaluation, and client communication. The deliverable was a two-stage predictive modeling system simulating how predictions update as shipments move through milestones.

## Business Problem
Kohl's needed better visibility into inventory arrival timing. Delays and inaccurate estimates affect inventory allocation, DC labor planning, store replenishment, downstream transportation coordination, operational scheduling, supply chain visibility, and business planning. A historical-average approach was insufficient because timing depends on many interacting factors (route, port congestion, carrier performance, seasonality, shipment size, product complexity, DC workload, milestone delays). Goal: build ML models to learn these patterns and provide more accurate arrival-date estimates.

## Project Scope
Focused on domestic transportation after port arrival; evolved into a two-stage framework:
- **Stage 1:** Forecast movement through early domestic processing after port arrival.
- **Stage 2:** Forecast the final leg from intermediate processing/deconsolidation to DC arrival.

This mirrored real supply-chain flow better than a single static model and allowed predictions to update as new info became available.

## Responsibilities
- Querying large-scale logistics data from BigQuery.
- Performing EDA in Vertex AI Workbench.
- Understanding the business meaning of supply-chain event timestamps.
- Identifying missingness, impossible sequences, negative durations, inconsistent timelines.
- Validating which variables could be used at each prediction point without leakage.
- Engineering predictive features from raw shipment data.
- Designing shipment-level aggregation logic from granular product-level records.
- Participating in model development and evaluation.
- Helping design a two-stage modeling architecture.
- Supporting client-facing communication and project delivery.
- Translating model results into operational/business implications.

## Data Environment
Data in Google BigQuery, analyzed in Vertex AI Workbench. Millions of shipment records with fields for purchase orders, containers, ports of entry, DCs, carriers, transportation modes, dimensions, weights, product-level contents, and operational timestamps across milestones. Highly granular — multiple records often represented different products within the same shipment/container, creating duplication that required careful aggregation to set the correct unit of prediction.

## Data Cleaning and Validation
Issues identified and addressed: large missingness in key arrival timestamp fields; records where destination arrival appeared before port arrival; negative transit durations; impossible/inconsistent event sequences; timestamp sequencing issues; redundancy from product-level granularity; leakage risk from variables unknown at prediction time; time-zone/operational timeline consistency. A core challenge was mapping real-world supply-chain processes onto the data — understanding not just what each field represented but *when* it would become available in a live environment (so the model doesn't use post-target information).

## Feature Engineering Contributions
- **Route-based:** recurring transportation lanes between ports and destinations, capturing historically different transit patterns.
- **Carrier performance:** historical reliability/transit-behavior metrics per carrier.
- **Seasonality:** month, quarter, day of week, end-of-month indicators (demand patterns, holiday surges, inventory pushes, recurring bottlenecks).
- **Operational workload / congestion:** recent shipment volume around ports and DCs as proxies for congestion and facility utilization.
- **Shipment complexity:** aggregated size, weight, volume, product diversity, density, operational complexity.
- **Timing-based operational:** time spent in different logistics stages to identify bottlenecks and predict downstream delays.

## Aggregation & Modeling Dataset Construction
A key contribution: redesigning dataset structure around the correct prediction unit. Original data was granular with multiple product-level rows per shipment; modeling directly would create duplication and distort the target. Roy helped develop aggregation logic consolidating records into shipment-level observations using purchase order, destination facility, and container-level identifiers. Timestamps and categoricals aggregated by business meaning; quantitative measures summed/consolidated. This reduced redundancy and produced a cleaner modeling table.

## Modeling Approach
Tree-based regression (final direction emphasized XGBoost) to handle nonlinear relationships and interactions across route, carrier, seasonal, and complexity features. Used **time-aware validation** rather than random splits — a critical decision, since random splits could let future shipment patterns leak into training and inflate performance. Time-aware validation simulated real deployment: historical shipments predict future shipments.

## Production-Style Prediction Workflow
Because shipment data arrives incrementally, the team designed a workflow simulating live data ingestion and model execution; as new milestone data became available, the system generated updated predictions. This made the deliverable realistic and operationally relevant rather than a static historical model.

## Results & Business Impact
Final deliverable: two predictive models covering the domestic flow; a two-stage framework aligned with real milestones; supporting data pipelines and modeling datasets; production-oriented logic for updating predictions; documentation and client-facing explanation. **Model achieved ~83–85% improvement in MAE vs. baseline.** Business impact: improved arrival visibility, better inventory planning, better DC scheduling, better transportation visibility, more accurate labor planning, more reliable downstream operations, and a scalable approach for future production integration.

## Skills Demonstrated
**Technical:** Python, SQL, BigQuery, Vertex AI Workbench, pandas, NumPy, scikit-learn, XGBoost, feature engineering, time-aware validation, data leakage prevention, large-scale data cleaning, shipment-level aggregation, supply chain analytics, forecasting, regression modeling, data pipeline design, EDA, model evaluation, MAE improvement tracking.
**Business / soft:** Client-facing communication, consulting project management, business problem translation, supply chain domain learning, ambiguous data interpretation, technical storytelling, cross-functional teamwork, stakeholder presentation, operational thinking, project ownership.

## Applicable Role Families
Data Scientist, Data Analyst, Business Analyst, Product Analyst, Machine Learning Engineer, Analytics Engineer, BI Analyst, Supply Chain Analyst, Operations Analyst, Technical Consultant, Product Manager (analytics-oriented).

## Interview Story Angles & STAR Stories
1. Discovering data leakage risk and redesigning validation around time-aware splits.
2. Turning granular product-level logistics data into shipment-level modeling data.
3. Identifying impossible shipment timelines and building data quality filters.
4. Engineering route, carrier, congestion, seasonality, and complexity features.
5. Designing a two-stage model mirroring real supply-chain operations.
6. Simulating live prediction updates using historical milestone data.
7. Communicating technical results to client stakeholders in operational terms.

**STAR — Data Leakage & Time-Aware Validation:** S: Kohl's needed accurate forecasts from historical logistics data. T: Build models that realistically predict future shipments without using info unavailable at prediction time. A: Analyzed timestamp fields, identified leakage risks, implemented time-aware validation instead of random splits. R: More realistic evaluation and a production-oriented forecasting framework. *Best for: DS, DA, ML, BA, supply chain analytics.*

**STAR — Shipment-Level Aggregation:** S: Raw data existed at granular product-record level. T: Create a shipment-level dataset. A: Designed aggregation logic around PO, container, and destination while preserving meaningful attributes. R: Reduced redundancy, cleaner modeling dataset. *Best for: DA, DS, analytics/data engineering.*

**STAR — Two-Stage Forecasting Framework:** S: Shipment info arrived incrementally across milestones. T: Build a forecasting system reflecting real operations. A: Contributed to a two-stage model updating predictions as new data arrived. R: ~83–85% MAE improvement; better planning support. *Best for: DS, PM, BA, product analytics.*

## Update Later
Add: exact final model names and stage definitions; baseline MAE and final MAE; feature importance findings; deployment/prototype details; client feedback; final presentation structure.
