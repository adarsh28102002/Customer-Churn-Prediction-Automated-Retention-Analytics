# Customer Churn Prediction & Automated Retention Analytics

## Project Overview

Customer Churn Prediction & Automated Retention Analytics is a data analytics project that studies customer behaviour to identify customers who may stop purchasing from a business. The project uses customer purchase history, satisfaction scores, support activity, membership plans and other details to understand churn patterns.

The project prepares and cleans customer data, builds an interactive Power BI dashboard, calculates important business KPIs using DAX formulas, and describes an automated retention workflow to help the business reconnect with at-risk customers.

## Project Objectives

- Analyse customer data to understand why customers stop purchasing.
- Identify active, at-risk, high-risk and churned customers.
- Calculate the customer churn rate and the financial value associated with churned customers.
- Study the relationship between customer satisfaction and purchase activity.
- Compare churn across acquisition channels and membership plans.
- Support data-driven customer retention decisions through dashboard insights.
- Plan automated, personalised win-back communication for selected customer segments.

## Tools and Technologies

- **Microsoft Excel:** Initial dataset handling.
- **Power Query:** Data cleaning, duplicate removal, blank-row removal and data type correction.
- **Power BI:** Interactive dashboard and data visualisation.
- **DAX (Data Analysis Expressions):** KPI and measure calculations.
- **n8n:** Workflow automation design.
- **Google Sheets:** Customer data source for the described automation workflow.
- **Email notifications:** Personalised retention communication as part of the workflow.

## Dataset Details

The project document describes a customer dataset containing 10,000 customer records.

| Column | Description |
|---|---|
| CustomerID | Unique customer identifier |
| SignupDate | Date the customer registered |
| SnapshotDate | Reference date for the dataset |
| Age, Gender | Customer demographic details |
| City, Region | Customer location |
| AcquisitionChannel | Channel through which the customer was acquired |
| MembershipPlan | Customer's membership plan |
| PreferredPaymentMethod | Preferred payment method |
| FavoriteCategory | Customer's preferred product category |
| TenureDays | Number of days since signup |
| TotalOrders | Total number of orders |
| AverageOrderValue_INR | Average amount spent per order |
| TotalSpend_INR | Total customer spending |
| EstimatedCustomerLifetimeValue_INR | Estimated customer lifetime value |
| DiscountUsagePct | Percentage of purchases using discounts |
| ReturnCount | Number of returns |
| SupportTickets | Number of support requests |
| ComplaintsCount | Number of complaints |
| EmailOpenRatePct | Email open rate |
| AppVisitsLast30Days | App visits in the last 30 days |
| SatisfactionScore_1to5 | Customer satisfaction score from 1 to 5 |
| LastPurchaseDate | Date of the latest purchase |
| DaysSinceLastPurchase | Days since the latest purchase |
| RecencySegment | Existing customer recency category |
| Churned_90d | Churn flag used by the analysis |
| RetentionActionSuggestion | Suggested customer retention action |

## Data Cleaning Using Power Query

1. Convert the customer data range into an Excel table and name it `CustomerData`.
2. Open the table in Power Query using Data → From Table/Range.
3. Remove duplicate records using `CustomerID`.
4. Remove blank rows.
5. Set suitable data types:
   - Date: `SignupDate`, `SnapshotDate`, `LastPurchaseDate`
   - Whole Number: `TotalOrders`, `SupportTickets`, `Churned_90d`, `DaysSinceLastPurchase`
   - Decimal Number: `TotalSpend_INR`, `SatisfactionScore_1to5`
6. Use View → Column Quality to inspect valid, error and empty values.
7. Create the `RiskSegment` conditional column using `DaysSinceLastPurchase`:
   - 0–30 days: Active
   - 31–60 days: At Risk
   - 61–90 days: High Risk
   - More than 90 days: Churned
8. Select Close & Load to load the cleaned data.

## Customer Churn Definition and Risk Segments

The project uses the `Churned_90d` field as its churn flag. A customer with no purchase for 90 or more days is treated as churned in the project definition.

| Risk Segment | Days Since Last Purchase | Meaning |
|---|---:|---|
| Active | 0–30 | Recently purchased |
| At Risk | 31–60 | May become inactive |
| High Risk | 61–90 | Close to the churn threshold |
| Churned | More than 90 | Meets the project's churn condition |

The `RiskSegment` conditional rules and the `Churned_90d` field are separate fields in the source dataset. Confirm that their boundary conditions are consistent when implementing the report.

## Power BI Dashboard

