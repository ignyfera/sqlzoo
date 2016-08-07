# sqlzoo

This README contains selected solutions to sqlzoo.net.

## SELECT basics

### 1. Show the population of Germany
```
SELECT population FROM world
 WHERE name='Germany';
```

### 2. Show the name and the population for 'Ireland', 'Iceland' and 'Denmark'.
```
SELECT name, population FROM world
 WHERE name IN ('Ireland', 'Iceland', 'Denmark');
```

## SELECT within SELECT

### 6. Which countries have a GDP greater than every country in Europe? (Some countries may have NULL gdp values.)
```
SELECT name FROM world
 WHERE gdp > ALL(SELECT gdp FROM world
                 WHERE  continent='Europe' AND gdp IS NOT NULL) AND
       gdp IS NOT NULL;
```

### 7. Find the largest country (by area) in each continent; show the continent, the name and the area.
```
SELECT continent, name, area FROM world x
 WHERE area >= ALL (SELECT area FROM world y
                     WHERE y.continent=x.continent AND area >= 0)
```

```
SELECT continent, name, area FROM world x
 WHERE area >= ALL (SELECT area FROM world y
                     WHERE y.continent=x.continent AND area IS NOT NULL)
```

### 8. List each continent and the name of the country that comes first alphabetically.
```
SELECT continent, name FROM world x
 WHERE name <= all (SELECT name FROM world y
                     WHERE y.continent=x.continent AND name IS NOT NULL)
```

### 9. Find the continents where all countries have a population <= 25000000. Then find the names of the countries associated with these continents. Show name, continent and population.
```
SELECT name, continent, population FROM world x
 WHERE 25000000 >= ALL(SELECT population FROM world y
                        WHERE y.continent=x.continent)
ORDER BY continent
```

### 10. Some countries have populations more than three times that of any of their neighbours (in the same continent). Give the countries and continents.
```
SELECT name, continent FROM world x
 WHERE population >= ALL(SELECT 3*population FROM world y
                          WHERE y.continent=x.continent AND x.name != y.name)
```

## SUM and COUNT

### 3. Give the total GDP of Africa.
```
SELECT SUM(gdp) FROM world
 WHERE continent='Africa'
```

### 4. How many countries have an area of at least 1000000.
```
SELECT COUNT(*) FROM world
 WHERE area >= 1000000 AND area IS NOT NULL
```

### 6. For each continent show the continent and number of countries.
```
SELECT continent, COUNT(name) FROM world
GROUP BY continent
```

### 7. For each continent show the continent and number of countries with populations of at least 10 million.
```
SELECT continent, COUNT(name) FROM world
 WHERE population > 10000000 AND population IS NOT NULL
GROUP BY continent
```

### 8. List the continents that have a total population of at least 100 million.
```
SELECT continent FROM (SELECT continent, SUM(population) as population
                         FROM world y
                     GROUP BY continent) x
 WHERE x.population >= 100000000
```

```
SELECT continent FROM world
GROUP BY continent
HAVING SUM(population) > 100000000
```

## JOIN

### 4. Show the team1, team2 and player for every goal scored by a player called Mario player LIKE 'Mario%'.
```
SELECT game.team1, game.team2, goal.player
  FROM game JOIN goal ON game.id=goal.matchid
 WHERE goal.player LIKE 'Mario %'
```

### 5. Show player, teamid, coach, gtime for all goals scored in the first 10 minutes, gtime<=10.
```
SELECT goal.player, goal.teamid, eteam.coach, goal.gtime
  FROM goal JOIN eteam ON goal.teamid=eteam.id
 WHERE goal.gtime<=10
```

### 6. List the the dates of the matches and the name of the team in which 'Fernando Santos' was the team1 coach.
```
SELECT mdate, teamname
  FROM game JOIN eteam ON game.team1=eteam.id
 WHERE eteam.coach='Fernando Santos'
```

### 7. List the player for every goal scored in a game where the stadium was 'National Stadium, Warsaw'.
```
SELECT player
  FROM goal JOIN game ON goal.matchid=game.id
 WHERE stadium='National Stadium, Warsaw'
```

