Leggere i dati da... Google Sheets
================

Caricamento delle librerie
--------------------------

Se non avete mai scaricato la libreria prima dovete installarla con il comando `install.packages("googlesheets")`, per buona prassi questo comando viene sempre eseguito dalla console, è **maleducazione** scrivere un file che mette le mani sui computer degli altri senza chiedere.

Dopodiché la libreria deve essere caricata:

``` r
library(googlesheets)
```

Un buon inizio per esplorare questo package è la sua [*vignette*](https://cran.r-project.org/web/packages/googlesheets/vignettes/basic-usage.html).

Vediamo un po' che combina
--------------------------

La funzione che si connette al mio account drive e mi dice quali files sono disponibili è:

``` r
(my_sheets <- gs_ls()) # Questa funzione ha le parentesi intorno... senza non funziona...
```

    ## # A tibble: 31 x 10
    ##    sheet_title  author perm  version updated             sheet_key ws_feed
    ##    <chr>        <chr>  <chr> <chr>   <dttm>              <chr>     <chr>  
    ##  1 030_dati_st… giova… rw    new     2018-04-18 15:06:59 1VW3BcZO… https:…
    ##  2 Iscritti La… giova… rw    new     2018-04-18 12:14:22 19U_JozM… https:…
    ##  3 "          … "   l… rw    new     2018-03-26 14:06:03 1dzvUxsd… https:…
    ##  4 "          … "   l… rw    new     2018-03-24 14:52:58 1gERmdAi… https:…
    ##  5 "          … "   l… rw    new     2018-03-24 14:52:40 1qB3n55o… https:…
    ##  6 Lista inter… giova… rw    new     2018-03-23 16:24:57 1mhapqjT… https:…
    ##  7 Risposte Bo… elisa… rw    new     2018-03-08 20:05:03 15v3uIyD… https:…
    ##  8 Risposte - … elisa… rw    new     2018-02-23 16:35:13 1GX-DleX… https:…
    ##  9 Foglio di l… giova… rw    new     2018-02-11 11:36:06 1LZdPHI-… https:…
    ## 10 Mailing lis… elisa… rw    new     2018-02-08 09:07:32 1cqE28c1… https:…
    ## # ... with 21 more rows, and 3 more variables: alternate <chr>,
    ## #   self <chr>, alt_key <chr>

L'oggetto che mi genera è una lista piuttosto incasinata, i titoli si leggono male, proviamo a stampare solo la colonna dei titoli:

``` r
my_sheets$sheet_title # andava bene anche "gs_ls()$sheet_title"
```

    ##  [1] "030_dati_studenti_eta_caramelle"                                       
    ##  [2] "Iscritti Laboratorio Statistica"                                       
    ##  [3] "Caselli 03"                                                            
    ##  [4] "Casole 01"                                                             
    ##  [5] "Casole 03"                                                             
    ##  [6] "Lista interessati corso statistica"                                    
    ##  [7] "Risposte Bosco degli Svizzeri"                                         
    ##  [8] "Risposte - Ponte Sospeso"                                              
    ##  [9] "Foglio di lavoro senza nome"                                           
    ## [10] "Mailing list 2° anno magistrale.xlsx"                                  
    ## [11] " La verna - gennaio (Risposte)"                                        
    ## [12] "Caselli 02"                                                            
    ## [13] "Caselli 01"                                                            
    ## [14] "Caselli 04"                                                            
    ## [15] "SFA2017_18"                                                            
    ## [16] "TUTTI"                                                                 
    ## [17] "uscita dicembre venerdì 15"                                            
    ## [18] "3° treinnale"                                                          
    ## [19] "1° triennale.xlsx"                                                     
    ## [20] "A spasso per la Toscana - Sasso Simone e Simoncello (Risposte)"        
    ## [21] "A spasso per la Toscana - Bivacco Lago Nero (Risposte)"                
    ## [22] "70560; How can I replace blank cells with zeros in Google Spreadsheet?"
    ## [23] "A spasso per la Toscana! - Scarlino (Risposte)"                        
    ## [24] "Castelvecchio (Risposte)"                                              
    ## [25] "A spasso per la Toscana! - Rocca Sillana (Risposte)"                   
    ## [26] "Foglio di lavoro senza nome"                                           
    ## [27] "A spasso per la Toscana! - Monti dell'Uccellina (Risposte)"            
    ## [28] "Alla Scoperta della Toscana! - Lago Nero AUTOMOBILI"                   
    ## [29] "Alla Scoperta della Toscana! - Parco dell'Uccellina"                   
    ## [30] "Bibliografia Consigliata"                                              
    ## [31] "testo esercitazione 29-10-2015.xls"

Il file che sto cercando è il secondo:

``` r
file_google_sheets <- gs_title("030_dati_studenti_eta_caramelle")
```

    ## Sheet successfully identified: "030_dati_studenti_eta_caramelle"

Se il file che volete non è dei vostri allora:

-   o avete la *chiave* e utilizzate la funzione `gs_key()`
-   o avete lo *sharing link* e utilizzate la funzione `gs_url()`

La lista dei fogli in quel file:

``` r
gs_ws_ls(file_google_sheets) ### gs = GoogleSheets, ws = WorkSheet, ls = LiSt
```

    ## [1] "Foglio1"

Salvo il `data frame` con il contenuto del foglio

``` r
dati_caramelle <- gs_read(file_google_sheets, ws = "Foglio1")
```

    ## Accessing worksheet titled 'Foglio1'.

    ## Parsed with column specification:
    ## cols(
    ##   nome = col_character(),
    ##   set_seed = col_integer(),
    ##   eta = col_integer(),
    ##   sample1 = col_integer(),
    ##   sample2 = col_integer(),
    ##   sample3 = col_integer(),
    ##   somma_dadi = col_integer(),
    ##   somma_dadi_eta = col_integer(),
    ##   peso_caramelle = col_integer()
    ## )

controlliamo cosa c'è:

``` r
dati_caramelle
```

    ## # A tibble: 18 x 9
    ##    nome   set_seed   eta sample1 sample2 sample3 somma_dadi somma_dadi_eta
    ##    <chr>     <int> <int>   <int>   <int>   <int>      <int>          <int>
    ##  1 giova…       42    36       6       6       2         14             50
    ##  2 Grego…       22    25       2       3       6         11             36
    ##  3 Carme…        9    24       2       1       2          5             29
    ##  4 vogli…       19    26       1       3       4          8             34
    ##  5 Saver…      123    25       2       5       3         10             35
    ##  6 Luca         74    25       3       6       4         13             38
    ##  7 Giova…       30    21       1       3       3          7             28
    ##  8 Betta        76    23       4       5       3         12             35
    ##  9 Ottav…       20    27       6       5       2         13             40
    ## 10 Aless…       10    29       4       2       3          9             38
    ## 11 Irene        11    26       2       1       4          7             33
    ## 12 Cicci…       33    23       3       3       3          9             32
    ## 13 Elia          4    24       4       1       2          7             31
    ## 14 Feder…       95    23       1       4       1          6             29
    ## 15 <NA>         NA    NA      NA      NA      NA         NA             NA
    ## 16 giada        57    24       2       4       1          7             31
    ## 17 Simone       34    22       5       1       2          8             30
    ## 18 tomma…        3    23       2       5       3         10             33
    ## # ... with 1 more variable: peso_caramelle <int>

``` r
knitr::kable(dati_caramelle)
```

| nome              |  set\_seed|  eta|  sample1|  sample2|  sample3|  somma\_dadi|  somma\_dadi\_eta|  peso\_caramelle|
|:------------------|----------:|----:|--------:|--------:|--------:|------------:|-----------------:|----------------:|
| giovanni          |         42|   36|        6|        6|        2|           14|                50|               NA|
| Gregorio          |         22|   25|        2|        3|        6|           11|                36|             2530|
| Carmelo           |          9|   24|        2|        1|        2|            5|                29|             2140|
| voglia di coccole |         19|   26|        1|        3|        4|            8|                34|             1540|
| Saverio           |        123|   25|        2|        5|        3|           10|                35|             1510|
| Luca              |         74|   25|        3|        6|        4|           13|                38|             1240|
| Giovanni          |         30|   21|        1|        3|        3|            7|                28|             1480|
| Betta             |         76|   23|        4|        5|        3|           12|                35|              970|
| Ottavia           |         20|   27|        6|        5|        2|           13|                40|             1470|
| Alessio           |         10|   29|        4|        2|        3|            9|                38|             1710|
| Irene             |         11|   26|        2|        1|        4|            7|                33|             2410|
| Ciccio XD         |         33|   23|        3|        3|        3|            9|                32|             1580|
| Elia              |          4|   24|        4|        1|        2|            7|                31|             1160|
| Federica          |         95|   23|        1|        4|        1|            6|                29|             1630|
| NA                |         NA|   NA|       NA|       NA|       NA|           NA|                NA|               NA|
| giada             |         57|   24|        2|        4|        1|            7|                31|             1310|
| Simone            |         34|   22|        5|        1|        2|            8|                30|              720|
| tommaso           |          3|   23|        2|        5|        3|           10|                33|             1380|

### Per concludere

Passo 1: si stabilisce la connessione con l'account:

``` r
(my_sheets <- gs_ls())
```

Passo 2: si salvano in un `dataframe` i dati del foglio giusto del file giusto:

``` r
nome_dataframe <- gs_read(gs_title("nome_del_file"), ws = "nome_del_foglio")
```

------------------------------------------------------------------------

[Pagina dei TUTORIALS](../../tutorials/)

------------------------------------------------------------------------

------------------------------------------------------------------------

[Torna al Syllabus](../../README.md)

------------------------------------------------------------------------
