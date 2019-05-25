# Tidiverse

Tidyverse is a collection of data science tools within R for transforming and visualizing data. 

> A [*tibble*](https://cran.r-project.org/web/packages/tibble/vignettes/tibble.html) is just a special kind of DataFrame.

The way the `dplyr` package has to approach data is really comfortable, as we can use different *verbs* such as `filter()` and apply them to our data and chain any number of transformations by using a *pipe*

```R
gapminder %>%
  filter(year == 2007, country == "United States")
```

What makes this pattern even better is that we are not modifying the original dataset, but just generating a new one resulting from all the operations in pipes.

## More Verbs

We have seen `filter()` to select some rows from a dataset based on some logic. We also have

* `arrenge()`: sorts a table based on a variable.

  ```R
  gapminder %>%
    arrange(desc(gdpPercap))
  ```

  We can chain multiple verbs as

  ```R
  gapminder %>%
    filter(year == 1957) %>%
    arrange(desc(pop)
  ```

* `mutate()`: changes or adds variables.

  ```R
  # Change the pop column
  gapminder %>%
    mutate(pop = pop / 1000000)
  ```

  ```R
  # Add a new gdp column
  gapminder %>%
    mutate(gdp = gdpPercap * pop)
  ```

* `summarize()`:  turns many rows into one by using an aggregation function.

  ```R
  gapminder %>%
    summarize(meanLifeExp = mean(lifeExp))
  ```

* `group_by()`: Using `group_by` before `summarize` turns groups into one row each.

  ```R
  gapminder %>%
    group_by(year) %>%
    summarize(meanLifeExp = mean(lifeExp),
              totalPop = sum(pop))
  ```

  

## Visualizing

We can use the `ggplot2` library to inspect our data. The first variable will be the dataset, the second the *aesthetics*, such as x and y axis, color and size, and finally we add a layer indicating what plot we use, such a scatter plot.

```R
library(gapminder)
library(dplyr)
library(ggplot2)

gapminder_1952 <- gapminder %>%
  filter(year == 1952)

# Create a scatter plot with pop on the x-axis and lifeExp on the y-axis
ggplot(gapminder_1952, aes(x = pop, y = lifeExp)) +
  geom_point()

# If we want log scale
ggplot(gapminder_1952, aes(x = pop, y = lifeExp)) +
  geom_point() + 
  scale_x_log10() +
  scale_y_log10()

# Add color and size to comunicate more information
ggplot(gapminder_1952, aes(x = pop, y = lifeExp, color = continent, size = pop)) +
  geom_point() + 
  scale_x_log10() 
```

### Faceting

Divide a graph into smaller subplots.

```R
ggplot(gapminder_1952, aes(x = pop, y = lifeExp)) +
  geom_point() + 
  scale_x_log10() +
  facet_wrap(~ continent)
```

### Starting y-axis at zero

```R
by_year <- gapminder %>%
  group_by(year) %>%
  summarize(medianLifeExp = median(lifeExp),
            maxGdpPercap = max(gdpPercap))

ggplot(by_year, aes(x = year, y = totalPop)) +
  geom_point() +
  expand_limits(y = 0)
```

### Line Plot

```R
ggplot(gapminder_1952, aes(x = pop, y = lifeExp)) +
  geom_line() +
  expand_limits(y = 0)
```

### Bar Plot

```R
ggplot(gapminder_1952, aes(x = pop, y = lifeExp)) +
  geom_col()
```

### Histograms

```R
gapminder_1952 <- gapminder %>%
  filter(year == 1952)

# Create a histogram of population (pop)
ggplot(gapminder_1952, aes(x = pop)) +
   geom_histogram(binwidth = 5)
```

### Boxplots

```R
ggplot(gapminder_1952, aes(x = continent, y = gdpPercap)) +
  geom_boxplot() +
  scale_y_log10() +
  ggtitle("Comparing GDP per capita across continents")
```





















