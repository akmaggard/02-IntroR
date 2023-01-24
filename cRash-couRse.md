Programming cRash couRse
================
Austin Hart

-   <a href="#objects" id="toc-objects">Objects</a>
-   <a href="#packages" id="toc-packages">Packages</a>
-   <a href="#directories" id="toc-directories">Directories</a>
-   <a href="#pipes" id="toc-pipes">Pipes</a>
-   <a href="#data-io" id="toc-data-io">Data I/O</a>
-   <a href="#explore" id="toc-explore">Explore</a>

Dive into the basics of R programming for data analytics. For those with
background in R, use this as an opportunity to get new perspective on
familiar functions, object structures, and more. For those without the
background, try to think about the operations below in terms of syntax
and function rather than as a fixed set of routines to memorize.

For everyone, note two things: 
-   I use generic object and element names
wherever possible. E.g., `summary(mydata$col.name)` references a generic
object, `mydata`, and a generic column named `col.name`. 
-   You need to load `tidyverse` before executing much of what’s here.

## Objects

R is an Object Oriented Language (OOL), and objects hold primary
importance in the way we write and execute code in R. If you want R to
record some information (rather than just print it out in the console),
you have to store it in a named object. If you want to call on or use
stored information, you have to call the object by name. As a matter of
style: keep names short, intuitive, and free of spaces.[^1]

Objects come in many flavors: vectors, matrices, data frames, lists, and
more. Object names are case sensitive. We’ll typically work with data
frames (and a special kind of frame called a `tibble`), but there are
moments where knowing the difference between a data frame and a vector
is really important.

``` r
# Vectors/values
  a = rnorm(100) # check: is.vector(a)
  b = rep(letters[1:10],10)
  
# Data frames/tibbles
  df1 = data.frame(a, b)
  df2 = tibble(a,b) # requires tidyverse
```

How to you call information stored inside an object? Know that R is very
rectangular in its thinking about objects: they’re `n` elements long
and/or `k` elements wide. A `tibble` recording grades for fifteen
students on four problem sets is a 15x4 rectangular object. A `list` of
2 `tibbles` and a `vector` is a 3x1 list. If you know this, you can
access information with reference to it’s location in the rectangle.

Notice below the difference between extraction and subsetting. Pay
special attention here to the type of object each creates relative to
the type of object you start with.

``` r
# Extract an item from list or frame
  object$item
  list$item$subitem... # for heirarchical lists
  object[['item']]
  object %>% 
    pull(item)
  
# Subset an element (maintains orig structure)
  object[row.num,col.num] #defaults to object[col.num]
  object['col.name'] # use quotes to call by name
  object[1:30, 4:5] # select rows 1-30, variables 4-5
  object %>%
    select(col1, col2, col3) %>%
    filter(criterion)
```

## Packages

How do you access the incredible functionality of user-written packages?
After you install a package, you can use it by attaching it for the
whole session (`library`) or calling it on the fly using the `::`
operator.[^2]

``` r
# Installation: a one-time deal
  install.packages('pack.name')

# attaching a library: for a full session
  library(pack.name)
  
# accessing on the fly
  pack.name::funkshun
  
# get help
  ?pack.name
  ?funkshun
```

## Directories

Where does your project live? Data analyses have life cycles that go
well beyond how we think about class notes or a script for some problem
set. Your project needs a folder where you can pull and push data, code,
figures, tables, etc. So make sure you set your directory in *every*
session.

``` r
# Check and set your project directory
  getwd() # where's my current directory?
  setwd("filepath/forward slashes")
  setwd("~/folder/subfolder") # to branch within
  
# what all's in there?
  list.files()
```

## Pipes

Pipes are awesome and *totally* intuitive… once you understand them. The
basic pipe, `%>%`, comes from the `magrittr` package and attaches
automatically with `tidyverse`. Pipes push the output from one operation
forward as the input for another operation. It saves you from creating
and tracking loads of intermediate objects. Compare:

