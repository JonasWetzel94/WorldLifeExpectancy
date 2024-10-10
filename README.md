# WorldLifeExpectancy

# Introduction
📊 **Explore global life expectancy trends!** This project focuses on cleaning and analyzing global life expectancy data, providing insights into patterns across different countries and years. Key topics include life expectancy improvements, comparisons with GDP and BMI, and how mortality rates are affecting life expectancy in different regions.

🔍 Curious about the SQL queries used in this analysis? You can explore them here: [project_sql folder](project_sql)

# Background
The goal of this project is to uncover patterns in global life expectancy, taking into account factors like GDP, BMI, and mortality rates. By cleaning the data and performing exploratory analysis, it provides a clear view of how different countries have evolved in terms of life expectancy over time.

### Key Questions Addressed:
1. **What countries have seen the biggest improvements in life expectancy?**
2. **How do GDP and BMI relate to life expectancy?**
3. **What impact does adult mortality have on life expectancy across different countries?**
4. **How does the overall global trend of life expectancy look over time?**
5. **Which countries have the highest and lowest life expectancy?**

# Tools I Used
For this analysis, I used the following tools:
- **MySQL:** The primary tool for managing and analyzing the data through structured queries.
- **Visual Studio Code:** Used for writing and executing MySQL queries in a structured environment.

# The Analysis
Each SQL query in this project was designed to clean, explore, and analyze key aspects of life expectancy across various countries. Below is a breakdown of how the analysis was conducted:

### 1. Data Cleaning
The first step was to clean the life expectancy data by removing duplicates, filling in missing values, and addressing any inconsistencies in the dataset.

```sql
-- Find and remove duplicates
SELECT Country, Year, CONCAT(Country, Year), COUNT(CONCAT(Country, Year))
FROM world_life_expectancy
GROUP BY Country, Year, CONCAT(Country, Year)
HAVING COUNT(CONCAT(Country, Year)) > 1;

-- Remove duplicates
DELETE FROM world_life_expectancy
WHERE Row_ID IN (
    SELECT Row_ID
    FROM (
        SELECT Row_ID, 
        CONCAT(Country, Year),
        ROW_NUMBER() OVER (PARTITION BY CONCAT(Country, Year) ORDER BY CONCAT(Country, Year)) AS Row_Num
        FROM world_life_expectancy
    ) AS Row_table
    WHERE Row_Num > 1
);
```

This query ensures that only unique records for each country and year are retained, preventing duplication of data.

### 2. Filling Missing Data
Next, missing values for the "Status" (Developed/Developing) and "Life expectancy" were filled based on patterns in the data.

```sql
-- Fill missing life expectancy values by calculating the average of previous and next years
UPDATE world_life_expectancy t1
JOIN world_life_expectancy t2
    ON t1.Country = t2.Country
    AND t1.Year = t2.Year - 1
JOIN world_life_expectancy t3
    ON t1.Country = t3.Country
    AND t1.Year = t3.Year + 1
SET t1.\`Life expectancy\` = ROUND((t2.\`Life expectancy\` + t3.\`Life expectancy\`)/2, 1)
WHERE t1.\`Life expectancy\` = '';
```

This method ensures that missing life expectancy data is populated in a logical way by averaging the values from surrounding years.

### 3. Exploratory Analysis
To gain insights into the life expectancy trends across countries, several analyses were performed:

- **Life expectancy improvement over 15 years:**

```sql
SELECT Country, MIN(\`Life expectancy\`), MAX(\`Life expectancy\`),
    ROUND(MAX(\`Life expectancy\`) - MIN(\`Life expectancy\`),1) AS Life_Increase_15_Years
FROM world_life_expectancy
GROUP BY Country
HAVING MIN(\`Life expectancy\`) <> 0
AND MAX(\`Life expectancy\`) <> 0
ORDER BY Life_Increase_15_Years DESC;
```

This query reveals the countries with the largest increases in life expectancy over the 15-year period, showing which regions have made the most progress.

- **Global trend over time:**

```sql
SELECT Year, ROUND(AVG(\`Life expectancy\`), 2)
FROM world_life_expectancy
WHERE \`Life expectancy\` <> 0
GROUP BY Year
ORDER BY Year;
```

This analysis displays the average global life expectancy for each year, allowing us to track overall improvements or declines in life expectancy over time.

### 4. Relationship Between Life Expectancy and GDP
One of the key questions addressed was how GDP influences life expectancy.

```sql
SELECT Country, 
    ROUND(AVG(\`Life expectancy\`), 1) AS Life_Exp, 
    ROUND(AVG(GDP), 1) AS GDP
FROM world_life_expectancy
GROUP BY Country
HAVING Life_Exp AND GDP > 0
ORDER BY GDP DESC;
```

This query ranks countries based on their GDP and provides insight into how wealthier nations tend to have higher life expectancy.

### 5. Adult Mortality and Life Expectancy
Another critical factor was adult mortality and its cumulative impact on life expectancy.

```sql
SELECT Country, Year, \`Life expectancy\`, \`Adult Mortality\`,
    SUM(\`Adult Mortality\`) OVER(PARTITION BY Country ORDER BY Year) AS Rolling_Total
FROM world_life_expectancy;
```

This query calculates the rolling total of adult mortality for each country over the years, allowing for an analysis of how mortality rates affect overall life expectancy trends.

# Additional SQL Queries
To further enrich the analysis, here are two additional queries that could provide valuable insights into the dataset:

1. **Top Countries with Highest Life Expectancy Improvement:**

```sql
SELECT Country, 
    MAX(\`Life expectancy\`) - MIN(\`Life expectancy\`) AS Life_Expectancy_Growth
FROM world_life_expectancy
GROUP BY Country
ORDER BY Life_Expectancy_Growth DESC
LIMIT 10;
```

This query shows the top 10 countries with the greatest improvement in life expectancy over time.

2. **Life Expectancy Based on Economic Status (Developed vs Developing):**

```sql
SELECT Status, 
    ROUND(AVG(\`Life expectancy\`), 2) AS Avg_Life_Expectancy
FROM world_life_expectancy
GROUP BY Status;
```

This query compares the average life expectancy between developed and developing countries, providing insights into the disparities between these economic groups.

# What I Learned
Throughout this project, I gained a deeper understanding of global life expectancy data and enhanced my SQL skills by working with real-world data challenges:

🧩 **Advanced Query Techniques:** Mastered complex queries using window functions, subqueries, and conditional aggregations.

📊 **Data Cleaning:** Improved my ability to clean datasets by identifying and resolving duplicate and missing data, ensuring data integrity for analysis.

💡 **Exploratory Data Analysis (EDA):** Enhanced my ability to uncover trends and patterns in data through SQL queries and visual representations.

# Conclusions

### Key Insights:
1. **Life Expectancy Trends:** Countries that have seen the most growth in life expectancy are often those with improving healthcare systems and economies.
2. **GDP and Life Expectancy Correlation:** Wealthier countries tend to have higher life expectancy, but there are exceptions where some lower-GDP countries have also made significant improvements.
3. **Adult Mortality’s Impact:** High adult mortality rates have a significant negative impact on life expectancy, particularly in developing countries.
4. **Global Improvements:** While life expectancy is generally improving globally, some regions are still lagging behind, and efforts are needed to address the disparities.
