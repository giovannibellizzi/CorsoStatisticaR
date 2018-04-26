Convertire le colonne in `numeric`
================

-   [**tl.dr.**](#tl.dr.)
-   [Facciamo le cose a modino:](#facciamo-le-cose-a-modino)
-   [1. importare tutte le colonne come testo:](#importare-tutte-le-colonne-come-testo)
-   [2. convertire le colonne in `numeric`](#convertire-le-colonne-in-numeric)
-   [3. individuare i dati persi:](#individuare-i-dati-persi)
-   [4. sostituire i dati sbagliati con quelli giusti:](#sostituire-i-dati-sbagliati-con-quelli-giusti)
-   [5. profit](#profit)

**tl.dr.**
----------

Lettura di un file con problemi:

``` r
data <- read.csv2("colonne_numeriche_01.csv")
```

    ## Warning in read.table(file = file, header = header, sep = sep,
    ## quote = quote, : incomplete final line found by readTableHeader on
    ## 'colonne_numeriche_01.csv'

``` r
str(data)
```

    ## 'data.frame':    3 obs. of  6 variables:
    ##  $ numbers    : Factor w/ 3 levels "0","1.1","2.2": 1 2 3
    ##  $ with_commas: num  0 1.1 2.2
    ##  $ with_nas   : Factor w/ 3 levels "","0","2.2": 2 1 3
    ##  $ with_blanks: Factor w/ 3 levels " ","0","2.2": 2 1 3
    ##  $ with_text  : Factor w/ 3 levels "0","2.2","foo": 1 3 2
    ##  $ with_errors: Factor w/ 3 levels "0","2.2","l.l": 1 3 2

``` r
data
```

    ##   numbers with_commas with_nas with_blanks with_text with_errors
    ## 1       0         0.0        0           0         0           0
    ## 2     1.1         1.1                            foo         l.l
    ## 3     2.2         2.2      2.2         2.2       2.2         2.2

#### Trasformazione delle colonne non numeriche, **occhio!!! `warnings`!!!**

``` r
library(magrittr)
wrong_cols <- c("numbers", "with_nas", "with_blanks", "with_text", "with_errors")
data[, wrong_cols] %<>% lapply(function(x) as.numeric(as.character(x)))
```

    ## Warning in FUN(X[[i]], ...): si è prodotto un NA per coercizione

    ## Warning in FUN(X[[i]], ...): si è prodotto un NA per coercizione

Controllo del risultato:

``` r
str(data)
```

    ## 'data.frame':    3 obs. of  6 variables:
    ##  $ numbers    : num  0 1.1 2.2
    ##  $ with_commas: num  0 1.1 2.2
    ##  $ with_nas   : num  0 NA 2.2
    ##  $ with_blanks: num  0 NA 2.2
    ##  $ with_text  : num  0 NA 2.2
    ##  $ with_errors: num  0 NA 2.2

``` r
data
```

    ##   numbers with_commas with_nas with_blanks with_text with_errors
    ## 1     0.0         0.0      0.0         0.0       0.0         0.0
    ## 2     1.1         1.1       NA          NA        NA          NA
    ## 3     2.2         2.2      2.2         2.2       2.2         2.2

Come vedete vengono comunque fuori un sacco di magagne, il comando è **corretto se il database di partenza non ha problemi**, ma...

Facciamo le cose a modino:
--------------------------

#### Il file `.csv`

È buona norma prima di caricare un file, controllare che faccia ha, nel modo più diretto possibile.
Il mio editor di testo, [Atom](https://atom.io/), lo mostra così:

<img src="csv_atom.png" width="500px" />

La prima riga è chiaramente composta dai nomi delle colonne (che sono 6), di seguito le 3 righe di dati.
La seconda riga mostra chiaramente, o meno, una serie di problemi piuttosto comuni (che sono scritti anche nei nomi delle colonne):

-   Decimali separati con il punto e con la virgola
-   Valori mancanti (quello è una regola)
-   Spazi bianchi (quasi impossibili da vedere, oppure altri caratteri che indicano dati mancanti)
-   Annotazioni di testo inframmezzate ai numeri (anche quello è una regola)
-   Lettere scritte per sbaglio al posto dei numeri: `o` anziché `0`, `1o` ancziché `10`, `l` al posto di `1`, capita più spesso di quanto si possa pensare!

**NB:** la seconda colonna ha i decimali separati da virgola anche sulla terza riga, questo perché ci fosse stato il punto si sarebbe comportata esattamente come l'ultima colonna.

### Caricamento del file

Il file di partenza è quindi un normale `.csv`, separato da **punti e virgola** (*semicolon separated*). La funzione standard per importarlo è `read.csv2`.

``` r
data <- read.csv2("colonne_numeriche_01.csv")
```

    ## Warning in read.table(file = file, header = header, sep = sep,
    ## quote = quote, : incomplete final line found by readTableHeader on
    ## 'colonne_numeriche_01.csv'

``` r
str(data)
```

    ## 'data.frame':    3 obs. of  6 variables:
    ##  $ numbers    : Factor w/ 3 levels "0","1.1","2.2": 1 2 3
    ##  $ with_commas: num  0 1.1 2.2
    ##  $ with_nas   : Factor w/ 3 levels "","0","2.2": 2 1 3
    ##  $ with_blanks: Factor w/ 3 levels " ","0","2.2": 2 1 3
    ##  $ with_text  : Factor w/ 3 levels "0","2.2","foo": 1 3 2
    ##  $ with_errors: Factor w/ 3 levels "0","2.2","l.l": 1 3 2

``` r
data
```

    ##   numbers with_commas with_nas with_blanks with_text with_errors
    ## 1       0         0.0        0           0         0           0
    ## 2     1.1         1.1                            foo         l.l
    ## 3     2.2         2.2      2.2         2.2       2.2         2.2

Come si vede, la funzione ha fatto un *casino*: ha automaticamente convertito gli *spazi* in *NA*, e ok, ma ha trovato delle virgole, e ha immaginato che il file fosse nel tradizionale formato europeo, intrpretando il punto come testo. Le colonne in cui compare del testo, **compresi i punti** quindi sono convertite automaticamente in `factor`.

`tidyverse` ci offre una funzione con un rendiconto più esplicito di cosa sta succedendo, anche questa però fa un macello, perdendo i punti e inventandosi dei numeri!!

``` r
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 2.2.1     ✔ purrr   0.2.4
    ## ✔ tibble  1.4.2     ✔ dplyr   0.7.4
    ## ✔ tidyr   0.8.0     ✔ stringr 1.3.0
    ## ✔ readr   1.1.1     ✔ forcats 0.2.0

    ## ── Conflicts ───────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ tidyr::extract()   masks magrittr::extract()
    ## ✖ dplyr::filter()    masks stats::filter()
    ## ✖ dplyr::lag()       masks stats::lag()
    ## ✖ purrr::set_names() masks magrittr::set_names()

``` r
data <- read_csv2("colonne_numeriche_01.csv")
```

    ## Using ',' as decimal and '.' as grouping mark. Use read_delim() for more control.

    ## Parsed with column specification:
    ## cols(
    ##   numbers = col_number(),
    ##   with_commas = col_double(),
    ##   with_nas = col_number(),
    ##   with_blanks = col_number(),
    ##   with_text = col_character(),
    ##   with_errors = col_character()
    ## )

``` r
str(data)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    3 obs. of  6 variables:
    ##  $ numbers    : num  0 11 22
    ##  $ with_commas: num  0 1.1 2.2
    ##  $ with_nas   : num  0 NA 22
    ##  $ with_blanks: num  0 NA 22
    ##  $ with_text  : chr  "0" "foo" "2.2"
    ##  $ with_errors: chr  "0" "l.l" "2.2"
    ##  - attr(*, "spec")=List of 2
    ##   ..$ cols   :List of 6
    ##   .. ..$ numbers    : list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_number" "collector"
    ##   .. ..$ with_commas: list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
    ##   .. ..$ with_nas   : list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_number" "collector"
    ##   .. ..$ with_blanks: list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_number" "collector"
    ##   .. ..$ with_text  : list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   .. ..$ with_errors: list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   ..$ default: list()
    ##   .. ..- attr(*, "class")= chr  "collector_guess" "collector"
    ##   ..- attr(*, "class")= chr "col_spec"

``` r
data
```

    ## # A tibble: 3 x 6
    ##   numbers with_commas with_nas with_blanks with_text with_errors
    ##     <dbl>       <dbl>    <dbl>       <dbl> <chr>     <chr>      
    ## 1     0          0         0           0   0         0          
    ## 2    11.0        1.10     NA          NA   foo       l.l        
    ## 3    22.0        2.20     22.0        22.0 2.2       2.2

Un buon sistema è:

1. importare tutte le colonne come testo:
-----------------------------------------

``` r
data <- read_csv2("colonne_numeriche_01.csv", 
                  col_types = paste(rep("c", ncol(data)), ### makes the "cccccc" string
                                    collapse = ""))
```

    ## Using ',' as decimal and '.' as grouping mark. Use read_delim() for more control.

``` r
str(data)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    3 obs. of  6 variables:
    ##  $ numbers    : chr  "0" "1.1" "2.2"
    ##  $ with_commas: chr  "0" "1,1" "2,2"
    ##  $ with_nas   : chr  "0" NA "2.2"
    ##  $ with_blanks: chr  "0" NA "2.2"
    ##  $ with_text  : chr  "0" "foo" "2.2"
    ##  $ with_errors: chr  "0" "l.l" "2.2"
    ##  - attr(*, "spec")=List of 2
    ##   ..$ cols   :List of 6
    ##   .. ..$ numbers    : list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   .. ..$ with_commas: list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   .. ..$ with_nas   : list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   .. ..$ with_blanks: list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   .. ..$ with_text  : list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   .. ..$ with_errors: list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   ..$ default: list()
    ##   .. ..- attr(*, "class")= chr  "collector_guess" "collector"
    ##   ..- attr(*, "class")= chr "col_spec"

``` r
knitr::kable(data)
```

| numbers | with\_commas | with\_nas | with\_blanks | with\_text | with\_errors |
|:--------|:-------------|:----------|:-------------|:-----------|:-------------|
| 0       | 0            | 0         | 0            | 0          | 0            |
| 1.1     | 1,1          | NA        | NA           | foo        | l.l          |
| 2.2     | 2,2          | 2.2       | 2.2          | 2.2        | 2.2          |

2. convertire le colonne in `numeric`
-------------------------------------

**NB:** i dati vengono salvati in un altro database!

``` r
library(magrittr)
data_numeric <- data
wrong_cols <- c("numbers", "with_commas", "with_nas", "with_blanks", "with_text", "with_errors")
data_numeric[, wrong_cols] %<>% lapply(function(x) as.numeric(as.character(x)))
```

    ## Warning in FUN(X[[i]], ...): si è prodotto un NA per coercizione

    ## Warning in FUN(X[[i]], ...): si è prodotto un NA per coercizione

    ## Warning in FUN(X[[i]], ...): si è prodotto un NA per coercizione

``` r
str(data_numeric)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    3 obs. of  6 variables:
    ##  $ numbers    : num  0 1.1 2.2
    ##  $ with_commas: num  0 NA NA
    ##  $ with_nas   : num  0 NA 2.2
    ##  $ with_blanks: num  0 NA 2.2
    ##  $ with_text  : num  0 NA 2.2
    ##  $ with_errors: num  0 NA 2.2
    ##  - attr(*, "spec")=List of 2
    ##   ..$ cols   :List of 6
    ##   .. ..$ numbers    : list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   .. ..$ with_commas: list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   .. ..$ with_nas   : list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   .. ..$ with_blanks: list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   .. ..$ with_text  : list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   .. ..$ with_errors: list()
    ##   .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
    ##   ..$ default: list()
    ##   .. ..- attr(*, "class")= chr  "collector_guess" "collector"
    ##   ..- attr(*, "class")= chr "col_spec"

``` r
data_numeric
```

    ## # A tibble: 3 x 6
    ##   numbers with_commas with_nas with_blanks with_text with_errors
    ##     <dbl>       <dbl>    <dbl>       <dbl>     <dbl>       <dbl>
    ## 1    0              0     0           0         0           0   
    ## 2    1.10          NA    NA          NA        NA          NA   
    ## 3    2.20          NA     2.20        2.20      2.20        2.20

Sono stati persi numerosi dati buoni, vediamo di recuperarli:

3. individuare i dati persi:
----------------------------

``` r
unlist(data)[which(is.na(data_numeric))]
```

    ## with_commas2 with_commas3    with_nas2 with_blanks2   with_text2 
    ##        "1,1"        "2,2"           NA           NA        "foo" 
    ## with_errors2 
    ##        "l.l"

se ci sono troppi dati ripetuti possiamo usare la funzione `unique()`, ma perderemo l'informazione circa la colonna a cui il dato appartiene:

``` r
unique(unlist(data)[which(is.na(data_numeric))])
```

    ## [1] "1,1" "2,2" NA    "foo" "l.l"

4. sostituire i dati sbagliati con quelli giusti:
-------------------------------------------------

Un metodo semplice è farlo dato per dato:

``` r
data_numeric[data == "1,1"] <- 1.1
data_numeric[data == "2,2"] <- 2.2
data_numeric[data == "l.l"] <- 1.1
```

``` r
data_numeric
```

    ## # A tibble: 3 x 6
    ##   numbers with_commas with_nas with_blanks with_text with_errors
    ##     <dbl>       <dbl>    <dbl>       <dbl>     <dbl>       <dbl>
    ## 1    0           0        0           0         0           0   
    ## 2    1.10        1.10    NA          NA        NA           1.10
    ## 3    2.20        2.20     2.20        2.20      2.20        2.20

5. profit
---------

Incassate i soldi!

------------------------------------------------------------------------

[Pagina dei TUTORIALS](../../tutorials/)

------------------------------------------------------------------------

------------------------------------------------------------------------

[Torna al Syllabus](../../README.md)

------------------------------------------------------------------------
