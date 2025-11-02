# Netflix Movies and TV show data analyis using sql
![Netflix Logo](https://github.com/AshwinRaj-U/netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives
Analyze the distribution of content types (movies vs TV shows).
Identify the most common ratings for movies and TV shows.
List and analyze content based on release years, countries, and durations.
Explore and categorize content based on specific criteria and keywords.

--1. Count the number of Movies vs TV Shows

```sql
select
	show_type,
	count(*) as movie_count
from netflix
group by show_type;
```

--2. Find the most common rating for movies and TV shows

```sql
select 
	show_type,
	rating,
	total_times
from 
(select
	show_type,
	rating,
	count(*) as total_times,
	rank() over (partition by show_type order by count(*) desc ) as ranks
from netflix
group by show_type,rating
order by show_type asc) as Rating_table 
where ranks=1;
```
--3. List all movies released in a specific year (e.g., 2020)

```sql
select 
	*
from netflix
where 
	release_year=2020 
	and 
	show_type='Movie';
```
--4. Find the top 5 countries with the most content on Netflix

```sql
select 
	trim(unnest(string_to_array(country,','))),
	count(*) as no_of_movies_released
from netflix
group by 1
order by 2 desc;

```
--5. Identify the longest movie

```sql
select
	title,
	TRIM(REGEXP_REPLACE(duration, '[^0-9]+', '', 'g'))::INT AS duration_minutes
from netflix
where 
	show_type='Movie'
	and
	duration is not null
order by 2 desc;
```
--6. Find content added in the last 5 years
```sql
select 
	*
from netflix
where 
to_date(date_added,'Month DD,YYYY') >= current_date - Interval '5 Years';
```

--7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

```sql
select
	*
from netflix
where director like ('%Rajiv Chilaka%');
```
--8. List all TV shows with more than 5 seasons

```sql
select 
	 * 
from netflix
where show_type='TV Show'
	  and
	  split_part(duration,' ',1)::INT>5
```
--9. Count the number of content items in each genre

```sql
select 
	trim(unnest(string_to_array(listed_in,','))) as Genre,
	count(*) as no_of_content
from netflix
group by 1
order by 2 desc;
	
```
--10.Find each year and the average numbers of content release in India on netflix return top 5 year with highest avg content release!

```sql
select 
	Released_year,
	count(*) as no_of_content,
	round(count(*) *1.0 /(select count(*) from netflix where country like ('%India%')) *100.0,2) as Percentage
from (select 
	*,
	extract(year from date_added::date) as Released_year,
	trim(unnest(string_to_array(country,','))) as ctry
from netflix) t1
where ctry='India'
group by 1
order by 2 desc
limit 5
```
--11. List all movies that are documentaries

```sql
select * from netflix
where listed_in like ('%Documentaries%')
and show_type='Movie';
```

--12. Find all content without a director

```sql
select * from netflix where director is null;
```
--13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
```sql
select *
from netflix
where show_type='Movie'
and release_year >= extract(year from current_date) - 10
and casts like ('%Salman Khan%');
```
--14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
```sql
select 
	unnest(string_to_array(casts,',')) as Actors,
	count(*)
from netflix
where country = 'India'
and show_type = 'Movie'
group by 1
order by 2 desc
limit 10;
```
--15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.
```sql
select 
	case
		when description ilike '%kill%' then 'Bad'
		when description ilike '%violence%' then 'Bad'
		else 'Good'
		end,
	count(*)
from netflix
group by 1;
```

