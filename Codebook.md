Codebook
========

This codebook describes the variables, the data, and any transformations or work performed to clean up the data


Variables
---------

Variable name     | Description
------------------|------------
subject           | Subject ID who performed the activity for each window sample. Its range is from 1 to 30.
activityName      | Activity name
featureName       | Variable name
average           | Average of each variable for each activity and each subject

Dataset structure
-----------------

```{r}
str(dtFinal)
```

###Visualizing the data
```{r}
dtFinal
```

###Summary of variables
```{r}
summary(dtTidy)
```


List of work performed to clean up the data
-------------------------------------------
###Download data
The data can be downloaded from "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"


###Load packages
```{r}
library(data.table)
library(reshape2)
library(dplyr)
```


###Read files
Suppose the data has been unzipped into the folder "./UCI HAR Dataset/"

####Load subject files 
```{r}
dtSubjectTrain <- tbl_df(fread("./UCI HAR Dataset/train/subject_train.txt"))
dtSubjectTest <- tbl_df(fread("./UCI HAR Dataset/test/subject_test.txt"))
```
####Load activity files 
```{r}
dtActivityTrain <- tbl_df(fread("./UCI HAR Dataset/train/Y_train.txt"))
dtActivityTest <- tbl_df(fread("./UCI HAR Dataset/test/Y_test.txt"))
```

####Load data files 
```{r}
dtTrain <- tbl_df(read.table("./UCI HAR Dataset/train/X_train.txt"))
dtTest  <- tbl_df(read.table("./UCI HAR Dataset/test/X_test.txt"))
```


###Merge the training and the test sets to create one data set
```{r}
dtSubject <- rbind(dtSubjectTrain, dtSubjectTest)
setnames(dtSubject, "V1", "subject")

dtActivity <- rbind(dtActivityTrain, dtActivityTest)
setnames(dtActivity, "V1", "activityCode")

dt <- rbind(dtTrain, dtTest)
dt <- cbind(dtSubject, dtActivity, dt)
```

###Extracts only the measurements on the mean and standard deviation for each measurement
```{r}
dtFeatures <- tbl_df(fread("./UCI HAR Dataset/features.txt"))
setnames(dtFeatures, c("featureCode", "featureName"))
dtFeatures1 <-
  dtFeatures %>%
  filter(grepl("mean\\(\\)|std\\(\\)", featureName)) %>%
  mutate(featureCode = paste0("V", featureCode))
dt2 <- dt[, c("subject","activityCode",dtFeatures1$featureCode)]
```

###Label the features with descriptive variable names
```{r}
setnames(dt2, names(dt2), c("subject","activityCode",dtFeatures1$featureName))
```

###Label the activities with descriptive activity names
```{r}
dtActivities <- tbl_df(fread("./UCI HAR Dataset/activity_labels.txt"))
setnames(dtActivities, names(dtActivities), c("activityCode", "activityName"))
dt2 <- merge(dt2, dtActivities, by="activityCode", all.x=TRUE)
dt2 <- select(dt2, -activityCode)
```

###Melt the data table to reshape it
```{r}
dt3 <- melt(dt2, id.vars=c("subject", "activityName"), variable.name="featureName")
```

###Create a tidy data set ith the average of each variable for each activity and each subject
```{r}
dtFinal <-
  dt3 %>%
  group_by(subject, activityName, featureName) %>%
  summarize(average = mean(value))
```

###Write tidy data set to a tab-delimited txt file
```{r}
write.table(dtFinal, "tidyData.txt", quote=FALSE, sep="\t", row.names=FALSE)
```