Import the cleaned `CustomerData` table into Power BI using Home → Get Data → Excel Workbook. Select the table, choose Transform Data if checks are needed, and then select Close & Apply.

### DAX Measures and Formulas

Create each measure from Modeling → New Measure. The formulas below follow the project document. Use the same table name consistently in your Power BI model. The examples use `CustomerData`.

<img width="1187" height="730" alt="image" src="https://github.com/user-attachments/assets/3a1ea38d-064b-4b0a-bd8e-4692c8023151" />

### 1. Total Customers

```dax
Total Customers =
DISTINCTCOUNT(CustomerData[CustomerID])
```

**Purpose:** Counts unique customers. It is the base count used by other KPI calculations.

### 2. Churned Customers

```dax
Churned Customers =
CALCULATE(
    [Total Customers],
    CustomerData[Churned_90d] = 1
)
```

**Purpose:** Counts customers whose churn flag is 1.

### 3. Churn Rate

```dax
Churn Rate =
DIVIDE(
    [Churned Customers],
    [Total Customers],
    0
)
```

**Formula:**

Churn Rate = (Churned Customers / Total Customers) × 100

**Purpose:** Shows the percentage of customers who have churned. Format this measure as Percentage in Power BI.

### 4. Average Order Value

```dax
Average Order Value =
DIVIDE(
    SUM(CustomerData[TotalSpend_INR]),
    SUM(CustomerData[TotalOrders]),
    0
)
```

**Formula:**

Average Order Value = Total Spend / Total Orders

**Purpose:** Calculates the average spend per order across the selected customer records.

### 5. Average Satisfaction

```dax
Average Satisfaction =
AVERAGE(CustomerData[SatisfactionScore_1to5])
```

**Purpose:** Calculates the average customer satisfaction score.

### 6. Revenue at Risk

```dax
Revenue at Risk =
CALCULATE(
    SUM(CustomerData[TotalSpend_INR]),
    CustomerData[Churned_90d] = 1
)
```

**Purpose:** Sums the recorded spend of customers whose churn flag is 1. In this project, it is used as an indicator of revenue associated with churned customers.

## Dashboard KPI Cards

Add these measures as Power BI Card visuals:

1. Total Customers
2. Churn Rate
3. Churned Customers
4. Average Satisfaction
5. Revenue at Risk

## Dashboard Charts

| Chart | Visualisation | Fields | Business purpose |
|---|---|---|---|
| Churn Rate by Acquisition Channel | Clustered Column Chart | X-axis: AcquisitionChannel; Y-axis: Churn Rate | Compare churn across acquisition channels |
| Customers by Risk Segment | Clustered Bar Chart | Category: RiskSegment; Value: Total Customers | See the number of customers in each risk group |
| Membership Plan Distribution | Donut Chart | Legend: MembershipPlan; Values: Total Customers | Compare customer distribution across plans |
| Satisfaction vs Days Since Last Purchase | Bar/Scatter visual as configured | DaysSinceLastPurchase, SatisfactionScore_1to5, CustomerID | Explore the relationship between satisfaction and purchase recency |
| Churn Rate by Satisfaction Score | Line Chart | X-axis: SatisfactionScore_1to5; Y-axis: Churn Rate | Examine churn patterns across satisfaction scores |

## Dashboard Slicers

Add slicers for:

- City
- MembershipPlan
- AcquisitionChannel

These slicers allow users to filter dashboard results by location, membership plan and acquisition source.

## Dynamic Formatting and Titles

The project document also includes DAX measures for conditional colours and dynamic chart titles. These examples use the CustomerData table name.

### Risk Segment Colour

```dax
RiskSegment Colour =
VAR MaxChurn =
    MAXX(
        ALLSELECTED(CustomerData[RiskSegment]),
        [Total Customers]
    )
VAR MinChurn =
    MINX(
        ALLSELECTED(CustomerData[RiskSegment]),
        [Total Customers]
    )
VAR CurrentValue = [Total Customers]
RETURN
    SWITCH(
        TRUE(),
        CurrentValue = MaxChurn, "#3C096C",
        CurrentValue = MinChurn, "#7B2CBF",
        "#E0AAFF"
    )
```

### Churn Rate Colour

```dax
Churn Rate Colour =
VAR MaxChurn =
    MAXX(
        ALLSELECTED(CustomerData[AcquisitionChannel]),
        [Churn Rate]
    )
VAR MinChurn =
    MINX(
        ALLSELECTED(CustomerData[AcquisitionChannel]),
        [Churn Rate]
    )
VAR CurrentChurn = [Churn Rate]
RETURN
    SWITCH(
        TRUE(),
        CurrentChurn = MaxChurn, "#3C096C",
        CurrentChurn = MinChurn, "#7B2CBF",
        "#E0AAFF"
    )
```

