-- Ticket Sales for 2021 Regular Season
-- Q1. Revenue and Seat Count by Event Date

SELECT
CONVERT(date, ts.event_date) as 'Event_Date'
, SUM(ts.Price) as 'Total_Revenue'
, COUNT(ts.SeatUniqueID) as 'Total_Seats'
FROM 
coltsdb.dbo.TicketSales ts 
WHERE
ts.Season = '2021' AND ts.status = 'SOLD'
GROUP BY 
ts.event_date
ORDER BY
SUM(ts.Price) desc; 

-- Q2. Revenue and Seat Count for New Full Seasons, and Renewal Full Seasons (separate columns for each)

SELECT
SUM(CASE WHEN TicketType = 'New' THEN ts.Price END) as 'New_FS_Total_Revenue'
, COUNT(CASE WHEN TicketType = 'New' THEN ts.SeatUniqueID END) as 'New_FS_Total_Seats'
, SUM(CASE WHEN TicketType = 'Renewal' THEN ts.Price END) as 'Renewal_FS_Total_Revenue'
, COUNT(CASE WHEN TicketType = 'Renewal' THEN ts.SeatUniqueID END) as 'Renewal_FS_Total_Seats'
FROM 
coltsdb.dbo.TicketSales ts 
WHERE ts.plan_event_name LIKE '21FS%' AND ts.TicketType IN ('New','Renewal');

-- Q3. Revenue and Seat Count for each Stadium Manifest Description

SELECT
sub.Description as 'Stadium_Manifest_Description'
, SUM(ts.Price) as 'Total_Revenue'
, COUNT(DISTINCT ts.SeatUniqueID) as 'Total_Seats'
FROM coltsdb.dbo.TicketSales ts 
LEFT JOIN (                                       -- Subquery to speed up the table join
	SELECT 
	DISTINCT m.Description
	, m.[Section]  
	FROM coltsdb.dbo.Manifest m
	) sub ON sub.Section = ts.section_name
WHERE ts.Season = '2021' AND ts.status = 'SOLD'
GROUP BY sub.Description
ORDER BY 'Total_Revenue' desc;

-- Q4. Revenue and Seats Sold per Day Leading Up to the Game (Use event date 11/4/2021)

SELECT
CONVERT(date, ts.sale_datetime) as 'Day'
, SUM(ts.Price) as 'Total Revenue'
, COUNT(ts.SeatUniqueID) as 'Total Seats'
FROM
coltsdb.dbo.TicketSales ts 
WHERE CONVERT(date, ts.event_date) = '2021-11-04' AND CONVERT(date, ts.sale_datetime) <= '2021-11-04' 
AND ts.Season = '2021' AND ts.status = 'SOLD' AND ts.sale_datetime IS NOT NULL
GROUP BY CONVERT(date, ts.sale_datetime)
ORDER BY CONVERT(date, ts.sale_datetime) DESC;

-- Email Stats
-- Q5. Percentage of 2021 ticket buyers who opened and clicked an email

SELECT
COUNT(DISTINCT ts.acct_id) as 'Total_2021_Ticket_Buyers',
	(SELECT COUNT(DISTINCT ce.id)					-- Subquery to only select buyers who clicked and opened an email
	FROM coltsdb.dbo.CampaignEngagement ce
	LEFT JOIN coltsdb.dbo.TicketSales ts
	ON CONVERT(VARCHAR(12), ce.id,1) = CONVERT(VARCHAR(12),ts.acct_id,1)
	WHERE ts.Season = '2021' AND ts.status = 'SOLD' 
	AND ce.Opened > 0 AND ce.ClickedOn > 0) 'Clicked_and_Opened_Buyers',
		CAST((SELECT COUNT(DISTINCT ce.id)			-- Subquery to calculate percentage of total buyers who clicked and opened
		FROM coltsdb.dbo.CampaignEngagement ce
		LEFT JOIN coltsdb.dbo.TicketSales ts
		ON CONVERT(VARCHAR(12), ce.id,1) = CONVERT(VARCHAR(12),ts.acct_id,1)
		WHERE ts.Season = '2021' AND ts.status = 'SOLD' 
		AND ce.Opened > 0 AND ce.ClickedOn > 0) as DECIMAL(7,2)) / (COUNT(DISTINCT ts.acct_id)) * 100 as '%_Buyers_who_Clicked_and_Opened'
