The following libraries must be loaded:

library(plyr)
library(reshape2)
library(dplyr)

the files are loaded into tables and then index are created for each table in order to join the tables

columns are renamed to facilitate manipulation

temp tables are created to separate columns with mean and std

once tables for test and train are ok, they are bind together

Descriptive names are applied

the data is grouped by subject and activity and then summarized by mean for each group

a final table is created with the information needed.

