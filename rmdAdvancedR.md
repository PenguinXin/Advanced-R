Advanced R Notes
================

## Packages used

``` r
# bench
# bookdown
# bslib
# dbplyr
# desc
# downlit
# emo
# ggbeeswarm
# ggplot2
# jsonlite
# knitr
# lobstr
# memoise
# png
# profvis
# Rcpp
# rlang
# RSQLite
# scales
# sessioninfo
# sloop
# testthat
# tidyr
# vctrs
# xml2
# zeallot
```

# Chapter 2: Names and values

    ## Installing package into 'C:/Users/xinpeng/Documents/R/win-library/4.0'
    ## (as 'lib' is unspecified)

    ## package 'lobstr' successfully unpacked and MD5 sums checked

    ## Warning: cannot remove prior installation of package 'lobstr'

    ## Warning in file.copy(savedcopy, lib, recursive = TRUE): problem copying C:
    ## \Users\xinpeng\Documents\R\win-library\4.0\00LOCK\lobstr\libs\x64\lobstr.dll
    ## to C:\Users\xinpeng\Documents\R\win-library\4.0\lobstr\libs\x64\lobstr.dll:
    ## Permission denied

    ## Warning: restored 'lobstr'

    ## 
    ## The downloaded binary packages are in
    ##  C:\Users\xinpeng\AppData\Local\Temp\RtmpKapyNP\downloaded_packages

    ## Warning: package 'lobstr' was built under R version 4.0.3

``` r
# 1. Create a new column called "3" that contains the sum of 1 and 2
df <- data.frame(runif(3), runif(3))
names(df) <- c(1, 2)
df$`3` <- df$`1` + df$`2` 

# 2. In the following code, how much memory does y occupy?
x <- runif(1e6)
y <- list(x, x, x)
obj_size(y)
```

    ## 8,000,128 B

``` r
# 3. On which line does a get copied in the following example?
a <- c(1, 5, 3, 2)
b <- a
b[[1]] <- 10

# access an object's identifier
x <- c(1, 2, 3) # it's creating an object, a vector of values, c(1, 2, 3) and it's binding that object to a name, x
obj_addr(x) 
```

    ## [1] "0x2a5e9230"

``` r
y <- x
obj_addr(y) 
```

    ## [1] "0x2a5e9230"

``` r
# these identifiers are long, and change every time you restart R

#2.2.2 Exercises
# 1. Explain the relationship between a, b, c and d in the following code:
a <- 1:10
b <- a
c <- b
d <- 1:10

list_of_names <- list(a, b, c, d)
obj_addrs(list_of_names)
```

    ## [1] "0x14d51590" "0x14d51590" "0x14d51590" "0x14e2d860"

``` r
# a, b, c point to the same object with the same address in memory. This object has the value 1:10.
# d points to a different object with the same value

# 2. The following code accesses the mean function in multiple ways. 
# Do they all point to the same underlying function object? Verify this with lobstr::obj_addr().
# mean
# base::mean
# get("mean")
# evalq(mean)
# match.fun("mean")
## Yes. They all point to the same object
mean_functions <- list(
  mean,
  base::mean,
  get("mean"),
  evalq(mean),
  match.fun("mean")
)
obj_addrs(mean_functions)
```

    ## [1] "0x18d7e6c8" "0x18d7e6c8" "0x18d7e6c8" "0x18d7e6c8" "0x18d7e6c8"

