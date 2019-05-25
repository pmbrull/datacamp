# Foundations of Functional Programming with Purrr

## Iterations

We first need to introduce the difference of lists vs. atomic vectors. Mainly it is the underlying data structure, being *recursive* in lists but not in vectors.

If we have a list of DataFrames, we could work as follows:

```R
# Subset the first element of the sw_films data
sw_films[[1]]

# Subset the first element of the sw_films data, title column 
sw_films[[1]]$title
```



We can use `map()` instead of a `for` loop, which is a function that outputs a list. Therefore, we can just focus on the functionality of the code - What it does - instead of being too much explicit with a loop - How it does it.

```R
# Initialize list
all_files <- list()

# For loop to read files into a list
for(i in seq_along(files)){
  all_files[[i]] <- read_csv(file = files[[i]])
}
```

```R
# Load purrr library
library(purrr)

# Use map to iterate
all_files_purrr <- map(files, read_csv) 

# Output size of list object
length(all_files_purrr)
```

Moreover, we can get the length of the rows of each element of a list of dataframes:

```R
map(survey_data, ~nrow(.x))
```

We use `.x` to represent where each element of the data object should be placed. This syntax needs to be headed by a ~. We are using `map(object, ~function(.x))` instead of `map(object, function)` to specify how the list is used in the function.

However,  what if we want that into a vector instead of a list?

### Different flavours of `map()`

* `map_dbl()` outputs a vector of numbers as a result.

  ```R
  map_dbl(survey_data, ~nrow(.x))
  ```

* `map_lgl()` gives a vector of logical type

  ```R
  map_dbl(survey_data, ~nrow(.x) == 14)
  ```

* `map_chr()` returns a vector of characters

  ```R
  map_chr(species_name, ~.x)
  ```

We can use this different versions of map to create new columns for a dataframe in a cleaner way

```R
# Create a numcolors column and fill with length of each wesanderson element
data.frame(numcolors = map_dbl(wesanderson, ~length(.x)))
```

Moreover, this approach ensures type safety in a language that usually does not.

* `map_df()` helps when building dataframes:

  ```R
  bird_measurements %>%
    map_df(~data.frame(weigth = .x[["weight"]],
                       wing_length = .x[["wing length"]]))
  ```

  Which outputs a tibble where each row is an element of the list.

> Note how `map()` returns a list and `map_*()` a vector.

## Maps and Pipes

Let's add some names

```R
library(repurrrsive)
data(sw_films)

sw_films <- sw_films %>%
  set_names(map_chr(sw_films, "title"))
```

We can also use pipes with `map()`, for example for normalizing the `watefowl_data`

```R
map(waterfowl_data, ~.x %>%
                       sum() %>%
                       log())
```

## Simulating data with map

Let's suppose that we have a list of means that we want to enforce to some simulated data for testing `list_of_means`. We can create simulated dataframes as follows:

```R
list_of_df <- map(list_of_means, 
                  ~data.frame(a = rnorm(
                     mean = .x, n = 200, sd = (5/2)    
                  )))
```

## Running models

We can also use map to run linear models and summarize the results for `education_data`:

```R
models <- education_data %>%
  map(~lm(income ~ education_level, data = .x)) %>%
  map(summary)
```

## `map2()` and `pmap()`

We can also iterate over 2 lists of more and combine them into our function:

```R
simdata <- map2(list_of_means, list_of_sd,
                ~data.frame(a = rnorm(mean = .x, n = 200, sd = .y),
                            b = rnorm(mean = 200, n = 200, sd = 15)
                           )
               )
```

For 3 or more lists, we need to use `pmap()`, which involves a slightly different syntax

```R
# First create a list of lists
input_list <- list(
  means = list_of_means,
  sd = list_of_sd,
  samplesize = list_of_samplesize
)

# Then apply pmap()
simdata <- pmap(input_list,
                function(means, sd, samplesize)
                    data.frame(a = rnorm(mean = means, sd = sd, n = samplesize))
                )
```

