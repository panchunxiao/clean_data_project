# Code book for getting and cleaning data final project

#### 1. set the working directory and download and unzip the file

the folder called *UCI HAR Dataset*

```{r}
# setwd("C:/Users/Chunxiao Pan/Documents/R_Tutorial/Data_Science_Foundations_using_R_Specialization/clean_data_project")
download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip","./dataset.zip")
unzip ("dataset.zip", exdir = "./")
```

#### 2. read the file one by one and combine them into one data file

There are quite many text files that store different variables and the values, so it's a lot of work to read the files and combine them into one. We read 8 test files altogether, and assign them into each different names of the data, including features, activities, subject_test etc.

```{r}
# 1 Merges the training and the test sets to create one data set
features <- read.table("UCI HAR Dataset/features.txt", col.names = c("n","functions"))
activities <- read.table("UCI HAR Dataset/activity_labels.txt", col.names = c("code", "activity"))
subject_test <- read.table("UCI HAR Dataset/test/subject_test.txt", col.names = "subject")
x_test <- read.table("UCI HAR Dataset/test/X_test.txt", col.names = features$functions)
y_test <- read.table("UCI HAR Dataset/test/y_test.txt", col.names = "code")
subject_train <- read.table("UCI HAR Dataset/train/subject_train.txt", col.names = "subject")
x_train <- read.table("UCI HAR Dataset/train/X_train.txt", col.names = features$functions)
y_train <- read.table("UCI HAR Dataset/train/y_train.txt", col.names = "code")

X <- rbind(x_train, x_test)
Y <- rbind(y_train, y_test)
Subject <- rbind(subject_train, subject_test)
Merged_Data <- cbind(Subject, Y, X)
```

#### 3. select data of mean and std from each measurement, here I use dplyr package to perform the task which is a lot easier and precise.

```{r}
#2 Extracts the measurements on the mean and sd for each measurement
library(dplyr)
tidydt <- Merged_Data %>% 
  select(subject, code, contains("mean"), contains("std")) 
  
tidydt$code <- activities[tidydt$code, 2]

```

#### 4. here we use functions name() to list all the column names, and use gsub() to rename them.

*code* column in TidyData renamed into *activities*\
All *Acc* in column’s name replaced by *Accelerometer*\
All *Gyro* in column’s name replaced by *Gyroscope*\
All *BodyBody* in column’s name replaced by *Body*\
All *Mag* in column’s name replaced by *Magnitude*\
All start with character *f* in column’s name replaced by *Frequency*\
All start with character *t* in column’s name replaced by *Time*

```{r}
# name the data set with descriptive variable names:gsub(pattern, replacement, x) 
names(tidydt)[2] = "activity"
names(tidydt)<-gsub("Acc", "Accelerometer", names(tidydt))
names(tidydt)<-gsub("Gyro", "Gyroscope", names(tidydt))
names(tidydt)<-gsub("BodyBody", "Body", names(tidydt))
names(tidydt)<-gsub("Mag", "Magnitude", names(tidydt))
names(tidydt)<-gsub("^t", "Time", names(tidydt))
names(tidydt)<-gsub("^f", "Frequency", names(tidydt))
names(tidydt)<-gsub("tBody", "TimeBody", names(tidydt))
names(tidydt)<-gsub("-mean()", "Mean", names(tidydt), ignore.case = TRUE)
names(tidydt)<-gsub("-std()", "STD", names(tidydt), ignore.case = TRUE)
names(tidydt)<-gsub("-freq()", "Frequency", names(tidydt), ignore.case = TRUE)
names(tidydt)<-gsub("angle", "Angle", names(tidydt))
names(tidydt)<-gsub("gravity", "Gravity", names(tidydt))

```

#### 5. the mean is calculated according to different subject and activity, which we use summarize_all(funs(mean)) to perform the work. and it make a new file "tidydata2.txt"

```{r}
# create a second tidy data set with the mean of each variable for each activity and each subject
tidydt2 <- tidydt %>% 
  group_by(subject, activity) %>% 
  summarise_all(funs(mean))

print(tidydt2,n=10)

write.table(tidydt2,"./tidydata2.txt",row.names = FALSE)
```
