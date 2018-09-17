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

- notes: read_csv() is different from read.csv(). The former can keep all the zeros in data frame, but the latter will change the characters in data frame into numeric type automatically. For example, value"001" will be kept in read_csv() while being presented as "1" in read.csv(). So, it is better to use read_csv().

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

- notes: %>% is an infix operator, which is not part of base R, but is in fact defined by the package magrittr (CRAN) and is heavily used by dplyr (CRAN). If you use the %>%, it means that all the variables metioned after the infix operatior is under the former data frame. And in order for the changes you made with mutate() to be saved to your data frame, you need to **overwrite** the old data frame with the new, mutated one. 

```ruby
acc2014 <- acc2014%>%mutate(TWAY_ID2 = na_if(TWAY_ID2,""))
table(is.na(acc2014$TWAY_ID2))
```

Next, run the dim() function (which prints **number of rows then columns**) on both tibbles and notice that
the number of columns differs.
```ruby
dim(acc2014)#标记对象的维度#
dim(acc2015)
```

Use the colnames() function and the %in% operator, you can identify which columns appear in one dataset
by not the other. Indexing with square brackets can be useful here as well. Write a comment identifying
these columns (there are four in total) and which dataset(s) they are missing from.

- notes: %in% is another operatior which can help you check whether one element is in a set of elements. For example, x %in% y means to check whether x is in y. [] refers to a subset, so if you type a [] after a data frame, it means that you want R to return the elements that  meet the condition in [], and if you want the opposite results, just type"==FALSE". 

```ruby
colnames(file2014)[colnames(file2014)%in%colnames(file2015)==FALSE]
colnames(file2015)[colnames(file2015)%in%colnames(file2014)==FALSE]
#or you can write it as#
colnames(file2014)[!colnames(file2014)%in%colnames(file2015)]
colnames(file2015)[!colnames(file2015)%in%colnames(file2014)]
colnames(file2014)
colnames(file2015)
##IN file2014:ROAD_FNC;IN file2015:RUR_URB, FUNC_SYS;RD_OWNER)
```

Finally, use bind_rows() to combine these two tibbles into a single new tibble called acc. Use the count()
function to create a frequency table of the RUR_URB variable - why are there over 30,000 NA values for this
column? Explain in a comment.

```ruby
acc<- bind_rows(file2014,file2015)
View(acc)
acc%>%count(RUR_URB)
#because file2014 dosen't contain this field#
```

## Merging on another data source.
Read the fips.csv dataset into R (this file is attached to this assignment on Canvas) as fips. FIPS stands for
“Federal Information Processing Standards” and is a common set of codes for United States counties provided
by the US Census Bureau. You can use the glimpse() function to look at this data, which contains county
and state names that are missing from FARS data.

```ruby
fips <- read_csv('D:/UChicago/coding longlive!/R/fips.csv')
glimpse(fips)
```
The fips tibble contains columns named State.FIPS.Code and County.FIPS.Code whose values should
almost match the codes in the STATE and COUNTY columns of the acc tibble. However, these values do not
quite match, you must first:
- Convert the State and County variables in acc from integers to characters;
- Use the str_pad() function from the stringr package to add leading zeros so that all entries of the
State variable have two characters and all values of the County variable have three characters.
- Rename the State and County variables in acc to StateFIPSCode and CountyFIPSCode - the rename()
function may be valuable.

Merge the fips data onto the acc tibble using either left_join() or right_join() from the dplyr package.
You do not want to lose any data within acc, so the number of rows in the acc tibble should not change.
You will need to use both StateFIPSCode and CountyFIPSCode.

- notes: you can just check the usage of str_pad in help. And just a reminder for rename(), the changed name should be at the left of the "=". left_join() means that you only kept the data of the left-hand side while right_join() is reservse.

```ruby
as.character(acc$STATE)
as.character(acc$COUNTY)
acc$STATE <- str_pad(acc$STATE,2,side=c('left'),pad='0')
acc$COUNTY <- str_pad(acc$COUNTY,3,side=c('left'),pad='0')
acc <- rename(acc,StateFIPSCode=STATE ,CountyFIPSCode=COUNTY)

View(acc)
merge_fa<-left_join(fips,acc,by=c("StateFIPSCode"="StateFIPSCode","CountyFIPSCode"="CountyFIPSCode"))
View(merge_fa)
```

