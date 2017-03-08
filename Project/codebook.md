---
title: "run_analysis.Rmd"
author: "Horacio Ballinas"
date: "March 8, 2017"
output: html_document
---
# run_analysis
Last updated: 3/7/2017 on RStudio Version 1.0.136

Instructions for project
The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. You will be graded by your peers on a series of yes/no questions related to the project. You will be required to submit: 1) a tidy data set as described below, 2) a link to a GitHub repository with your script for performing the analysis, and 3) a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data called CodeBook.md. You should also include a README.md in the repo with your scripts. This repo explains how all of the scripts work and how they are connected.
One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained:

http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

Here are the data for the project:
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

You should create one R script called run_analysis.R that does the following.
1.	DONE Merges the training and the test sets to create one data set.
2.	DONE Extracts only the measurements on the mean and standard deviation for each measurement.
3.	DONE Uses descriptive activity names to name the activities in the data set.
4.	DONE Appropriately labels the data set with descriptive activity names.
5.	DONE Creates a second, independent tidy data set with the average of each variable for each activity and each subject.
Good luck!

## Preliminaries
Load packages.


```r
packages <- c("data.table", "reshape2", "knitr", "markdown")
sapply(packages, require, character.only = TRUE, quietly = TRUE)
```

Set path:


```r
path <- getwd()
path
```

## Get the data

Download the file. Put it in the Data folder. This was done on 2017-03-07


```r
url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
f <- "Dataset.zip"
if (!file.exists(f)) {
    file.create(f)
    download.file(url, file.path(path, f))
}
```

I proceeded to download an open-source, free unzip package called 7-Zip


```r
executable <- file.path("C:", "Program Files", "7-Zip", "7z.exe")
parameters <- "x"
cmd <- paste(paste0("\"", executable, "\""), parameters, paste0("\"", file.path(path, f), "\""))
system(cmd)
```

The archive put the files in a folder called UCI HAR Dataset on the working directory. We need to set this folder as the input path and list the files in the folder.


```r
pathIn <- file.path(path, "UCI HAR Dataset")
list.files(pathIn, recursive = TRUE)
```

See the README.txt file in C:\Users\Horacio\Desktop\Data-Science-JOHN-HOPKINS\datasciencecoursera\UCI HAR Dataset for detailed information on this dataset.

For the purposes of this project, the files in the Inertial Signals folders are not used, just the x_train, y_train and subject_test files are used.

## Read the files

Read the subject files.

```r
dtSubjectTrain <- fread(file.path(pathIn, "train", "subject_train.txt"))
dtSubjectTest <- fread(file.path(pathIn, "test", "subject_test.txt"))
```

Read the activity files. Called "label"" files in the README.txt documentation.

```r
dtActivityTrain <- fread(file.path(pathIn, "train", "Y_train.txt"))
dtActivityTest <- fread(file.path(pathIn, "test", "Y_test.txt"))
```

Read the data files with read.table.  fread did not work.


```r
fileToDataTable <- function(f) {
    df <- read.table(f)
    dt <- data.table(df)
}
dtTrain <- fileToDataTable(file.path(pathIn, "train", "X_train.txt"))
dtTest <- fileToDataTable(file.path(pathIn, "test", "X_test.txt"))
```

## Merge the training and the test sets

Concatenate the data tables.


```r
dtSubject <- rbind(dtSubjectTrain, dtSubjectTest)
setnames(dtSubject, "V1", "subject")
dtActivity <- rbind(dtActivityTrain, dtActivityTest)
setnames(dtActivity, "V1", "activityNum")
dt <- rbind(dtTrain, dtTest)
```

## Merge columns and set key


```r
dtSubject <- cbind(dtSubject, dtActivity)
dt <- cbind(dtSubject, dt)
setkey(dt, subject, activityNum)
```

## Extract only the mean and standard deviation
Read the features.txt file. This tells which variables in dt are measurements for the mean and standard deviation.

```r
dtFeatures <- fread(file.path(pathIn, "features.txt"))
setnames(dtFeatures, names(dtFeatures), c("featureNum", "featureName"))
```

