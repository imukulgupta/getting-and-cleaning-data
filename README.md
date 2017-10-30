---
title: "README"
date: "October 28, 2017"
output: html_document
---


### Repository
This repository contains:

- README.md 
- CodeBook.md
- run_analysis.R
- tidyData.R

### Data set

Site where the data was obtained:  
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

The zipfile consists of:

- Main folder: UCI HAR Dataset
- subfolders 'test' and 'train'
- separate files
    + activity_labels = activity Ids with corresponding activity name
    + features = list of all features in dataset
    + feature_info = information on each feature
    + README
- subfolders 'test'  and 'train' each contain:
    + folder Inertional Signals (not used)
    + subject_train/test = list of subject Ids
    + X_train/test  = data set
    + y_train/test = list of activity Ids

### Data processing

To process the dataset use run_analysis.R with working directory main folder 'UCI HAR Dataset'

All files except Inertional Signals are read as csv files with separator = "" and  header = FALSE:

```{r eval =F}
#import activity labels and features
activity_labels <- read.csv("activity_labels.txt", sep="", header = F)
features <- read.csv("features.txt", sep="", header = F)

#import test files
subject_test <-read.csv("test/subject_test.txt", sep="" , col.names = "subjectId", header = F)
X_test <-read.csv("test/X_test.txt", sep="", header = F)
y_test <-read.csv("test/Y_test.txt", col.names = "activityId", sep="", header = F)

#import train files
subject_train <-read.csv("train/subject_train.txt", sep="", col.names = "subjectId", header = F)
X_train <-read.csv("train/X_train.txt", sep="", header = F)
y_train <-read.csv("train/Y_train.txt", sep="", col.names = "activityId", header = F)
```

#### Merge the training and the test sets to create one data set

Both test and train sets have their own file with subject Ids and activity Ids, which can be combined using rbind. After merging test and train into a complete dataset, their respective subject ids and activity Ids can be added using cbind. 

```{r eval = F}
#combine test and train
completeSet <- rbind(X_test, X_train)
activityIds <-rbind(y_test, y_train)
subjectIds <- rbind(subject_test, subject_train)

#add subjectIds and activityIds to datasey
completeSet <- cbind(subjectIds, activityIds, completeSet)
```

#### Appropriately label the data set with descriptive variable names
Every row of the combined dataset represents a 561-feature vector with time and frequency domain variables. The columns must me labeled accordingly:

```{r eval = F}
#add appropriate labels to dataset
colnames(completeSet) <- features[,2]
```

To tidy the data the following steps are taken:

- merge the activity labels with completeSet on the activity Ids and drop the column with activity Ids. Every measurement is now labeled with a descriptivity variable name instead of a non-descriptive id number
- Rearrange the set according to subjectId and activity name

```{r eval = F}
#add activitynames to dataset and rearrange to tidy 
completeSet <- merge(activity_labels, completeSet, by.x = "V1", by.y = "activityId")
completeSet <- rename(completeSet, activityId = V1)
completeSet <- rename(completeSet, activity = V2)
completeSet <-arrange(completeSet, subjectId, activityId)
completeSet <- completeSet[,c(3, 2, 4:564)]
```

#### Extract only the measurements on the mean and standard deviation for each measurement
To create a second, independent tidy data set with the average of each variable for each activity and each subject a subset is created using only the columns containing 'mean' or 'std'. 

```{r eval = F}
#subset all the columns related to mean and standard deviation
subSet <- completeSet[,c(1,2,grep(pattern = "mean|std", x=colnames(completeSet)))]
subSet <- subSet[,c(!grepl(pattern = "meanFreq", x=colnames(subSet)))]
```


#### Create a second, independent tidy data set with the average of each variable for each activity and each subject
The subset is grouped by subject and activity to calculate the required means for every subject/activity combination:
```{r eval = F}
#create tidy data set with the average of each variable for each activity and each subject
tidyData <- subSet %>%
            group_by( subjectId, activity) %>%
            summarise_each(funs(mean))
```

To export tidyData *write.table* was used with row.names = FALSE

TidyData is tidy (long form) because:

- Each variable forms a column
- Each observation forms a row
- Column headers are variable names, not values
- No multiple variables stored in one column


### Libraries

- dplyr
