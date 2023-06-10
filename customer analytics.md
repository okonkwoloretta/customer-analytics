


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
 ---                                        
 
110:    Red Rock Deli Chikn&Garlic Aioli 150g 1434

111:      RRD SR Slow Rst     Pork Belly 150g 1526

112:                 RRD Pc Sea Salt     165g 1431

113:       Smith Crinkle Cut   Bolognese 150g 1451

114:                 Doritos Salsa Mild  300g 1472





