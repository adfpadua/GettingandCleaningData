#load relevant packages
library(plyr)
library(reshape2)
library(dplyr)

#read relevant tables

 train_set <- read.table("./data/W4P/train/X_train.txt")
 test_label <- read.table("./data/W4P/test/y_test.txt")
 train_label <- read.table("./data/W4P/train/y_train.txt")
 test_set <- read.table("./data/W4P//X_test.txt")
 sub_test <- read.table("./data/W4P/test/subject_test.txt")
 sub_train <- read.table("./data/W4P/train/subject_train.txt")
 features <- read.table("./data/W4P/features.txt")
 activity <- read.table("./data/W4P/activity_labels.txt")
 
 #create an index for each table
  train_set$IDTS <- rownames(train_set)
 train_label$IDTL <- rownames(train_label)
 sub_train$IDST <- rownames(sub_train)
 
 #merge tables "train_set" & "train_label" in a new table called "train_set_label"
  train_set_label <- merge(train_set, train_label, by.x="IDTS", by.y="IDTL", all=TRUE)
 
 #merge tables "train_set_label" & "sub_train" in a new table called "train_set_label_subject"
  train_set_label_subject <- merge(train_set_label, sub_train, by.x="IDTS", by.y="IDST", all=TRUE)
  
 # rename the table columns to specific names
  train_cons <- rename(train_set_label_subject, c("IDTS"="index", "V1.y"="label", "V1"="subject", "V1.x"="V1"))
  
  #create an index for each table
  test_set$ID <- rownames(test_set)
  test_label$ID <- rownames(test_label)
  sub_test$ID <- rownames(sub_test)
  
  #merge tables "test_set" & "test_label" in a new table called "test_set_label"
  test_set_label <- merge(test_set, test_label, by.x="ID", by.y="ID", all=TRUE)
  
  #merge tables "test_set_label" & "sub_test" in a new table called "test_set_label_subject"
  test_set_label_subject <- merge(test_set_label, sub_test, by.x="ID", by.y="ID", all=TRUE)
  
  # rename the table columns to specific names
  test_cons <- rename(test_set_label_subject, c("ID"="index", "V1.y"="label", "V1"="subject", "V1.x"="V1"))
  train_cons <- rename(train_set_label_subject, c("IDTS"="index", "V1.y"="label", "V1"="subject", "V1.x"="V1"))
  
  #binds tables "test_cons" & "train_cons"
  test_train_cons <- rbind(test_cons, train_cons)
  
  #substitutes the names of the columns by numbers, to match features index
  names(test_train_cons) <- sub("V", "", names(test_train_cons))
  
  #substitutes the names of the columns as per features names
  colnames(test_train_cons)[2:562] <- as.character(features$V2)
  
  #creates tables with columns which contains "-mean", "std, "index", "label", "subject"
  mean_std <- test_train_cons[, grepl("-mean", names(test_train_cons))]
  mean_std_1 <- test_train_cons[, grepl("std", names(test_train_cons))]
  mean_std_2 <- cbind(mean_std, mean_std_1)
  mean_std_index <- test_train_cons[, grepl("index", names(test_train_cons))]
  mean_std_label <- test_train_cons[, grepl("label", names(test_train_cons))]
  mean_std_subject <- test_train_cons[, grepl("subject", names(test_train_cons))]
  
  #creates a final table with all relevant colums
  mean_std_final <- cbind(mean_std_index, mean_std_label, mean_std_subject, mean_std_2)
  
  #creates a column with the names of the activities
  mean_std_final_act <- merge(mean_std_final, activity, by.x="mean_std_label", by.y="V1", all=TRUE)
  
  #Appropriately labels the data set with descriptive variable names
  names(mean_std_final_act) <- gsub("^t", "Time", names(mean_std_final_act))
  names(mean_std_final_act) <- gsub("^f", "Frequency", names(mean_std_final_act))
  descriptive <- rename(mean_std_final_act, c("mean_std_label"="activityCode", "mean_std_index"="index", "mean_std_subject"="subject", "V2"="activity"))
  
  group_sub_act <- group_by(descriptive, subject, activity, add = TRUE)
  
  table_sub_act <- summarize_each(group_sub_act, funs(mean)) 
  
  table_final <- as.data.frame(table_sub_act)
  
  head(table_final)
