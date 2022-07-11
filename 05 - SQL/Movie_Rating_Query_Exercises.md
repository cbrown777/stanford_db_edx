# SQL Movie-Rating Query Exercises Answers

These are my solutions to the SQL Movie Rating Query exercises
You can view the database here: https://courses.edx.org/asset-v1:StanfordOnline+SOE.YDB-SQL0001+2T2020+type@asset+block/moviedata.html


## Q1

Find the titles of all movies directed by Steven Spielberg.

```sql
select title
from Movie
where director = "Steven Spielberg";
```

## Q2

Find all years that have a movie that received a rating of 4 or 5, and sort 
them in increasing order. 

```sql
select distinct year
from Movie, Rating
where (stars = 4 or stars = 5) and Movie.mID = Rating.mID
order by year;
```

## Q3

Find the titles of all movies that have no ratings. 

```sql
select title
from Movie
where Movie.mID not in (select mID from Rating);
```

## Q4

Some reviewers didn't provide a date with their rating. Find the names of all reviewers 
who have ratings with a NULL value for the date. 

```sql
select name
from Reviewer
where Reviewer.rID in (select rID from Rating where ratingDate is null);
```

## Q5

Write a query to return the ratings data in a more readable format: reviewer name, movie title, 
stars, and ratingDate. Also, sort the data, first by reviewer name, then by movie title, and 
lastly by number of stars. 

```sql
select Reviewer.name, Movie.title, Rating.stars, Rating.ratingDate
from ((Movie join Rating using(mID)) join Reviewer using(rID))
order by name, title, stars;
```

## Q6

For all cases where the same reviewer rated the same movie twice and gave it a higher 
rating the second time, return the reviewer's name and the title of the movie. 

```sql
select name, title
from Reviewer join 
	(select R1.rID, R2.mID
	from Rating R1, Rating R2
	where R1.rID = R2.rID and R1.mID = R2.mID and R1.stars < R2.stars and R1.ratingDate < R2.ratingDate)
	using(rID)
	join Movie using(mID);
```

## Q7

For each movie that has at least one rating, find the highest number of stars that movie 
received. Return the movie title and number of stars. Sort by movie title. 

```sql
select title, stars
from Movie join (select mID, max(stars) as stars from Rating group by mID) using(mID)
order by title;
```

## Q8

For each movie, return the title and the 'rating spread', that is, the difference between 
highest and lowest ratings given to that movie. Sort by rating spread from highest to 
lowest, then by movie title. 

```sql
select title, ratingSpread
from Movie join (select mID, max(stars) - min(stars) as ratingSpread from Rating group by mID) using(mID)
order by ratingSpread desc, title;
```

## Q9

Find the difference between the average rating of movies released before 1980 and the 
average rating of movies released after 1980. (Make sure to calculate the average rating 
for each movie, then the average of those averages for movies before 1980 and movies 
after. Don't just calculate the overall average rating before and after 1980.) 

```sql

select 

(select avg(avgStars)
from (select *, avg(stars) as avgStars from Rating join Movie using(mID) group by mID)
where year < 1980)
-
(select avg(avgStars)
from (select *, avg(stars) as avgStars from Rating join Movie using(mID) group by mID)
where year > 1980)
;
```
