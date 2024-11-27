## Olympic-Dataset-EDA

Datasets 

[Download Dataset from Google Drive](https://drive.google.com/drive/folders/16Te0zG1W1oqAt9usAohhw4-GaSTNweC6?usp=drive_link)

[View the interactive jupyter notebook file](https://nbviewer.org/github/Nishk700/Olympic-Dataset-EDA/blob/main/Olympics_dataset_EDA._1.ipynb)

### Analysis Using SQL 

#### Table name -  olympic

Here's a summary of each column:

ID: A unique identifier for each athlete, ensuring that individuals can be tracked across multiple entries if they participated in multiple events or Olympics.

Name: The full name of each athlete, allowing for identification and analysis of individual performance across events or years.

Sex: The gender of the athlete, often recorded as "M" for male and "F" for female.

Age: The age of the athlete at the time of the Olympic Games, which can offer insights into age distributions across different sports or events.

Height: The height of the athlete in centimeters, which can be used to analyze the physical profiles of athletes in different sports.

Weight: The weight of the athlete in kilograms, another physical characteristic that may vary significantly by sport.

Team: The name of the team or country represented by the athlete, useful for tracking national performance and representation.

NOC: The National Olympic Committee (NOC) code, a three-letter abbreviation representing the country (e.g., "USA" for the United States).

Games: A textual representation of the specific Olympics (e.g., "2016 Summer" or "2012 Winter"), combining the year and season.

Year: The year of the Olympic Games, useful for identifying the edition of the Olympics and trends over time.

Season: The season of the Olympic Games, either "Summer" or "Winter," distinguishing between the two major Olympic competitions.

City: The host city of the Olympic Games, providing context on location and possibly influencing performance or participation.

Sport: The general category of sport (e.g., "Athletics," "Swimming"), enabling analysis by sport type.

Event: The specific event within the sport (e.g., "100m Freestyle," "Marathon"), useful for more granular analysis within each sport.

Medal: The medal won by the athlete ("Gold," "Silver," "Bronze," or null if no medal was won).

Region: Country each athlete represented.

### Questions

### 1. How many olympics games have been held?

Select count(distinct games) as total_games 

from olympic

(1.png)

### 2. List down all Olympics games held so far?

SELECT distinct (games)

FROM olympic

### 3. Mention the total no of nations who participated in each olympics game?

SELECT games, count (distinct noc) FROM olympic

group by games

order by games asc

### 4. Which year saw the highest and lowest no of countries participating in olympics?

SELECT games, count (distinct noc) total_countries FROM olympic

group by games

order by total_countries desc

Limit 1


SELECT games, count (distinct noc) total_countries FROM olympic

group by games

order by total_countries asc

Limit 1


### 5.  Which nation has participated in all of the olympic games?

SELECT noc

FROM olympic

GROUP BY noc

HAVING COUNT(DISTINCT year) = (

    SELECT COUNT(DISTINCT year) 
    
    FROM olympic
    
)



### 6. Identify the sport which was played in all summer olympics.

SELECT sport

FROM olympic

WHERE season = 'Summer'

GROUP BY sport

HAVING COUNT(DISTINCT year) = (

    SELECT COUNT(DISTINCT year)
    
    WHERE season = 'Summer'
    
)

### 7. Which Sports were just played only once in the olympics?

SELECT sport

FROM olympic

GROUP BY sport

HAVING COUNT(DISTINCT year) = 1


### 8. Fetch the total no of sports played in each olympic games.

select games,count (distinct sport) as number_sports

from olympic

group by games


### 9. Fetch details of the oldest athletes to win a gold medal.
    
select *

from olympic

where age <> 'NA'

and medal = 'Gold'

order by age desc

limit 1


### 10. Find the Ratio of male and female athletes participated in all olympic games.

with cte1 as

(select distinct games, count( name) over (partition by games) as males

from olympic

where sex = 'M'

order by games ),

cte2 as 

(select distinct games, count( name) over (partition by games) as females

from olympic

where sex = 'F'

order by games )

select cte1.games,cte1.males,cte2.females, cte1.males/cte2.females as ratio from cte1

left join cte2

on cte1.games = cte2.games

### 11. Fetch the top 5 athletes who have won the most gold medals.
    
select distinct (name), medal, count (medal) over (partition by name) total

from olympic

where medal = 'Gold'

order by total desc

limit 5

### 12. Fetch the top 5 athletes who have won the most medals (gold/silver/bronze).

select distinct (name), count (medal) over (partition by name) total

from olympic

order by total desc

limit 5

### 13. Fetch the top 5 most successful countries in olympics. Success is defined by no of medals won.

select distinct (olympic.noc),noc_regions.region,count (olympic.medal) over (partition by olympic.noc) total

from olympic

left join noc_regions

on olympic.noc = noc_regions.noc

order by total desc

limit 5

### 14. List down total gold, silver and broze medals won by each country.

SELECT noc,

  COUNT(CASE WHEN medal = 'Gold' THEN 1 END) AS total_gold,
  
  COUNT(CASE WHEN medal = 'Silver' THEN 1 END) AS total_silver,
  
  COUNT(CASE WHEN medal = 'Bronze' THEN 1 END) AS total_bronze,
  
  COUNT(CASE WHEN medal IN ('Gold', 'Silver', 'Bronze') THEN 1 END) AS total_medals
  
FROM olympic

group by noc

ORDER BY total_medals DESC;


### 15. List down total gold, silver and broze medals won by each country corresponding to each olympic games.

SELECT noc,games,  

    COUNT(CASE WHEN medal = 'Gold' THEN 1 END) AS total_gold,
    
    COUNT(CASE WHEN medal = 'Silver' THEN 1 END) AS total_silver,
    
    COUNT(CASE WHEN medal = 'Bronze' THEN 1 END) AS total_bronze,
    
    COUNT(CASE WHEN medal IN ('Gold', 'Silver', 'Bronze') THEN 1 END) AS total_medals
    
FROM olympic

group by noc,games

having COUNT(CASE WHEN medal IN ('Gold', 'Silver', 'Bronze') THEN 1 END) <> 0

ORDER BY games desc,total_medals DESC



### 16. Identify which country won the most gold, most silver and most bronze medals in each olympic games.


with cte1 as

(SELECT noc as Countries,games,  

    COUNT(CASE WHEN medal = 'Gold' THEN 1 END) AS total_gold,
    
    COUNT(CASE WHEN medal = 'Silver' THEN 1 END) AS total_silver,
    
    COUNT(CASE WHEN medal = 'Bronze' THEN 1 END) AS total_bronze,
    
    COUNT(CASE WHEN medal IN ('Gold', 'Silver', 'Bronze') THEN 1 END) AS total_medals
    
FROM olympic

group by noc,games

having COUNT(CASE WHEN medal IN ('Gold', 'Silver', 'Bronze') THEN 1 END) <> 0

ORDER BY games desc,total_medals DESC)

,

cte2 as

(select games, Max(total_gold) as max_gold, Countries

from cte1 

group by games,countries

order by games, max_gold desc

),

cte3 as 

( select games, max_gold, countries as highest_gold, case when (rank() over (partition by games order by max_gold desc ) =1 )then 1 else 0 end as Rank_Gold 

from cte2

),

cte4 as

(select games, Max(total_silver) as max_silver, Countries

from cte1 

group by games,countries

order by games, max_silver desc

),

cte5 as 

( select games, max_silver, countries as highest_silver, case when (rank() over (partition by games order by max_silver desc ) =1 )then 1 else 0 end as Rank_silver 

from cte4
),

cte6 as

(select games, Max(total_bronze) as max_bronze, Countries

from cte1 

group by games,countries

order by games, max_bronze desc
),

cte7 as 

( select games, max_bronze, countries as highest_bronze, case when (rank() over (partition by games order by max_bronze desc ) =1 )then 1 else 0 end as Rank_bronze

from cte6

)
select cte3.games,cte3.max_gold,cte3.highest_gold, cte5.max_silver,cte5.highest_silver,

cte7.max_bronze,cte7.highest_bronze

from cte3 

left join cte5 on cte3.games = cte5.games

left join cte7 on cte5.games = cte7.games

where  cte3.Rank_Gold = 1

and cte5.Rank_silver =1

and Rank_bronze = 1

order by cte3.games desc


### 17.  Which countries have never won gold medal but have won silver/bronze medals?

with cte1 as 

(SELECT noc as Countries,  

    COUNT(CASE WHEN medal = 'Gold' THEN 1 END) AS total_gold,
    
    COUNT(CASE WHEN medal = 'Silver' THEN 1 END) AS total_silver,
    
    COUNT(CASE WHEN medal = 'Bronze' THEN 1 END) AS total_bronze
   
FROM olympic

group by noc)


select cte1.countries,noc_regions.region as country_name,cte1.total_gold,cte1.total_silver,cte1.total_bronze

from cte1

join noc_regions

on cte1.countries = noc_regions.noc

where total_gold =0

and total_silver >0 and total_bronze > 0 


### 18. In which top ten  Sport/event, Germany has won highest medals.
    
select sport, count(medal) as total_medals from olympic

where noc = 'GER'

group by sport,noc

order by total_medals desc

limit 10


























































