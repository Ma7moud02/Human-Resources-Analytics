# HR Analytics Project: Data Camp Competition

## Table of Contents
1. [Introduction](#introduction)
2. [Overview](#overview)
3. [Project Background](#project-background)
4. [Data Exploration](#data-exploration)
5. [Data Modeling](#data-modeling)
   - [DimDate Table](#dimdate-table)
6. [Key Measures](#key-measures)
7. [Dashboard Development](#dashboard-development)
   - [Page 1: Company Structure](#page-1-company-structure)
   - [Page 2: Demographics Analysis](#page-2-demographics-analysis)
   - [Page 3: Attrition Rates](#page-3-attrition-rates)
   - [Page 4: Employees Satisfaction](#page-4-employees-satisfaction)
   - [Page 5: Satisfaction Levels and Performance Tracker](#page-5-satisfaction-levels-and-performance-tracker)

8. [Insights and Analysis](#insights-and-analysis)
9. [Conclusion](#conclusion)
10. [Attachments](#attachments)

## Introduction
This project involves a comprehensive analysis of employee data to derive actionable insights into workforce dynamics, satisfaction metrics, and attrition trends. Leveraging tools like Power BI and DAX, the project culminates in an interactive dashboard designed to support HR decision-making with key metrics and trends.

🎯 **Objective**
The primary goal of this project was to perform a detailed analysis of employee data, focusing on several key aspects:

- **Understand Workforce Distribution**: Analyze demographics, roles, and regional employee concentration.
- **Track Attrition Trends**: Identify patterns in attrition and factors influencing employee turnover.
- **Evaluate Satisfaction Metrics**: Measure and analyze job satisfaction, environment satisfaction, and work-life balance.
- **Provide Actionable Insights**: Use data visualization to enable informed decision-making for retention strategies and organizational improvements.

## Overview
📊 This README file documents the technical journey of the HR Analytics project, developed during a data camp competition. The project earned second place in the competition and showcases advanced data analysis and visualization techniques using Power BI. Below is a comprehensive breakdown of the process, methodologies, and outcomes.

## Project Background
📈 The competition involved analyzing a dataset of employees in a company to derive actionable insights about HR metrics such as satisfaction levels, attrition rates, and performance evaluations. The primary goals were:

- Perform exploratory data analysis (EDA) to understand the dataset.
- Implement an effective data modeling strategy.
- Build insightful dashboards using Power BI.

## Data Exploration
🔍 The dataset provided contained detailed information about employees, including demographics, job roles, satisfaction levels, and performance metrics. Below is the data dictionary summarizing the key tables and fields:

### Key Tables
- **Employee**: Contains demographic and job-related data.
- **Performance**: Tracks performance review metrics.
- **Satisfied Level**: Maps satisfaction levels to descriptive categories.
- **Rating Level**: Maps performance ratings to descriptive categories.

For more details, refer to the attached data dictionary.

## Data Modeling
📐 After EDA, a snowflake schema was chosen for the data model as it allowed for:

- Efficient normalization of related dimensions.
- Simplified maintenance and flexibility for complex queries.

The schema design includes fact and dimension tables linked appropriately to ensure a clear and logical representation of the data.

### DimDate Table
📅 A custom date table was created to facilitate time-based analysis. The table includes fields for fiscal year, month, quarter, and week calculations, providing granular control over temporal data. Below is the DAX formula used:

```DAX
DimDate = 
VAR _minYear = YEAR(MIN(DimEmployee[HireDate]))
VAR _maxYear = YEAR(MAX(DimEmployee[HireDate]))
VAR _fiscalStart = 4 

RETURN
ADDCOLUMNS(

    CALENDAR(

                DATE(_minYear,1,1),

                DATE(_maxYear,12,31)

),

"Year",YEAR([Date]),
"Year Start",DATE( YEAR([Date]),1,1),
"YearEnd",DATE( YEAR([Date]),12,31),
"MonthNumber",MONTH([Date]),
"MonthStart",DATE( YEAR([Date]), MONTH([Date]), 1),
"MonthEnd",EOMONTH([Date],0),
"DaysInMonth",DATEDIFF(DATE( YEAR([Date]), MONTH([Date]), 1),EOMONTH([Date],0),DAY)+1,
"YearMonthNumber",INT(FORMAT([Date],"YYYYMM")),
"YearMonthName",FORMAT([Date],"YYYY-MMM"),
"DayNumber",DAY([Date]),
"DayName",FORMAT([Date],"DDDD"),
"DayNameShort",FORMAT([Date],"DDD"),
"DayOfWeek",WEEKDAY([Date]),
"MonthName",FORMAT([Date],"MMMM"),
"MonthNameShort",FORMAT([Date],"MMM"),
"Quarter",QUARTER([Date]),
"QuarterName","Q"&FORMAT([Date],"Q"),
"YearQuarterNumber",INT(FORMAT([Date],"YYYYQ")),
"YearQuarterName",FORMAT([Date],"YYYY")&" Q"&FORMAT([Date],"Q"),
"QuarterStart",DATE( YEAR([Date]), (QUARTER([Date])*3)-2, 1),
"QuarterEnd",EOMONTH(DATE( YEAR([Date]), QUARTER([Date])*3, 1),0),
"WeekNumber",WEEKNUM([Date]),
"WeekStart", [Date]-WEEKDAY([Date])+1,
"WeekEnd",[Date]+7-WEEKDAY([Date]),
"FiscalYear",if(_fiscalStart=1,YEAR([Date]),YEAR([Date])+ QUOTIENT(MONTH([Date])+ (13-_fiscalStart),13)),
"FiscalQuarter",QUARTER( DATE( YEAR([Date]),MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1,1) ),
"FiscalMonth",MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1
)
```

This table enhances reporting capabilities by aligning analysis with both calendar and fiscal periods.

## Key Measures
🔢 Below are some of the key measures used in the project, along with their descriptions and DAX formulas:

### 1. % Attrition Rate
**Description**: Calculates the percentage of employees who have left the company.
```DAX
% Attrition Rate = 
COALESCE(DIVIDE([InactiveEmployees], [TotalEmployees], 0), 0)
```

### 2. Cumulative Active Employees
**Description**: Tracks the cumulative count of active employees over time.
```DAX
Cumulative Active Employees = 
CALCULATE(
    DISTINCTCOUNT('DimEmployee'[EmployeeID]),
    FILTER(
        ALL('DimDate'[Year]),
        'DimDate'[Year] <= MAX('DimDate'[Year])
    ),
    'DimEmployee'[Attrition] = "No",
    USERELATIONSHIP(DimEmployee[HireDate], DimDate[Date])
)
```

### 3. Attrition Rate by Group
**Description**: Calculates the attrition rate for specific groups of employees.
```DAX
AttritionRateByGroup = 
DIVIDE(
    CALCULATE(
        COUNTROWS(dimEmployee),
        dimEmployee[Attrition] = "Yes"
    ),
    COUNTROWS(dimEmployee)
) * 100
```

### 4. Last Review Date
**Description**: Returns the most recent review date for an employee or indicates no review if unavailable.
```DAX
LastReviewDate = 
IF (
    MAX ( FactPerformanceRating[ReviewDate] ) = BLANK (),
    "No Review Yet",
    MAX ( FactPerformanceRating[ReviewDate] )
)
```

### 5. Avg Years at Company by Group
**Description**: Calculates the average years employees have spent at the company for specific groups.
```DAX
AvgYearsAtCompanyByGroup = 
AVERAGEX(
    SUMMARIZE(
        dimEmployee,
        dimEmployee[YearsAtCompany]
    ),
    dimEmployee[YearsAtCompany]
)
```

### 6. Avg Satisfaction by Group
**Description**: Computes the average job satisfaction for employee groups.
```DAX
AvgSatisfactionByGroup = 
AVERAGEX(
    SUMMARIZE(
        factPerformanceRating,
        factPerformanceRating[JobSatisfaction]
    ),
    factPerformanceRating[JobSatisfaction]
)
```
## Dashboard Development
📊 Using Power BI, interactive dashboards were created to visualize key insights. Below is a detailed breakdown of each dashboard page:

### Page 1: Company Structure
- **Highlights**:
  - The company operates in the US, mainly in New York, California, and Illinois.
  - Total Employees: 1,470.
  - Active Employees: 1,233 (84%).
  - Inactive Employees: 237 (16%).
  - Attrition Rate: 16.1%.
  - The company primarily works in tech and sales roles.
  - Steady growth over the past 10 years (2012-2023) indicates healthy organizational expansion.
- **Placeholder for Screenshot**

### Page 2: Demographics Analysis
- **Highlights**:
  - Gender Distribution: Nearly equal, with a 2.7% increase in female employees last year.
  - Ethnicity Breakdown:
    - Significant spikes in average salaries for White and Native Hawaiian employees.
    - White employees earn 12% more on average than all other races combined.
  - Age Distribution:
    - 59.5% (874 employees) are aged 20-29.
    - 19.65% (289 employees) are aged 30-39.
    - Hiring trends focus on younger age groups.
  - California has the highest employee concentration with 875 employees (59.5%).
- **Placeholder for Screenshot**

### Page 3: Attrition Rates
- **Highlights**:
  - Inactive Employees by Year:
    - Significant drops in attrition during 2015 (15 employees) and 2017 (11 employees).
    - 2020 marked the highest attrition with 28 employees.
  - Highest Attrition Rates:
    - Data Scientists (Tech Department), Recruiters (HR Department), and Sales Representatives (Sales Department).
    - Clear link between low pay and high attrition in these roles.
  - Attrition Rate by Travel Frequency:
    - Frequent travelers exhibit the highest attrition rate (over 20%).
- **Placeholder for Screenshot**

### Page 4: Employees Satisfaction
- **Highlights**:
  - Average Satisfaction Score: 3.54.
  - Illinois has the highest employee satisfaction.
  - Satisfaction dips for Native Hawaiian and Asian employees, raising equity concerns.
  - Declines in satisfaction metrics (environment, job, relationship satisfaction, and work-life balance) since 2016.
  - Scatter plot indicates satisfaction increases with tenure, suggesting the need for better engagement strategies for newer employees.
- **Placeholder for Screenshot**

### Page 5: Satisfaction Levels and Performance Tracker
- **Highlights**:
  - Tool for HR to filter employees and view satisfaction and performance ratings over time.
  - Provides a comprehensive view of satisfaction trends and their impact on performance.
- **Placeholder for Screenshot**



## Insights and Analysis
💡 **Key Takeaways:**
- **Diversity and Inclusion**:
  - Gender distribution is nearly balanced, but representation of non-binary individuals could be improved.
  - Ethnic representation varies significantly, with disparities in average salaries and satisfaction levels.

- **Employee Satisfaction**:
  - Satisfaction scores indicate room for improvement in relationship and environment satisfaction.
  - Tenure positively correlates with satisfaction, suggesting a need for better onboarding and engagement strategies for newer employees.

- **Attrition Trends**:
  - High attrition rates among single employees and frequent travelers highlight potential areas for retention efforts.
  - Roles like Data Scientist and Sales Representatives experience the highest attrition, correlating with lower pay in these roles.

- **Financial Insights**:
  - Average salary disparities exist across ethnic and job role groups, raising questions about equity.

## Conclusion
🎯 This project underscores the importance of combining technical expertise with effective storytelling to derive actionable insights. The structured approach to data modeling, analysis, and visualization enabled comprehensive insights into workforce dynamics, contributing to a successful presentation that secured second place in the competition.

## Attachments
📎 **Attachments**:
1. Data Dictionary
2. Data Model Schema (**Placeholder for Screenshot**)
3. Dashboard Screenshots (**Placeholders for Screenshots**)
