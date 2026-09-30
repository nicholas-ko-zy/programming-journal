# R

## Stats

- [Useful blog post on `dnorm`, `pnorm`, `qnorm` functions in R](https://seankross.com/notes/dpqr/)

## Common things I do

- Check for na values 
```r
# In a specific column
df %>% 
  filter(is.na(your_col))

# Across all columns
df %>% 
    filter(if_any(everything(), is.na))
```

- Replace NA values with something
```r
df %>% 
  mutate(your_col = replace_na(your_col, 0), # suppose you want to replace with 0
         )
```
