# ✈️ Flight Data Analysis Using SQL

This repository contains SQL scripts used to analyze flight operations, airline performance, ticket sales, passenger travel patterns, and flight durations. The project uses MySQL to answer five key business questions, helping identify the busiest airports, evaluate airline performance, analyze passenger spending behavior, and understand flight operations.

## Introduction

The aviation industry generates large amounts of data related to flights, airports, airlines, passengers, and ticket sales. Analyzing this data helps airlines and airport authorities understand operational performance, passenger behavior, and revenue patterns.

This project aims to analyze flight data using SQL by applying joins, aggregate functions, Common Table Expressions (CTEs), window functions, and conditional statements. The analysis provides insights into airport activity, airline ticket sales, flight durations, passenger spending, and route-level ticket pricing.

## Data Sources

The project uses a structured relational database created in MySQL. The database consists of five tables:

- **Airports:** Contains airport IDs, airport names, locations, and countries.
- **Airlines:** Contains airline IDs, airline names, and countries.
- **Flights:** Contains flight IDs, origin and destination airports, departure and arrival times, and airline information.
- **Passengers:** Contains passenger details, dates of birth, and frequent flyer status.
- **Tickets:** Contains ticket IDs, passenger IDs, flight IDs, and ticket prices.

These tables are connected through primary and foreign key relationships to support comprehensive SQL analysis.

## Project Overview

1. Created the `flight_data_analysis` database using MySQL.
2. Designed five relational tables to store airport, airline, flight, passenger, and ticket information.
3. Inserted sample data and established relationships between tables.
4. Executed SQL queries using joins, aggregate functions, CTEs, window functions, and CASE statements.
5. Analyzed flight operations, airline performance, passenger spending, and ticket pricing to generate business insights.

## Business Requests

### 1. Identify the Busiest Airport by Departing Flights

**Objective:** Identify the airport with the highest number of departing flights to understand airport activity and departure traffic.

```sql
SELECT
    a.Name AS AirportName,
    COUNT(*) AS TotalFlights
FROM Flights f
JOIN Airports a
    ON f.Origin = a.AirportID
GROUP BY a.AirportID, a.Name
ORDER BY TotalFlights DESC
LIMIT 1;
```

**Business Value:** Helps identify airports with higher departure activity and supports airport resource planning and operational management.

### 2. Analyze Total Tickets Sold by Airline

**Objective:** Calculate the total number of tickets sold for each airline to compare airline-level booking activity.

```sql
SELECT
    a.Name AS Airline,
    COUNT(*) AS TicketsSold
FROM Tickets t
JOIN Flights f
    ON t.FlightID = f.FlightID
JOIN Airlines a
    ON f.AirlineID = a.AirlineID
GROUP BY a.AirlineID, a.Name
ORDER BY TicketsSold DESC;
```

**Business Value:** Helps compare ticket sales across airlines and identify airlines with higher booking volumes.

### 3. Categorize Flights Based on Duration

**Objective:** Calculate the duration of each flight in hours and classify flights into Short, Medium, and Long categories.

```sql
SELECT
    FlightID,
    DepartureTime,
    ArrivalTime,
    ROUND(
        TIMESTAMPDIFF(
            MINUTE,
            DepartureTime,
            ArrivalTime
        ) / 60.0,
        2
    ) AS DurationHours,
    CASE
        WHEN TIMESTAMPDIFF(
            MINUTE,
            DepartureTime,
            ArrivalTime
        ) < 120 THEN 'Short'
        WHEN TIMESTAMPDIFF(
            MINUTE,
            DepartureTime,
            ArrivalTime
        ) <= 300 THEN 'Medium'
        ELSE 'Long'
    END AS FlightCategory
FROM Flights
ORDER BY DurationHours;
```

**Business Value:** Helps categorize flight operations by duration and supports route analysis and scheduling decisions.

**Classification:**
- Short: Less than 2 hours.
- Medium: 2 to 5 hours, inclusive.
- Long: More than 5 hours.

### 4. Identify the Highest-Priced Ticket for Each Route

**Objective:** Find the highest-priced ticket sold for each origin-to-destination route using ranking functions.

```sql
WITH CTE_RouteTickets AS (
    SELECT
        f.FlightID,
        f.Origin,
        f.Destination,
        t.TicketID,
        t.Price,
        RANK() OVER (
            PARTITION BY f.Origin, f.Destination
            ORDER BY t.Price DESC
        ) AS rnk
    FROM Tickets t
    JOIN Flights f
        ON t.FlightID = f.FlightID
)
SELECT
    A1.Name AS Origin,
    A2.Name AS Destination,
    rt.FlightID,
    rt.TicketID,
    rt.Price
FROM CTE_RouteTickets rt
JOIN Airports A1
    ON rt.Origin = A1.AirportID
JOIN Airports A2
    ON rt.Destination = A2.AirportID
WHERE rt.rnk = 1
ORDER BY rt.Price DESC;
```