FROM coltsdb.dbo.TicketSales ts
WHERE ts.Season = '2021' AND ts.status = 'SOLD';

-- Q6. Total number of Group buyers who opened or clicked an email

SELECT 
COUNT(DISTINCT(ts.acct_id))
FROM coltsdb.dbo.TicketSales ts 
LEFT JOIN coltsdb.dbo.CampaignEngagement ce 
ON CONVERT(VARCHAR(12), ce.id,1) = CONVERT(VARCHAR(12),ts.acct_id,1)
WHERE ts.Season = '2021' AND ts.status = 'SOLD' AND ts.TicketType = 'Group' AND ce.Opened > 0 
OR ts.Season = '2021' AND ts.status = 'SOLD' AND ts.TicketType = 'Group' AND ce.ClickedOn > 0;

-- Manifest
-- Q7. Overall Sell-Through Rate by Section in 2021 (Top 10 and Bottom 10 sections)

WITH temp1						-- Temporary Table 1 (Sum_Capacity)
as (
SELECT 
m.[Section] 
, SUM(m.capacity) as 'Sum_Capacity'
FROM coltsdb.dbo.manifest m
GROUP BY m.[Section] 
),
temp2							-- Temporary Table 2 (Sum_Seats)
as (
SELECT ts.section_name
, COUNT(DISTINCT ts.SeatUniqueID) as 'Sum_Seats'
FROM coltsdb.dbo.TicketSales ts
WHERE ts.Season = '2021' AND ts.status= 'SOLD'
GROUP BY ts.section_name 
),
temp3 							-- Temporary Table 3 (Sell_Through_Rate)
as (
SELECT
t2.section_name
, SUM(t1.Sum_Capacity) as 'Total_Capacity'
, SUM(t2.Sum_Seats) 'Total_Seats'
, CAST(SUM(t2.Sum_Seats) / SUM(t1.Sum_Capacity) AS DECIMAL(7,2)) * 100 as 'Sell_Through_Rate'
FROM temp1 t1
LEFT JOIN temp2 t2
ON t1.Section = t2.section_name
GROUP BY t2.section_name
),
TOP_10 (Section_Name, Sell_Through_Rate)	-- Top 10 Table
AS (
SELECT TOP(10) 
t3.section_name as 'Section_Name'
, t3.Sell_Through_Rate as 'Sell_Through_Rate'
FROM temp3 t3 
WHERE t3.section_name IS NOT NULL
GROUP BY t3.section_name, t3.Sell_Through_Rate
ORDER BY t3.Sell_Through_Rate desc
),
Bottom_10 (Section_Name, Sell_Through_Rate)  -- Bottom 10 Table
AS (
SELECT TOP(10) 
t3.section_name as 'Section_Name'
, t3.Sell_Through_Rate as 'Sell_Through_Rate'
FROM temp3 t3 
WHERE t3.section_name IS NOT NULL
GROUP BY t3.section_name, t3.Sell_Through_Rate
ORDER BY t3.Sell_Through_Rate
)
SELECT * FROM (SELECT TOP(10) * FROM TOP_10 ORDER BY Sell_Through_Rate) a  -- Union for Top and Bottom 10 with a subquery to include order by
UNION ALL
SELECT * FROM (SELECT TOP(10) * FROM Bottom_10 ORDER BY Sell_Through_Rate desc) b;

-- Q8. Sell-Through for Bottom 10 sections by Game Date in 2021