syntactic name must consist of letters, digits, . and \_ but can’t begin
with \_ or a digit, syntactic name also can’t use reserved words like
TRUE, NULL, if, and function ?Reserved to see the complete list of
reserved words non-syntactic names are those that don’t follow the rules
you need to surround non-syntactic names with backticks \`\`

3.  By default, base R data import functions, like read.csv(), will
    automatically convert non-syntactic names to syntactic ones. Why
    might this be problematic? What option allows you to suppress this
    behaviour? Column names are often data, and the underlying
    make.names() transformation is non-invertible, so the default
    behaviour corrupts data. To avoid this, set check.names = FALSE. To
    avoid this, set check.names = FALSE

4.  What rules does make.names() use to convert non-syntactic names into
    syntactic ones? A valid name must start with a letter or a dot (not
    followed by a number) and may further contain numbers and
    underscores ("\_"s are allowed since R version 1.9.0).

<!-- end list -->

``` r
make.names("non-valid")
```

    ## [1] "non.valid"

``` r
make.names(".1")
```

    ## [1] "X.1"

``` r
make.names("  R") 
```

    ## [1] "X..R"

``` r
make.names("@") 
```

    ## [1] "X."

``` r
make.names("if")
```

    ## [1] "if."

``` r
# copy-on-modifying
x <- c(1, 2, 3)
y <- x

y[[3]] <- 4 # Modifying y didn't modify x
y
```

    ## [1] 1 2 4

``` r
# you can see when an object gets copied using tracemem()
x <- c(1, 2, 3)
cat(tracemem(x), "\n")
```

    ## <000000002ADA5738>

``` r
y <- x
y[[3]] <- 4L
```

    ## tracemem[0x000000002ada5738 -> 0x000000002ad04058]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous>

``` r
y[[3]] <- 5

untracemem(x) # turns tracing off

f <- function(a) {
  a
}

x <- c(1, 2, 3)
cat(tracemem(x), "\n")
```

    ## <000000002AD0AE28>

``` r
#> <0x7fc8257ebae8>

z <- f(x)
# there's no copy here!

untracemem(x)
```

Like vectors, lists use copy-on-modify behaviour; the original list is
left unchanged, and R creates a modified copy

To see values that are shared across lists, use lobstr::ref(). ref()
prints the memory address of each object, along with a local ID so that
you can easily cross-reference shared components.

``` r
l1 <- list(1, 2, 3)
l2 <- l1
l2[[3]] <- 4
ref(l1, l2) # the first value and the second value in both lists have the same address but the third one differs.
```

    ## o [1:0x2aa325a0] <list> 
    ## +-[2:0x2af6ee38] <dbl> 
    ## +-[3:0x2af6ee00] <dbl> 
    ## \-[4:0x2af6edc8] <dbl> 
    ##  
    ## o [5:0x13e324d0] <list> 
    ## +-[2:0x2af6ee38] 
    ## +-[3:0x2af6ee00] 
    ## \-[6:0x2af6ece8] <dbl>

If you modify a column, only that column needs to be modified; the
others will still point to their original references (still have the
same address) HOWEVER, if you modify a row, every column is modified,
which means every column must be copied

``` r
d1 <- data.frame(x = c(1, 5, 6), y = c(2, 4, 3))
d2 <- d1
d2[, 2] <- d2[, 2] * 2 
d3 <- d1
d3[1, ] <- d3[1, ] * 3 
```

R uses a global string pool; the same character will point to the same
address

``` r
x <- c("a", "a", "abc", "d")
ref(x, character = TRUE)
```

    ## o [1:0x2a6297d0] <chr> 
    ## +-[2:0x165c87c0] <string: "a"> 
    ## +-[2:0x165c87c0] 
    ## +-[3:0x197299b8] <string: "abc"> 
    ## \-[4:0x167c0e28] <string: "d">

## 2.3.6 Exercises

1.  Why is tracemem(1:10) not useful? Because you didn’t save the object
    1:10 to a name, executing this code will give you a different
    address each time

2.  Explain why tracemem() shows two copies when you run this code.
    Hint: carefully look at the difference between this code and the
    code shown earlier in the section.

x initially has integer type; now you assign a double to the third
element of x, which triggers copy-on-modifying

``` r
x <- c(1L, 2L, 3L) # we use "L" suffix to make it an explicit integer; an integer vector consumes less memory than a double numeric vector
tracemem(x)
```

    ## [1] "<000000002A61C620>"

``` r
x[[3]] <- 4
```

    ## tracemem[0x000000002a61c620 -> 0x000000002a625d60]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
    ## tracemem[0x000000002a625d60 -> 0x000000002a646740]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous>

``` r
untracemem(x)
```

3.  Sketch out the relationship between the following objects:

<!-- end list -->

``` r
a <- 1:10
b <- list(a, a)
c <- list(b, a, 1:10)

