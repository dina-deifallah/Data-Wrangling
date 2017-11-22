---
title: "README"
author: "DDeifallah"
date: "22 November 2017"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## R Markdown
This files containes all the required R script to tidy and clean the UCI HAR 
Dataset <http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones>.It is assumed here that the zip file containing the data files has been downloaded and unzipped in the working directory. The script below is divided into steps.

## Step 1
In this step we are saving the current working directory in wd variable and setting it to the unzipped Dataset folder for simplicity of files' reading

```{r echo = FALSE} 
wd <- getwd()
setwd("./UCI HAR Dataset")
```

## Step 2
Here we are loading the libraries we need

```{r}
library(dplyr)
library(plyr)
library(rmarkdown)
```
## Step 3
We are reading here the test data into a data frame

```{r}
subject_test <- read.table("./test/subject_test.txt") 
# this is the subject identifier for the test data
names(subject_test) <- "subject_id"
activity_test <- read.table("./test/y_test.txt")
# this is the activity number for the test data, ranges between 1 and 6
names(activity_test) <- "activity"
features_test <- read.table("./test/X_test.txt")
# this is the features for the test data, 561 features' readings for each record
## (the test data has 2947 records)
test_data <- cbind(subject_test,activity_test,features_test)
## binding all three dfs into one
```
## Step 4
We reading here the train data into a data frame

```{r}
subject_train <- read.table("./train/subject_train.txt") 
# this is the subject identifier for the train data
names(subject_train) <- "subject_id"
activity_train <- read.table("./train/y_train.txt")
# this is the activity number for the train data, ranges between 1 and 6
names(activity_train) <- "activity"
features_train <- read.table("./train/X_train.txt")
# this is the features for the train data, 561 features' readings for each record
## (the train data has 7352 records)
train_data <- cbind(subject_train,activity_train,features_train)
## binding all three dfs into one
```
## Step 5
We are merging here the training and the test sets to create one data set of 10299 records

```{r}

my_data <- arrange (rbind(test_data,train_data) , subject_id , activity)

```

## Step 6 
We are extracting here only the measurements on the mean and standard deviation
for each measurement. Please refer to features.txt, which was used to identify the indices of the required measurements

```{r}
features_data <- select(my_data , -c(subject_id , activity) )
## isolating the features' measurement to fascilitate extraction of required
## measurements
features_list <- read.table("features.txt")
select_features<- data.frame(matrix(nrow=10299, ncol=0))
select_indx <- grep( "std|mean"  , features_list$V2) # selecting only the mean
## and std of each measurement in the features data
select_features <- cbind(select_features , features_data[ , select_indx])
names(select_features) <- grep("std|mean" , features_list$V2 , value = TRUE) ##
## Appropriately labeling the selected features data set with descriptive
## variable names as in the file features.txt
tidy_data <- cbind(my_data[ ,1:2] , select_features)
```

## Step 7
We are naming the activities in the activity column in tidy_data with proper descriptive name:"walk", "walk_up", "walk_down" , "sit" , "stand" , "lay"

```{r}
tidy_data$activity <- factor(tidy_data$activity, levels = c(1,2,3,4,5,6), labels = c("walk", "walk_up", "walk_down" , "sit" , "stand" , "lay"))
```

## Step 8
We are constructing here the new tidy data frame with every feature measurement averaged over each subject_id and each activity

```{r}
by_sub_act <- tidy_data %>% group_by(subject_id, activity)
mean_tidy_data <- by_sub_act %>% summarise_all(mean)
```

## Step 9
we are returning here the working directory to the previous path

```{r}
setwd(wd)
```


