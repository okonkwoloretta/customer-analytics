


## Load required libraries and datasets

```R
#### Load required libraries
library(data.table)
library(ggplot2)
library(ggmosaic)
library(readr)
library(readxl)
library(dplyr)
library(tidyr)
library(stringr)
library(arules)
```

```R
#### Import the dataset
transactionData <- read_excel("C:/Users/LORA/Documents/Quantium Virtual Internship/QVI_transaction_data.xlsx")
customerData <- read_csv("C:/Users/LORA/Documents/Quantium Virtual Internship/QVI_purchase_behaviour.csv", show_col_types = FALSE)
```

 Exploratory data analysis
```R
#### Examining transaction data
#### Let's take a look at each of the datasets provided
str(transactionData)
```
tibble [264,836 × 8] (S3: tbl_df/tbl/data.frame)

 $ DATE          : num [1:264836] 43390 43599 43605 43329 43330 ...
 
 $ STORE_NBR     : num [1:264836] 1 1 1 2 2 4 4 4 5 7 ...
 
 $ LYLTY_CARD_NBR: num [1:264836] 1000 1307 1343 2373 2426 ...
 
 $ TXN_ID        : num [1:264836] 1 348 383 974 1038 ...
 
 $ PROD_NBR      : num [1:264836] 5 66 61 69 108 57 16 24 42 52 ...
 
 $ PROD_NAME     : chr [1:264836] "Natural Chip        Compny SeaSalt175g" "CCs Nacho Cheese    175g" "Smiths Crinkle Cut  Chips Chicken 170g" "Smiths Chip Thinly  S/Cream&Onion 175g" ...
 
 $ PROD_QTY      : num [1:264836] 2 3 2 5 3 1 1 1 1 2 ...
 
 $ TOT_SALES     : num [1:264836] 6 6.3 2.9 15 13.8 5.1 5.7 3.6 3.9 7.2 ...
 
 
The "DATE" column in the "transaction_data" data frame is currently in numeric format, which means it is not in date format
convert "DATE" column to date format

```R
transactionData$DATE <- as.Date(transactionData$DATE, origin = "1899-12-30")
```

Check again if date column is in date format
```R
str(transactionData)
```

tibble [264,836 × 8] (S3: tbl_df/tbl/data.frame)

 $ DATE          : Date[1:264836], format: "2018-10-17" "2019-05-14" "2019-05-20" ...
 
 $ STORE_NBR     : num [1:264836] 1 1 1 2 2 4 4 4 5 7 ...
 
 $ LYLTY_CARD_NBR: num [1:264836] 1000 1307 1343 2373 2426 ...
 
 $ TXN_ID        : num [1:264836] 1 348 383 974 1038 ...
 
 $ PROD_NBR      : num [1:264836] 5 66 61 69 108 57 16 24 42 52 ...
 
 $ PROD_NAME     : chr [1:264836] "Natural Chip        Compny SeaSalt175g" "CCs Nacho Cheese    175g" "Smiths Crinkle Cut  Chips Chicken 170g" "Smiths Chip Thinly  S/Cream&Onion 175g" ...
 
 $ PROD_QTY      : num [1:264836] 2 3 2 5 3 1 1 1 1 2 ...
 
 $ TOT_SALES     : num [1:264836] 6 6.3 2.9 15 13.8 5.1 5.7 3.6 3.9 7.2 ...

We should check that we are looking at the right products by examining PROD_NAME

```R
#### Examine PROD_NAME
# Convert tibble to data.table
transactionData <- as.data.table(transactionData)

# Examine PROD_NAME
transactionData[, .N, by = PROD_NAME]
```

                                    PROD_NAME    N
                                    
  1:   Natural Chip        Compny SeaSalt175g 1468
  
  2:                 CCs Nacho Cheese    175g 1498
  
  3:   Smiths Crinkle Cut  Chips Chicken 170g 1484
  
  4:   Smiths Chip Thinly  S/Cream&Onion 175g 1473
  
  5: Kettle Tortilla ChpsHny&Jlpno Chili 150g 3296
                                         
110:    Red Rock Deli Chikn&Garlic Aioli 150g 1434

111:      RRD SR Slow Rst     Pork Belly 150g 1526

112:                 RRD Pc Sea Salt     165g 1431

113:       Smith Crinkle Cut   Bolognese 150g 1451

114:                 Doritos Salsa Mild  300g 1472


Looks like we are looking at potato chips but how can we check that these are all chips?
We can do some basic text analysis by summarising the individual words in the product name.

```R
#### Examine the words in PROD_NAME to see if there are any products that are not chips
productWords <- data.table(unlist(strsplit(unique(transactionData[,PROD_NAME]), " ")))
setnames(productWords, 'words')
```

As we are only interested in words that will tell us if the product is chips or not, let’s remove all words with
digits and special characters such as ‘&’ from our set of product words.

```R
### Using grepl
#### Removing digits
productWords <- productWords[grepl("\\d", words) == FALSE, ]
#### Removing special characters
productWords <- productWords[grepl("[:alpha:]", words), ]
```
Let's look at the most common words by counting the number of times a word appears and sorting them by this frequency in order of highest to lowest frequency

```R
#### sorting most common words by counting the number of times a word appears 
productWords[, .N, words][order(N, decreasing = TRUE)]
```

           words  N
           
  1:        Chips 21
  
  2:       Smiths 16
  
  3:      Crinkle 14
  
  4:       Kettle 13
  
  5:       Cheese 12
              
127: Chikn&Garlic  1

128:        Aioli  1

129:         Slow  1

130:        Belly  1

131:    Bolognese  1

131:    Bolognese  1

There are salsa products in the dataset but we are only interested in the chips category, so let’s remove
these.

```R
#### Add SALSA column to transaction_data
transactionData[, SALSA := grepl("salsa", tolower(PROD_NAME))]
#### Remove salsa products
transactionData <- transactionData[SALSA == FALSE, ][, SALSA := NULL]
```

Checking summary statistics such as mean, min and max values for each feature to see if there are any obvious outliers in the data and if there are any nulls in any of the columns
(NA's : number of nulls will appear in the output if there are any nulls).

    DATE              STORE_NBR     LYLTY_CARD_NBR        TXN_ID           PROD_NBR     
 Min.   :2018-07-01   Min.   :  1.0   Min.   :   1000   Min.   :      1   Min.   :  1.00  
 1st Qu.:2018-09-30   1st Qu.: 70.0   1st Qu.:  70015   1st Qu.:  67569   1st Qu.: 26.00  
 Median :2018-12-30   Median :130.0   Median : 130367   Median : 135183   Median : 53.00  
 Mean   :2018-12-30   Mean   :135.1   Mean   : 135531   Mean   : 135131   Mean   : 56.35  
 3rd Qu.:2019-03-31   3rd Qu.:203.0   3rd Qu.: 203084   3rd Qu.: 202654   3rd Qu.: 87.00  
 Max.   :2019-06-30   Max.   :272.0   Max.   :2373711   Max.   :2415841   Max.   :114.00  
  PROD_NAME            PROD_QTY         TOT_SALES      
 Length:246742      Min.   :  1.000   Min.   :  1.700  
 Class :character   1st Qu.:  2.000   1st Qu.:  5.800  
 Mode  :character   Median :  2.000   Median :  7.400  
                    Mean   :  1.908   Mean   :  7.321  
                    3rd Qu.:  2.000   3rd Qu.:  8.800  
                    Max.   :200.000   Max.   :650.000 





