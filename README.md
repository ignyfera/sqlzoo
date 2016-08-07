# sqlzoo

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
WHERE  gdp > ALL(SELECT gdp FROM world
                 WHERE  continent='Europe' AND gdp IS NOT NULL)
AND gdp IS NOT NULL;
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
WHERE  name <= all (SELECT name FROM world y
                    WHERE  y.continent=x.continent AND name IS NOT NULL)
```

### 9.
```
SELECT name, continent, population FROM world x
WHERE  25000000 >= ALL(SELECT population FROM world y
                        WHERE y.continent=x.continent)
ORDER BY continent
```

### 10. Some countries have populations more than three times that of any of their neighbours (in the same continent). Give the countries and continents.
```
SELECT name, continent FROM world x
WHERE population >= ALL(SELECT 3*population FROM world y
                         WHERE y.continent=x.continent AND x.name != y.name)
```
