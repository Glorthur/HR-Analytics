# Exploratory Data Analysis_HR-Analytics
HR Analytics: Exploratory Data Analysis on employee retention and performance factors

# Purpose
The purpose of this project was to perform exploratory data analysis on the performance and productivity data, generate useful insights, and generate visualizations.

## Dataset Used
The dataset comprises 100,000 rows of employee data, covering performance, productivity, and demographic information in a corporate setting. It contains details about employees' jobs, work habits, education, performance levels, and job satisfaction. This comprehensive dataset supports multiple analytical purposes, including HR analytics, predicting employee turnover, analyzing productivity, and evaluating performance. Out of the total, 2525 rows were imported.

**Dataset Description**

- **Employee_ID**: Unique ID
- **Department**: Work area (Sales, HR, IT)
- **Gender**: Male, Female, Other
- **Age**: 22-60 years
- **Job_Title**: Role at company
- **Hire_Date**: Start date
- **Years_At_Company**: Time with company
- **Education_Level**: Highest degree
- **Performance_Score**: Rating (1-5)
- **Monthly_Salary**: Pay in USD
- **Work_Hours_Per_Week**: Weekly hours
- **Projects_Handled**: Project count
- **Overtime_Hours**: Extra hours/year
- **Sick_Days**: Days off sick
- **Remote_Work_Frequency**: % remote work
- **Team_Size**: People in team
- **Training_Hours**: Training time
- **Promotions**: Times promoted
- **Employee_Satisfaction_Score**: Rating (1-5)
- **Resigned**: Left company (Yes/No)

## Overview of the Dataset
- **Employee HR Analytics EDA**: Comment out USE statement for now
- **USE perf_prod**: Comment out USE statement

### 1. Basic overview of the dataset
```sql
SELECT COUNT(*) AS total_employees FROM employees;
```

### 2. Understanding the data structure
```sql
DESC employees;
```

The output of this query revealed that there were 2525 imported rows out of the total of 100,000.

The first 10 rows of the dataset were sampled to inspect the data for any anomalies.

The basic structure of the table was investigated using the describe keyword to figure out the datatypes for the various columns.

# Data Quality Checks
Data quality is crucial for accurate insights. The following checks were performed:

## Check for Missing Values
```sql
-- Check for missing values in key columns
SELECT
  COUNT(*) AS total_records,
  SUM(CASE WHEN Employee_ID IS NULL THEN 1 ELSE 0 END) AS missing_employee_id,
  SUM(CASE WHEN Department IS NULL THEN 1 ELSE 0 END) AS missing_department,
  SUM(CASE WHEN Gender IS NULL THEN 1 ELSE 0 END) AS missing_gender,
  SUM(CASE WHEN Age IS NULL THEN 1 ELSE 0 END) AS missing_age,
  SUM(CASE WHEN Monthly_Salary IS NULL THEN 1 ELSE 0 END) AS missing_salary,
  SUM(CASE WHEN Performance_Score IS NULL THEN 1 ELSE 0 END) AS missing_performance,
  SUM(CASE WHEN Employee_Satisfaction_Score IS NULL THEN 1 ELSE 0 END) AS missing_satisfaction,
  SUM(CASE WHEN Resigned IS NULL THEN 1 ELSE 0 END) AS missing_resignation
FROM employees;
```

**Insights**: No missing values were found in the dataset.

## Check for Duplicate Records
```sql
-- Check for duplicate employee records
SELECT
  Employee_ID,
  COUNT(*) AS record_count
FROM employees
GROUP BY Employee_ID
HAVING COUNT(*) > 1;
```

**Insights**: No duplicate records were found.

## Get Column Information
```sql
-- Get column information including data types
SELECT
COLUMN_NAME,
DATA_TYPE,
CHARACTER_MAXIMUM_LENGTH,
IS_NULLABLE
FROM
INFORMATION_SCHEMA.COLUMNS
WHERE
TABLE_SCHEMA = 'perf_prod';
```

**Insights**: All columns have appropriate data types and constraints.

