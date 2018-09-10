# Harris-Coding-Lab
Recuperation of R 

This is a assignment for coding lab at Harris School of Public Policy. It involves a series of data analysis work, including importing data, data cleaning, basic statistics of frequency and ploting with ggplot2. 

## Load the FARS Data into R.
Use the **read_csv()** function from the readr package and the **read_sas()** function from the haven package
to load in the two accident datasets as acc2014 and acc2015. Use **ls()** to confirm these datasets have been
read into R’s memory. Use the class() function to examine the objects read into R - you should see they are of several classes.
Most importantly for our purposes, they are (1) of class **data.frame**, which is R’s original conception of
a tabular dataset and (2) of class **tbl**, pronounced tibble. Tibbles inherit the dataframe class and can be
considered a modern version of the dataframe with convenient defaults. 

- notes: read_csv() is different from read.csv(). The former can keep all the zeros in data frame,  for example, value"001" will be kept in read_csv() while being presented as "1" in read.csv(). So, it is better to use read_csv()
```ruby
#Assignment1:DoT FARS data#
#Yuwei Zhang 2020CAPP#
#intsall readr, haven, dplyr, tidyr, stringr, and ggplot2#

library(readr)
library(haven)
library(dplyr)
library(tidyr)
library(stringr)
library(ggplot2)
library(ggrepel)

acc2014 <- read_sas('D:/UChicago/coding longlive!/R/FARS2014NationalSAS/accident.sas7bdat')
acc2015 <- read_csv('D:/UChicago/coding longlive!/R/FARS2015NationalCSV/accident.csv')

ls()

class("acc2014")
class("acc2015")
```
## Combining the two years of FARS data
The two years of FARS data are not formatted exactly the same way, so some changes need to be made
before combining them. For instance, in acc2014, **missing data in the TWAY_ID2 variable is represented by an
empty string, while in acc2015 it is represented by NA**. NA, for Not Available, is the representation of missing
data in R.
Use the **mutate()** function and **na_if()** helper function from the dplyr library to convert the missing values
of TWAY_ID2 in the 2014 tibble from empty strings to NA values. Keep the name of the variable the same.

- notes: %>% is an infix operator, which is not part of base R, but is in fact defined by the package magrittr (CRAN) and is heavily used by dplyr (CRAN). 

```ruby
acc2014 <- acc2014%>%mutate(TWAY_ID2 = na_if(TWAY_ID2,""))
table(is.na(acc2014$TWAY_ID2))
```
