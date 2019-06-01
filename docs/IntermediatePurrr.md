# Intermediate Functional Programming with Purrr


```R
# Extract the "urls_url" elements, and flatten() the result
urls_clean <- map(rstudioconf, "urls_url") %>%
  flatten()

# Remove the NULL
compact_urls <- compact(urls_clean)

# Create a mapper that detects the patten "github"
has_github <- as_mapper(~ str_detect(.x, "github"))

# Look for the "github" pattern, and sum the result
has_github( compact_urls, has_github ) %>%
  sum()
```
```R
# Complete the function
ratio_pattern <- function(vec, pattern){
  n_pattern <- str_detect(vec, pattern) %>%
    sum()
  n_pattern / length(vec)
}

# Create flatten_and_compact()
flatten_and_compact <- compose(compact, flatten)

# Complete the pipe to get the ratio of URLs with "github"
map(rstudioconf, "urls_url") %>%
  flatten_and_compact() %>% 
  ratio_pattern("github")
```

```R
# Create mean_above, a mapper that tests if .x is over 3.3
mean_above <- as_mapper(~ .x > 3.3)

# Prefil map_at() with "retweet_count", mean_above for above, 
# and mean_above negation for below
above <- partial(map_at, .at = "retweet_count", .f = mean_above )
below <- partial(map_at, .at = "retweet_count", .f = negate(mean_above) )

# Map above() and below() on non_rt, keep the "retweet_count"
ab <- map(non_rt, above) %>% map("retweet_count")
bl <- map(non_rt, below) %>% map("retweet_count")

# Compare the size of both elements
length(ab)
length(bl)
```
```R
# Get the max() of "retweet_count" 
max_rt <- map_dbl(non_rt, "retweet_count") %>% 
  max()

# Prefill map_at() with a mapper testing if .x equal max_rt
max_rt_calc <- partial(map_at, .at = "retweet_count", .f = ~ .x == max_rt )

# Map max_rt_calc on non_rt, keep the retweet_count & flatten
res <- map(non_rt, max_rt_calc) %>% 
  map("retweet_count") %>% 
  flatten()

# Print the "screen_name" and "text" of the result
print(res$screen_name)
print(res$text)

```