### Membership Plan Colour

```dax
MembershipPlan Colour =
VAR MaxValue =
    MAXX(
        ALLSELECTED(CustomerData[MembershipPlan]),
        [Total Customers]
    )
VAR MinValue =
    MINX(
        ALLSELECTED(CustomerData[MembershipPlan]),
        [Total Customers]
    )
VAR CurrentValue = [Total Customers]
RETURN
    SWITCH(
        TRUE(),
        CurrentValue = MaxValue, "#3C096C",
        CurrentValue = MinValue, "#7B2CBF",
        "#E0AAFF"
    )
```

### Dynamic Chart Titles

```dax
AcquisitionChannel Title =
"Acquisition Channel by "
    & SELECTEDVALUE(CustomerData[City], "All City")
    & " and "
    & SELECTEDVALUE(CustomerData[MembershipPlan], "All MembershipPlan")
```

```dax
Membership Plan Title =
"Membership Plan by "
    & SELECTEDVALUE(CustomerData[City], "All City")
```

```dax
Rating Title =
"Churn Rating by "
    & SELECTEDVALUE(CustomerData[City], "All City")
    & " and "
    & SELECTEDVALUE(CustomerData[MembershipPlan], "All MembershipPlan")
```

```dax
Risk Segment Title =
"Risk Segment by "
    & SELECTEDVALUE(CustomerData[City], "All City")
    & " and "
    & SELECTEDVALUE(CustomerData[MembershipPlan], "All MembershipPlan")
```

## Automated Retention Workflow

The project describes an n8n workflow intended to reduce manual customer follow-up:

1. **Scheduled trigger:** The workflow is planned to run daily at 10:00 AM.
2. **Read customer data:** Fetch customer records from Google Sheets.
3. **Filter customers:** Select records in the required risk segment.
4. **Prepare communication:** Create a personalised win-back message using customer details and an appropriate retention offer.
5. **Send notification:** Send the retention message through the configured email service.
6. **Retention goal:** Encourage inactive customers to return and make another purchase.

The workflow is a design described in the project document. Actual execution depends on the n8n workflow, connected services, credentials and data being configured correctly.

## Key Findings Reported in the Project

The project document reports the following dashboard findings:

- **Customer base:** 10,000 customer records.
- **Churned customers:** 917 customers, reported as a 9.17% churn rate.
- **Revenue at risk:** ₹13.97 million, as reported by the project.
- **At-risk segment:** More than 3,600 customers were reported in the 31–60 day inactivity segment.
- **Satisfaction pattern:** Customers with satisfaction scores of 1–2 had an average recency of 75 days; customers with a score of 5 had an average recency of 39 days.
- **Acquisition channels:** Referral was reported with 10.40% churn and Paid Search with 9.69% churn.

These are figures reported in the supplied project document; they should be checked against the final refreshed dataset and dashboard before being used in a presentation or business decision.

## Business Recommendations

1. **Target at-risk customers:** Use personalised win-back communication and suitable limited-time offers to encourage another purchase.
2. **Resolve customer complaints:** Give priority to customers with low satisfaction scores. Understand and address their issues before relying on discounts.
3. **Review acquisition channels:** Audit Referral and Paid Search campaigns, targeting and landing pages to improve customer quality.
4. **Review membership plans:** Compare Basic and Premium plan benefits and consider loyalty rewards or useful upgrade benefits.

## Project Workflow

```text
Customer Dataset (Excel)
          |
          v
Power Query Data Cleaning
          |
          v
Cleaned Customer Data
          |
          v
Power BI Dashboard + DAX Measures
          |
          v
Churn Analysis and Business Insights
          |
          v
n8n Retention Workflow Design
          |
          v
Personalised Customer Follow-up
```

## Conclusion

This project combines data cleaning, business intelligence and workflow automation to understand customer churn and support customer retention. Power Query prepares the customer data, Power BI presents the findings, and DAX measures calculate important KPIs such as total customers, churn rate, average order value and revenue at risk. The resulting insights help a business prioritise at-risk customers, improve customer support, review acquisition channels and strengthen membership strategies.

## Note

The supplied project document describes the analysis and workflow design. Validate the table and column names, DAX measures, dashboard figures and automation configuration in the actual project files before deployment.
