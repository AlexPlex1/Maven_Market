# Maven Market Report

## Intro

This project is provided by Maven Analytics in the Microsoft Power BI Desktop for Business Intelligence course.
The dataset is about Maven Market, a fictional multi-national grocery chain with locations in Canada, Mexico, and the United States.

## Problem Statement

This report helps the owners of Maven Market understand valuable key insights and track KPIs about their company. It specifically targets the year 1998 since that is when the data is referring to. 

Some of the KPIs we track are the current month's transactions, the month's profit, and returns. We also have a detailed list of product brands where we monitor the total transactions, total profit, profit margin, and return rate.

This report aims to provide business value to the company so they can make data-driven decisions in the future.


### Steps followed 

- Step 1 : Loaded data into Power BI Desktop, datasets are stored in CSV files.
- Step 2 : Opened power query editor & in the view tab under Data preview section, checked "column distribution", "column quality" & "column profile" options.
- Step 3 : Also since by default, profile will be opened only for 1000 rows we selected "column profiling based on the entire dataset".
- Step 4 : After observing the data for errors, any minor issues were fixed. For example, in the low_fat column of the products table, null values were replaced by 0. We also handled data type errors and populated the calendar table with extra columns based on the date.
- Step 5 : We proceeded to create the data model connecting the fact tables to the dimension tables with appropriate relationships in the Model view.
- Step 6 : In the Data view, we made adjustments to some fields, for example changing the currency columns to show as currency and categorize customer_city, customer_postal_code, and customer_country.
- Step 7 : Again in the Data view, we created calculated columns to use in our analysis. Specifically (along with DAX):
  - In the Calendar table, added a column named "Weekend". Equals "Y" for Saturdays or Sundays (otherwise "N").
```dax 
        Weekend = 
        IF(
        OR('Calendar'[Day Name] = "Saturday" ,
        'Calendar'[Day Name] = "Sunday"),
        "Y",
        "N")
```

  - In the Calendar table, added a column named "End of Month". Returns the last date of the current month for each row.
```dax 
        End of Month = 
        EOMONTH('Calendar'[date],
                0)
```

  - In the Customers table, added a column named "Current Age". Calculates current customer ages using the "birthdate" column and the TODAY() function.
```dax 
        Current Age = 
        DATEDIFF(
        Customers[birthdate],
        TODAY(),
        YEAR
        )
```
  - In the Customers table, added a column named "Priority". Equals "High" for customers who own homes and have Golden membership cards (otherwise "Standard").  
```dax 
        Priority = 
        IF(
        AND(Customers[member_card] = "Golden",
        Customers[homeowner] = "Y"),
        "High",
        "Standard"
        )
```
  - In the Customers table, added a column named "Short_Country". Returns the first three characters of the customer country, and converts to all uppercase.
```dax 
        Short_Country = 
        UPPER(
        LEFT(
                Customers[customer_country],
                3
        )
        )
```
  - In the Customers table, added a column named "House Number". Extracts all characters/numbers before the first space in the "customer_address" column.
```dax 
        House_Number = 
        LEFT(
        Customers[customer_address],
        SEARCH(
                " ",
                Customers[customer_address]
        ) -1
        )
```
  - In the Products table, added a column named "Price_Tier". Equals "High" if the retail price is >$3, "Mid" if the retail price is >$1, and "Low" otherwise.
```dax 
        Price_Tier = 
        SWITCH(
        TRUE(),
        Products[product_retail_price] > 3, "High",
        Products[product_retail_price] > 1, "Mid",
        "Low"
        )
```
  - In the Stores table, added a column named "Years_Since_Remodel". Calculates the number of years between the current date (TODAY()) and the last remodel date.
```dax 
        Years_Since_Remodel = 
        DATEDIFF(
        Stores[last_remodel_date],
        TODAY(),
        YEAR
        )
```

- Step 8 : In Report view, we created measures to use in our analysis. Specifically (along with DAX):
  - Created new measures named "Quantity Sold" and "Quantity Returned" to calculate the sum of quantity from each data table.
```dax 
        Quantity Sold = 
        SUM(
        Transaction_Data[quantity]
        )

        Quantity Returned = 
        SUM(
        Return_Data[quantity]
        )
```
  - Created new measures named "Total Transactions" and "Total Returns" to calculate the count of rows from each data table.
```dax
        Total Transactions = 
        COUNT(
        Transaction_Data[customer_id]
        )

        Total Returns = 
        COUNT(
        Return_Data[product_id]
        )
```
  - Created a new measure named "Return Rate" to calculate the ratio of quantity returned to quantity sold.
```dax
        Return Rate = 
        DIVIDE(
        [Quantity Returned],
        [Quantity Sold]
        )
```
  - Created a new measure named "Weekend Transactions" to calculate transactions on weekends.
```dax
        Weekend Transactions = 
        CALCULATE(
        COUNT(
                Transaction_Data[customer_id]
        ),
        FILTER(
                'Calendar',
                'Calendar'[Weekend] = "Y"
        )
        )
```
  - Created a new measure named "% Weekend Transactions" to calculate weekend transactions as a percentage of total transactions.
```dax
        % Weekend Transactions = 
        DIVIDE(
        [Weekend Transactions],
        [Total Transactions]
        )
```
  - Created new measures named "All Transactions" and "All Returns" to calculate grand total transactions and returns (regardless of filter context).