# Demographic Analysis
## Age Distribution
```sql
-- Age distribution
SELECT 
    age_bracket,
    CONCAT(age_bracket, '-', age_bracket+4) AS age_range,
    employee_count
FROM (
    SELECT 
        FLOOR(Age/5)*5 AS age_bracket,
        COUNT(*) AS employee_count
    FROM employees
    GROUP BY age_bracket
) AS subquery;
```

**Key Insights**:
- Balanced Multi-Generational Workforce: The organization has a balanced age distribution, with the 35-39 age group being the largest and the 60-64 group being the smallest.
- Demographic Stability: Consistent hiring practices and good retention across age groups.

## Gender Distribution
```sql
-- Gender distribution
SELECT 
    Gender,
    COUNT(*) AS count,
    ROUND(COUNT(*) / (SELECT COUNT(*) FROM employees) * 100, 1) AS percentage
FROM employees
GROUP BY Gender;
```

**Key Insights**:
- Near Gender Balance: The organization has a nearly balanced gender distribution (49.3% Male, 46.5% Female, 4.2% Other).
- Inclusive Representation: The presence of "Other" indicates diversity beyond binary classification.

## Education Level Analysis
```sql
-- Performance by education level with additional metrics
SELECT 
    Education_Level,
    COUNT(*) AS employee_count,
    ROUND(AVG(Performance_Score), 1) AS avg_performance,
    ROUND(AVG(Monthly_Salary), 2) AS avg_salary,
    ROUND(AVG(Employee_Satisfaction_Score), 2) AS avg_satisfaction,
    SUM(CASE WHEN Resigned = 1 THEN 1 ELSE 0 END) AS resignation_count,
    ROUND(SUM(CASE WHEN Resigned = 1 THEN 1 ELSE 0 END) / COUNT(*) * 100, 1) AS resignation_rate
FROM employees
GROUP BY Education_Level;
```

**Key Insights**:
- Education Distribution: Bachelor's degree holders dominate the workforce.
- Performance vs. Education: Small performance variation across education levels.
- Satisfaction Patterns: Master's degree holders have the highest satisfaction.

# Performance Analysis
```sql
-- Department-level KPI summary for dashboard
SELECT 
    Department,
    COUNT(*) AS employee_count,
    ROUND(AVG(Employee_Satisfaction_Score), 2) AS avg_satisfaction,
    ROUND((SUM(CASE WHEN Resigned = 1 THEN 1 ELSE 0 END) / COUNT(*)) * 100, 1) AS attrition_rate_percent,
    ROUND(AVG(Performance_Score), 1) AS avg_performance,
    ROUND(AVG(Monthly_Salary), 2) AS avg_salary
FROM employees
GROUP BY Department;
```

**Key Insights**:
- Performance Consistency: Minimal variation in performance scores across departments.
- Departmental Excellence: IT leads in performance, satisfaction, and compensation but has high attrition.
- Attrition Concerns: Legal and Engineering have high attrition rates.

## Compensation Analysis
```sql
-- Department salary comparison analysis
SELECT
    Department,
    COUNT(*) AS employee_count,
    ROUND(AVG(Monthly_Salary), 2) AS avg_salary,
    ROUND(MIN(Monthly_Salary), 2) AS min_salary,
    ROUND(MAX(Monthly_Salary), 2) AS max_salary,
    ROUND(STDDEV(Monthly_Salary), 2) AS salary_deviation,
    ROUND(
        (
            AVG(Monthly_Salary) / (
                SELECT
                    AVG(Monthly_Salary)
                FROM
                    employees
            ) * 100 - 100
        ),
        1
    ) AS salary_parity
FROM employees
GROUP BY Department
ORDER BY avg_salary DESC;
```

**Key Insights**:
- Standardized Compensation: Narrow salary differentials across departments.
- Technical Premiums: IT and Engineering have modest salary premiums.
- Customer-Facing Compensation: Sales and Customer Support have slightly below-average pay.