**Business Value:** Helps identify the highest ticket prices across routes and supports route-level pricing analysis.

### 5. Find the Highest-Spending Passenger in Each Frequent Flyer Category

**Objective:** Identify the passenger who has spent the most on tickets in each frequent flyer status group.

```sql
WITH CTE_Spending AS (
    SELECT
        p.PassengerID,
        p.Name,
        p.FrequentFlyerStatus,
        SUM(t.Price) AS TotalSpent,
        RANK() OVER (
            PARTITION BY p.FrequentFlyerStatus
            ORDER BY SUM(t.Price) DESC
        ) AS rn
    FROM Passengers p
    JOIN Tickets t
        ON p.PassengerID = t.PassengerID
    GROUP BY
        p.PassengerID,
        p.Name,
        p.FrequentFlyerStatus
)
SELECT
    Name,
    FrequentFlyerStatus,
    TotalSpent
FROM CTE_Spending
WHERE rn = 1
ORDER BY TotalSpent DESC;
```

**Business Value:** Helps identify high-value passengers in each loyalty category and supports customer retention and frequent flyer program planning.

## SQL Concepts Used

- **DDL (Data Definition Language):** `CREATE DATABASE`, `CREATE TABLE`
- **DML (Data Manipulation Language):** `INSERT INTO`
- **Joins:** `INNER JOIN` and `JOIN` for combining related tables.
- **Aggregate Functions:** `COUNT()`, `SUM()`, and `MIN()` / `MAX()` concepts.
- **Window Functions:** `RANK()` for ranking ticket prices and passenger spending.
- **Common Table Expressions (CTEs):** Simplified multi-step analytical queries.
- **Conditional Logic:** `CASE WHEN` for classifying flight durations.
- **Date and Time Functions:** `TIMESTAMPDIFF()` for calculating flight duration.
- **Sorting and Filtering:** `ORDER BY`, `WHERE`, and `LIMIT`.

## Results and Insights

The project is designed to generate the following insights:

- Identify the busiest airport based on the number of departing flights.
- Compare ticket sales across different airlines.
- Understand the distribution of flights by duration.
- Identify the highest-priced tickets for individual routes.
- Discover the highest-spending passengers across frequent flyer categories.

These insights can support airline performance comparisons, airport planning, pricing analysis, and passenger relationship management.

## Limitations and Challenges

- **Sample Dataset:** The project uses a limited sample dataset, so the findings may not represent actual airline or airport performance.
- **Ticket Sales vs. Revenue:** Ticket counts measure booking volume, not total revenue. Revenue comparisons require aggregating ticket prices.
- **Flight Scheduling:** The dataset does not contain actual departure or arrival records, delays, cancellations, or missed connections.
- **Pricing Analysis:** The highest ticket price alone does not establish the most profitable route or explain pricing differences.
- **Flight Duration:** Duration is calculated from scheduled departure and arrival timestamps; actual flight duration may differ.
- **Passenger Analysis:** Passenger spending is calculated from available ticket records and does not include other charges, refunds, or ancillary purchases.

## Conclusion

This Flight Data Analysis project demonstrates the practical application of MySQL for solving business problems using relational data. By analyzing airport traffic, airline ticket sales, flight durations, route-level ticket prices, and passenger spending, the project showcases essential SQL skills and generates insights that can support data-driven decision-making in the aviation industry.

## Future Work

Future enhancements could include:

- Analyzing flight delays, cancellations, and on-time performance.
- Calculating total ticket revenue and average ticket prices by airline and route.
- Identifying the most frequently traveled routes and busiest destinations.
- Analyzing passenger travel frequency and loyalty program engagement.
- Developing interactive dashboards in Power BI or Tableau.
- Using larger, real-world flight datasets to identify trends and improve the reliability of findings.

## Tools and Technologies

- **Database:** MySQL
- **Query Language:** SQL
- **Analysis Techniques:** Joins, CTEs, aggregate functions, window functions, and date/time calculations
- **Version Control:** Git and GitHub

---

**Project:** Flight Data Analysis Using SQL

**Database:** `flight_data_analysis`

**Focus Areas:** Airline Performance | Airport Operations | Ticket Pricing | Passenger Analytics
