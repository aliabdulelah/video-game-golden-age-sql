
# The Golden Age of Video Games — SQL Analysis
![video_game](https://github.com/aliabdulelah/The-Golden-Age-of-Video-Games/assets/129835709/2257714f-80c3-4cea-8d33-edc91095fe14)


SQL analysis of critic scores, user scores, and global sales data for the top 400 video games released between 1977 and the mid-2000s — searching for the era where commercial success and critical acclaim aligned most closely.

**Business question:** Was there a true "golden age" of video games, and does critical acclaim actually correlate with sales?

---

## Key findings

- **Top-rated years by critics** and **top-rated years by users** overlap only partially — critics and players disagreed more than expected
- **Best years for both** (where critic AND user scores were both high): a narrow band in the early-to-mid 1990s
- **Sales vs. scores:** High critic scores correlate positively with sales, but user score correlation is weaker — suggesting marketing and brand matter as much as quality

---

## Tools used

- **PostgreSQL / SQL**
- **Data source** — game sales and review dataset (top 400 games by global sales, 1977 onwards)

---

## SQL techniques demonstrated

| Technique | Applied in |
|---|---|
| `JOIN` (inner, left) | Combining sales, critic score, and user score tables |
| `GROUP BY` + `AVG` | Calculating average scores by year |
| `HAVING` | Filtering years with sufficient game count for statistical validity |
| Set operations (`INTERSECT`, `EXCEPT`) | Identifying years that appear in both top critic and top user lists |
| Subqueries | Filtering aggregated results |



For the "game_sales" table:


| column       | type   | meaning                           |
| ------------ | ------ | --------------------------------- |
| game         | varchar | Name of the video game            |
| platform     | varchar | Gaming platform                   |
| publisher    | varchar | Game publisher                    |
| developer    | varchar | Game developer                    |
| games_sold   | float  | Number of copies sold (millions)  |
| year         | int    | Release year                      |

<be>


For the "reviews" table :

  
| column       | type   | meaning                            |
| ------------ | ------ | ---------------------------------- |
| game         | varchar | Name of the video game             |
| critic_score | float  | Critic score according to Metacritic|
| user_score   | float  | User score according to Metacritic  |


<br>
<be>

### Tools 
- Excel - Data Cleaning
- PostgreSQL - Data Analysis


<br>
<be>

### Data Cleaning Preparation 

 In the initial data preparation phase, we performed the following tasks:

- Data loading and inspection.
- Handling missing values.
- Data cleaning and formatting.

<br>
<be>


### Exploratory Data Analysis
 EDA involves exploring the video Games data to answer key questions, such as:


1. The ten best-selling video games
2. Missing review scores
3. Years that video game critics loved
4. Was 1982 really that great?
5. Years that dropped off the critics' favorites list
6. Years video game players loved
7. Years that both players and critics loved
8. Sales in the best video game years

   


  
 ### Data Analysis Results Findings


1. The ten best-selling video games
Let's begin by looking at some of the top-selling video games of all time!

```sql

SELECT *
    FROM game_sales
    ORDER BY games_sold DESC
    LIMIT 10;
```
<br>

| game                                   | platform | publisher           | developer           | games_sold | year |
| -------------------------------------- | -------- | ------------------- | ------------------- | ---------- | ---- |
| Wii Sports for Wii                     | Wii      | Nintendo            | Nintendo EAD        | 82.90      | 2006 |
| Super Mario Bros. for NES              | NES      | Nintendo            | Nintendo EAD        | 40.24      | 1985 |
| Counter-Strike: Global Offensive for PC| PC       | Valve               | Valve Corporation   | 40.00      | 2012 |
| Mario Kart Wii for Wii                 | Wii      | Nintendo            | Nintendo EAD        | 37.32      | 2008 |
| PLAYERUNKNOWN'S BATTLEGROUNDS for PC   | PC       | PUBG Corporation    | PUBG Corporation    | 36.60      | 2017 |
| Minecraft for PC                       | PC       | Mojang              | Mojang AB           | 33.15      | 2010 |
| Wii Sports Resort for Wii              | Wii      | Nintendo            | Nintendo EAD        | 33.13      | 2009 |
| Pokemon Red / Green / Blue Version for GB | GB    | Nintendo            | Game Freak          | 31.38      | 1998 |
| New Super Mario Bros. for DS            | DS       | Nintendo            | Nintendo EAD        | 30.80      | 2006 |
| New Super Mario Bros. Wii for Wii      | Wii      | Nintendo            | Nintendo EAD        | 30.30      | 2009 |



Wow, the best-selling video games were released between 1985 to 2017! That's quite a range; we'll have to use data from the reviews table to gain more insight on the best years for video games.

<br>
<be>

2. Missing review scores
   
First, it's important to explore the limitations of our database. One big shortcoming is that there is not any reviews data for some of the games on the game_sales table.

```sql
SELECT COUNT(g.game)
FROM game_sales as g
LEFT JOIN reviews AS r
ON g.game = r.game 
WHERE critic_score is NULL AND user_score IS NULL

```
<br>


| count |
| ----- |
| 31    |

It looks like a little less than ten percent of the games on the game_sales table don't have any reviews data. That's a small enough percentage that we can continue our exploration, but the missing reviews data is a good thing to keep in mind as we move on to evaluating results from more sophisticated queries.

<br>
<be>
  
3. Years that video game critics loved
   
   There are lots of ways to measure the best years for video games! Let's start with what the critics think.
```sql
SELECT g.year , ROUND(AVG(r.critic_score),2) AS avg_critic_score
FROM game_sales AS g
INNER JOIN reviews AS r
ON g.game = r.game
GROUP BY year
ORDER BY avg_critic_score DESC
LIMIT 10;
```
<br>

| year | avg_critic_score |
| ---- | ---------------- |
| 1990 | 9.80             |
| 1992 | 9.67             |
| 1998 | 9.32             |
| 2020 | 9.20             |
| 1993 | 9.10             |
| 1995 | 9.07             |
| 2004 | 9.03             |
| 1982 | 9.00             |
| 2002 | 8.99             |
| 1999 | 8.93             |


<br>
<be>

The range of great years according to critic reviews goes from 1982 until 2020: we are no closer to finding the golden age of video games!

Hang on, though. Some of those avg_critic_score values look like suspiciously round numbers for averages. The value for 1982 looks especially fishy. Maybe there weren't a lot of video games in our dataset that were released in certain years.

<br>
<be>
  
4. Was 1982 really that great?
Let's update our query and find out whether 1982 really was such a great year for video games.

```sql
SELECT g.year , ROUND(AVG(r.critic_score),2) AS avg_critic_score , COUNT(g.game) AS num_games
FROM game_sales AS g
INNER JOIN reviews AS r
ON g.game = r.game
GROUP BY year
HAVING COUNT(g.game) > 4
ORDER BY avg_critic_score DESC
LIMIT 10;

```
<br>



| year | avg_critic_score | num_games |
| ---- | ---------------- | --------- |
| 1998 | 9.32             | 10        |
| 2004 | 9.03             | 11        |
| 2002 | 8.99             | 9         |
| 1999 | 8.93             | 11        |
| 2001 | 8.82             | 13        |
| 2011 | 8.76             | 26        |
| 2016 | 8.67             | 13        |
| 2013 | 8.66             | 18        |
| 2008 | 8.63             | 20        |
| 2017 | 8.62             | 13        |

<br>
<be>

That looks better! The num_games column convinces us that our new list of the critics' top games reflects years that had quite a few well-reviewed games rather than just one or two hits. But which years dropped off the list due to having four or fewer reviewed games? Let's identify them so that someday we can track down more game reviews for those years and determine whether they might rightfully be considered as excellent years for video game releases!


5-Years that dropped off the critics' favorites list

 we've created tables with the results of our previous two queries:

top_critic_years:

| column          | type  | meaning                                       |
| --------------- | ----- | --------------------------------------------- |
| year            | int   | Year of video game release                    |
| avg_critic_score| float | Average of all critic scores for games released in that year |

<br>
<be>

top_critic_years_more_than_four_games:

| column          | type  | meaning                                                   |
| --------------- | ----- | --------------------------------------------------------- |
| year            | int   | Year of video game release                                |
| num_games       | int   | Count of the number of video games released in that year  |
| avg_critic_score| float | Average of all critic scores for games released in that year |


```sql
SELECT year ,avg_critic_score
    FROM top_critic_years
    EXCEPT
    SELECT year , avg_critic_score
    FROM top_critic_years_more_than_four_games
    ORDER BY avg_critic_score DESC
```
<br>

| year | avg_critic_score |
| ---- | ---------------- |
| 1990 | 9.80             |
| 1992 | 9.67             |
| 2020 | 9.20             |
| 1993 | 9.10             |
| 1995 | 9.07             |
| 1982 | 9.00             |

Based on our work in the task above, it looks like the early 1990s might merit consideration as the golden age of video games based on critic_score alone, but we'd need to gather more games and reviews data to do further analysis.


6. Years video game players loved

   Let's move on to looking at the opinions of another important group of people: players! To begin, let's create a query very similar to the one we used in question Four, except this one will look at user_score averages by year rather than critic_score averages.


```sql
SELECT g.year , ROUND(AVG(r.user_score),2) AS avg_user_score , COUNT(g.game) AS num_games
FROM game_sales AS g
INNER JOIN reviews AS r
ON g.game = r.game
GROUP BY year
HAVING COUNT(g.game) > 4
ORDER BY avg_user_score DESC
LIMIT 10;
```
<br>


| year | avg_user_score | num_games |
| ---- | -------------- | --------- |
| 1997 | 9.50           | 8         |
| 1998 | 9.40           | 10        |
| 2010 | 9.24           | 23        |
| 2009 | 9.18           | 20        |
| 2008 | 9.03           | 20        |
| 1996 | 9.00           | 5         |
| 2005 | 8.95           | 13        |
| 2006 | 8.95           | 16        |
| 2000 | 8.80           | 8         |
| 2002 | 8.80           | 9         |

Alright, we've got a list of the top ten years according to both critic reviews and user reviews. Are there any years that showed up on both tables? If so, those years would certainly be excellent ones!

<br>

7. Years that both players and critics loved

Recall that we have access to the top_critic_years_more_than_four_games table, which stores the results of our top critic years query from Task 4:

top_critic_years_more_than_four_games:

| column          | type  | meaning                                                   |
| --------------- | ----- | --------------------------------------------------------- |
| year            | int   | Year of video game release                                |
| num_games       | int   | Count of the number of video games released in that year  |
| avg_critic_score| float | Average of all critic scores for games released in that year |


top_user_years_more_than_four_games:

| column          | type  | meaning                                                 |
| --------------- | ----- | ------------------------------------------------------- |
| year            | int   | Year of video game release                              |
| num_games       | int   | Count of the number of video games released in that year |
| avg_user_score  | float | Average of all user scores for games released in that year |

```sql
SELECT year 
FROM top_critic_years_more_than_four_games
INTERSECT
SELECT year
FROM top_user_years_more_than_four_games
```
<br>


| year |
| ---- |
| 1998 |
| 2008 |
| 2002 |

Looks like we've got three years that both users and critics agreed were in the top ten! There are many other ways of measuring what the best years for video games are, but let's stick with these years for now. We know that critics and players liked these years.

<br>

 8- Sales in the best video game years
 
 This time, we haven't saved the results from the previous task in a table for you. Instead, we'll use the query from the previous task as a subquery in this one! This is a great skill to have, as we don't always have write permissions on the database we are querying.

```sql
 SELECT g.year , SUM(g.games_sold) AS total_games_sold
FROM game_sales AS g
WHERE g.year IN (SELECT year 
FROM top_critic_years_more_than_four_games
INTERSECT
SELECT year
FROM top_user_years_more_than_four_games)
GROUP BY g.year
ORDER BY total_games_sold DESC;
```
<br>

| year | total_games_sold |
| ---- | ---------------- |
| 2008 | 175.07           |
| 1998 | 101.52           |
| 2002 | 58.67            |
---

## Files

| File | Description |
|---|---|
| `README.md` | This file |
| [Kaggle](https://www.kaggle.com/datasets/holmjason2/videogamedata) | Data |