WITH temp1    -- Temporary Table 1 (Sum_Capacity)
as (
SELECT 
m.[Section] 
, SUM(capacity) as 'Sum_Capacity'
FROM coltsdb.dbo.manifest m
GROUP BY m.[Section] 
),
temp2			-- Temporary Table 2 (Sum_Seats)
as (
SELECT ts.section_name
, CONVERT(date, ts.event_date) as 'Event_Date'     -- Added ts.event_date to the Select and Group By here for future reference
, COUNT(DISTINCT ts.SeatUniqueID) as 'Sum_Seats'
FROM coltsdb.dbo.TicketSales ts
WHERE ts.Season = '2021' AND ts.status= 'SOLD'
GROUP BY ts.section_name, ts.event_date  
),
temp3  			-- Temporary Table 3 (Sell-Through Rate)
as (
SELECT
t2.section_name
, t2.event_date 
, SUM(Sum_Capacity) as 'Total_Capacity'
, SUM(Sum_Seats) 'Total_Seats'
, CAST(SUM(Sum_Seats) / SUM(Sum_Capacity) AS DECIMAL(7,2)) * 100 as 'Sell_Through_Rate'
FROM temp1 t1
LEFT JOIN temp2 t2
ON t1.Section = t2.section_name
GROUP BY t2.section_name, t2.event_date 
),
Bottom_10 (Section_Name, Event_Date, Sell_Through_Rate) -- Bottom 10 Table
as (
SELECT TOP 10 
t3.section_name as 'Section'
, t3.event_date as 'Game_Date'
, t3.Sell_Through_Rate as 'Sell_Through_Rate'
FROM temp3 t3 
WHERE t3.section_name IS NOT NULL
GROUP BY t3.section_name, t3.event_date, t3.Sell_Through_Rate
ORDER BY t3.Sell_Through_Rate
)
SELECT * FROM Bottom_10 b10 ORDER BY b10.Sell_Through_Rate; -- Select all from Bottom 10 Table

-- F&B
-- Q9. Overall Revenue by Item in 2021 (Top 5 and Bottom 5 items)

WITH temp     -- Temporary Table (Ranking Overall_Revenue)
as (
SELECT
fnb.Item
, SUM(fnb.Sales) as 'Overall_Revenue'
, ROW_NUMBER () OVER (ORDER BY SUM(fnb.Sales) asc) as asc_order_rank
, ROW_NUMBER () OVER (ORDER BY SUM(fnb.Sales) desc) as desc_order_rank
FROM coltsdb.dbo.FNB fnb
GROUP BY fnb.Item)
SELECT        -- Selecting Top 5 and Bottom 5
t.Item
, t.Overall_Revenue
FROM temp t
WHERE asc_order_rank <= 5 or desc_order_rank <= 5;

-- Q10. Section with highest ratio of Tickets Sold to Overall F&B Quantity Sold (Numbered Sections Only)

WITH fnb	-- Temporary Table
as (
SELECT
LEFT(TRIM('Location: ' FROM fnb.Concat),3) as 'Section_Name',    -- Trimming/cleaning the data
SUM(TRY_CONVERT(NUMERIC(10, 0), fnb.Actual_Qty)) as 'FNB_Quantity_Sold'  -- Converting Actual_Qty to numeric value and summing
FROM coltsdb.dbo.FNB fnb 
WHERE TRY_CONVERT(NUMERIC(10, 0), fnb.Actual_Qty) IS NOT NULL
AND ISNUMERIC (LEFT(TRIM('Location: ' FROM fnb.Concat),3)) = 1    -- Selecting numbered sections only
GROUP BY
LEFT(TRIM('Location: ' FROM fnb.Concat),3)
),
Tickets_Sold  -- Temporary Table (Number of Tickets Sold)
as (
SELECT
ts.section_name,
COUNT(ts.SeatUniqueID) as 'Tickets_Sold'
FROM coltsdb.dbo.TicketSales ts
WHERE ISNUMERIC (ts.section_name) = 1   -- Numbered sections only
AND ts.status = 'SOLD'
GROUP BY ts.section_name
)
SELECT TOP(1)	-- Selecting highest ratio 
t.section_name as 'Section_Name'
, t.Tickets_Sold / f.FNB_Quantity_Sold as 'Ratio_of_Tickets_Sold_to_Overall_FNB_Qty_Sold'
FROM Tickets_Sold t 
INNER JOIN fnb f
ON f.section_name = t.section_name
ORDER BY Ratio_of_Tickets_Sold_to_Overall_FNB_Qty_Sold desc;