``` r
# with pipes
  p1 =
    mydata %>% # start with my data
    filter(group == 'A') %>%
    count(var1) %>% # tabulate scores on var1
    mutate(percent = 100*n/sum(n)) %>% # add a percent column
    ggplot(aes(x = var1, y = percent)) +
    geom_col()

# Without a pipe
  d1 = filter(data = mydata, group == 'A')
  d2 = count(data = d1, var1)
  d3 = mutate(data = d2, percent = 100*n/sum(n))
  p1 = ggplot(data = d3, aes(x = var1, y = percent)) + 
    geom_col()
    rm(d1,d2,d3)
```

Notice in comparing these sequences where the “piped” output fits in the
next line of code. For example, `count(data = mydata, var1)` becomes
`mydata %>% count(x = ., var1)` where `mydata` flows directly into `.`.
And you can simplify to `mydata %>% count(var1)`.

Not everything plays nicely with this basic pipe, especially some common
functions in `base` R. If you’re feeling spicy, you can use the
exposition pipe `%$%` as a workaround.

``` r
# won't work:
  mydata %>%
    lm(var1 ~ var2)

# Bring that exposition with you:
  library(magrittr)
  mydata %$%
    lm(var1 ~ var2)
```

## Data I/O

How do you get that data frame into R so that you can go to work? One of
the best parts of R is it’s capacity to input all types of data. Simply
identify the type of data you have (look at the extension!) and choose
the right command. Don’t forget about outputting, or saving, the data
too. Write it into your preferred format.[^3]

``` r
# Import R object (data, list, etc)
  load('file.RData')
  
# Other inputs (must input as a new object)
  df1 = read_csv(file = 'file.csv', na = c("","."))
  df2 = readxl::read_excel(file = 'file.xlsx')
  df3 = haven::read_dta(file = 'file.dta')
  
# Output
  save(df1, 'newdata.RData')
  write_csv(df1, 'newdata.csv')
```

## Explore

Get those hands dirty! Check the data structure. Take a peak at the raw
data. Get a quick summary of the variables.

``` r
# Structure and head
  str(mydata)
  head(mydata)
  
# Explore variables
  summary(mydata)
```

### Character and factor variables

What to do with these non-numeric variables? Start with a relative
frequency table and bar chart.

``` r
# Tabulation
  mytab = 
    mydata %>%
    count(var.name or criterion) %>%
    mutate(percent = 100 * n/sum(n))

  # Beautify the table for printing
    mytab %>%
      knitr::kable(digits = 1) # optional last step

# Graphing
  p1 = 
    mytab %>%
    ggplot(mytab, aes(x = var.name, y = percent)) +
    geom_col()
```

### Numeric and integer variables

Get those summary statistics and make another picture. Notice the
flexibility of `summarise` versus a quicker but inflexible `summary`.

``` r
# Summary stats
  summary(mydata[['var.name']])
  mydata %>% # get summary stats
    summarise(
      stat1 = some_function(var.name),
      stat2 = 100 * var.name / (var.name + other.var)
    )

# Plot the distribution or ranks    
  p2 = 
    mydata %>%
    ggplot(mydata, aes(x = var.name)) +
    geom_histogram() # compare to geom_boxplot()
```

[^1]: Strictly speaking, you can assign any name to an object. However,
    if it has spaces or other special/illegal characters, you’ll have to
    wrap it in backquotes, `` ` ``, every time you call it:
    `` `$hi++y Name` ``.

[^2]: Attaching is great for packages you want to use repeatedly (e.g.,
    `tidyverse`). The `::` operator is excellent for something you may
    only use once (e.g., `haven::read_dta()`). `::` is necessary in the
    odd case where two packages have the same function and you need to
    use, say, `stats::lag()` and not `dplyr::lag()`.

[^3]: One important caveat is that writing to `.csv` will lose any
    factor ordering and/or `haven` labeling that you created.
