Assignment04
================
Yuwei Wu
2022-11-18

# HPC

## Problem 1: Make sure your code is nice

Rewrite the following R functions to make them faster. It is OK (and
recommended) to take a look at Stackoverflow and Google

``` r
# Total row sums
fun1 <- function(mat) {
  n <- nrow(mat)
  ans <- double(n) 
  for (i in 1:n) {
    ans[i] <- sum(mat[i, ])
  }
  ans
}

fun1alt <- function(mat) {
  # YOUR CODE HERE
  ans<-rowSums(mat)
  ans
}

# Cumulative sum by row
fun2 <- function(mat) {
  n <- nrow(mat)
  k <- ncol(mat)
  ans <- mat
  for (i in 1:n) {
    for (j in 2:k) {
      ans[i,j] <- mat[i, j] + ans[i, j - 1]
    }
  }
  ans
}

fun2alt <- function(mat) {
  # YOUR CODE HERE
  ans <- t(apply(mat, 1, cumsum))
  ans
}

# Use the data with this code
set.seed(2315)
dat <- matrix(rnorm(200 * 100), nrow = 200)

# Test for the first
first <- microbenchmark::microbenchmark(
  fun1(dat),
  fun1alt(dat), check = "equivalent"
)
print(first, unit = "relative")
```

    ## Unit: relative
    ##          expr      min       lq     mean   median       uq       max neval
    ##     fun1(dat) 4.340385 5.280867 4.441589 5.185388 5.343585 0.5843928   100
    ##  fun1alt(dat) 1.000000 1.000000 1.000000 1.000000 1.000000 1.0000000   100

``` r
# Test for the second
sec <- microbenchmark::microbenchmark(
  fun2(dat),
  fun2alt(dat), check = "equivalent"
)
print(sec, unit = "relative")
```

    ## Unit: relative
    ##          expr      min       lq     mean   median       uq       max neval
    ##     fun2(dat) 4.434443 3.495775 2.441871 3.284241 3.024582 0.3254695   100
    ##  fun2alt(dat) 1.000000 1.000000 1.000000 1.000000 1.000000 1.0000000   100

The last argument, check = “equivalent”, is included to make sure that
the functions return the same result.

## Problem 2: Make things run faster with parallel computing

The following function allows simulating PI

``` r
sim_pi <- function(n = 1000, i = NULL) {
  p <- matrix(runif(n*2), ncol = 2)
  mean(rowSums(p^2) < 1) * 4
}

# Here is an example of the run
set.seed(156)
sim_pi(1000) # 3.132
```

    ## [1] 3.132

In order to get accurate estimates, we can run this function multiple
times, with the following code:

``` r
# This runs the simulation a 4,000 times, each with 10,000 points
set.seed(1231)
system.time({
  ans <- unlist(lapply(1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

    ## [1] 3.14124

    ##    user  system elapsed 
    ##    1.89    0.50    2.41

Rewrite the previous code using parLapply() to make it run faster. Make
sure you set the seed using clusterSetRNGStream():

``` r
# YOUR CODE HERE
cl <- makePSOCKcluster(4L) 
clusterSetRNGStream(cl, 123)
clusterExport(cl, c("sim_pi"), envir = environment())

system.time({
  ans <- unlist(parLapply(cl,1:4000, sim_pi, n = 10000)) 
  print(mean(ans))
})
```

    ## [1] 3.141482

    ##    user  system elapsed 
    ##    0.00    0.00    0.96

``` r
stopCluster(cl)
```

# SQL

Setup a temporary database by running the following chunk

``` r
# install.packages(c("RSQLite", "DBI"))

library(RSQLite)
```

    ## Warning: package 'RSQLite' was built under R version 4.2.2

``` r
library(DBI)
```

    ## Warning: package 'DBI' was built under R version 4.2.2

``` r
# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
film <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film.csv")
film_category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film_category.csv")
category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/category.csv")

# Copy data.frames to database
dbWriteTable(con, "film", film)
dbWriteTable(con, "film_category", film_category)
dbWriteTable(con, "category", category)
```

## Question 1

How many many movies is there avaliable in each rating catagory.

``` sql
SELECT rating,
  COUNT(*) AS count
FROM film
GROUP BY rating
```

| rating | count |
|:-------|------:|
| G      |   180 |
| NC-17  |   210 |
| PG     |   194 |
| PG-13  |   223 |
| R      |   195 |

5 records

There are 180 movies with the rating “G”; 210 movies with the rating
“NC-17”; 194 movies with the rating “PG”; 233 movies with the rating
“PG-13”; 195 movies with the rating “R”.

## Question 2

What is the average replacement cost and rental rate for each rating
category.

``` sql
SELECT rating,
  AVG(replacement_cost) AS avg_replcement,
  AVG(rental_rate) AS avg_rental
FROM film
GROUP BY rating
```

| rating | avg_replcement | avg_rental |
|:-------|---------------:|-----------:|
| G      |       20.12333 |   2.912222 |
| NC-17  |       20.13762 |   2.970952 |
| PG     |       18.95907 |   3.051856 |
| PG-13  |       20.40256 |   3.034843 |
| R      |       20.23103 |   2.938718 |

5 records

For the rating “G”, the average replacement cost is 20.12 and the
average rental rate is 2.91. For the rating “NC-17”, the average
replacement cost is 20.13 and the average rental rate is 2.97. For the
rating “PG”, the average replacement cost is 18.96 and the average
rental rate is 3.05. For the rating “PG-13”, the average replacement
cost is 20.40 and the average rental rate is 3.03. For the rating “R”,
the average replacement cost is 20.23 and the average rental rate is
2.94.

## Question 3

Use table film_category together with film to find the how many films
there are witth each category ID

``` sql
SELECT category_id,
  COUNT (*) AS Counts
FROM film AS f
  INNER JOIN film_category AS c
ON f.film_id = c.film_id
GROUP BY category_id
```

| category_id | Counts |
|:------------|-------:|
| 1           |     64 |
| 2           |     66 |
| 3           |     60 |
| 4           |     57 |
| 5           |     58 |
| 6           |     68 |
| 7           |     62 |
| 8           |     69 |
| 9           |     73 |
| 10          |     61 |

Displaying records 1 - 10

## Question 4

Incorporate table category into the answer to the previous question to
find the name of the most popular category.

``` sql
SELECT film_category.category_id,category.name,
  COUNT(*) AS count
FROM film_category
  INNER JOIN film ON film_category.film_id=film.film_id
  INNER JOIN category ON film_category.category_id=category.category_id
GROUP BY category.category_id
ORDER BY count DESC
```

| category_id | name        | count |
|------------:|:------------|------:|
|          15 | Sports      |    74 |
|           9 | Foreign     |    73 |
|           8 | Family      |    69 |
|           6 | Documentary |    68 |
|           2 | Animation   |    66 |
|           1 | Action      |    64 |
|          13 | New         |    63 |
|           7 | Drama       |    62 |
|          14 | Sci-Fi      |    61 |
|          10 | Games       |    61 |

Displaying records 1 - 10

The most popular category is sports.
