# SQL-Scripts
This Repository contains some of my SQL Projects

## Athlete Events Data Analysis
A Dive into the Athlete Events Data and querying it to discover important insights on the winning and award statistics and trends.

![Case 1 pix](https://media.self.com/photos/6659efa72e2794fa6f47f803/4:3/w_1600,c_limit/GettyImages-846563176.jpg?fit=scale)

## 📑 Catalogue
- 📖 Overview / Introduction
- 📁 Dataset
- ❓ Case Study Questions
- ⚒️ Skills & Techniques
- ♻️ Queries / Solutions

## 📖 Overview / Introduction
The Olympic Games (a major international sporting event featuring summer and winter sports competitions) is an event held every four years, involving thousands of athletes from around the world, who represent their countries in various sporting competitions.
This Project seeks to explore the vast amount of events data (which has been collected decades over), uncovering insights amongst the competing countries, across the various games, and analyzing them to profer descriptive analogies as well as providing automated processes to fish out important statistics.

## 📂 Dataset
The Dataset used for this project is [Olympics Event Dataset](https://www.kaggle.com/datasets/semaaabumousa/athlete-eventscsv) available on Kaggle
/*
## ❓ Case Study Questions
<details><summary>Open List of Questions</summary>
<p> 

1.	How many olympics games have been held?
2.	List down all Olympics games held so far.
3.	Mention the total no of nations who participated in each olympics game?
4.	Which year saw the highest and lowest no of countries participating in olympics?
5.	Which nation has participated in all of the olympic games?
6.	Identify the sport which was played in all summer olympics.
7.	Which Sports were just played only once in the olympics?
8.	Fetch the total no of sports played in each olympic games.
9.	Fetch details of the oldest athletes to win a gold medal.
10.	Find the Ratio of male and female athletes participated in all olympic games.
11.	Fetch the top 5 athletes who have won the most gold medals.
12.	Fetch the top 5 athletes who have won the most medals (gold/silver/bronze).
13.	Fetch the top 5 most successful countries in olympics. Success is defined by no of medals won.
14.	List down total gold, silver and broze medals won by each country.
15.	List down total gold, silver and broze medals won by each country corresponding to each olympic games.
16.	Identify which country won the most gold, most silver and most bronze medals in each olympic games.
17.	Identify which country won the most gold, most silver, most bronze medals and the most medals in each olympic games.
18.	Which countries have never won gold medal but have won silver/bronze medals?
19.	In which Sport/event, India has won highest medals.
20.	Break down all olympic games where india won medal for Hockey and how many medals in each olympic games.
21.	Create a function that returns the Athlete(s) with the highest amount of medals. Make the Medal type (Gold, Silver or Bronze) be in the input as well as the positions required (1st, 2nd and/or 3rd)
22.	Create a similar function like the one above but for the Nations (NOC) with the highest amount of medals. While, putting the function to use, return with it the region and notes from the noc_regions table.


</p>
</details>

## ⚒️ Skills & Techniques
- Data Cleaning & Exploration
- Data Aggregation & Segmentation
- Window Functions
- Sub-queries and Common Table Expressions (CTEs)
- Joins and Function Creation

## ♻️ Queries / Solutions
<details><summary>View List of Queries</summary>
<p>

1. How many olympics games have been held?

```bash
SELECT COUNT(DISTINCT Games) total_games
FROM athlete_events;
```

2. List down all olympic games so far.

```bash
SELECT DISTINCT Games olympic_games
FROM athlete_events;
```

3. Mention the total no of nations who participated in each olympics game?

```bash
SELECT DISTINCT Games olympic_game
     , COUNT(DISTINCT NOC) total_no_of_nations
FROM athlete_events
GROUP BY Games;
```

4. Which year saw the highest and lowest no of countries participating in olympics?

```bash
WITH Country_sub AS (
		SELECT Year
         , COUNT(DISTINCT NOC) country_count
    FROM athlete_events
    GROUP BY Year    )
SELECT Year, country_count 
	     , CASE WHEN country_count = (SELECT MAX(country_count) FROM Country_sub) THEN 'Highest'
              WHEN country_count = (SELECT MIN(country_count) FROM Country_sub) THEN 'Lowest'
              ELSE NULL END AS Level
FROM Country_sub
WHERE country_count = (SELECT MIN(country_count)
					             FROM Country_sub)	OR 
	    country_count = (SELECT MAX(country_count)
					             FROM Country_sub)
ORDER BY country_count DESC;
```

5. Which nation has participated in all of the olympic games?

```bash
WITH total_olym_game AS (
		SELECT NOC
         		, COUNT(DISTINCT Games) no_of_olympic_games
		FROM athlete_events 
		GROUP BY NOC
			) 

SELECT *
FROM total_olym_game
WHERE no_of_olympic_games = (SELECT COUNT(DISTINCT Games) total_games
				FROM athlete_events
                            );
```

6. Identify the sport which was played in all summer olympics

```bash
WITH total_olym_game AS (
	SELECT Sport
	     , COUNT(DISTINCT Games) no_of_summer_games
  FROM athlete_events
  WHERE Games like '%Summer'
  GROUP BY Sport	)

SELECT *
FROM total_olym_game
WHERE no_of_summer_games = (SELECT COUNT(DISTINCT Games) total_games
			            FROM athlete_events
			            WHERE Games like '%Summer'
                            );
```

7. Which Sports were just played only once in the olympics?

```bash
WITH total_olym_game AS (
	SELECT Sport
	       , COUNT(DISTINCT Games) no_of_olym_games
	FROM athlete_events
  	GROUP BY Sport
			)

SELECT *
FROM total_olym_game
WHERE no_of_olym_games = 1;
```

8. Fetch the total no of sports played in each olympic games

```bash
SELECT Games
     , COUNT(DISTINCT Sport) no_of_sport
FROM athlete_events
GROUP BY Games;
```

9. Fetch details of the oldest athletes to win a gold medal

```bash
WITH gold_medalist as (	
		SELECT *
		FROM athlete_events
		WHERE Medal = 'Gold'
			)
SELECT *
FROM gold_medalist
WHERE Age = (   SELECT MAX(Age)
		FROM gold_medalist
		);
```

10. Find the Ratio of male and female athletes participated in all olympic games

``` bash
SELECT Sex
	, COUNT(Sex) as total_participants
FROM athlete_events
GROUP BY Sex;
```

11. Fetch the top 5 athletes who have won the most gold medals

```bash
SELECT TOP 5 --WITH TIES	-- To indicate ties among participants and avoid involuntary bias selection
	[Name]
	, COUNT(Medal) total_gold_medals
FROM athlete_events
WHERE Medal = 'Gold'
GROUP BY [Name]
ORDER BY COUNT(Medal) DESC;
```

12. Fetch the top 5 athletes who have won the most medals (gold/silver/bronze)

```bash
SELECT TOP 5 -- WITH TIES	-- To indicate ties among participants
	[Name]
    , COUNT(Medal) total_gold_medals
FROM athlete_events
WHERE Medal != 'NA'
GROUP BY [Name]
ORDER BY COUNT(Medal) DESC
;
```

13. Fetch the top 5 most successful countries in olympics. Success is defined by no of medals won

```bash
SELECT TOP 5 WITH TIES
      NOC
    , COUNT(Medal) medals_won
FROM athlete_events
WHERE Medal != 'NA'
GROUP BY NOC
ORDER BY COUNT(Medal) DESC;
```

14. List down total gold, silver and bronze medals won by each country.

```bash
SELECT NOC
     , COUNT(CASE WHEN Medal = 'Gold' THEN 1 ELSE NULL END) Gold
     , COUNT(CASE WHEN Medal = 'Silver' THEN 1 ELSE NULL END) Silver
     , COUNT(CASE WHEN Medal = 'Bronze' THEN 1 ELSE NULL END) Bronze
FROM athlete_events
GROUP BY NOC;
```

15. List down total gold, silver and bronze medals won by each country corresponding to each olympic games

```bash
SELECT NOC, Games
     , COUNT(CASE WHEN Medal = 'Gold' THEN 1 ELSE NULL END) Gold
     , COUNT(CASE WHEN Medal = 'Silver' THEN 1 ELSE NULL END) Silver
     , COUNT(CASE WHEN Medal = 'Bronze' THEN 1 ELSE NULL END) Bronze
FROM athlete_events
GROUP BY NOC, Games
ORDER BY RANK() OVER(PARTITION BY Games ORDER BY NOC ASC) ;
```

16. Identify which country won the most gold, most silver and most bronze medals in each olympic games

```bash
WITH medals_won AS (
		SELECT NOC, Games
		 , COUNT(CASE WHEN Medal = 'Gold' THEN 1 ELSE NULL END) Gold
		 , COUNT(CASE WHEN Medal = 'Silver' THEN 1 ELSE NULL END) Silver
		 , COUNT(CASE WHEN Medal = 'Bronze' THEN 1 ELSE NULL END) Bronze
		FROM athlete_events
		GROUP BY NOC, Games
                    )
SELECT NOC, Games, Gold, Silver, Bronze
FROM (	SELECT *, ROW_NUMBER() OVER(PARTITION BY Games ORDER BY  Gold DESC
					                        , Silver DESC
                                            			, Bronze DESC) row_no
	FROM medals_won
      ) sub
WHERE row_no = 1;
```

17. Identify which country won the most gold, most silver, most bronze medals and the most medals in each olympic games

``` bash
WITH medals_won AS (
			SELECT NOC, Games
			, COUNT(CASE WHEN Medal = 'Gold' THEN 1 ELSE NULL END) Gold
			, COUNT(CASE WHEN Medal = 'Silver' THEN 1 ELSE NULL END) Silver
			, COUNT(CASE WHEN Medal = 'Bronze' THEN 1 ELSE NULL END) Bronze
			FROM athlete_events
			GROUP BY NOC, Games
			)

SELECT NOC, Games, Gold, Silver, Bronze, total_medal
FROM	(	SELECT *
			, ROW_NUMBER() OVER(PARTITION BY Games ORDER BY total_medal DESC) totals_row_no
		FROM 	(SELECT * 
				, ROW_NUMBER() OVER(PARTITION BY Games ORDER BY Gold DESC) Gold_row_no
				, ROW_NUMBER() OVER(PARTITION BY Games ORDER BY Silver DESC) Silver_row_no
				, ROW_NUMBER() OVER(PARTITION BY Games ORDER BY Bronze DESC) Bronze_row_no 
				, (Gold + Silver + Bronze) total_medal
			FROM medals_won
			) sub
	) sub_outer
WHERE Gold_row_no = 1 OR Silver_row_no = 1 OR Bronze_row_no = 1 OR totals_row_no = 1
;

-- Another easily understandable but slightly longer query can be written using multiple CTEs (as shown below);
WITH medals_won AS (
    SELECT 
        NOC 
      , Games
      , COUNT(CASE WHEN Medal = 'Gold' THEN 1 END) Gold
      , COUNT(CASE WHEN Medal = 'Silver' THEN 1 END) Silver
      , COUNT(CASE WHEN Medal = 'Bronze' THEN 1 END) Bronze
    FROM athlete_events
    GROUP BY NOC, Games
                    ),

medals_with_total AS (
    SELECT *
         , Gold + Silver + Bronze AS total_medals
    FROM medals_won
                      ),

ranked_medals AS (
    SELECT  *
          , ROW_NUMBER() OVER (PARTITION BY Games ORDER BY Gold DESC) AS gold_rank,
          , ROW_NUMBER() OVER (PARTITION BY Games ORDER BY Silver DESC) AS silver_rank,
          , ROW_NUMBER() OVER (PARTITION BY Games ORDER BY Bronze DESC) AS bronze_rank,
          , ROW_NUMBER() OVER (PARTITION BY Games ORDER BY total_medals DESC) AS total_rank
    FROM medals_with_total
                  )
SELECT 
    NOC
  , Games
  , Gold
  , Silver
  , Bronze
  , total_medals
FROM ranked_medals
WHERE gold_rank = 1 
   OR silver_rank = 1 
   OR bronze_rank = 1 
   OR total_rank = 1;
```

18. Which countries have never won gold medal but have won silver/bronze medals?

```bash
SELECT sub.NOC
FROM	(	SELECT NOC
		, COUNT(CASE WHEN Medal = 'Gold' THEN 1 ELSE NULL END) Gold
		, COUNT(CASE WHEN Medal = 'Silver' THEN 1 ELSE NULL END) Silver
		, COUNT(CASE WHEN Medal = 'Bronze' THEN 1 ELSE NULL END) Bronze
		FROM athlete_events
		GROUP BY NOC	) sub
WHERE Gold = 0;
```

19. In which Sport/event, India has won highest medals

```bash
WITH india AS 
		(SELECT 
	NOC, Sport, [Event]
	, COUNT(CASE WHEN Medal = 'Gold' THEN 1 ELSE NULL END) Gold
	, COUNT(CASE WHEN Medal = 'Silver' THEN 1 ELSE NULL END) Silver
	, COUNT(CASE WHEN Medal = 'Bronze' THEN 1 ELSE NULL END) Bronze
FROM athlete_events
WHERE NOC = 'IND'
GROUP BY NOC, Sport, [Event]
		)
SELECT *
FROM india
WHERE Gold = (SELECT MAX(Gold) FROM india) 
  OR  Silver = (SELECT MAX(Silver) FROM india)
  OR  Bronze = (SELECT MAX(Bronze) FROM india);
```

20. Break down all olympic games where india won medal for Hockey and how many medals in each olympic games

```bash
WITH india_medals AS (
    SELECT NOC
         , Games
      	 , COUNT(CASE WHEN Medal = 'Gold' THEN 1 ELSE NULL END) Gold
      	 , COUNT(CASE WHEN Medal = 'Silver' THEN 1 ELSE NULL END) Silver
      	 , COUNT(CASE WHEN Medal = 'Bronze' THEN 1 ELSE NULL END) Bronze
    FROM athlete_events
    WHERE NOC = 'IND'
      AND Sport = 'Hockey'
    GROUP BY NOC, Games
	       		)
SELECT *
FROM india_medals
WHERE Gold != 0 
  OR  Silver <> 0
  OR  Bronze != 0;
```

21. Create a function that returns the Athlete(s) with the highest amount of medals. Make the Medal type (Gold, Silver or Bronze) be in the input as well as the positions required (1st, 2nd and/or 3rd)
```bash
CREATE OR ALTER FUNCTION top_3_medalist
	(
	@Medal VARCHAR(10),
	@rnk1 INT,
	@rnk2 INT = 1,
	@rnk3 INT = 1 
	)
RETURNS TABLE
AS
RETURN
	WITH TopMedalist AS 
	(	SELECT [Name], Medal,
		COUNT(Medal) AS MedalCount
		FROM athlete_events
		WHERE Medal = @Medal
		GROUP BY [Name], Medal
	)
	SELECT *
	FROM (
		SELECT [Name], Medal, MedalCount,
		DENSE_RANK() OVER (ORDER BY MedalCount desc) MedalistRanks
			FROM TopMedalist
		) sub
	WHERE MedalistRanks IN (@rnk1, @rnk2, @rnk3)
	;
END
-- Function Creation ends here.. Run the above query separately

-- Testing Function
-- Selecting Top 3
SELECT * FROM top_3_medalist('Gold',1,2,3);

-- Selecting only the first and second position
SELECT * FROM top_3_medalist('Gold',1,2,DEFAULT);

-- Selecing only the first position
SELECT * FROM top_3_medalist('Gold',1,DEFAULT,DEFAULT);
```

22. Create a similar function like the one above but for the Nations (NOC) with the highest amount of medals. While, putting the function to use, return with it the region and notes from the noc_regions table.
```bash
CREATE FUNCTION top_nations_overall
	(
	@pos1 INT,
	@pos2 INT = 1,
	@pos3 INT = 1
	)
RETURNS TABLE
AS
RETURN
WITH TopNoc_total AS 
	(	SELECT [NOC],
		COUNT(Medal) AS TotalMedal
		FROM athlete_events
		WHERE Medal != 'NA'
		GROUP BY [NOC]
		)
SELECT *
FROM (
		SELECT [NOC], TotalMedal,
			DENSE_RANK() OVER (ORDER BY TotalMedal desc) Position
		FROM TopNoc_total
	) sub
WHERE Position IN (@pos1, @pos2, @pos3)
;

-- Testing the function
SELECT *
FROM top_nations_overall(1,2,3) AS nat
INNER JOIN noc_regions AS ncr
	ON nat.NOC = ncr.NOC
ORDER BY Position ;

------------------------------------------------------------------
-- If we want to know the NOC with the highest individual medals 
CREATE FUNCTION top_nations 
	(
	@Medal VARCHAR (10),
	@rank1 INT,
	@rank2 INT = 1,
	@rank3 INT = 1
	)
RETURNS TABLE
AS
RETURN
WITH TopNoc AS 
	(	SELECT [NOC], Medal,
		Count(Medal) AS MedalCount
		FROM athlete_events
		WHERE Medal = @Medal
		GROUP BY [NOC], Medal
	)
SELECT *
FROM (
		SELECT [NOC], Medal, MedalCount,
			DENSE_RANK() OVER (ORDER BY MedalCount desc) MedalRanks
		FROM TopNoc
	) sub
WHERE MedalRanks IN (@rank1, @rank2, @rank3)
;

-- Testing Function
-- Selecting Top 3 nations with the highest gold medals, also indicating their region and notes (if present)
SELECT nat.*,
	ncr.region AS Region,
	ncr.notes AS Notes
FROM top_nations('Gold',1,2,3) nat
INNER JOIN noc_regions AS ncr
	ON nat.NOC = ncr.NOC
ORDER BY MedalRanks ASC;
```

</p>
</details>

-------------


-------------