Subset only measurements for the mean and standard deviation.


```r
dtFeatures <- dtFeatures[grepl("mean\\(\\)|std\\(\\)", featureName)]
```

Convert the column numbers to a vector of variable names matching columns in dt


```r
dtFeatures$featureCode <- dtFeatures[, paste0("V", featureNum)]
head(dtFeatures)
```

```r
dtFeatures$featureCode
```

Subset these variables using variable names.


```r
select <- c(key(dt), dtFeatures$featureCode)
dt <- dt[, select, with = FALSE]
```

## Use descriptive activity names

Read activity_labels.txt file. This will be used to add descriptive names to the activities.


```r
dtActivityNames <- fread(file.path(pathIn, "activity_labels.txt"))
setnames(dtActivityNames, names(dtActivityNames), c("activityNum", "activityName"))
```

## Label with descriptive activity names

Merge activity labels.


```r
dt <- merge(dt, dtActivityNames, by = "activityNum", all.x = TRUE)
```

Add activityName as a key.


```r
setkey(dt, subject, activityNum, activityName)
```

Melt the data table to reshape it from a short and wide format to a tall and narrow format


```r
dt <- data.table(melt(dt, key(dt), variable.name = "featureCode"))
```

Merge activity name.


```r
dt <- merge(dt, dtFeatures[, list(featureNum, featureCode, featureName)], by = "featureCode", all.x = TRUE)
```

Create a new variable, activity that is equivalent to activityName as a factor class. Create a new variable, feature that is equivalent to featureName as a factor class


```r
dt$activity <- factor(dt$activityName)
dt$feature <- factor(dt$featureName)
```

Separate features from featureName using the helper function grepthis


```r
grepthis <- function(regex) {
    grepl(regex, dt$feature)
}
## Features with 2 categories
n <- 2
y <- matrix(seq(1, n), nrow = n)
x <- matrix(c(grepthis("^t"), grepthis("^f")), ncol = nrow(y))
dt$featDomain <- factor(x %*% y, labels = c("Time", "Freq"))
x <- matrix(c(grepthis("Acc"), grepthis("Gyro")), ncol = nrow(y))
dt$featInstrument <- factor(x %*% y, labels = c("Accelerometer", "Gyroscope"))
x <- matrix(c(grepthis("BodyAcc"), grepthis("GravityAcc")), ncol = nrow(y))
dt$featAcceleration <- factor(x %*% y, labels = c(NA, "Body", "Gravity"))
x <- matrix(c(grepthis("mean()"), grepthis("std()")), ncol = nrow(y))
dt$featVariable <- factor(x %*% y, labels = c("Mean", "SD"))
## Features with 1 category
dt$featJerk <- factor(grepthis("Jerk"), labels = c(NA, "Jerk"))
dt$featMagnitude <- factor(grepthis("Mag"), labels = c(NA, "Magnitude"))
## Features with 3 categories
n <- 3
y <- matrix(seq(1, n), nrow = n)
x <- matrix(c(grepthis("-X"), grepthis("-Y"), grepthis("-Z")), ncol = nrow(y))
dt$featAxis <- factor(x %*% y, labels = c(NA, "X", "Y", "Z"))
```

Check to make sure all possible combinations of feature are accounted for by all possible combinations of the factor class variables


```r
r1 <- nrow(dt[, .N, by = c("feature")])
r2 <- nrow(dt[, .N, by = c("featDomain", "featAcceleration", "featInstrument", 
    "featJerk", "featMagnitude", "featVariable", "featAxis")])
r1 == r2
```

I accounted for all possible combinations. feature is now redundant.

## Create a tidy data set

Create a data set with the average of each variable for each activity and each subject.


```r
setkey(dt, subject, activity, featDomain, featAcceleration, featInstrument, 
    featJerk, featMagnitude, featVariable, featAxis)
dtTidy <- dt[, list(count = .N, average = mean(value)), by = key(dt)]
```

## Make CodeBook


