# The Detailed Job Posting Process

Date: May 9, 2025

- Findings
    
    So basically we have four main jobs here
    
    1. Work Job (UT Job)
    2. Ferment Job
    3. BBT Job
    4. Keg Job
    
    There are many different brands here by the way.
    
    Every job has two orders by the way, Planned order number & Process order number
    
    1. UT Job = 6 Brews (4400 HL)
    2. Basically I need to find the planned order and process order and link it to the RM To WIP To FG
    3. The values of Materials are picked from the excel sheet maintained by the production team, which is maintained every 24 hours not shift wise
        1. These values are taken from brew cards maintained matched with Raw Material Value of BOM
    4. 002 - RM Consumption
    5. 003 - UT (Unit Tank) - WIP
    6. Batch Number is UT → There are 21 UT numbers here
    7. The Codes for Material are related to jobs like WORT, FRMT, BBTX

---

# Technical Report: Production Job Tracking & Material Flow Analysis

**Prepared by: Senior Business Intelligence Analyst**

**Date: 09-05-2025**

---

## **1. Executive Summary**

This report outlines a structured approach to link **Planned/Process Orders** across four production jobs (UT, Ferment, BBT, Keg) to **RM → WIP → FG** material flows, leveraging batch-level data from Excel sheets maintained by Production team. Key challenges include reconciling daily-updated manual data with real-time process orders, aligning material codes (WORT, FRMT, BBTX) to BOMs, and ensuring traceability across 21 UT batches. Recommendations focus on automation, validation, and hierarchical data modeling.

---

## **2. Data Sources & System Context**

### **2.1 Data Inputs**

| Source | Description | Update Frequency | Key Fields |
| --- | --- | --- | --- |
| Production Excel Sheets | Manual entries from brew cards (aligned to BOMs) | 24 hours | Materials Consumed. (Matched with Brew Cards) |
| ERP/MES System | Transactional records (e.g., SAP, Oracle) | Real-time | Process Order #, Batch Status (UT → Ferment → BBT → Keg), Material Movements (002-RM, 003-WIP) |

### **2.2 Key Constraints**

- **Data Latency**: Excel data lags by up to 24 hours or more vs. real-time process orders (**for instance they are entering a job for 02-05-2025 on 09-05-2025**)
- **Manual Entry Risks**: Potential mismatches between brew cards and BOMs (e.g., RM over/under-consumption).
- **Batch Complexity**: 21 UT batch numbers are there with multi-stage dependencies (6 brews → 4400 HL total).

---

## **3. Technical Architecture**

### **3.1 Data Flow Diagram**

```
[Excel Sheets] → [SAP Goods Movement MM module] → [Staging DB] → [Data Warehouse] → 
                          ↑
[MANDT/MTKL Orders] → [Remote Server For SAP HANA 4] → [Power BI ETL Using Mquery] → [DAX Measures] → Dashboard 

```

- **ETL Logic**: Map Excel’s Planned/Process Orders to ERP’s transactional 002/003 movements.
- **Key Joins**: `UT Batch #` + `Material Code` + `Order #`.

### **3.2 Data Model**

### **3.2.1 Dimensional Model**

- **Fact Tables**:
    - `FactMaterialConsumption` (002-RM): Links Planned Orders to RM usage.
    - `FactWIPMovement` (003-UT): Tracks UT batches to Ferment/BBT/Keg jobs.
- **Dimension Tables**:
    - `DimBatch` (21 UT Batch #s + Brand)
    - `DimMaterial` (WORT, FRMT, BBTX mapped to BOMs)
    - `DimOrder` (Planned vs. Process Order metadata)

### **3.2.2 Hierarchical Relationships**

```
Planned Order → Process Order → UT Batch → Material Code → RM/WIP/FG
```

---

## **4. Key Components & Logic**

### **4.1 Order Reconciliation**

- **Planned Order**: Theoretical RM demand from BOMs (e.g., 4400 HL for 6 brews).
- **Process Order**: Actual consumption (Excel) vs. planned (BOM).
- **Loss Analysis**:
    
    ```sql
    SELECT
      PlannedOrder,
      SUM(BOM_Qty) AS Planned_Qty,
      SUM(Excel_Qty) AS Actual_Qty,
      (Planned_Qty - Actual_Qty) AS Variance
    FROM FactMaterialConsumption
    GROUP BY UT_Batch;
    ```
    

### **4.2 Material Code Mapping**

| Material Code | Job Stage | Description |
| --- | --- | --- |
| WORT | UT | Wort pre-fermentation |
| FRMT | Ferment | Fermented product |
| BBTX | BBT | Bright beer tank |

### **4.3 Batch Traceability**

- **UT → FG Path**: Trace a UT batch through Ferment, BBT, and Keg jobs using `Batch #`.
- **Example**:
    
    ```sql
    SELECT * FROM FactWIPMovement
    WHERE UT_Batch = 'UT21'
    ORDER BY Job_Stage_Time;
    ```
    

---

## 

---

## **6. Recommendations**

### **6.1 Governance**

- **Data Validation Rules**:
    - Flag variances >5% between Planned/Actual RM consumption.
    - Reconcile UT Batch #s in Excel vs. ERP daily.
- **Role-Based Access**: Restrict Excel edit rights to production supervisors.

### **6.2 Reporting Enhancements**

- **Power BI Dashboard**:
    - Layer 1: Batch-level RM/WIP/FG status.
    - Layer 2: Brand-specific BOM vs. Actual trends.
    - Layer 3: Drill-down to UT → Keg job timelines.
- **KPI Alerts**: Email notifications for overdue batches or negative variances.

---

## **7. Conclusion**

By structuring data around **UT Batch #** and **Material Codes**, this model enables end-to-end traceability and variance analysis. Prioritizing automation and governance will mitigate risks of manual data handling and ensure alignment with BOMs across brands.

**Next Steps**: 

1. Get into a meeting with Abhishekji or Rahulji to use command codes to search the tables and verify as the SAP Users here do not have the authority for the same
2. Prototype the Power BI dashboard and validate with a pilot UT batch (e.g., UT01-UT05).

---