```dax
        All Transactions = 
        CALCULATE(
        [Total Transactions],
        ALL(
                Transaction_Data
        )
        )

        All Returns = 
        CALCULATE(
        [Total Returns],
        ALL(Return_Data)
        )
```
  - Created a new measure to calculate "Total Revenue" based on transaction quantity and product retail price.
```dax
        Total Revenue = 
        SUMX(
        Transaction_Data,
        Transaction_Data[quantity] *
        RELATED(
                Products[product_retail_price]
        )
        )
```
  - Created a new measure to calculate "Total Cost" based on transaction quantity and product cost.
```dax
        Total Cost = 
        SUMX(
        Transaction_Data,
        Transaction_Data[quantity] *
        RELATED(
                Products[product_cost]
        )
        )
```
  - Created a new measure named "Total Profit" to calculate total revenue minus total cost.
```dax
        Total Profit = 
        [Total Revenue] - [Total Cost]
```
  - Created a new measure to calculate "Profit Margin" by dividing total profit by total revenue calculate total revenue.
```dax
        Profit Margin = 
        DIVIDE(
        [Total Profit],
        [Total Revenue]
        )
```
  - Created a new measure named "Unique Products" to calculate the number of unique product names in the Products table.
```dax
        Unique Products = 
        DISTINCTCOUNT(
        Products[product_name]
        )
```
  - Created a new measure named "YTD Revenue" to calculate year-to-date total revenue.
```dax
        YTD Revenue = 
        TOTALYTD(
        [Total Revenue],
        'Calendar'[date])
```
  - Created a new measure named "60-Day Revenue" to calculate a running revenue total over a 60-day period.
```dax
        60-Day Revenue = 
        CALCULATE(
        [Total Revenue],
        DATESINPERIOD(
                'Calendar'[date],
                MAX('Calendar'[date]),
                -60,
                DAY)
        )
```
  - Created new measures named  "Last Month Transactions", "Last Month Revenue", "Last Month Profit", and "Last Month Returns".
```dax
        Last Month Transactions = 
        CALCULATE(
        [Total Transactions],
        PREVIOUSMONTH(
                'Calendar'[date]
        )
        )

        Last Month Revenue = 
        CALCULATE(
        [Total Revenue],
        PREVIOUSMONTH(
                'Calendar'[date]
        )
        )

        Last Month Profit = 
        CALCULATE(
        [Total Profit],
        PREVIOUSMONTH(
                'Calendar'[date]
        )
        )

        Last Month Returns = 
        CALCULATE(
        [Total Returns],
        PREVIOUSMONTH(
                'Calendar'[date]
        )
        )
```
  - Created a new measure named "Revenue Target" based on a 5% lift over the previous month revenue.
```dax
        Revenue Target = 
        [Last Month Revenue v2] * 1.05
```


- Step 9 : Added a page-level filter for the Year to be 1998.

- Step 10 : Added all visuals needed for the report.

# Report Snapshot (Power BI Desktop)

![MavenMarket_Report_Full_Report](https://github.com/user-attachments/assets/821c3ff5-5ae2-4f12-9b88-8a690ce78015)

# Description of visuals in the report

- Return icon
  - A user can select this icon to clear all filters on the report, essentially "restarting" the report.

- KPI Cards
  - At the top right of the report we included three KPI cards. For each card, we show the corresponding metric vs. the previous month's metric. By default, it shows the latest month vs the previous month. To see the comparison, a user can select any range of months from the filters (at least two). Our goal each month is to perform better than the previous month.

  ![KPI Cards](https://github.com/user-attachments/assets/c9ef49d5-f2a4-4b98-98d9-607a489759df)

- Table
  - At the bottom left of the report we have a table that shows various metrics per product brand for the top 30 brands by total transactions. In this table, the user can the total transactions, total profit, profit margin, and return rate for the full year of 1998. Additionally, the user can sort by any metric they want and even select a brand specifically, and the report will adapt to the choice, showing only information about that brand. Note: Only the treemap won't adapt because it is set up this way.

 ![Product Brand Table](https://github.com/user-attachments/assets/ad148334-9863-4ecf-8692-c527d374e37d)

- Slicer
  - We have included a slicer(filter) where a user can select any particular country for which they want to see data.

- Map
  - We also have a map that shows the total transactions by city.

- Treemap
  - We have a treemap where a user can see the total transactions by country by default. There is also the ability to drill down to state and city levels by clicking the double arrow button.

 ![Slicer Map Treemap](https://github.com/user-attachments/assets/83d4a86d-34a3-4c40-a7b6-bf164e1e4447)

- Column chart
  - We have a column chart showing the total revenue generated for each week of the year. Note that each bar represents the week from where the start of week is the date shown.

- Gauge chart
  - This is a chart where we can compare the total revenue against the revenue target for the date selected. As mentioned above the revenue target is the previous month revenue increased by 5%.

 ![Additional Charts](https://github.com/user-attachments/assets/08f0b95b-a947-488b-a14f-f88e8c2b6926)

# Insights

In the second page of our report the user can see some insights we uncovered. In addition, they can select the bookmark icon next to each insight to navigate to a view that shows that particular insight.

![Insights](https://github.com/user-attachments/assets/88e4f918-1300-4cef-861e-2bfc1d418f66)

Insights discovered:
 - Portland hits 1,000 sales in December.
 - High Top product returns doubled in Mexico (4 to 8), at a return rate of 1.2%.
 - Plato products drove the strongest overall profit margin (63.55%) in 1998.
 - Mexico achieved a staggering +29.9% increase in profits compared to its targets for December.