# Retention Analysis
```sql
-- Satisfaction and Retention Analysis by Department
SELECT 
    Department,
    ROUND(AVG(Employee_Satisfaction_Score), 2) AS avg_satisfaction,
    COUNT(*) AS total_employees,
    SUM(CASE WHEN Resigned = 1 THEN 1 ELSE 0 END) AS resignation_count,
    ROUND(SUM(CASE WHEN Resigned = 1 THEN 1 ELSE 0 END) / COUNT(*) * 100, 1) AS resignation_rate,
    ROUND(AVG(CASE WHEN Resigned = 1 THEN Employee_Satisfaction_Score ELSE NULL END), 2) AS resigned_avg_satisfaction,
    ROUND(AVG(CASE WHEN Resigned = 0 THEN Employee_Satisfaction_Score ELSE NULL END), 2) AS active_avg_satisfaction,
    ROUND(AVG(Years_At_Company), 1) AS avg_tenure
FROM employees
GROUP BY Department
ORDER BY resignation_rate DESC;
```

**Key Insights**:
- Satisfaction-Attrition Paradox: Higher satisfaction doesn't always mean lower attrition.
- Resigned vs. Active Satisfaction Gap: Significant differences in satisfaction between resigned and active employees in some departments.
- Departmental Retention Challenges: Legal and IT have high attrition rates.

# Strategic Recommendations
1. **Performance Management Enhancement**
   - Implement Calibration: Introduce robust calibration processes to ensure meaningful performance differentiation.
   - Department-Specific Metrics: Develop tailored performance indicators that reflect each department's unique contributions.
   - Address Rating Biases: Provide manager training to recognize and mitigate central tendency bias.

2. **Targeted Retention Strategies**
   - Legal Department Focus: Develop urgent retention initiatives for the Legal department to address high attrition and satisfaction gaps.
   - Technical Talent Retention: Create specialized retention programs for IT and Engineering to preserve critical technical expertise.
   - High School Graduate Support: Implement career development pathways for employees without college degrees.

3. **Compensation Structure Optimization**
   - Function-Based Differentiation: Consider greater salary band variations that better reflect market realities for different functions.
   - Customer-Facing Compensation: Review compensation for Sales and Customer Support to ensure market competitiveness.
   - Progression Transparency: Develop clear communication about advancement through salary ranges.

4. **Employee Experience Improvement**
   - Department-Specific Engagement: Target engagement initiatives to departments with lower satisfaction scores.
   - Education-Based Development: Create tailored development opportunities based on educational background.
   - PhD Role Optimization: Review job responsibilities for PhD holders to better leverage their capabilities and address satisfaction gaps.

5. **Retention Beyond Satisfaction**
   - Exit Interview Analysis: Implement structured exit interviews, particularly in departments like IT and Finance where satisfied employees are still leaving.
   - Career Progression Clarity: Develop and communicate clear career paths, especially in departments with high attrition despite good satisfaction.
   - Targeted Stay Interviews: Conduct proactive stay interviews with high performers to identify retention risk factors beyond satisfaction.

# **Conclusion**
The workforce analysis reveals an organization with standardized practices across performance management and compensation, but with notable variations in satisfaction and retention. While the consistency in certain metrics demonstrates organizational maturity, opportunities exist to introduce greater differentiation where appropriate.

The data suggests that the organization would benefit from more tailored approaches to performance management, compensation, and retention strategies that acknowledge the unique characteristics of different departments and employee segments. By addressing these opportunities while maintaining its strengths in standardization and equity, the company can enhance both organizational performance and employee experience.

This analysis provides a foundation for data-driven HR decision-making and should be supplemented with qualitative insights from employee feedback and market benchmarking to develop comprehensive workforce strategies.

## Future Research Directions
- **Predictive Modeling for Retention**: Use logistic regression or decision trees to predict resignation based on variables like satisfaction, salary, tenure, and performance.
- **Root Cause Analysis for Attrition**: Drill down into reasons for resignation using case studies or additional qualitative data.
- **Employee Engagement Analysis**: Investigate factors influencing employee engagement and its impact on performance and retention.
- **Market Benchmarking**: Compare the organization's metrics with industry standards to identify areas for improvement.

By pursuing these research directions, the organization can gain deeper insights into its workforce dynamics and make informed decisions to enhance employee satisfaction, retention, and overall performance.
