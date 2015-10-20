
R script called run_analysis.R does the following:

1. Merges the training and the test sets to create one data set.
  * Reads data from training and testing datasets and merges them together
  * Uses rbind() to merge both data frames.

```r
library(dplyr)

# Read features
features <- read.table("UCI HAR Dataset/features.txt", 
                       header = FALSE, 
                       col.names = c("code", "name"))


# read training data set
X_train <- read.table("UCI HAR Dataset/train/X_train.txt", 
                      col.names = features$name)

# read testing data set
X_test <- read.table("UCI HAR Dataset/test/X_test.txt", 
                     col.names = features$name)

# 1. Merging the training and the test sets to create one data set.
X <- rbind(X_train, X_test)

```

2. Extracts only the measurements on the mean and standard deviation for each measurement. 
  * Creates a subset of data with just 66 columns that has only mean and standard deviation data.

```r
# 2. Extracting only the measurements on the mean and standard 
#    deviation for each measurement.
targetFeatures <- features[grep("(mean|std)\\(\\)", features$name),]

X_subset <- X[, targetFeatures$name]

# remove objects that are no longer needed
remove(X_train, X_test, targetFeatures, features)
```

3. Uses descriptive activity names to name the activities in the data set
  * Reads y datasets and merges them together
  * Uses merge command to resolve activity labels in the dataset.

```r
# 3. Uses descriptive activity names to name the activities in the 
#    data set

activityLabels <- read.table("UCI HAR Dataset/activity_labels.txt", 
                             col.names = c("code", "activity"))

y_test <- read.table("UCI HAR Dataset/test/y_test.txt", 
                     col.names = c("code"))

y_train <- read.table("UCI HAR Dataset/train/y_train.txt", 
                      col.names = c("code"))

y <- rbind(y_train, y_test) 

y2 <- merge(y, 
            activityLabels, 
            by = c("code"), 
            all.x = TRUE, 
            sort = FALSE)

X <- cbind(y2$activity, X)
X_subset <- cbind(y2$activity, X_subset)

names(X)[1] <- "activity"
names(X_subset)[1] <- "activity"

# remove y and activityLabels datasets since they are no longer needed.
remove(y_train, y_test, activityLabels, y, y2)
```

4. Appropriately labels the data set with descriptive variable names. 
  * Replaces abbreviations "Acc", "Mag", "Gyro" with more descriptive words "Accelerometer", "Magnitude", and "Gyroscope". Also replace prefixes "t" and "f" with "Time" and "Freguency". Additionally replaces all ".." and "..." in the column names with ".".

```r
# 4. Appropriately labels the data set with descriptive variable names. 

# Tidy up Feature Names
featureNames <- names(X)

featureNames <- gsub("Acc", "Accelerometer", featureNames)
featureNames <- gsub("Mag", "Magnitude", featureNames)
featureNames <- gsub("Gyro", "Gyroscope", featureNames)
featureNames <- sub("^t", "Time", featureNames)
featureNames <- sub("^f", "Frequency", featureNames)
featureNames <- sub("\\.\\.\\.", ".", featureNames)
featureNames <- sub("\\.\\.", ".", featureNames)
featureNames <- sub("\\.tBody", ".TimeBody", featureNames)

names(X) <- featureNames

featureNames <- names(X_subset)

featureNames <- gsub("Acc", "Accelerometer", featureNames)
featureNames <- gsub("Mag", "Magnitude", featureNames)
featureNames <- gsub("Gyro", "Gyroscope", featureNames)
featureNames <- sub("^t", "Time", featureNames)
featureNames <- sub("^f", "Frequency", featureNames)
featureNames <- sub("\\.\\.\\.", ".", featureNames)
featureNames <- sub("\\.\\.", ".", featureNames)
featureNames <- sub("\\.tBody", ".TimeBody", featureNames)

names(X_subset) <- featureNames

remove(featureNames)
```

5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
  * Reads subject datasets and merges them together.
  * Adds subject column to the dataset.
  * Uses dplyr to calculate the mean of all columns grouped by subject and activity.
  * Writes the results to a file named tidy_X_subset.txt.

```r
# 5. From the data set in step 4, creates a second, independent tidy 
#    data set with the average of each variable for each activity and 
#    each subject.

subjects_train <- read.table("UCI HAR Dataset/train/subject_train.txt")
subjects_test <- read.table("UCI HAR Dataset/test/subject_test.txt")

subjects <- rbind(subjects_train, subjects_test) 

subjects$subjectName <- paste("Subject", subjects[, 1])

X <- cbind(subjects$subjectName, X)
X_subset <- cbind(subjects$subjectName, X_subset)

names(X)[1] <- "subject"
names(X_subset)[1] <- "subject"

# remove original datasets since they are no longer needed.
remove(subjects_train, subjects_test, subjects)

tidy_X_subset <- 
  X_subset %>%                                
  group_by(subject, activity) %>%                    
  summarize_each(funs(mean))

write.table(tidy_X_subset, file = "tidy_X_subset.txt", row.names = FALSE)
```