We use a named function herein order to being able to use the names specified in the list instead of .x, .y and .z.

## Purrr safely()

When we want to map over a list and an element has the wrong datatype, then the map function won't work and it may be difficult to debug errors risen from data itself. Therefore, we can use the function `safely()` to check where those errors occur:

```R
a <- list("unkown", 10) %>%
  map(safely(function(x) x * 10, otherwise = NA_real_))
```

The resulting list will have different types *error* or *result*, whether we could run the function successfully or not. When using `safely()` we want to determine what the errors are and what elements are causing them. Thus, we could group all the error messages together:

```R
a <- list("unkown", 10) %>%
  map(safely(function(x) x * 10, otherwise = NA_real_)) %>%
  transpose()
```

## Another way to `possibly()` purrr

Before we have used `safely()` to look for errors and deal with them and we got back both result and error elements. However, we may just want to fill wrong data with NA elements and keep on with our lives. This is when we use `possibly()`.

```R
a <- list("unkown", 10) %>%
  map(possibly(function(x) x * 10, otherwise = NA_real_))
```

With this we just get the result part of the map.

## Purrr is a `walk()` in the park

`walk()` helps us print the result of a list in a much more compact and easier to read way. Is the function we should use when dealing with this kind of side-effects to also being able to notice them.

```R
walk(myList, print)
```

We can also use `walk()` to avoid the printed output when plotting. For example, in a list of plots that we want to show, we can use walk to only create the plot and not the useless console output:

```R
walk(plot_list, print)
```

## Using purrr in your workflow

Examples

```R
# Name gh_users with the names of the users
gh_users_named <- gh_users %>% 
    set_names(map_chr(gh_users, "name"))

# Check gh_repos structure
str(gh_repos)

# Name gh_repos with the names of the repo owner
gh_repos_named <- gh_repos %>% 
    map_chr(~ .[[1]]$owner$login) %>% 
    set_names(gh_repos, .)
```

```R
# Determine who joined github first
map_chr(gh_users, ~.x[["created_at"]]) %>%
      set_names(map_chr(gh_users, "name")) %>%
    sort()

# Determine who has the most public repositories
map_int(gh_users, ~.x[["public_repos"]]) %>%
    set_names(map_chr(gh_users, "name")) %>%
    sort()
```

## More complex examples

What if your values are buried? We can nest `map()` functions

```R
forks <- gh_repos %>%
  map(~map(.x, "forks"))
```

We can also create summary stats:

```R
bird_measurements %>%
  map_df(~data.frame(weigth = .x[["weight"]],
                     wing_length = .x[["wing length"]],
                     taxa = "bird")) %>%
  select_if(is.numeric) %>%
  summary(.x)
```

```R
# Map over gh_repos to generate numeric output
map(gh_repos, 
    ~map_dbl(.x, 
             ~.x[["size"]])) %>%
# Grab the largest element
map(~max(.x))
```

## Graphs in Purrr

```R
bird_measurements %>%
  map_df(~data.frame(weigth = .x[["weight"]],
                     wing_length = .x[["wing length"]])) %>%
  ggplot(aes(x = weigth, y = wing_length))
    + geom_point()
```

```R
# Turn data into correct dataframe format
film_by_character <- tibble(filmtitle = map_chr(sw_films, ~.x[["title"]])) %>%
    mutate(filmtitle, characters = map(sw_films, ~.x[["characters"]])) %>%
    unnest()

# Pull out elements from sw_people
sw_characters <- map_df(sw_people, `[`, c("height", "mass", "name", "url"))

# Join the two new objects
character_data <- inner_join(film_by_character, sw_characters, by = c("characters" = "url")) %>%
    # Make sure the columns are numbers
    mutate(height = as.numeric(height), mass = as.numeric(mass))
```