ref(c)
```

    ## o [1:0x13f59ae8] <list> 
    ## +-o [2:0x2a638360] <list> 
    ## | +-[3:0x1553a1e0] <int> 
    ## | \-[3:0x1553a1e0] 
    ## +-[3:0x1553a1e0] 
    ## \-[4:0x155ce200] <int>

``` r
ref(a)
```

    ## [1:0x1553a1e0] <int>

``` r
ref(b)
```

    ## o [1:0x2a638360] <list> 
    ## +-[2:0x1553a1e0] <int> 
    ## \-[2:0x1553a1e0]

4.  What happens when you run this code? – STILL CONFUSED

<!-- end list -->

``` r
x <- list(1:10)
x[[2]] <- x
x
```

    ## [[1]]
    ##  [1]  1  2  3  4  5  6  7  8  9 10
    ## 
    ## [[2]]
    ## [[2]][[1]]
    ##  [1]  1  2  3  4  5  6  7  8  9 10

``` r
ref(x)
```

    ## o [1:0x2a9aaef8] <list> 
    ## +-[2:0x159a8040] <int> 
    ## \-o [3:0x2adea8d8] <list> 
    ##   \-[2:0x159a8040]

``` r
obj_size(1:1e3) # R just stores the first and last number. Therefore, no matter how large, every sequence is the same size
```

    ## 680 B

``` r
obj_size(1:1e9)
```

    ## 680 B

``` r
#object.size(letters)
#obj_size(letters)
```

## 2.4.1 Exercises

1.  In the following example, why are object.size(y) and obj\_size(y) so
    radically different? Consult the documentation of object.size().

Object.size() doesn’t account for shared elements within lists

``` r
y <- rep(list(runif(1e4)), 100)

object.size(y)
```

    ## 8005648 bytes

``` r
#> 8005648 bytes
obj_size(y)
```

    ## 80,896 B

``` r
#> 80,896 B
```

2.  Take the following list. Why is its size somewhat misleading? –
    DON’T UNDERSTAND

<!-- end list -->

``` r
funs <- list(mean, sd, var)
obj_size(funs)
```

    ## 17,608 B

``` r
#> 17,608 B
```

3.  Predict the output of the following code:

<!-- end list -->

``` r
# a length-0 vector has 48 bytes of overhead
obj_size(list())
```

    ## 48 B

``` r
obj_size(double())
```

    ## 48 B

``` r
obj_size(character())
```

    ## 48 B

``` r
# a single double takes 8 bytes
obj_size(double(1)) # 48 + 8
```

    ## 56 B

``` r
obj_size(double(2)) # 48 + 8 + 8
```

    ## 64 B

``` r
# So a million double should take up 8,000,048 bytes

a <- runif(1e6)
obj_size(a)
```

    ## 8,000,048 B

``` r
# Because both b and a have the same reference address; so no additional memory is required for the second list element
# the list itself requires 64 bytes: 48 for an empty list and 8 bytes for each elements

#obj_size(vector("list", 2))
b <- list(a, a)
ref(b)
```

    ## o [1:0x29b65698] <list> 
    ## +-[2:0x7ff4fbc70010] <dbl> 
    ## \-[2:0x7ff4fbc70010]

``` r
ref(a)
```

    ## [1:0x7ff4fbc70010] <dbl>

``` r
obj_size(b)
```

    ## 8,000,112 B

``` r
b[[1]][[1]] <- 10 # NEED MORE UNDERSTANDING
obj_size(b)
```

    ## 16,000,160 B

``` r
obj_size(a, b)
```

    ## 16,000,160 B

``` r
b[[2]][[1]] <- 10 # NEED MORE UNDERSTANDING
obj_size(b)
```

    ## 16,000,160 B

``` r
obj_size(a, b)
```

    ## 24,000,208 B

The slowness of for loop is cased by every iteration of the loop
creating a copy Using a list instead of a data frame can reduce the
number of copies

``` r
library(rlang)
x <- data.frame(matrix(runif(5 * 1e4), ncol = 5))
head(x)
```

    ##          X1        X2         X3        X4         X5
    ## 1 0.2750749 0.9154945 0.16951229 0.1657623 0.75940142
    ## 2 0.2486498 0.5264314 0.12149675 0.7021707 0.48541256
    ## 3 0.7373731 0.6791692 0.03333705 0.8160656 0.02093758
    ## 4 0.4395307 0.8049804 0.21453498 0.6119096 0.81918761
    ## 5 0.2211348 0.9726916 0.94065995 0.0781211 0.04243848
    ## 6 0.4437952 0.3785991 0.15220272 0.6272415 0.16158191

``` r
medians <- vapply(x, median, numeric(1))