### 8. Show the name of all players who scored a goal against Germany.
```
SELECT DISTINCT player
  FROM game JOIN goal ON game.id=goal.matchid
 WHERE (game.team1='GER' OR game.team2='GER') AND goal.teamid!='GER'
```

### 9. Show teamname and the total number of goals scored.
```
SELECT teamname, COUNT(*) as goals
  FROM eteam JOIN goal ON eteam.id=goal.teamid
GROUP BY teamname
ORDER BY teamname
```

### 10. Show the stadium and the number of goals scored in each stadium.
```
SELECT stadium, COUNT(*) AS goals
  FROM game JOIN goal ON game.id=goal.matchid
GROUP BY stadium
ORDER BY stadium
```

### 11. For every match involving 'POL', show the matchid, date and the number of goals scored.
```
SELECT matchid, mdate, COUNT(*) as goals
  FROM game JOIN goal ON game.id=goal.matchid
 WHERE (team1='POL' OR team2='POL')
GROUP BY matchid, mdate
```

### 12. For every match where 'GER' scored, show matchid, match date and the number of goals scored by 'GER'
```
SELECT matchid, mdate, COUNT(*) as goals
  FROM game JOIN goal ON game.id=goal.matchid
 WHERE goal.teamid='GER'
GROUP BY matchid, mdate
```

### 13. List every match with the goals scored by each team as shown.
```
SELECT mdate, team1, SUM(score1) AS score1, team2, sum(score2) AS score2
  FROM (SELECT mdate,
          team1,
          CASE WHEN teamid=team1 THEN 1 ELSE 0 END score1,
          team2,
          CASE WHEN teamid=team2 THEN 1 ELSE 0 END score2
        FROM game LEFT OUTER JOIN goal ON game.id=goal.matchid) x
GROUP BY mdate, team1, team2
ORDER BY mdate, team1, team2
```

## More JOIN

### 7. Obtain the cast list for 'Casablanca'.
```
SELECT actor.name 
FROM actor JOIN (SELECT casting.actorid
                 FROM   movie JOIN casting ON movie.id=casting.movieid
                 WHERE title='Casablanca') x on actor.id=x.actorid
```

### 9. List the films in which 'Harrison Ford' has appeared.
```
SELECT title FROM movie
 WHERE id IN (SELECT movieid FROM casting
              WHERE actorid=(SELECT id FROM actor
                             WHERE  name='Harrison Ford'))
```

### 10. List the films where 'Harrison Ford' has appeared but not in the starring role.
```
SELECT title FROM movie
 WHERE id IN (SELECT movieid FROM casting
              WHERE actorid=(SELECT id FROM actor
                             WHERE  name='Harrison Ford' and ord!=1))
```

### 11. List the films together with the leading star for all 1962 films.
```
SELECT x.title, actor.name
  FROM actor JOIN (SELECT movie.id, movie.title, casting.actorid
                 FROM movie JOIN casting ON movie.id=casting.movieid
                 WHERE movie.yr=1962 and ord=1) x ON actor.id=x.actorid
```

### 12. Which were the busiest years for 'John Travolta'? Show the year and the number of movies he made each year for any year in which he made more than 2 movies.
```
SELECT movie.yr, COUNT(movie.title) as count
  FROM movie JOIN casting ON movie.id=casting.movieid
             JOIN actor   ON casting.actorid=actor.id
 WHERE actor.name='John Travolta'
GROUP BY movie.yr
HAVING COUNT(movie.title) > 2
```

### 13. List the film title and the leading actor for all of the films 'Julie Andrews' played in.
```
SELECT x.title, actor.name
  FROM (SELECT DISTINCT movie.id, movie.title
        FROM movie JOIN casting ON movie.id=casting.movieid
                   JOIN actor   ON casting.actorid=actor.id
       WHERE actor.name='Julie Andrews') x
        JOIN casting ON x.id=casting.movieid
        JOIN actor   ON casting.actorid=actor.id
 WHERE casting.ord=1
```