## Exploratory Data Analysis in R’s dplyr and tidyr package
- Use group_by and summarize() functions to calculate the total fatalities (using the FATALS variable)
by state for each of the past two years. Save this new variable as TOTAL and the data as a new tibble
called agg
- Use tidyr’s spread() function to change this new data format from long form to wide form. Save this
new tibble as agg_wide. Each state should now occupy only one row of data, with one column for
fatalities in 2014 and another column for fatalities in 2015.
Note: When using spread(), R may by default create new columns called 2014 and 2015, which you will
want to rename.(-notes: be sure to use '' since 2014 and 2015 are strings in this file)
- Use mutate() to calculate the percent difference between 2014 and 2015 fatalities for each state.
- Use arrange() to sort the tibble from largest increase to biggest percent decline.
- Use filter() to subset the data down to only the states with more than a 15% increase. Also remove
the NA state value.
- Rewrite the prior six steps using the same functions, but now only using one assignment <- with dplyr’s
chain operator: %>%

- notes: read carefully about the help of filter(), the observations will be kept if them match the conditions in the code. And becasue I didn't delete the NA in original data frame, so the wide one contains a 'NA' column. I delete it by subset().            

```ruby
#data analysis#
agg <- summarize( group_by(merge_fa,StateName,YEAR),TOTAL = sum(FATALS))
View(merge_fa)
View(agg)

#change data form from length to width#
agg_wide <- spread(agg,YEAR,TOTAL)
View(agg_wide)

agg_wide <- rename(agg_wide,Year2014='2014',Year2015='2015')

agg_wide <- agg_wide%>%mutate(Diff_Percent = (Year2015-Year2014)/Year2014)
agg_wide <- agg_wide%>%arrange(Diff_Percent)
agg_wide <- filter(agg_wide, Diff_Percent>0.15,StateName != 'NA')
##delete the 'NA' column##
agg_wide <- agg_wide%>%subset(select=c("StateName","Year2014","Year2015","Diff_Percent"))

glimpse(agg_wide)
```
## Lastly, ggplot2:
-notes: that's the most torturing part without many instructions!!! lol
Please recreate the graph below to the best of your ability. You will need to use the ggplot2 and scales
package to do so, with the ggrepel package being helpflu for exact replication. Note the y-axis is used a log
scale with a base of 10.

![example plot](https://raw.githubusercontent.com/haonen/Harris-Coding-Lab/master/ggplot%20example.JPG)

- notes: in order to make graphing easier, I change the data frame from width back to length by gather(). Also, for the convience of concision of ggplot2 commands, I use regular expression to change the name of Year2014 and Year2015(but you can also use rename() to make it easier). gsub is used for replacing text here. 

```ruby
#graphing#
#preparation#
agg_length <- gather(agg_wide,Year,Fatals,Year2014:Year2015)
View(agg_length)
agg_length$Year <- gsub('[Year]',"",agg_length$Year)

#graph now!#
graph <- ggplot(agg_length, aes(x = Year, y = Fatals, color = StateName, group = StateName)) + geom_line()+#basic plots#
  labs(y="Traffic Fatalitics(Log Base 10)", title="Traffic Fatalities Rise in Many States", subtitle="13 States Saw a 15% or Greater Rise in Traffic Fatalities",caption  ="DoT  FARS  Data")+##ylabs+title+subtitle+caption#
  scale_y_log10(breaks=c(10^2,10^2.5,10^3,10^3.5),labels=expression(10^{2},10^{2.5},10^{3},10^{3.5}))+annotation_logticks(base=10,sides="l",scaled=TRUE,short = unit(0.1, "cm"), mid = unit(0.2, "cm"), long = unit(0.3, "cm"),color="black")+#modification fo y-axis#
  theme_classic()+#basic patterns of this plot#
  theme(legend.position="none",axis.line.y =element_blank(),axis.line.x = element_blank())+#no legend, no x-axis and y-axis line#
  geom_text_repel(label = ifelse(agg_length$Year==2014,agg_length$StateName,""),nudge_x  =-0.25,direction  ="y",hjust  =0)#label for StateName only once#
  ```
  - notes: scale_y_log10() is used for designing scales of log base on 10 particularly. annotation_logsticks can help me to draw the ticks of y-axis and I think I just copy the parameters in help. Those are suggestions from other people. 