for (i in seq_along(medians)) {
  x[[i]] <- x[[i]] - medians[[i]]
}

cat(tracemem(x), "\n")
```

    ## <00000000146BF788>

``` r
for (i in 1:5) {
  x[[i]] <- x[[i]] - medians[[i]]
}
```

    ## tracemem[0x00000000146bf788 -> 0x00000000146b1698]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
    ## tracemem[0x00000000146b1698 -> 0x00000000146add38]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
    ## tracemem[0x00000000146add38 -> 0x00000000146ae358]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
    ## tracemem[0x00000000146ae358 -> 0x00000000146ae588]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
    ## tracemem[0x00000000146ae588 -> 0x00000000146ae6d8]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
    ## tracemem[0x00000000146ae6d8 -> 0x00000000146aeba8]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
    ## tracemem[0x00000000146aeba8 -> 0x00000000146af2a8]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
    ## tracemem[0x00000000146af2a8 -> 0x00000000146af5b8]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
    ## tracemem[0x00000000146af5b8 -> 0x00000000146abe18]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> 
    ## tracemem[0x00000000146abe18 -> 0x00000000146ac048]: [[<-.data.frame [[<- eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous>

``` r
untracemem(x)

y <- as.list(x)
cat(tracemem(y), "\n")
```

    ## <00000000146A8828>

``` r
for (i in 1:5) {
  y[[i]] <- y[[i]] - medians[[i]]
}
```

    ## tracemem[0x00000000146a8828 -> 0x000000001469de08]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous>

``` r
untracemem(y)

# environment: reference semantics
e1 <- env(a = 1, b = 2, c = 3)
e2 <- e1

# if we change a binding, the environment is modified in place
e1$c <- 4
e2$c
```

    ## [1] 4

``` r
# environments can contain themselves
e <- rlang::env()
e$self <- e
ref(e)
```

    ## o [1:0x14e3a4e8] <env> 
    ## \-self = [1:0x14e3a4e8]

## 2.5.3 Exercises

1.  Explain why the following code doesn’t create a circular list. In
    this situation copy-on-modify prevents the creation of a circular
    list \# DON’T UNDERSTAND

<!-- end list -->

``` r
x <- list()
x[[1]] <- x

obj_addr(x)
```

    ## [1] "0x2aa75e88"

``` r
tracemem(x)
```

    ## [1] "<000000002AA75E88>"

``` r
x[[1]] <- x
```

    ## tracemem[0x000000002aa75e88 -> 0x000000002aa95e50]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous>

``` r
obj_addr(x) # copied object has new memory address
```

    ## [1] "0x2aa95e50"

``` r
obj_addr(x[[1]]) # list element contains old memory address
```

    ## [1] "0x2aa75e88"

``` r
untracemem(x)
```

Garbage collector frees up memory by deleting R objects that are no
longer used, and by request more memory from the operating system if
needed R uses a tracing GC

``` r
# Garbage collector
gc()
```

    ##           used (Mb) gc trigger (Mb) max used (Mb)
    ## Ncells  620414 33.2    1417112 75.7  1417112 75.7
    ## Vcells 4199619 32.1   10146329 77.5  8385413 64.0

``` r
mem_used() # prints the total number of bytes used
```

    ## 68,337,928 B