### 14. Obtain a list, in alphabetical order, of actors who've had at least 30 starring roles.
```
SELECT actor.name
  FROM actor JOIN casting ON actor.id=casting.actorid
 WHERE casting.ord=1
GROUP BY actor.name
HAVING COUNT(movieid) >= 30
ORDER BY actor.name
```

### 15. List the films released in the year 1978 ordered by the number of actors in the cast.
```
SELECT movie.title, COUNT(casting.actorid) as count
  FROM movie JOIN casting ON movie.id=casting.movieid
 WHERE movie.yr=1978
GROUP BY movie.title
ORDER BY COUNT(casting.actorid) desc
```

### 16. List all the people who have worked with 'Art Garfunkel'.
```
SELECT actor.name
  FROM actor JOIN casting ON actor.id=casting.actorid
 WHERE casting.movieid IN (SELECT casting.movieid
                             FROM casting JOIN actor ON casting.actorid=actor.id
                            WHERE actor.name='Art Garfunkel') AND
       actor.name!='Art Garfunkel'
```

## Misc.

### 1. List the teachers who have NULL for their department.
```
SELECT name FROM teacher
 WHERE dept IS NULL
```

### 2. Note the INNER JOIN misses the teachers with no department and the departments with no teacher.
```
SELECT teacher.name, dept.name as dept
  FROM teacher INNER JOIN dept ON teacher.dept=dept.id
```

### 3. Use a different JOIN so that all teachers are listed.
```
SELECT teacher.name, dept.name as dept
  FROM teacher LEFT OUTER JOIN dept on teacher.dept=dept.id
```

### 4. Use a different JOIN so that all departments are listed.
```
SELECT teacher.name, dept.name as dept
  FROM teacher RIGHT OUTER JOIN dept on teacher.dept=dept.id
```

### 6. Use the COALESCE function and a LEFT JOIN to print the teacher name and department name. Use the string 'None' where there is no department.
```
SELECT teacher.name, COALESCE(dept.name, 'None') as dept
  FROM teacher LEFT OUTER JOIN dept on teacher.dept=dept.id
```

### 7. Use COUNT to show the number of teachers and the number of mobile phones.
```
SELECT COUNT(name), COUNT(mobile)
  FROM teacher
```

### 8. Use COUNT and GROUP BY dept.name to show each department and the number of staff. Use a RIGHT JOIN to ensure that the Engineering department is listed.
```
SELECT dept.name, COUNT(teacher.name) as teachers
  FROM teacher RIGHT OUTER JOIN dept ON teacher.dept=dept.id
GROUP BY dept.name
```

### 9. Use CASE to show the name of each teacher followed by 'Sci' if the teacher is in dept 1 or 2 and 'Art' otherwise.
```
SELECT name,
       CASE WHEN dept=1 OR dept=2 THEN 'Sci'
            ELSE 'Art'
       END
  FROM teacher
```

### 10. Use CASE to show the name of each teacher followed by 'Sci' if the teacher is in dept 1 or 2, show 'Art' if the teacher's dept is 3 and 'None' otherwise.
```
SELECT name,
       CASE WHEN dept=1 OR dept=2 THEN 'Sci'
            WHEN dept=3 THEN 'Art'
            ELSE 'None'
       END
  FROM teacher
```

## UK National Student Survey, 2012

### 4. Show the subject and total number of students who responded to question 22 for each of the subjects '(8) Computer Science' and '(H) Creative Arts and Design'.
```
SELECT subject, SUM(response)
  FROM nss
 WHERE question='Q22'
   AND (subject='(8) Computer Science' OR
        subject='(H) Creative Arts and Design')
GROUP BY subject
```

### 6. Show the percentage of students who A_STRONGLY_AGREE to question 22 for the subject '(8) Computer Science' show the same figure for the subject '(H) Creative Arts and Design'. Use the ROUND function to show the percentage without decimal places.
```
SELECT subject, ROUND(100*SUM(response*A_STRONGLY_AGREE/100)/SUM(response),0)
  FROM nss
 WHERE question='Q22'
   AND (subject='(8) Computer Science' OR
        subject='(H) Creative Arts and Design')
GROUP BY subject
```
