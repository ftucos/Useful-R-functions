# Useful R functions

## Priceless advice

[What They Forgot to Teach You About R](https://whattheyforgot.org/) 

- Nominare i file evitando spazi, accenti e puntuazione. Usa “-” per la human redability e “_” per isolare i metadata dal nome (es. "2020-02-10_sample1_WB") e identificarli facilmente con regex: 

```r
metadata <- stringr::str_split_fixed(x, [_\\.]
```

* Crea Rprojects: avviando un notebook da progetto sei sicuro che la wd sia settata nella cartella di appartenenza e che l’ambiente delle variabili sia quello giusto. Non usare mai patwhay assoluti, soprattutto mai usare setwd(). Lascia che sia l’Rproj a richiamare la WD

* Riavviare R prima di ogni esecuzione perchè:
  
  * potresti star usando funzioni omonime di librerie precedentemente caricate
  
  * un altro progetto potrebbe averti portato nella WD sbagliata
- lasciare settato options(stringsAsFactors = FALSE)

- Carica la libreria tidyverse per ultima, in modo da evitare che altre librerie mascherino le sue funzioni

## Basic R

#### define a function

```r
myfunction <- function(arg1, arg2, ... ){                  
  # statements  
  return(object)  
}
```

#### match operator

```r
x %in% c("value1", "value2", "value3")
```

#### opposite of match operator

```r
&#39;%!in%&#39; <- function(x,y)!(&#39;%in%&#39;(x,y))
```

## Magrittr

#### Pipe operator

```r
f(x) # is equivalent to
x %>% f

f(x, parameters) # is equivalent to
x %>% f(parameters) 

h(g(f(x))) # is equivalent to
x %>% f %>% g %>% h 

f(y, x) # is equivalent to
x %>% f(y, .)

f(y, data = x) # is equivalent to
x %>% f(y, data = .)

f(g(x))
x %>% {f(g(.))}
```

#### Compound assigmnent

```r
data$some_variable <- data$some_variable %>% transform 
# is equivalent to
data$some_variable %<>% transform
```

## Data exploration

```r
summary(iris)
skimr::skim(iris)
```

## Dplyr

#### Matches at least with one element of the list

```r
3 %in% c(1,2,3,4,5,6,7,8)  
[1] TRUE
```

#### Select rows by criteria

```r
flights[flights$month == 1 & flights$day == 1, ]
# is equivalent to
flights %>% filter(month == 1, day == 1)
# multiple arguments are equivalent to &
flights %>% filter(month == 1 & day == 1)
```

#### Select only certains columns

```r
# select a single column

flights %>% select(tailnum)

# select multiple column

flights %>% select(tailnum, day, month)

# select and rename the column

flights %>% select(tail_num = tailnum)

# select all but some columns

flights %>% select(-(year:day)) # or
flights %>% select(-c(year, month))

# select column based on theire name
flights %>% select(starts_with(c("one", "th")))
```

#### Rename a column

```r
flights %>% rename(tail_num = tailnum)
```

#### Move a colon to the first position

```r
df <- df %>% select(fisrt_column, second_column, everything())
```

#### Create a new column with formula

```r
mutate(flights,  
 gain = arr_delay - dep_delay,  
 gain_per_hour = gain / (air_time / 60)  
)
# use the function transmutate() if you want to remove the used columns
```

#### Replace a single value by criterion

```r
df <- df %>% mutate(height = replace(height, name == “Mike”, NA))
```

#### Summarize

```r
summarise(flights,  
  delay.mean = mean(dep_delay, na.rm = TRUE), delay.sd = sd(dep_delay, na.rm = TRUE)  
)
```

## tidyr

#### gather() - wide to long format

![](https://lh5.googleusercontent.com/sW47p8p8ifQo4ArWuXIWriSrw8d_7VPL94oa_OW2bbllXmjsxsB6yFE57Z9Y91isS2OoEbJiNvld_Hdm0qx0vhoftAqs_dGRawPhQV0L7UX0V-NRr1rIw5znSeLx_h92TseLCjsU)

#### spread() - long to wide format![](https://lh3.googleusercontent.com/paaS7OyTGxSAE5z9uOil23kgut2mBSF7qFCy8Uqx0slVcqnI2QQ-fsRAvGiQjtH8K6xFhJ75lF2JkgQOsO-dJmvFgBRQj39o79paDOvsEwJ2sJcv7uVXvmGs9BjcFZBOesFfnXMs)

## Favorite structures

#### Rename a specific column

```r
# rename the second column
colnames(data.frame)[2] <- "newname2"
```

#### Select all the rows that contain a certain match inside a specified column

```r
# selects all the rows containing a certain match inside a certain col
data.frame %>% filter(str_detect(col, fixed("match")))
```

#### Split the string inside a column into multiple column

```r
# will split a string like "CTRL-2day LowGlucose"
data.frame %>% separate(col, # the column to split  
 c("col1", "col2", "col3"), # names of the columns  
 sep = "[:punct:]|[:blank:]", # separator  
 remove = TRUE, # remove the original column  
)
```

#### Change Factor levels

```r
data.frame$col %<>% factor(levels = c("first", "second", "third"))
```

#### Calculate Standard Error

```r
st.err <- function(x) {sd(x, na.rm=TRUE)/sqrt(length(x[!is.na(x)]))
```

## ggplot2

#### Convert Y axis in percentage

```r
scale_y_continuous(labels = function(x) paste0(x*100, "%"))
```

#### Rename the legend

```r
+ scale_fill_x(name = 'Title')
```

#### Remove legend for one aesthetics

```r
+  guides(fill = FALSE)
# or
+ scale_fill_x(guide = FALSE)
```

#### Declare the specific color for each factor level

```r
myColors <- c("level1" = "#000000", "level2" = "#003300") 
# or
library(RColorBrewer)  
myColors  <- brewer.pal(length(levels(df$factors)), "Set1") 

names(myColors) <- levels(df$factors) 
+ scale_colour_manual(values = myColors)
```

## Generate random data

```r
set.seed(999)  
n = 1000  
df = data.frame(factors = sample(letters[1:8], n, replace = TRUE),  
x = rnorm(n), y = runif(n))
```

## Bioinformatics

#### Convert Gene Symbol in Ensembl and Entrez IDs

```r
#Genome wide annotation for Human  
library(org.Hs.eg.db)  
ENTREZID <- mapIds(org.Hs.eg.db, df$SYMBOL, 'ENTREZID', 'SYMBOL')
```

Use ALIAS in place of SYMBOL if you are not shure that you are using RefSeq/Hugo approved symbols

#### Convert Ensembl ID retriving data from BioMart

**Look-up ENSEMBL gene IDs:**

```r
ex <- c("ENSG00000215417", "ENSG00000224078", "ENSG00000198366",
"ENSG00000196176", "ENSG00000166012", "ENSG00000158406",
"ENSG00000196787")
mart <- useMart("ensembl", dataset = "hsapiens_gene_ensembl")
gene2genomeEx <- getBM(values = ex,
  filters = "ensembl_gene_id",
  mart = mart,
  attributes = c("ensembl_gene_id", "entrezgene_id",
    "hgnc_symbol", "external_gene_name",
    "description", "chromosome_name",
    "strand"))
gene2genomeEx

  ensembl_gene_id entrezgene hgnc_symbol external_gene_name
1 ENSG00000158406       8365    HIST1H4H           HIST1H4H
2 ENSG00000166012      79101       TAF1D              TAF1D
3 ENSG00000196787       8969   HIST1H2AG          HIST1H2AG
4 ENSG00000215417     407975     MIR17HG            MIR17HG
5 ENSG00000224078       3653      SNHG14             SNHG14
```

You can use also "ensembl_gene_id_version" for the format ENSG00000223972.4

**Look-up 'external' gene names:**

```r
ex <- c("ACTN4","TUBA1B","ACTN1","TP53")
mart <- useMart("ensembl", dataset = "hsapiens_gene_ensembl")
gene2genomeEx <- getBM(values = ex,
  filters = "external_gene_name",
  mart = mart,
  attributes = c("external_gene_name", "entrezgene_id",
    "hgnc_symbol", "description",
    "chromosome_name", "strand"))
gene2genomeEx

  external_gene_name entrezgene hgnc_symbol
1               TP53       7157        TP53
2              ACTN4         81       ACTN4
3             TUBA1B      10376      TUBA1B
4              ACTN1         87       ACTN1
5              ACTN4         81       ACTN4
                                            description chromosome_name strand
1 tumor protein p53 [Source:HGNC Symbol;Acc:HGNC:11998]              17     -1
2     actinin alpha 4 [Source:HGNC Symbol;Acc:HGNC:166]              19      1
3  tubulin alpha 1b [Source:HGNC Symbol;Acc:HGNC:18809]              12     -1
4     actinin alpha 1 [Source:HGNC Symbol;Acc:HGNC:163]              14     -1
5     actinin alpha 4 [Source:HGNC Symbol;Acc:HGNC:166]  CHR_HG26_PATCH      1
```
